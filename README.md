<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Plate & SIM Detection — Flask + OCR + YOLO</title>
  <link href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.3.2/css/bootstrap.min.css" rel="stylesheet">
  <style>
    :root{
      --accent:#0d6efd;
      --muted:#6c757d;
      --card-bg: rgba(255,255,255,0.95);
    }
    body{background: linear-gradient(180deg,#f8fafc 0%, #eef2f7 100%);font-family: Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial}
    .hero{padding:3.5rem 0}
    .feature-icon{font-size:1.9rem}
    .uploader{border:2px dashed #e9ecef;border-radius:12px;padding:1.25rem;background:#fff}
    .webcam-card{height:320px;background:#000;border-radius:8px;display:flex;align-items:center;justify-content:center;color:#fff}
    pre.snippet{background:#0b1220;color:#e6eef8;padding:1rem;border-radius:8px;overflow:auto}
    footer{padding:2rem 0;color:var(--muted)}
  </style>
</head>
<body>
  <nav class="navbar navbar-expand-lg navbar-light bg-white shadow-sm">
    <div class="container">
      <a class="navbar-brand fw-bold" href="#">Plate & SIM Detection</a>
      <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#nav" aria-controls="nav" aria-expanded="false">
        <span class="navbar-toggler-icon"></span>
      </button>
      <div class="collapse navbar-collapse" id="nav">
        <ul class="navbar-nav ms-auto">
          <li class="nav-item"><a class="nav-link" href="#features">Features</a></li>
          <li class="nav-item"><a class="nav-link" href="#demo">Demo</a></li>
          <li class="nav-item"><a class="nav-link" href="#tech">Tech</a></li>
        </ul>
      </div>
    </div>
  </nav>

  <main class="container">
    <section class="hero text-center mt-5">
      <h1 class="display-6">Plate &amp; SIM Detection</h1>
      <p class="lead text-muted">Flask + YOLO + EasyOCR — detect vehicle license plates and extract phone numbers from SIM cards. Works with uploads and live camera streams.</p>
      <div class="d-flex justify-content-center gap-2 mt-3">
        <a href="#demo" class="btn btn-primary">Try Demo (UI preview)</a>
        <a href="#tech" class="btn btn-outline-secondary">View Tech Stack</a>
      </div>
    </section>

    <section id="features" class="row g-4 align-items-stretch">
      <div class="col-md-6 col-lg-4">
        <div class="card h-100 shadow-sm p-3">
          <div class="card-body">
            <h5 class="card-title">🔍 Plate Detection</h5>
            <p class="card-text text-muted">Upload car photos or use your webcam to detect license plates. Bounding boxes, confidence scores and recognized plate strings are shown.</p>
          </div>
        </div>
      </div>
      <div class="col-md-6 col-lg-4">
        <div class="card h-100 shadow-sm p-3">
          <div class="card-body">
            <h5 class="card-title">📱 SIM OCR</h5>
            <p class="card-text text-muted">Drop an image of a SIM card to extract phone numbers using EasyOCR and light preprocessing (deskew, contrast).</p>
          </div>
        </div>
      </div>
      <div class="col-md-6 col-lg-4">
        <div class="card h-100 shadow-sm p-3">
          <div class="card-body">
            <h5 class="card-title">🎥 Live Camera</h5>
            <p class="card-text text-muted">Real-time detection using webcam stream. Frame-by-frame YOLO inference with optional on-device cropping for OCR.</p>
          </div>
        </div>
      </div>
    </section>

    <hr class="my-5">

    <section id="demo" class="row gy-4">
      <div class="col-lg-6">
        <h3 class="h5">Image Uploads</h3>
        <div class="uploader mt-3">
          <form id="uploadForm" enctype="multipart/form-data">
            <div class="mb-3">
              <label class="form-label">Choose type</label>
              <select class="form-select" id="typeSelect">
                <option value="plate">Car (License Plate)</option>
                <option value="sim">SIM Card (Phone Number)</option>
              </select>
            </div>
            <div class="mb-3">
              <input class="form-control" type="file" id="fileInput" accept="image/*">
            </div>
            <div class="d-flex gap-2">
              <button type="button" class="btn btn-primary" id="uploadBtn">Upload & Detect</button>
              <button type="button" class="btn btn-outline-secondary" id="clearBtn">Clear</button>
            </div>
          </form>
        </div>

        <div class="mt-3">
          <h6>Annotated result</h6>
          <div class="card mt-2 shadow-sm p-2">
            <img id="resultImg" src="" alt="Result" style="max-width:100%;border-radius:6px;display:none;" />
            <div id="resultMeta" class="p-2"><small class="text-muted">No result yet</small></div>
            <div class="p-2"><button id="downloadBtn" class="btn btn-sm btn-success" style="display:none">Export Annotated Image</button></div>
          </div>
        </div>
      </div>

      <div class="col-lg-6">
        <h3 class="h5">Live Camera</h3>
        <div class="card mt-3 p-3">
          <div class="webcam-card" id="webcamBox">Camera preview</div>
          <div class="mt-3 d-flex gap-2">
            <button id="startCam" class="btn btn-primary">Start Camera</button>
            <button id="stopCam" class="btn btn-outline-secondary">Stop</button>
            <button id="captureFrame" class="btn btn-success">Capture & Detect</button>
          </div>
          <small class="text-muted mt-2 d-block">Tip: For best results, hold the SIM or license plate steady and well-lit.</small>
        </div>
      </div>
    </section>

    <hr class="my-5">

    <section id="tech" class="row gy-3">
      <div class="col-md-8">
        <h4 class="h6">🛠️ Technologies</h4>
        <ul>
          <li>Python 3.10+</li>
          <li>Flask — REST endpoints and streaming routes</li>
          <li>Ultralytics YOLOv8 — detection model for plates</li>
          <li>EasyOCR — text recognition (SIM numbers & plate OCR fallback)</li>
          <li>OpenCV — image preprocessing, annotation & streaming</li>
          <li>Bootstrap — quick UI</li>
        </ul>

        <h5 class="mt-3">Typical flow</h5>
        <ol>
          <li>Client uploads image or sends camera frames to `/detect`.</li>
          <li>Server runs YOLO to detect objects ("plate", "sim").</li>
          <li>If a plate is detected, crop and preprocess then pass to OCR.</li>
          <li>Return annotated image + JSON with detections and recognized text.</li>
        </ol>

        <h6 class="mt-3">Flask endpoints (example)</h6>
        <pre class="snippet"><code># POST /detect
# returns: { detections:[{label, conf, bbox}], ocr:[{text, conf}], annotated_image: base64 }
</code></pre>
      </div>

      <div class="col-md-4">
        <div class="card shadow-sm p-3">
          <h6 class="mb-2">Quick start</h6>
          <ol class="small">
            <li>Clone repo</li>
            <li>python -m venv .venv &amp; pip install -r requirements.txt</li>
            <li>Set model paths and run <code>flask run</code></li>
          </ol>
          <p class="small text-muted">Need a full backend example? I can generate Flask route templates, model loader code, and a ready-to-run `requirements.txt`.</p>
        </div>
      </div>
    </section>

    <footer class="mt-5 text-center">
      <small class="text-muted">Made with ❤️ — Plate &amp; SIM Detection UI • If you want, ask me to generate the Flask backend code or a zip with a runnable demo.</small>
    </footer>
  </main>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.3.2/js/bootstrap.bundle.min.js"></script>
  <script>
    // Minimal client-side preview behaviour (UI only)
    const fileInput = document.getElementById('fileInput');
    const resultImg = document.getElementById('resultImg');
    const resultMeta = document.getElementById('resultMeta');
    const downloadBtn = document.getElementById('downloadBtn');

    document.getElementById('uploadBtn').addEventListener('click', ()=>{
      const f = fileInput.files[0];
      if(!f){ alert('Please choose an image first'); return; }
      const url = URL.createObjectURL(f);
      resultImg.src = url; resultImg.style.display='block';
      resultMeta.innerHTML = '<strong>Preview:</strong> This is a client-side preview. Send file to server for full detection.';
      downloadBtn.style.display='inline-block';
    });

    document.getElementById('clearBtn').addEventListener('click', ()=>{
      fileInput.value=''; resultImg.style.display='none'; resultMeta.innerHTML='<small class="text-muted">No result yet</small>';
      downloadBtn.style.display='none';
    });

    // Simple webcam start/stop
    let stream=null; const webcamBox=document.getElementById('webcamBox');
    document.getElementById('startCam').addEventListener('click', async ()=>{
      if(stream) return;
      try{ stream = await navigator.mediaDevices.getUserMedia({video:{facingMode:'environment'}}); const v = document.createElement('video'); v.autoplay=true; v.playsInline=true; v.srcObject=stream; v.style.width='100%'; v.style.height='100%'; v.onloadedmetadata = ()=>v.play(); webcamBox.innerHTML=''; webcamBox.appendChild(v);
      }catch(e){ alert('Camera access denied or unavailable'); }
    });
    document.getElementById('stopCam').addEventListener('click', ()=>{
      if(!stream) return; stream.getTracks().forEach(t=>t.stop()); stream=null; webcamBox.innerHTML='Camera preview';
    });

    // No
