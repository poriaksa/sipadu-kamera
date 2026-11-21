<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Kamera Selfie SIPADU</title>

<style>
body {
  font-family: Arial, sans-serif;
  background: #f1f5f9;
  text-align: center;
  padding: 20px;
}

.box {
  max-width: 500px;
  margin: auto;
  background: white;
  padding: 20px;
  border-radius: 15px;
}

video, canvas, img {
  width: 100%;
  border-radius: 12px;
  margin-top: 12px;
}

button {
  width: 100%;
  padding: 14px;
  margin-top: 12px;
  border: none;
  border-radius: 8px;
  background: #1a73e8;
  color: white;
  font-size: 16px;
}

#btnGunakan {
  background: #188038;
  display: none;
}

#pesan {
  margin-top: 15px;
}
</style>
</head>

<body>

<div class="box">

<h2>KAMERA SELFIE SIPADU</h2>

<p>Silakan ambil foto untuk presensi.</p>

<video id="video" autoplay playsinline muted></video>

<canvas id="canvas" style="display:none"></canvas>

<img id="preview" style="display:none">

<button onclick="bukaKamera()">AKTIFKAN KAMERA</button>

<button onclick="ambilFoto()">AMBIL SELFIE</button>

<button id="btnGunakan" onclick="kirimFoto()">
GUNAKAN FOTO INI
</button>

<button onclick="ulangFoto()">ULANGI FOTO</button>

<p id="pesan">Silakan aktifkan kamera.</p>

</div>

<script>

let stream = null;
let fotoSelfieDataUrl = "";

async function bukaKamera() {

  try {

    stream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode: "user" },
      audio: false
    });

    const video = document.getElementById("video");

    video.srcObject = stream;
    video.style.display = "block";

    await video.play();

    document.getElementById("pesan").innerText =
      "Kamera berhasil diaktifkan.";

  } catch (error) {

    document.getElementById("pesan").innerText =
      "Kamera gagal dibuka: " + error.name;

  }
}

function ambilFoto() {

  const video = document.getElementById("video");

  if (!stream || !video.videoWidth) {
    document.getElementById("pesan").innerText =
      "Aktifkan kamera terlebih dahulu.";
    return;
  }

  const canvas = document.getElementById("canvas");

  let lebar = video.videoWidth;
  let tinggi = video.videoHeight;

  if (lebar > 800) {
    tinggi = tinggi * 800 / lebar;
    lebar = 800;
  }

  canvas.width = lebar;
  canvas.height = tinggi;

  canvas.getContext("2d").drawImage(
    video, 0, 0, lebar, tinggi
  );

  fotoSelfieDataUrl = canvas.toDataURL(
    "image/jpeg", 0.75
  );

  document.getElementById("preview").src =
    fotoSelfieDataUrl;

  document.getElementById("preview").style.display =
    "block";

  video.style.display = "none";

  hentikanKamera();

  document.getElementById("btnGunakan").style.display =
    "block";

  document.getElementById("pesan").innerText =
    "Periksa foto. Jika sesuai, tekan GUNAKAN FOTO INI.";
}

function kirimFoto() {

  if (!fotoSelfieDataUrl) {
    alert("Foto belum tersedia.");
    return;
  }

  if (!window.opener || window.opener.closed) {

    document.getElementById("pesan").innerText =
      "Halaman SIPADU tidak ditemukan. Buka kamera dari SIPADU.";

    return;
  }

  window.opener.postMessage({

    type: "SIPADU_SELFIE",
    foto: fotoSelfieDataUrl

  }, "*");

  document.getElementById("pesan").innerText =
    "Foto dikirim ke SIPADU.";

  setTimeout(function() {
    window.close();
  }, 700);
}

function ulangFoto() {

  fotoSelfieDataUrl = "";

  document.getElementById("preview").style.display =
    "none";

  document.getElementById("btnGunakan").style.display =
    "none";

  bukaKamera();
}

function hentikanKamera() {

  if (stream) {

    stream.getTracks().forEach(function(track) {
      track.stop();
    });

    stream = null;
  }
}

</script>

</body>
</html>
