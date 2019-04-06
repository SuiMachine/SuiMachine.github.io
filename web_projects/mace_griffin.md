---
layout: page
title: Mace Griffin Bounty Hunter FOV calculator
menu: none
permalink: web_projects/mace_griffin_fov_calculator/
---
<link rel="stylesheet" href="{{ base }}/css/game-grid.css">
<p>Desired Horizontal FOV: <input type="number" id="desiredFOV" min="10" max="351" value="90" autofocus /> <button onclick="calculateFOV()">Calculate</button></p>
<p>Screen Width / Height: <input type="number" id="screenX" value="640" /> / <input type="number" id="screenY" value="480" />


<p id="results"></p>

Vars:
<p id="log"></p>

<script>
function calculateFOV() {
	var f_desiredFOV = parseFloat(document.getElementById("desiredFOV").value);
	var f_FovY = Math.PI * f_desiredFOV / 180.0;
	var f_aspectRatio = parseFloat(document.getElementById("screenX").value) / parseFloat(document.getElementById("screenY").value);

	var f_FovX = f_FovY / (f_aspectRatio / 1.3333333333);
	
	var f_ResFovY = f_FovY * 512;
	var f_ResFovX = f_FovX * 512;


    document.getElementById("results").innerHTML = "<b>Your FOV config value is:</b> <u>" + f_desiredFOV + "</u>";
	
	document.getElementById("log").innerHTML = "Aspect = " +  f_aspectRatio +
		"<br>FovX = " + f_FovX +
		"<br>FovY = " + f_FovY +
		"<br>ResX = " + f_ResFovX +
		"<br>ResY = " + f_ResFovY;
}

calculateFOV();
</script>
