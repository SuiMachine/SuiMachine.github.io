---
layout: page
title: VTube Studio VRAM calcuator
menu: none
permalink: web_projects/vts_calculator/
---
The intention of this calculator is to help people realize how much VRAM their VTube Studio models are using. All you need is to provide a size of your texture atlas and the amount of texture atlasses used. 

Texture size: <select name="texture_size" id="texture_size" onchange="calculateVRAMUSage();">
    <option value="256">256</option>
    <option value="512">512</option>
    <option value="1024">1024</option>
    <option value="2048">2048</option>
    <option value="4096" selected=true>4096</option>
    <option value="8192">8192</option>
    <option value="16384">16384</option>
</select>

Texture count <input type="number" id="texture_count" min="1" max="128" value="6" onchange="calculateVRAMUSage();" autofocus />

<button onclick="calculateVRAMUSage()">Calculate</button>

<p id="results"></p>
<script>
function calculateVRAMUSage() {
	var textureResolution = parseFloat(document.getElementById("texture_size").value);
	var sum = 4;
	
	do  {
		sum += textureResolution * textureResolution  * 4;
		textureResolution = textureResolution / 2;
	} while (textureResolution != 1);
	
	sum *= parseFloat(document.getElementById("texture_count").value);
	var result = "";
	
	if (sum > 536870912)
		result += "<u>" + Math.round(sum * 100 / 1073741824) / 100 + " GB</u> or <u>" + Math.round((sum / 4) * 100 / 1073741824) / 100 + " GB </u> when using BC7.";
	else
		result += "<u>" + Math.round(sum * 100 / 1048576) / 100 + " MB</u> or <u>" + Math.round((sum / 4) * 100 / 1048576) / 100 + " MB </u> when using BC7.";
	
	document.getElementById("results").innerHTML = "<b>Your VRAM usage is:</b> " + result;
}

calculateVRAMUSage();
</script>
