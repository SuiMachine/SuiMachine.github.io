---
layout: page
title: Vision Engine FOV calculator (Emergency 4)
menu: none
permalink: /web_projects/vision_fov_calculator/emergency4
---


Desired Horizontal FOV (for 4:3 aspect ratio): <input type="number" id="desiredFOV" min="10" max="351" value="90" autofocus />

Engine const multiplier: <input type="number" id="engineMultiplier" value="114.59155" style="width: 144px; text-align:center;" /> <button onclick="calculateFOV()">Calculate</button>

<p id="results"></p>
<script>
function calculateFOV() {
	var f_desiredFOV = parseFloat(document.getElementById("desiredFOV").value);
	var f_engineConst = parseFloat(document.getElementById("engineMultiplier").value);
	var f_Result = 160 / Math.tan(f_desiredFOV/ f_engineConst);
    document.getElementById("results").innerHTML = "<b>Your FOV config value is:</b> <u>" + f_Result + "</u>";
}

function getValuesFromLocationHash() {
    var x = location.hash;
	var splt = x.split("&");
	if(splt[0].startsWith("#"))
	{
		splt[0] = splt[0].substr(1,splt[0].lenght);
	}
	
	var i;
	for(i=0; i<splt.length; i++)
	{
	   if(splt[i].startsWith("DesiredFOV"))
	   {
		   var helper = splt[i].split("=");
		   document.getElementById("desiredFOV").value = helper[1];
	   }

	   if(splt[i].startsWith("EngineConst"))
	   {
		   var helper = splt[i].split("=");
		   document.getElementById("engineMultiplier").value = helper[1];
	   }
	}
}

getValuesFromLocationHash();
calculateFOV();
</script>

Formula:
<pre>ConfigsFOV = 160 / tan(HorizontalFOV / EngineConst)</pre>