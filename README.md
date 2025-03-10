# qrcodematching3
<!DOCTYPE html>
<html>
<head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>双二维码比对扫描器</title>
    <script src="https://unpkg.com/@zxing/library@latest"></script>
    <style>
        #scanner-container { 
            position: relative;
            width: 100%;
            max-width: 600px;
            margin: auto;
        }
        #preview {
            width: 100%;
            height: auto;
        }
        .status-light {
            width: 40px;
            height: 40px;
            border-radius: 50%;
            position: absolute;
            top: 20px;
            right: 20px;
            border: 2px solid white;
        }
        .result-box {
            padding: 10px;
            margin: 10px;
            background: rgba(255,255,255,0.9);
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <div id="scanner-container">
        <video id="preview" playsinline></video>
        <div id="statusLight" class="status-light"></div>
        <div id="results">
            <div class="result-box">二维码1：<span id="result1">等待扫描...</span></div>
            <div class="result-box">二维码2：<span id="result2">等待扫描...</span></div>
        </div>
    </div>

    <script>
        const codeReader = new ZXing.BrowserMultiFormatReader();
        let scanning = false;
        let lastResults = { qr1: null, qr2: null };

        async function startScanning() {
            try {
                const videoInputDevices = await ZXing.BrowserCodeReader.listVideoInputDevices();
                await codeReader.decodeFromVideoDevice(
                    videoInputDevices[0].deviceId,
                    'preview',
                    (result, error) => {
                        if (result) {
                            handleResult(result.text);
                        }
                    }
                );
                scanning = true;
            } catch (error) {
                console.error(error);
            }
        }

        function handleResult(text) {
            if (!lastResults.qr1) {
                lastResults.qr1 = text;
                document.getElementById('result1').textContent = text;
            } else if (!lastResults.qr2 && text !== lastResults.qr1) {
                lastResults.qr2 = text;
                document.getElementById('result2').textContent = text;
                checkResults();
            } else {
                // 当两个二维码都扫描到后，重置检测
                if (text === lastResults.qr1 || text === lastResults.qr2) {
                    checkResults();
                }
            }
        }

        function checkResults() {
            const statusLight = document.getElementById('statusLight');
            if (lastResults.qr1 && lastResults.qr2) {
                if (lastResults.qr1 === lastResults.qr2) {
                    statusLight.style.backgroundColor = '#00ff00';
                } else {
                    statusLight.style.backgroundColor = '#ff0000';
                }
            } else {
                statusLight.style.backgroundColor = '#ff0000';
            }
            
            // 2秒后重置扫描结果
            setTimeout(() => {
                lastResults = { qr1: null, qr2: null };
                document.getElementById('result1').textContent = '等待扫描...';
                document.getElementById('result2').textContent = '等待扫描...';
                statusLight.style.backgroundColor = '';
            }, 2000);
        }

        // 初始化摄像头
        window.onload = () => {
            startScanning();
        }

        // 清理摄像头
        window.onbeforeunload = () => {
            if (scanning) {
                codeReader.reset();
            }
        }
    </script>
</body>
</html>
