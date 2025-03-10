# qrcodematching3
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>二维码对比工具</title>
    <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
        }
        #preview {
            width: 100%;
            height: 300px;
            background: #333;
            position: relative;
        }
        .status {
            padding: 10px;
            margin: 10px 0;
            border-radius: 5px;
        }
        .scanning {
            background: #e3f2fd;
        }
        .result {
            background: #e8f5e9;
        }
        button {
            padding: 12px 24px;
            font-size: 16px;
            margin: 5px;
            background: #2196F3;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        #comparison {
            font-size: 18px;
            font-weight: bold;
            margin: 20px 0;
            padding: 15px;
            text-align: center;
        }
    </style>
</head>
<body>
    <h2>二维码对比工具</h2>
    <div id="preview">
        <video id="video" playsinline style="width: 100%;"></video>
    </div>
    
    <div id="status1" class="status"></div>
    <div id="status2" class="status"></div>
    
    <button id="startBtn1" onclick="startScan(1)">开始扫描第一个二维码</button>
    <button id="startBtn2" onclick="startScan(2)" disabled>开始扫描第二个二维码</button>
    <div id="comparison"></div>

    <script>
        let video = document.getElementById("video");
        let canvas = document.createElement("canvas");
        let ctx = canvas.getContext("2d");
        let currentScan = 0;
        let result1 = "";
        let result2 = "";
        let scanning = false;

        async function setupCamera() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ 
                    video: { facingMode: "environment" } 
                });
                video.srcObject = stream;
                await video.play();
            } catch (err) {
                alert("无法访问摄像头，请检查权限设置");
            }
        }

        function scanQR() {
            if (!scanning) return;
            
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            ctx.drawImage(video, 0, 0);
            
            const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
            const code = jsQR(imageData.data, imageData.width, imageData.height);
            
            if (code) {
                handleResult(code.data);
            } else if (scanning) {
                requestAnimationFrame(scanQR);
            }
        }

        function handleResult(data) {
            scanning = false;
            if (currentScan === 1) {
                result1 = data;
                document.getElementById("status1").innerHTML = 
                    `第一个二维码内容：<strong>${data}</strong>`;
                document.getElementById("startBtn2").disabled = false;
            } else {
                result2 = data;
                document.getElementById("status2").innerHTML = 
                    `第二个二维码内容：<strong>${data}</strong>`;
                compareResults();
            }
            updateUI();
        }

        function startScan(step) {
            currentScan = step;
            scanning = true;
            updateUI();
            document.getElementById(`startBtn${step}`).disabled = true;
            scanQR();
        }

        function compareResults() {
            const comparisonDiv = document.getElementById("comparison");
            if (result1 === result2) {
                comparisonDiv.style.backgroundColor = "#C8E6C9";
                comparisonDiv.textContent = "✅ 两个二维码内容一致";
            } else {
                comparisonDiv.style.backgroundColor = "#FFCDD2";
                comparisonDiv.textContent = "❌ 二维码内容不一致";
            }
        }

        function updateUI() {
            const statusDivs = document.querySelectorAll(".status");
            statusDivs.forEach(div => div.classList.remove("scanning", "result"));
            
            if (scanning) {
                document.getElementById(`status${currentScan}`).classList.add("scanning");
                document.getElementById(`status${currentScan}`).textContent = 
                    `正在扫描第 ${currentScan} 个二维码...`;
            } else {
                document.getElementById(`status${currentScan}`).classList.add("result");
            }
        }

        // 初始化摄像头
        setupCamera();
    </script>
</body>
</html>
