<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sensor Resolution Calculator</title>

<style>
body {
    font-family: Arial, sans-serif;
    background: #111;
    color: #f5f5f5;
    display: flex;
    justify-content: center;
    padding: 40px 0;
    margin: 0;
}

.container {
    background: #1c1c1c;
    padding: 30px;
    border-radius: 12px;
    width: 620px;
    box-shadow: 0 0 20px rgba(0,0,0,0.5);
}

h1 {
    text-align: center;
    margin-bottom: 20px;
}

label {
    display: block;
    margin-top: 15px;
    font-size: 14px;
}

input {
    width: 100%;
    padding: 8px;
    margin-top: 5px;
    border-radius: 6px;
    border: none;
    font-size: 14px;
}

.presets, .sensor-presets {
    display: flex;
    flex-wrap: wrap;
    margin-top: 10px;
}

.presets button,
.sensor-presets button {
    flex: 1;
    margin: 3px;
    padding: 6px;
    font-size: 12px;
    border-radius: 6px;
    border: none;
    background: #333;
    color: white;
    cursor: pointer;
}

.presets button:hover,
.sensor-presets button:hover {
    background: #555;
}

.result {
    margin-top: 20px;
    padding: 15px;
    background: #2b2b2b;
    border-radius: 8px;
    text-align: center;
    font-size: 16px;
}

/* Invisible normalization square */
.preview-container {
    margin-top: 30px;
    width: 400px;
    height: 400px;
    position: relative;
    margin-left: auto;
    margin-right: auto;
}

/* Rectangles */
.rect {
    position: absolute;
    bottom: 0;
    left: 0;
    display: none; /* hidden by default */
}

.landscape {
    background: rgba(33,150,243,0.35);
    border: 2px solid #2196F3;
}

.portrait {
    background: rgba(255,152,0,0.35);
    border: 2px solid #FF9800;
}

.sensor {
    background: rgba(255,255,255,0.4);
    border: 2px solid white;
}

.dimension-tag {
    position: absolute;
    bottom: 5px;
    right: 5px;
    font-size: 12px;
    padding: 4px 6px;
    border-radius: 4px;
    font-weight: bold;
    color: white;
}

.tag-landscape {
    background: #2196F3;
}

.tag-portrait {
    background: #FF9800;
}

.formula {
    margin-top: 15px;
    font-size: 12px;
    opacity: 0.6;
    text-align: center;
}
</style>
</head>

<body>

<div class="container">
    <h1>Sensor → Resolution</h1>

    <label>Sensor Width</label>
    <input type="number" step="any" id="sensorWidth" oninput="calculate()">

    <label>Sensor Height</label>
    <input type="number" step="any" id="sensorHeight" oninput="calculate()">

    <label>Final Image Long Side (pixels)</label>
    <input type="number" step="any" id="longSide" oninput="calculate()">

    <div class="sensor-presets">
        <button onclick="setSensor(23.76,13.365)">16:9 Digital</button>
        <button onclick="setSensor(24.892,18.9121)">35mm Full</button>
        <button onclick="setSensor(22.2,14.8)">APS-C</button>
        <button onclick="setSensor(36,24)">Full Frame</button>
    </div>

    <div class="presets">
        <button onclick="setPreset(1920)">1920</button>
        <button onclick="setPreset(3840)">3840</button>
        <button onclick="setPreset(4000)">4000</button>
        <button onclick="setPreset(4096)">4096</button>
    </div>

    <div class="result" id="output">
        Enter values above
    </div>

    <div class="preview-container">
        <div id="landscapeRect" class="rect landscape">
            <div id="landscapeTag" class="dimension-tag tag-landscape"></div>
        </div>

        <div id="portraitRect" class="rect portrait">
            <div id="portraitTag" class="dimension-tag tag-portrait"></div>
        </div>

        <div id="sensorRect" class="rect sensor"></div>
    </div>

    <div class="formula">
        Short Side = Long Side × (Short ÷ Long)
    </div>

</div>

<script>
function setPreset(value) {
    document.getElementById('longSide').value = value;
    calculate();
}

function setSensor(w, h) {
    document.getElementById('sensorWidth').value = w;
    document.getElementById('sensorHeight').value = h;
    calculate();
}

function hidePreviews() {
    document.getElementById('landscapeRect').style.display = "none";
    document.getElementById('portraitRect').style.display = "none";
    document.getElementById('sensorRect').style.display = "none";
}

function showPreviews() {
    document.getElementById('landscapeRect').style.display = "block";
    document.getElementById('portraitRect').style.display = "block";
    document.getElementById('sensorRect').style.display = "block";
}

function calculate() {
    const w = parseFloat(document.getElementById('sensorWidth').value);
    const h = parseFloat(document.getElementById('sensorHeight').value);
    const longSide = parseFloat(document.getElementById('longSide').value);

    if (isNaN(w) || isNaN(h) || isNaN(longSide) || w <= 0 || h <= 0 || longSide <= 0) {
        document.getElementById('output').innerText = "Enter valid positive numbers.";
        hidePreviews();
        return;
    }

    const sensorLong = Math.max(w, h);
    const sensorShort = Math.min(w, h);

    const shortSide = Math.round(longSide * (sensorShort / sensorLong));
    const ratio = (sensorLong / sensorShort).toFixed(2);

    document.getElementById('output').innerHTML =
        `Landscape: ${longSide} × ${shortSide}<br>
         Portrait: ${shortSide} × ${longSide}<br>
         Aspect Ratio: ${ratio}:1`;

    updatePreview(w, h, longSide, shortSide);
    showPreviews();
}

function updatePreview(sensorW, sensorH, longSide, shortSide) {

    const containerSize = 400;

    const landscapeW = longSide;
    const landscapeH = shortSide;

    const portraitW = shortSide;
    const portraitH = longSide;

    const maxDimension = Math.max(
        landscapeW, landscapeH,
        portraitW, portraitH
    );

    const scale = containerSize / maxDimension;

    const landscapeRect = document.getElementById('landscapeRect');
    landscapeRect.style.width = (landscapeW * scale) + "px";
    landscapeRect.style.height = (landscapeH * scale) + "px";
    document.getElementById('landscapeTag').innerText =
        `${landscapeW} × ${landscapeH}`;

    const portraitRect = document.getElementById('portraitRect');
    portraitRect.style.width = (portraitW * scale) + "px";
    portraitRect.style.height = (portraitH * scale) + "px";
    document.getElementById('portraitTag').innerText =
        `${portraitW} × ${portraitH}`;

    const sensorRect = document.getElementById('sensorRect');
    sensorRect.style.width = (sensorW * scale) + "px";
    sensorRect.style.height = (sensorH * scale) + "px";
}
</script>

</body>
</html>
