<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Live Stream Test</title>
    <!-- HLS.js library -->
    <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
    <style>
        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
            background: #000;
            color: #fff;
            font-family: Arial, sans-serif;
        }
        video {
            max-width: 100%;
            border: 2px solid #fff;
        }
    </style>
</head>
<body>
    <h1>Live Stream Test</h1>
    <!-- Video element -->
    <video id="video" controls autoplay muted playsinline></video>

    <script>
        const video = document.getElementById('video');
        // Replace with your LAN IP and stream key
        const hlsUrl = 'http://10.13.4.134:8080/hls/demo.m3u8';

        if (Hls.isSupported()) {
            const hls = new Hls();
            hls.loadSource(hlsUrl);
            hls.attachMedia(video);
            hls.on(Hls.Events.MANIFEST_PARSED, () => {
                video.play().catch(() => console.log('Autoplay blocked, tap video to play'));
            });
        } else if (video.canPlayType('application/vnd.apple.mpegurl')) {
            video.src = hlsUrl;
            video.addEventListener('loadedmetadata', () => {
                video.play().catch(() => console.log('Autoplay blocked, tap video to play'));
            });
        }
    </script>
</body>
</html>
