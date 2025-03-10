# qrcodematching3
圣昌电子专用
<!DOCTYPE html>
<html>
<head>
    <title>二维码比较器</title>
    <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.min.js"></script>
    <style>
        #video {
            width: 100%;
            max-width: 400px;
            margin: 10px 0;
        }
        .button-container {
            margin: 10px 0;
        }
        button {
            padding: 10px;
            margin: 5px;
            font-size: 16px;
        }
        #result {
            white-space: pre-wrap;
            font-size: 18px;
            margin: 10px 0;
        }
    </style>
</head>
<body>
    <video id="video" playsinline></video>
    <div class="button-container">
        <button id="scanFirst">扫描第一个二维码</button>
        <button id="scanSecond" disabled>扫描第二个二维码</button>
    </div>
    <div id="result"></div>

    <script>
        const video = document.getElementById('video');
        const scanFirstBtn = document.getElementById('scanFirst');
        const scanSecondBtn = document.getElementById('scanSecond');
        const resultDiv = document.getElementById('result');
        
        let scanning = false;
        let firstCode = null;
        let secondCode = null;

        // 初始化按钮状态
        scanSecondBtn.disabled = true;

        scanFirstBtn.addEventListener('click', () => startScanning('first'));
        scanSecondBtn.addEventListener('click', () => startScanning('second'));

        function startScanning(step) {
            // 停止之前的摄像头流
            if (video.srcObject) {
                video.srcObject.getTracks().forEach(track => track.stop());
            }

            resultDiv.textContent = step === 'first' ? '正在扫描第一个二维码...' : '正在扫描第二个二维码...';
            
            navigator.mediaDevices.getUserMedia({
                video: {
                    facingMode: 'environment',
                    width: { ideal: 1280 },
                    height: { ideal: 720 }
                }
            }).then(stream => {
                video.srcObject = stream;
                video.play();
                scanning = true;
                scanFrame(step);
            }).catch(err => {
                resultDiv.textContent = '无法访问摄像头: ' + err;
            });
        }

        function scanFrame(step) {
            if (!scanning) return;

            const canvas = document.createElement('canvas');
            const context = canvas.getContext('2d');
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            context.drawImage(video, 0, 0, canvas.width, canvas.height);

            const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
            const code = jsQR(imageData.data, imageData.width, imageData.height);

            if (code) {
                scanning = false;
                video.srcObject.getTracks().forEach(track => track.stop());
                
                if (step === 'first') {
                    firstCode = code.data;
                    resultDiv.textContent = `第一个二维码内容：\n${firstCode}`;
                    scanSecondBtn.disabled = false;
                    scanFirstBtn.disabled = true;
                } else {
                    secondCode = code.data;
                    resultDiv.textContent += `\n\n第二个二维码内容：\n${secondCode}`;
                    
                    const isMatch = firstCode === secondCode;
                    resultDiv.textContent += `\n\n比较结果：${isMatch ? '✅ 二维码内容一致' : '❌ 二维码内容不同'}`;
                    
                    scanFirstBtn.disabled = false;
                    scanSecondBtn.disabled = true;
                    // 重置状态以便重新扫描
                    firstCode = null;
                    secondCode = null;
                }
            } else {
                requestAnimationFrame(() => scanFrame(step));
            }
        }
    </script>
</body>
</html>
