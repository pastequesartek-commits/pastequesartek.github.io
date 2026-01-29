<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>Roboflow Camera IA</title>
  <style>
    body { margin: 0; background: black; }
    video, canvas {
      position: absolute;
      top: 0; left: 0;
      width: 100vw;
      height: 100vh;
    }
  </style>
</head>
<body>

<video id="video" autoplay playsinline></video>
<canvas id="canvas"></canvas>

<script>
const API_KEY = "TA_API_KEY";
const MODEL = "NOM_DU_PROJET";
const VERSION = "3";

const video = document.getElementById("video");
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

// Accès caméra
navigator.mediaDevices.getUserMedia({
  video: { facingMode: "environment" }
}).then(stream => {
  video.srcObject = stream;
});

// Resize canvas
video.addEventListener("loadedmetadata", () => {
  canvas.width = video.videoWidth;
  canvas.height = video.videoHeight;
  runInference();
});

async function runInference() {
  setInterval(async () => {
    ctx.drawImage(video, 0, 0, canvas.width, canvas.height);

    const blob = await new Promise(resolve =>
      canvas.toBlob(resolve, "image/jpeg")
    );

    const formData = new FormData();
    formData.append("file", blob);

    const res = await fetch(
      `https://detect.roboflow.com/${MODEL}/${VERSION}?api_key=${API_KEY}`,
      { method: "POST", body: formData }
    );

    const data = await res.json();

    drawBoxes(data.predictions);
  }, 800); // 1 image toutes les 0.8s
}

function drawBoxes(predictions) {
  ctx.drawImage(video, 0, 0, canvas.width, canvas.height);

  predictions.forEach(p => {
    ctx.strokeStyle = "lime";
    ctx.lineWidth = 3;
    ctx.strokeRect(
      p.x - p.width / 2,
      p.y - p.height / 2,
      p.width,
      p.height
    );

    ctx.fillStyle = "lime";
    ctx.font = "20px Arial";
    ctx.fillText(
      `${p.class} ${(p.confidence * 100).toFixed(1)}%`,
      p.x - p.width / 2,
      p.y - p.height / 2 - 5
    );
  });
}
</script>

</body>
</html>
