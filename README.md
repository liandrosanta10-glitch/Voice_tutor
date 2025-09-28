// server.js
import express from "express";
import fetch from "node-fetch";
import dotenv from "dotenv";

dotenv.config();
const app = express();

app.get("/session", async (req, res) => {
  try {
    const r = await fetch("https://api.openai.com/v1/realtime/sessions", {
      method: "POST",
      headers: {
        "Authorization": `Bearer ${process.env.OPENAI_API_KEY}`,
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        model: "gpt-4o-realtime-preview",
        voice: "verse"
      })
    });
    const data = await r.json();
    res.json(data);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: "session failed" });
  }
});

app.listen(3000, () => console.log("Server läuft auf http://localhost:3000"));

<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>PrepMyMind Voice Tutor</title>
</head>
<body>
  <h1>🎤 Dein Sprach-Tutor</h1>
  <button id="startBtn">Start</button>
  <audio id="aiAudio" autoplay></audio>

<script>
async function startVoice() {
  const tokenResp = await fetch("/session");
  const tokenJson = await tokenResp.json();
  const EPHEMERAL_KEY = tokenJson.client_secret?.value || tokenJson.client_secret;

  const pc = new RTCPeerConnection();
  const audioEl = document.getElementById("aiAudio");
  pc.ontrack = (event) => { audioEl.srcObject = event.streams[0]; };

  const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
  stream.getTracks().forEach(t => pc.addTrack(t, stream));

  const offer = await pc.createOffer();
  await pc.setLocalDescription(offer);

  const model = "gpt-4o-realtime-preview";
  const sdpResp = await fetch(`https://api.openai.com/v1/realtime?model=${model}`, {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${EPHEMERAL_KEY}`,
      "Content-Type": "application/sdp"
    },
    body: offer.sdp
  });
  const answer = await sdpResp.text();
  await pc.setRemoteDescription({ type: "answer", sdp: answer });
}

document.getElementById("startBtn").addEventListener("click", startVoice);
</script>
</body>
</html>
{
  "name": "voice-tutor",
  "version": "1.0.0",
  "main": "server.js",
  "type": "module",
  "dependencies": {
    "dotenv": "^16.0.0",
    "express": "^4.18.0",
    "node-fetch": "^3.3.0"
  }
}
