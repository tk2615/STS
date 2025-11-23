<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Aether VJ: White & Rainbow Control</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.6.0/p5.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.6.0/addons/p5.sound.min.js"></script>
    <script src="https://www.youtube.com/iframe_api"></script>
    <style>
        /* --- Base Styling --- */
        body { margin: 0; padding: 0; background: #000; overflow: hidden; font-family: 'Helvetica Neue', sans-serif; color: #ccc; }
        * { box-sizing: border-box; outline: none; -webkit-tap-highlight-color: transparent; }
        canvas { display: block; position: fixed; top: 0; left: 0; z-index: 1; pointer-events: none; }

        /* --- UI Layer --- */
        #ui-container {
            position: fixed; inset: 0; pointer-events: none; z-index: 100;
            display: flex; flex-direction: column; justify-content: space-between;
            padding: 25px;
        }

        .header { display: flex; align-items: flex-start; justify-content: space-between; pointer-events: auto; }
        .title-group h1 { margin: 0; font-size: 11px; letter-spacing: 5px; opacity: 0.8; text-transform: uppercase; color: #fff; text-shadow: 0 0 10px rgba(255,255,255,0.5); }
        
        /* Sync Switch */
        .sync-switch {
            display: inline-flex; align-items: center; gap: 8px;
            background: rgba(255,255,255,0.05); padding: 6px 14px; border-radius: 20px; border: 1px solid rgba(255,255,255,0.1);
            cursor: pointer; transition: all 0.2s; backdrop-filter: blur(10px); pointer-events: auto;
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
            border: 1px solid rgba(255,255,255,0.1); border-radius: 4px;
            padding: 20px; width: 280px; align-self: flex-end;
            margin-bottom: 160px; 
            transition: transform 0.6s cubic-bezier(0.16, 1, 0.3, 1), opacity 0.6s;
            box-shadow: 0 20px 50px rgba(0,0,0,0.8);
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
        .slider-group { margin-bottom: 18px; position: relative; }
        .label-row { display: flex; justify-content: space-between; margin-bottom: 6px; }
        label { font-size: 9px; color: #888; text-transform: uppercase; letter-spacing: 1px; font-weight: 500; }
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
        input[type=range]::-webkit-slider-runnable-track { width: 100%; height: 2px; background: rgba(255,255,255,0.2); border-radius: 2px;}
        input[type=range]::-webkit-slider-thumb {
            -webkit-appearance: none; height: 14px; width: 14px; border-radius: 50%;
            background: #111; margin-top: -6px; box-shadow: 0 0 0 1px rgba(255,255,255,0.8); transition: transform 0.1s; 
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
            background: rgba(10,10,10,0.95); border-top: 1px solid rgba(255,255,255,0.1);
            z-index: 101; display: flex; flex-direction: column; pointer-events: auto;
            transition: transform 0.6s cubic-bezier(0.16, 1, 0.3, 1); backdrop-filter: blur(20px);
            padding-bottom: env(safe-area-inset-bottom);
        }
        .bottom-bar.hidden { transform: translateY(120%); }

        /* YT Input Row */
        .yt-input-row {
            padding: 10px 20px; border-bottom: 1px solid rgba(255,255,255,0.05);
            display: flex; gap: 10px; align-items: center;
        }
        .yt-input {
            flex-grow: 1; background: rgba(0,0,0,0.3); border: 1px solid rgba(255,255,255,0.1);
            color: #fff; padding: 8px 12px; border-radius: 4px; font-size: 11px; font-family: monospace;
        }
        .yt-input::placeholder { color: #555; }
        .yt-load-btn {
            background: #cc0000; color: #fff; border: none; padding: 8px 15px;
            border-radius: 4px; font-size: 10px; font-weight: bold; cursor: pointer; letter-spacing: 1px; white-space: nowrap;
        }
        .yt-load-btn:hover { background: #ff0000; }

        /* File Controls */
        .file-controls {
            padding: 15px 20px; display: flex; align-items: center; gap: 15px;
        }
        .custom-file-btn {
            background: #222; border: 1px solid rgba(255,255,255,0.2); color: #fff;
            padding: 8px 16px; border-radius: 20px; font-size: 10px; letter-spacing: 1px;
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
        .mic-mode-btn { background: #004400; border-color: #00ff00; color: #00ff00; }

        /* YouTube Hidden Player */
        .youtube-area {
            position: absolute; top: 0; left: 0; width: 1px; height: 1px; opacity: 0; pointer-events: none;
        }

        /* Overlay */
        #overlay {
            position: fixed; inset: 0; background: #000; z-index: 9999;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            cursor: pointer; color: #fff; transition: opacity 0.5s ease-out;
            pointer-events: auto;
        }
        .start-text { font-size: 14px; letter-spacing: 8px; font-weight: 300; margin-bottom: 20px; opacity: 0.9; text-align: center; pointer-events: none; }
        .sub-text { font-size: 10px; color: #0ff; letter-spacing: 2px; text-transform: uppercase; background: rgba(0,255,255,0.1); padding: 10px 20px; border-radius: 30px; border: 1px solid rgba(0,255,255,0.3); pointer-events: none; }

        @media (max-width: 600px) {
            .controls-panel { width: 100%; margin-bottom: 160px; border-radius: 0; border:none; border-top: 1px solid rgba(255,255,255,0.1);}
        }
    </style>
</head>
<body>

<div id="overlay" onclick="initApp()">
    <div class="start-text">TAP TO START</div>
    <div class="sub-text">VISUALIZER CORE</div>
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
            <div class="label-row"><label>Flow (Speed)</label> <span class="value" id="val-1">0.5</span></div>
            <input type="range" id="param1" min="0" max="1" step="0.01" value="0.5">
        </div>
        
        <div class="slider-group" id="grp-2">
            <div class="sync-dot"></div>
            <div class="label-row"><label>Grid (Structure)</label> <span class="value" id="val-2">0.3</span></div>
            <input type="range" id="param2" min="0" max="1" step="0.01" value="0.3">
        </div>
        
        <div class="slider-group" id="grp-3">
            <div class="sync-dot"></div>
            <div class="label-row"><label>Float (Particles)</label> <span class="value" id="val-3">0.4</span></div>
            <input type="range" id="param3" min="0" max="1" step="0.01" value="0.4">
        </div>

        <hr style="border: 0; border-top: 1px solid rgba(255,255,255,0.08); margin: 20px 0;">
        
        <div class="slider-group">
            <div class="label-row"><label>Reactivity</label> <span class="value" id="val-react">1.5</span></div>
            <input type="range" id="reactivity" min="0" max="3" step="0.1" value="1.5">
        </div>
        
        <div class="slider-group" id="grp-color">
            <div class="sync-dot"></div>
            <div class="label-row"><label>Color (0=White, 1=Rainbow)</label> <span class="value" id="val-color">0.2</span></div>
            <input type="range" id="colorshift" min="0" max="1" step="0.01" value="0.2">
        </div>
    </div>
</div>

<div class="bottom-bar" id="btm-bar">
    <div class="yt-input-row">
        <input type="text" id="yt-url" class="yt-input" placeholder="YouTube URL (ex: https://youtu.be/...)">
        <button class="yt-load-btn" onclick="loadYoutube()">LOAD YT</button>
    </div>

    <div class="file-controls">
        <button class="custom-file-btn mic-mode-btn" onclick="switchToMic()">🎤 MIC</button>
        <label for="audio-upload" class="custom-file-btn">📂 FILE</label>
        <input type="file" id="audio-upload" accept=".mp3,audio/*" style="display:none" onchange="handleFileSelect(this)">
        
        <div id="file-status">MIC MODE</div>
        <button class="play-pause-btn" id="play-btn" onclick="togglePlay()">❚❚</button>
    </div>

    <div class="youtube-area">
        <div id="player"></div>
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
        avgBass: 0, avgMid: 0, avgTreble: 0, avgVol: 0
    };

    let myShader, fft, amplitude, mic;
    let soundFile = null; 
    let vizCtx; 

    const vert = `
        attribute vec3 aPosition; attribute vec2 aTexCoord; varying vec2 vTexCoord;
        void main() { vTexCoord = aTexCoord; gl_Position = vec4(aPosition * 2.0 - 1.0, 1.0); }
    `;

    // --- SHADER: WHITE to RAINBOW CONTROL ---
    const frag = `
        precision mediump float;
        varying vec2 vTexCoord;
        uniform float u_time; uniform vec2 u_res;
        uniform vec3 u_params; 
        uniform float u_react; uniform float u_paletteVal; 
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
            for (int i = 0; i < 4; i++) { 
                if(i >= oct) break;
                v += a * noise(x); x = rot * x * 2.0 + shift; a *= 0.5;
            }
            return v;
        }

        // Rainbow Generator
        vec3 getRainbow(float t) {
            vec3 a = vec3(0.5, 0.5, 0.5);
            vec3 b = vec3(0.5, 0.5, 0.5); 
            vec3 c = vec3(1.0, 1.0, 1.0);
            vec3 d = vec3(0.0, 0.33, 0.67) + u_time * 0.1; 
            return a + b * cos( 6.28318 * (c * t + d) );
        }

        float particle(vec2 uv, vec2 offset, float scale, float z_pos) {
            vec2 pos = uv - offset;
            float d = length(pos);
            float size_scale = 1.0 / (1.0 + abs(z_pos * 0.8));
            return 0.06 / (d + 0.005) * smoothstep(scale * size_scale, 0.0, d);
        }

        void main() {
            vec2 uv = vTexCoord;
            uv.x *= u_res.x / u_res.y;
            
            vec2 centered = uv - 0.5;
            centered *= rot2d(u_rot * 0.05); 
            uv = centered + 0.5;

            // Fluid
            float speed = u_params.x * 0.2; 
            float warpStrength = 1.0 + u_params.x * 0.5 + u_bass * u_react; 
            
            vec2 q = vec2(fbm(uv + u_time * speed, 2), fbm(uv + vec2(5.2, 1.3) + u_time * speed, 2));
            vec2 r = vec2(fbm(uv + 2.0 * q + vec2(1.7, 9.2) + u_time * speed + u_mid*0.2, 2), 
                          fbm(uv + 2.0 * q + vec2(8.3, 2.8) + u_time * speed - u_mid*0.2, 2));         
            float liquid = fbm(uv + r * warpStrength, 3);

            // Grid
            float gridScale = 4.0 + u_params.y * 30.0; 
            vec2 gridUV = (uv + r * 0.05) * gridScale;
            float gridLine = smoothstep(0.48, 0.5, abs(fract(gridUV.x + u_time*0.02) - 0.5));
            gridLine += smoothstep(0.48, 0.5, abs(fract(gridUV.y - u_time*0.01) - 0.5));
            float structure = gridLine * min(u_params.y, 0.8) * (1.0 + u_bass * 0.3); 

            // Particles
            float particles = 0.0;
            vec3 particleColorAccum = vec3(0.0);
            
            float particleGridScale = 6.0 + u_params.z * 10.0;
            vec2 pUV = uv * particleGridScale;
            vec2 id = floor(pUV);
            pUV = fract(pUV) - 0.5;

            for(int y=-1; y<=1; y++) {
                for(int x=-1; x<=1; x++) {
                    vec2 neighbor = vec2(float(x), float(y));
                    vec2 neighborID = id + neighbor;
                    float n = hash(neighborID); 
                    
                    float t = u_time * (0.2 + n * 0.2); 
                    float x_freq = 2.0 + n * 3.0;
                    float y_freq = 2.0 + fract(n * 10.0) * 3.0;
                    
                    float posX = sin(t * x_freq + n * 10.0) * (0.6 + u_treble * u_react * 0.5);
                    float posY = cos(t * y_freq + n * 20.0) * (0.6 + u_treble * u_react * 0.5);
                    float z_pos = sin(t * 1.5 + n * 5.0) * 1.5; 
                    
                    vec2 pOffset = neighbor + vec2(posX, posY);
                    float pSize = (0.1 + u_params.z * 0.2) * (0.5 + n) * (1.0 + u_treble * u_react);
                    float fade = smoothstep(2.0, -1.0, abs(z_pos));
                    
                    float pIntensity = particle(pUV, pOffset, pSize, z_pos) * fade;
                    
                    // --- COLOR LOGIC: WHITE <-> RAINBOW MIX ---
                    // Generate rainbow color for this particle
                    vec3 rainbowCol = getRainbow(n + u_time * 0.2 + u_bass * 0.1); 
                    
                    // Pure White color
                    vec3 whiteCol = vec3(0.9, 0.95, 1.0);
                    
                    // Mix based on u_paletteVal (Slider)
                    // 0.0 = White, 1.0 = Rainbow
                    vec3 pCol = mix(whiteCol, rainbowCol, u_paletteVal);
                    
                    particleColorAccum += pIntensity * pCol; 
                    particles += pIntensity;
                }
            }
            particles *= min(u_params.z * 2.0, 1.2); 

            // Mix Final Color
            // Background is subtle rainbow or white
            vec3 bgRainbow = getRainbow(liquid * 0.5 + u_time * 0.05);
            vec3 bgWhite = vec3(0.1);
            vec3 bgCol = mix(bgWhite, bgRainbow, u_paletteVal) * (0.2 + liquid * 0.3);
            
            // Grid Color
            vec3 gridCol = mix(vec3(0.3), vec3(0.0, 1.0, 1.0), u_paletteVal) * structure;

            vec3 finalColor = bgCol + gridCol + particleColorAccum;
            
            // Boost brightness on volume
            finalColor += vec3(u_vol * u_react * 0.3); 

            // Tone Mapping
            finalColor = finalColor / (finalColor + vec3(1.0)); 
            finalColor = pow(finalColor, vec3(0.8)); 

            gl_FragColor = vec4(finalColor, 1.0);
        }
    `;

    // --- YouTube API ---
    let player;
    function onYouTubeIframeAPIReady() {
        player = new YT.Player('player', {
            height: '100%', width: '100%', videoId: '', 
            playerVars: { 'playsinline': 1, 'controls': 1, 'autoplay': 1 }, 
            events: { 'onError': onPlayerError }
        });
    }

    function onPlayerError(event) {
        console.error("YT Error:", event.data);
        alert("YouTube再生エラー: 動画が再生できません。制限付き動画か、IDが間違っている可能性があります。");
    }

    function loadYoutube() {
        let input = document.getElementById('yt-url').value;
        const status = document.getElementById('file-status');
        
        let videoId = "";
        const regExp = /^.*((youtu.be\/)|(v\/)|(\/u\/\w\/)|(embed\/)|(watch\?))\??v?=?([^#&?]*).*/;
        const match = input.match(regExp);
        
        if (match && match[7].length == 11) {
            videoId = match[7];
        } else if (input.length === 11) {
            videoId = input;
        }

        if (videoId) {
            if (soundFile && soundFile.isPlaying()) soundFile.stop();
            mic.stop();
            status.innerText = "YT: " + videoId;
            player.loadVideoById(videoId);
            player.playVideo(); 
            alert("YouTubeをロードしました。\n【重要】\n1. マイクを許可してください（URLバーの鍵アイコン等から）。\n2. 端末のスピーカーから音を出してください。");
            switchToMic(); 
        } else {
            alert("URLが無効です。");
        }
    }

    // --- Setup ---
    function setup() {
        let cnv = createCanvas(windowWidth, windowHeight, WEBGL);
        pixelDensity(1); 
        cnv.style('z-index', '1'); 
        cnv.id('defaultCanvas0');
        noStroke();
        myShader = createShader(vert, frag);
        
        mic = new p5.AudioIn();
        fft = new p5.FFT(0.8, 32); 
        amplitude = new p5.Amplitude(); 
        
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

    function switchToMic() {
        if (soundFile && soundFile.isPlaying()) {
            soundFile.stop();
            document.getElementById('play-btn').style.display = "none";
        }
        mic.start();
        fft.setInput(mic);
        amplitude.setInput(mic);
        document.getElementById('file-status').innerText = "MIC / YT MODE";
        if(!state.syncMode) toggleSync();
    }
    
    function handleFileSelect(fileInput) {
        if (fileInput.files.length === 0) return;
        const file = fileInput.files[0];
        const statusEl = document.getElementById('file-status');
        
        if (file.size > MAX_FILE_SIZE) {
            alert("ファイルが大きすぎます (20MB制限)");
            return;
        }

        if(player && player.stopVideo) player.stopVideo();
        statusEl.innerText = "LOADING...";
        
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
            statusEl.innerText = "ERROR";
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
        let treble = fft.getEnergy("treble") / 255.0;
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

        let smoothFactor = 0.08; 
        state.avgBass = lerp(state.avgBass, bass, smoothFactor);
        state.avgMid = lerp(state.avgMid, mid, smoothFactor);
        state.avgTreble = lerp(state.avgTreble, treble, smoothFactor);
        state.avgVol = lerp(state.avgVol, vol, smoothFactor);

        let targetP = [0, 0, 0];
        
        if (state.syncMode) {
            targetP[0] = state.params[0] + (state.avgVol * 2.0); 
            targetP[1] = state.params[1] + (state.avgBass * 2.0 * state.global.react); 
            targetP[2] = state.params[2] + (state.avgTreble * 2.0 * state.global.react);
            rotAccumulator += (0.0005 + state.avgVol * 0.05); 
        } else {
            targetP = [...state.params];
            rotAccumulator += 0.0005;
        }

        for(let i=0; i<3; i++) {
            state.dynamicParams[i] = lerp(state.dynamicParams[i], targetP[i], 0.05);
            state.dynamicParams[i] = constrain(state.dynamicParams[i], 0.0, 5.0); 
        }
        
        shader(myShader);
        myShader.setUniform('u_res', [width, height]); 
        myShader.setUniform('u_time', millis() / 1000.0);
        myShader.setUniform('u_params', state.dynamicParams);
        myShader.setUniform('u_react', state.global.react); 
        myShader.setUniform('u_paletteVal', state.global.paletteVal);
        myShader.setUniform('u_rot', rotAccumulator);
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
