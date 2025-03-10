# qrcodematching3
圣昌电子专用 测试7
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>双码校验系统</title>
    <script src="https://cdn.jsdelivr.net/npm/@ericblade/quagga2/dist/quagga.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
            background: #f5f5f5;
        }

        .container {
            width: 100%;
            min-height: 100vh;
            padding: 20px;
            position: relative;
        }

        .scan-section {
            display: none;
            flex-direction: column;
            align-items: center;
            gap: 20px;
        }

        .active {
            display: flex;
        }

        #preview {
            width: 100%;
            max-width: 600px;
            height: 40vh;
            background: #000;
            border-radius: 12px;
            overflow: hidden;
            position: relative;
        }

        .result-box {
            width: 100%;
            max-width: 600px;
            padding: 20px;
            background: #fff;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
            word-break: break-all;
        }

        .confirm-btn {
            width: 90%;
            max-width: 600px;
            padding: 16px;
            background: #007AFF;
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 17px;
            cursor: pointer;
            position: fixed;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            transition: opacity 0.2s;
        }

        .confirm-btn:active {
            opacity: 0.8;
        }

        .status-text {
            font-size: 20px;
            color: #333;
            margin: 15px 0;
            text-align: center;
        }

        .camera-frame {
            border: 2px solid #ffffff;
            border-radius: 12px;
            box-shadow: 0 0 15px rgba(0,0,0,0.2);
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- 第一步：扫描第一个二维码 -->
        <div class="scan-section active" id="step1">
            <h2 class="status-text">请扫描第一个二维码</h2>
            <div class="camera-frame">
                <div id="preview"></div>
            </div>
        </div>

        <!-- 第二步：显示第一个结果 -->
        <div class="scan-section" id="step2">
            <h2 class="status-text">已扫描内容</h2>
            <div class="result-box" id="result1"></div>
            <button class="confirm-btn" onclick="startSecondScan()">确认并扫描第二个二维码</button>
        </div>

        <!-- 第三步：扫描第二个二维码 -->
        <div class="scan-section" id="step3">
            <h2 class="status-text">请扫描第二个二维码</h2>
            <div class="camera-frame">
                <div id="preview2"></div>
            </div>
        </div>

        <!-- 第四步：显示比对结果 -->
        <div class="scan-section" id="step4">
            <h2 class="status-text">校验结果</h2>
            <div class="result-box" id="finalResult"></div>
        </div>
    </div>

<script>
    let firstCode = '';
    let secondCode = '';
    let currentScanner = null;

    // 初始化扫描器
    function initScanner(previewId, onDetected) {
        const config = {
            inputStream: {
                name: "Live",
                type: "LiveStream",
                target: document.querySelector(`#${previewId}`),
                constraints: {
                    width: { ideal: 1280 },
                    height: { ideal: 720 },
                    facingMode: "environment"
                }
            },
            decoder: {
                readers: ["qrcode_reader"]
            }
        };

        Quagga.init(config, err => {
            if (err) {
                console.error(err);
                alert('摄像头初始化失败');
                return;
            }
            Quagga.start();
        });

        Quagga.onDetected(result => {
            if (result.codeResult) {
                onDetected(result.codeResult.code);
                Quagga.stop();
            }
        });
    }

    // 第一步扫描
    function startFirstScan() {
        initScanner('preview', (data) => {
            firstCode = data;
            document.getElementById('result1').textContent = data;
            switchStep(2);
        });
    }

    // 第二步扫描
    function startSecondScan() {
        switchStep(3);
        initScanner('preview2', (data) => {
            secondCode = data;
            compareCodes();
            switchStep(4);
        });
    }

    // 比对结果
    function compareCodes() {
        const resultDiv = document.getElementById('finalResult');
        resultDiv.innerHTML = firstCode === secondCode 
            ? `<div style="color:#28a745; text-align:center;">
                <h3>✅ 验证通过</h3>
                <p>两个二维码内容一致</p >
               </div>`
            : `<div style="color:#dc3545; text-align:center;">
                <h3>❌ 验证失败</h3>
                <p>第一个二维码：${firstCode}</p >
                <p>第二个二维码：${secondCode}</p >
               </div>`;
    }

    // 切换步骤
    function switchStep(step) {
        document.querySelectorAll('.scan-section').forEach(el => {
            el.classList.remove('active');
        });
        document.getElementById(`step${step}`).classList.add('active');
    }

    // 初始化
    window.onload = startFirstScan;
</script>
</body>
</html>
