<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8" />
  <title>Ultra Wide + Live Photo style</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
    }

    body {
      background: #05060a;
      color: #fff;
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 16px;
    }

    #app {
      width: 100%;
      max-width: 420px;
      background: radial-gradient(circle at top, #151827, #05060a);
      border-radius: 24px;
      padding: 16px;
      box-shadow: 0 20px 60px rgba(0, 0, 0, 0.8);
    }

    h1 {
      font-size: 18px;
      margin-bottom: 8px;
    }

    #subtitle {
      font-size: 12px;
      opacity: 0.8;
      margin-bottom: 12px;
    }

    #controls {
      display: flex;
      flex-direction: column;
      gap: 8px;
      margin-bottom: 12px;
      font-size: 12px;
    }

    select, button {
      border-radius: 999px;
      border: none;
      padding: 6px 10px;
      font-size: 12px;
      outline: none;
    }

    select {
      background: #111522;
      color: #fff;
    }

    button {
      cursor: pointer;
      background: linear-gradient(90deg, #00e5ff, #00ff95);
      color: #000;
      font-weight: 600;
    }

    button:disabled {
      opacity: 0.4;
      cursor: default;
    }

    #video-wrapper {
      position: relative;
      border-radius: 18px;
      overflow: hidden;
      background: #000;
      margin-bottom: 10px;
    }

    video {
      width: 100%;
      display: block;
      background: #000;
    }

    #live-indicator {
      position: absolute;
      top: 8px;
      left: 8px;
      padding: 2px 8px;
      border-radius: 999px;
      background: rgba(0, 0, 0, 0.6);
      border: 1px solid rgba(255, 255, 255, 0.4);
      font-size: 10px;
      display: flex;
      align-items: center;
      gap: 4px;
    }

    #live-dot {
      width: 8px;
      height: 8px;
      border-radius: 50%;
      background: #ff1744;
      box-shadow: 0 0 6px rgba(255, 23, 68, 0.8);
    }

    #bottom-bar {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-top: 6px;
    }

    #status {
      font-size: 11px;
      opacity: 0.8;
    }

    #capture-btn {
      width: 52px;
      height: 52px;
      border-radius: 50%;
      border: 3px solid #fff;
      background: radial-gradient(circle at center, #ffea00, #ff1744);
      box-shadow: 0 0 12px rgba(255, 23, 68, 0.8);
    }

    #thumb-wrapper {
      width: 52px;
      height: 52px;
      border-radius: 16px;
      overflow: hidden;
      border: 1px solid rgba(255, 255, 255, 0.3);
      background: #000;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 10px;
      opacity: 0.7;
      cursor: pointer;
    }

    #thumb-video {
      width: 100%;
      height: 100%;
      object-fit: cover;
      display: none;
    }

    #thumb-label {
      padding: 4px;
    }

    /* Modal per riprodurre la live photo */
    #live-modal {
      position: fixed;
      inset: 0;
      background: rgba(0, 0, 0, 0.85);
      display: none;
      align-items: center;
      justify-content: center;
      z-index: 10;
    }

    #live-modal-inner {
      width: 90%;
      max-width: 420px;
      border-radius: 20px;
      overflow: hidden;
      background: #000;
      position: relative;
    }

    #live-modal video {
      width: 100%;
      display: block;
    }

    #live-modal-close {
      position: absolute;
      top: 8px;
      right: 8px;
      width: 26px;
      height: 26px;
      border-radius: 50%;
      border: none;
      background: rgba(0, 0, 0, 0.6);
      color: #fff;
      font-size: 14px;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div id="app">
    <h1>Camera Ultra Wide (se disponibile)</h1>
    <div id="subtitle">
      Seleziona la camera posteriore che sembra ultra‑grandangolare.<br />
      Le “Live Photo” sono mini video.
    </div>

    <div id="controls">
      <div>
        <label for="deviceSelect">Camera:</label>
        <select id="deviceSelect"></select>
      </div>
      <button id="startBtn">Avvia camera</button>
    </div>

    <div id="video-wrapper">
      <video id="preview" autoplay playsinline></video>
      <div id="live-indicator">
        <div id="live-dot"></div>
        LIVE
      </div>
    </div>

    <div id="bottom-bar">
      <div id="status">In attesa della camera…</div>
      <button id="capture-btn" disabled></button>
      <div id="thumb-wrapper">
        <video id="thumb-video" muted loop></video>
        <div id="thumb-label">Live</div>
      </div>
    </div>
  </div>

  <!-- Modal Live Photo -->
  <div id="live-modal">
    <div id="live-modal-inner">
      <video id="live-playback" controls></video>
      <button id="live-modal-close">✕</button>
    </div>
  </div>

  <script>
    const deviceSelect = document.getElementById("deviceSelect");
    const startBtn = document.getElementById("startBtn");
    const preview = document.getElementById("preview");
    const statusEl = document.getElementById("status");
    const captureBtn = document.getElementById("capture-btn");
    const thumbWrapper = document.getElementById("thumb-wrapper");
    const thumbVideo = document.getElementById("thumb-video");
    const thumbLabel = document.getElementById("thumb-label");
    const liveModal = document.getElementById("live-modal");
    const livePlayback = document.getElementById("live-playback");
    const liveModalClose = document.getElementById("live-modal-close");

    let currentStream = null;
    let mediaRecorder = null;
    let recordedChunks = [];

    async function listDevices() {
      try {
        const devices = await navigator.mediaDevices.enumerateDevices();
        const videoDevices = devices.filter(d => d.kind === "videoinput");

        deviceSelect.innerHTML = "";
        videoDevices.forEach((d, i) => {
          const option = document.createElement("option");
          option.value = d.deviceId;
          option.textContent = d.label || `Camera ${i + 1}`;
          deviceSelect.appendChild(option);
        });

        if (videoDevices.length === 0) {
          statusEl.textContent = "Nessuna camera trovata.";
        } else {
          statusEl.textContent = "Seleziona una camera e premi Avvia.";
        }
      } catch (err) {
        console.error(err);
        statusEl.textContent = "Errore nel recupero dei dispositivi.";
      }
    }

    async function startCamera() {
      if (currentStream) {
        currentStream.getTracks().forEach(t => t.stop());
        currentStream = null;
      }

      const deviceId = deviceSelect.value;
      const constraints = {
        video: {
          deviceId: deviceId ? { exact: deviceId } : undefined,
          facingMode: "environment",
          width: { ideal: 1920 },
          height: { ideal: 1080 }
        },
        audio: false
      };

      try {
        const stream = await navigator.mediaDevices.getUserMedia(constraints);
        currentStream = stream;
        preview.srcObject = stream;
        statusEl.textContent = "Camera attiva. Premi il pulsante per una Live Photo.";
        captureBtn.disabled = false;
      } catch (err) {
        console.error(err);
        statusEl.textContent = "Impossibile accedere alla camera.";
        captureBtn.disabled = true;
      }
    }

    async function init() {
      if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
        statusEl.textContent = "getUserMedia non supportato in questo browser.";
        startBtn.disabled = true;
        return;
      }

      // Prima richiesta “generica” per sbloccare le label dei device
      try {
        await navigator.mediaDevices.getUserMedia({ video: true, audio: false });
      } catch (e) {
        // anche se l’utente rifiuta, proviamo comunque a elencare i device
      }

      await listDevices();
    }

    function captureLivePhoto() {
      if (!currentStream) return;

      recordedChunks = [];
      mediaRecorder = new MediaRecorder(currentStream, {
        mimeType: "video/webm;codecs=vp9"
      });

      mediaRecorder.ondataavailable = (e) => {
        if (e.data.size > 0) {
          recordedChunks.push(e.data);
        }
      };

      mediaRecorder.onstop = () => {
        const blob = new Blob(recordedChunks, { type: "video/webm" });
        const url = URL.createObjectURL(blob);

        thumbVideo.src = url;
        thumbVideo.style.display = "block";
        thumbLabel.style.display = "none";
        thumbVideo.play();

        livePlayback.src = url;
        livePlayback.currentTime = 0;
      };

      // registra ~1.5 secondi
      mediaRecorder.start();
      statusEl.textContent = "Registrazione Live Photo…";
      captureBtn.disabled = true;

      setTimeout(() => {
        mediaRecorder.stop();
        statusEl.textContent = "Live Photo salvata. Tocca la miniatura per vederla.";
        captureBtn.disabled = false;
      }, 1500);
    }

    // Eventi
    startBtn.addEventListener("click", startCamera);
    captureBtn.addEventListener("click", captureLivePhoto);

    thumbWrapper.addEventListener("click", () => {
      if (!thumbVideo.src) return;
      liveModal.style.display = "flex";
      livePlayback.play();
    });

    liveModalClose.addEventListener("click", () => {
      liveModal.style.display = "none";
      livePlayback.pause();
    });

    liveModal.addEventListener("click", (e) => {
      if (e.target === liveModal) {
        liveModal.style.display = "none";
        livePlayback.pause();
      }
    });

    init();
  </script>
</body>
</html>
