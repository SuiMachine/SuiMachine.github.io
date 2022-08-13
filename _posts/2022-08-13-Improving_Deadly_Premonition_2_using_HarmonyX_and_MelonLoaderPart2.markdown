---
layout: post
title:  "Improving Deadly Premonition 2 using HarmonyX and MelonLoader - Part 2"
date:   2022-08-13 15:00:00 +0200
---

This is an addition to the blog post entry I have written a month ago about how I managed to improve and fix some aspects of Deadly Premonition 2 using HarmonyX and MelonLoader. In it, I did mention that the terrains used by the game are not using Unity's terrain component and instead were converted to be meshes and use Mesh Renderer and Mesh Colliders. I also mentioned that terrains often don't connect to each other correctly, having visible gaps (gaps that are on a few occasions so massive, a player can fall through them) and there would be a chance to fix them if they were using terrain component.

[![broken_terrain.jpg](/images/articles/deadlypremonition2/broken_terrain.jpg)](/images/articles/deadlypremonition2/broken_terrain.jpg)

Turns out, I was wrong and all meshes used by mesh filters can be exported with a bit of clever code writing.

<!--more-->

Exporting meshes
---
To export meshes, we essentially need to add a component to the game - this can easily be done with Melon Loader (as mentioned in [Part 1]({% post_url 2022-07-07-Improving_Deadly_Premonition_2_using_HarmonyX_and_MelonLoader %})). Then we just need to perform the code:

```cs
Directory.CreateDirectory("Squares");
var path = Path.Combine("Squares", component.name + ".xml");
if (File.Exists(path))
    continue;

var mf = component.GetComponent<MeshFilter>();
if (mf == null)
{
    SuisHackMain.loggerInst.Msg($"Mesh Filter was null in {component.name}");
    continue;
}

if (mf.sharedMesh == null)
{
    SuisHackMain.loggerInst.Msg($"Mesh was null in {component.name}");
    continue;
}

var verts = mf.sharedMesh.vertices;
var triangles = mf.sharedMesh.triangles;
var uvs = mf.sharedMesh.uv;

var store = new Store();
store.verts = new Vector3[verts.Length];

for (int l = 0; l < verts.Length; l++)
{
    store.verts[l] = verts[l];
}

store.triangles = new int[triangles.Length];
for (int l = 0; l < triangles.Length; l++)
{
    store.triangles[l] = triangles[l];
}

store.uvs = new Vector2[uvs.Length];
for (int l = 0; l < uvs.Length; l++)
{
    store.uvs[l] = uvs[l];
}

var fileWritter = new StreamWriter(path);
XmlSerializer serializer = new XmlSerializer(typeof(Store));
serializer.Serialize(fileWritter, store);
fileWritter.Close();
```

`Store` is a basic struct that allows us to serialize them cleanly:
```cs
[Serializable]
public struct Store
{
	public Vector3[] verts;
	public int[] triangles;
	public Vector2[] uvs;
}
```

This way, we get nice (and oversized) information about how to build a model in XML files.

Of course, since Deadly Premonition 2 is an open-world game and constantly loads and unloads terrain chunks, we can't just use FindObjectsOfType to export them all at once. My solution to that problem was just to make a script that iterates over objects in the scene and checks if there is a terrain square that hasn't been exported yet and if it finds one - then it exports it. Then all it was left to do is just to drive around the town for like an hour or two making sure all playable squares get exported.

The next step is to reuse this struct and read it on the side of Unity to build meshes out of them. Technically we could also get normals and tangents of the mesh, but for the basic purpose of just correcting the positions of vertices this is already more than enough. We could even not use UVs and we'd still probably be fine.

Importing to Unity
---
Since the squares used by Deadly Premonition 2 have a fixed width and lenght and their position is provided by object name, assuming we exported them to files that have the name of the game object, we can reuse that information to mass import all squares. First off - getting names:

```cs
var squareName = Path.GetFileNameWithoutExtension(square);
var regex = new Regex(@"Terrain_([A-Z])_(\d+?)_x(\d+)_y(\d+)", RegexOptions.IgnoreCase);
var match = regex.Match(squareName);
if (match.Success)
{
    var captureGroups = match.Groups;
    var majorX = (int)captureGroups[1].Value[0] - (int)'A';
    var majorY = float.Parse(captureGroups[2].Value);
    var position = new Vector3(majorX * -512, 0, majorY * 512);

    (...)
}
```

The terrain game object names are all formed like this: `Terrain_B_05_x0_y0`.
So first, we use regular expressions to obtain "B" and "05" as "majorX" and "majorY" positions. This naming might be a bit confusing though, as it should technically be "majorZ" as Unity uses this order of axis:
- X - left / right
- Y - up / down
- Z - forward / backward

This is something that you always have to remember about and not just when working with Unity, as different programs have different order of axis, with - if I remember correctly - 3D Studio Max for example using Z as up/down and Y as forward / backwards. It is very common to just export a model from a 3D modelling program, import it to Unity (or other engines) and have the entire model flipped, due to the engine expecting different order of axis.

Anyway, using the above example of `Terrain_B_05_x0_y0`, we obtain 1 letter, and 3 numbers - beware... paint!
[![positions.png](/images/articles/deadlypremonition2/positions.png)](/images/articles/deadlypremonition2/positions.png)

First letter - B. This is a bit tricky, as it is a first position divided by 512. What I mean is:
* A = 0
* B = 1
* C = 2
* D = 3

etc.

Since writing a massive switch case would be just silly and it would be more of what an amateur programmer making a yandere game would do, than what an actual programmer with experience would do - we are just going to do some type casting to convert it into a number with this line of code:
```cs
var majorX = (int)'B' - (int)'A';
```

Nice and easy. And so, in our regex matching this becomes:
```cs
var majorX = (int)captureGroups[1].Value[0] - (int)'A';
```
As stated before - that letter is a position divided by 512 or more to the point - it tells us about where on the grid that terrain square is located, where the resolution of each square is 512 units. And so we build a vector from the first letter (converted into a number) and first number. However, after some experimentation, it turned out that the order of squares wasn't correct and so I had to invert one of the axes, which was done by multiplying majorX by negative 512 instead of positive.
```cs
var position = new Vector3(majorX * -512, 0, majorY * 512);
```

Now for the 3rd and 4th numbers. These can only be either 0 or 1, as in x0 or x1 and y0 or y1.
And they tell us whether the square's vertices are offset by 256 or not. Since these are positions of vertices within the mesh - we are just going to ignore them for now and perform the rest of the code to import the squares:

```cs
var reader = new StreamReader(square);
var deseriliser = new XmlSerializer(typeof(Store));
var meshData = (Store)deseriliser.Deserialize(reader);
reader.Close();

var GameObject = new GameObject(squareName);
var meshFilter = GameObject.AddComponent<MeshFilter>();
var meshRenderer = GameObject.AddComponent<MeshRenderer>();
var meshCorrections = GameObject.AddComponent<MeshCorrections>(); //a custom component that I used to attach some additional information
meshCorrections.OriginalDefinitionPath = square;
var mesh = new Mesh();
mesh.vertices = meshData.verts;
mesh.uv = meshData.uvs;
mesh.triangles = meshData.triangles;
mesh.RecalculateBounds();
mesh.RecalculateNormals();
mesh.RecalculateTangents();

meshFilter.sharedMesh = mesh;
meshRenderer.material = material;
GameObject.transform.position = position;
```

And there we go:
[![terrain_in_unity.jpg](/images/articles/deadlypremonition2/terrain_in_unity.jpg)](/images/articles/deadlypremonition2/terrain_in_unity.jpg)

Finding edge vertices
---
Since each terrain has many, many vertices, to make the script that is going to correct all the broken connections between them, first we have to find the vertices that are on the edge of the square. 
[![terrain_verts.jpg](/images/articles/deadlypremonition2/terrain_verts.jpg)](/images/articles/deadlypremonition2/terrain_verts.jpg)

To make the work simple - since I am not that great at math, I have decided to use a Bounds struct that is already defined in Unity. It is axis aligned struct, which is very useful if one needs to check if the point isn't inside another box. Since our terrains are always axis aligned - this make is perfect for our needs. First, we create it in the position where the first vertex is and then we use encapsulation to extend its size in a way that contains every other vertex we provide to it.

```cs
var bounds = new Bounds(EdgeVerts[0], Vector3.zero);
for (int i = 1; i < meshFilter.sharedMesh.vertices.Lenght; i++)
{
    bounds.Encapsulate(meshFilter.sharedMesh.vertices[i]);
}
```

What we get is an axis-aligned box that contains ALL the vertices of the mesh inside of it. How does this help? Well, this box can be scaled and so we change its X and Z size:
```cs
var middleFilterBounds = new Bounds(bounds.center, new Vector3(bounds.size.x - 5f, bounds.size.y, bounds.size.z - 5f));
```

Then we can just use a method `Contains` (and some Linq) to see if a vertex is inside of our newly scaled box and get all of the vertices that are outside of it.
```cs
EdgeVerts = EdgeVerts.Where(x => !middleFilterBounds.Contains(x)).ToList();
```

[![terrain_verts_outside.jpg](/images/articles/deadlypremonition2/terrain_verts_outside.jpg)](/images/articles/deadlypremonition2/terrain_verts_outside.jpg)

Then using 2 more bounds (one a bit wider on the X-axis and one a bit wider on the Y-axis, we can get information on which vertices are in the corners of each terrain square).

```cs
var boundsCorner1 = new Bounds(bounds.center, new Vector3(bounds.size.x - 5f, bounds.size.y, bounds.size.z));
var boundsCorner2 = new Bounds(bounds.center, new Vector3(bounds.size.x, bounds.size.y, bounds.size.z - 5f));

CornerVerts = EdgeVerts.Where(x => !(boundsCorner1.Contains(x.Value) || boundsCorner2.Contains(x.Value))).ToDictionary(x => x.Key, x => x.Value);
```

And finally, we can once again process EdgeVerts to get rid of the vertices that are now found to be corner vertices:
```cs
EdgeVerts = EdgeVerts.Where(x => !CornerVerts.ContainsKey(x.Key)).ToDictionary(x => x.Key, x => x.Value);
```

[![terrain_verts_corner.jpg](/images/articles/deadlypremonition2/terrain_verts_corner.jpg)](/images/articles/deadlypremonition2/terrain_verts_corner.jpg)

Processing
---
What's left to do is get the world position of each square, find its neighbour, get the position of each vertex in world coordinates (as the mesh coordinates are local ones) and check each neighbour finding the closest neighbour in X and Z axis. If the vertex is an edge vertex, a minimum of 0 and a maximum of 1 neighbour vertices should be found. If the vertex is a corner vertex up to 3 neighbour vertices can be found. We do have to remember the position inside of the array for each vertex, though unless we want to rebuild entire triangles array in the process.

Once this is done, we compare height differences - if the height difference is 0 - our neighbour vertices overlap with each other and nothing has to be done. If the difference isn't 0 a correction has to be made - whatever it's a corner vertex or edge vertex the idea is the same - calculate the average height and set that height for all of the vertices that have the same X/Z position. Also set the flag if a correction was made, to make sure, we can easily differentiate terrains that need to have their vertex positions changed and which stayed the same.

```cs
public void CorrectConnections()
{
    this.GenerateWorldPosEdgeVertecies();
    var pos = GetPosition();
	Neighbours = FindObjectsOfType<MeshCorrections>().Where(x => x.GetPosition() == pos + new Vector2(0, -1) || x.GetPosition() == pos + new Vector2(0, 1) || x.GetPosition() == pos + new Vector2(-1, 0) || x.GetPosition() == pos + new Vector2(1, 0)).ToArray();
	foreach (var Neighbour in Neighbours)
	{
		Neighbour.GenerateWorldPosEdgeVertecies();
	}

	List<UpgradeInformation> modifiedKeyPairs = new List<UpgradeInformation>();

	foreach (var edgeVert in EdgeVerts)
	{
		bool foundExact = false;
		var closestVert = new Vector3(float.MaxValue, float.MaxValue, float.MaxValue);
		var thisVert2D = new Vector2(edgeVert.Value.x, edgeVert.Value.z);

		var connectedVerts = new List<UpgradeInformation>();
	    connectedVerts.Add(new UpgradeInformation(this, edgeVert.Key, edgeVert.Value));

		foreach (var neighbour in Neighbours)
		{
			foreach (var neighbourVert in neighbour.EdgeVerts)
			{
		    	var checkVert2D = new Vector2(neighbourVert.Value.x, neighbourVert.Value.z);
				var distance = Vector2.Distance(thisVert2D, checkVert2D);
				if (distance < 1)
				{
					var distance3D = Vector3.Distance(edgeVert.Value, neighbourVert.Value);
					if (distance3D == 0)
					{
						foundExact = true;
						break;
					}
					else
					{
						connectedVerts.Add(new UpgradeInformation(neighbour, neighbourVert.Key, neighbourVert.Value));
					}
				}
			}

			if (foundExact)
				break;
		}
		
        if (foundExact)
			continue;
		else if(connectedVerts.Count > 1)
		{
			NeededCorrection = true;

			var averagePosition = Average(connectedVerts.Select(x => x.Position).ToArray());

			foreach(var connectedVert in connectedVerts)
			{
				modifiedKeyPairs.Add(new UpgradeInformation(connectedVert.meshCorrectionsReference, connectedVert.VertexID, averagePosition));
			}
		}
	}

	foreach (var cornerVert in CornerVerts)
	{
		var closestVert = new Vector3(float.MaxValue, float.MaxValue, float.MaxValue);
		var thisVert2D = new Vector2(cornerVert.Value.x, cornerVert.Value.z);

		var connectedVerts = new List<UpgradeInformation>();
		connectedVerts.Add(new UpgradeInformation(this, cornerVert.Key, cornerVert.Value));

		foreach (var neighbour in Neighbours)
		{
	    	foreach (var neighbourCornerVert in neighbour.CornerVerts)
			{
				var checkVert2D = new Vector2(neighbourCornerVert.Value.x, neighbourCornerVert.Value.z);
				var distance = Vector2.Distance(thisVert2D, checkVert2D);
				if (distance < 1)
				{
					var distance3D = Vector3.Distance(cornerVert.Value, neighbourCornerVert.Value);
					if (distance3D == 0)
					{
						continue;
					}
					else
					{
						connectedVerts.Add(new UpgradeInformation(neighbour, neighbourCornerVert.Key, neighbourCornerVert.Value));
					}
				}
			}
		}

		if (connectedVerts.Count > 1)
		{
			NeededCorrection = true;

			var averagePosition = Average(connectedVerts.Select(x => x.Position).ToArray());

			foreach (var connectedVert in connectedVerts)
			{
				modifiedKeyPairs.Add(new UpgradeInformation(connectedVert.meshCorrectionsReference, connectedVert.VertexID, averagePosition));
			}
		}
	}

	if (NeededCorrection)
	{
		var squaresToUpdate = modifiedKeyPairs.Select(x => x.meshCorrectionsReference).Distinct().ToArray();
		foreach (var squareToUpdate in squaresToUpdate)
		{
			var vertsToupgrade = modifiedKeyPairs.Where(x => x.meshCorrectionsReference == squareToUpdate).Select(x => new KeyValuePair<int, Vector3>(x.VertexID, x.Position)).ToList();
			squareToUpdate.UpdateVertecies(vertsToupgrade);
		}
	}
}

public void UpdateVertecies(List<KeyValuePair<int, Vector3>> elementsToUpdate)
{
	var mesh = GetComponent<MeshFilter>();

	var meshVerts = mesh.sharedMesh.vertices.ToList();
	foreach(var keypair in elementsToUpdate)
	{
		meshVerts[keypair.Key] = keypair.Value - this.transform.position;
	}

	mesh.sharedMesh.SetVertices(meshVerts);
	NeededCorrection = true;

	EditorUtility.SetDirty(mesh);
}
```

Technically the above code isn't "quite correct", as it doesn't check for diagonal neighbours. However - because it is going to be run for all of the terrain squares - that does not matter much.

Exporting vertices back to the game
---
Finally, after these are corrected it is time to export them back to the game. I was originally thinking about using an asset bundle or serializable struct, but ultimately all I need is positions of vertices. I don't need information about which vertices are used to make triangles and I don't need UVs - both stay as they were. And since I had a problem doing some serialization on the game's side, I have decided to simply store vertices using the binary format as sequences of float (single) values. In hindsight - all I needed to do was simply export the Y position of each vertex, but what I did was export all 3 coordinates of all vertices of each "broken" terrain square. There are no null separators or anything like that in the file. Just sets of 3 float values.

```cs
public void Export()
{
    if (!NeededCorrection)
        return;
        
    var path = Path.Combine(Application.streamingAssetsPath, "TerrainFixes", this.gameObject.name + ".bin");

	if (!Directory.Exists(Directory.GetParent(path).FullName))
		Directory.CreateDirectory(Directory.GetParent(path).FullName);

	if (File.Exists(path))
		File.Delete(path);

	FileStream fs = new FileStream(path, FileMode.CreateNew);
	var binaryWritter = new BinaryWriter(fs);
	var mf = GetComponent<MeshFilter>();

	foreach(var vertex in mf.sharedMesh.vertices)
	{
		binaryWritter.Write(vertex.x);
		binaryWritter.Write(vertex.y);
		binaryWritter.Write(vertex.z);
	}

	fs.Close();
}
```

Then all that's left is to have a way to differentiate squares which were "checked" and which need to be checked, I have done this by adding a custom component, which also checks if the file that contains corrected vertices positions exists and if it does, loads it and applies it.
```cs
[RegisterTypeInIl2Cpp]
public class TerrainCorrectionData : MonoBehaviour
{
	public TerrainCorrectionData(IntPtr ptr) : base(ptr) { }

	void Start()
	{
		Import();
	}

	public void Import()
	{
		var path = Path.Combine(Path.Combine(Application.streamingAssetsPath, "terrainfixes"), this.gameObject.name + ".bin");
		if(File.Exists(path))
		{
			FileStream fs = new FileStream(path, FileMode.Open, FileAccess.Read);
			var binaryReader = new BinaryReader(fs);

			try
			{
				var mf = this.GetComponent<MeshFilter>();
				var verticies = mf.sharedMesh.vertices;

				int i = 0;
				while (fs.Position < fs.Length)
				{
					var x = binaryReader.ReadSingle();
					var y = binaryReader.ReadSingle();
					var z = binaryReader.ReadSingle();
					verticies[i] = new Vector3(x, y, z);
					i++;
				}
				mf.sharedMesh.vertices = verticies;


				var collider = this.GetComponent<MeshCollider>();
				if(collider != null)
				{
					collider.sharedMesh = mf.sharedMesh;
				}
			}
			catch(Exception e)
			{
				SuisHackMain.loggerInst.Error($"Error when loading {path}: {e}");
			}
			finally
			{
				binaryReader.Close();
				fs.Close();
			}
		}
	}
}
```

And there we go:
[![terrain_verts_fixed.png](/images/articles/deadlypremonition2/terrain_verts_fixed.png)](/images/articles/deadlypremonition2/terrain_verts_fixed.png)

For the first blog entry about improving and fixing Deadly Premonition 2 see: [Improving Deadly Premonition 2 using HarmonyX and MelonLoader]({% post_url 2022-07-07-Improving_Deadly_Premonition_2_using_HarmonyX_and_MelonLoader %}).


Source code as always available on [Github](https://github.com/SuiMachine/Deadly-Premonition-2---Sui-s-hack).