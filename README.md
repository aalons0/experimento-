<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Experimento Auditivo</title>
<style>
body { margin:0; font-family:Arial; background:#121212; color:white; }
.container { max-width:800px; margin:auto; padding:20px; }
.card { background:#1e1e1e; padding:20px; border-radius:18px; margin-bottom:20px; box-shadow:0 4px 20px rgba(0,0,0,0.5);} 
button { background:#3a86ff; border:none; padding:12px 20px; border-radius:14px; color:white; cursor:pointer; margin:5px; }
button:hover { background:#265ecf; }
input, select { width:100%; padding:10px; margin-top:10px; border-radius:12px; border:none; }
.hidden { display:none; }
.slider { width:100%; }
</style>
</head>
<body>
<div class="container">

<div id="intro" class="card">
<h2>Experimento auditivo</h2>
<p>Escucharás pares de sonidos. Indica cuál suena más fuerte o si no percibes diferencia.</p>

<label>Edad</label>
<input type="number" id="age">

<label>Conocimientos musicales</label>
<select id="music">
<option value="ninguno">Sin conocimientos</option>
<option value="basico">Estudios básicos</option>
<option value="medio">Estudios medios</option>
<option value="superior">Estudios superiores</option>
</select>

<button onclick="goToCalibration()">Siguiente</button>
</div>

<div id="calibration" class="card hidden">
<h2>Calibración de sonido</h2>
<p>Ajusta el volumen hasta que escuches el tono de forma cómoda (ni muy bajo ni molesto).</p>

<input type="range" min="0.01" max="0.5" step="0.01" value="0.2" id="volume" class="slider">
<button onclick="playCalibration()">Reproducir tono</button>
<button onclick="startExperiment()">Comenzar experimento</button>
</div>

<div id="experiment" class="card hidden">
<h3 id="trialTitle"></h3>
<button onclick="playTrial()">Escuchar</button>

<p>¿Cuándo suena más fuerte?</p>
<button onclick="answer(1)">Primer sonido</button>
<button onclick="answer(2)">Segundo sonido</button>
<button onclick="answer(0)">No percibo diferencia</button>
</div>

<div id="end" class="card hidden">
<h2>Gracias por participar</h2>
</div>

</div>

<script>
const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
const types = ["sine","sawtooth"];
const freqs = [100,500,2000];

let trials = [];
let currentTrial = 0;
let results = [];
let baseVolume = 0.2;

function goToCalibration() {
  document.getElementById("intro").classList.add("hidden");
  document.getElementById("calibration").classList.remove("hidden");
}

function playCalibration() {
  baseVolume = parseFloat(document.getElementById("volume").value);
  let t = audioCtx.currentTime;
  playTone("sine", 500, baseVolume, t);
}

function generateTrials() {
  let variations = [0.5,1,1.5,2,2.5];
  let all = [];

  types.forEach(type => {
    freqs.forEach(freq => {
      let shuffled = [...variations].sort(()=>Math.random()-0.5);
      shuffled.forEach(v => {
        let dir = Math.random() > 0.5 ? 1 : -1;
        all.push({type, freq, change: v * dir});
      });
    });
  });

  return all.sort(()=>Math.random()-0.5);
}

function dbToGain(db) {
  return Math.pow(10, db/20);
}

function playTone(type, freq, gainVal, startTime) {
  let osc = audioCtx.createOscillator();
  let gain = audioCtx.createGain();

  osc.type = type;
  osc.frequency.value = freq;
  gain.gain.value = gainVal;

  osc.connect(gain).connect(audioCtx.destination);
  osc.start(startTime);
  osc.stop(startTime + 1);
}

function playTrial() {
  let t = audioCtx.currentTime;
  let trial = trials[currentTrial];

  let alteredGain = baseVolume * dbToGain(trial.change);

  playTone(trial.type, trial.freq, baseVolume, t);
  playTone(trial.type, trial.freq, alteredGain, t + 1.5);
}

function answer(choice) {
  let trial = trials[currentTrial];

  results.push({
    type: trial.type,
    freq: trial.freq,
    change: trial.change,
    answer: choice
  });

  currentTrial++;

  if(currentTrial >= trials.length) {
    document.getElementById("experiment").classList.add("hidden");
    document.getElementById("end").classList.remove("hidden");
  } else {
    updateUI();
  }
}

function updateUI() {
  let t = trials[currentTrial];
  document.getElementById("trialTitle").innerText =
    `Prueba ${currentTrial+1} de 30: ${t.type} - ${t.freq} Hz`;
}

function startExperiment() {
  baseVolume = parseFloat(document.getElementById("volume").value);
  trials = generateTrials();

  document.getElementById("calibration").classList.add("hidden");
  document.getElementById("experiment").classList.remove("hidden");

  updateUI();
}
</script>

</body>
</html>
