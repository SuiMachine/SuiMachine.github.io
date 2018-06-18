---
layout: page
title: Vision Engine FOV calculator
menu: none
permalink: /web_projects/vision_fov_calculator/
---
<link rel="stylesheet" href="{{ base }}/css/game-grid.css">
<h3>Tested games (click to load default FOV for a game):</h3>

<div class="gametable-container">
{% include game_button.html name="Emergency 3" url="emergency3" developer="Sixteen Tons Entertainment" year="2005" color="#7e373e" %}
{% include game_button.html name="Emergency 4" url="emergency4" developer="Sixteen Tons Entertainment" year="2006" color="#6d1c00" %}
{% include game_button.html name="Psychotoxic" url="psychotoxic" developer="Nuclear Vision" year="2005" color="#A00000" %}
</div><br>

Desired Horizontal FOV (for 4:3 aspect ratio): <input type="number" id="desiredFOV" min="10" max="351" value="90" autofocus /> <button onclick="calculateFOV()">Calculate</button>

<p id="results"></p>
<script>
function calculateFOV() {
	var f_desiredFOV = parseFloat(document.getElementById("desiredFOV").value);
	var f_Result = 160 / Math.tan(f_desiredFOV/ 114.59155);
    document.getElementById("results").innerHTML = "<b>Your FOV config value is:</b> <u>" + f_Result + "</u>";
}

function getValuesFromLocationHash() {
    var hsh = location.hash;
	
	if(hsh != "")
	{
		hsh = hsh.toLowerCase().substring(1);	
		loadDefaultValue(hsh);
	}
}

function loadDefaultValue(game)
{
	var tField = document.getElementById("desiredFOV");

	switch(game)
	{
		case "emergency3":
			tField.value = 33;
			break;
		case "emergency4":
			tField.value = 49;
			break;
		case "psychotoxic":
			tField.value = 75;
			break;
		default:
			tField.value = 90;
			break;
	}
	
	calculateFOV();
}

getValuesFromLocationHash();
calculateFOV();
</script>

Formula:
<pre>ConfigsFOV = 160 / tan(HorizontalFOV / 114.59155)</pre>

<br/>
<h3>Known games (unsupported):</h3>
* Lula 3D
* Demonicon
* Emergency 3
* Emergency 2012
* Gotcha!
* Back to Gaya

<h3>Known games that don't need calculator:</h3>
* Arcania: Gothic 4

<h3>Known games with FOV slider:</h3>
* Dark