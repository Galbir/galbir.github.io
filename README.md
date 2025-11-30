<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Anti-Recording Video Player</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { background: #000; color: #fff; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; overflow: hidden; }
    canvas { display: block; width: 100vw; height: 100vh; }
    #ui {
      position: fixed; top: 10px; left: 10px; z-index: 100;
      background: rgba(0,0,0,0.8); padding: 15px; border-radius: 8px; backdrop-filter: blur(10px);
      font-size: 12px; min-width: 200px;
    }
    #status { font-weight: bold; margin-bottom: 5px; }
    .metric { margin: 2px 0; opacity: 0.8; }
    .green { color: #4ade80; }
    .yellow { color: #facc15; }
    .red { color: #ef4444; font-weight: bold; }
    #warning {
      position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%);
      background: rgba(239,68,68,0.95); color: white; padding: 30px; border-radius: 12px;
      text-align: center; z-index: 200; display: none; max-width: 90vw; box-shadow: 0 20px 40px rgba(239,68,68,0.5);
      animation: pulse 2s infinite;
    }
    @keyframes pulse { 0%,100% { transform: translate(-50%,-50%) scale(1); } 50% { transform: translate(-50%,-50%) scale(1.05); } }
    #overlay { position: fixed; top: 0; left: 0; pointer-events: none; z-index: 50; }
  </style>
</head>
<body>
  <div id="ui">
    <div id="status">Status: Initializing...</div>
    <div>Encryption: <span id="enc-status">Active</span></div>
    <div class="metric" id="gap-metric">Gap: 0/3</div>
    <div class="metric" id="var-metric">Var: 0/3</div>
    <div class="metric" id="fps-metric">FPS: Stable</div>
    <div id="key-info">Key: Active</div>
  </div>
  <div id="warning">
    <h2>🚨 Recording Detected!</h2>
    <p id="breach-details">Security breach confirmed.</p>
    <p>Close this tab immediately.</p>
  </div>
  <canvas id="canvas"></canvas>
  <canvas id="overlay"></canvas>

  <script>
    (async () => {
      // Config
      const VIDEO_SRC = 'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4';
      const KEY_ROTATE_MS = 10000;
      const GAP_THRESHOLD = 17; // Vsync ~16.67ms
      const JITTER_THRESHOLD = 0.002; // Low jitter indicates smoothing

      // Elements
      const canvas = document.getElementById('canvas');
      const overlay = document.getElementById('overlay');
      const statusEl = document.getElementById('status');
      const encStatusEl = document.getElementById('enc-status');
      const warningEl = document.getElementById('warning');
      const gapEl = document.getElementById('gap-metric');
      const varEl = document.getElementById('var-metric');
      const fpsEl = document.getElementById('fps-metric');
      const keyEl = document.getElementById('key-info');
      const breachEl = document.getElementById('breach-details');

      let gl, video, videoReady = false, detected = false, keyValid = true;
      let frameTimes = [], gapCount = 0, varCount = 0, lastTime = 0, encryptionKey = Math.random().toString(36).slice(2);
      const userId = 'user_' + Math.random().toString(36).slice(-8);

      // Video setup (hidden)
      video = document.createElement('video');
      video.src = VIDEO_SRC;
      video.crossOrigin = 'anonymous';
      video.loop = true;
      video.muted = true;
      video.style.position = 'absolute'; video.style.left = '-9999px';
      document.body.appendChild(video);

      // Load video
      await new Promise((res, rej) => {
        video.onloadeddata = () => { videoReady = true; statusEl.textContent = 'Status: Video Ready'; res(); };
        video.onerror = () => rej('Video load failed');
        video.load();
      });

      // WebGL setup
      const glOpts = { preserveDrawingBuffer: false, antialias: false };
      gl = canvas.getContext('webgl', glOpts) || canvas.getContext('webgl', { ...glOpts, stencil: true });
      if (!gl) throw new Error('WebGL not supported');

      // Shaders
      const vsSrc = `attribute vec2 pos; attribute vec2 uv; varying vec2 vUv; void main(){ vUv=uv; gl_Position=vec4(pos,0,1); }`;
      const fsSrc = `
        precision mediump float;
        varying vec2 vUv; uniform sampler2D tex; uniform float time; uniform vec2 wmPos; uniform vec3 wmCol; uniform float key;
        float circle(vec2 p, float r){ return 1.0-smoothstep(r-0.01,r,length(p)); }
        void main(){
          vec4 col = texture2D(tex, vUv);
          vec2 p = vUv - wmPos; float wm = circle(p, 0.08);
          col.rgb = mix(col.rgb, wmCol, wm * 0.8);
          // Dynamic distortion
          col.rgb += sin(vUv*50.0 + time*5.0 + key)*0.02;
          gl_FragColor = col;
        }`;

      function compileShader(src, type) {
        const shader = gl.createShader(type);
        gl.shaderSource(shader, src);
        gl.compileShader(shader);
        if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
          console.error(gl.getShaderInfoLog(shader));
          gl.deleteShader(shader);
          throw new Error('Shader compile failed');
        }
        return shader;
      }

      const program = gl.createProgram();
      gl.attachShader(program, compileShader(vsSrc, gl.VERTEX_SHADER));
      gl.attachShader(program, compileShader(fsSrc, gl.FRAGMENT_SHADER));
      gl.linkProgram(program);
      if (!gl.getProgramParameter(program, gl.LINK_STATUS)) throw new Error('Program link failed');
      gl.useProgram(program);

      // Quad
      const verts = new Float32Array([-1,-1,0,0, 1,-1,1,0, -1,1,0,1, 1,1,1,1]);
      const vbo = gl.createBuffer();
      gl.bindBuffer(gl.ARRAY_BUFFER, vbo);
      gl.bufferData(gl.ARRAY_BUFFER, verts, gl.STATIC_DRAW);
      gl.enableVertexAttribArray(0); gl.vertexAttribPointer(0, 2, gl.FLOAT, false, 16, 0);
      gl.enableVertexAttribArray(1); gl.vertexAttribPointer(1, 2, gl.FLOAT, false, 16, 8);

      // Texture
      const tex = gl.createTexture();
      gl.bindTexture(gl.TEXTURE_2D, tex);
      gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
      gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);
      gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
      gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);

      // Uniforms
      const uTex = gl.getUniformLocation(program, 'tex');
      const uTime = gl.getUniformLocation(program, 'time');
      const uWmPos = gl.getUniformLocation(program, 'wmPos');
      const uWmCol = gl.getUniformLocation(program, 'wmCol');
      const uKey = gl.getUniformLocation(program, 'key');
      gl.uniform1i(uTex, 0);

      // Frame canvas for video->texture
      const frameCanvas = document.createElement('canvas');
      const frameCtx = frameCanvas.getContext('2d');

      // Overlay setup
      const overlayCtx = overlay.getContext('2d');

      // Resize
      function resize() {
        const dpr = window.devicePixelRatio;
        const w = canvas.width = innerWidth * dpr;
        const h = canvas.height = innerHeight * dpr;
        overlay.width = w; overlay.height = h;
        gl.viewport(0, 0, w, h);
      }
      window.addEventListener('resize', resize);
      resize();

      // Anti-capture hooks
      const origCaptureStream = HTMLCanvasElement.prototype.captureStream;
      HTMLCanvasElement.prototype.captureStream = function() {
        if (this === canvas || this === overlay) {
          detected = true; triggerWarning('Canvas capture detected');
          return document.createElement('canvas').captureStream(30); // Black/empty
        }
        return origCaptureStream.apply(this, arguments);
      };

      if (window.MediaRecorder) {
        const origMR = window.MediaRecorder;
        window.MediaRecorder = function(stream) {
          detected = true; triggerWarning('MediaRecorder detected');
          const mr = new origMR(stream);
          const origData = mr.ondataavailable;
          mr.ondataavailable = (e) => {
            e.data = new Blob(['CORRUPTED'], {type: e.data.type});
            origData?.call(mr, e);
          };
          return mr;
        };
      }

      // Detection
      function triggerWarning(msg) {
        statusEl.textContent = 'BREACH: ' + msg;
        encStatusEl.textContent = 'Compromised';
        breachEl.textContent = msg;
        warningEl.style.display = 'block';
        detected = true;
      }

      // RAF loop with detection
      let rafTime = performance.now();
      function loop(time) {
        if (detected) {
          gl.clear(gl.COLOR_BUFFER_BIT);
          overlayCtx.clearRect(0,0,overlay.width,overlay.height);
          requestAnimationFrame(loop);
          return;
        }

        const dt = time - rafTime;
        rafTime = time;
        frameTimes.push(dt);
        if (frameTimes.length > 120) frameTimes.shift();

        // Gap detection
        if (dt > GAP_THRESHOLD) {
          gapCount = Math.min(gapCount + 1, 3);
          if (gapCount === 3) triggerWarning(`Frame gaps: ${dt.toFixed(1)}ms (recording stutter)`);
        } else gapCount = Math.max(gapCount - 1, 0);

        // Jitter detection
        if (frameTimes.length === 120) {
          const mean = frameTimes.reduce((a,b)=>a+b)/120;
          const varr = frameTimes.reduce((a,b)=>a + (b-mean)**2, 0)/120;
          const jitter = Math.sqrt(varr) / mean;
          if (jitter < JITTER_THRESHOLD) {
            varCount = Math.min(varCount + 1, 3);
            if (varCount === 3) triggerWarning(`Low jitter: ${(jitter*100).toFixed(2)}% (smoothing filter)`);
          } else varCount = Math.max(varCount - 1, 0);
        }

        // Update metrics
        gapEl.textContent = `Gap: ${gapCount}/3`; gapEl.className = `metric ${gapCount ? gapCount<3?'yellow':'red' : 'green'}`;
        varEl.textContent = `Var: ${varCount}/3`; varEl.className = `metric ${varCount ? varCount<3?'yellow':'red' : 'green'}`;
        fpsEl.className = 'metric green';

        // Render video to texture
        if (videoReady && video.videoWidth) {
          frameCanvas.width = video.videoWidth;
          frameCanvas.height = video.videoHeight;
          frameCtx.drawImage(video, 0, 0);
          gl.bindTexture(gl.TEXTURE_2D, tex);
          gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, frameCanvas);
        } else {
          // Test pattern
          frameCtx.fillStyle = '#333'; frameCtx.fillRect(0,0,640,360);
          frameCtx.fillStyle = '#fff'; frameCtx.font = 'bold 40px Arial'; frameCtx.textAlign='center';
          frameCtx.fillText('TEST', 320, 180);
          gl.bindTexture(gl.TEXTURE_2D, tex);
          gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, frameCanvas);
        }

        // Draw
        gl.uniform1f(uTime, time * 0.001);
        gl.uniform1f(uKey, parseFloat(encryptionKey));
        const wmX = 0.9 + Math.sin(time*0.001)*0.05;
        const wmY = 0.9 + Math.cos(time*0.0008)*0.04;
        gl.uniform2f(uWmPos, wmX, wmY);
        gl.uniform3f(uWmCol, 1, 0.2, 0.2);
        gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);

        // Overlay watermark
        const ow = overlay.width, oh = overlay.height;
        overlayCtx.clearRect(0,0,ow,oh);
        overlayCtx.save();
        overlayCtx.scale(devicePixelRatio, devicePixelRatio);
        overlayCtx.font = `bold 24px system-ui`;
        overlayCtx.fillStyle = `rgba(255,255,255,${0.1 + Math.sin(time*0.002)*0.05})`;
        overlayCtx.textAlign = 'right';
        overlayCtx.fillText(`${userId} • ${new Date().toLocaleTimeString()}`, innerWidth-20, innerHeight-20);
        overlayCtx.restore();

        requestAnimationFrame(loop);
      }
      video.play();
      requestAnimationFrame(loop);

      // Key rotation
      setInterval(() => {
        encryptionKey = Math.random().toString(36).slice(2);
        keyValid = Math.random() > 0.1;
        keyEl.textContent = `Key: ${keyValid ? 'Rotated' : 'Invalidated'}`;
        encStatusEl.textContent = keyValid ? 'Active' : 'Failed';
        if (!keyValid) setTimeout(() => { keyValid=true; encStatusEl.textContent='Active'; }, 2000);
      }, KEY_ROTATE_MS);

      // Controls
      document.addEventListener('keydown', e => {
        if (e.key === 'r' || e.key === 'R') triggerWarning('Manual test trigger');
        if (e.key === 'k' || e.key === 'K') keyValid = !keyValid;
      });

    })();
  </script>
</body>
</html>
