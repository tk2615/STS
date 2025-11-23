<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>STRANGER SOURCE: REALTIME</title>
    
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Benguiat+ITC&family=Roboto+Slab:wght@900&family=Share+Tech+Mono&display=swap" rel="stylesheet">
    
    <style>
        body { 
            margin: 0; overflow: hidden; color: #ff1100; 
            font-family: 'Roboto Slab', serif;
            background: #000;
            touch-action: manipulation;
        }

        #canvas-container { width: 100vw; height: 100vh; position: fixed; top: 0; left: 0; z-index: 1; }
        
        /* --- UI DESIGN --- */
        #overlay {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background: #000; z-index: 100; display: flex; flex-direction: column;
            align-items: center; justify-content: center; transition: opacity 1s ease;
        }
        
        /* STRANGER THINGS LOGO STYLE */
        .title { 
            font-family: 'Roboto Slab', serif; font-weight: 900; font-size: 7vw; 
            letter-spacing: -0.03em; color: transparent; 
            -webkit-text-stroke: 2px #ff1100;
            text-shadow: 0 0 20px rgba(255, 17, 0, 0.6); 
            margin-bottom: 20px; text-align: center; 
            transform: scaleY(1.1);
        }
        .title span { display: block; font-size: 0.4em; letter-spacing: 0.2em; -webkit-text-stroke: 1px #ff1100; margin-top: -10px;}

        #hud, #legend, #source-list {
            position: absolute; z-index: 20; pointer-events: none;
            font-family: 'Share Tech Mono', monospace; text-shadow: 0 0 5px currentColor;
            transition: all 1s ease;
        }
        #hud { top: 30px; left: 30px; font-size: 12px; line-height: 1.8; }
        #legend { top: 30px; right: 30px; font-size: 10px; text-align: right; }
        #source-list { bottom: 30px; right: 30px; font-size: 10px; text-align: right; max-width: 60vw; opacity: 0.8; color: #8899aa; }
        
        .hud-val { font-weight: bold; margin-left: 10px; color: #fff; }
        .legend-item { display: flex; align-items: center; justify-content: flex-end; gap: 8px; margin-bottom: 4px;}
        .dot { width: 8px; height: 8px; border-radius: 50%; display: inline-block; box-shadow: 0 0 5px currentColor; }

        #start-btn {
            background: transparent; border: 3px solid #ff1100; color: #ff1100;
            padding: 15px 50px; font-family: 'Roboto Slab', serif; font-weight: 900; font-size: 18px;
            letter-spacing: 3px; cursor: pointer; transition: 0.2s; text-transform: uppercase;
            box-shadow: 0 0 30px rgba(255, 17, 0, 0.4); text-shadow: 0 0 10px rgba(255, 17, 0, 0.8);
            border-radius: 4px;
        }
        #start-btn:hover { 
            background: #ff1100; color: #000; box-shadow: 0 0 60px rgba(255, 17, 0, 1); transform: scale(1.05);
        }

        /* GLITCH EFFECT */
        .noise-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyBAMAAADsEZWCAAAAGFBMVEUAAAA5OTkAAABERERmZmYzMzOZmZlVVVUPKLulAAAACHRSTlMvAAAAAAAACE1LMNQAAAAJcEhZcwAADsQAAA7EAZUrDhsAAABDSURBVDjLY2AYBaNg2AJGJiBpCqSsgCScgCQzkLQGkhhAtBaQxACSxkAqbSCpDCRVAEkcIImDJPKAJAmQJAGSJEAyCgD1zxO130O5GgAAAABJRU5ErkJggg==');
            opacity: 0.08; pointer-events: none; z-index: 10;
        }

        @media (max-width: 768px) {
            .title { font-size: 13vw; -webkit-text-stroke: 1px #ff1100; }
            #hud { font-size: 10px; top: 20px; left: 20px; }
            #legend { font-size: 9px; top: 20px; right: 20px; }
            #source-list { font-size: 8px; bottom: 20px; right: 20px; }
        }
    </style>
</head>
<body>

    <div class="noise-overlay" id="noise"></div>

    <div id="overlay">
        <div class="title">STRANGER<br><span>SOURCE</span></div>
        <button id="start-btn">CONNECT</button>
    </div>

    <div id="hud">
        <div>DIMENSION <span id="mode-display" class="hud-val">INITIALIZING</span></div>
        <div>TIME <span id="clock-display" class="hud-val">--:--</span></div>
        <div>LOC <span id="loc-display" class="hud-val">Scanning...</span></div>
        <div>SIGNAL <span id="status-display" class="hud-val">Offline</span></div>
    </div>

    <div id="legend">
        <div class="legend-item">HAWKINS <span class="dot" id="dot-local"></span></div>
        <div class="legend-item">NETFLIX <span class="dot" id="dot-media"></span></div>
        <div class="legend-item">CAST <span class="dot" id="dot-cast"></span></div>
        <div class="legend-item">LAB <span class="dot" id="dot-lab"></span></div>
        <div class="legend-item">UPSIDE DOWN <span class="dot" id="dot-upside"></span></div>
    </div>

    <div id="source-list">System Standby...</div>
    <div id="canvas-container"></div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/simplex-noise/2.4.0/simplex-noise.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>

    <script>
        // --- CONFIGURATION ---
        const isMobile = window.innerWidth < 768;
        const CONFIG = {
            spawnRate: 180,      
            wordLife: 60000,     
            baseSize: isMobile ? 45 : 65,
            connectDist: isMobile ? 450 : 600,   
            particleCount: isMobile ? 800 : 2500, 
            fogDensity: 0.0004
        };

        // --- ROBUST DATA ACQUISITION ---
        // Google News RSS via Proxy (High Volume, Reliable)
        const NEWS_FEEDS = [
            "https://news.google.com/rss/search?q=Stranger+Things+Netflix&hl=en-US&gl=US&ceid=US:en",
            "https://news.google.com/rss/search?q=Millie+Bobby+Brown&hl=en-US&gl=US&ceid=US:en",
            "https://news.google.com/rss/search?q=Duffer+Brothers&hl=en-US&gl=US&ceid=US:en",
            "https://news.google.com/rss/search?q=Stranger+Things+Season+5&hl=en-US&gl=US&ceid=US:en"
        ];
        
        // Fallback Proxy
        const CORS_PROXY = "https://api.allorigins.win/get?url=";

        // Internal Data (Fail-safe)
        const LORE_TERMS = [
            "ELEVEN", "VECNA", "DEMOGORGON", "MIND FLAYER", "HAWKINS", "UPSIDE DOWN", "NETFLIX",
            "HOPPER", "JOYCE", "WILL", "MIKE", "DUSTIN", "LUCAS", "MAX", "EDDIE", "STEVE", "NANCY", "ROBIN",
            "001", "011", "008", "THE GATE", "RUSSIAN", "HELLFIRE", "D&D", "WAFFLES", "RUN", "FRIENDS",
            "CODE RED", "STARCOURT", "MALL", "LABORATORY", "MKULTRA", "ENERGY", "LIGHTS", "CHRISTMAS"
        ];

        // Color Classification
        const TYPE_LOCAL = 0; // Yellow
        const TYPE_MEDIA = 1; // Pink
        const TYPE_CAST = 2;  // Cyan
        const TYPE_LAB = 3;   // Green
        const TYPE_UPSIDE = 4; // Red

        // Palettes
        const PALETTES = {
            day: { // 1980s Surface
                bg: ['#2a1035', '#401020', '#101030', '#200510'],
                fog: 0x2a1035,
                isUpsideDown: false,
                textCols: {
                    [TYPE_LOCAL]: "#ffcc00", [TYPE_MEDIA]: "#ff0066", [TYPE_CAST]: "#00ffff",
                    [TYPE_LAB]: "#ccff00", [TYPE_UPSIDE]:"#ff3300"
                }
            },
            night: { // Upside Down
                bg: ['#050a15', '#000000', '#0a0505', '#101520'],
                fog: 0x020205,
                isUpsideDown: true,
                textCols: {
                    [TYPE_LOCAL]: "#556677", [TYPE_MEDIA]: "#aa0000", [TYPE_CAST]: "#660000",
                    [TYPE_LAB]: "#223344", [TYPE_UPSIDE]:"#ff0000"
                }
            }
        };

        let wordBuffer = [];
        let appState = { mode: 'day', targetFogCol: new THREE.Color(0x000000), currentBgCols: [], targetPalette: null };
        let isAudioReady = false;

        // --- AUDIO SYSTEM ---
        const Audio = {
            synth: null, pad: null, bass: null, clock: null, reverb: null, delay: null, dist: null,
            async init() {
                await Tone.start();
                Tone.Transport.bpm.value = 84;
                this.reverb = new Tone.Reverb({decay: 8, wet: 0.5}).toDestination();
                await this.reverb.generate();
                this.delay = new Tone.PingPongDelay("8n.", 0.4).connect(this.reverb);
                this.dist = new Tone.Distortion(0.4).connect(this.delay);

                this.synth = new Tone.PolySynth(Tone.Synth, {
                    oscillator: { type: "sawtooth" }, envelope: { attack: 0.01, decay: 0.1, sustain: 0.1, release: 1 }
                }).connect(this.delay);
                this.synth.volume.value = -12;

                this.pad = new Tone.PolySynth(Tone.Synth, {
                    oscillator: { type: "fatsooth", spread: 20 }, envelope: { attack: 1, decay: 3, sustain: 0.8, release: 3 }
                }).connect(this.reverb);
                this.pad.volume.value = -20;

                this.bass = new Tone.MonoSynth({
                    oscillator: { type: "pulse", width: 0.2 }, filter: { type: "lowpass", frequency: 400, Q: 4 },
                    envelope: { attack: 0.01, decay: 0.2, sustain: 0.4, release: 0.5 }
                }).connect(this.reverb);
                this.bass.volume.value = -12;

                this.clock = new Tone.MembraneSynth().toDestination();
                this.clock.volume.value = -15;

                this.startLoops();
                isAudioReady = true;
                Tone.Transport.start();
            },
            startLoops() {
                const ARPEGGIO = ["C2", "E2", "G2", "B2", "C3", "E3", "G3", "B3"];
                let idx = 0;
                Tone.Transport.scheduleRepeat((time) => {
                    if (appState.mode === 'day') {
                        this.synth.triggerAttackRelease(ARPEGGIO[idx % 8], "16n", time);
                    } else {
                        if (Math.random() > 0.6) this.synth.triggerAttackRelease(ARPEGGIO[Math.floor(Math.random()*4)], "8n", time);
                        if (idx % 4 === 0) this.clock.triggerAttackRelease("C1", "32n", time);
                    }
                    if (idx % 4 === 0) this.bass.triggerAttackRelease(appState.mode === 'day' ? "C2" : "C1", "8n", time);
                    idx++;
                }, "8n");
                Tone.Transport.scheduleRepeat((time) => {
                    this.pad.triggerAttackRelease(appState.mode === 'day' ? ["C3","E3","G3"] : ["C2","Eb2","G2"], "4m", time);
                }, "4m");
            },
            setMode(mode) {
                if(!isAudioReady) return;
                if(mode === 'day') {
                    this.synth.disconnect(this.dist); this.synth.connect(this.delay);
                } else {
                    this.synth.disconnect(this.delay); this.synth.connect(this.dist);
                }
            },
            triggerEvent() {
                // Just a visual trigger hook
            }
        };

        // --- DATA FETCHING ---
        async function fetchFeeds() {
            document.getElementById('status-display').innerText = "Intercepting Signals...";
            
            // 1. Load internal lore first (Instant Data)
            LORE_TERMS.sort(()=>0.5-Math.random()).slice(0, 10).forEach(w => addWord(w, "ARCHIVE", Math.random()>0.7));

            // 2. Fetch Real Data via AllOrigins (Raw XML Parsing)
            let hitCount = 0;
            const parser = new DOMParser();

            const fetchPromises = NEWS_FEEDS.map(url => 
                fetch(CORS_PROXY + encodeURIComponent(url))
                .then(r => r.json())
                .then(data => {
                    if (data && data.contents) {
                        const xmlDoc = parser.parseFromString(data.contents, "text/xml");
                        const items = xmlDoc.querySelectorAll("item > title");
                        items.forEach(item => {
                            const title = item.textContent;
                            processTitle(title, "G-NEWS");
                            hitCount++;
                        });
                    }
                })
                .catch(e => console.log("Signal Lost", e))
            );

            await Promise.all(fetchPromises);
            
            document.getElementById('status-display').innerText = `Signal Locked: ${hitCount} Entities`;
            document.getElementById('source-list').innerHTML = "CONNECTED TO:<br>GOOGLE NEWS RELAY<br>HAWKINS LAB ARCHIVE";
        }

        function processTitle(title, source) {
            // Remove source name often found in Google News like " - Variety"
            const clean = title.split(" - ")[0]; 
            const words = clean.split(/\s+/);
            
            words.forEach(w => {
                const c = w.replace(/[^a-zA-Z0-9]/g, '').toUpperCase();
                // Filter boring words
                if (c.length > 3 && !["WHAT","WHEN","THIS","THAT","WITH","FROM"].includes(c)) {
                    addWord(c, source, false);
                }
            });
        }

        function addWord(text, source, isPriority) {
            let type = TYPE_LOCAL;
            const t = text.toUpperCase();
            
            // Classification
            if (t.includes("NETFLIX") || t.includes("SEASON")) type = TYPE_MEDIA;
            else if (t.includes("MILLIE") || t.includes("FINN") || t.includes("DUFFER")) type = TYPE_CAST;
            else if (t.includes("LAB") || t.includes("SCIENCE") || t.includes("GATE")) type = TYPE_LAB;
            else if (t.includes("VECNA") || t.includes("DEMOGORGON") || t.includes("UPSIDE")) type = TYPE_UPSIDE;

            wordBuffer.push({ text: text, type: type, source: source, isPriority: isPriority });
        }

        // Always keep buffer full with Lore if needed
        setInterval(() => {
            if (wordBuffer.length < 20) {
                const lore = LORE_TERMS[Math.floor(Math.random() * LORE_TERMS.length)];
                addWord(lore, "SYSTEM_REBOOT", false);
            }
        }, 1000);

        setInterval(fetchFeeds, 30000);

        // --- VISUALS ---
        function setInitialCurrentBgCols(mode) {
            appState.currentBgCols = PALETTES[mode].bg.map(hex => new THREE.Color(hex));
        }
        function setTargetPalette(mode) {
            appState.targetPalette = PALETTES[mode].bg.map(hex => new THREE.Color(hex));
            appState.targetFogCol.setHex(PALETTES[mode].fog);
        }
        function updateHUDColors(mode) {
            const p = PALETTES[mode];
            const isUpside = p.isUpsideDown;
            const uiColor = isUpside ? '#aa0000' : '#00ffcc';
            
            document.body.style.color = isUpside ? '#ff0000' : '#ffffff';
            document.getElementById('mode-display').innerText = isUpside ? "THE UPSIDE DOWN" : "1983 / SURFACE";
            document.getElementById('mode-display').style.color = isUpside ? 'red' : '#ffcc00';
            document.getElementById('noise').style.opacity = isUpside ? 0.15 : 0.05;

            const c = p.textCols;
            document.getElementById('dot-local').style.backgroundColor = c[TYPE_LOCAL];
            document.getElementById('dot-media').style.backgroundColor = c[TYPE_MEDIA];
            document.getElementById('dot-cast').style.backgroundColor = c[TYPE_CAST];
            document.getElementById('dot-lab').style.backgroundColor = c[TYPE_LAB];
            document.getElementById('dot-upside').style.backgroundColor = c[TYPE_UPSIDE];
        }

        function updateTime() {
            const now = new Date();
            const timeStr = now.toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
            document.getElementById('clock-display').innerText = timeStr;
            const h = now.getHours();
            let newMode = (h >= 6 && h < 18) ? 'day' : 'night';
            if(appState.mode !== newMode) {
                appState.mode = newMode;
                setTargetPalette(newMode);
                updateHUDColors(newMode);
                if(isAudioReady) Audio.setMode(newMode);
            }
        }
        setInterval(updateTime, 1000);

        async function fetchEnv() {
            try {
                const ip = await fetch('https://ipapi.co/json/').then(r=>r.json());
                document.getElementById('loc-display').innerText = `HAWKINS LAB (RELAY: ${ip.city||'UNKNOWN'})`;
            } catch(e){}
        }

        // THREE.js Setup
        const scene = new THREE.Scene();
        const fogCol = new THREE.Color(0x000000);
        scene.fog = new THREE.FogExp2(fogCol, CONFIG.fogDensity);
        const camera = new THREE.PerspectiveCamera(60, window.innerWidth/window.innerHeight, 1, 4000);
        camera.position.z = isMobile ? 800 : 600;
        const renderer = new THREE.WebGLRenderer({antialias: true, alpha: true});
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        document.getElementById('canvas-container').appendChild(renderer.domElement);

        const group = new THREE.Group(); scene.add(group);
        const wordGroup = new THREE.Group(); group.add(wordGroup);
        
        // Particles
        const pGeo = new THREE.BufferGeometry();
        const pPos = [];
        for(let i=0; i<CONFIG.particleCount; i++) pPos.push((Math.random()-0.5)*3000, (Math.random()-0.5)*3000, (Math.random()-0.5)*3000);
        pGeo.setAttribute('position', new THREE.Float32BufferAttribute(pPos, 3));
        const pMat = new THREE.PointsMaterial({ color: 0xffffff, size: 2, transparent: true, opacity: 0.4 });
        group.add(new THREE.Points(pGeo, pMat));

        // Lines
        const MAX_LINES = 1000;
        const lineGeo = new THREE.BufferGeometry();
        const linePos = new Float32Array(MAX_LINES * 6);
        const lineCols = new Float32Array(MAX_LINES * 6);
        lineGeo.setAttribute('position', new THREE.BufferAttribute(linePos, 3));
        lineGeo.setAttribute('color', new THREE.BufferAttribute(lineCols, 3));
        const lineMat = new THREE.LineBasicMaterial({ vertexColors: true, transparent: true, opacity: 0.3, blending: THREE.AdditiveBlending });
        group.add(new THREE.LineSegments(lineGeo, lineMat));

        const activeSprites = [];

        function createTexture(text, source, type) {
            const canvas = document.createElement('canvas');
            const ctx = canvas.getContext('2d');
            canvas.width = 512; canvas.height = 256;
            const p = PALETTES[appState.mode].textCols;
            const color = p[type] || "#ffffff";
            const isUpside = appState.mode === 'night';
            
            ctx.textAlign = "center"; ctx.textBaseline = "middle";
            ctx.shadowColor = color; ctx.shadowBlur = isUpside ? 20 : 15;
            ctx.fillStyle = color;
            // Safer Font Stack
            ctx.font = `900 80px "Roboto Slab", "Arial Black", sans-serif`; 
            ctx.fillText(text, 256, 100);
            
            ctx.shadowBlur = 0; ctx.fillStyle = isUpside ? "#ff0000" : "#aaaaaa";
            ctx.font = `400 24px "Share Tech Mono", monospace`;
            ctx.fillText(source, 256, 180);

            return { tex: new THREE.CanvasTexture(canvas), col: new THREE.Color(color) };
        }

        function spawnWord() {
            if(wordBuffer.length === 0) return;
            const data = wordBuffer.shift();
            if(wordBuffer.length < 200) wordBuffer.push(data); // Loop buffer

            const { tex, col } = createTexture(data.text, data.source, data.type);
            const mat = new THREE.SpriteMaterial({ map: tex, transparent: true, opacity: 0, blending: THREE.AdditiveBlending, depthWrite: false });
            const sprite = new THREE.Sprite(mat);
            
            const range = isMobile ? 600 : 1000;
            sprite.position.set((Math.random()-0.5)*range, (Math.random()-0.5)*range*0.6, (Math.random()-0.5)*800);
            const baseS = CONFIG.baseSize * (data.isPriority ? 1.5 : 1.0);
            sprite.scale.set(baseS * 2, baseS, 1); // Aspect ratio approx 2:1
            
            wordGroup.add(sprite);
            activeSprites.push({
                mesh: sprite, age: 0, life: CONFIG.wordLife + Math.random()*10000,
                velocity: new THREE.Vector3((Math.random()-0.5)*0.05, (Math.random()-0.5)*0.05, 0),
                col: col
            });
        }

        const simplex = new SimplexNoise();
        let time = 0;

        function animate() {
            requestAnimationFrame(animate);
            time += 0.001; 

            // Bg Lerp
            if(appState.targetPalette && appState.currentBgCols.length > 0) {
                for(let i = 0; i < appState.currentBgCols.length; i++) {
                    appState.currentBgCols[i].lerp(appState.targetPalette[i], 0.005);
                    document.body.style.setProperty(`--c${i+1}`, `#${appState.currentBgCols[i].getHexString()}`);
                }
                fogCol.lerp(appState.targetFogCol, 0.005);
                scene.fog.color.copy(fogCol);
            }
            scene.background = fogCol;

            // Upside Down Effects
            const isUpside = appState.mode === 'night';
            pMat.color.setHex(isUpside ? 0xaa8888 : 0xffffee);
            lineMat.color.setHex(isUpside ? 0xff0000 : 0xffffff);
            lineMat.opacity = isUpside ? 0.4 : 0.2;
            group.rotation.y = time * 0.05;

            // Sprites Logic
            let lineIdx = 0;
            const connectSq = CONFIG.connectDist * CONFIG.connectDist;

            for(let i = activeSprites.length - 1; i >= 0; i--) {
                const s = activeSprites[i]; s.age += 16;
                
                // Fade In/Out
                let op = 1;
                if(s.age < 2000) op = s.age / 2000;
                else if (s.age > s.life - 2000) op = (s.life - s.age) / 2000;
                
                const flicker = isUpside ? (Math.random() > 0.9 ? 0.2 : 1) : 1;
                s.mesh.material.opacity = op * flicker;
                s.mesh.position.add(s.velocity);
                s.mesh.position.y += simplex.noise3D(s.mesh.position.x*0.001, s.mesh.position.y*0.001, time) * 0.1;

                if(s.age > s.life) { wordGroup.remove(s.mesh); s.mesh.material.map.dispose(); s.mesh.material.dispose(); activeSprites.splice(i, 1); continue; }

                // Lines
                if(s.mesh.material.opacity > 0.1) {
                    for(let k=1; k<3; k++) {
                        const j = (i + k*4) % activeSprites.length;
                        if(i===j) continue;
                        const s2 = activeSprites[j];
                        const d2 = s.mesh.position.distanceToSquared(s2.mesh.position);
                        if(d2 < connectSq) {
                            if(lineIdx < MAX_LINES * 6) {
                                const p1=s.mesh.position; const p2=s2.mesh.position;
                                linePos[lineIdx++]=p1.x; linePos[lineIdx++]=p1.y; linePos[lineIdx++]=p1.z;
                                linePos[lineIdx++]=p2.x; linePos[lineIdx++]=p2.y; linePos[lineIdx++]=p2.z;
                                lineCols[lineIdx-6] = s.col.r; lineCols[lineIdx-5] = s.col.g; lineCols[lineIdx-4] = s.col.b;
                                lineCols[lineIdx-3] = s2.col.r; lineCols[lineIdx-2] = s2.col.g; lineCols[lineIdx-1] = s2.col.b;
                            }
                        }
                    }
                }
            }
            lineGeo.setDrawRange(0, lineIdx/3);
            lineGeo.attributes.position.needsUpdate = true;
            lineGeo.attributes.color.needsUpdate = true;

            camera.rotation.z = Math.sin(time * 0.2) * 0.02;
            renderer.render(scene, camera);
        }

        function spawnLoop() {
            spawnWord();
            setTimeout(spawnLoop, CONFIG.spawnRate);
        }

        document.getElementById('start-btn').addEventListener('click', async () => {
            document.getElementById('overlay').style.opacity = 0;
            setTimeout(() => document.getElementById('overlay').remove(), 1500);
            
            await Audio.init();
            const now = new Date();
            const h = now.getHours();
            appState.mode = (h >= 6 && h < 18) ? 'day' : 'night';
            
            setInitialCurrentBgCols(appState.mode);
            setTargetPalette(appState.mode);
            updateHUDColors(appState.mode);
            Audio.setMode(appState.mode);

            fetchEnv();
            fetchFeeds(); // Data Fetch
            spawnLoop();  // Visual Loop
            animate();    // Render Loop
        });

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth/window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });
    </script>
</body>
</html>
