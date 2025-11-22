<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BrakePoint Physics: Stopping Distance Lab</title>
    <style>
        :root {
            --primary: #3b82f6;
            --danger: #ef4444;
            --warning: #f59e0b;
            --success: #10b981;
            --dark: #1e293b;
            --light: #f8fafc;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            background-color: var(--light);
            color: var(--dark);
            display: flex;
            flex-direction: column;
            height: 100vh;
        }

        /* Header */
        header {
            background-color: var(--dark);
            color: white;
            padding: 1rem;
            text-align: center;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }

        /* Main Layout */
        .container {
            display: flex;
            flex: 1;
            overflow: hidden;
        }

        /* Left Panel: Controls */
        .controls {
            width: 300px;
            background: white;
            padding: 20px;
            border-right: 1px solid #ccc;
            overflow-y: auto;
            box-shadow: 2px 0 5px rgba(0,0,0,0.05);
        }

        .control-group {
            margin-bottom: 25px;
        }

        label {
            font-weight: bold;
            display: block;
            margin-bottom: 5px;
            font-size: 0.9rem;
        }

        input[type="range"] {
            width: 100%;
            cursor: pointer;
        }

        .value-display {
            font-family: monospace;
            float: right;
            color: var(--primary);
        }

        button {
            width: 100%;
            padding: 12px;
            background-color: var(--primary);
            color: white;
            border: none;
            border-radius: 6px;
            font-size: 1rem;
            cursor: pointer;
            transition: background 0.2s;
        }

        button:hover {
            background-color: #2563eb;
        }

        button.reset {
            background-color: var(--dark);
            margin-top: 10px;
        }

        /* Middle Panel: Simulation */
        .simulation-area {
            flex: 1;
            position: relative;
            background-color: #e2e8f0;
            display: flex;
            flex-direction: column;
        }

        canvas {
            background: #94a3b8;
            width: 100%;
            height: 60%; /* Top 60% is the road */
            display: block;
        }

        /* Graphs Area */
        .charts {
            height: 40%;
            background: white;
            padding: 10px;
            display: flex;
            justify-content: center;
            align-items: center;
            border-top: 1px solid #cbd5e1;
        }

        /* Right Panel: The Math (The "Tackle" part) */
        .educational-panel {
            width: 350px;
            background: #fff;
            border-left: 1px solid #ccc;
            padding: 20px;
            overflow-y: auto;
        }

        .math-card {
            background: #f1f5f9;
            border-radius: 8px;
            padding: 15px;
            margin-bottom: 15px;
            border-left: 4px solid var(--primary);
        }

        .math-card.thinking { border-color: var(--warning); }
        .math-card.braking { border-color: var(--danger); }
        
        .equation {
            font-family: 'Courier New', Courier, monospace;
            background: white;
            padding: 8px;
            border-radius: 4px;
            margin: 10px 0;
            font-weight: bold;
        }

        .breakdown {
            font-size: 0.9rem;
            color: #475569;
        }

        .highlight {
            color: var(--primary);
            font-weight: bold;
        }

        /* Responsive */
        @media (max-width: 900px) {
            .container { flex-direction: column; overflow-y: scroll;}
            .controls, .educational-panel { width: 100%; border: none; }
            canvas { height: 300px; }
        }
    </style>
</head>
<body>

<header>
    <h2>BrakePoint Physics 🚗</h2>
    <small>Apply SUVAT Equations to Master Stopping Distance</small>
</header>

<div class="container">
    <!-- 1. CONTROLS -->
    <div class="controls">
        <h3>Simulation Parameters</h3>
        
        <div class="control-group">
            <label>Initial Velocity ($u$) <span id="val-u" class="value-display">20 m/s</span></label>
            <input type="range" id="input-u" min="5" max="40" step="1" value="20">
            <small style="color: #666;">(approx 72 km/h)</small>
        </div>

        <div class="control-group">
            <label>Reaction Time ($t_{react}$) <span id="val-t" class="value-display">1.0 s</span></label>
            <input type="range" id="input-t" min="0.2" max="3.0" step="0.1" value="1.0">
            <small style="color: #666;">Thinking Phase</small>
        </div>

        <div class="control-group">
            <label>Deceleration ($a$) <span id="val-a" class="value-display">-5 m/s²</span></label>
            <input type="range" id="input-a" min="2" max="12" step="0.5" value="5">
            <small style="color: #666;">Braking Power (Dry vs Wet Road)</small>
        </div>

        <div class="control-group">
            <label>Obstacle Distance <span id="val-obs" class="value-display">60 m</span></label>
            <input type="range" id="input-obs" min="20" max="150" step="5" value="60">
        </div>

        <button onclick="startSimulation()">▶ Run Test</button>
        <button class="reset" onclick="resetSim()">↺ Reset</button>

        <div id="status-msg" style="margin-top:20px; text-align: center; font-weight: bold;"></div>
    </div>

    <!-- 2. SIMULATION -->
    <div class="simulation-area">
        <canvas id="simCanvas"></canvas>
        <div class="charts">
            <canvas id="graphCanvas"></canvas>
        </div>
    </div>

    <!-- 3. EDUCATIONAL BREAKDOWN -->
    <div class="educational-panel">
        <h3>Equation Tackle Box</h3>
        <p>See how the physics engines calculates your fate.</p>

        <!-- Thinking Phase -->
        <div class="math-card thinking">
            <h4>1. Thinking Phase</h4>
            <p>Newton's 1st Law (Constant Velocity)</p>
            <div class="equation">s₁ = u × t</div>
            <div class="breakdown" id="math-thinking">
                Adjust sliders to see calculation.
            </div>
        </div>

        <!-- Braking Phase -->
        <div class="math-card braking">
            <h4>2. Braking Phase</h4>
            <p>SUVAT Equation (No time variable)</p>
            <div class="equation">v² = u² + 2as</div>
            <div class="breakdown">
                We know final velocity ($v$) is 0. <br>
                Rearranging for distance ($s$):
            </div>
            <div class="equation">s₂ = -u² / 2a</div>
            <div class="breakdown" id="math-braking">
                Adjust sliders to see calculation.
            </div>
        </div>

        <!-- Total -->
        <div class="math-card">
            <h4>Total Stopping Distance</h4>
            <div class="equation">S_total = s₁ + s₂</div>
            <div class="breakdown" id="math-total">
                Waiting for data...
            </div>
        </div>
    </div>
</div>

<script>
    // --- CONFIGURATION & STATE ---
    const canvas = document.getElementById('simCanvas');
    const ctx = canvas.getContext('2d');
    const graphCanvas = document.getElementById('graphCanvas');
    const gCtx = graphCanvas.getContext('2d');

    let state = {
        u: 20,      // Initial velocity
        tr: 1.0,    // Reaction time
        a: -5,      // Deceleration (magnitude stored as pos, applied as neg)
        obs: 60,    // Obstacle distance
        running: false,
        finished: false,
        time: 0,    // Simulation time
        carX: 0,    // Car position
        carV: 0,    // Current velocity
        phase: 'idle' // idle, thinking, braking, stopped, crashed
    };

    // Physics scale: 1 meter = 10 pixels
    const SCALE = 10; 
    const CAR_WIDTH = 40; // visual width
    const CAR_HEIGHT = 20;

    // --- INITIALIZATION ---
    function resizeCanvas() {
        canvas.width = canvas.parentElement.offsetWidth;
        canvas.height = canvas.parentElement.offsetHeight * 0.6;
        graphCanvas.width = canvas.parentElement.offsetWidth;
        graphCanvas.height = canvas.parentElement.offsetHeight * 0.4;
        drawScene();
        drawGraphEmpty();
    }
    window.addEventListener('resize', resizeCanvas);

    // Input Listeners
    function updateValues() {
        state.u = parseFloat(document.getElementById('input-u').value);
        state.tr = parseFloat(document.getElementById('input-t').value);
        state.a = -parseFloat(document.getElementById('input-a').value); // Make negative
        state.obs = parseFloat(document.getElementById('input-obs').value);

        // Update text displays
        document.getElementById('val-u').innerText = state.u + " m/s";
        document.getElementById('val-t').innerText = state.tr + " s";
        document.getElementById('val-a').innerText = state.a + " m/s²";
        document.getElementById('val-obs').innerText = state.obs + " m";

        updateMathPanel();
        if(!state.running) drawScene();
    }

    ['input-u', 'input-t', 'input-a', 'input-obs'].forEach(id => {
        document.getElementById(id).addEventListener('input', updateValues);
    });

    // --- PHYSICS ENGINE ---
    function updateMathPanel() {
        // Calculate Theoretical Values
        const s1 = state.u * state.tr; // Thinking distance
        const s2 = -(state.u * state.u) / (2 * state.a); // Braking distance
        const total = s1 + s2;

        // Update HTML
        document.getElementById('math-thinking').innerHTML = `
            s₁ = ${state.u}m/s × ${state.tr}s <br>
            <span class="highlight">s₁ = ${s1.toFixed(2)} m</span>
        `;

        document.getElementById('math-braking').innerHTML = `
            s₂ = -(${state.u})² / (2 × ${state.a}) <br>
            s₂ = -${(state.u*state.u)} / ${2*state.a} <br>
            <span class="highlight">s₂ = ${s2.toFixed(2)} m</span>
        `;

        const isSafe = total < state.obs;
        const color = isSafe ? 'var(--success)' : 'var(--danger)';
        const text = isSafe ? 'SAFE STOP' : 'CRASH PREDICTED';

        document.getElementById('math-total').innerHTML = `
            S_total = ${s1.toFixed(2)} + ${s2.toFixed(2)} <br>
            <span class="highlight" style="font-size:1.2em">${total.toFixed(2)} m</span> <br>
            <strong style="color:${color}">[${text}]</strong>
        `;
    }

    // --- SIMULATION LOOP ---
    let animationFrame;
    let dataPoints = []; // For the graph

    function startSimulation() {
        if(state.running) return;
        resetSim();
        state.running = true;
        state.carV = state.u;
        state.phase = 'thinking';
        lastTime = performance.now();
        document.getElementById('status-msg').innerText = "👀 THINKING PHASE...";
        document.getElementById('status-msg').style.color = "var(--warning)";
        
        loop();
    }

    let lastTime = 0;
    function loop() {
        if(!state.running) return;

        const now = performance.now();
        // Calculate total time elapsed since start
        state.time = (now - lastTime) / 1000; 

        // --- FIX: CALCULATE EXACT PHYSICS BASED ON TIME ---
        
        // 1. Check if we are still in Reaction Phase
        if (state.time < state.tr) {
            state.phase = 'thinking';
            
            // Exact Position: s = v * t
            state.carX = state.u * state.time;
            
            // Velocity is constant
            state.carV = state.u;
        
        } else {
            // 2. We are in Braking Phase
            if(state.phase === 'thinking') {
                state.phase = 'braking';
                document.getElementById('status-msg').innerText = "🛑 BRAKING PHASE!";
                document.getElementById('status-msg').style.color = "var(--danger)";
            }

            // Time spent braking
            const brakeTime = state.time - state.tr;

            // Calculate current velocity: v = u + at
            let currentV = state.u + (state.a * brakeTime);

            if (currentV <= 0) {
                // CAR HAS STOPPED
                state.carV = 0;
                state.running = false;
                state.finished = true;
                
                // Snap position to the exact theoretical stopping distance to ensure perfect match
                // Total = (Thinking Dist) + (Braking Dist)
                const s1 = state.u * state.tr;
                const s2 = -(state.u * state.u) / (2 * state.a);
                state.carX = s1 + s2;
                
                endGame();
            } else {
                // CAR IS STILL MOVING
                state.carV = currentV;

                // Exact Position: 
                // Total Dist = (Thinking Dist) + (Distance covered during braking time)
                // Braking Dist Formula: s = ut + 0.5at^2 (where t is brakeTime)
                const thinkingDist = state.u * state.tr;
                const brakingDist = (state.u * brakeTime) + (0.5 * state.a * (brakeTime * brakeTime));
                
                state.carX = thinkingDist + brakingDist;
            }
        }

        // Check Crash
        if (state.carX >= state.obs) {
            state.carX = state.obs; // Clamp at obstacle
            state.carV = 0;
            state.phase = 'crashed';
            state.running = false;
            state.finished = true;
            endGame();
        }

        // Store Data for Graph
        if (state.running) {
            dataPoints.push({t: state.time, v: state.carV});
        }

        drawScene();
        drawGraphLive();
        
        if(state.running) {
            animationFrame = requestAnimationFrame(loop);
        }
    }

    function endGame() {
        if(state.phase === 'crashed') {
            document.getElementById('status-msg').innerText = "💥 CRASH! You needed more distance.";
        } else {
            document.getElementById('status-msg').innerText = "✅ SAFE STOP. Good job.";
            document.getElementById('status-msg').style.color = "var(--success)";
        }
        drawScene(); // Final draw
    }

    function resetSim() {
        state.running = false;
        state.finished = false;
        state.time = 0;
        state.carX = 0;
        state.carV = 0;
        state.phase = 'idle';
        dataPoints = [];
        cancelAnimationFrame(animationFrame);
        document.getElementById('status-msg').innerText = "";
        updateValues(); // Resets calculations
        drawScene();
        drawGraphEmpty();
    }

    // --- RENDERING ---
    function drawScene() {
        // Clear
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // Road
        const roadY = canvas.height / 2;
        ctx.fillStyle = "#334155";
        ctx.fillRect(0, roadY - 40, canvas.width, 80);
        
        // Road Markings
        ctx.strokeStyle = "#fff";
        ctx.setLineDash([20, 20]);
        ctx.beginPath();
        ctx.moveTo(0, roadY);
        ctx.lineTo(canvas.width, roadY);
        ctx.stroke();
        ctx.setLineDash([]);

        // Camera / Viewport Logic
        // Keep car in left 20% of screen, or scroll if x > 20% width
        let cameraOffset = 0;
        const carPixelX = state.carX * SCALE;
        const centerTrigger = canvas.width * 0.2;
        
        if (carPixelX > centerTrigger) {
            cameraOffset = carPixelX - centerTrigger;
        }

        ctx.save();
        ctx.translate(-cameraOffset, 0);

        // Draw Start Line
        ctx.fillStyle = "#fff";
        ctx.fillRect(0, roadY - 40, 5, 80);
        ctx.fillText("0m", 5, roadY + 55);

        // Draw Obstacle
        const obsPixelX = state.obs * SCALE;
        ctx.fillStyle = "#ef4444"; // Red wall
        ctx.fillRect(obsPixelX, roadY - 50, 10, 100);
        ctx.fillStyle = "#fff";
        ctx.fillText("HAZARD", obsPixelX - 20, roadY - 60);
        ctx.fillText(state.obs + "m", obsPixelX - 10, roadY + 55);

        // Draw Markers every 10m
        ctx.fillStyle = "#94a3b8";
        for(let m=10; m < state.obs + 50; m+=10) {
            ctx.fillRect(m*SCALE, roadY + 35, 2, 10);
        }

        // Draw Car
        const drawX = state.carX * SCALE;
        
        // Thinking Color: Yellow, Braking Color: Red
        ctx.fillStyle = (state.phase === 'thinking') ? '#f59e0b' : (state.phase === 'idle' ? '#3b82f6' : '#ef4444');
        
        // Simple Car Shape
        ctx.fillRect(drawX, roadY - 15, 40, 30); // Body
        ctx.fillStyle = "#000";
        ctx.beginPath();
        ctx.arc(drawX + 10, roadY + 15, 6, 0, Math.PI*2); // Wheel
        ctx.arc(drawX + 30, roadY + 15, 6, 0, Math.PI*2); // Wheel
        ctx.fill();

        // Effects (Skid marks)
        if (state.phase === 'braking' || state.finished) {
            // Calculate where braking started
            const brakeStartX = (state.u * state.tr) * SCALE;
            const currentX = state.carX * SCALE;
            if (currentX > brakeStartX) {
                ctx.globalCompositeOperation = 'destination-over';
                ctx.fillStyle = "#1e293b";
                ctx.fillRect(brakeStartX, roadY + 10, currentX - brakeStartX, 4);
                ctx.fillRect(brakeStartX, roadY - 10, currentX - brakeStartX, 4);
                ctx.globalCompositeOperation = 'source-over';
            }
        }

        // Bang effect if crashed
        if (state.phase === 'crashed') {
            ctx.fillStyle = "orange";
            ctx.beginPath();
            ctx.arc(drawX + 40, roadY, 30, 0, Math.PI*2);
            ctx.fill();
            ctx.fillStyle = "yellow";
            ctx.fillText("CRASH!", drawX + 20, roadY - 30);
        }

        ctx.restore();

        // HUD (Heads up display) - Fixed on screen
        ctx.fillStyle = "rgba(0,0,0,0.7)";
        ctx.fillRect(10, 10, 160, 60);
        ctx.fillStyle = "#fff";
        ctx.font = "14px monospace";
        ctx.fillText(`Speed: ${state.carV.toFixed(1)} m/s`, 20, 30);
        ctx.fillText(`Dist : ${state.carX.toFixed(1)} m`, 20, 50);
    }

    // --- GRAPHING ---
    function drawGraphEmpty() {
        gCtx.clearRect(0,0, graphCanvas.width, graphCanvas.height);
        gCtx.fillStyle = "#64748b";
        gCtx.font = "12px Arial";
        gCtx.fillText("Velocity (v) vs Time (t)", 10, 20);
        
        // Axis
        gCtx.strokeStyle = "#ccc";
        gCtx.beginPath();
        gCtx.moveTo(40, 10);
        gCtx.lineTo(40, graphCanvas.height - 20); // Y axis
        gCtx.lineTo(graphCanvas.width, graphCanvas.height - 20); // X axis
        gCtx.stroke();
    }

    function drawGraphLive() {
        drawGraphEmpty();
        if(dataPoints.length < 2) return;

        gCtx.strokeStyle = "#3b82f6";
        gCtx.lineWidth = 2;
        gCtx.beginPath();

        // Map data to canvas pixels
        // Max Time approx 10s, Max V approx 40
        const originX = 40;
        const originY = graphCanvas.height - 20;
        const scaleX = (graphCanvas.width - 50) / (state.tr + 5); // Dynamic scaling approx
        const scaleY = (graphCanvas.height - 40) / 45; // Max speed 40 + buffer

        dataPoints.forEach((pt, i) => {
            const x = originX + pt.t * scaleX;
            const y = originY - pt.v * scaleY;
            if(i===0) gCtx.moveTo(x,y);
            else gCtx.lineTo(x,y);
        });
        gCtx.stroke();

        // Draw Breakpoint Line
        const breakX = originX + state.tr * scaleX;
        gCtx.strokeStyle = "#f59e0b";
        gCtx.setLineDash([5, 5]);
        gCtx.beginPath();
        gCtx.moveTo(breakX, 10);
        gCtx.lineTo(breakX, originY);
        gCtx.stroke();
        gCtx.setLineDash([]);
        gCtx.fillStyle = "#f59e0b";
        gCtx.fillText("Brake", breakX + 5, 30);
    }

    // Init
    updateValues();
    resizeCanvas();

</script>
</body>
</html>
