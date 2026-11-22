<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Mô hình 3D: Giao điểm của 4 mặt cầu vệ tinh (GPS thực tế)</title>
  <style>
    :root {
      --bg-dark: #070d19;
      --panel-bg: #0f172a;
      --card-bg: #1e293b;
      --border: #334155;
      --text-main: #f8fafc;
      --text-muted: #94a3b8;
      --accent-blue: #38bdf8;
      --accent-gold: #facc15;
      --accent-green: #22c55e;
      --accent-red: #ef4444;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
    }

    body {
      background-color: var(--bg-dark);
      color: var(--text-main);
      height: 100vh;
      overflow: hidden;
      display: flex;
      flex-direction: column;
    }

    header {
      background: linear-gradient(90deg, #0b132b, #1c2541);
      border-bottom: 1px solid var(--border);
      padding: 10px 20px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      flex-shrink: 0;
    }

    header h1 {
      font-size: 1.05rem;
      font-weight: 700;
      color: var(--accent-blue);
      letter-spacing: 0.3px;
    }

    header .subtitle {
      font-size: 0.8rem;
      color: var(--text-muted);
    }

    .main-container {
      display: flex;
      flex: 1;
      height: calc(100vh - 54px);
      overflow: hidden;
    }

    /* Khung nhìn 3D bên trái */
    #canvas-wrapper {
      position: relative;
      flex: 1;
      height: 100%;
      background: radial-gradient(circle at center, #0f1c3a 0%, #050811 100%);
    }

    #three-canvas {
      width: 100%;
      height: 100%;
      display: block;
    }

    .hud-top-left {
      position: absolute;
      top: 14px;
      left: 14px;
      background: rgba(15, 23, 42, 0.85);
      backdrop-filter: blur(6px);
      border: 1px solid var(--border);
      border-radius: 10px;
      padding: 10px 14px;
      font-size: 0.82rem;
      max-width: 310px;
      pointer-events: none;
    }

    .hud-top-left .legend-item {
      display: flex;
      align-items: center;
      gap: 8px;
      margin-top: 5px;
    }

    .dot {
      width: 11px;
      height: 11px;
      border-radius: 50%;
      display: inline-block;
      flex-shrink: 0;
    }

    .hud-bottom-left {
      position: absolute;
      bottom: 14px;
      left: 14px;
      background: rgba(15, 23, 42, 0.82);
      border: 1px solid var(--border);
      border-radius: 8px;
      padding: 8px 12px;
      font-size: 0.78rem;
      color: var(--text-muted);
      display: flex;
      gap: 12px;
      align-items: center;
    }

    .btn-mini {
      background: var(--card-bg);
      color: var(--text-main);
      border: 1px solid var(--border);
      padding: 4px 10px;
      border-radius: 6px;
      cursor: pointer;
      font-size: 0.78rem;
      transition: 0.2s;
    }

    .btn-mini:hover {
      border-color: var(--accent-blue);
      color: var(--accent-blue);
    }

    /* Bảng điều khiển bên phải */
    .sidebar {
      width: 390px;
      background: var(--panel-bg);
      border-left: 1px solid var(--border);
      padding: 16px;
      overflow-y: auto;
      display: flex;
      flex-direction: column;
      gap: 14px;
    }

    .card {
      background: var(--card-bg);
      border: 1px solid var(--border);
      border-radius: 12px;
      padding: 14px;
    }

    .card-title {
      font-size: 0.85rem;
      font-weight: 700;
      text-transform: uppercase;
      color: var(--accent-blue);
      margin-bottom: 10px;
      letter-spacing: 0.5px;
    }

    /* Stepper 4 bước */
    .step-grid {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 8px;
    }

    .step-btn {
      background: #0f172a;
      color: var(--text-muted);
      border: 1px solid var(--border);
      border-radius: 8px;
      padding: 9px 10px;
      font-size: 0.82rem;
      font-weight: 600;
      cursor: pointer;
      text-align: left;
      transition: all 0.2s;
    }

    .step-btn:hover {
      border-color: var(--accent-blue);
      color: var(--text-main);
    }

    .step-btn.active {
      background: rgba(56, 189, 248, 0.16);
      border-color: var(--accent-blue);
      color: #ffffff;
      box-shadow: 0 0 12px rgba(56, 189, 248, 0.2);
    }

    .step-num {
      display: block;
      font-size: 0.72rem;
      color: var(--accent-blue);
      margin-bottom: 2px;
    }

    /* Giải thích kiến thức */
    .explanation-box {
      font-size: 0.88rem;
      line-height: 1.5;
      color: #e2e8f0;
    }

    .explanation-box strong {
      color: var(--accent-gold);
    }

    .formula-badge {
      display: inline-block;
      background: #0f172a;
      border: 1px solid var(--border);
      color: var(--accent-blue);
      padding: 3px 8px;
      border-radius: 6px;
      font-family: 'Consolas', monospace;
      font-size: 0.82rem;
      margin-top: 6px;
    }

    /* Thanh trượt sai lệch đồng hồ */
    .slider-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-size: 0.85rem;
      margin-bottom: 6px;
    }

    .dt-value {
      font-family: 'Consolas', monospace;
      font-weight: 700;
      padding: 2px 8px;
      border-radius: 6px;
      background: #0f172a;
    }

    input[type="range"] {
      width: 100%;
      accent-color: var(--accent-blue);
      cursor: pointer;
      margin: 6px 0;
    }

    .error-metrics {
      display: flex;
      justify-content: space-between;
      font-size: 0.8rem;
      color: var(--text-muted);
      margin-top: 4px;
    }

    .btn-sync {
      width: 100%;
      margin-top: 12px;
      padding: 11px;
      border: none;
      border-radius: 8px;
      font-size: 0.88rem;
      font-weight: 700;
      cursor: pointer;
      color: #052e16;
      background: linear-gradient(90deg, #4ade80, #22c55e);
      box-shadow: 0 4px 14px rgba(34, 197, 94, 0.35);
      transition: transform 0.15s, box-shadow 0.15s;
    }

    .btn-sync:hover {
      transform: translateY(-1px);
      box-shadow: 0 6px 18px rgba(34, 197, 94, 0.5);
    }

    /* Trạng thái khóa mục tiêu */
    .status-banner {
      padding: 11px 12px;
      border-radius: 8px;
      font-size: 0.84rem;
      line-height: 1.45;
      border: 1px solid;
    }

    .status-locked {
      background: rgba(34, 197, 94, 0.14);
      border-color: var(--accent-green);
      color: #bbf7d0;
    }

    .status-unlocked {
      background: rgba(239, 68, 68, 0.14);
      border-color: var(--accent-red);
      color: #fecaca;
    }

    .toggle-row {
      display: flex;
      align-items: center;
      justify-content: space-between;
      font-size: 0.82rem;
      color: var(--text-muted);
      margin-top: 8px;
      padding-top: 8px;
      border-top: 1px solid var(--border);
    }

    @media (max-width: 900px) {
      .main-container {
        flex-direction: column;
        overflow-y: auto;
        height: auto;
      }
      body {
        overflow-y: auto;
        height: auto;
      }
      #canvas-wrapper {
        height: 52vh;
        min-height: 340px;
      }
      .sidebar {
        width: 100%;
        border-left: none;
        border-top: 1px solid var(--border);
      }
    }
  </style>
</head>
<body>

<header>
  <div>
    <h1>🛰️ MÔ HÌNH 3D: GIAO ĐIỂM CỦA 4 MẶT CẦU VỆ TINH (GPS THỰC TẾ)</h1>
    <div class="subtitle">Chủ đề STEM Toán 9: “Mật mã GPS – Toán học đằng sau hệ thống định vị”</div>
  </div>
  <button class="btn-mini" onclick="resetCamera()">🔄 Đặt lại góc nhìn</button>
</header>

<div class="main-container">
  <!-- KHUNG NHÌN 3D -->
  <div id="canvas-wrapper">
    <canvas id="three-canvas"></canvas>

    <div class="hud-top-left">
      <div style="font-weight:700; color:#38bdf8; margin-bottom:4px;">CHÚ GIẢI KHÔNG GIAN 3D (Oxyz)</div>
      <div class="legend-item"><span class="dot" style="background:#ef4444;"></span> Vệ tinh 1 (Mặt cầu S₁)</div>
      <div class="legend-item"><span class="dot" style="background:#3b82f6;"></span> Vệ tinh 2 (Mặt cầu S₂)</div>
      <div class="legend-item"><span class="dot" style="background:#22c55e;"></span> Vệ tinh 3 (Mặt cầu S₃)</div>
      <div class="legend-item"><span class="dot" style="background:#f97316;"></span> Vệ tinh 4 (Hiệu chỉnh đồng hồ S₄)</div>
      <div class="legend-item"><span class="dot" style="background:#facc15; box-shadow:0 0 6px #facc15;"></span> P₁: Vị trí thật trên mặt đất</div>
      <div class="legend-item"><span class="dot" style="background:#a855f7;"></span> P₂: Giao điểm ảo ngoài vũ trụ</div>
    </div>

    <div class="hud-bottom-left">
      <span>🖱️ Kéo chuột / Chạm để xoay 360°</span>
      <span>•</span>
      <span>🔍 Cuộn / Chụm ngón tay để thu phóng</span>
    </div>
  </div>

  <!-- BẢNG ĐIỀU KHIỂN SƯ PHẠM -->
  <aside class="sidebar">
    <!-- BƯỚC 1-4 -->
    <div class="card">
      <div class="card-title">1. Tiến trình giao hội không gian 3D</div>
      <div class="step-grid">
        <button class="step-btn" id="btn-step-1" onclick="setStep(1)">
          <span class="step-num">BƯỚC 1</span>
          1 Vệ tinh (Mặt cầu)
        </button>
        <button class="step-btn" id="btn-step-2" onclick="setStep(2)">
          <span class="step-num">BƯỚC 2</span>
          2 Vệ tinh (Giao tuyến)
        </button>
        <button class="step-btn" id="btn-step-3" onclick="setStep(3)">
          <span class="step-num">BƯỚC 3</span>
          3 Vệ tinh (2 điểm)
        </button>
        <button class="step-btn active" id="btn-step-4" onclick="setStep(4)">
          <span class="step-num">BƯỚC 4</span>
          4 Vệ tinh (Chốt GPS)
        </button>
      </div>
    </div>

    <!-- GIẢI THÍCH TOÁN HỌC -->
    <div class="card">
      <div class="card-title">2. Phân tích hình học & Vật lí</div>
      <div class="explanation-box" id="explanation-text"></div>
    </div>

    <!-- MÔ PHỎNG SAI LỆCH ĐỒNG HỒ -->
    <div class="card" id="clock-card">
      <div class="card-title">3. Thử nghiệm sai lệch đồng hồ máy thu (Δt)</div>
      <div class="slider-header">
        <span>Độ lệch đồng hồ Δt:</span>
        <span class="dt-value" id="dt-display" style="color:#22c55e;">0,0 μs</span>
      </div>
      <input type="range" id="dt-slider" min="-10" max="10" step="0.5" value="0" oninput="onSliderChange(this.value)" />
      <div class="error-metrics">
        <span>Sai số bán kính Δd = c · Δt:</span>
        <strong id="dd-display" style="color:#facc15;">0 m</strong>
      </div>
      <div style="font-size:0.78rem; color:#94a3b8; margin-top:6px;">
        * Tốc độ sóng vô tuyến c ≈ 300 000 km/s ⇒ sai lệch <strong>1 μs</strong> (1 phần triệu giây) làm bán kính sai lệch <strong>300 m</strong>!
      </div>

      <button class="btn-sync" onclick="autoSyncClock()">
        ⚡ Tự động đồng bộ & Khóa mục tiêu (4 vệ tinh)
      </button>

      <div class="toggle-row">
        <label style="display:flex; align-items:center; gap:6px; cursor:pointer;">
          <input type="checkbox" id="toggle-wireframe" checked onchange="toggleWireframe(this.checked)" />
          Hiển thị lưới mặt cầu (dễ nhìn giao tuyến)
        </label>
      </div>
    </div>

    <!-- KẾT QUẢ CHỐT TỌA ĐỘ -->
    <div id="status-box" class="status-banner status-locked"></div>
  </aside>
</div>

<!-- THREE.JS & ORBIT CONTROLS -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>

<script>
  // ==========================================
  // 1. KHỞI TẠO SCENE, CAMERA, RENDERER
  // ==========================================
  const wrapper = document.getElementById('canvas-wrapper');
  const canvas = document.getElementById('three-canvas');

  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(45, wrapper.clientWidth / wrapper.clientHeight, 0.1, 1000);
  const defaultCamPos = new THREE.Vector3(7.2, 5.2, 8.2);
  camera.position.copy(defaultCam
