<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Aether VJ: Seamless Flow</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.6.0/p5.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.6.0/addons/p5.sound.min.js"></script>
    <script src="https://www.youtube.com/iframe_api"></script>
    <style>
        /* --- Base Styling --- */
        body { margin: 0; padding: 0; background: #111; overflow: hidden; font-family: 'Helvetica Neue', sans-serif; color: #ccc; }
        * { box-sizing: border-box; outline: none; -webkit-tap-highlight-color: transparent; }
        canvas { display: block; position: fixed; top: 0; left: 0; z-index: 1; pointer-events: none; }

        /* --- UI Layer (z-index 100+) --- */
        #ui-container {
            position: fixed; inset: 0; pointer-events: none; z-index: 100;
            display: flex; flex-direction: column; justify-content: space-between;
            padding: 25px;
        }

        /* Header */
        .header { display: flex; align-items: flex-start; justify-content: space-between; pointer-events: auto; }
        .title-group { display: flex; flex-direction: column; gap: 8px; }
        h1 { margin: 0; font-size: 11px; letter-spacing: 5px; opacity: 0.6; text-transform: uppercase; color: #fff; }
        
        /* Sync Switch */
        .sync-switch {
            display: inline-flex; align-items: center; gap: 8px;
            background: rgba(255,255,255,0.05); padding: 6px 14px; border-radius: 20px; border: 1px solid rgba(255,255,255,0.1);
            cursor: pointer; transition: all 0.2s; backdrop-filter: blur(10px);
        }
        .sync-switch:hover { background: rgba(255,255,255,0.1); }
        .sync-switch.active { background: rgba(0, 255, 255, 0.15); border-color: rgba(0, 255, 255, 0.5); box-shadow: 0 0 15px rgba(0, 255, 255, 0.2); }
        .indicator { width: 8px; height: 8px; border-radius: 50%; background: #444; transition: all 0.3s; }
        .sync-switch.active .indicator { background: #0ff; box-shadow: 0 0 8px #0ff; animation: pulse-beat 0.5s infinite alternate; }
        .switch-label { font-size: 10px; letter-spacing: 1px; color: #777; font-weight: 700; }
        .sync-switch.active .switch-label { color: #fff; }
        @keyframes pulse-beat { 0% { opacity: 0.5; transform: scale(1.0); } 100% { opacity: 1.0; transform: scale(1.2); } }

        /* Controls Panel */
        .controls-panel {
            pointer-events: auto;
            background: rgba(10,10,10,0.85); 
            backdrop-filter: blur(40px); -webkit-backdrop-filter: blur(40px);
            border: 1px solid rgba(255,255,255,0.05);
            border-radius: 4px;
            padding: 25px; width: 280px; align-self: flex-end;
            margin-bottom: 120px;
            transition: transform 0.6s cubic-bezier(0.16, 1, 0.3, 1), opacity 0.6s;
            box-shadow: 0 20px 50px rgba(0,0,0,0.5);
            z-index: 101;
        }
        .controls-panel.hidden { transform: translateX(calc(100% + 50px)); opacity: 0; pointer-events: none; }

        /* Visualizer Canvas */
        #viz-canvas {
            width: 100%; height: 40px; background: rgba(0,0,0,0.3);
            border-bottom: 1px solid rgba(255,255,255,0.1); margin-bottom: 20px;
            border-radius: 2px; display: block;
        }

        /* Sliders */
        .slider-group { margin-bottom: 22px; position: relative; }
        .label-row { display: flex; justify-content: space-between; margin-bottom: 8px; }
        label { font-size: 9px; color: #888; text-transform: uppercase; letter-spacing: 2px; font-weight: 500; }
        .value { font-size: 9px; font-family: 'Courier New', monospace; color: #bbb; }
        
        .sync-dot {
            position: absolute; right: -8px; top: 2px; width: 4px; height: 4px; background: #0ff; border-radius: 50%;
            opacity: 0; transition: opacity 0.3s; box-shadow: 0 0 8px #0ff;
        }
        .sync-active .sync-dot { opacity: 1.0; }

        input[type=range] { 
            width: 100%; -webkit-appearance: none; background: transparent; height: 20px; 
            cursor: pointer; margin: 0; touch-action: none; position: relative; z-index: 200; 
        }
        input[type=range]::-webkit-slider-runnable-track { width: 100%; height: 2px; background: rgba(255,255,255,0.15); border-radius: 2px;}
        input[type=range]::-webkit-slider-thumb {
            -webkit-appearance: none; height: 14px; width: 14px; border-radius: 50%;
            background: #111; margin-top: -6px; box-shadow: 0 0 0 1px rgba(255,255,255,0.5); transition: transform 0.1s; 
        }
        input[type=range]:active::-webkit-slider-thumb { background: #fff; transform: scale(1.2); }

        /* Toggle Button */
        .toggle-btn-fixed {
            position: fixed; top: 25px; right: 25px; z-index: 200; pointer-events: auto;
            width: 30px; height: 30px; border-radius: 50%; display: flex; align-items: center; justify-content: center;
            cursor: pointer; border: 1px solid rgba(255,255,255,0.1); color: #888;
            backdrop-filter: blur(10px); transition: all 0.3s; font-size: 14px; background: rgba(255,255,255,0.02);
        }
        .toggle-btn-fixed:hover { background: rgba(255,255,255,0.1); color: #fff; border-color: rgba(255,255,255,0.3); }

        /* Bottom Bar */
        .bottom-bar {
            position: fixed; bottom: 0; left: 0; right: 0;
            background: rgba(5,5,5,0.95); border-top: 1px solid rgba(255,255,255,0.05);
            z-index: 101; display: flex; flex-direction: column; pointer-events: auto;
            transition: transform 0.6s cubic-bezier(0.16, 1, 0.3, 1); backdrop-filter: blur(20px);
            padding-bottom: env(safe-area-inset-bottom);
        }
        .bottom-bar.hidden { transform: translateY(120%); }

        /* File Controls */
        .file-controls {
            padding: 15px 25px; display: flex; align-items: center; gap: 15px;
            border-bottom: 1px solid rgba(255,255,255,0.05);
        }
        .custom-file-btn {
            background: #222; border: 1px solid rgba(255,255,255,0.2); color: #fff;
            padding: 10px 20px; border-radius: 20px; font-size: 11px; letter-spacing: 1px;
            cursor: pointer; display: flex; align-items: center; gap: 8px; transition: all 0.3s;
            white-space: nowrap;
        }
        .custom-file-btn:active { transform: scale(0.95); background: #fff; color: #000; }
        #file-status { font-size: 10px; color: #888; font-family: 'Courier New', monospace; flex-grow: 1; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;}
        .play-pause-btn {
            width: 35px; height: 35px; border-radius: 50%; background: #0ff; border: none;
            color: #000; display: flex; align-items: center; justify-content: center; cursor: pointer;
            display: none; font-weight: bold;
        }
        .mic-mode-btn {
            background: #005500;
            border-color: #00ff00;
            color: #00ff00;
        }
        .mic-mode-btn:active {
            background: #00ff00;
            color: #000;
        }


        /* YouTube Area */
        .youtube-area {
            display: flex; align-items: center; padding: 10px 25px; gap: 15px; height: 50px;
            opacity: 0.6; transition: opacity 0.3s;
        }
        .youtube-area:hover { opacity: 1.0; }

        /* Overlay */
        #overlay {
            position: fixed; inset: 0; background: #000; z-index: 9999;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            cursor: pointer; color: #fff; transition: opacity 0.5s ease-out;
            pointer-events: auto;
        }
        #overlay:active { background: #111; }
        .start-text { font-size: 14px; letter-spacing: 8px; font-weight: 300; margin-bottom: 20px; opacity: 0.9; text-align: center; pointer-events: none; }
        .sub-text { font-size: 10px; color: #0ff; letter-spacing: 2px; text-transform: uppercase; background: rgba(0,255,255,0.1); padding: 10px 20px; border-radius: 30px; border: 1px solid rgba(0,255,255,0.3); pointer-events: none; }

        @media (max-width: 600px) {
            .controls-panel { width: 100%; margin-bottom: 140px; border-radius: 0; border:none; border-top: 1px solid rgba(255,255,255,0.1);}
            .file-controls { padding: 12px 15px; }
            .youtube-area { display: none; } 
        }
    </style>
</head>
<body>

<div id="overlay" onclick="initApp()">
    <div class="start-text">TAP TO START</div>
    <div class="sub-text">SEAMLESS FLOW</div>
</div>

<div class="toggle-btn-fixed" onclick="toggleUI()">✕</div>

<div id="ui-container">
    <div class="header">
        <div class="title-group">
            <h1>Aether VJ</h1>
            <div class="sync-switch" id="sync-btn" onclick="toggleSync()">
                <div class="indicator"></div>
                <div class="switch-label">AUDIO SYNC: OFF</div>
            </div>
        </div>
    </div>

    <div class="controls-panel" id="ctrl-panel">
        <canvas id="viz-canvas" width="230" height="40"></canvas>

        <div class="slider-group" id="grp-1">
            <div class="sync-dot"></div>
            <div class="label-row"><label>Fluidity (Speed)</label> <span class="value" id="val-1">0.5</span></div>
            <input type="range" id="param1" min="0" max="1" step="0.01" value="0.5">
        </div>
        
        <div class="slider-group" id="grp-2">
            <div class="sync-dot"></div>
            <div class="label-row"><label>Structure (Grid)</label> <span class="value" id="val-2">0.3</span></div>
            <input type="range" id="param2" min="0" max="1" step="0.01" value="0.3">
        </div>
        
        <div class="slider-group" id="grp-3">
            <div class="sync-dot"></div>
            <div class="label-row"><label>Texture (Particles)</label> <span class="value" id="val-3">0.4</span></div>
            <input type="range" id="param3" min="0" max="1" step="0.01" value="0.4">
        </div>

        <hr style="border: 0; border-top: 1px solid rgba(255,255,255,0.08); margin: 25px 0;">
        
        <div class="slider-group">
            <div class="label-row"><label>Audio Reactivity</label> <span class="value" id="val-react">1.5</span></div>
            <input type="range" id="reactivity" min="0" max="3" step="0.1" value="1.5">
        </div>
        
        <div class="slider-group" id="grp-color">
            <div class="sync-dot"></div>
            <div class="label-row"><label>Palette (Mono -> Deep Color)</label> <span class="value" id="val-color">0.2</span></div>
            <input type="range" id="colorshift" min="0" max="1" step="0.01" value="0.2">
        </div>
    </div>
</div>

<div class="bottom-bar" id="btm-bar">
    <div class="file-controls">
        <button class="custom-file-btn mic-mode-btn" onclick="switchToMic()">🎤 MIC MODE</button>
        
        <label for="audio-upload" class="custom-file-btn">
            📂 LOAD FILE
        </label>
        <input type="file" id="audio-upload" accept=".mp3,audio/*" style="display:none" onchange="handleFileSelect(this)">
        
        <div id="file-status">MIC ACTIVE</div>
        <button class="play-pause-btn" id="play-btn" onclick="togglePlay()">❚❚</button>
    </div>

    <div class="youtube-area">
        <div id="youtube-placeholder" style="width: 50px; height: 28px; background:#000; overflow:hidden;"><div id="player"></div></div>
        <div style="flex-grow:1; font-size: 9px; color:#555;">YOUTUBE (NO SYNC ON MOBILE)</div>
    </div>
</div>

<script>
    // --- Configuration ---
    const MAX_FILE_SIZE = 20 * 1024 * 1024; // 20 MB Limit

    // --- State Management ---
    let state = {
        running: false, uiHidden: false, 
        syncMode: false,
        params: [0.5, 0.3, 0.4], 
        dynamicParams: [0.5, 0.3, 0.4],
        global: { react: 1.5, paletteVal: 0.2, dynPalette: 0.2, rot: 0.0 },
        // Smooth Audio Variables
        avgBass: 0, avgMid: 0, avgTreble: 0, avgVol: 0
    };

    let myShader, fft, amplitude, mic;
    let soundFile = null; 
    let vizCtx; 

    const vert = `
        attribute vec3 aPosition; attribute vec2 aTexCoord; varying vec2 vTexCoord;
        void main() { vTexCoord = aTexCoord; gl_Position = vec4(aPosition * 2.0 - 1.0, 1.0); }
    `;

    // --- SMOOTH & SEAMLESS SHADER ---
    const frag = `
        precision highp float; 
        varying vec2 vTexCoord;
        uniform float u_time; uniform vec2 u_res;
        uniform vec3 u_params; 
        uniform float u_react; uniform float u_paletteVal; 
        // These receive the SMOOTHED audio values now
        uniform float u_bass; uniform float u_treble; uniform float u_mid; uniform float u_vol; uniform float u_rot;
        
        #define PI 3.14159265359
        mat2 rot2d(float a){ return mat2(cos(a),-sin(a),sin(a),cos(a)); }
        float hash(vec2 p) { return fract(1e4 * sin(17.0 * p.x + p.y * 0.1) * (0.1 + abs(sin(p.y * 13.0 + p.x)))); }
        
        float noise(vec2 x) {
            vec2 i = floor(x); vec2 f = fract(x);
            float a = hash(i); float b = hash(i + vec2(1.0, 0.0));
            float c = hash(i + vec2(0.0, 1.0)); float d = hash(i + vec2(1.0, 1.0));
            vec2 u = f * f * (3.0 - 2.0 * f);
            return mix(a, b, u.x) + (c - a) * u.y * (1.0 - u.x) + (d - b) * u.x * u.y;
        }
        
        float fbm(vec2 x, int oct) {
            float v = 0.0; float a = 0.5;
            vec2 shift = vec2(100.0);
            mat2 rot = rot2d(0.5);
            for (int i = 0; i < 8; i++) {
                if(i >= oct) break;
                v += a * noise(x); x = rot * x * 2.0 + shift; a *= 0.5;
            }
            return v;
        }

        vec3 getPalette(float t, float pVal, float bass, float mid) {
            vec3 a = vec3(0.3, 0.3, 0.3);
            vec3 b = vec3(0.3, 0.3, 0.3); 
            vec3 c = vec3(1.0, 1.0, 1.0);
            vec3 d = vec3(0.0, 0.33, 0.67) + (pVal * vec3(0.5, 0.1, 0.0)); 
            
            // Slow down the color cycling significantly
            vec3 col = a + b * cos( 6.28318 * (c * t * 0.5 + d) );
            vec3 col2 = a + b * cos( 6.28318 * (c * t * 0.7 + d + vec3(0.2, 0.0, -0.1)) );
            
            // Mixing is more subtle
            return mix(col, col2, mid * 0.3);
        }

        float particle(vec2 uv, vec2 offset, float scale, float n, float z_pos) {
            vec2 pos = uv - offset;
            float d = length(pos);
            // Flicker is independent of audio to stay smooth
            float flicker = hash(offset + u_time * 0.2) * 0.3 + 0.7; 
            
            float size_scale = 1.0 / (1.0 + abs(z_pos * 0.5)); 
            return 0.05 / (d + 0.001) * smoothstep(scale * size_scale, scale * size_scale * 0.1, d) * flicker; 
        }

        void main() {
            vec2 uv = vTexCoord;
            uv.x *= u_res.x / u_res.y;
            
            // Rotation is now very slow and continuous
            vec2 centered = uv - 0.5;
            centered *= rot2d(u_rot * 0.05); 
            uv = centered + 0.5;

            // Fluid
            float speed = u_params.x * 0.2; 
            // Dampen bass impact on fluid distortion to prevent jerking
            float bassImpact = u_bass * u_react * 2.0; 
            float warpStrength = 1.0 + u_params.x * 0.5 + bassImpact; 
            
            vec2 q = vec2(fbm(uv + u_time * speed, 4), fbm(uv + vec2(5.2, 1.3) + u_time * speed, 4));
            vec2 r = vec2(fbm(uv + 4.0 * q + vec2(1.7, 9.2) + u_time * speed + u_mid*0.2, 4), 
                          fbm(uv + 4.0 * q + vec2(8.3, 2.8) + u_time * speed - u_mid*0.2, 4));
                                  
            float liquid = fbm(uv + r * warpStrength, 6);
            liquid = pow(smoothstep(0.1, 0.9, liquid), 1.5);

            // Grid
            float gridScale = 4.0 + u_params.y * 35.0; 
            vec2 gridUV = (uv + r * 0.05) * gridScale * (1.0 + liquid * 0.05 * u_params.y);
            
            float thick = 0.48; 
            float gridLine = smoothstep(thick, 0.5, abs(fract(gridUV.x + u_time*0.02) - 0.5));
            gridLine += smoothstep(thick, 0.5, abs(fract(gridUV.y - u_time*0.01) - 0.5));
            float structure = gridLine * min(u_params.y, 0.8) * (1.0 + bassImpact * 0.3); 

            // Particles
            float particles = 0.0;
            float particleGridScale = 8.0 + u_params.z * 15.0;
            vec2 pUV = uv * particleGridScale;
            
            // Flow Field for Particles
            float global_flow_speed = 0.05 + u_vol * 0.1;
            vec2 global_flow_offset = vec2(u_time * global_flow_speed, u_time * global_flow_speed * 0.5);
            vec2 offset_pID = floor(pUV - global_flow_offset); 

            pUV = fract(pUV) - 0.5;

            for(int y=-1; y<=1; y++) {
                for(int x=-1; x<=1; x++) {
                    vec2 neighbor = vec2(float(x), float(y));
                    vec2 id = offset_pID + neighbor;
                    float n = hash(id);
                    
                    float z_offset = hash(id + vec2(100.0));
                    float z_pos = 2.0 * z_offset - 1.0; 
                    
                    // SMOOTH MOVEMENT:
                    // Removed u_treble from position calculation to prevent teleporting
                    float moveSpeed = u_time * (0.3 + n * 0.2); 
                    float posX = sin(moveSpeed + n * 6.28) * 0.3;
                    float posY = cos(moveSpeed + n * 6.28) * 0.3;
                    
                    float perspective_scale = 1.0 / (1.0 + z_pos * 0.5); 
                    vec2 pOffset = (neighbor + vec2(posX, posY) - fract(global_flow_offset) ) * perspective_scale; 
                    
                    // Audio mainly affects SIZE and BRIGHTNESS, not position
                    float pSize = (0.08 + u_params.z * 0.15) * (1.0 + u_treble * u_react * 2.5) * n;
                    particles += particle(pUV, pOffset, pSize, n, z_pos);
                }
            }
            particles *= min(u_params.z * 1.5, 1.0); 

            // Mix
            // Use smoothed audio for color mixing to prevent flashing
            vec3 col = getPalette(liquid + u_bass * 0.1, u_paletteVal, u_bass, u_mid);
            
            vec3 structCol = vec3(0.0, 0.8, 1.0) * (0.5 + u_bass*0.3) * (1.0 + u_paletteVal); 
            col = mix(col, structCol * 1.0, structure * 0.7); 
            
            col += vec3(particles * 1.2) * structCol; 

            // Post Processing
            vec3 finalCol = col;
            float aber = u_vol * u_react * 0.015; // Subtle aberration
            finalCol.r = mix(col.r, col.r * 1.1, aber);
            
            float satFactor = smoothstep(0.0, 0.2, u_paletteVal);
            float gray = dot(finalCol, vec3(0.299, 0.587, 0.114));
            float noirContrast = 1.0 + (1.0 - satFactor) * 0.5; 
            vec3 noirGray = vec3(pow(gray, noirContrast));
            finalCol = mix(noirGray, finalCol, satFactor);

            float vig = 1.0 - smoothstep(0.3, 1.8, length(vTexCoord - 0.5) * 1.5);
            finalCol *= vig;
            
            // Tone Mapping
            finalCol = finalCol / (finalCol + vec3(1.2)); 
            finalCol = pow(finalCol, vec3(1.0/1.8)); 

            gl_FragColor = vec4(finalCol, 1.0);
        }
    `;

    // --- YouTube API ---
    let player;
    function onYouTubeIframeAPIReady() {
        player = new YT.Player('player', {
            height: '100%', width: '100%', videoId: 'jfKfPfyJRdk',
            playerVars: { 'playsinline': 1, 'controls': 1 }
        });
    }

    // --- Setup ---
    function setup() {
        let cnv = createCanvas(windowWidth, windowHeight, WEBGL);
        cnv.style('z-index', '1'); 
        cnv.id('defaultCanvas0');
        noStroke();
        myShader = createShader(vert, frag);
        
        mic = new p5.AudioIn();
        fft = new p5.FFT(0.8, 32); 
        amplitude = new p5.Amplitude(); 
        
        // Default mic
        fft.setInput(mic);
        amplitude.setInput(mic);
        
        const vC = document.getElementById('viz-canvas');
        vizCtx = vC.getContext('2d');

        ['param1', 'param2', 'param3', 'reactivity', 'colorshift'].forEach((id, i) => {
            const el = document.getElementById(id);
            el.addEventListener('input', (e) => {
                let val = parseFloat(e.target.value);
                if (i < 3) state.params[i] = val;
                else if (i === 3) state.global.react = val;
                else state.global.paletteVal = val; 
                e.target.previousElementSibling.querySelector('.value').innerText = val.toFixed(2);
            });
        });
    }

    function initApp() {
        userStartAudio(); 
        mic.start(); 
        state.running = true;
        const ov = document.getElementById('overlay');
        ov.style.opacity = 0;
        setTimeout(()=> ov.style.display='none', 500);
    }

    // --- Audio Source Switching ---

    function switchToMic() {
        if (soundFile && soundFile.isPlaying()) {
            soundFile.stop();
            document.getElementById('play-btn').style.display = "none";
        }
        mic.start();
        fft.setInput(mic);
        amplitude.setInput(mic);
        document.getElementById('file-status').innerText = "MIC ACTIVE";
        if(!state.syncMode) toggleSync();
    }
    
    function handleFileSelect(fileInput) {
        if (fileInput.files.length === 0) return;
        const file = fileInput.files[0];
        const statusEl = document.getElementById('file-status');
        
        if (file.size > MAX_FILE_SIZE) {
            statusEl.innerText = "ERROR: FILE TOO LARGE (>20MB)";
            alert("ファイルがデカすぎます（20MBまで）。別のファイルを選んでください。");
            return;
        }

        statusEl.innerText = "LOADING...";
        if(soundFile && soundFile.isPlaying()) soundFile.stop();
        if(player && player.pauseVideo) player.pauseVideo();

        const blobUrl = URL.createObjectURL(file);
        soundFile = loadSound(blobUrl, () => {
            statusEl.innerText = file.name.toUpperCase();
            document.getElementById('play-btn').style.display = "flex";
            mic.stop();
            soundFile.play();
            soundFile.setLoop(true);
            fft.setInput(soundFile);
            amplitude.setInput(soundFile);
            if(!state.syncMode) toggleSync();
            URL.revokeObjectURL(blobUrl);
        }, (err) => {
            console.error(err);
            statusEl.innerText = "ERROR LOADING FILE";
            alert("読み込みエラー: このファイル形式は利用できません。");
        });
    }

    function togglePlay() {
        if(soundFile && soundFile.isLoaded()) {
            if(soundFile.isPlaying()) {
                soundFile.pause();
                document.getElementById('play-btn').innerText = "▶";
            } else {
                soundFile.play();
                document.getElementById('play-btn').innerText = "❚❚";
            }
        }
    }

    let rotAccumulator = 0;

    function draw() {
        if (!state.running && millis() < 1000) return;

        let spectrum = fft.analyze();
        let bass = fft.getEnergy("bass") / 255.0;      
        let mid = fft.getEnergy("mid") / 255.0;
        let treble = fft.getEnergy("treble") / 255.0; // Raw value
        let vol = amplitude.getLevel();

        if (!state.uiHidden) {
            vizCtx.clearRect(0, 0, 230, 40);
            vizCtx.fillStyle = state.syncMode ? 'rgba(0, 255, 255, 0.5)' : 'rgba(255, 255, 255, 0.2)';
            let barW = 230 / spectrum.length;
            for (let i = 0; i < spectrum.length; i++) {
                let h = (spectrum[i] / 255) * 38 * (0.5 + vol * 1.5); 
                vizCtx.fillRect(i * barW, 40 - h, barW - 1, h);
            }
        }

        // --- AUDIO SMOOTHING (The Key to Seamlessness) ---
        // Lerp factor 0.1 means "move 10% towards the target". 
        // Smaller number = smoother/slower reaction.
        let smoothFactor = 0.08; 
        state.avgBass = lerp(state.avgBass, bass, smoothFactor);
        state.avgMid = lerp(state.avgMid, mid, smoothFactor);
        state.avgTreble = lerp(state.avgTreble, treble, smoothFactor);
        state.avgVol = lerp(state.avgVol, vol, smoothFactor);

        let targetP = [0, 0, 0];
        let targetPal = state.global.paletteVal;

        if (state.syncMode) {
            targetP[0] = state.params[0] + (state.avgVol * 2.0); 
            targetP[1] = state.params[1] + (state.avgBass * 2.0 * state.global.react); 
            targetP[2] = state.params[2] + (state.avgTreble * 2.0 * state.global.react);
            
            if (state.global.paletteVal > 0.15) {
                targetPal = state.global.paletteVal + (state.avgMid * 0.2 * state.global.react) + (state.avgBass * 0.05);
            }
            rotAccumulator += (0.0005 + state.avgVol * 0.05); 
        } else {
            targetP = [...state.params];
            rotAccumulator += 0.0005;
        }

        for(let i=0; i<3; i++) {
            state.dynamicParams[i] = lerp(state.dynamicParams[i], targetP[i], 0.05);
            state.dynamicParams[i] = constrain(state.dynamicParams[i], 0.0, 5.0); 
        }
        // Palette lerp is even slower for drifting colors
        state.global.dynPalette = lerp(state.global.dynPalette, targetPal, 0.02);
        
        shader(myShader);
        myShader.setUniform('u_res', [width, height]); 
        myShader.setUniform('u_time', millis() / 1000.0);
        myShader.setUniform('u_params', state.dynamicParams);
        myShader.setUniform('u_react', state.global.react); 
        myShader.setUniform('u_paletteVal', state.global.dynPalette);
        myShader.setUniform('u_rot', rotAccumulator);
        
        // Pass the SMOOTHED values to the shader
        myShader.setUniform('u_bass', state.avgBass); 
        myShader.setUniform('u_mid', state.avgMid);
        myShader.setUniform('u_treble', state.avgTreble);
        myShader.setUniform('u_vol', state.avgVol);
        
        rect(-width/2, -height/2, width, height);
    }

    window.toggleUI = () => {
        state.uiHidden = !state.uiHidden;
        document.getElementById('ctrl-panel').classList.toggle('hidden', state.uiHidden);
        document.getElementById('btm-bar').classList.toggle('hidden', state.uiHidden);
        document.querySelector('.header').style.opacity = state.uiHidden ? '0' : '1';
        document.querySelector('.toggle-btn-fixed').innerText = state.uiHidden ? '☰' : '✕';
    };
    
    window.toggleSync = () => {
        state.syncMode = !state.syncMode;
        const btn = document.getElementById('sync-btn');
        const lbl = btn.querySelector('.switch-label');
        if(state.syncMode) {
            btn.classList.add('active'); 
            lbl.innerText = "AUDIO SYNC: ON";
            document.querySelectorAll('.slider-group').forEach(el => el.classList.add('sync-active'));
        } else {
            btn.classList.remove('active'); 
            lbl.innerText = "AUDIO SYNC: OFF";
            document.querySelectorAll('.slider-group').forEach(el => el.classList.remove('sync-active'));
        }
    };
    
    function windowResized() { resizeCanvas(windowWidth, windowHeight); }
</script>
</body>
</html>
