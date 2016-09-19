---
layout: page
title: Twitch Authorization
menu: none
permalink: /twitchauthy/
---

<p id="ac"></p>
<p id="scp"></p>

<script>
var acctok = "?";
var scopes = "?";

function getAccessToken() {
    var x = location.hash;
    return x;
}

var s_anchor = getAccessToken();
var splt = s_anchor.split("&");
if(splt[0].startsWith("#"))
{
    splt[0] = splt[0].substr(1,splt[0].lenght);
}

var i;
for(i=0; i<splt.length; i++)
{
   if(splt[i].startsWith("access_token"))
   {
       var helper = splt[i].split("=");
       acctok = helper[1];
   }

   if(splt[i].startsWith("scope"))
   {
       var helper = splt[i].split("=");
       scopes = helper[1];
   }
   
}

document.getElementById("ac").innerHTML = "<b>Access token is:</b> " + acctok;
document.getElementById("scp").innerHTML = "<b>Scopes are:</b> " + scopes;
</script>