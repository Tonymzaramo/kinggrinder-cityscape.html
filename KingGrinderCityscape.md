<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>KingGrinder — Procedural Cityscape Designer</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { font-family: system-ui, sans-serif; }
        canvas { border: 1px solid #222; image-rendering: crisp-edges; }
        .panel { background: rgba(15, 15, 20, 0.97); }
    </style>
</head>
<body class="bg-zinc-950 text-white">
    <div class="flex h-screen">
        <!-- Preview -->
        <div class="flex-1 flex flex-col">
            <div class="p-4 border-b border-zinc-800 flex items-center justify-between bg-black">
                <h1 class="text-2xl font-bold flex items-center gap-3">
                    <span class="text-cyan-400">★</span> KingGrinder
                </h1>
                <div class="flex gap-4">
                    <button id="playBtn" class="px-6 py-2 bg-violet-600 hover:bg-violet-700 rounded-lg font-medium transition">▶ Play</button>
                    <button id="exportBtn" class="px-6 py-2 bg-emerald-600 hover:bg-emerald-700 rounded-lg font-medium transition">Export PNG</button>
                </div>
            </div>
            <div class="flex-1 flex items-center justify-center bg-black p-8">
                <canvas id="canvas" width="1080" height="1080" class="shadow-2xl rounded-xl"></canvas>
            </div>
            <div class="p-4 border-t border-zinc-800 text-xs text-zinc-400 flex items-center gap-6 bg-black">
                <div>Time: <span id="timeDisplay" class="font-mono text-cyan-300">0.00s</span></div>
                <input type="range" id="timeSlider" min="0" max="20" step="0.01" value="0" class="flex-1 accent-cyan-500">
            </div>
        </div>
<!-- Controls -->
        <div class="w-96 border-l border-zinc-800 overflow-auto panel">
            <div class="p-6 space-y-8">
<!-- Atmospheric Fog -->
                <div>
                    <h2 class="text-lg font-semibold mb-3 text-emerald-300 flex items-center gap-2">
                        <span>🌫️</span> Atmospheric Fog
                    </h2>
                    <div class="space-y-4">
                        <div>
                            <label class="text-xs block mb-1">Fog Intensity <span id="fogIntensityVal" class="font-mono">0.48</span></label>
                            <input type="range" id="fogIntensity" min="0" max="0.85" step="0.01" value="0.48" class="w-full accent-emerald-400">
                        </div>
                        <div>
                            <label class="text-xs block mb-1">Fog Height <span id="fogHeightVal" class="font-mono">0.68</span></label>
                            <input type="range" id="fogHeight" min="0.35" max="0.95" step="0.01" value="0.68" class="w-full accent-emerald-400">
                        </div>
                        <div>
                            <label class="text-xs block mb-1">Fog Density Noise</label>
                            <input type="range" id="fogNoise" min="0" max="1" step="0.05" value="0.45" class="w-full accent-emerald-400">
                        </div>
                        <div>
                            <label class="text-xs block mb-1">Fog Color</label>
                            <input type="color" id="fogColor" value="#0a1a2a" class="w-full h-9 rounded-lg">
                        </div>
                    </div>
                </div>
              <!-- Shooting Stars -->
                <div>
                    <h2 class="text-lg font-semibold mb-3 text-cyan-300">Shooting Stars</h2>
                    <div class="space-y-4">
                        <div>
                            <label class="text-xs block mb-1">Frequency <span id="freqVal" class="font-mono">2.1</span></label>
                            <input type="range" id="shootingFreq" min="0.4" max="7" step="0.1" value="2.1" class="w-full accent-cyan-400">
                        </div>
                        <div>
                            <label class="text-xs block mb-1">Speed <span id="speedVal" class="font-mono">1.0</span></label>
                            <input type="range" id="shootingSpeed" min="0.5" max="2.5" step="0.05" value="1.0" class="w-full accent-cyan-400">
                        </div>
                    </div>
                </div>
<!-- Procedural City -->
                <div>
                    <h2 class="text-lg font-semibold mb-3 text-amber-300">Procedural City</h2>
                    <div class="space-y-4">
                        <div>
                            <label class="text-xs block mb-1">Building Density <span id="densityVal">58</span></label>
                            <input type="range" id="buildingCount" min="25" max="110" value="58" class="w-full accent-amber-400">
                        </div>
                        <div>
                            <label class="text-xs block mb-1">Height Chaos <span id="chaosVal">1.15</span></label>
                            <input type="range" id="heightVar" min="0.4" max="2.2" step="0.05" value="1.15" class="w-full accent-amber-400">
                        </div>
                        <div>
                            <label class="text-xs block mb-1">Window Glow</label>
                            <input type="range" id="glowSlider" min="10" max="95" value="52" class="w-full accent-orange-400">
                        </div>
                    </div>
                </div>

  <div class="pt-6">
                    <button onclick="randomizeScene()" class="w-full py-3.5 bg-zinc-800 hover:bg-zinc-700 rounded-2xl text-sm font-medium">Randomize Entire Scene</button>
                </div>
            </div>
        </div>
    </div>

  <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        let time = 0;
        let isPlaying = false;
        let frame;
        let fps = 30;

        let params = {
            buildingCount: 58,
            heightVar: 1.15,
            glow: 52,
            moonSize: 138,
            starCount: 480,
            shootingFreq: 2.1,
            shootingSpeed: 1.0,
            // Fog parameters
            fogIntensity: 0.48,
            fogHeight: 0.68,
            fogNoise: 0.45,
            fogColor: '#0a1a2a'
        };

        let stars = [];
        let shootingStars = [];
        let buildings = [];

        function generateStars() {
            stars = [];
            for (let i = 0; i < 480; i++) {
                stars.push({
                    x: Math.random() * canvas.width,
                    y: Math.random() * canvas.height * 0.55,
                    size: Math.random() * 2.4 + 0.7,
                    speed: Math.random() * 4 + 1.5
                });
            }
        }

        function generateBuildings() {
            buildings = [];
            const step = canvas.width / (params.buildingCount + 4);
            for (let i = 0; i < params.buildingCount; i++) {
                const w = 32 + Math.random() * 38;
                const h = 210 + (Math.sin(i) * 0.5 + Math.random()) * 390 * params.heightVar;
                buildings.push({
                    x: step * (i + 2) + (Math.random() - 0.5) * 28,
                    w: w,
                    h: h
                });
            }
        }

        function spawnShootingStar() {
            if (shootingStars.length >= 3) return;
            shootingStars.push({
                x: Math.random() * canvas.width * 0.6,
                y: Math.random() * canvas.height * 0.35,
                vx: (9 + Math.random() * 8) * params.shootingSpeed,
                vy: (5 + Math.random() * 6) * params.shootingSpeed,
                life: 45 + Math.random() * 30,
                maxLife: 45 + Math.random() * 30
            });
        }

        function drawBackground() {
            ctx.fillStyle = '#06060f';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Stars
            for (let s of stars) {
                const tw = Math.sin(time * s.speed * 1.8) * 0.35 + 0.75;
                ctx.globalAlpha = tw;
                ctx.fillStyle = '#e0e8ff';
                ctx.fillRect(s.x, s.y, s.size, s.size);
            }
            ctx.globalAlpha = 1;

            // Shooting Stars
            for (let i = shootingStars.length - 1; i >= 0; i--) {
                const s = shootingStars[i];
                const a = Math.pow(s.life / s.maxLife, 1.5);

                ctx.strokeStyle = `rgba(180, 230, 255, ${a * 0.8})`;
                ctx.lineWidth = 4 * a;
                ctx.beginPath();
                ctx.moveTo(s.x, s.y);
                ctx.lineTo(s.x - s.vx * 0.85, s.y - s.vy * 0.85);
                ctx.stroke();

                ctx.fillStyle = `rgba(255, 252, 230, ${a})`;
                ctx.beginPath();
                ctx.arc(s.x, s.y, 3.2, 0, Math.PI * 2);
                ctx.fill();

                s.x += s.vx;
                s.y += s.vy;
                s.life--;
                if (s.life <= 0) shootingStars.splice(i, 1);
            }
        }

        function drawFog() {
            const fogHeightPx = canvas.height * params.fogHeight;

            // Base fog gradient
            const grad = ctx.createLinearGradient(0, canvas.height - fogHeightPx, 0, canvas.height);
            grad.addColorStop(0, 'rgba(10, 26, 42, 0)');
            grad.addColorStop(1, params.fogColor);

            ctx.globalAlpha = params.fogIntensity;
            ctx.fillStyle = grad;
            ctx.fillRect(0, canvas.height - fogHeightPx, canvas.width, fogHeightPx);

            // Fog noise / volumetric effect
            ctx.globalAlpha = params.fogIntensity * params.fogNoise * 0.6;
            for (let i = 0; i < 650; i++) {
                const x = Math.random() * canvas.width;
                const y = canvas.height - Math.random() * fogHeightPx * 0.9;
                const size = Math.random() * 3 + 1.5;
                ctx.fillStyle = '#aaccdd';
                ctx.fillRect(x, y, size, size * 0.6);
            }
            ctx.globalAlpha = 1;
        }

        function drawMoon() {
            const cx = canvas.width * 0.73;
            const cy = canvas.height * 0.29;
            const r = params.moonSize;

            for (let i = 70; i > 0; i -= 10) {
                ctx.globalAlpha = (i / 80) * 0.1;
                ctx.fillStyle = '#fff8e0';
                ctx.beginPath();
                ctx.arc(cx, cy, r + i * 3, 0, Math.PI * 2);
                ctx.fill();
            }
            ctx.globalAlpha = 1;

            ctx.fillStyle = '#f8f0d8';
            ctx.beginPath();
            ctx.arc(cx, cy, r, 0, Math.PI * 2);
            ctx.fill();
        }

        function drawCity() {
            const baseY = canvas.height * 0.81;
            for (let b of buildings) {
                ctx.fillStyle = '#111118';
                ctx.fillRect(b.x - b.w/2, baseY - b.h, b.w, b.h);

                // Windows
                ctx.fillStyle = `rgba(255, 240, 170, ${params.glow / 125})`;
                const rows = Math.floor(b.h / 27);
                for (let r = 2; r < rows; r++) {
                    if (Math.random() > 0.33) {
                        const wx = b.x - b.w/2 + 12 + (r % 3) * (b.w / 4);
                        const wy = baseY - b.h + 20 + r * 27;
                        ctx.fillRect(wx, wy, 7, 12);
                    }
                }

                ctx.fillStyle = '#1e1e2a';
                ctx.fillRect(b.x - b.w/2, baseY - b.h - 8, b.w, 14);
            }
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            drawBackground();
            drawMoon();
            drawCity();
            drawFog();

            // Final vignette
            const vig = ctx.createRadialGradient(540, 520, 380, 540, 540, 1100);
            vig.addColorStop(0, 'transparent');
            vig.addColorStop(1, 'rgba(5, 8, 22, 0.9)');
            ctx.fillStyle = vig;
            ctx.fillRect(0, 0, canvas.width, canvas.height);
        }

        function animate() {
            if (!isPlaying) return;
            time += 1 / fps;
            if (time > 20) time = 0;

            document.getElementById('timeDisplay').textContent = time.toFixed(2) + 's';
            document.getElementById('timeSlider').value = time;

            if (Math.random() < params.shootingFreq / (fps * 52)) spawnShootingStar();

            draw();
            frame = requestAnimationFrame(animate);
        }

        function randomizeScene() {
            params.buildingCount = 40 + Math.random() * 55 | 0;
            params.heightVar = 0.7 + Math.random() * 1.3;
            params.glow = 30 + Math.random() * 55;
            params.fogIntensity = 0.35 + Math.random() * 0.4;
            generateBuildings();
            draw();
        }

        // Setup listeners
        document.getElementById('fogIntensity').addEventListener('input', e => {
            params.fogIntensity = +e.target.value;
            document.getElementById('fogIntensityVal').textContent = params.fogIntensity.toFixed(2);
        });
        document.getElementById('fogHeight').addEventListener('input', e => {
            params.fogHeight = +e.target.value;
            document.getElementById('fogHeightVal').textContent = params.fogHeight.toFixed(2);
        });
        document.getElementById('fogNoise').addEventListener('input', e => params.fogNoise = +e.target.value);
        document.getElementById('fogColor').addEventListener('input', e => params.fogColor = e.target.value);

        document.getElementById('buildingCount').addEventListener('input', e => {
            params.buildingCount = +e.target.value;
            generateBuildings();
            document.getElementById('densityVal').textContent = params.buildingCount;
        });
        document.getElementById('heightVar').addEventListener('input', e => {
            params.heightVar = +e.target.value;
            generateBuildings();
            document.getElementById('chaosVal').textContent = params.heightVar.toFixed(2);
        });
        document.getElementById('glowSlider').addEventListener('input', e => params.glow = +e.target.value);
        document.getElementById('shootingFreq').addEventListener('input', e => params.shootingFreq = +e.target.value);
        document.getElementById('shootingSpeed').addEventListener('input', e => params.shootingSpeed = +e.target.value);

        document.getElementById('playBtn').addEventListener('click', () => {
            isPlaying = !isPlaying;
            document.getElementById('playBtn').textContent = isPlaying ? '❚❚ Pause' : '▶ Play';
            if (isPlaying) animate();
            else cancelAnimationFrame(frame);
        });

        document.getElementById('exportBtn').addEventListener('click', () => {
            const a = document.createElement('a');
            a.download = `kinggrinder-${Date.now()}.png`;
            a.href = canvas.toDataURL('image/png');
            a.click();
        });

        // Initialize
        generateStars();
        generateBuildings();
        document.getElementById('fogIntensityVal').textContent = params.fogIntensity.toFixed(2);
        document.getElementById('fogHeightVal').textContent = params.fogHeight.toFixed(2);
        document.getElementById('densityVal').textContent = params.buildingCount;
        document.getElementById('chaosVal').textContent = params.heightVar.toFixed(2);

        draw();

        setTimeout(() => {
            isPlaying = true;
            document.getElementById('playBtn').textContent = '❚❚ Pause';
            animate();
        }, 600);
    </script>
</body>
</html>
