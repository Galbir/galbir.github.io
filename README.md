<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SecureStream Anti-Recording Protection System</title>
    <style>
        :root {
            --primary-color: #00ff88;
            --danger-color: #ff3366;
            --warning-color: #ffaa00;
            --bg-dark: #0a0a0a;
            --bg-medium: #1a1a1a;
            --text-primary: #ffffff;
            --text-secondary: #cccccc;
            --border-color: #333333;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            background: var(--bg-dark);
            color: var(--text-primary);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            overflow: hidden;
            position: relative;
        }

        #canvas {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            z-index: 1;
        }

        #overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            pointer-events: none;
            z-index: 2;
        }

        .control-panel {
            position: fixed;
            top: 20px;
            left: 20px;
            background: rgba(26, 26, 26, 0.9);
            backdrop-filter: blur(10px);
            border: 1px solid var(--border-color);
            border-radius: 12px;
            padding: 20px;
            min-width: 280px;
            z-index: 10;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
        }

        .control-panel h2 {
            color: var(--primary-color);
            font-size: 18px;
            margin-bottom: 15px;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .status-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
            font-size: 14px;
        }

        .status-label {
            color: var(--text-secondary);
        }

        .status-value {
            font-weight: 600;
            padding: 4px 8px;
            border-radius: 4px;
            background: rgba(255, 255, 255, 0.1);
        }

        .status-value.secure {
            color: var(--primary-color);
            background: rgba(0, 255, 136, 0.2);
        }

        .status-value.warning {
            color: var(--warning-color);
            background: rgba(255, 170, 0, 0.2);
        }

        .status-value.danger {
            color: var(--danger-color);
            background: rgba(255, 51, 102, 0.2);
        }

        .metrics {
            margin-top: 20px;
            padding-top: 15px;
            border-top: 1px solid var(--border-color);
        }

        .metrics h3 {
            color: var(--primary-color);
            font-size: 14px;
            margin-bottom: 10px;
            text-transform: uppercase;
        }

        .metric {
            display: flex;
            justify-content: space-between;
            margin-bottom: 5px;
            font-size: 12px;
        }

        .metric-label {
            color: var(--text-secondary);
        }

        .metric-value {
            font-family: 'Courier New', monospace;
        }

        .metric-value.pass { color: var(--primary-color); }
        .metric-value.warn { color: var(--warning-color); }
        .metric-value.fail { color: var(--danger-color); }

        .alert-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            background: rgba(0, 0, 0, 0.95);
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 100;
            backdrop-filter: blur(5px);
        }

        .alert-box {
            background: linear-gradient(135deg, var(--danger-color) 0%, #cc0044 100%);
            border-radius: 16px;
            padding: 30px;
            max-width: 600px;
            width: 90%;
            box-shadow: 0 20px 60px rgba(255, 51, 102, 0.4);
            border: 2px solid rgba(255, 255, 255, 0.2);
            animation: alertPulse 2s ease-in-out infinite;
        }

        @keyframes alertPulse {
            0%, 100% { transform: scale(1); box-shadow: 0 20px 60px rgba(255, 51, 102, 0.4); }
            50% { transform: scale(1.02); box-shadow: 0 25px 70px rgba(255, 51, 102, 0.6); }
        }

        .alert-header {
            text-align: center;
            margin-bottom: 25px;
        }

        .alert-icon {
            font-size: 48px;
            margin-bottom: 10px;
        }

        .alert-title {
            font-size: 24px;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 2px;
            margin-bottom: 5px;
        }

        .alert-subtitle {
            font-size: 16px;
            opacity: 0.9;
        }

        .alert-details {
            background: rgba(0, 0, 0, 0.3);
            border-radius: 8px;
            padding: 20px;
            margin-bottom: 20px;
        }

        .detail-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin-bottom: 15px;
        }

        .detail-item {
            text-align: center;
        }

        .detail-label {
            font-size: 12px;
            color: rgba(255, 255, 255, 0.7);
            text-transform: uppercase;
            letter-spacing: 1px;
            margin-bottom: 5px;
        }

        .detail-value {
            font-size: 16px;
            font-weight: bold;
            color: white;
        }

        .alert-actions {
            background: rgba(0, 0, 0, 0.2);
            border-radius: 8px;
            padding: 15px;
            margin-bottom: 20px;
        }

        .alert-actions h4 {
            color: var(--primary-color);
            margin-bottom: 10px;
            font-size: 14px;
            text-transform: uppercase;
        }

        .action-list {
            list-style: none;
            font-size: 14px;
            line-height: 1.6;
        }

        .action-list li {
            margin-bottom: 5px;
            padding-left: 20px;
            position: relative;
        }

        .action-list li:before {
            content: "▶";
            position: absolute;
            left: 0;
            color: var(--primary-color);
        }

        .alert-footer {
            text-align: center;
            font-size: 12px;
            color: rgba(255, 255, 255, 0.7);
            border-top: 1px solid rgba(255, 255, 255, 0.2);
            padding-top: 15px;
        }

        .device-warning {
            position: fixed;
            top: 20px;
            right: 20px;
            background: linear-gradient(135deg, #0066cc 0%, #004499 100%);
            border-radius: 12px;
            padding: 20px;
            max-width: 300px;
            z-index: 15;
            box-shadow: 0 8px 32px rgba(0, 102, 204, 0.3);
            border: 1px solid rgba(255, 255, 255, 0.2);
            display: none;
        }

        .device-warning h3 {
            color: #66ccff;
            margin-bottom: 10px;
            font-size: 16px;
        }

        .device-warning p {
            font-size: 14px;
            line-height: 1.4;
            opacity: 0.9;
        }

        .extension-list {
            margin-top: 15px;
            padding-top: 15px;
            border-top: 1px solid var(--border-color);
            display: none;
        }

        .extension-list h4 {
            color: var(--danger-color);
            font-size: 12px;
            margin-bottom: 10px;
            text-transform: uppercase;
        }

        .extension-item {
            background: rgba(255, 51, 102, 0.1);
            border-radius: 4px;
            padding: 5px 8px;
            margin-bottom: 5px;
            font-size: 12px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .whitelist-btn {
            background: var(--primary-color);
            color: var(--bg-dark);
            border: none;
            padding: 6px 12px;
            border-radius: 4px;
            font-size: 11px;
            cursor: pointer;
            font-weight: 600;
            text-transform: uppercase;
            letter-spacing: 0.5px;
            transition: all 0.3s ease;
        }

        .whitelist-btn:hover {
            background: #00cc6a;
            transform: translateY(-1px);
        }

        .loading-screen {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            background: var(--bg-dark);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }

        .loader {
            width: 50px;
            height: 50px;
            border: 3px solid var(--border-color);
            border-top: 3px solid var(--primary-color);
            border-radius: 50%;
            animation: spin 1s linear infinite;
            margin-bottom: 20px;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .loading-text {
            color: var(--primary-color);
            font-size: 18px;
            font-weight: 600;
        }
    </style>
</head>
<body>
    <div class="loading-screen" id="loadingScreen">
        <div class="loader"></div>
        <div class="loading-text">Initializing SecureStream...</div>
    </div>

    <canvas id="canvas"></canvas>
    <canvas id="overlay"></canvas>

    <div class="control-panel">
        <h2>🛡️ SecureStream</h2>
        
        <div class="status-item">
            <span class="status-label">Status:</span>
            <span class="status-value" id="status">Initializing</span>
        </div>
        
        <div class="status-item">
            <span class="status-label">Encryption:</span>
            <span class="status-value secure" id="encryption">Active</span>
        </div>
        
        <div class="status-item">
            <span class="status-label">User:</span>
            <span class="status-value" id="userId">user_****</span>
        </div>
        
        <div class="status-item">
            <span class="status-label">Key Rotation:</span>
            <span class="status-value" id="keyRotation">8s</span>
        </div>
        
        <div class="status-item">
            <span class="status-label">Threats:</span>
            <span class="status-value" id="threatCount">0</span>
        </div>

        <div class="extension-list" id="extensionList">
            <h4>🚨 Suspicious Extensions</h4>
            <div id="extensionDetails"></div>
            <button class="whitelist-btn" id="whitelistBtn">Whitelist All</button>
        </div>

        <div class="metrics">
            <h3>📊 Security Metrics</h3>
            <div class="metric">
                <span class="metric-label">Frame Analysis:</span>
                <span class="metric-value pass" id="frameMetric">OK</span>
            </div>
            <div class="metric">
                <span class="metric-label">GPU Performance:</span>
                <span class="metric-value pass" id="gpuMetric">OK</span>
            </div>
            <div class="metric">
                <span class="metric-label">Media Detection:</span>
                <span class="metric-value pass" id="mediaMetric">OK</span>
            </div>
            <div class="metric">
                <span class="metric-label">Extension Scan:</span>
                <span class="metric-value pass" id="extensionMetric">OK</span>
            </div>
            <div class="metric">
                <span class="metric-label">Render Time:</span>
                <span class="metric-value" id="renderTime">0ms</span>
            </div>
        </div>
    </div>

    <div class="device-warning" id="deviceWarning">
        <h3>📱 Device Alert</h3>
        <p>Non-desktop device detected. Some security features may be limited.</p>
    </div>

    <div class="alert-overlay" id="alertOverlay">
        <div class="alert-box">
            <div class="alert-header">
                <div class="alert-icon">🚨</div>
                <div class="alert-title">SECURITY BREACH DETECTED</div>
                <div class="alert-subtitle">Recording/Tamper Attempt Identified</div>
            </div>
            
            <div class="alert-details">
                <div class="detail-grid">
                    <div class="detail-item">
                        <div class="detail-label">Threat Type</div>
                        <div class="detail-value" id="threatType">Unknown</div>
                    </div>
                    <div class="detail-item">
                        <div class="detail-label">Severity</div>
                        <div class="detail-value" id="threatSeverity">Critical</div>
                    </div>
                    <div class="detail-item">
                        <div class="detail-label">Source</div>
                        <div class="detail-value" id="threatSource">Unknown</div>
                    </div>
                    <div class="detail-item">
                        <div class="detail-label">Time</div>
                        <div class="detail-value" id="threatTime">--:--:--</div>
                    </div>
                </div>
            </div>
            
            <div class="alert-actions">
                <h4>🛡️ Immediate Actions</h4>
                <ul class="action-list">
                    <li>Close this browser session immediately</li>
                    <li>Check for suspicious browser extensions</li>
                    <li>Review system processes for recording software</li>
                    <li>Monitor network activity for unauthorized access</li>
                    <li>Consider changing security credentials</li>
                </ul>
            </div>
            
            <div class="alert-footer">
                Session ID: <span id="sessionId">SECURE-0000</span> | 
                Risk Level: <span style="color: #ff6666; font-weight: bold;">HIGH</span>
            </div>
        </div>
    </div>

    <script>
        class SecureStream {
            constructor() {
                this.canvas = document.getElementById('canvas');
                this.gl = this.canvas.getContext('webgl') || this.canvas.getContext('experimental-webgl');
                this.overlay = document.getElementById('overlay');
                this.overlayCtx = this.overlay.getContext('2d');
                
                this.video = null;
                this.texture = null;
                this.program = null;
                this.uniforms = {};
                this.detected = false;
                this.blackout = false;
                this.decryptionKeyValid = true;
                
                this.encryptionKey = this.generateEncryptionKey();
                this.userId = this.generateUserId();
                this.sessionId = this.generateSessionId();
                
                this.metrics = {
                    frameCount: 0,
                    gpuProbes: [],
                    renderTimes: [],
                    threatCount: 0
                };
                
                this.extensionWhitelist = new Set([
                    'lastpass', 'bitwarden', '1password', 'dashlane', 'roboform', 'keeper',
                    'ublock', 'adblock', 'privacybadger', 'ghostery', 'duckduckgo',
                    'reactdeveloper', 'reduxdevtools', 'vuejsdevtools', 'angular',
                    'grammarly', 'evernote', 'pocket', 'notion', 'saveto', 'onenote',
                    'darkreader', 'tampermonkey', 'greasemonkey', 'videospeed', 'stylus',
                    'googlemeet', 'zoom', 'teams', 'slack', 'discord', 'whatsapp'
                ]);
                
                this.suspiciousExtensions = new Set();
                
                this.init();
            }
            
            generateEncryptionKey() {
                return Array.from(crypto.getRandomValues(new Uint8Array(32)))
                    .map(b => b.toString(16).padStart(2, '0')).join('');
            }
            
            generateUserId() {
                return 'user_' + Math.random().toString(36).substr(2, 9);
            }
            
            generateSessionId() {
                return 'SECURE-' + Date.now().toString(36).toUpperCase();
            }
            
            async init() {
                try {
                    await this.setupWebGL();
                    await this.loadVideo();
                    this.setupEventListeners();
                    this.startMonitoring();
                    this.startKeyRotation();
                    this.hideLoadingScreen();
                    this.startRenderLoop();
                } catch (error) {
                    console.error('Initialization failed:', error);
                    this.updateStatus('Error', 'danger');
                }
            }
            
            async setupWebGL() {
                if (!this.gl) {
                    throw new Error('WebGL not supported');
                }
                
                const vertexShaderSource = `
                    attribute vec2 a_position;
                    attribute vec2 a_texCoord;
                    varying vec2 v_texCoord;
                    
                    void main() {
                        gl_Position = vec4(a_position, 0.0, 1.0);
                        v_texCoord = a_texCoord;
                    }
                `;
                
                const fragmentShaderSource = `
                    precision mediump float;
                    varying vec2 v_texCoord;
                    uniform float u_time;
                    uniform float u_encryptionKey;
                    uniform vec2 u_wmPos;
                    uniform float u_wmAlpha;
                    uniform vec3 u_wmColor;
                    
                    void main() {
                        vec2 uv = v_texCoord;
                        
                        // Encryption effect
                        float encryption = sin(uv.x * 10.0 + u_time * 2.0) * 0.1 + 0.9;
                        encryption *= sin(uv.y * 8.0 + u_time * 1.5) * 0.1 + 0.9;
                        
                        // Watermark
                        float wmDist = distance(uv, u_wmPos);
                        float wm = smoothstep(0.02, 0.0, wmDist) * u_wmAlpha;
                        
                        vec3 color = vec3(0.1, 0.1, 0.1) + encryption * vec3(0.8, 0.9, 1.0);
                        color += wm * u_wmColor;
                        
                        gl_FragColor = vec4(color, 1.0);
                    }
                `;
                
                this.program = this.createProgram(vertexShaderSource, fragmentShaderSource);
                this.setupGeometry();
                this.setupUniforms();
                
                console.log('WebGL initialized successfully');
            }
            
            createShader(type, source) {
                const shader = this.gl.createShader(type);
                this.gl.shaderSource(shader, source);
                this.gl.compileShader(shader);
                
                if (!this.gl.getShaderParameter(shader, this.gl.COMPILE_STATUS)) {
                    console.error('Shader compilation error:', this.gl.getShaderInfoLog(shader));
                    this.gl.deleteShader(shader);
                    return null;
                }
                
                return shader;
            }
            
            createProgram(vertexSource, fragmentSource) {
                const vertexShader = this.createShader(this.gl.VERTEX_SHADER, vertexSource);
                const fragmentShader = this.createShader(this.gl.FRAGMENT_SHADER, fragmentSource);
                
                const program = this.gl.createProgram();
                this.gl.attachShader(program, vertexShader);
                this.gl.attachShader(program, fragmentShader);
                this.gl.linkProgram(program);
                
                if (!this.gl.getProgramParameter(program, this.gl.LINK_STATUS)) {
                    console.error('Program linking error:', this.gl.getProgramInfoLog(program));
                    return null;
                }
                
                return program;
            }
            
            setupGeometry() {
                const positions = new Float32Array([
                    -1, -1,  0, 1,
                     1, -1,  1, 1,
                    -1,  1,  0, 0,
                     1,  1,  1, 0
                ]);
                
                const buffer = this.gl.createBuffer();
                this.gl.bindBuffer(this.gl.ARRAY_BUFFER, buffer);
                this.gl.bufferData(this.gl.ARRAY_BUFFER, positions, this.gl.STATIC_DRAW);
                
                const positionLocation = this.gl.getAttribLocation(this.program, 'a_position');
                const texCoordLocation = this.gl.getAttribLocation(this.program, 'a_texCoord');
                
                this.gl.enableVertexAttribArray(positionLocation);
                this.gl.vertexAttribPointer(positionLocation, 2, this.gl.FLOAT, false, 16, 0);
                
                this.gl.enableVertexAttribArray(texCoordLocation);
                this.gl.vertexAttribPointer(texCoordLocation, 2, this.gl.FLOAT, false, 16, 8);
            }
            
            setupUniforms() {
                this.uniforms = {
                    u_time: this.gl.getUniformLocation(this.program, 'u_time'),
                    u_encryptionKey: this.gl.getUniformLocation(this.program, 'u_encryptionKey'),
                    u_wmPos: this.gl.getUniformLocation(this.program, 'u_wmPos'),
                    u_wmAlpha: this.gl.getUniformLocation(this.program, 'u_wmAlpha'),
                    u_wmColor: this.gl.getUniformLocation(this.program, 'u_wmColor')
                };
            }
            
            async loadVideo() {
                const videoUrls = [
                    'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4',
                    'https://samplelib.com/lib/preview/mp4/sample-10s.mp4'
                ];
                
                for (const url of videoUrls) {
                    try {
                        this.video = document.createElement('video');
                        this.video.crossOrigin = 'anonymous';
                        this.video.loop = true;
                        this.video.muted = true;
                        this.video.preload = 'auto';
                        
                        await new Promise((resolve, reject) => {
                            this.video.addEventListener('loadedmetadata', resolve);
                            this.video.addEventListener('error', reject);
                            this.video.src = url;
                        });
                        
                        this.video.play();
                        this.createTexture();
                        console.log('Video loaded successfully:', url);
                        break;
                    } catch (error) {
                        console.warn('Video loading failed:', url, error);
                    }
                }
            }
            
            createTexture() {
                this.texture = this.gl.createTexture();
                this.gl.bindTexture(this.gl.TEXTURE_2D, this.texture);
                this.gl.texParameteri(this.gl.TEXTURE_2D, this.gl.TEXTURE_WRAP_S, this.gl.CLAMP_TO_EDGE);
                this.gl.texParameteri(this.gl.TEXTURE_2D, this.gl.TEXTURE_WRAP_T, this.gl.CLAMP_TO_EDGE);
                this.gl.texParameteri(this.gl.TEXTURE_2D, this.gl.TEXTURE_MIN_FILTER, this.gl.LINEAR);
                this.gl.texParameteri(this.gl.TEXTURE_2D, this.gl.TEXTURE_MAG_FILTER, this.gl.LINEAR);
            }
            
            setupEventListeners() {
                window.addEventListener('resize', () => this.resize());
                this.canvas.addEventListener('click', () => this.handleCanvasClick());
                window.addEventListener('keydown', (e) => this.handleKeyPress(e));
                document.getElementById('whitelistBtn').addEventListener('click', () => this.whitelistExtensions());
                
                this.resize();
            }
            
            resize() {
                const dpr = window.devicePixelRatio || 1;
                const width = window.innerWidth;
                const height = window.innerHeight;
                
                this.canvas.width = width * dpr;
                this.canvas.height = height * dpr;
                this.overlay.width = width * dpr;
                this.overlay.height = height * dpr;
                
                this.canvas.style.width = width + 'px';
                this.canvas.style.height = height + 'px';
                this.overlay.style.width = width + 'px';
                this.overlay.style.height = height + 'px';
                
                this.gl.viewport(0, 0, this.canvas.width, this.canvas.height);
            }
            
            handleCanvasClick() {
                if (!this.detected && this.video) {
                    this.video.muted = false;
                }
            }
            
            handleKeyPress(e) {
                switch (e.key.toUpperCase()) {
                    case 'R':
                        this.detectThreat('Manual trigger', 'User-initiated test');
                        break;
                    case 'K':
                        this.toggleDecryptionKey();
                        break;
                    case 'E':
                        this.rotateEncryptionKey();
                        break;
                }
            }
            
            whitelistExtensions() {
                this.suspiciousExtensions.clear();
                this.updateExtensionDisplay();
                this.updateMetrics();
            }
            
            startMonitoring() {
                this.detectDeviceType();
                this.detectExtensions();
                this.detectRecordingAttempts();
                this.startGPUMonitoring();
            }
            
            detectDeviceType() {
                const isMobile = /android|webos|iphone|ipad|ipod|blackberry|iemobile|opera mini/i.test(navigator.userAgent.toLowerCase());
                if (isMobile) {
                    document.getElementById('deviceWarning').style.display = 'block';
                }
            }
            
            detectExtensions() {
                const extensions = [
                    'chrome-extension://', 'moz-extension://', 'edge-extension://',
                    'safari-extension://', 'opera-extension://'
                ];
                
                extensions.forEach(ext => {
                    if (document.location.href.startsWith(ext)) {
                        this.suspiciousExtensions.add('Browser Extension');
                    }
                });
                
                // Check for known recording software extensions
                const recordingExtensions = ['obs', 'streamlabs', 'bandicam', 'camtasia'];
                recordingExtensions.forEach(ext => {
                    if (document.referrer.toLowerCase().includes(ext)) {
                        this.suspiciousExtensions.add(ext + ' referral');
                    }
                });
                
                this.updateExtensionDisplay();
            }
            
            detectRecordingAttempts() {
                // Monitor for MediaRecorder API usage
                const originalMediaRecorder = window.MediaRecorder;
                window.MediaRecorder = function(...args) {
                    console.warn('MediaRecorder detected - potential recording attempt');
                    return originalMediaRecorder.apply(this, args);
                };
                
                // Monitor for canvas capture attempts
                const originalToBlob = HTMLCanvasElement.prototype.toBlob;
                HTMLCanvasElement.prototype.toBlob = function(...args) {
                    console.warn('Canvas toBlob called - potential capture attempt');
                    return originalToBlob.apply(this, args);
                };
                
                // Monitor for screen sharing
                const originalGetDisplayMedia = navigator.mediaDevices.getDisplayMedia;
                navigator.mediaDevices.getDisplayMedia = async function(...args) {
                    console.warn('getDisplayMedia called - potential screen sharing attempt');
                    return originalGetDisplayMedia.apply(this, args);
                };
            }
            
            startGPUMonitoring() {
                setInterval(() => {
                    if (!this.gl) return;
                    
                    const pixels = new Uint8Array(4);
                    const startTime = performance.now();
                    
                    try {
                        this.gl.readPixels(this.canvas.width - 1, this.canvas.height - 1, 1, 1, this.gl.RGBA, this.gl.UNSIGNED_BYTE, pixels);
                        const endTime = performance.now();
                        const renderTime = endTime - startTime;
                        
                        this.metrics.gpuProbes.push(renderTime);
                        if (this.metrics.gpuProbes.length > 30) {
                            this.metrics.gpuProbes.shift();
                        }
                        
                        this.updateRenderTime(renderTime);
                        
                        // Detect slow GPU performance (potential recording software)
                        if (this.metrics.gpuProbes.length >= 10) {
                            const avgRenderTime = this.metrics.gpuProbes.reduce((a, b) => a + b, 0) / this.metrics.gpuProbes.length;
                            if (avgRenderTime > 6.0) {
                                this.detectThreat('Slow GPU Performance', `Average render time: ${avgRenderTime.toFixed(1)}ms`);
                            }
                        }
                    } catch (error) {
                        console.warn('GPU probe failed:', error);
                    }
                }, 1000);
            }
            
            startKeyRotation() {
                setInterval(() => {
                    this.rotateEncryptionKey();
                }, 8000); // 8 second key rotation
            }
            
            rotateEncryptionKey() {
                this.encryptionKey = this.generateEncryptionKey();
                this.updateStatus('Key rotated', 'secure');
            }
            
            toggleDecryptionKey() {
                this.decryptionKeyValid = !this.decryptionKeyValid;
                const status = this.decryptionKeyValid ? 'Key valid' : 'Key invalidated';
                const className = this.decryptionKeyValid ? 'secure' : 'danger';
                this.updateStatus(status, className);
            }
            
            detectThreat(type, source) {
                this.detected = true;
                this.metrics.threatCount++;
                
                const now = new Date();
                const timeString = now.toLocaleTimeString();
                
                document.getElementById('threatType').textContent = type;
                document.getElementById('threatSource').textContent = source;
                document.getElementById('threatTime').textContent = timeString;
                document.getElementById('sessionId').textContent = this.sessionId;
                document.getElementById('threatCount').textContent = this.metrics.threatCount;
                
                document.getElementById('alertOverlay').style.display = 'flex';
                this.updateStatus('SECURITY BREACH', 'danger');
                
                // Apply blackout
                this.blackout = true;
                setTimeout(() => {
                    this.blackout = false;
                }, 5000);
                
                console.warn(`Threat detected: ${type} - ${source}`);
            }
            
            updateExtensionDisplay() {
                const extensionList = document.getElementById('extensionList');
                const extensionDetails = document.getElementById('extensionDetails');
                
                if (this.suspiciousExtensions.size > 0) {
                    extensionList.style.display = 'block';
                    extensionDetails.innerHTML = Array.from(this.suspiciousExtensions)
                        .map(ext => `<div class="extension-item">${ext}</div>`).join('');
                } else {
                    extensionList.style.display = 'none';
                }
            }
            
            updateMetrics() {
                const frameOk = this.metrics.frameCount % 60 < 45;
                const gpuOk = this.metrics.gpuProbes.length > 0 && this.metrics.gpuProbes[this.metrics.gpuProbes.length - 1] < 6.0;
                const mediaOk = !this.detected;
                const extensionOk = this.suspiciousExtensions.size === 0;
                
                this.updateMetric('frameMetric', frameOk ? 'OK' : 'SLOW', frameOk ? 'pass' : 'warn');
                this.updateMetric('gpuMetric', gpuOk ? 'OK' : 'SLOW', gpuOk ? 'pass' : 'warn');
                this.updateMetric('mediaMetric', mediaOk ? 'OK' : 'DETECTED', mediaOk ? 'pass' : 'fail');
                this.updateMetric('extensionMetric', extensionOk ? 'OK' : 'ISSUES', extensionOk ? 'pass' : 'warn');
            }
            
            updateMetric(elementId, value, className) {
                const element = document.getElementById(elementId);
                element.textContent = value;
                element.className = 'metric-value ' + className;
            }
            
            updateRenderTime(time) {
                document.getElementById('renderTime').textContent = time.toFixed(1) + 'ms';
            }
            
            updateStatus(text, className = 'secure') {
                const statusElement = document.getElementById('status');
                statusElement.textContent = text;
                statusElement.className = 'status-value ' + className;
            }
            
            hideLoadingScreen() {
                setTimeout(() => {
                    document.getElementById('loadingScreen').style.display = 'none';
                    this.updateStatus('Active', 'secure');
                    document.getElementById('userId').textContent = this.userId;
                    document.getElementById('sessionId').textContent = this.sessionId;
                }, 1000);
            }
            
            startRenderLoop() {
                const startTime = performance.now();
                
                const render = (currentTime) => {
                    const renderStart = performance.now();
                    
                    if (this.blackout || this.detected || !this.decryptionKeyValid) {
                        this.gl.clearColor(0, 0, 0, 1);
                        this.gl.clear(this.gl.COLOR_BUFFER_BIT);
                    } else {
                        this.renderFrame(currentTime - startTime);
                    }
                    
                    this.renderOverlay();
                    
                    const renderEnd = performance.now();
                    this.metrics.renderTimes.push(renderEnd - renderStart);
                    if (this.metrics.renderTimes.length > 60) {
                        this.metrics.renderTimes.shift();
                    }
                    
                    this.metrics.frameCount++;
                    if (this.metrics.frameCount % 60 === 0) {
                        this.updateMetrics();
                    }
                    
                    requestAnimationFrame(render);
                };
                
                requestAnimationFrame(render);
            }
            
            renderFrame(time) {
                this.gl.useProgram(this.program);
                
                // Update uniforms
                if (this.uniforms.u_time !== null) {
                    this.gl.uniform1f(this.uniforms.u_time, time / 1000);
                }
                
                if (this.uniforms.u_encryptionKey !== null) {
                    const keyValue = parseInt(this.encryptionKey.substr(0, 2), 16) / 255.0;
                    this.gl.uniform1f(this.uniforms.u_encryptionKey, keyValue);
                }
                
                // Watermark position
                const wmX = 0.82 + Math.sin(time * 0.0007) * 0.03;
                const wmY = 0.88 + Math.cos(time * 0.00053) * 0.02;
                
                if (this.uniforms.u_wmPos !== null) {
                    this.gl.uniform2f(this.uniforms.u_wmPos, wmX, wmY);
                }
                
                if (this.uniforms.u_wmAlpha !== null) {
                    this.gl.uniform1f(this.uniforms.u_wmAlpha, 0.5);
                }
                
                if (this.uniforms.u_wmColor !== null) {
                    this.gl.uniform3f(this.uniforms.u_wmColor, 1.0, 0.15, 0.15);
                }
                
                // Render video if available
                if (this.video && this.video.readyState >= 2) {
                    this.gl.bindTexture(this.gl.TEXTURE_2D, this.texture);
                    this.gl.texImage2D(this.gl.TEXTURE_2D, 0, this.gl.RGBA, this.gl.RGBA, this.gl.UNSIGNED_BYTE, this.video);
                }
                
                this.gl.drawArrays(this.gl.TRIANGLE_STRIP, 0, 4);
            }
            
            renderOverlay() {
                const dpr = window.devicePixelRatio || 1;
                const ctx = this.overlayCtx;
                
                ctx.clearRect(0, 0, this.overlay.width, this.overlay.height);
                ctx.save();
                ctx.scale(dpr, dpr);
                
                // Watermark text
                ctx.font = '16px system-ui, Arial';
                ctx.fillStyle = 'rgba(255, 255, 255, 0.1)';
                ctx.textAlign = 'right';
                ctx.textBaseline = 'top';
                
                const watermarkText = `${this.userId} • SECURE • ${new Date().toLocaleString()}`;
                ctx.fillText(watermarkText, window.innerWidth - 20, 20);
                
                // Encryption status indicator
                const statusColor = this.decryptionKeyValid ? 'rgba(0, 255, 136, 0.3)' : 'rgba(255, 51, 102, 0.3)';
                ctx.fillStyle = statusColor;
                ctx.fillRect(window.innerWidth - 160, window.innerHeight - 50, 150, 25);
                
                ctx.fillStyle = '#ffffff';
                ctx.font = '12px monospace';
                ctx.textAlign = 'left';
                ctx.fillText(this.decryptionKeyValid ? 'ENCRYPTED' : 'DECRYPTED', window.innerWidth - 155, window.innerHeight - 38);
                
                ctx.restore();
            }
        }
        
        // Initialize the system when the page loads
        window.addEventListener('load', () => {
            new SecureStream();
        });
    </script>
</body>
</html>
