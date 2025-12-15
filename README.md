<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Image + Audio to Video (Mobile Tool)</title>
  <style>
    body{font-family:Arial, sans-serif;background:#0f172a;color:#fff;padding:16px}
    .card{background:#111827;border-radius:12px;padding:16px;box-shadow:0 10px 30px rgba(0,0,0,.3)}
    h1{font-size:20px;margin-bottom:8px}
    input,button{width:100%;margin:8px 0;padding:10px;border-radius:8px;border:none}
    button{background:#22c55e;color:#062;font-weight:bold}
    canvas{display:none}
    .note{font-size:12px;color:#cbd5f5}
  </style>
</head>
<body>
  <div class="card">
    <h1>Image + Audio ➜ Video (No Watermark)</h1>
    <input type="file" id="image" accept="image/*" />
    <input type="file" id="audio" accept="audio/*" />
    <button onclick="makeVideo()">Create Video</button>
    <p class="note">Works on modern mobile browsers (Chrome). Keep screen ON during export.</p>
  </div>
  <canvas id="c"></canvas>
  <script>
    async function makeVideo(){
      const imgFile = document.getElementById('image').files[0];
      const audFile = document.getElementById('audio').files[0];
      if(!imgFile || !audFile){alert('Select image and audio');return;}const canvas = document.getElementById('c');
  const ctx = canvas.getContext('2d');
  const img = new Image();
  img.src = URL.createObjectURL(imgFile);
  await img.decode();
  canvas.width = img.width; canvas.height = img.height;

  const stream = canvas.captureStream(30);
  const audio = document.createElement('audio');
  audio.src = URL.createObjectURL(audFile);
  await audio.play(); audio.pause();

  const dest = new MediaStream();
  stream.getVideoTracks().forEach(t=>dest.addTrack(t));
  const ac = new AudioContext();
  const src = ac.createMediaElementSource(audio);
  const out = ac.createMediaStreamDestination();
  src.connect(out); src.connect(ac.destination);
  out.stream.getAudioTracks().forEach(t=>dest.addTrack(t));

  const rec = new MediaRecorder(dest,{mimeType:'video/webm'});
  const chunks=[]; rec.ondataavailable=e=>chunks.push(e.data);
  rec.start(); audio.play();
  const draw=()=>{ctx.drawImage(img,0,0,canvas.width,canvas.height); requestAnimationFrame(draw)}; draw();
  audio.onended=()=>{rec.stop()}
  rec.onstop=()=>{
    const blob=new Blob(chunks,{type:'video/webm'});
    const a=document.createElement('a'); a.href=URL.createObjectURL(blob); a.download='song-video.webm'; a.click();
  }
}

  </script>
</body>
</html>
