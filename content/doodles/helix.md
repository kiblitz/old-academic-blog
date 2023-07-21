---
title: Helix
description: Helix rotation animation
slug: helix
date: 2023-07-21 00:00:00+0000
readingTime: false
toc: false
---

{{<raw_html>}}
<style>
html, body {
  margin: 0;
  padding: 0;
  top: 0.5px;
  bottom: 0.5px;
  width: 100%;
  height: 100%;
}

canvas {
  width: 100%;
  height: 100%;
}
</style>
<html>
  <body onload="init();" onresize="updateDimensions();">
    <canvas id="canvas"></canvas>
  </body>
</html>

<script>
const LINKS = 10;
function init() {
  window.canvas = document.getElementById("canvas");
  window.ctx = window.canvas.getContext("2d");
  window.ctx.webkitImageSmoothingEnabled = true;

  window.color = 'aqua';
  const urlParams = new URLSearchParams(window.location.search);
  if (urlParams.has('color')) {
    window.color = urlParams.get('color');
  }
  updateDimensions();
}

function updateDimensions() {
  window.width = window.canvas.getBoundingClientRect().width;
  window.height = window.width / 2;
  window.canvas.width = window.width;
  window.canvas.height = window.height;
  window.helixHeight = window.height * 5 / 6;
  window.helixOffsetY = (window.height - window.helixHeight) / 2;
  window.helixWidth = window.helixHeight / 2;
  window.helixOffsetX = window.width / 2 - window.helixWidth / 2;
  window.ballSize = window.helixHeight / LINKS / 4;

  window.t = 0;
  redraw();
}

function redraw() {
  window.ctx.clearRect(0, 0, window.canvas.width, window.canvas.height);
  window.ctx.fillStyle = window.color;
  window.ctx.strokeStyle = 'white';

  for (let i = 0; i < LINKS; i++) {
    let delta = i * 3 / 5;
    let y = window.helixOffsetY + (i * 4 + 2) * window.ballSize;
    let x1 = window.width / 2 - Math.cos(t + delta) * window.helixWidth / 2;
    let x2 = window.width / 2 + Math.cos(t + delta) * window.helixWidth / 2;
    let size1 = window.ballSize - Math.sin(t) * window.ballSize / 3;
    let size2 = window.ballSize + Math.sin(t) * window.ballSize / 3;

    window.ctx.beginPath();
    window.ctx.moveTo(x1, y);
    window.ctx.lineTo(x2, y);
    window.ctx.stroke()
    drawBall(x1, y, size1); 
    drawBall(x2, y, size2); 
  }
}

function drawBall(x, y, size) {
  window.ctx.arc(x, y, size, 0, 2 * Math.PI);
  window.ctx.closePath();
  window.ctx.fill();
}

function update() {
  window.t+=0.02;
  redraw()
}

setInterval(update, 10);
</script>
{{</raw_html>}}
