
<html lang="en">
<head>
<meta charset="UTF-8">
<title>SecureScan — Information Leak Detector</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    font-family: 'Segoe UI', Tahoma, sans-serif;
    background: radial-gradient(circle at top, #0a1929 0%, #000 100%);
    color: #e0e6ed;
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 20px;
  }
  .container {
    background: rgba(15, 25, 40, 0.95);
    border: 1px solid #1e3a5f;
    border-radius: 12px;
    padding: 40px;
    max-width: 600px;
    width: 100%;
    box-shadow: 0 0 40px rgba(0, 150, 255, 0.15);
  }
  .header {
    display: flex;
    align-items: center;
    gap: 15px;
    margin-bottom: 30px;
    padding-bottom: 20px;
    border-bottom: 1px solid #1e3a5f;
  }
  .shield {
    width: 50px;
    height: 50px;
    background: linear-gradient(135deg, #00d4ff, #0077cc);
    border-radius: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 28px;
  }
  h1 { font-size: 22px; font-weight: 600; }
  .subtitle { font-size: 13px; color: #7a94b8; margin-top: 2px; }

  .status-panel {
    background: #0a1420;
    border: 1px solid #1e3a5f;
    border-radius: 8px;
    padding: 25px;
    margin-bottom: 25px;
    min-height: 180px;
  }
  .status-line {
    font-family: 'Consolas', monospace;
    font-size: 13px;
    color: #7a94b8;
    margin: 6px 0;
    opacity: 0;
    animation: fadeIn 0.3s forwards;
  }
  .status-line.ok { color: #4ade80; }
  .status-line.warn { color: #fbbf24; }
  .status-line.alert { color: #ef4444; font-weight: 600; }
  @keyframes fadeIn { to { opacity: 1; } }

  .progress-bar {
    width: 100%;
    height: 6px;
    background: #0a1420;
    border-radius: 3px;
    overflow: hidden;
    margin: 20px 0;
    display: none;
  }
  .progress-fill {
    height: 100%;
    background: linear-gradient(90deg, #00d4ff, #0077cc);
    width: 0%;
    transition: width 0.1s linear;
  }

  button {
    width: 100%;
    padding: 15px;
    background: linear-gradient(135deg, #00d4ff, #0077cc);
    color: #000;
    border: none;
    border-radius: 8px;
    font-size: 16px;
    font-weight: 700;
    cursor: pointer;
    text-transform: uppercase;
    letter-spacing: 1px;
    transition: transform 0.1s, box-shadow 0.2s;
  }
  button:hover:not(:disabled) {
    box-shadow: 0 0 25px rgba(0, 212, 255, 0.5);
  }
  button:disabled { opacity: 0.5; cursor: not-allowed; }

  .result {
    display: none;
    margin-top: 20px;
    padding: 20px;
    background: rgba(239, 68, 68, 0.1);
    border: 1px solid #ef4444;
    border-radius: 8px;
    text-align: center;
  }
  .result h2 { color: #ef4444; font-size: 20px; margin-bottom: 10px; }
  .result p { color: #fca5a5; font-size: 14px; line-height: 1.6; }
  .pulse {
    display: inline-block;
    width: 10px;
    height: 10px;
    background: #ef4444;
    border-radius: 50%;
    margin-right: 8px;
    animation: pulse 1s infinite;
  }
  @keyframes pulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.3; }
  }
</style>
</head>
<body>
  <div class="container">
    <div class="header">
      <div class="shield">🛡</div>
      <div>
        <h1>SecureScan Pro</h1>
        <div class="subtitle">Information Leak Detection System v4.2.1</div>
      </div>
    </div>

    <div class="status-panel" id="statusPanel">
      <div class="status-line">System idle. Press SCAN to begin deep analysis.</div>
    </div>

    <div class="progress-bar" id="progressBar">
      <div class="progress-fill" id="progressFill"></div>
    </div>

    <button id="scanBtn" onclick="startScan()">Start Scan</button>

    <div class="result" id="result">
      <h2><span class="pulse"></span>LEAK DETECTED</h2>
      <p id="resultText"></p>
    </div>
  </div>

<script>
const steps = [
  { text: "> Initializing scan engine...", cls: "", delay: 400 },
  { text: "> Loading signature database [2,847,391 entries]...", cls: "", delay: 600 },
  { text: "> Checking system registry...", cls: "ok", delay: 700 },
  { text: "> Scanning network connections...", cls: "ok", delay: 800 },
  { text: "> Analyzing browser cookies and cache...", cls: "ok", delay: 700 },
  { text: "> Inspecting clipboard history...", cls: "warn", delay: 600 },
  { text: "> Monitoring outbound traffic on port 443...", cls: "warn", delay: 700 },
  { text: "> Deep-scanning memory segments...", cls: "warn", delay: 800 },
  { text: "> ⚠ Suspicious data pattern found at 0x7FFE2A4C...", cls: "alert", delay: 600 },
  { text: "> ⚠ Unauthorized credential exposure identified...", cls: "alert", delay: 500 },
  { text: "> ⚠ LEAK CONFIRMED. Compiling report...", cls: "alert", delay: 500 }
];

function startScan() {
  const btn = document.getElementById('scanBtn');
  const panel = document.getElementById('statusPanel');
  const bar = document.getElementById('progressBar');
  const fill = document.getElementById('progressFill');
  const result = document.getElementById('result');
  const resultText = document.getElementById('resultText');

  btn.disabled = true;
  btn.textContent = 'Scanning...';
  panel.innerHTML = '';
  result.style.display = 'none';
  bar.style.display = 'block';
  fill.style.width = '0%';

  let total = steps.reduce((a, s) => a + s.delay, 0);
  let elapsed = 0;

  steps.forEach((step, i) => {
    setTimeout(() => {
      const line = document.createElement('div');
      line.className = 'status-line ' + step.cls;
      line.textContent = step.text;
      panel.appendChild(line);
      panel.scrollTop = panel.scrollHeight;
    }, elapsed);
    elapsed += step.delay;
    setTimeout(() => {
      fill.style.width = ((elapsed / total) * 100) + '%';
    }, elapsed);
  });

  setTimeout(() => {
    const ip = `192.168.${Math.floor(Math.random()*255)}.${Math.floor(Math.random()*255)}`;
    const items = Math.floor(Math.random()*8) + 3;
    resultText.innerHTML = `
      <strong>${items} sensitive data points</strong> have been exposed from this device.<br><br>
      Compromised items include: saved passwords, session tokens, browser autofill records.<br>
      Local endpoint: <code>${ip}</code><br>
      Threat level: <strong style="color:#ef4444">CRITICAL</strong>
    `;
    result.style.display = 'block';
    btn.disabled = false;
    btn.textContent = 'Rescan';
  }, elapsed + 300);
}
</script>
</body>
</html>
