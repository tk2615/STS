<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Aether VJ Final Edition</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.6.0/p5.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.6.0/addons/p5.sound.min.js"></script>
    <style>
        /* --- Base Styling --- */
        body { margin: 0; padding: 0; background: #000; overflow: hidden; font-family: 'Helvetica Neue', sans-serif; color: #ccc; }
        * { box-sizing: border-box; outline: none; -webkit-tap-highlight-color: transparent; }
        
        canvas { display: block; position: fixed; top: 0; left: 0; z-index: 1; pointer-events: none; }

        /* --- YouTube Layer --- */
        #yt-wrapper {
            position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%);
            z-index: 2; pointer-events: none; 
            display: flex; align-items: center; justify-content: center;
            overflow: hidden; transition: opacity 0.3s;
        }
        #yt-wrapper.interactive { pointer-events: auto; cursor: pointer; border: 2px solid #0f0; }
        #yt-player { width: 100%; height: 100%; }

        /* --- UI Layer --- */
        #ui-container {
            position: fixed; inset: 0; pointer-events: none; z-index: 100;
            display: flex; flex-direction: column; justify-content: space-between;
            padding: 20px;
        }

        .header { 
            display: flex; align-items: flex-start; justify-content: space-between; 
            pointer-events: auto; margin-bottom: 10px; 
        }
        .title-group { display: flex; flex-direction: column; gap: 8px; }
        .title-row { display: flex; align-items: center; gap: 15px; }
        
        .reload-btn {
            background: rgba(255,255,255,0.1); border: 1px solid rgba(255,255,255,0.2);
            color: #fff; border-radius: 4px; padding: 4px 8px; cursor: pointer; font-size: 10px;
            transition: all 0.2s;
        }
        .reload-btn:hover { background: rgba(255,255,255,0.3); }

        /* Sync Switch */
        .sync-switch {
            display: inline-flex; align-items: center; gap: 8px; width: fit-content;
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

        /* Mini Switch */
        .mini-switch {
            width: 30px; height: 16px; background: #333; border-radius: 10px; position: relative; cursor: pointer; transition: background 0.2s;
            border: 1px solid rgba(255,255,255,0.2);
        }
        .mini-switch.active { background: rgba(0, 255, 255, 0.3); border-color: #0ff; }
        .mini-indicator {
            width: 12px; height: 12px; background: #888; border-radius: 50%; 
            position: absolute; top: 1px; left: 1px; transition: transform 0.2s, background 0.2s;
        }
        .mini-switch.active .mini-indicator { transform: translateX(14px); background: #0ff; box-shadow: 0 0 5px #0ff; }

        /* --- CONTROLS PANEL (GLASS) --- */
        .controls-panel {
            pointer-events: auto;
            position: absolute;
            top: 20px;
            right: 20px;
            bottom: 100px;
            width: 520px;
            
            background: rgba(10, 10, 10, 0.6); 
            backdrop-filter: blur(30px); -webkit-backdrop-filter: blur(30px);
            border: 1px solid rgba(255,255,255,0.15); border-radius: 8px;
            padding: 20px; 
            
            transition: transform 0.4s cubic-bezier(0.16, 1, 0.3, 1), opacity 0.4s;
            box-shadow: 0 10px 40px rgba(0,0,0,0.5);
            z-index: 101;
            
            overflow-y: auto;
            scrollbar-width: thin; scrollbar-color: #444 rgba(0,0,0,0.3);
            
            display: flex;
            flex-direction: column;
        }
        .controls-panel.hidden { transform: translateX(120%); opacity: 0; pointer-events: none; }
        .controls-panel::-webkit-scrollbar { width: 4px; }
        .controls-panel::-webkit-scrollbar-thumb { background: #444; border-radius: 2px; }

        .panel-grid {
            display: grid;
            grid-template-columns: 1fr 1fr; 
            gap: 25px; 
            flex: 1;
        }

        #viz-canvas {
            width: 100%; height: 30px; background: rgba(0,0,0,0.3);
            border-bottom: 1px solid rgba(255,255,255,0.1); margin-bottom: 15px;
            border-radius: 2px; display: block; flex-shrink: 0;
        }

        .section-title {
            font-size: 11px; color: #0ff; font-weight: bold; letter-spacing: 1px;
            margin: 0 0 10px 0; padding-bottom: 6px;
            border-bottom: 1px solid rgba(0,255,255,0.3);
            text-transform: uppercase; 
            display: flex; justify-content: space-between; align-items: center;
        }
        .section-spacer { margin-top: 18px; }

        .slider-group { margin-bottom: 12px; position: relative; }
        .label-row { display: flex; justify-content: space-between; margin-bottom: 4px; align-items: center; }
        label { font-size: 10px; color: #ccc; text-transform: uppercase; letter-spacing: 0.5px; font-weight: 500; text-shadow: 0 1px 2px rgba(0,0,0,0.8); display:flex; align-items:center; gap:8px; }
        .value { font-size: 10px; font-family: 'Courier New', monospace; color: #fff; text-shadow: 0 1px 2px rgba(0,0,0,0.8); }
        
        .sync-dot {
            position: absolute; right: -8px; top: 2px; width: 4px; height: 4px; background: #0ff; border-radius: 50%;
            opacity: 0; transition: opacity 0.3s; box-shadow: 0 0 8px #0ff;
        }
        .sync-active .sync-dot { opacity: 1.0; }

        input[type=range] { 
            width: 100%; -webkit-appearance: none; background: transparent; height: 14px; 
            cursor: pointer; margin: 0; touch-action: none; position: relative; z-index: 200; 
        }
        input[type=range]::-webkit-slider-runnable-track { width: 100%; height: 2px; background: rgba(255,255,255,0.3); border-radius: 2px;}
        input[type=range]::-webkit-slider-thumb {
            -webkit-appearance: none; height: 12px; width: 12px; border-radius: 50%;
            background: #eee; margin-top: -5px; box-shadow: 0 0 5px rgba(0,0,0,0.5); transition: transform 0.1s; 
        }
        input[type=range]:active::-webkit-slider-thumb { background: #fff; transform: scale(1.2); }

        select, input[type=text] {
            width: 100%; background: rgba(0,0,0,0.4); color: #0ff; border: 1px solid rgba(0,255,255,0.3);
            padding: 6px; font-size: 11px; border-radius: 4px; outline: none; cursor: pointer;
            font-family: 'Courier New', monospace; text-transform: uppercase;
        }
        select option { background: #111; color: #ccc; }
        input[type=text] { cursor: text; margin-bottom: 8px; color: #fff; }
        input[type=text]::placeholder { color: #aaa; }

        .btn-row { display: flex; gap: 8px; margin-bottom: 10px; }
        .small-btn {
            background: rgba(255,255,255,0.1); border: 1px solid rgba(255,255,255,0.2); color: #fff;
            padding: 6px 10px; border-radius: 4px; font-size: 10px; cursor: pointer; flex: 1;
            text-transform: uppercase; transition: 0.2s;
        }
        .small-btn:hover { background: rgba(255,255,255,0.2); border-color: #0ff; }

        .toggle-btn-fixed {
            position: fixed; top: 25px; right: 25px; z-index: 200; pointer-events: auto;
            width: 30px; height: 30px; border-radius: 50%; display: flex; align-items: center; justify-content: center;
            cursor: pointer; border: 1px solid rgba(255,255,255,0.1); color: #888;
            backdrop-filter: blur(10px); transition: all 0.3s; font-size: 14px; background: rgba(255,255,255,0.05);
        }
        .toggle-btn-fixed:hover { background: rgba(255,255,255,0.2); color: #fff; border-color: rgba(255,255,255,0.3); }

        /* --- BOTTOM BAR (GLASS) --- */
        .bottom-bar {
            position: fixed; bottom: 0; left: 0; right: 0;
            background: rgba(10, 10, 10, 0.6); 
            border-top: 1px solid rgba(255,255,255,0.15);
            backdrop-filter: blur(30px); -webkit-backdrop-filter: blur(30px);
            z-index: 101; display: flex; flex-direction: column; pointer-events: auto;
            transition: transform 0.6s cubic-bezier(0.16, 1, 0.3, 1);
            padding-bottom: env(safe-area-inset-bottom);
        }
        .bottom-bar.hidden { transform: translateY(120%); }

        #progress-container {
            padding: 15px 20px 0 20px; display: none; flex-direction: column; gap: 5px;
        }
        #seek-slider { width: 100%; height: 20px; cursor: pointer; }
        #seek-slider::-webkit-slider-thumb { background: #0ff; box-shadow: 0 0 10px #0ff; }
        .time-labels {
            display: flex; justify-content: space-between;
            font-family: 'Courier New', monospace; font-size: 10px; color: #0ff; text-shadow: 0 1px 2px #000;
        }

        .file-controls {
            padding: 20px; display: flex; align-items: center; gap: 15px; justify-content: center;
        }
        .custom-file-btn {
            background: rgba(30,30,30,0.8); border: 1px solid rgba(255,255,255,0.2); color: #fff;
            padding: 10px 20px; border-radius: 30px; font-size: 11px; letter-spacing: 1px;
            cursor: pointer; display: flex; align-items: center; gap: 8px; transition: all 0.3s;
            white-space: nowrap;
        }
        .custom-file-btn:active { transform: scale(0.95); background: #fff; color: #000; }
        #file-status { font-size: 11px; color: #0ff; font-family: 'Courier New', monospace; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; max-width: 150px; text-align: center; text-shadow: 0 0 5px rgba(0,255,255,0.5); }
        .play-pause-btn {
            width: 40px; height: 40px; border-radius: 50%; background: #0ff; border: none;
            color: #000; display: flex; align-items: center; justify-content: center; cursor: pointer;
            display: none; font-weight: bold; font-size: 16px; box-shadow: 0 0 15px rgba(0,255,255,0.4);
        }
        .mic-mode-btn { background: rgba(0, 68, 0, 0.8); border-color: #00ff00; color: #00ff00; }
        .img-mode-btn { background: rgba(0, 0, 68, 0.8); border-color: #4444ff; color: #aaaaff; }

        #overlay {
            position: fixed; inset: 0; background: #000; z-index: 9999;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            cursor: pointer; color: #fff; transition: opacity 0.5s ease-out;
            pointer-events: auto;
        }
        .start-text { font-size: 24px; letter-spacing: 8px; font-weight: 900; margin-bottom: 20px; opacity: 1.0; text-align: center; pointer-events: none; color: #fff; text-shadow: 0 0 20px rgba(0,255,255,0.8); }
        .sub-text { font-size: 14px; color: #0ff; letter-spacing: 4px; text-transform: uppercase; background: rgba(0,255,255,0.1); padding: 15px 40px; border-radius: 40px; border: 1px solid rgba(0,255,255,0.5); pointer-events: none; font-weight: bold; box-shadow: 0 0 20px rgba(0,255,255,0.2); }

        .key-tip {
            position: absolute; bottom: 80px; width: 100%; text-align: center; 
            font-size: 9px; color: rgba(255,255,255,0.5); letter-spacing: 1px; pointer-events: none;
            transition: opacity 0.5s; text-shadow: 0 1px 2px #000;
        }
        .bottom-bar.hidden ~ .key-tip { opacity: 0; }

        @media (max-width: 750px) {
            .controls-panel { width: 100%; top:auto; right:0; left:0; bottom: 120px; border-radius: 0; border:none; border-top: 1px solid rgba(255,255,255,0.1); height: 60vh; background: rgba(10,10,10,0.8); }
            .panel-grid { grid-template-columns: 1fr; gap: 0; }
            .section-spacer { margin-top: 20px; }
            .header { position: relative; top: 0; left: 0; margin-bottom: 10px; }
        }
    </style>
</head>
<body>

<div id="overlay" onclick="initApp()">
    <div class="start-text">AETHER VJ</div>
    <div class="sub-text">START</div>
</div>

<div class="toggle-btn-fixed" onclick="toggleUI()">✕</div>

<div id="yt-wrapper">
    <div id="yt-player"></div>
</div>

<div id="ui-container">
    <div class="header">
        <div class="title-group">
            <div class="title-row">
                <h1>Aether VJ</h1>
                <button class="reload-btn" onclick="reloadPage()">↻ RELOAD</button>
            </div>
            <div class="sync-switch" id="sync-btn" onclick="toggleSync()">
                <div class="indicator"></div>
                <div class="switch-label">AUDIO SYNC: OFF</div>
            </div>
        </div>
    </div>

    <div class="controls-panel" id="ctrl-panel">
        <canvas id="viz-canvas" width="480" height="30"></canvas>

        <div class="panel-grid">
            <div class="col-left">
                <div class="section-title">GLOBAL</div>
                <div class="slider-group" id="grp-1">
                    <div class="sync-dot"></div>
                    <div class="label-row"><label>Flow (Speed)</label> <span class="value" id="val-1">0.5</span></div>
                    <input type="range" id="param1" min="0" max="1" step="0.01" value="0.5">
                </div>
                <div class="slider-group">
                    <div class="label-row"><label>Reactivity (Audio)</label> <span class="value" id="val-react">1.5</span></div>
                    <input type="range" id="reactivity" min="0" max="3" step="0.1" value="1.5">
                </div>
                <div class="slider-group" style="margin-bottom: 12px;">
                    <div class="label-row"><label id="paletteLabel">Color Palette</label></div>
                    <select id="paletteSelect">
                        <option value="0">Rainbow (Classic)</option>
                        <option value="1">Cyberpunk (Pk/Cy)</option>
                        <option value="2">Heatwave (Rd/Or)</option>
                        <option value="3">Deep Ocean (Bl/Aq)</option>
                        <option value="4">Forest Neon (Gr/Li)</option>
                    </select>
                </div>
                <div class="slider-group" id="grp-saturation">
                    <div class="sync-dot"></div>
                    <div class="label-row"><label>Saturation</label> <span class="value" id="val-saturation">0.8</span></div>
                    <input type="range" id="saturationSlider" min="0" max="1" step="0.01" value="0.8">
                </div>

                <div class="section-title section-spacer">
                    GRID SYSTEM
                    <div class="mini-switch active" id="grid-toggle-btn" onclick="toggleGrid()">
                        <div class="mini-indicator"></div>
                    </div>
                </div>
                <div class="slider-group" id="grp-grid-size">
                    <div class="sync-dot"></div>
                    <div class="label-row"><label>Grid Zoom</label> <span class="value" id="val-grid-size">0.5</span></div>
                    <input type="range" id="gridSize" min="0" max="1" step="0.01" value="0.5">
                </div>
                <div class="slider-group" id="grp-grid-line">
                    <div class="sync-dot"></div>
                    <div class="label-row"><label>Line Weight</label> <span class="value" id="val-grid-line">0.3</span></div>
                    <input type="range" id="gridLine" min="0" max="1" step="0.01" value="0.3">
                </div>
                <div class="slider-group" id="grp-2">
                    <div class="sync-dot"></div>
                    <div class="label-row"><label>Distortion</label> <span class="value" id="val-2">0.3</span></div>
                    <input type="range" id="param2" min="0" max="1" step="0.01" value="0.3">
                </div>
                <div class="slider-group" id="grp-3">
                    <div class="sync-dot"></div>
                    <div class="label-row">
                        <label>Particles
                            <div class="mini-switch active" id="part-toggle-btn" onclick="toggleParticles()">
                                <div class="mini-indicator" style="width:8px; height:8px; top:2px; left:2px;"></div>
                            </div>
                        </label> 
                        <span class="value" id="val-3">0.4</span>
                    </div>
                    <input type="range" id="param3" min="0" max="1" step="0.01" value="0.4">
                </div>
            </div>

            <div class="col-right">
                <div class="section-title">
                    BORDER FX
                    <div class="mini-switch active" id="border-toggle-btn" onclick="toggleBorder()">
                        <div class="mini-indicator"></div>
                    </div>
                </div>
                <div class="slider-group" id="grp-border-width">
                    <div class="sync-dot"></div>
                    <div class="label-row"><label>Width</label> <span class="value" id="val-border-width">0.0</span></div>
                    <input type="range" id="borderWidth" min="0" max="0.5" step="0.01" value="0.0">
                </div>
                <div class="slider-group" id="grp-border-density">
                    <div class="sync-dot"></div>
                    <div class="label-row"><label>Density</label> <span class="value" id="val-border-density">5.0</span></div>
                    <input type="range" id="borderDensity" min="1" max="20" step="0.1" value="5.0">
                </div>
                <div class="slider-group" id="grp-border-distortion">
                    <div class="sync-dot"></div>
                    <div class="label-row"><label>Distortion</label> <span class="value" id="val-border-distortion">0.5</span></div>
                    <input type="range" id="borderDistortion" min="0" max="1" step="0.01" value="0.5">
                </div>
                <div class="slider-group" id="grp-border-alpha">
                    <div class="sync-dot"></div>
                    <div class="label-row"><label>Alpha</label> <span class="value" id="val-border-alpha">0.8</span></div>
                    <input type="range" id="borderAlpha" min="0" max="1" step="0.01" value="0.8">
                </div>

                <div class="section-title section-spacer">LAYERS (YT & IMG)</div>
                
                <div class="slider-group" style="margin-bottom:8px;">
                    <input type="text" id="yt-url" placeholder="YouTube ID/URL">
                    <div class="btn-row">
                        <button class="small-btn" id="yt-load-btn" onclick="loadYouTube()">LOAD & PLAY</button>
                        <button class="small-btn" onclick="clearYouTube()">CLEAR</button>
                    </div>
                    <div style="display:flex; justify-content: space-between; gap: 10px;">
                        <div style="display:flex; justify-content: space-between; align-items: center; background:rgba(255,255,255,0.05); padding:4px 8px; border-radius:4px; flex:1;">
                            <label style="margin:0; font-size:9px;">INTERACT</label>
                            <div class="mini-switch" id="yt-interact-btn" onclick="toggleYTInteract()" title="Click Interact">
                                <div class="mini-indicator"></div>
                            </div>
                        </div>
                        <div style="display:flex; justify-content: space-between; align-items: center; background:rgba(255,255,255,0.05); padding:4px 8px; border-radius:4px; flex:1;">
                            <label style="margin:0; font-size:9px;">MUTE</label>
                            <div class="mini-switch" id="yt-mute-btn" onclick="toggleYTMute()" title="Mute Toggle">
                                <div class="mini-indicator"></div>
                            </div>
                        </div>
                    </div>
                </div>
                
                <div style="display:grid; grid-template-columns: 1fr 1fr; gap:10px;">
                    <div class="slider-group">
                        <div class="label-row"><label>YT Size</label> <span class="value" id="val-yt-size">0.5</span></div>
                        <input type="range" id="ytSize" min="0.1" max="1.5" step="0.01" value="0.5">
                    </div>
                    <div class="slider-group">
                        <div class="label-row"><label>YT Opacity</label> <span class="value" id="val-yt-opacity">0.5</span></div>
                        <input type="range" id="ytOpacity" min="0" max="1" step="0.01" value="0.5">
                    </div>
                </div>
                <div class="slider-group">
                    <div class="label-row"><label>YT Pos X / Y</label></div>
                    <div style="display:flex; gap:10px;">
                        <input type="range" id="ytXParam" min="-1" max="1" step="0.01" value="0.0">
                        <input type="range" id="ytYParam" min="-1" max="1" step="0.01" value="0.0">
                    </div>
                </div>
                <div class="slider-group">
                    <select id="ytBlend" style="margin-top:5px;">
                        <option value="normal">Normal</option>
                        <option value="screen">Screen (Add)</option>
                        <option value="overlay">Overlay</option>
                        <option value="difference">Difference</option>
                        <option value="multiply">Multiply</option>
                    </select>
                </div>

                <div class="slider-group" style="display:flex; justify-content: space-between; align-items: center; margin-top:15px; border-top:1px solid rgba(255,255,255,0.1); padding-top:10px;">
                    <label style="margin:0;">IMAGE LAYER</label>
                    <div class="mini-switch active" id="img-toggle-btn" onclick="toggleImgVisibility()">
                        <div class="mini-indicator"></div>
                    </div>
                </div>
                <div style="display:grid; grid-template-columns: 1fr 1fr; gap:10px;">
                    <div class="slider-group">
                        <div class="label-row"><label>Size</label> <span class="value" id="val-img-size">0.5</span></div>
                        <input type="range" id="imgSizeParam" min="0" max="2" step="0.01" value="0.5">
                    </div>
                    <div class="slider-group">
                        <div class="label-row"><label>Opacity</label> <span class="value" id="val-img-opacity">1.0</span></div>
                        <input type="range" id="imgOpacityParam" min="0" max="1" step="0.01" value="1.0">
                    </div>
                </div>
                <div class="slider-group">
                    <div class="label-row"><label>Img Pos X / Y</label></div>
                    <div style="display:flex; gap:10px;">
                        <input type="range" id="imgXParam" min="-1" max="1" step="0.01" value="0.0">
                        <input type="range" id="imgYParam" min="-1" max="1" step="0.01" value="0.0">
                    </div>
                </div>

            </div> </div> </div>
</div>

<div class="bottom-bar" id="btm-bar">
    <div id="progress-container">
        <input type="range" id="seek-slider" min="0" max="100" value="0" step="0.1">
        <div class="time-labels">
            <span id="current-time">0:00</span>
            <span id="total-time">0:00</span>
        </div>
    </div>
    
    <div class="file-controls">
        <button class="custom-file-btn mic-mode-btn" onclick="switchToMic()">🎤 MIC</button>
        <label for="img-upload" class="custom-file-btn img-mode-btn">🖼️ IMG</label>
        <input type="file" id="img-upload" accept="image/*" style="display:none" onchange="handleImageSelect(this)">
        <div id="file-status">MIC MODE</div>
        <label for="audio-upload" class="custom-file-btn">📂 FILE</label>
        <input type="file" id="audio-upload" accept=".mp3,audio/*" style="display:none" onchange="handleFileSelect(this)">
        <button class="play-pause-btn" id="play-btn" onclick="togglePlay()">▶</button>
    </div>
</div>
<div class="key-tip">KEYS: SPACE=PLAY/PAUSE | ENTER=UI | ARROWS=FX</div>

<script>
    const MAX_FILE_SIZE = 20 * 1024 * 1024; 

    // Inject YouTube API
    var tag = document.createElement('script');
    tag.src = "https://www.youtube.com/iframe_api";
    var firstScriptTag = document.getElementsByTagName('script')[0];
    firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);

    let ytPlayer;
    let ytReady = false;

    window.onYouTubeIframeAPIReady = function() {
        ytReady = true;
        console.log("YT API Ready");
    }

    let state = {
        running: false, uiHidden: false, 
        syncMode: false,
        params: [0.5, 0.3, 0.4], 
        dynamicParams: [0.5, 0.3, 0.4],
        global: { 
            react: 1.5, gridSize: 0.5, lineWeight: 0.3, 
            saturation: 0.8, colorMode: 0, 
            rot: 0.0, imgSize: 0.5, imgOpacity: 1.0, 
            imgX: 0.0, imgY: 0.0, imgVisible: true,
            borderWidth: 0.0, borderDensity: 5.0, borderAlpha: 0.8, borderDistortion: 0.5,
            ytSize: 0.5, ytOpacity: 0.5, ytBlend: 'normal', ytInteractive: false,
            ytX: 0.0, ytY: 0.0, ytMuted: false,
            gridVisible: true, particlesVisible: true, borderVisible: true
        },
        dynamicBorder: { width: 0.0, density: 5.0, alpha: 0.8, distortion: 0.5 },
        avgBass: 0, avgMid: 0, avgTreble: 0, avgVol: 0,
        customTime: 0, drift: 0, paletteTime: 0
    };

    let myShader, fft, amplitude, mic;
    let soundFile = null;
    let centerImg = null; 
    let vizCtx; 
    let isScrubbing = false; 

    const vert = `
        attribute vec3 aPosition; attribute vec2 aTexCoord; varying vec2 vTexCoord;
        void main() { vTexCoord = aTexCoord; gl_Position = vec4(aPosition * 2.0 - 1.0, 1.0); }
    `;

    const frag = `
        precision mediump float;
        varying vec2 vTexCoord;
        uniform float u_time; uniform vec2 u_res;
        uniform vec3 u_params; 
        uniform float u_react; 
        uniform float u_saturation; 
        uniform int u_colorMode;    
        uniform float u_useAutoPalette; 
        uniform float u_paletteTime;    
        uniform float u_gridSize; 
        uniform float u_lineWeight; 
        uniform float u_drift; 
        uniform float u_bass; uniform float u_treble; uniform float u_mid; uniform float u_vol; uniform float u_rot;
        uniform vec4 u_borderParams;
        
        uniform float u_showGrid;
        uniform float u_showParticles;
        uniform float u_showBorder;

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
        vec3 getRainbow(float t, float shift) {
            vec3 a = vec3(0.5, 0.5, 0.5); vec3 b = vec3(0.5, 0.5, 0.5); 
            vec3 c = vec3(1.0, 1.0, 1.0); vec3 d = vec3(0.0, 0.33, 0.67) + shift; 
            return a + b * cos( 6.28318 * (c * t + d) );
        }
        void getThemeColors(int mode, float t, float audioBoost, out vec3 colBG, out vec3 colStruct, out vec3 colPart) {
            float beat = audioBoost * 0.5;
            if (mode == 0) {
                 colBG = getRainbow(t + beat * 0.2, u_drift * 0.1);
                 colStruct = vec3(0.2, 0.9, 1.0) + beat * 0.3; 
                 colPart = getRainbow(t + 0.5 + beat * 0.3, 0.1);
            } else if (mode == 1) {
                 colBG = mix(vec3(0.1, 0.0, 0.2), vec3(0.6, 0.0, 0.5), sin(t + beat)*0.5+0.5);
                 colStruct = vec3(0.0, 0.9, 1.0) * (1.0 + beat);
                 colPart = vec3(1.0, 0.2, 0.6) + beat * 0.5;
            } else if (mode == 2) {
                 colBG = mix(vec3(0.2, 0.0, 0.0), vec3(0.5, 0.1, 0.0), sin(t*0.7 + beat)*0.5+0.5);
                 colStruct = vec3(1.0, 0.5 + beat*0.5, 0.0); 
                 colPart = vec3(1.0, 0.8, 0.2) * (1.0 + beat*2.0);
            } else if (mode == 3) {
                 colBG = mix(vec3(0.0, 0.05, 0.2), vec3(0.0, 0.2, 0.4), sin(t*0.5 + beat*0.2)*0.5+0.5);
                 colStruct = vec3(0.0, 0.7, 0.8) * (0.8 + beat*0.4);
                 colPart = vec3(0.4, 1.0, 0.9) + beat;
            } else {
                 colBG = mix(vec3(0.0, 0.2, 0.05), vec3(0.1, 0.4, 0.1), sin(t*0.6 + beat*0.3)*0.5+0.5);
                 colStruct = vec3(0.2, 0.9, 0.3) * (1.0 + beat*0.5);
                 colPart = vec3(0.7, 1.0, 0.2) + beat;
            }
        }
        float particle(vec2 uv, vec2 offset, float scale, float z_pos) {
            vec2 pos = uv - offset;
            float d = length(pos);
            float dynamicSize = 1.0 + u_bass * u_react * 0.5;
            float size_scale = 1.0 / (1.0 + abs(z_pos * 0.8 / dynamicSize));
            return 0.06 / (d + 0.005) * smoothstep(scale * size_scale, 0.0, d);
        }
        void main() {
            vec2 uv = vTexCoord;
            uv.x *= u_res.x / u_res.y;
            vec2 centered = uv - 0.5;
            centered *= rot2d(u_rot * 0.05 + u_drift * 0.1); 
            vec2 mainUV = centered + 0.5;
            float warpStrength = 1.0 + u_params.x * 0.5 + u_bass * u_react; 
            vec2 q = vec2(fbm(mainUV + u_time * 0.5 + u_drift, 2), fbm(mainUV + vec2(5.2, 1.3) + u_time * 0.5 - u_drift, 2));
            vec2 r = vec2(fbm(mainUV + 2.0 * q + vec2(1.7, 9.2) + u_time * 0.5 + u_mid*0.2, 2), 
                          fbm(mainUV + 2.0 * q + vec2(8.3, 2.8) + u_time * 0.5 - u_mid*0.2, 2));                       
            float liquid = fbm(mainUV + r * warpStrength, 3);
            float ripple = fbm(mainUV * 4.0 + u_time + u_drift, 2);
            vec2 organicShift = vec2(ripple, ripple) * u_bass * u_react * 0.8; 
            float userGridScale = 2.0 + u_gridSize * 48.0; 
            vec2 gridUV = (mainUV + r * 0.05 + organicShift) * userGridScale;
            gridUV.x += sin(gridUV.y * 0.5 + u_time * 2.0) * u_vol * u_react * 0.8; 
            float lineNoise = fbm(gridUV * 0.1 + u_time * 0.2, 2);
            float thickness = (0.01 + lineNoise * 0.1) * u_lineWeight + (u_bass * u_react * 0.04 * u_lineWeight); 
            float distToLineX = abs(fract(gridUV.x + u_time*0.5) - 0.5);
            float distToLineY = abs(fract(gridUV.y - u_time*0.5) - 0.5);
            float gridX = smoothstep(thickness, thickness - 0.01, distToLineX);
            float gridY = smoothstep(thickness, thickness - 0.01, distToLineY);
            float alphaNoise = fbm(gridUV * 0.3 - u_time * 0.1 + u_drift, 2);
            float gridAlpha = smoothstep(0.3 - (u_vol*0.3), 0.7, alphaNoise); 
            float weightFade = smoothstep(0.0, 0.05, u_lineWeight);
            float rawGrid = max(gridX, gridY);
            float structureIntensity = rawGrid * gridAlpha * weightFade * min(u_params.y * 3.0, 2.0) * (1.0 + u_bass * 0.8);
            structureIntensity += structureIntensity * lineNoise * 0.5; 
            
            structureIntensity *= u_showGrid;

            vec3 themeBG, themeStruct, themePart;
            float themeTime = liquid * 0.5 + u_time * 0.05;
            float themeBeat = u_bass * u_react;
            if (u_useAutoPalette > 0.5) {
                int modeA = int(mod(u_paletteTime, 5.0));
                int modeB = int(mod(u_paletteTime + 1.0, 5.0));
                float mixFactor = fract(u_paletteTime);
                vec3 bg1, st1, pt1; vec3 bg2, st2, pt2;
                getThemeColors(modeA, themeTime, themeBeat, bg1, st1, pt1);
                getThemeColors(modeB, themeTime, themeBeat, bg2, st2, pt2);
                themeBG = mix(bg1, bg2, mixFactor);
                themeStruct = mix(st1, st2, mixFactor);
                themePart = mix(pt1, pt2, mixFactor);
            } else {
                getThemeColors(u_colorMode, themeTime, themeBeat, themeBG, themeStruct, themePart);
            }
            vec2 borderUV = centered;
            if (u_react > 0.1) {
                float distAmt = u_borderParams.w * 0.5; 
                float waveFreq = 10.0;
                float waveAmp = u_bass * u_react * distAmt; 
                float randomOffset = fbm(borderUV * 2.0 + u_time, 2); 
                float waveX = sin(borderUV.y * waveFreq + u_time * 5.0 + randomOffset * 5.0) * waveAmp;
                float waveY = cos(borderUV.x * waveFreq + u_time * 4.0 - randomOffset * 5.0) * waveAmp;
                borderUV += vec2(waveX, waveY);
            }
            vec2 absUV = abs(borderUV);
            float dX = (u_res.x/u_res.y)*0.5 - absUV.x;
            float dY = 0.5 - absUV.y;
            float edgeDist = min(dX, dY);
            float angle = atan(borderUV.y, borderUV.x);
            float noiseMask = fbm(vec2(angle * 6.0, edgeDist * 20.0 - u_time * 2.0), 2);
            float opacityMod = smoothstep(0.3, 0.6, noiseMask); 
            float borderZone = smoothstep(u_borderParams.x + 0.01, u_borderParams.x, edgeDist); 
            float move = u_time * 3.0 + u_bass * u_react * 2.0;
            float pattern = sin(edgeDist * (50.0 + u_borderParams.y * 100.0) + move);
            float borderLines = smoothstep(0.8, 0.9, pattern);
            float borderIntensity = borderLines * borderZone * u_borderParams.z * (1.0 + u_treble * u_react);
            borderIntensity *= opacityMod; 
            
            borderIntensity *= u_showBorder;

            float particleIntensity = 0.0;
            float particleGridScale = 6.0 + u_params.z * 10.0;
            vec2 pUV = mainUV * particleGridScale;
            vec2 id = floor(pUV);
            pUV = fract(pUV) - 0.5;
            for(int y=-1; y<=1; y++) {
                for(int x=-1; x<=1; x++) {
                    vec2 neighbor = vec2(float(x), float(y));
                    vec2 neighborID = id + neighbor;
                    float n = hash(neighborID); 
                    float t = u_time * (0.8 + n * 0.5) + u_drift * n;
                    float x_freq = 2.0 + n * 3.0;
                    float y_freq = 2.0 + fract(n * 10.0) * 3.0;
                    float shake = min(u_treble * u_react, 0.4);
                    float posX = sin(t * x_freq + n * 10.0) * (0.6 + shake);
                    float posY = cos(t * y_freq + n * 20.0) * (0.6 + shake);
                    float z_pos = sin(t * 1.5 + n * 5.0) * 1.5; 
                    vec2 pOffset = neighbor + vec2(posX, posY);
                    float sizeBoost = 1.0 + shake * 2.0;
                    float pSize = (0.1 + u_params.z * 0.2) * (0.5 + n) * sizeBoost;
                    float fade = smoothstep(2.0, -1.0, abs(z_pos));
                    particleIntensity += particle(pUV, pOffset, pSize, z_pos) * fade;
                }
            }
            particleIntensity *= min(u_params.z * 2.0, 1.2); 
            
            particleIntensity *= u_showParticles;

            vec3 borderCol = themePart * borderIntensity; 
            vec3 fullColor = (themeBG * (0.2 + liquid * 0.3)) 
                           + (themeStruct * structureIntensity) 
                           + (themePart * particleIntensity)
                           + borderCol;
            float luminance = dot(fullColor, vec3(0.299, 0.587, 0.114));
            vec3 monoColor = vec3(luminance) * 1.2; 
            monoColor = pow(monoColor, vec3(1.4));
            float volCurve = pow(u_vol * u_react, 1.5) * 3.0; 
            float dynamicSat = u_saturation + (u_saturation * volCurve * 0.3); 
            dynamicSat = clamp(dynamicSat, 0.0, 1.0);
            vec3 finalColor = mix(monoColor, fullColor, dynamicSat);
            finalColor += vec3(pow(u_bass * u_react, 2.0) * 0.15 * dynamicSat); 
            finalColor = finalColor / (finalColor + vec3(1.0)); 
            finalColor = pow(finalColor, vec3(0.8)); 
            gl_FragColor = vec4(finalColor, 1.0);
        }
    `;

    function reloadPage() { location.reload(); }

    function updateSliderUI(id, val) {
        const el = document.getElementById(id);
        if (el) {
            el.value = val;
            if(id === 'imgXParam') document.getElementById('val-img-x').innerText = val.toFixed(2);
            else if(id === 'imgYParam') document.getElementById('val-img-y').innerText = val.toFixed(2);
            else if(id === 'ytXParam') document.getElementById('val-yt-x').innerText = val.toFixed(2);
            else if(id === 'ytYParam') document.getElementById('val-yt-y').innerText = val.toFixed(2);
            else el.previousElementSibling.querySelector('.value').innerText = val.toFixed(2);
        }
    }

    function keyPressed() {
        let changed = false;
        if (keyCode === RIGHT_ARROW) {
            state.global.saturation = constrain(state.global.saturation + 0.05, 0, 1); changed = true;
        } else if (keyCode === LEFT_ARROW) {
            state.global.saturation = constrain(state.global.saturation - 0.05, 0, 1); changed = true;
        } else if (keyCode === UP_ARROW) {
            state.global.react = constrain(state.global.react + 0.1, 0, 3); changed = true;
        } else if (keyCode === DOWN_ARROW) {
            state.global.react = constrain(state.global.react - 0.1, 0, 3); changed = true;
        } else if (key === ' ') {
            togglePlay();
            return false; 
        } else if (keyCode === ENTER) {
            toggleUI();
        }

        if (changed) {
            updateSliderUI('saturationSlider', state.global.saturation);
            updateSliderUI('reactivity', state.global.react);
            return false;
        }
    }

    window.toggleGrid = () => {
        state.global.gridVisible = !state.global.gridVisible;
        const btn = document.getElementById('grid-toggle-btn');
        if(state.global.gridVisible) btn.classList.add('active'); else btn.classList.remove('active');
    }
    window.toggleParticles = () => {
        state.global.particlesVisible = !state.global.particlesVisible;
        const btn = document.getElementById('part-toggle-btn');
        if(state.global.particlesVisible) btn.classList.add('active'); else btn.classList.remove('active');
    }
    window.toggleBorder = () => {
        state.global.borderVisible = !state.global.borderVisible;
        const btn = document.getElementById('border-toggle-btn');
        if(state.global.borderVisible) btn.classList.add('active'); else btn.classList.remove('active');
    }

    window.toggleImgVisibility = () => {
        state.global.imgVisible = !state.global.imgVisible;
        const btn = document.getElementById('img-toggle-btn');
        if(state.global.imgVisible) btn.classList.add('active');
        else btn.classList.remove('active');
    }

    window.toggleYTInteract = () => {
        state.global.ytInteractive = !state.global.ytInteractive;
        const wrapper = document.getElementById('yt-wrapper');
        const btn = document.getElementById('yt-interact-btn');
        if(state.global.ytInteractive) {
            wrapper.classList.add('interactive');
            btn.classList.add('active');
        } else {
            wrapper.classList.remove('interactive');
            btn.classList.remove('active');
        }
    }

    window.toggleYTMute = () => {
        state.global.ytMuted = !state.global.ytMuted;
        const btn = document.getElementById('yt-mute-btn');
        if(state.global.ytMuted) {
            btn.classList.add('active');
            if(ytPlayer && typeof ytPlayer.mute === 'function') ytPlayer.mute();
        } else {
            btn.classList.remove('active');
            if(ytPlayer && typeof ytPlayer.unMute === 'function') ytPlayer.unMute();
        }
    }

    window.loadYouTube = () => {
        const btn = document.getElementById('yt-load-btn');
        if(!ytReady) return alert("API not ready. Wait.");
        
        let inputUrl = document.getElementById('yt-url').value;
        let vidId = "";
        let regExp = /^.*(youtu.be\/|v\/|u\/\w\/|embed\/|watch\?v=|\&v=)([^#\&\?]*).*/;
        let match = inputUrl.match(regExp);
        if (match && match[2].length == 11) vidId = match[2];
        else vidId = inputUrl; 

        if(vidId.length !== 11) return alert("Invalid ID");

        btn.innerText = "LOADING...";

        const onPlayerError = (e) => {
            alert("Error: Video cannot be played (restricted).");
            btn.innerText = "LOAD & PLAY";
        };

        const onPlayerReady = (event) => {
            // 2-step force play
            event.target.mute();
            event.target.playVideo();
            setTimeout(() => {
                if(!state.global.ytMuted) event.target.unMute();
                btn.innerText = "PLAYING!";
            }, 1000);
        };

        if(ytPlayer) {
            ytPlayer.loadVideoById(vidId);
            ytPlayer.mute(); 
            ytPlayer.playVideo();
            setTimeout(() => {
                if(!state.global.ytMuted) ytPlayer.unMute();
                btn.innerText = "PLAYING!";
            }, 1000);
        } else {
            ytPlayer = new YT.Player('yt-player', {
                height: '100%', width: '100%',
                videoId: vidId,
                playerVars: { 
                    'autoplay': 1, 'controls': 0, 'mute': 1, // Init mute for autoplay
                    'loop': 1, 'playlist': vidId,
                    'origin': window.location.origin
                },
                events: {
                    'onReady': onPlayerReady,
                    'onError': onPlayerError
                }
            });
        }
        updateYTStyles();

        // Sync: Restart Audio
        if(soundFile && soundFile.isLoaded()) {
            soundFile.jump(0);
            if(!soundFile.isPlaying()) soundFile.play();
            document.getElementById('play-btn').innerText = "❚❚";
        }
    }

    window.clearYouTube = () => {
        if(ytPlayer) {
            ytPlayer.stopVideo();
            document.getElementById('yt-player').innerHTML = "";
            ytPlayer = null;
            document.getElementById('yt-load-btn').innerText = "LOAD & PLAY";
        }
    }

    function updateYTStyles() {
        let wrapper = document.getElementById('yt-wrapper');
        if(wrapper) {
            let scale = state.global.ytSize;
            let dim = Math.min(window.innerWidth, window.innerHeight) * scale * 2.0; 
            wrapper.style.width = dim + "px";
            wrapper.style.height = (dim * 0.5625) + "px";
            wrapper.style.opacity = state.global.ytOpacity;
            wrapper.style.mixBlendMode = state.global.ytBlend;
            
            let moveX = state.global.ytX * (window.innerWidth / 2);
            let moveY = state.global.ytY * (window.innerHeight / 2);
            wrapper.style.transform = `translate(calc(-50% + ${moveX}px), calc(-50% + ${moveY}px))`;
        }
    }

    function setup() {
        let cnv = createCanvas(windowWidth, windowHeight, WEBGL);
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

        const paletteSelect = document.getElementById('paletteSelect');
        paletteSelect.addEventListener('change', (e) => {
             state.global.colorMode = parseInt(e.target.value);
        });

        const controls = [
            'param1', 'saturationSlider', 'reactivity', 
            'gridSize', 'gridLine', 'param2', 'param3',
            'borderWidth', 'borderDensity', 'borderDistortion', 'borderAlpha',
            'ytSize', 'ytOpacity',
            'imgSizeParam', 'imgOpacityParam', 'imgXParam', 'imgYParam',
            'ytXParam', 'ytYParam'
        ];

        controls.forEach((id) => {
            const el = document.getElementById(id);
            if(el) {
                el.addEventListener('input', (e) => {
                    let val = parseFloat(e.target.value);
                    if(id === 'param1') state.params[0] = val; 
                    else if(id === 'saturationSlider') state.global.saturation = val; 
                    else if(id === 'reactivity') state.global.react = val;
                    else if(id === 'gridSize') state.global.gridSize = val; 
                    else if(id === 'gridLine') state.global.lineWeight = val; 
                    else if(id === 'param2') state.params[1] = val; 
                    else if(id === 'param3') state.params[2] = val; 
                    else if(id === 'borderWidth') state.global.borderWidth = val;
                    else if(id === 'borderDensity') state.global.borderDensity = val;
                    else if(id === 'borderDistortion') state.global.borderDistortion = val;
                    else if(id === 'borderAlpha') state.global.borderAlpha = val;
                    else if(id === 'ytSize') { state.global.ytSize = val; updateYTStyles(); }
                    else if(id === 'ytOpacity') { state.global.ytOpacity = val; updateYTStyles(); }
                    else if(id === 'ytXParam') { state.global.ytX = val; updateYTStyles(); }
                    else if(id === 'ytYParam') { state.global.ytY = val; updateYTStyles(); }
                    else if(id === 'imgSizeParam') state.global.imgSize = val; 
                    else if(id === 'imgOpacityParam') state.global.imgOpacity = val; 
                    else if(id === 'imgXParam') state.global.imgX = val; 
                    else if(id === 'imgYParam') state.global.imgY = val; 

                    updateSliderUI(id, val);
                });
            }
        });

        document.getElementById('ytBlend').addEventListener('change', (e) => {
            state.global.ytBlend = e.target.value;
            updateYTStyles();
        });
        
        const seekSlider = document.getElementById('seek-slider');
        seekSlider.addEventListener('mousedown', () => { isScrubbing = true; });
        seekSlider.addEventListener('touchstart', () => { isScrubbing = true; }, {passive: true});
        seekSlider.addEventListener('input', (e) => {
            if(soundFile && soundFile.isLoaded()) {
                let duration = soundFile.duration();
                let seekTo = (parseFloat(e.target.value) / 100) * duration;
                document.getElementById('current-time').innerText = formatTime(seekTo);
            }
        });
        seekSlider.addEventListener('change', (e) => {
            if(soundFile && soundFile.isLoaded()) {
                let duration = soundFile.duration();
                let seekTo = (parseFloat(e.target.value) / 100) * duration;
                soundFile.jump(seekTo);
                if (!soundFile.isPlaying()) {
                    soundFile.play();
                    document.getElementById('play-btn').innerText = "❚❚";
                }
            }
            isScrubbing = false;
        });
        seekSlider.addEventListener('mouseup', () => { isScrubbing = false; });
        seekSlider.addEventListener('touchend', () => { isScrubbing = false; });

        document.getElementById('paletteSelect').value = state.global.colorMode;
        updateSliderUI('saturationSlider', state.global.saturation);
        updateYTStyles();
    }

    function initApp() {
        userStartAudio(); 
        mic.start(); 
        state.running = true;
        const ov = document.getElementById('overlay');
        ov.style.opacity = 0;
        setTimeout(()=> ov.style.display='none', 500);
    }

    function cleanupSound() {
        if (soundFile) {
            try {
                if (soundFile.isPlaying()) soundFile.stop();
                soundFile.disconnect();
                soundFile.dispose();
            } catch(e) { console.log("Cleanup error", e); }
            soundFile = null;
        }
    }

    function switchToMic() {
        cleanupSound(); 
        mic.start();
        fft.setInput(mic);
        amplitude.setInput(mic);
        document.getElementById('file-status').innerText = "MIC MODE";
        document.getElementById('play-btn').style.display = "none";
        document.getElementById('progress-container').style.display = "none";
        if(!state.syncMode) toggleSync();
    }
    
    function handleFileSelect(fileInput) {
        if (fileInput.files.length === 0) return;
        const file = fileInput.files[0];
        const statusEl = document.getElementById('file-status');
        if (file.size > MAX_FILE_SIZE) { alert("ファイルが大きすぎます"); return; }

        cleanupSound(); 

        statusEl.innerText = "LOADING...";
        const blobUrl = URL.createObjectURL(file);
        
        soundFile = loadSound(blobUrl, () => {
            statusEl.innerText = "FILE READY"; 
            document.getElementById('play-btn').style.display = "flex";
            document.getElementById('play-btn').innerText = "▶"; 
            document.getElementById('progress-container').style.display = "flex";
            
            mic.stop();
            soundFile.setLoop(true); 
            
            fft.setInput(soundFile);
            amplitude.setInput(soundFile);
            if(!state.syncMode) toggleSync();
            
            let dur = soundFile.duration();
            document.getElementById('total-time').innerText = formatTime(dur);
            document.getElementById('seek-slider').value = 0;

            if(ytPlayer && typeof ytPlayer.pauseVideo === 'function') {
                ytPlayer.pauseVideo();
            }

            URL.revokeObjectURL(blobUrl);
        }, (err) => {
            console.error(err);
            statusEl.innerText = "ERROR";
        });
    }

    function formatTime(seconds) {
        let m = Math.floor(seconds / 60);
        let s = Math.floor(seconds % 60);
        return m + ":" + (s < 10 ? "0" : "") + s;
    }

    function handleImageSelect(fileInput) {
        if (fileInput.files.length === 0) return;
        const file = fileInput.files[0];
        const blobUrl = URL.createObjectURL(file);
        const status = document.getElementById('file-status');
        status.innerText = "IMG LOADING...";
        
        centerImg = loadImage(blobUrl, () => {
            console.log("Image loaded successfully");
            status.innerText = "IMG LOADED";
            setTimeout(()=> status.innerText="MIC MODE", 2000);
            URL.revokeObjectURL(blobUrl);
        }, (err) => {
            console.error("Img Error:", err);
            status.innerText = "IMG ERROR";
        });
    }

    function togglePlay() {
        let isPlaying = false;
        if(soundFile && soundFile.isLoaded()) {
            if(soundFile.isPlaying()) {
                soundFile.pause();
                isPlaying = false;
                document.getElementById('play-btn').innerText = "▶";
            } else {
                soundFile.play();
                isPlaying = true;
                document.getElementById('play-btn').innerText = "❚❚";
            }
        }

        if(ytPlayer && typeof ytPlayer.getPlayerState === 'function') {
            if(soundFile && soundFile.isLoaded()) {
                if(isPlaying) ytPlayer.playVideo();
                else ytPlayer.pauseVideo();
            } else {
                let state = ytPlayer.getPlayerState();
                if (state === 1) ytPlayer.pauseVideo();
                else ytPlayer.playVideo();
            }
        }
    }

    let rotAccumulator = 0;

    function draw() {
        if (!state.running && millis() < 1000) return;

        if (soundFile && soundFile.isPlaying() && !isScrubbing) {
            let currentTime = soundFile.currentTime();
            let duration = soundFile.duration();
            if (duration > 0) {
                let pct = (currentTime / duration) * 100;
                document.getElementById('seek-slider').value = pct;
                document.getElementById('current-time').innerText = formatTime(currentTime);
            }
        }

        let spectrum = fft.analyze();
        let bass = fft.getEnergy("bass") / 255.0;      
        let mid = fft.getEnergy("mid") / 255.0;
        let treble = fft.getEnergy("treble") / 255.0;
        let vol = amplitude.getLevel();

        if (!state.uiHidden) {
            const w = vizCtx.canvas.width;
            const h = vizCtx.canvas.height;
            vizCtx.clearRect(0, 0, w, h);
            vizCtx.fillStyle = state.syncMode ? 'rgba(0, 255, 255, 0.5)' : 'rgba(255, 255, 255, 0.2)';
            let barW = w / spectrum.length;
            for (let i = 0; i < spectrum.length; i++) {
                let barH = (spectrum[i] / 255) * (h - 2) * (0.5 + vol * 1.5); 
                vizCtx.fillRect(i * barW, h - barH, barW - 1, barH);
            }
        }

        let smoothFactor = 0.08; 
        state.avgBass = lerp(state.avgBass, bass, smoothFactor);
        state.avgMid = lerp(state.avgMid, mid, smoothFactor);
        state.avgTreble = lerp(state.avgTreble, treble, smoothFactor);
        state.avgVol = lerp(state.avgVol, vol, smoothFactor);

        let speedVal = state.params[0]; 
        let increment = 0;

        if (state.syncMode) {
            let minDrift = speedVal * 0.0002; 
            let dynamicJump = speedVal * 0.15 * pow(state.avgBass, 4.0) * state.global.react;
            increment = minDrift + dynamicJump;
            state.drift += 0.002 + (state.avgBass * 0.01); 
            state.paletteTime += 0.005 + (state.avgBass * 0.05);
        } else {
            let baseSpeed = speedVal * 0.02;
            let audioBoost = state.avgVol * 0.2 * state.global.react;
            increment = baseSpeed + audioBoost;
            state.drift += 0.002;
        }
        
        state.customTime += increment;

        let targetP = [0, 0, 0];
        let targetBorder = { 
            width: state.global.borderWidth, 
            density: state.global.borderDensity, 
            alpha: state.global.borderAlpha,
            distortion: state.global.borderDistortion 
        };

        if (state.syncMode) {
            targetP[0] = state.params[0] + (pow(state.avgVol, 1.5) * 2.5); 
            targetP[1] = state.params[1] + (state.avgBass * 2.0 * state.global.react); 
            targetP[2] = Math.min(1.0, state.params[2] + (state.avgTreble * 1.5 * state.global.react)); 
            rotAccumulator += (0.0005 + state.avgVol * 0.05); 
            
            if (state.global.borderWidth > 0.01) {
                targetBorder.width = state.global.borderWidth + (state.avgBass * 0.1 * state.global.react);
                targetBorder.alpha = Math.min(1.0, state.global.borderAlpha + (state.avgTreble * 0.2));
                targetBorder.distortion = state.global.borderDistortion + (state.avgBass * state.global.react);
            }
        } else {
            targetP = [...state.params];
            rotAccumulator += 0.0005;
        }

        for(let i=0; i<3; i++) {
            state.dynamicParams[i] = lerp(state.dynamicParams[i], targetP[i], 0.05);
            state.dynamicParams[i] = constrain(state.dynamicParams[i], 0.0, 5.0); 
        }

        state.dynamicBorder.width = lerp(state.dynamicBorder.width, targetBorder.width, 0.1);
        state.dynamicBorder.density = lerp(state.dynamicBorder.density, targetBorder.density, 0.1);
        state.dynamicBorder.alpha = lerp(state.dynamicBorder.alpha, targetBorder.alpha, 0.1);
        state.dynamicBorder.distortion = lerp(state.dynamicBorder.distortion, targetBorder.distortion, 0.1);

        shader(myShader);
        myShader.setUniform('u_res', [width, height]); 
        myShader.setUniform('u_time', state.customTime);
        myShader.setUniform('u_drift', state.drift); 
        myShader.setUniform('u_params', state.dynamicParams);
        myShader.setUniform('u_react', state.global.react); 
        myShader.setUniform('u_saturation', state.global.saturation); 
        myShader.setUniform('u_colorMode', state.global.colorMode); 
        myShader.setUniform('u_useAutoPalette', state.syncMode ? 1.0 : 0.0);
        myShader.setUniform('u_paletteTime', state.paletteTime);
        myShader.setUniform('u_gridSize', state.global.gridSize); 
        myShader.setUniform('u_lineWeight', state.global.lineWeight); 
        myShader.setUniform('u_rot', rotAccumulator);
        myShader.setUniform('u_bass', state.avgBass); 
        myShader.setUniform('u_mid', state.avgMid);
        myShader.setUniform('u_treble', state.avgTreble);
        myShader.setUniform('u_vol', state.avgVol);
        
        myShader.setUniform('u_borderParams', [
            state.dynamicBorder.width, 
            state.dynamicBorder.density, 
            state.dynamicBorder.alpha, 
            state.dynamicBorder.distortion
        ]);
        
        myShader.setUniform('u_showGrid', state.global.gridVisible ? 1.0 : 0.0);
        myShader.setUniform('u_showParticles', state.global.particlesVisible ? 1.0 : 0.0);
        myShader.setUniform('u_showBorder', state.global.borderVisible ? 1.0 : 0.0);

        rect(-width/2, -height/2, width, height);

        if (centerImg && state.global.imgVisible) {
            if (centerImg.width > 0) {
                resetShader(); 
                push();
                let gl = this._renderer.GL;
                gl.disable(gl.DEPTH_TEST);
                
                let offX = state.global.imgX * (width / 2); 
                let offY = state.global.imgY * (height / 2); 
                translate(offX, offY, 100); 
                noLights(); 
                
                let aspect = centerImg.width / centerImg.height;
                let baseScale = min(width, height) * 0.8; 
                let displayW, displayH;

                if (aspect > 1) {
                    displayW = baseScale * state.global.imgSize;
                    displayH = displayW / aspect;
                } else {
                    displayH = baseScale * state.global.imgSize;
                    displayW = displayH * aspect;
                }

                tint(255, state.global.imgOpacity * 255);
                texture(centerImg);
                noStroke();
                plane(displayW, displayH);
                gl.enable(gl.DEPTH_TEST);
                pop();
            }
        }
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
        const palLabel = document.getElementById('paletteLabel');
        if(state.syncMode) {
            btn.classList.add('active'); 
            lbl.innerText = "AUDIO SYNC: ON";
            palLabel.innerText = "Color Palette (CYCLING)";
            document.querySelectorAll('.slider-group').forEach(el => el.classList.add('sync-active'));
        } else {
            btn.classList.remove('active'); 
            lbl.innerText = "AUDIO SYNC: OFF";
            palLabel.innerText = "Color Palette";
            document.querySelectorAll('.slider-group').forEach(el => el.classList.remove('sync-active'));
        }
    };
    
    function windowResized() { resizeCanvas(windowWidth, windowHeight); }
</script>
</body>
</html>
