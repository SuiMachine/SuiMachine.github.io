---
layout: page
title: Mace Griffin Bounty Hunter FOV calculator
menu: none
permalink: web_projects/mace_griffin_fov_calculator/
---
<link rel="stylesheet" href="{{ base }}/css/game-grid.css">
<p>Desired Horizontal FOV: <input type="number" id="desiredFOV" min="10" max="351" value="64.01579624" autofocus /> <button onclick="calculateFOV()">Calculate</button></p>
<p>Screen Width x Height: <input type="number" id="screenX" value="1024" style="width: 65px"/> x <input type="number" id="screenY" value="768" style="width: 65px"/>


<p id="results"></p>

<p id="log"></p>

<script>
function calculateFOV() {
	var f_desiredFOV = parseFloat(document.getElementById("desiredFOV").value);
	var f_ScreenWidth = parseFloat(document.getElementById("screenX").value);
	var f_ScreenHeight = parseFloat(document.getElementById("screenY").value);
	
	//Horizontal FOV in Radian
	var f_FovXInitial =  f_desiredFOV * Math.PI / 180;
	
	//Calculate vertical FOV and then calculate horizontal FOV, extending it from 4:3 (hor+ FOV scaling)
	var f_FovRadY = 2 * Math.atan(Math.tan(f_FovXInitial / 2) * (3.0 / 4.0));
	var f_aspectRatio = f_ScreenWidth / f_ScreenHeight;
	var f_FovRadX = 2 * Math.atan(Math.tan(f_FovRadY / 2) * f_aspectRatio);
	
	//Calculate values used in View Matrix (yes it could be done in previous step, but it's easier to debug like this)
	var f_ViewMatrixFovX = Math.tan(f_FovRadX / 2);
	var f_ViewMatrixFovY = Math.tan(f_FovRadY / 2);
	
	//I still have no idea what these values are, that's how they are converted by the game
	var f_ConfFovX = f_ScreenWidth / (f_ViewMatrixFovX * 2);
	var f_ConfFovY = f_ScreenHeight / (f_ViewMatrixFovY * 2);
	
	//Calculate config FOV
	var f_ResFovX = f_ConfFovX / ( (f_ScreenWidth-0.1)*0.0015625);
	var f_ResFovY = f_ConfFovY / ( (f_ScreenHeight-0.1)*0.0020833334);


    document.getElementById("results").innerHTML = "<b>Your FOV config values are:</b><br/><pre>FOV_X = " + Math.round(f_ResFovX) + "<br/>FOV_Y = " + Math.round(f_ResFovY) + "</pre>";
	
	/*
	document.getElementById("log").innerHTML = "" +
		"<br>Aspect = " +  f_aspectRatio +
		"<br>f_FovRadX = " + f_FovRadX +
		"<br>f_FovRadY = " + f_FovRadY +
		"<br>f_ViewMatrixFovX = " + f_ViewMatrixFovX +
		"<br>f_ViewMatrixFovY = " + f_ViewMatrixFovY +
		"<br>f_ConfFovX = " + f_ConfFovX +
		"<br>f_ConfFovY = " + f_ConfFovY +
		"<br>f_ResFovX = " + f_ResFovX +
		"<br>f_ResFovY = " + f_ResFovY;*/
}

calculateFOV();
</script>


