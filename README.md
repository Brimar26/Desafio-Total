
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>MateVida Digital - Desafío Total</title>
    <style>
        :root { --amarillo: #FFD700; --azul: #3498DB; --verde: #2ECC71; --naranja: #FF8C00; }
        body { background: var(--amarillo); font-family: 'Segoe UI', sans-serif; text-align: center; padding: 20px; }
        .game-container { display: flex; justify-content: center; gap: 40px; margin-top: 20px; }
        .col { display: flex; flex-direction: column; gap: 10px; width: 160px; }
        .item { padding: 15px; background: white; border-radius: 12px; cursor: pointer; border: 4px solid #fff; font-weight: bold; transition: 0.1s; box-shadow: 0 4px 0 #ccc; }
        .item.active { border-color: var(--naranja); background: #ffe0b2; }
        .item.correct { border-color: var(--verde); background: #d4edda; }
        #pantalla-final { display: none; margin-top: 50px; }
        .btn { padding: 15px 30px; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; color: white; font-size: 1.2em; background: var(--azul); }
    </style>
</head>
<body>

<div id="ui">
    <div id="avatar-display" style="font-size: 70px;">🎓</div>
    <div id="mensaje">¡Toca un ejercicio y luego su resultado!</div>
    <div id="cronometro" style="font-weight:bold; font-size: 1.2em; margin: 10px;">Tiempo: 0s</div>
    <button id="btn-iniciar" class="btn" style="background: var(--verde);" onclick="iniciarJuego()">COMENZAR PARTIDA</button>
</div>

<div class="game-container" id="area-juego">
    <div class="col" id="col-a"></div>
    <div class="col" id="col-b"></div>
</div>

<div id="pantalla-final">
    <div id="resultado-texto" style="font-size: 2em; font-weight: bold; color: #333;"></div>
    <br><br>
    <button class="btn" onclick="location.reload()">REINICIAR PARTIDA</button>
</div>

<script>
    const baseDatos = [
        {op: "12 x 2", res: 24}, {op: "81 ÷ 9", res: 9}, {op: "7 x 3", res: 21}, 
        {op: "48 ÷ 6", res: 8}, {op: "5 x 5", res: 25}, {op: "120 ÷ 10", res: 12},
        {op: "34 + 18", res: 52}, {op: "75 + 46", res: 121}, {op: "61 - 19", res: 42},
        {op: "90 - 45", res: 45}, {op: "25 + 25", res: 50}, {op: "144 ÷ 12", res: 12},
        {op: "8 x 4", res: 32}, {op: "100 - 30", res: 70}, {op: "55 + 45", res: 100}
    ];

    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    
    function playTone(freq, duration, type = 'sine') {
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = type; osc.frequency.value = freq;
        osc.connect(gain); gain.connect(audioCtx.destination);
        osc.start(); gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + duration);
        osc.stop(audioCtx.currentTime + duration);
    }

    let aciertos = 0, intentos = 0, selOp = null, terminados = 0, segundos = 0, timer;
    const totalEjercicios = 15;

    function iniciarJuego() {
        document.getElementById('btn-iniciar').style.display = 'none';
        aciertos = 0; intentos = 0; terminados = 0; segundos = 0;
        
        timer = setInterval(() => { segundos++; document.getElementById('cronometro').innerText = "Tiempo: " + segundos + "s"; }, 1000);
        
        // Mezclar y tomar los 15 ejercicios
        const ops = baseDatos.sort(() => Math.random() - 0.5);
        const colA = document.getElementById('col-a');
        const colB = document.getElementById('col-b');
        colA.innerHTML = ''; colB.innerHTML = '';
        
        ops.forEach((item, i) => {
            colA.innerHTML += `<div class="item" id="a${i}" onclick="seleccionar('a${i}', ${item.res})">${item.op}</div>`;
        });
        
        // Mezclar respuestas
        [...ops].sort(() => Math.random() - 0.5).forEach((item, i) => {
            colB.innerHTML += `<div class="item" id="b${i}" onclick="verificar('b${i}', ${item.res})">${item.res}</div>`;
        });
    }

    function seleccionar(id, res) {
        document.querySelectorAll('.col .item').forEach(e => e.classList.remove('active'));
        document.getElementById(id).classList.add('active');
        selOp = {id, res};
    }

    function verificar(id, res) {
        if(!selOp) return;
        intentos++;
        if(selOp.res === res) {
            playTone(600, 0.2);
            document.getElementById(id).classList.add('correct');
            document.getElementById(selOp.id).classList.add('correct');
            aciertos++; terminados++;
            if(terminados === totalEjercicios) finalizar();
        } else {
            playTone(150, 0.2, 'sawtooth');
        }
        selOp = null;
    }

    function finalizar() {
        clearInterval(timer);
        document.getElementById('area-juego').style.display = 'none';
        document.getElementById('pantalla-final').style.display = 'block';
        let porcentaje = Math.round((aciertos / intentos) * 100);
        [523, 659, 783, 1046].forEach((f, i) => setTimeout(() => playTone(f, 0.3), i * 200));
        document.getElementById('resultado-texto').innerHTML = `🏆<br>Puntaje Final: ${porcentaje}%<br>Tiempo: ${segundos}s`;
    }
</script>
</body>
</html>
