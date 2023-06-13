---
title: Doodle Test
description: Doooooooooodle
slug: doodle-test
readingTime: false
toc: false
---

{{< raw_html >}}
<!DOCTYPE html>
<p id="test"></p>
<canvas id="canvas"></canvas>
<script>
  let test = document.getElementById("test");
  let canvas = document.getElementById("canvas");
  const parent = canvas.parentElement;
  const ctx = canvas.getContext("2d");
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  const head = canvas.parentElement.parentElement.parentElement.parentElement;

  let resize = function() {
    canvas.height = canvas.offsetWidth;
    test.innerHTML = canvas.offsetWidth;
  };
  window.onresize = resize;
  resize();
 </script>
{{< /raw_html >}}
