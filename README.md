<html>
<head>
<meta charset="utf-8"/>
<title>Enhanced Anti-Recording & Encryption System</title>
<style>
  html,body { height:100%; margin:0; background:#000; color:#fff; font-family:Arial,Helvetica,sans-serif; }
  #ui { position:fixed; left:12px; top:12px; z-index:50; background:rgba(0,0,0,0.45); padding:10px; border-radius:8px; }
  #status { color:#ff6b6b; margin-top:6px; }
  #encryption-status { color:#4ecdc4; margin-top:4px; font-size:12px; }
  canvas { display:block; width:100vw; height:100vh; }
  #warning {
    position:fixed;
    left:50%;
    top:50%;
    transform:translate(-50%,-50%);
    background:linear-gradient(135deg, rgba(139,0,0,0.95) 0%, rgba(220,20,60,0.95) 100%);
    color:#fff;
    padding:24px;
    border-radius:15px;
    font-size:18px;
    display:none;
    z-index:60;
    box-shadow:0 10px 30px rgba(255,0,0,0.5);
    border:2px solid rgba(255,255,255,0.2);
    animation:warningPulse 2s ease-in-out infinite;
    max-width:90vw;
    max-height:90vh;
    overflow-y:auto;
  }
  
  #device-warning {
    position:fixed;
    left:50%;
    top:50%;
    transform:translate(-50%,-50%);
    background:linear-gradient(135deg, rgba(0,123,255,0.95) 0%, rgba(0,86,179,0.95) 100%);
    color:#fff;
    padding:24px;
    border-radius:15px;
    font-size:18px;
    display:none;
    z-index:60;
    box-shadow:0 10px 30px rgba(0,123,255,0.5);
    border:2px solid rgba(255,255,255,0.2);
    animation:deviceWarningPulse 2s ease-in-out infinite;
    max-width:90vw;
    max-height:90vh;
    overflow-y:auto;
    text-align:center;
  }
  
  @keyframes deviceWarningPulse {
    0%, 100% {
      box-shadow:0 10px 30px rgba(0,123,255,0.5);
      transform:translate(-50%,-50%) scale(1);
    }
    50% {
      box-shadow:0 15px 40px rgba(0,123,255,0.8);
      transform:translate(-50%,-50%) scale(1.02);
    }
  }
  
  @keyframes warningPulse {
    0%, 100% {
      box-shadow:0 10px 30px rgba(255,0,0,0.5);
      transform:translate(-50%,-50%) scale(1);
    }
    50% {
      box-shadow:0 15px 40px rgba(255,0,0,0.8);
      transform:translate(-50%,-50%) scale(1.02);
    }
  }
  
  #warning:hover {
    animation:warningPulse 1s ease-in-out infinite;
  }
  #extension-warning { position:fixed; right:12px; top:12px; z-index:55; background:rgba(255,0,0,0.2); padding:8px; border-radius:6px; font-size:12px; display:none; }
  #loading { position:fixed; left:50%; top:50%; transform:translate(-50%,-50%); background:rgba(0,0,0,0.9); padding:20px; border-radius:10px; z-index:70; }
  .spinner { border:3px solid #333; border-top:3px solid #4ecdc4; border-radius:50%; width:30px; height:30px; animation:spin 1s linear infinite; margin:0 auto 10px; }
  @keyframes spin { 0% { transform:rotate(0deg); } 100% { transform:rotate(360deg); } }
  
  #whitelist-btn {
    background: rgba(78, 205, 196, 0.3);
    border: 1px solid rgba(78, 205, 196, 0.5);
    color: #4ecdc4;
    padding: 4px 8px;
    border-radius: 4px;
    font-size: 10px;
    cursor: pointer;
    margin-top: 6px;
  }
  
  #whitelist-btn:hover {
    background: rgba(78, 205, 196, 0.5);
  }

  #metrics {
    margin-top: 8px;
    font-size: 10px;
    color: #aaa;
    max-height: 120px;
    overflow-y: auto;
  }
  .metric { margin: 2px 0; }
  .metric.green { color: #4ecdc4; }
  .metric.yellow { color: #ffaa44; }
  .metric.red { color: #ff6b6b; font-weight: bold; }
</style>
</head>
<body>
  <div id="loading">
    <div class="spinner"></div>
    <div>Initializing Security System...</div>
  </div>
  
  <div id="ui">
    <div><strong>Enhanced Anti-Recording System</strong></div>
    <div id="info">User: <span id="userId">secure_user</span></div>
    <div id="status">Status: <span id="stat">Initializing</span></div>
    <div id="encryption-status">Encryption: <span id="encStat">Active</span></div>
    <div style="margin-top:6px;font-size:12px;color:#aaa">Key rotation: <span id="keyI">8</span>s</div>
    <div style="margin-top:4px;font-size:12px;color:#aaa">Extensions blocked: <span id="extCount">0</span></div>
    <div id="extension-list" style="margin-top:8px;font-size:11px;color:#ff6b6b;max-height:60px;overflow-y:auto;display:none;">
      <div style="font-weight:bold;margin-bottom:4px;">Detected Extensions:</div>
      <div id="extension-details"></div>
      <button id="whitelist-btn">Whitelist Current Extensions</button>
    </div>
    <div id="metrics">
      <div style="font-weight:bold; margin-bottom:4px; color:#4ecdc4;">📊 Detection Metrics</div>
      <div class="metric" id="gap-metric">Gap: 0/3</div>
      <div class="metric" id="fps-metric">FPS: 0/3</div>
      <div class="metric" id="var-metric">Var: 0/3</div>
      <div class="metric" id="gpu-metric">GPU: 0/3</div>
      <div class="metric" id="dt-metric">DT: -- ms</div>
      <div class="metric" id="stddev-metric">StdDev: -- ms</div>
      <div class="metric" id="gpuavg-metric">GPU Avg: -- ms</div>
    </div>
  </div>
  
  <div id="extension-warning">⚠ Suspicious extensions detected</div>
  <div id="device-warning">
    <div style="text-align:center; margin-bottom:15px;">
      <div style="font-size:28px; font-weight:bold; color:#4dabf7; margin-bottom:10px; text-shadow:2px 2px 4px rgba(0,0,0,0.5);">📱 DEVICE DETECTED</div>
      <div style="font-size:18px; color:#74c0fc; margin-bottom:8px; font-weight:500;">Non-Desktop Device Detected</div>
    </div>
    
    <div style="background:rgba(0,123,255,0.15); padding:16px; border-radius:8px; margin-bottom:16px; font-size:14px; border:1px solid rgba(255,255,255,0.1);">
      <div style="font-weight:bold; color:#339af0; margin-bottom:10px; font-size:16px;">📋 Device Information:</div>
      <div id="device-info" style="color:#ffffff; line-height:1.4;">
        Loading device information...
      </div>
    </div>
    
    <div style="font-size:12px; color:#a5d8ff; margin-top:12px; padding-top:8px; border-top:1px solid rgba(255,255,255,0.1);">
      This system is optimized for desktop browsers. Mobile/tablet access may have limited functionality.
    </div>
  </div>
  <div id="warning">
    <div style="text-align:center; margin-bottom:15px;">
      <div style="font-size:28px; font-weight:bold; color:#ff4444; margin-bottom:10px; text-shadow:2px 2px 4px rgba(0,0,0,0.5);">🚨 SECURITY BREACH</div>
      <div style="font-size:18px; color:#ffaa44; margin-bottom:8px; font-weight:500;">Recording/Tamper Detected</div>
    </div>
    
    <!-- Enhanced breach details with severity-based styling -->
    <div id="breach-details" style="background:rgba(255,0,0,0.15); padding:16px; border-radius:8px; margin-bottom:16px; font-size:14px; border:1px solid rgba(255,255,255,0.1);">
      <div style="font-weight:bold; color:#ff6666; margin-bottom:10px; font-size:16px;">🔍 Detection Details:</div>
      
      <div style="display:grid; grid-template-columns:1fr 1fr; gap:8px; margin-bottom:12px;">
        <div>
          <div style="font-size:12px; color:#aaa; margin-bottom:2px;">TYPE</div>
          <div id="breach-type" style="color:#ff8888; font-weight:500;">Type: Unknown</div>
        </div>
        <div>
          <div style="font-size:12px; color:#aaa; margin-bottom:2px;">SEVERITY</div>
          <div id="breach-severity" style="color:#ff8888; font-weight:500;">Severity: Unknown</div>
        </div>
      </div>
      
      <div style="display:grid; grid-template-columns:1fr 1fr; gap:8px;">
        <div>
          <div style="font-size:12px; color:#aaa; margin-bottom:2px;">SOURCE</div>
          <div id="breach-source" style="color:#ff8888; font-weight:500;">Source: Unknown</div>
        </div>
        <div>
          <div style="font-size:12px; color:#aaa; margin-bottom:2px;">TIME</div>
          <div id="breach-time" style="color:#ff8888; font-weight:500;">Time: Unknown</div>
        </div>
      </div>
    </div>
    
    <!-- Enhanced action recommendations -->
    <div id="breach-actions" style="background:rgba(255,255,255,0.05); padding:12px; border-radius:6px; margin-bottom:16px; font-size:13px;">
      <div style="font-weight:bold; color:#ffaa44; margin-bottom:8px;">🛡️ Recommended Actions:</div>
      <div id="action-recommendations" style="color:#ffcccc; line-height:1.4;">
        • Close this browser session immediately<br>
        • Check for suspicious browser extensions<br>
        • Monitor for unusual system activity<br>
        • Consider changing security credentials
      </div>
    </div>
    
    <!-- Enhanced risk assessment -->
    <div id="risk-assessment" style="text-align:center; font-size:12px; color:#ffaaaa; padding:8px; background:rgba(255,0,0,0.1); border-radius:4px;">
      <strong>Risk Level:</strong> <span id="risk-level">HIGH</span> - This session has been compromised and encrypted data may be at risk
    </div>
    
    <!-- Recovery information -->
    <div style="text-align:center; font-size:11px; color:#ff8888; margin-top:12px; padding-top:8px; border-top:1px solid rgba(255,255,255,0.1);">
      Session ID: <span id="session-id" style="font-family:monospace; color:#ff6666;">SECURE-0000</span>
    </div>
  </div>
  <canvas id="glcanvas"></canvas>

<script>
(async function(){
  // ========== ENHANCED CONFIGURATION ==========
  const VIDEO_URL = "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4";
  const FALLBACK_VIDEO_URL = "https://samplelib.com/lib/preview/mp4/sample-10s.mp4";
  const USER_ID = "secure_user_" + Math.random().toString(36).substr(2, 9);
  const KEY_ROTATE_SECONDS = 8;
  const FRAME_GAP_THRESHOLD_MS = 50; // Tuned for OBS: balanced sensitivity
  const ENCRYPTION_KEY = generateEncryptionKey();
  let videoReady = false;
  
  // WHITELIST APPROACH - Known safe extensions
  const EXTENSION_WHITELIST = [
    // Password Managers
    'lastpass', 'bitwarden', '1password', 'dashlane', 'roboform', 'keeper',
    // Ad Blockers & Privacy
    'ublock', 'adblock', 'privacy badger', 'ghostery', 'duckduckgo', 'privacy',
    // Developer Tools
    'react developer', 'redux devtools', 'vue.js devtools', 'angular',
    // Productivity
    'grammarly', 'evernote', 'pocket', 'notion', 'save to', 'onenote',
    // Browser Enhancement
    'dark reader', 'tampermonkey', 'greasemonkey', 'video speed', 'stylus',
    // Communication
    'google meet', 'zoom', 'teams', 'slack', 'discord', 'whatsapp',
    // Shopping & Utilities
    'honey', 'rakuten', 'translate', 'dictionary'
  ];

  const EXTENSION_BLACKLIST = [
    'chrome capture', 'screenshot', 'gif recorder', 'screen capture',
    'recorder', 'nimbus', 'lighthshot', 'awesome screenshot', 'capture'
  ];
  // ===========================================

  const canvas = document.getElementById('glcanvas');
  const statEl = document.getElementById('stat');
  const encStatEl = document.getElementById('encStat');
  const warningEl = document.getElementById('warning');
  const extWarningEl = document.getElementById('extension-warning');
  const wmTextEl = document.getElementById('wmText');
  const userIdEl = document.getElementById('userId');
  const keyIntervalEl = document.getElementById('keyI');
  const extCountEl = document.getElementById('extCount');
  const loadingEl = document.getElementById('loading');
  const breachTypeEl = document.getElementById('breach-type');
  const breachSourceEl = document.getElementById('breach-source');
  const breachTimeEl = document.getElementById('breach-time');
  const breachSeverityEl = document.getElementById('breach-severity');
  const actionRecommendationsEl = document.getElementById('action-recommendations');
  const riskLevelEl = document.getElementById('risk-level');
  const sessionIdEl = document.getElementById('session-id');
  const deviceWarningEl = document.getElementById('device-warning');
  const deviceInfoEl = document.getElementById('device-info');
  const whitelistBtn = document.getElementById('whitelist-btn');
  
  keyIntervalEl.textContent = KEY_ROTATE_SECONDS;
  userIdEl.textContent = USER_ID;

  // Enhanced device detection and warning
  function detectDevice() {
    const userAgent = navigator.userAgent.toLowerCase();
    const isMobile = /android|webos|iphone|ipad|ipod|blackberry|iemobile|opera mini/i.test(userAgent);
    const isTablet = /ipad|android(?!.*mobile)/i.test(userAgent);
    
    // Additional detection methods for desktop site mode
    const isTouchDevice = 'ontouchstart' in window || navigator.maxTouchPoints > 0;
    
    const isMobileScreen = screen.width <= 768 || screen.height <= 768;
    const isMobilePlatform = /android|iphone|ipad|ipod|mobile/i.test(navigator.platform.toLowerCase());
    
    // Check if device characteristics suggest mobile even in desktop mode
    const isMobileByCharacteristics = isTouchDevice || isMobileScreen || isMobilePlatform;
    
    // Show warning if any mobile indicator is detected
    if (isMobile || isTablet || isMobileByCharacteristics) {
      deviceWarningEl.style.display = 'block';
      
      // Determine device type more accurately
      let deviceType = 'Desktop';
      if (isMobile) deviceType = 'Mobile';
      else if (isTablet) deviceType = 'Tablet';
      else if (isTouchDevice) deviceType = 'Touch Device';
     
      else if (isMobileScreen) deviceType = 'Mobile Screen';
      else if (isMobilePlatform) deviceType = 'Mobile Platform';
      
      // Get device information
      const deviceInfo = {
        type: deviceType,
        userAgent: navigator.userAgent,
        screenResolution: `${screen.width}x${screen.height}`,
        viewportSize: `${window.innerWidth}x${window.innerHeight}`,
        platform: navigator.platform,
        language: navigator.language,
        online: navigator.onLine,
        touchDevice: isTouchDevice,
        mobileScreen: isMobileScreen,
        mobilePlatform: isMobilePlatform
      };
      
      // Display device information
      deviceInfoEl.innerHTML = `
        <div><strong>Device Type:</strong> ${deviceInfo.type}</div>
        <div><strong>User Agent:</strong> ${deviceInfo.userAgent}</div>
        <div><strong>Screen Resolution:</strong> ${deviceInfo.screenResolution}</div>
        <div><strong>Viewport Size:</strong> ${deviceInfo.viewportSize}</div>
        <div><strong>Platform:</strong> ${deviceInfo.platform}</div>
        <div><strong>Language:</strong> ${deviceInfo.language}</div>
        <div><strong>Online Status:</strong> ${deviceInfo.online ? 'Online' : 'Offline'}</div>
        <div><strong>Touch Support:</strong> ${deviceInfo.touchDevice ? 'Yes' : 'No'}</div>
        <div><strong>Mobile Indicators:</strong> ${deviceInfo.mobileScreen || deviceInfo.mobilePlatform ? 'Detected' : 'None'}</div>
      `;
    }
  }

  // Run device detection
  detectDevice();

  // Hide loading screen
  setTimeout(() => { loadingEl.style.display = 'none'; }, 2000);

  // Enhanced encryption system
  function generateEncryptionKey() {
    const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/';
    let key = '';
    for (let i = 0; i < 32; i++) {
      key += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    return key;
  }

  function encryptData(data, key) {
    // Simple XOR encryption for demo purposes
    let encrypted = '';
    for (let i = 0; i < data.length; i++) {
      encrypted += String.fromCharCode(data.charCodeAt(i) ^ key.charCodeAt(i % key.length));
    }
    return btoa(encrypted);
  }

  function decryptData(encryptedData, key) {
    try {
      const decoded = atob(encryptedData);
      let decrypted = '';
      for (let i = 0; i < decoded.length; i++) {
        decrypted += String.fromCharCode(decoded.charCodeAt(i) ^ key.charCodeAt(i % key.length));
      }
      return decrypted;
    } catch (e) {
      return null;
    }
  }

  // Chrome extension detection and blocking with WHITELIST approach
  class ExtensionDetector {
    constructor() {
      this.detectedExtensions = new Set();
      this.whitelistedExtensions = new Set();
      this.monitoringInterval = null;
      this.loadWhitelistedExtensions();
    }

    loadWhitelistedExtensions() {
      // Load previously whitelisted extensions from sessionStorage
      try {
        const whitelisted = sessionStorage.getItem('extension_whitelist');
        if (whitelisted) {
          const parsed = JSON.parse(whitelisted);
          parsed.forEach(ext => this.whitelistedExtensions.add(ext));
        }
      } catch (e) {
        // Ignore errors
      }
    }

    saveWhitelistedExtensions() {
      try {
        sessionStorage.setItem('extension_whitelist', JSON.stringify(Array.from(this.whitelistedExtensions)));
      } catch (e) {
        // Ignore errors
      }
    }

    whitelistCurrentExtensions() {
      const current = Array.from(this.detectedExtensions);
      current.forEach(ext => this.whitelistedExtensions.add(ext));
      this.detectedExtensions.clear();
      this.saveWhitelistedExtensions();
      console.log('Whitelisted extensions:', current);
    }

    startMonitoring() {
      this.checkForExtensions();
      this.monitoringInterval = setInterval(() => {
        this.checkForExtensions();
      }, 3000);
    }

    stopMonitoring() {
      if (this.monitoringInterval) {
        clearInterval(this.monitoringInterval);
      }
    }

    checkForExtensions() {
      // Check for common extension patterns in localStorage, sessionStorage, cookies
      this.checkStorage();
      this.checkBrowserAPIs();
      this.checkUserAgent();
      this.checkForRecordingExtensions();
      this.checkForChromeCaptureExtension();
    }

    checkStorage() {
      const storageTypes = ['localStorage', 'sessionStorage'];
      storageTypes.forEach(type => {
        try {
          const storage = window[type];
          for (let i = 0; i < storage.length; i++) {
            const key = storage.key(i);
            if (this.containsBlacklistKeywords(key)) {
              this.detectedExtensions.add(`${type}:${key}`);
            }
          }
        } catch (e) {
          // Ignore access errors
        }
      });
    }

    checkBrowserAPIs() {
      // Check for suspicious API usage patterns
      const suspiciousAPIs = [
        'chrome.runtime', 'chrome.extension', 'chrome.tabs', 'chrome.webRequest',
        'browser.runtime', 'browser.extension', 'browser.tabs'
      ];

      suspiciousAPIs.forEach(api => {
        try {
          if (window[api] || window.chrome?.[api.split('.')[1]]) {
            // Check if this is likely a whitelisted extension
            const isWhitelisted = this.checkWhitelistedExtension(api);
            if (!isWhitelisted && !this.whitelistedExtensions.has(`API:${api}`)) {
              this.detectedExtensions.add(`API:${api}`);
            }
          }
        } catch (e) {
          // Ignore access errors
        }
      });
      
      // Check for Chrome extension specific APIs that capture extensions use
      const captureAPIs = [
        'chrome.tabs.captureVisibleTab',
        'chrome.desktopCapture.chooseDesktopMedia',
        'chrome.mediaDevices.getDisplayMedia'
      ];
      
      captureAPIs.forEach(api => {
        try {
          if (window.chrome?.[api.split('.')[1]]?.[api.split('.')[2]]) {
            this.detectedExtensions.add(`CaptureAPI:${api}`);
          }
        } catch (e) {
          // Ignore access errors
        }
      });
    }

    checkWhitelistedExtension(apiCall) {
      // Check if this API usage matches known whitelisted extension patterns
      try {
        // Method 1: Check performance entries for extension scripts
        if (performance.getEntriesByType) {
          const entries = performance.getEntriesByType('resource');
          for (const entry of entries) {
            const url = entry.name.toLowerCase();
            if (EXTENSION_WHITELIST.some(whitelist => url.includes(whitelist))) {
              return true;
            }
          }
        }

        // Method 2: Check for known extension patterns in DOM
        const allScripts = document.querySelectorAll('script');
        for (const script of allScripts) {
          const src = (script.src || '').toLowerCase();
          if (EXTENSION_WHITELIST.some(whitelist => src.includes(whitelist))) {
            return true;
          }
        }

        // Method 3: Check for extension-specific globals
        const whitelistGlobals = {
          'grammarly': 'grammarly',
          'lastpass': 'lastpass',
          'bitwarden': 'bitwarden',
          'ublock': 'ublock',
          'adblock': 'adblock',
          'honey': 'honey'
        };

        for (const [extension, global] of Object.entries(whitelistGlobals)) {
          if (window[global]) {
            return true;
          }
        }

        return false;
      } catch (e) {
        return false; // Default to suspicious if we can't verify
      }
    }

    checkUserAgent() {
      const ua = navigator.userAgent.toLowerCase();
      EXTENSION_BLACKLIST.forEach(keyword => {
        if (ua.includes(keyword.toLowerCase())) {
          this.detectedExtensions.add(`UA:${keyword}`);
        }
      });
    }

    checkForRecordingExtensions() {
      // Check for screen capture APIs being used in suspicious ways
      if (navigator.mediaDevices?.getDisplayMedia) {
        try {
          const originalGetDisplayMedia = navigator.mediaDevices.getDisplayMedia;
          let extensionDetected = false;
          
          // Temporarily override to detect calls
          navigator.mediaDevices.getDisplayMedia = function() {
            extensionDetected = true;
            return originalGetDisplayMedia.apply(this, arguments);
          };
          
          // Small delay to allow any potential extension calls
          setTimeout(() => {
            if (extensionDetected) {
              this.detectedExtensions.add('ScreenCaptureAPI');
            }
            // Restore original
            navigator.mediaDevices.getDisplayMedia = originalGetDisplayMedia;
          }, 100);
          
        } catch (e) {
          // Ignore errors in detection
        }
      }
    }

    checkForChromeCaptureExtension() {
      // Check for Chrome Capture extension specific patterns
      const extensionPatterns = [
        'chrome-capture',
        'screenshot',
        'gif recorder',
        'screen capture',
        'capture extension'
      ];
      
      // Check in various places where extension info might be stored
      extensionPatterns.forEach(pattern => {
        // Check in localStorage
        try {
          for (let i = 0; i < localStorage.length; i++) {
            const key = localStorage.key(i);
            if (key && (key.toLowerCase().includes(pattern) || localStorage.getItem(key)?.toLowerCase().includes(pattern))) {
              this.detectedExtensions.add(`Storage:${pattern}`);
            }
          }
        } catch (e) {}
        
        // Check in sessionStorage
        try {
          for (let i = 0; i < sessionStorage.length; i++) {
            const key = sessionStorage.key(i);
            if (key && (key.toLowerCase().includes(pattern) || sessionStorage.getItem(key)?.toLowerCase().includes(pattern))) {
              this.detectedExtensions.add(`Storage:${pattern}`);
            }
          }
        } catch (e) {}
        
        // Check in cookies
        try {
          const cookies = document.cookie.split(';');
          cookies.forEach(cookie => {
            const [name, value] = cookie.trim().split('=');
            if (name && (name.toLowerCase().includes(pattern) || value?.toLowerCase().includes(pattern))) {
              this.detectedExtensions.add(`Cookie:${pattern}`);
            }
          });
        } catch (e) {}
      });
    }

    containsBlacklistKeywords(text) {
      return EXTENSION_BLACKLIST.some(keyword => 
        text.toLowerCase().includes(keyword.toLowerCase())
      );
    }

    getDetectedCount() {
      return this.detectedExtensions.size;
    }

    getDetectedList() {
      return Array.from(this.detectedExtensions);
    }
  }

  // Initialize extension detector
  const extensionDetector = new ExtensionDetector();
  extensionDetector.startMonitoring();

  // Whitelist button handler
  whitelistBtn.addEventListener('click', () => {
    extensionDetector.whitelistCurrentExtensions();
    whitelistBtn.textContent = 'Whitelisted!';
    setTimeout(() => {
      whitelistBtn.textContent = 'Whitelist Current Extensions';
    }, 2000);
  });

  // Update extension count and list display
  setInterval(() => {
    const count = extensionDetector.getDetectedCount();
    const detectedList = extensionDetector.getDetectedList();
    
    extCountEl.textContent = count;
    
    // Update extension list
    const extensionListEl = document.getElementById('extension-list');
    const extensionDetailsEl = document.getElementById('extension-details');
    
    if (count > 0) {
      extWarningEl.style.display = 'block';
      extensionListEl.style.display = 'block';
      
      // Format extension details with whitelist info
      const formattedExtensions = detectedList.map(ext => {
        let displayName = ext;
        let isSuspicious = true;
        
        // Special handling for runtime APIs that might be whitelisted
        if (ext.startsWith('API:chrome.runtime') || ext.startsWith('API:browser.runtime')) {
          displayName = 'Browser Extension (Runtime API)';
          // Check if we should show as potentially safe
          if (extensionDetector.checkWhitelistedExtension('runtime')) {
            displayName += ' - Potentially Safe Extension';
            isSuspicious = false;
          }
        } else if (ext === 'ScreenCaptureAPI') {
          displayName = 'Screen Capture API (built-in browser capability)';
        } else {
          const [type, name] = ext.split(':');
          if (type === 'API') {
            displayName = name.replace('chrome.', '').replace('browser.', '');
          } else if (type === 'UA') {
            displayName = `User Agent: ${name}`;
          } else if (type === 'localStorage' || type === 'sessionStorage') {
            displayName = `${type}: ${name}`;
          }
        }
        
        return `• ${displayName} ${isSuspicious ? '🚨' : '⚠️'}`;
      }).join('<br>');
      
      extensionDetailsEl.innerHTML = formattedExtensions;
      
      // Only mark as detected if there are actual suspicious extensions
      const suspiciousCount = detectedList.filter(ext => {
        if (ext.startsWith('API:chrome.runtime') || ext.startsWith('API:browser.runtime')) {
          return !extensionDetector.checkWhitelistedExtension('runtime');
        }
        return true;
      }).length;
      
      if (suspiciousCount > 0) {
        markDetected(`Detected ${suspiciousCount} suspicious extensions`);
      }
      
      // Log to console for debugging
      console.log('Detected extensions:', detectedList);
    } else {
      extWarningEl.style.display = 'none';
      extensionListEl.style.display = 'none';
      extensionDetailsEl.innerHTML = '';
    }
  }, 1000);

  // Create video element
  const video = document.createElement('video');
  video.crossOrigin = "anonymous";
  video.muted = true;
  video.loop = true;
  video.playsInline = true;
  
  // Hide video completely - OBS cannot capture hidden elements
  video.style.position = 'absolute';
  video.style.left = '-9999px';
  video.style.top = '-9999px';
  video.style.width = '1px';
  video.style.height = '1px';
  video.style.zIndex = '-1';
  video.style.display = 'none';
  document.body.appendChild(video);
  
  // Removed visible play button - video hidden and autoplays
  
  // Try to load video with fallback
  let videoLoaded = false;
  let useTestPattern = false;
  
  function loadVideo(url) {
    return new Promise((resolve, reject) => {
      console.log('🎬 Attempting to load video from:', url);
      console.log('🔍 Video element state:', {
        readyState: video.readyState,
        networkState: video.networkState,
        src: video.src
      });
      
      video.src = url;
      
      const onLoadData = () => {
        videoLoaded = true;
        console.log('✅ Video loaded data event fired');
        console.log('📹 Video details:', {
          videoWidth: video.videoWidth,
          videoHeight: video.videoHeight,
          duration: video.duration,
          readyState: video.readyState
        });
        video.removeEventListener('loadeddata', onLoadData);
        video.removeEventListener('error', onError);
        video.removeEventListener('canplay', onCanPlay);
        resolve();
      };
      
      const onError = (e) => {
        console.error('❌ Video error event:', e);
        console.error('🔍 Video error details:', {
          code: e.target.error?.code,
          message: e.target.error?.message,
          src: e.target.src
        });
        video.removeEventListener('loadeddata', onLoadData);
        video.removeEventListener('error', onError);
        video.removeEventListener('canplay', onCanPlay);
        reject(new Error(`Failed to load video from ${url}`));
      };
      
      const onCanPlay = () => {
        console.log('🎮 Video canplay event fired');
        console.log('📹 Video canplay details:', {
          videoWidth: video.videoWidth,
          videoHeight: video.videoHeight,
          duration: video.duration
        });
      };
      
      video.addEventListener('loadeddata', onLoadData);
      video.addEventListener('error', onError);
      video.addEventListener('canplay', onCanPlay);
      
      // Timeout fallback
      setTimeout(() => {
        if (!videoLoaded) {
          console.error('⏰ Video loading timeout from:', url);
          video.removeEventListener('loadeddata', onLoadData);
          video.removeEventListener('error', onError);
          video.removeEventListener('canplay', onCanPlay);
          reject(new Error(`Video loading timeout from ${url}`));
        }
      }, 5000);
    });
  }
  
  try {
    await loadVideo(VIDEO_URL);
    console.log('✅ Primary video loaded successfully');
    // video remains hidden
  } catch (error) {
    console.log('❌ Primary video failed:', error.message, 'Trying fallback...');
    try {
      await loadVideo(FALLBACK_VIDEO_URL);
      console.log('✅ Fallback video loaded successfully');
      // video remains hidden
    } catch (fallbackError) {
      console.error('❌ Both videos failed to load:', fallbackError);
      console.log('🎨 Using test pattern instead');
      useTestPattern = true;
      statEl.textContent = 'Using test pattern';
      // video already hidden
    }
  }
  
  if (!useTestPattern) {
    // Try to play the video
    try {
      await video.play();
      console.log('✅ Video playing successfully');
    } catch (e) {
      console.log('⚠️ Video autoplay prevented:', e);
      // Try to play without autoplay
      video.play().catch(() => {
        console.log('❌ Video play failed even without autoplay');
      });
    }
  }

  // Resize canvas to window
  function resize() {
    canvas.width = innerWidth * devicePixelRatio;
    canvas.height = innerHeight * devicePixelRatio;
    gl.viewport(0, 0, canvas.width, canvas.height);
  }

  // Setup WebGL with enhanced security and diagnostics
  console.log('🎨 Setting up WebGL context...');
  const glOptions = {
    preserveDrawingBuffer: false,
    antialias: true,
    powerPreference: "high-performance",
    failIfMajorPerformanceCaveat: true
  };
  console.log('🔧 WebGL options:', glOptions);
  
  const gl = canvas.getContext('webgl', glOptions) || canvas.getContext('experimental-webgl', glOptions);
  
  if (!gl) {
    console.error('❌ WebGL context creation failed');
    console.error('🔍 Available canvas contexts:', {
      webgl: !!canvas.getContext('webgl'),
      experimentalWebgl: !!canvas.getContext('experimental-webgl'),
      '2d': !!canvas.getContext('2d')
    });
    alert('WebGL not available. Use a modern browser.');
    return;
  }
  
  console.log('✅ WebGL context created successfully');
  console.log('🔧 WebGL info:', {
    version: gl.getParameter(gl.VERSION),
    shadingLanguageVersion: gl.getParameter(gl.SHADING_LANGUAGE_VERSION),
    vendor: gl.getParameter(gl.VENDOR),
    renderer: gl.getParameter(gl.RENDERER),
    maxTextureSize: gl.getParameter(gl.MAX_TEXTURE_SIZE),
    maxViewportDims: gl.getParameter(gl.MAX_VIEWPORT_DIMS)
  });

  // Enhanced shader with encryption visualization
  const vs = `
    attribute vec2 a_pos;
    attribute vec2 a_uv;
    varying vec2 v_uv;
    void main(){ v_uv = a_uv; gl_Position = vec4(a_pos, 0.0, 1.0); }`;

  const fs = `
    precision mediump float;
    varying vec2 v_uv;
    uniform sampler2D u_tex;
    uniform vec2 u_res;
    uniform float u_time;
    uniform vec2 u_wmPos;
    uniform float u_wmAlpha;
    uniform vec3 u_wmColor;
    uniform float u_encryptionKey;
    
    float circle(vec2 p, vec2 c, float r){
      return smoothstep(r+0.02, r-0.0, distance(p,c));
    }
    
    void main(){
      vec2 uv = v_uv;
      vec4 col = texture2D(u_tex, uv);
      
      // Encryption effect - subtle color shifting based on key
      float encryptShift = sin(u_time * 0.5 + u_encryptionKey) * 0.02;
      col.r += encryptShift;
      col.b -= encryptShift;
      
      // Enhanced watermark with encryption indicator
      vec2 p = uv;
      float wm = circle(p, u_wmPos, 0.12);
      col.rgb = mix(col.rgb, u_wmColor, wm * u_wmAlpha * 0.65);
      
      // Add encryption pattern overlay
      float encryptPattern = sin(uv.x * 50.0 + u_time) * sin(uv.y * 50.0 + u_time);
      encryptPattern = smoothstep(0.98, 1.0, encryptPattern);
      col.rgb = mix(col.rgb, vec3(0.1, 0.3, 0.5), encryptPattern * 0.1);
      
      gl_FragColor = col;
    }`;

  function compileShader(src, type) {
    console.log('🔨 Compiling shader:', type === gl.VERTEX_SHADER ? 'Vertex' : 'Fragment');
    console.log('📄 Shader source length:', src.length);
    
    const s = gl.createShader(type);
    gl.shaderSource(s, src);
    gl.compileShader(s);
    
    const compileStatus = gl.getShaderParameter(s, gl.COMPILE_STATUS);
    console.log('📊 Shader compilation status:', compileStatus);
    
    if (!compileStatus) {
      const infoLog = gl.getShaderInfoLog(s);
      console.error('❌ Shader compilation error:', infoLog);
      console.error('🔍 Shader source preview:', src.substring(0, 200) + '...');
      gl.deleteShader(s);
      throw new Error('Shader compile error');
    }
    
    console.log('✅ Shader compiled successfully');
    return s;
  }

  console.log('🔗 Creating and linking shader program...');
  const prog = gl.createProgram();
  gl.attachShader(prog, compileShader(vs, gl.VERTEX_SHADER));
  gl.attachShader(prog, compileShader(fs, gl.FRAGMENT_SHADER));
  
  const linkStatus = gl.linkProgram(prog);
  console.log('📊 Program link status:', linkStatus);
  console.log('📊 Program link info:', gl.getProgramInfoLog(prog));
  
  if (!gl.getProgramParameter(prog, gl.LINK_STATUS)) {
    console.error('❌ Program linking failed');
    throw new Error('Program linking failed');
  }
  
  console.log('✅ Program linked successfully');
  gl.useProgram(prog);

  // Quad setup
  const pos = new Float32Array([
    -1,-1,  0,0,
     1,-1,  1,0,
    -1, 1,  0,1,
     1, 1,  1,1
  ]);
  const buf = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, buf);
  gl.bufferData(gl.ARRAY_BUFFER, pos, gl.STATIC_DRAW);

  const a_pos = gl.getAttribLocation(prog, 'a_pos');
  const a_uv = gl.getAttribLocation(prog, 'a_uv');
  gl.enableVertexAttribArray(a_pos);
  gl.enableVertexAttribArray(a_uv);
  gl.vertexAttribPointer(a_pos, 2, gl.FLOAT, false, 16, 0);
  gl.vertexAttribPointer(a_uv, 2, gl.FLOAT, false, 16, 8);

  // Create texture with anti-fingerprinting and enhanced logging
  console.log('🖼️ Creating WebGL texture...');
  const tex = gl.createTexture();
  gl.bindTexture(gl.TEXTURE_2D, tex);
  
  // Set texture parameters
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);
  gl.pixelStorei(gl.UNPACK_FLIP_Y_WEBGL, true);
  
  // Anti-fingerprinting: Randomize texture parameters
  const randomSeed = Math.random();
  const wrapMode = randomSeed > 0.5 ? gl.CLAMP_TO_EDGE : gl.REPEAT;
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, wrapMode);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, wrapMode);
  
  console.log('📊 Texture parameters set:', {
    minFilter: gl.LINEAR,
    magFilter: gl.LINEAR,
    wrapS: wrapMode,
    wrapT: wrapMode,
    randomSeed: randomSeed
  });

  // Enhanced uniforms with logging
  console.log('🔍 Getting uniform locations...');
  const uniforms = {
    u_tex: gl.getUniformLocation(prog, 'u_tex'),
    u_time: gl.getUniformLocation(prog, 'u_time'),
    u_wmPos: gl.getUniformLocation(prog, 'u_wmPos'),
    u_wmAlpha: gl.getUniformLocation(prog, 'u_wmAlpha'),
    u_wmColor: gl.getUniformLocation(prog, 'u_wmColor'),
    u_encryptionKey: gl.getUniformLocation(prog, 'u_encryptionKey')
  };
  
  console.log('📊 Uniform locations:', uniforms);
  
  // Check for missing uniforms
  Object.entries(uniforms).forEach(([name, location]) => {
    if (location === null) {
      console.warn('⚠️ Uniform not found:', name);
    }
  });

  // Overlay canvas for enhanced watermarking
  const overlay = document.createElement('canvas');
  overlay.style.position = 'fixed';
  overlay.style.left = '0';
  overlay.style.top = '0';
  overlay.style.pointerEvents = 'none';
  overlay.style.width = '100vw';
  overlay.style.height = '100vh';
  overlay.width = innerWidth * devicePixelRatio;
  overlay.height = innerHeight * devicePixelRatio;
  overlay.style.zIndex = 40;
  const overlayCtx = overlay.getContext('2d');
  document.body.appendChild(overlay);

  // Enhanced watermark state
  let wmX = 0.82, wmY = 0.88;
  let start = performance.now();

  // Security state
  let detected = false;
  let blackout = false;
  let decryptionKeyValid = true;
  let encryptionKey = ENCRYPTION_KEY;

  // Enhanced rAF monitoring with screen detection
  let lastRAF = performance.now();
  let frameCount = 0;
  let lastFrameTime = performance.now();
  let rafGraceEnd = Date.now() + 2000; // 2s grace: fast start, low false pos
  let frameDts = [];
  let readPixelsTimes = [];
  let probeCounter = 0;
  let gapCount = 0, fpsCount = 0, varCount = 0, gpuCount = 0; // Confirmation counters

  // Metrics UI
  const metricEls = {
    gap: document.getElementById('gap-metric'),
    fps: document.getElementById('fps-metric'),
    var: document.getElementById('var-metric'),
    gpu: document.getElementById('gpu-metric'),
    dt: document.getElementById('dt-metric'),
    stddev: document.getElementById('stddev-metric'),
    gpuavg: document.getElementById('gpuavg-metric')
  };
  let lastDt = 0, lastStdDev = 0, lastGpuAvg = 0;

  function updateMetrics() {
    const setColor = (el, count) => {
      el.className = 'metric ' + (count === 0 ? 'green' : count < 3 ? 'yellow' : 'red');
    };
    setColor(metricEls.gap, gapCount);
    setColor(metricEls.fps, fpsCount);
    setColor(metricEls.var, varCount);
    setColor(metricEls.gpu, gpuCount);
    metricEls.gap.textContent = `Gap: ${gapCount}/3`;
    metricEls.fps.textContent = `FPS: ${fpsCount}/3`;
    metricEls.var.textContent = `Var: ${varCount}/3`;
    metricEls.gpu.textContent = `GPU: ${gpuCount}/3`;
    metricEls.dt.textContent = `DT: ${Math.round(lastDt)} ms`;
    metricEls.stddev.textContent = `StdDev: ${lastStdDev.toFixed(1)} ms`;
    metricEls.gpuavg.textContent = `GPU Avg: ${lastGpuAvg.toFixed(1)} ms`;
  }
  
  function rafCheck(now) {
    const dt = now - lastRAF;
    lastRAF = now;
    frameCount++;
    
    frameDts.push(dt);
    if (frameDts.length > 120) frameDts.shift();
    
    // Grace period for initial load lag
    if (now < rafGraceEnd) {
      return requestAnimationFrame(rafCheck);
    }
    
    // Gap detection with counter (3+ confirms OBS stutter)
    if (dt > FRAME_GAP_THRESHOLD_MS) {
      gapCount = Math.min(gapCount + 1, 5);
      if (gapCount >= 3) {
        markDetected(`rAF gaps x3+: ${Math.round(dt)}ms (dropped frames/OBS)`);
      }
    } else {
      gapCount = Math.max(gapCount - 1, 0);
    }
    
    if (frameCount > 60) {
      const avgFrameTime = (now - lastFrameTime) / frameCount;
      if (Math.abs(avgFrameTime - 16.67) < 3.0) {  // Tuned tolerance
        fpsCount = Math.min(fpsCount + 1, 5);
        if (fpsCount >= 3) {
          markDetected(`Consistent FPS x3+: ${(1000/avgFrameTime).toFixed(1)} (recording)`);
        }
      } else {
        fpsCount = Math.max(fpsCount - 1, 0);
      }
      frameCount = 0;
      lastFrameTime = now;
    }
    
    if (frameDts.length >= 60) {
      const mean = frameDts.reduce((a,b)=>a+b,0) / frameDts.length;
      const variance = frameDts.reduce((a,b)=>a + Math.pow(b-mean,2),0) / frameDts.length;
      const stdDev = Math.sqrt(variance);
      if (stdDev < 1.0) {  // Tuned variance
        varCount = Math.min(varCount + 1, 5);
        if (varCount >= 3) {
          markDetected(`Low variance x3+: ${stdDev.toFixed(2)}ms stddev (smoothing/OBS)`);
        }
      } else {
        varCount = Math.max(varCount - 1, 0);
      }
    }
    
    requestAnimationFrame(rafCheck);
  }
  requestAnimationFrame(rafCheck);

  // Enhanced canvas capture detection and anti-screen recording
  const _origCanvasCapture = HTMLCanvasElement.prototype.captureStream;
  HTMLCanvasElement.prototype.captureStream = function(...args) {
    try {
      const id = (this === canvas) ? 'GL_CANVAS' : (this === overlay ? 'OVERLAY' : 'OTHER_CANVAS');
      markDetected('canvas.captureStream() called on ' + id);
      
      // Return corrupted stream for anti-screen recording
      if (this === canvas || this === overlay) {
        console.log('🚫 Blocking screen capture attempt');
        // Return a black stream instead
        const blackCanvas = document.createElement('canvas');
        blackCanvas.width = this.width;
        blackCanvas.height = this.height;
        const blackCtx = blackCanvas.getContext('2d');
        blackCtx.fillStyle = '#000';
        blackCtx.fillRect(0, 0, blackCanvas.width, blackCanvas.height);
        return blackCanvas.captureStream.apply(blackCanvas, args);
      }
    } catch (e) {}
    return _origCanvasCapture.call(this, ...args);
  };

  // Enhanced MediaRecorder detection with anti-recording
  const _origMediaRecorder = window.MediaRecorder;
  if (_origMediaRecorder) {
    window.MediaRecorder = function(stream, opt) {
      markDetected('MediaRecorder created (possible recording)');
      
      // Return corrupted recorder for anti-screen recording
      try {
        const recorder = new _origMediaRecorder(stream, opt);
        
        // Override the dataavailable event to return corrupted data
        const originalOnDataAvailable = recorder.ondataavailable;
        recorder.ondataavailable = function(e) {
          if (e.data && e.data.size > 0) {
            // Corrupt the video data
            const corruptedBlob = new Blob(['[CORRUPTED DATA]'], { type: 'video/webm' });
            e.data = corruptedBlob;
          }
          if (originalOnDataAvailable) {
            originalOnDataAvailable.call(this, e);
          }
        };
        
        return recorder;
      } catch (e) {
        return new _origMediaRecorder(stream, opt);
      }
    };
    Object.setPrototypeOf(window.MediaRecorder, _origMediaRecorder);
    window.MediaRecorder.prototype = _origMediaRecorder.prototype;
  }

  // Enhanced devtools detection - only trigger on F12 or right-click
  let devtoolsOpen = false;
  
  // F12 key detection
  document.addEventListener('keydown', (e) => {
    if (e.key === 'F12') {
      e.preventDefault();
      markDetected('DevTools opened via F12 key');
      devtoolsOpen = true;
    }
  });
  
  // Right-click context menu detection for inspect element
  document.addEventListener('contextmenu', (e) => {
    // Check if right-click is on an element that might be inspected
    const target = e.target;
    if (target && (target.tagName === 'BODY' || target.tagName === 'HTML' || target.id === 'glcanvas')) {
      // Add a small delay to detect if inspect element was actually used
      setTimeout(() => {
        // Check if devtools might have been opened
        const widthDiff = window.outerWidth - window.innerWidth;
        const heightDiff = window.outerHeight - window.innerHeight;
        if (widthDiff > 100 || heightDiff > 100) {
          markDetected('DevTools opened via right-click inspect element');
          devtoolsOpen = true;
        }
      }, 500);
    }
  });
  
  // Fallback: traditional devtools detection (less aggressive)
  (function detectDevtoolsFallback() {
    const threshold = 200;
    function check() {
      const widthDiff = window.outerWidth - window.innerWidth;
      const heightDiff = window.outerHeight - window.innerHeight;
      if (widthDiff > threshold || heightDiff > threshold) {
        if (!devtoolsOpen) markDetected('DevTools or large UI detected');
        devtoolsOpen = true;
      }
      
      // Additional security checks (less frequent)
      if (Math.random() < 0.02) { // Only check 2% of the time
        checkForSecurityBreach();
      }
      
      setTimeout(check, 5000); // Increased interval
    }
    check();
  })();

  // Additional security breach detection
  function checkForSecurityBreach() {
    // Check for debugger statements
    try {
      eval('debugger');
    } catch (e) {
      markDetected('Debugger statement detected');
    }
    
    // Check for performance monitoring
    if (window.performance && performance.memory) {
      // Memory monitoring might indicate analysis tools
      markDetected('Performance monitoring detected');
    }
    
    // Check for keyboard event listeners
    const keyListeners = getEventListeners(window);
    if (keyListeners && keyListeners.key && keyListeners.key.length > 10) {
      markDetected('Excessive keyboard event listeners');
    }
  }

  // Helper to get event listeners (enhanced detection)
  function getEventListeners(obj) {
    if (obj.__getEventListeners) {
      return obj.__getEventListeners();
    }
    return null;
  }

  // Enhanced detection helper with detailed breach information
  function markDetected(msg) {
    if (detected) return;
    detected = true;
    console.warn('SECURITY BREACH:', msg);
    statEl.textContent = 'BREACH: ' + msg;
    encStatEl.textContent = 'Compromised';
    warningEl.style.display = 'block';
    decryptionKeyValid = false;
    blackout = true; // Immediately blackout
    
    // Hide video frame and play button on security breach
    if (video) {
      video.style.display = 'none';
    }
    if (playButton) {
      playButton.style.display = 'none';
    }
    
    // Populate detailed breach information
    const now = new Date();
    const timeString = now.toLocaleTimeString();
    const sessionId = 'SECURE-' + Math.random().toString(36).substr(2, 8).toUpperCase();
    
    // Categorize breach type and severity
    let breachType = 'Unknown';
    let breachSource = 'Unknown';
    let breachSeverity = 'Unknown';
    let riskLevel = 'HIGH';
    let recommendations = [];
    
    if (msg.includes('rAF gap') || msg.includes('OBS hook')) {
      breachType = 'Frame Rate Tampering';
      breachSource = 'Recording Software';
      breachSeverity = 'High';
      riskLevel = 'HIGH';
      recommendations = [
        '• Immediate session closure recommended',
        '• Check for screen recording software',
        '• Monitor system performance',
        '• Consider system security scan'
      ];
    } else if (msg.includes('canvas.captureStream')) {
      breachType = 'Canvas Capture';
      breachSource = 'Recording Extension';
      breachSeverity = 'Critical';
      riskLevel = 'CRITICAL';
      recommendations = [
        '• CRITICAL: Close browser immediately',
        '• Remove suspicious extensions',
        '• Clear browser cache and cookies',
        '• Change security credentials'
      ];
    } else if (msg.includes('MediaRecorder')) {
      breachType = 'Media Recording';
      breachSource = 'Recording API';
      breachSeverity = 'Critical';
      riskLevel = 'CRITICAL';
      recommendations = [
        '• CRITICAL: Session compromised',
        '• Stop all media recording',
        '• Check browser permissions',
        '• Revoke API access if possible'
      ];
    } else if (msg.includes('DevTools') || msg.includes('console opened')) {
      breachType = 'Developer Tools';
      breachSource = 'Browser DevTools';
      breachSeverity = 'Medium';
      riskLevel = 'MEDIUM';
      recommendations = [
        '• Close developer tools',
        '• Disable browser debugging',
        '• Check for debugging extensions',
        '• Monitor network activity'
      ];
    } else if (msg.includes('Debugger')) {
      breachType = 'Debugger Detection';
      breachSource = 'Debugging Tools';
      breachSeverity = 'High';
      riskLevel = 'HIGH';
      recommendations = [
        '• Debugging detected',
        '• Close debugging sessions',
        '• Check for debug extensions',
        '• Review code execution'
      ];
    } else if (msg.includes('Performance monitoring')) {
      breachType = 'Performance Analysis';
      breachSource = 'Monitoring Tools';
      breachSeverity = 'Medium';
      riskLevel = 'MEDIUM';
      recommendations = [
        '• Performance monitoring detected',
        '• Check monitoring extensions',
        '• Review system resources',
        '• Disable unnecessary monitoring'
      ];
    } else if (msg.includes('Excessive keyboard')) {
      breachType = 'Input Monitoring';
      breachSource = 'Keylogging Extension';
      breachSeverity = 'High';
      riskLevel = 'HIGH';
      recommendations = [
        '• Keylogging suspected',
        '• Change passwords immediately',
        '• Remove keyboard extensions',
        '• Use secure password manager'
      ];
    } else if (msg.includes('Suspicious extensions')) {
      breachType = 'Extension Detection';
      breachSource = 'Browser Extension';
      breachSeverity = 'Medium';
      riskLevel = 'MEDIUM';
      recommendations = [
        '• Suspicious extensions found',
        '• Review and remove extensions',
        '• Check extension permissions',
        '• Use only trusted extensions'
      ];
    } else if (msg.includes('readPixels')) {
      breachType = 'Pixel Analysis';
      breachSource = 'Screen Capture';
      breachSeverity = 'High';
      riskLevel = 'HIGH';
      recommendations = [
        '• Screen capture detected',
        '• Close browser session',
        '• Check for capture software',
        '• Review screen sharing settings'
      ];
    } else if (msg.includes('Manual trigger')) {
      breachType = 'Manual Test';
      breachSource = 'User Action';
      breachSeverity = 'Low';
      riskLevel = 'LOW';
      recommendations = [
        '• Manual test triggered',
        '• No security threat detected',
        '• System functioning normally',
        '• Continue monitoring'
      ];
    } else {
      breachType = 'Security Breach';
      breachSource = 'Unknown Source';
      breachSeverity = 'High';
      riskLevel = 'HIGH';
      recommendations = [
        '• Unknown security breach',
        '• Close browser session',
        '• Check system security',
        '• Monitor for unusual activity'
      ];
    }
    
    // Update breach details display
    breachTypeEl.textContent = breachType;
    breachSourceEl.textContent = breachSource;
    breachTimeEl.textContent = timeString;
    breachSeverityEl.textContent = breachSeverity;
    
    // Update enhanced warning elements
    actionRecommendationsEl.innerHTML = recommendations.join('<br>');
    riskLevelEl.textContent = riskLevel;
    sessionIdEl.textContent = sessionId;
    
    // Apply severity-based styling
    const breachDetailsEl = document.getElementById('breach-details');
    const riskAssessmentEl = document.getElementById('risk-assessment');
    
    if (riskLevel === 'CRITICAL') {
      breachDetailsEl.style.background = 'rgba(255,0,0,0.25)';
      breachDetailsEl.style.border = '1px solid rgba(255,100,100,0.5)';
      riskLevelEl.style.color = '#ff4444';
      riskLevelEl.style.fontWeight = 'bold';
    } else if (riskLevel === 'HIGH') {
      breachDetailsEl.style.background = 'rgba(255,100,0,0.2)';
      breachDetailsEl.style.border = '1px solid rgba(255,150,0,0.3)';
      riskLevelEl.style.color = '#ff6666';
      riskLevelEl.style.fontWeight = 'bold';
    } else if (riskLevel === 'MEDIUM') {
      breachDetailsEl.style.background = 'rgba(255,200,0,0.15)';
      breachDetailsEl.style.border = '1px solid rgba(255,200,0,0.2)';
      riskLevelEl.style.color = '#ffaa44';
      riskLevelEl.style.fontWeight = 'normal';
    } else {
      breachDetailsEl.style.background = 'rgba(100,200,255,0.1)';
      breachDetailsEl.style.border = '1px solid rgba(100,200,255,0.2)';
      riskLevelEl.style.color = '#88ccff';
      riskLevelEl.style.fontWeight = 'normal';
    }
    
    // Log encrypted detection data
    const encryptedMsg = encryptData(msg, encryptionKey);
    console.log('Encrypted log:', encryptedMsg);
  }

  // Enhanced key rotation with encryption
  function startKeyRotation() {
    setInterval(() => {
      // Simulate server key rotation with encryption
      const gotKey = Math.random() > 0.05;
      decryptionKeyValid = gotKey;
      
      if (!gotKey) {
        blackout = true;
        statEl.textContent = 'Key invalidated - requesting new key...';
        encStatEl.textContent = 'Key rotation';
        
        // Simulate encrypted key request
        setTimeout(() => {
          const newKey = generateEncryptionKey();
          encryptionKey = newKey;
          decryptionKeyValid = true;
          blackout = false;
          statEl.textContent = 'OK (key rotated)';
          encStatEl.textContent = 'Active';
        }, 800 + Math.random() * 1200);
      } else {
        // Rotate encryption key
        encryptionKey = generateEncryptionKey();
        statEl.textContent = 'OK (key rotated)';
        encStatEl.textContent = 'Active';
      }
    }, KEY_ROTATE_SECONDS * 1000);
  }
  startKeyRotation();

  // Enhanced draw loop
  function draw() {
    resizeIfNeeded();
    if (blackout || detected || !decryptionKeyValid) {
      gl.clearColor(0, 0, 0, 1);
      gl.clear(gl.COLOR_BUFFER_BIT);
      overlayCtx.clearRect(0, 0, overlay.width, overlay.height);
      requestAnimationFrame(draw);
      return;
    }

    try {
      if (useTestPattern) {
        // Create a simple test pattern
        const canvas = document.createElement('canvas');
        canvas.width = 640;
        canvas.height = 360;
        const ctx = canvas.getContext('2d');
        
        // Create gradient test pattern
        const gradient = ctx.createLinearGradient(0, 0, canvas.width, canvas.height);
        gradient.addColorStop(0, '#ff0000');
        gradient.addColorStop(0.25, '#00ff00');
        gradient.addColorStop(0.5, '#0000ff');
        gradient.addColorStop(0.75, '#ffff00');
        gradient.addColorStop(1, '#ff00ff');
        
        ctx.fillStyle = gradient;
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        
        // Add some text
        ctx.fillStyle = 'white';
        ctx.font = '48px Arial';
        ctx.textAlign = 'center';
        ctx.fillText('TEST PATTERN', canvas.width/2, canvas.height/2);
        
        gl.bindTexture(gl.TEXTURE_2D, tex);
        gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, canvas);
        
        if (!videoReady) {
          videoReady = true;
          console.log('🎨 Test pattern loaded');
          statEl.textContent = 'OK - Test pattern';
        }
      } else {
        gl.bindTexture(gl.TEXTURE_2D, tex);
        gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, video);
        
        // Debug: Check if video is ready
        if (video.readyState >= 2) {
          if (!videoReady) {
            videoReady = true;
            console.log('🎬 Video is ready for rendering');
            console.log('Video dimensions:', video.videoWidth, 'x', video.videoHeight);
            statEl.textContent = 'OK - Video playing';
          }
        } else {
          if (videoReady) {
            videoReady = false;
            console.log('⏳ Video not ready for rendering (readyState:', video.readyState, ')');
            statEl.textContent = 'Waiting for video...';
          }
        }
      }
    } catch (e) {
      console.log('❌ Texture error:', e);
      statEl.textContent = 'Texture error';
    }

    // Enhanced uniforms with encryption and logging
    const t = (performance.now() - start) / 1000;
    console.log('🎨 Setting uniforms at time:', t.toFixed(2));
    
    if (uniforms.u_time !== null) {
      gl.uniform1f(uniforms.u_time, t);
    }
    if (uniforms.u_encryptionKey !== null) {
      gl.uniform1f(uniforms.u_encryptionKey, encryptionKey.charCodeAt(0) / 255.0);
    }
    
    // Enhanced watermark movement
    const sx = 0.82 + Math.sin(t * 0.7) * 0.03, sy = 0.88 + Math.cos(t * 0.53) * 0.02;
    console.log('💧 Watermark position:', { sx: sx.toFixed(3), sy: sy.toFixed(3) });
    
    if (uniforms.u_wmPos !== null) {
      gl.uniform2f(uniforms.u_wmPos, sx, sy);
    }
    if (uniforms.u_wmAlpha !== null) {
      gl.uniform1f(uniforms.u_wmAlpha, 0.5);
    }
    if (uniforms.u_wmColor !== null) {
      gl.uniform3f(uniforms.u_wmColor, 1.0, 0.15, 0.15);
    }

    gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);

    // GPU load probe for screen recording detection (OBS etc.)
    probeCounter++;
    if (probeCounter % 120 === 0) {  // ~2s @60fps: faster checks
      const pixels = new Uint8Array(4);
      const t0 = performance.now();
      gl.readPixels(canvas.width - 1, canvas.height - 1, 1, 1, gl.RGBA, gl.UNSIGNED_BYTE, pixels);
      const t1 = performance.now();
      const duration = t1 - t0;
      readPixelsTimes.push(duration);
      if (readPixelsTimes.length > 30) readPixelsTimes.shift();
      if (readPixelsTimes.length >= 10) {
        const recentAvg = readPixelsTimes.slice(-10).reduce((a,b)=>a+b,0) / 10;
        if (recentAvg > 3.5) {  // Tuned threshold
          gpuCount = Math.min(gpuCount + 1, 5);
          if (gpuCount >= 3) {
            markDetected(`Slow GPU x3+: ${recentAvg.toFixed(1)}ms avg (OBS encoding)`);
          }
        } else {
          gpuCount = Math.max(gpuCount - 1, 0);
        }
      }
    }

    // Enhanced overlay watermark
    overlayCtx.clearRect(0, 0, overlay.width, overlay.height);
    overlayCtx.save();
    overlayCtx.scale(devicePixelRatio, devicePixelRatio);
    overlayCtx.font = '20px system-ui, Arial';
    overlayCtx.fillStyle = 'rgba(255, 255, 255, 0.07)';
    overlayCtx.textAlign = 'right';
    
    const wmstr = USER_ID + ' • Encrypted • ' + (new Date()).toLocaleString();
    overlayCtx.fillText(wmstr, innerWidth - 12, innerHeight - 12);
    
    // Add encryption status indicator
    overlayCtx.fillStyle = decryptionKeyValid ? 'rgba(78, 205, 196, 0.3)' : 'rgba(255, 107, 107, 0.3)';
    overlayCtx.fillRect(innerWidth - 150, innerHeight - 40, 140, 20);
    overlayCtx.fillStyle = '#fff';
    overlayCtx.font = '12px monospace';
    overlayCtx.fillText(decryptionKeyValid ? 'ENCRYPTED' : 'DECRYPTED', innerWidth - 145, innerHeight - 25);
    
    overlayCtx.restore();

    requestAnimationFrame(draw);
  }

  // Enhanced resize handling
  function resizeIfNeeded() {
    const w = Math.floor(innerWidth * devicePixelRatio);
    const h = Math.floor(innerHeight * devicePixelRatio);
    if (canvas.width !== w || canvas.height !== h) {
      canvas.width = w; canvas.height = h;
      overlay.width = w; overlay.height = h;
      gl.viewport(0, 0, w, h);
    }
  }

  // User interaction handling
  canvas.addEventListener('click', () => { 
    if (!detected) { 
      video.muted = false; 
      video.play(); 
    } 
  });

  // Enhanced keyboard controls
  window.addEventListener('keydown', (e) => {
    if (e.key === 'R') { markDetected('Manual trigger (R)'); }
    if (e.key === 'K') { 
      decryptionKeyValid = !decryptionKeyValid; 
      statEl.textContent = decryptionKeyValid ? 'OK (key valid)' : 'Key invalidated (manual)';
      encStatEl.textContent = decryptionKeyValid ? 'Active' : 'Inactive';
    }
    if (e.key === 'E') {
      // Manual encryption key rotation
      encryptionKey = generateEncryptionKey();
      console.log('Manual key rotation:', encryptionKey);
    }
  });

  // Enhanced initialization with diagnostics
  console.log('🔍 Starting Enhanced Anti-Recording System initialization...');
  console.log('📱 Device Info:', {
    userAgent: navigator.userAgent,
    isMobile: /android|webos|iphone|ipad|ipod|blackberry|iemobile|opera mini/i.test(navigator.userAgent.toLowerCase()),
    screenResolution: `${screen.width}x${screen.height}`,
    viewport: `${window.innerWidth}x${window.innerHeight}`
  });
  
  // Test WebGL support
  try {
    const testCanvas = document.createElement('canvas');
    const gl = testCanvas.getContext('webgl') || testCanvas.getContext('experimental-webgl');
    if (!gl) {
      console.error('❌ WebGL not supported');
      statEl.textContent = 'WebGL Error';
      return;
    }
    console.log('✅ WebGL supported:', gl.getParameter(gl.VERSION));
  } catch (e) {
    console.error('❌ WebGL test failed:', e);
    statEl.textContent = 'WebGL Error';
    return;
  }
  
  // Test video loading
  console.log('🎬 Testing video loading...');
  const testVideo = document.createElement('video');
  testVideo.preload = 'metadata';
  testVideo.addEventListener('loadedmetadata', () => {
    console.log('✅ Video metadata loaded successfully');
  });
  testVideo.addEventListener('error', (e) => {
    console.error('❌ Video loading error:', e);
    console.log('🔄 Trying fallback video...');
  });
  testVideo.src = VIDEO_URL;
  
  // Initialize
  console.log('📐 Initializing canvas and resize handling...');
  resize();
  window.addEventListener('resize', resize);
  statEl.textContent = 'OK';
  encStatEl.textContent = 'Active';
  
  // Start the draw loop
  console.log('🎨 Starting draw loop...');
  draw();

  // Cleanup on page unload
  window.addEventListener('beforeunload', () => {
    extensionDetector.stopMonitoring();
  });

})();
</script>
</body>
</html>
