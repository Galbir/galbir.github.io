<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Live Stream</title>
    <!-- Include HLS.js -->
    <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
</head>
<body>
    <h1>My Live Stream</h1>

    <!-- Video element -->
    <video id="video" controls autoplay muted width="640" height="360"></video>

    <!-- HLS.js script -->
    <script>
        const video = document.getElementById('video');
        const hlsUrl = 'http://10.13.4.134:8080/hls/demo.m3u8';

        if (Hls.isSupported()) {
            const hls = new Hls();
            hls.loadSource(hlsUrl);
            hls.attachMedia(video);
            hls.on(Hls.Events.MANIFEST_PARSED, () => video.play());
        } else if (video.canPlayType('application/vnd.apple.mpegurl')) {
            video.src = hlsUrl;
            video.addEventListener('loadedmetadata', () => video.play());
        }
    </script>
</body>
</html>
