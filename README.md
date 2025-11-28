<!doctype html>
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
  #extension-warning { position:fixed; right:12px; top:12px; z-index:55; background:rgba(255,0,0,0.2); padding:8px; border-radius:6px; font-size:12px; display:none; border:1px solid rgba(255,0,0,0.3); }
  .extension-item { margin:2px 0; padding:3px 6px; border-radius:3px; font-size:11px; }
  .extension-blocked { background:rgba(255,68,68,0.15); color:#ff4444; border-left:3px solid #ff4444; }
  .extension-safe { background:rgba(78,205,196,0.15); color:#4ecdc4; border-left:3px solid #4ecdc4; }
  .extension-medium { background:rgba(255,136,0,0.15); color:#ff8800; border-left:3px solid #ff8800; }
  .extension-low { background:rgba(255,170,0,0.15); color:#ffaa00; border-left:3px solid #ffaa00; }
  #loading { position:fixed; left:50%; top:50%; transform:translate(-50%,-50%); background:rgba(0,0,0,0.9); padding:20px; border-radius:10px; z-index:70; }
  .spinner { border:3px solid #333; border-top:3px solid #4ecdc4; border-radius:50%; width:30px; height:30px; animation:spin 1s linear infinite; margin:0 auto 10px; }
  @keyframes spin { 0% { transform:rotate(0deg); } 100% { transform:rotate(360deg); } }
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
    <div id="extension-list" style="margin-top:8px;font-size:11px;color:#ff6b6b;max-height:200px;overflow-y:auto;display:none;">
      <div style="font-weight:bold;margin-bottom:4px;">🔍 Extension Detection Results:</div>
      <div id="extension-details"></div>
      <div style="margin-top:8px;font-size:9px;color:#888; border-top:1px solid rgba(255,255,255,0.1); padding-top:4px;">
        <strong>📋 Important Notes:</strong><br>
        • Websites cannot directly access your extension list<br>
        • This shows only extension-related data found on this page<br>
        • Use "View Extensions" button to see all installed extensions<br>
        • Some extensions may not leave any detectable traces
      </div>
    </div>
    <div style="margin-top:4px;font-size:11px;color:#4ecdc4;">
      <span id="total-extensions">0</span> total items detected |
      <span id="blocked-extensions">0</span> flagged as suspicious
    </div>
    <div style="margin-top:2px;font-size:10px;color:#aaa;">
      <label style="cursor:pointer;">
        <input type="checkbox" id="showAllExtensions" checked> Show all detected items (highlight suspicious)
      </label>
      <button id="open-extensions" style="margin-left:8px;padding:2px 6px;background:rgba(78,205,196,0.2);border:1px solid rgba(78,205,196,0.3);border-radius:3px;color:#4ecdc4;font-size:9px;cursor:pointer;">View Extensions</button>
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
  const VIDEO_URL = "https://www.w3schools.com/html/mov_bbb.mp4";
  const USER_ID = "secure_user_" + Math.random().toString(36).substr(2, 9);
  const KEY_ROTATE_SECONDS = 8;
  const FRAME_GAP_THRESHOLD_MS = 300; // Increased from 140 to reduce false positives
  const ENCRYPTION_KEY = generateEncryptionKey();
  const EXTENSION_BLACKLIST = [
    'screen capture extension', 'video recorder extension', 'tab recorder extension',
    'extension recorder', 'screencast extension', 'video download extension',
    'tab capture extension', 'screen recorder extension', 'capture extension',
    'record extension', 'extension capture', 'extension record', 'Chrome Capture - Screenshot & GIF',
    'lighthouse', 'web developer', 'firebug', 'wappalyzer', 'http header',
    'network', 'inspector', 'debugger', 'developer tools', 'element inspector',
    'screenshot', 'screen capture', 'video capture', 'audio capture'
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
  const showAllExtensionsCheckbox = document.getElementById('showAllExtensions');
  const blockedExtensionsEl = document.getElementById('blocked-extensions');
  const openExtensionsBtn = document.getElementById('open-extensions');
  
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
        mobileViewport: isMobileViewport,
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
        <div><strong>Mobile Indicators:</strong> ${deviceInfo.mobileViewport || deviceInfo.mobileScreen || deviceInfo.mobilePlatform ? 'Detected' : 'None'}</div>
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

  // Chrome extension detection and blocking
  class ExtensionDetector {
    constructor() {
      this.detectedExtensions = new Set();
      this.monitoringInterval = null;
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
      this.checkForCommonExtensions();
      this.checkForExtensionIcons();
      this.checkForPermissions();
    }

    checkStorage() {
      const storageTypes = ['localStorage', 'sessionStorage'];
      storageTypes.forEach(type => {
        try {
          const storage = window[type];
          for (let i = 0; i < storage.length; i++) {
            const key = storage.key(i);
            if (this.containsExtensionKeywords(key)) {
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
            this.detectedExtensions.add(`API:${api}`);
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
      // Only flag if we detect actual extension-like behavior
      if (navigator.mediaDevices?.getDisplayMedia) {
        // Check if getDisplayMedia is being called unexpectedly
        // This is a more conservative approach to avoid false positives
        try {
          // Check if there are any event listeners that might indicate extension activity
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

    checkForCommonExtensions() {
      // Check for common extension patterns in various browser storage areas
      const commonExtensionPatterns = [
        'extension', 'addon', 'plugin', 'module', 'script',
        'greasemonkey', 'tampermonkey', 'userscript', 'violentmonkey',
        'adblock', 'ublock', 'privacy', 'security', 'developer',
        'webstore', 'chrome-extension', 'moz-extension', 'edge-extension'
      ];

      // Check in localStorage
      try {
        for (let i = 0; i < localStorage.length; i++) {
          const key = localStorage.key(i);
          const value = localStorage.getItem(key);
          
          commonExtensionPatterns.forEach(pattern => {
            if (key && (key.toLowerCase().includes(pattern) ||
                (value && value.toLowerCase().includes(pattern)))) {
              this.detectedExtensions.add(`Common:${pattern}`);
            }
          });
        }
      } catch (e) {}

      // Check in sessionStorage
      try {
        for (let i = 0; i < sessionStorage.length; i++) {
          const key = sessionStorage.key(i);
          const value = sessionStorage.getItem(key);
          
          commonExtensionPatterns.forEach(pattern => {
            if (key && (key.toLowerCase().includes(pattern) ||
                (value && value.toLowerCase().includes(pattern)))) {
              this.detectedExtensions.add(`Common:${pattern}`);
            }
          });
        }
      } catch (e) {}
    }

    checkForExtensionIcons() {
      // Check for extension icon patterns in DOM
      const iconPatterns = [
        'extension-icon', 'extension-icon-16', 'extension-icon-32',
        'extension-icon-48', 'extension-icon-128', 'extension-icon-256',
        'chrome-toolbar-icon', 'browser-action', 'page-action'
      ];

      // Check for icon-related CSS classes
      const allClasses = document.querySelectorAll('*');
      allClasses.forEach(element => {
        const classList = element.className || '';
        iconPatterns.forEach(pattern => {
          if (classList.toLowerCase().includes(pattern)) {
            this.detectedExtensions.add(`Icon:${pattern}`);
          }
        });
      });
    }

    checkForPermissions() {
      // Check for extension permissions patterns
      const permissionPatterns = [
        'permissions', 'activeTab', 'storage', 'tabs', 'webRequest',
        'webNavigation', 'cookies', 'downloads', 'history', 'bookmarks',
        'management', 'runtime', 'extension'
      ];

      // Check in localStorage and sessionStorage for permission-related items
      ['localStorage', 'sessionStorage'].forEach(type => {
        try {
          const storage = window[type];
          for (let i = 0; i < storage.length; i++) {
            const key = storage.key(i);
            if (key && permissionPatterns.some(pattern =>
                key.toLowerCase().includes(pattern))) {
              this.detectedExtensions.add(`Permission:${key}`);
            }
          }
        } catch (e) {}
      });
    }

    containsExtensionKeywords(text) {
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

  // Enhanced extension detection with total count
  function getAllExtensions() {
    const allExtensions = new Set();
    
    // Browser security limitations documentation
    console.log('=== EXTENSION DETECTION LIMITATIONS ===');
    console.log('Due to browser security policies, websites cannot directly access:');
    console.log('• List of installed browser extensions');
    console.log('• Extension names and versions');
    console.log('• Extension permissions');
    console.log('• Extension icons');
    console.log('');
    console.log('Available detection methods:');
    console.log('• localStorage/sessionStorage keys (extension data)');
    console.log('• Browser API presence (chrome.*, browser.*)');
    console.log('• UserAgent strings (extension-related keywords)');
    console.log('• DOM element patterns (extension icons)');
    console.log('• Cookie patterns (extension tracking)');
    console.log('========================================');
    
    // Check localStorage for extension-related data
    try {
      for (let i = 0; i < localStorage.length; i++) {
        const key = localStorage.key(i);
        if (key) {
          // Look for extension-related storage keys
          const extensionPatterns = [
            'extension', 'addon', 'plugin', 'webstore', 'chrome-extension',
            'moz-extension', 'edge-extension', 'greasemonkey', 'tampermonkey',
            'userscript', 'violentmonkey', 'adblock', 'ublock', 'privacy',
            'security', 'developer', 'webstore', 'manifest', 'extension_id'
          ];
          
          const isExtensionRelated = extensionPatterns.some(pattern =>
            key.toLowerCase().includes(pattern)
          );
          
          if (isExtensionRelated) {
            allExtensions.add(`localStorage:${key} [EXTENSION_DATA]`);
          }
        }
      }
    } catch (e) {
      console.log('localStorage access blocked:', e.message);
    }
    
    // Check sessionStorage for extension-related data
    try {
      for (let i = 0; i < sessionStorage.length; i++) {
        const key = sessionStorage.key(i);
        if (key) {
          // Look for extension-related storage keys
          const extensionPatterns = [
            'extension', 'addon', 'plugin', 'webstore', 'chrome-extension',
            'moz-extension', 'edge-extension', 'greasemonkey', 'tampermonkey',
            'userscript', 'violentmonkey', 'adblock', 'ublock', 'privacy',
            'security', 'developer', 'webstore', 'manifest', 'extension_id'
          ];
          
          const isExtensionRelated = extensionPatterns.some(pattern =>
            key.toLowerCase().includes(pattern)
          );
          
          if (isExtensionRelated) {
            allExtensions.add(`sessionStorage:${key} [EXTENSION_DATA]`);
          }
        }
      }
    } catch (e) {
      console.log('sessionStorage access blocked:', e.message);
    }
    
    // Check cookies for extension-related data
    try {
      const cookies = document.cookie.split(';');
      cookies.forEach(cookie => {
        const [name] = cookie.trim().split('=');
        if (name) {
          // Look for extension-related cookies
          const extensionPatterns = [
            'extension', 'addon', 'plugin', 'webstore', 'chrome-extension',
            'moz-extension', 'edge-extension', 'greasemonkey', 'tampermonkey',
            'userscript', 'violentmonkey', 'adblock', 'ublock', 'privacy',
            'security', 'developer', 'webstore', 'manifest', 'extension_id'
          ];
          
          const isExtensionRelated = extensionPatterns.some(pattern =>
            name.toLowerCase().includes(pattern)
          );
          
          if (isExtensionRelated) {
            allExtensions.add(`cookie:${name} [EXTENSION_DATA]`);
          }
        }
      });
    } catch (e) {
      console.log('cookie access blocked:', e.message);
    }
    
    // Check browser APIs
    const apiExtensions = [
      'chrome.runtime', 'chrome.extension', 'chrome.tabs', 'chrome.webRequest',
      'browser.runtime', 'browser.extension', 'browser.tabs', 'chrome.permissions',
      'chrome.storage', 'chrome.management', 'chrome.identity'
    ];
    
    apiExtensions.forEach(api => {
      try {
        if (window[api] || window.chrome?.[api.split('.')[1]]) {
          allExtensions.add(`API:${api} [AVAILABLE]`);
        }
      } catch (e) {
        console.log(`API ${api} access blocked:`, e.message);
      }
    });
    
    // Check for common extension user agent strings
    const ua = navigator.userAgent.toLowerCase();
    const extensionUAKeywords = [
      'extension', 'addon', 'plugin', 'webstore', 'chrome-extension',
      'moz-extension', 'edge-extension', 'greasemonkey', 'tampermonkey',
      'userscript', 'violentmonkey', 'adblock', 'ublock', 'privacy',
      'security', 'developer', 'webstore', 'manifest', 'extension_id'
    ];
    
    extensionUAKeywords.forEach(keyword => {
      if (ua.includes(keyword)) {
        allExtensions.add(`UA:${keyword} [DETECTED]`);
      }
    });
    
    // Check for extension-related DOM elements
    try {
      const allClasses = document.querySelectorAll('*');
      allClasses.forEach(element => {
        const classList = element.className || '';
        const extensionPatterns = [
          'extension-icon', 'extension-icon-16', 'extension-icon-32',
          'extension-icon-48', 'extension-icon-128', 'extension-icon-256',
          'chrome-toolbar-icon', 'browser-action', 'page-action',
          'extension-popup', 'extension-options', 'extension-settings'
        ];
        
        extensionPatterns.forEach(pattern => {
          if (classList.toLowerCase().includes(pattern)) {
            allExtensions.add(`DOM:${pattern} [ELEMENT]`);
          }
        });
      });
    } catch (e) {
      console.log('DOM access limited:', e.message);
    }
    
    // Only add browser info if no extension data was found
    if (allExtensions.size === 0) {
      allExtensions.add(`Browser: ${navigator.userAgent.split(' ')[0]}`);
      allExtensions.add(`Platform: ${navigator.platform}`);
      allExtensions.add(`Language: ${navigator.language}`);
      allExtensions.add(`Online: ${navigator.onLine}`);
    }
    
    return Array.from(allExtensions);
  }

  // Update extension count and list display
  function updateExtensionDisplay() {
    const count = extensionDetector.getDetectedCount();
    const detectedList = extensionDetector.getDetectedList();
    const totalExtensions = getAllExtensions().length;
    const showAllExtensions = showAllExtensionsCheckbox.checked;
    
    extCountEl.textContent = count;
    document.getElementById('total-extensions').textContent = totalExtensions;
    blockedExtensionsEl.textContent = count;
    
    // Update extension list
    const extensionListEl = document.getElementById('extension-list');
    const extensionDetailsEl = document.getElementById('extension-details');
    
    if (count > 0 || totalExtensions > 0) {
      extWarningEl.style.display = count > 0 ? 'block' : 'none';
      extensionListEl.style.display = 'block';
      
      // Format extension details with highlighting
      let formattedExtensions = '';
      
      if (showAllExtensions) {
        // Show ALL extensions with blocked ones highlighted
        formattedExtensions += '<div style="color:#4ecdc4; font-weight:bold; margin-bottom:6px; font-size:12px;">📋 ALL_EXTENSIONS:</div>';
        
        // Get all extensions and mark which ones are blocked
        const allExtensions = getAllExtensions();
        allExtensions.forEach(ext => {
          const [type, name] = ext.split(':');
          let displayName = name;
          let isBlocked = detectedList.includes(ext);
          let riskLevel = isBlocked ? 'blocked' : 'safe';
          
          // Format display name
          if (type === 'localStorage' || type === 'sessionStorage') {
            displayName = `${type}: ${name}`;
          } else if (type === 'cookie') {
            displayName = `Cookie: ${name}`;
          } else if (type === 'API') {
            displayName = name.replace('chrome.', '').replace('browser.', '');
          }
          
          // Add icon and styling based on status
          const icon = isBlocked ? '🚫' : '✓';
          const statusText = isBlocked ? 'BLOCKED' : 'SAFE';
          const statusColor = isBlocked ? '#ff4444' : '#4ecdc4';
          
          formattedExtensions += `<div class="extension-item extension-${riskLevel}" style="border-left: 3px solid ${statusColor};">
            ${icon} ${displayName} <span style="color:${statusColor}; font-size:9px; font-weight:bold;">[${statusText}]</span>
          </div>`;
        });
        
        if (count > 0) {
          formattedExtensions += `<div style="color:#ff4444; font-weight:bold; margin-top:8px; font-size:11px; border-top:1px solid rgba(255,0,0,0.2); padding-top:4px;">
            ⚠️ ${count} extensions flagged as suspicious
          </div>`;
        }
      } else {
        // Show ONLY blocked extensions
        if (count > 0) {
          formattedExtensions += '<div style="color:#ff4444; font-weight:bold; margin-bottom:6px; font-size:12px;">🚨 BLOCKED_EXTENSIONS:</div>';
          
          detectedList.forEach(ext => {
            let displayName = ext;
            let riskLevel = 'blocked';
            
            // Special handling for ScreenCaptureAPI
            if (ext === 'ScreenCaptureAPI') {
              displayName = 'Screen Capture API (built-in browser capability)';
              riskLevel = 'medium';
            } else {
              const [type, name] = ext.split(':');
              if (type === 'API') {
                displayName = name.replace('chrome.', '').replace('browser.', '');
                riskLevel = 'medium';
              } else if (type === 'UA') {
                displayName = `User Agent: ${name}`;
                riskLevel = 'low';
              } else if (type === 'localStorage' || type === 'sessionStorage') {
                displayName = `${type}: ${name}`;
                riskLevel = 'medium';
              } else if (type === 'Common') {
                displayName = `Common Pattern: ${name}`;
                riskLevel = 'medium';
              } else if (type === 'Icon') {
                displayName = `Icon Pattern: ${name}`;
                riskLevel = 'low';
              } else if (type === 'Permission') {
                displayName = `Permission: ${name}`;
                riskLevel = 'high';
              } else if (type === 'Storage') {
                displayName = `Storage: ${name}`;
                riskLevel = 'medium';
              } else if (type === 'CaptureAPI') {
                displayName = `Capture API: ${name}`;
                riskLevel = 'high';
              }
            }
            
            formattedExtensions += `<div class="extension-item extension-${riskLevel}">
              🚫 ${displayName}
            </div>`;
          });
        } else {
          formattedExtensions += '<div style="color:#4ecdc4; font-size:12px; text-align:center; padding:10px;">✅ No suspicious extensions detected</div>';
        }
      }
      
      extensionDetailsEl.innerHTML = formattedExtensions;
      
      // Log to console for debugging
      console.log('Detected extensions:', detectedList);
      console.log('Total extensions:', totalExtensions);
      
      if (count > 0) {
        markDetected(`Detected ${count} suspicious extensions out of ${totalExtensions} total`);
      }
    } else {
      extWarningEl.style.display = 'none';
      extensionListEl.style.display = 'none';
      extensionDetailsEl.innerHTML = '';
    }
  }

  // Update extension display
  setInterval(updateExtensionDisplay, 1000);
  
  // Update display when checkbox is changed
  showAllExtensionsCheckbox.addEventListener('change', updateExtensionDisplay);
  
  // Open browser extensions page
  openExtensionsBtn.addEventListener('click', () => {
    try {
      // Try to open extensions page for different browsers
      const browser = navigator.userAgent.toLowerCase();
      
      if (browser.includes('chrome')) {
        window.open('chrome://extensions', '_blank');
      } else if (browser.includes('firefox')) {
        window.open('about:addons', '_blank');
      } else if (browser.includes('edge')) {
        window.open('edge://extensions', '_blank');
      } else if (browser.includes('safari')) {
        window.open('safari://extensions', '_blank');
      } else {
        // Fallback: try common extension URLs
        window.open('about:blank', '_blank');
        alert('Extension page not automatically supported for this browser. Please open your browser\'s extensions page manually.');
      }
    } catch (e) {
      alert('Could not open extensions page. Please open it manually through your browser settings.');
    }
  });

  // Create hidden video
  const video = document.createElement('video');
  video.crossOrigin = "anonymous";
  video.src = VIDEO_URL;
  video.muted = true;
  video.loop = true;
  video.playsInline = true;
  await video.play().catch(() => {});

  // Resize canvas to window
  function resize() {
    canvas.width = innerWidth * devicePixelRatio;
    canvas.height = innerHeight * devicePixelRatio;
    gl.viewport(0, 0, canvas.width, canvas.height);
  }

  // Setup WebGL with enhanced security
  const gl = canvas.getContext('webgl', { 
    preserveDrawingBuffer: false, 
    antialias: true,
    powerPreference: "high-performance"
  });
  if (!gl) { 
    alert('WebGL not available. Use a modern browser.'); 
    return; 
  }

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
    const s = gl.createShader(type);
    gl.shaderSource(s, src);
    gl.compileShader(s);
    if (!gl.getShaderParameter(s, gl.COMPILE_STATUS)) {
      console.error(gl.getShaderInfoLog(s));
      throw new Error('Shader compile error');
    }
    return s;
  }

  const prog = gl.createProgram();
  gl.attachShader(prog, compileShader(vs, gl.VERTEX_SHADER));
  gl.attachShader(prog, compileShader(fs, gl.FRAGMENT_SHADER));
  gl.linkProgram(prog);
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

  // Create texture
  const tex = gl.createTexture();
  gl.bindTexture(gl.TEXTURE_2D, tex);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);
  gl.pixelStorei(gl.UNPACK_FLIP_Y_WEBGL, true);

  // Enhanced uniforms
  const u_tex = gl.getUniformLocation(prog, 'u_tex');
  const u_time = gl.getUniformLocation(prog, 'u_time');
  const u_wmPos = gl.getUniformLocation(prog, 'u_wmPos');
  const u_wmAlpha = gl.getUniformLocation(prog, 'u_wmAlpha');
  const u_wmColor = gl.getUniformLocation(prog, 'u_wmColor');
  const u_encryptionKey = gl.getUniformLocation(prog, 'u_encryptionKey');

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

  // Enhanced rAF monitoring
  let lastRAF = performance.now();
  function rafCheck(now) {
    const dt = now - lastRAF;
    lastRAF = now;
    if (dt > FRAME_GAP_THRESHOLD_MS) {
      markDetected('High rAF gap: ' + Math.round(dt) + 'ms (possible OBS hook)');
    }
    requestAnimationFrame(rafCheck);
  }
  requestAnimationFrame(rafCheck);

  // Enhanced canvas capture detection
  const _origCanvasCapture = HTMLCanvasElement.prototype.captureStream;
  HTMLCanvasElement.prototype.captureStream = function(...args) {
    try {
      const id = (this === canvas) ? 'GL_CANVAS' : (this === overlay ? 'OVERLAY' : 'OTHER_CANVAS');
      markDetected('canvas.captureStream() called on ' + id);
    } catch (e) {}
    return _origCanvasCapture.call(this, ...args);
  };

  // Enhanced MediaRecorder detection
  const _origMediaRecorder = window.MediaRecorder;
  if (_origMediaRecorder) {
    window.MediaRecorder = function(stream, opt) {
      markDetected('MediaRecorder created (possible recording)');
      return new _origMediaRecorder(stream, opt);
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
    setTimeout(() => { blackout = true; }, 80);
    
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
      gl.bindTexture(gl.TEXTURE_2D, tex);
      gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, video);
    } catch (e) {
      // Ignore video readiness errors
    }

    // Enhanced uniforms with encryption
    const t = (performance.now() - start) / 1000;
    gl.uniform1f(u_time, t);
    gl.uniform1f(u_encryptionKey, encryptionKey.charCodeAt(0) / 255.0);
    
    // Enhanced watermark movement
    const sx = 0.82 + Math.sin(t * 0.7) * 0.03, sy = 0.88 + Math.cos(t * 0.53) * 0.02;
    gl.uniform2f(u_wmPos, sx, sy);
    gl.uniform1f(u_wmAlpha, 0.5);
    gl.uniform3f(u_wmColor, 1.0, 0.15, 0.15);

    gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);

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

  // Enhanced pixel probe with encryption check
  // function pixelProbe() {
  //   try {
  //     const px = new Uint8Array(4);
  //     gl.readPixels(0, 0, 1, 1, gl.RGBA, gl.UNSIGNED_BYTE, px);
  //     const sum = px[0] + px[1] + px[2] + px[3];
  //     if (sum === 0 && !video.paused && !video.ended) {
  //       markDetected('readPixels returned zero - possible capture hook');
  //     }
      
  //     // Check for pixel tampering
  //     if (sum > 0 && px[0] === 255 && px[1] === 255 && px[2] === 255) {
  //       markDetected('Unusual pixel pattern detected');
  //     }
  //   } catch (e) {
  //     markDetected('readPixels exception - possible capture hook');
  //   }
  //   setTimeout(pixelProbe, 5000); // Increased interval to reduce false positives
  // }
  // setTimeout(pixelProbe, 1200);

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

  // Initialize
  resize();
  window.addEventListener('resize', resize);
  statEl.textContent = 'OK';
  encStatEl.textContent = 'Active';

  // Cleanup on page unload
  window.addEventListener('beforeunload', () => {
    extensionDetector.stopMonitoring();
  });

})();
</script>
</body>
</html>
