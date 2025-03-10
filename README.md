# qrcodematching3
圣昌电子专用 测试7
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>双条形码验证系统</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/quagga/0.12.1/quagga.min.js"></script>
    <style>
        /* 保持原有样式不变 */
        /* 只需修改所有"二维码"文字为"条形码" */
    </style>
</head>
<body>
    <div id="camera-container">
        <div id="interactive" class="viewport"></div>
    </div>
    <div id="status">请扫描第一个条形码</div>
    <div id="result" class="hidden"></div>
    <button id="confirm-btn" class="hidden" onclick="startSecondScan()">确认并扫描第二个条形码</button>

    <script>
        let firstCode = null;
        let secondCode = null;
        let scannerActive = false;

        // 初始化条形码扫描器
        function initScanner() {
            Quagga.init({
                inputStream: {
                    name: "Live",
                    type: "LiveStream",
                    target: document.querySelector('#interactive'),
                    constraints: {
                        width: { min: 640 },
                        height: { min: 480 },
                        facingMode: "environment"
                    }
                },
                decoder: {
                    readers: [
                        "code_128_reader",
                        "ean_reader",
                        "ean_8_reader",
                        "code_39_reader",
                        "upc_reader"
                    ]
                }
            }, function(err) {
                if (err) {
                    alert("条形码扫描器初始化失败: " + err);
                    return;
                }
                Quagga.start();
            });

            Quagga.onDetected(function(result) {
                if (!scannerActive) return;
                
                const code = result.codeResult.code;
                if (!firstCode) {
                    handleFirstCode(code);
                } else if (!secondCode) {
                    handleSecondCode(code);
                }
            });
        }

        // 修改后的处理函数
        function handleFirstCode(data) {
            scannerActive = false;
            firstCode = String(data);
            document.getElementById('camera-container').classList.add('hidden');
            resultDiv.textContent = `第一个条形码内容：${data}`;
            resultDiv.classList.remove('hidden');
            statusDiv.textContent = "请确认第一个条形码内容";
            confirmBtn.classList.remove('hidden');
        }

        function startSecondScan() {
            scannerActive = true;
            confirmBtn.classList.add('hidden');
            resultDiv.classList.add('hidden');
            document.getElementById('camera-container').classList.remove('hidden');
            statusDiv.textContent = "请扫描第二个条形码";
        }

        function handleSecondCode(data) {
            scannerActive = false;
            secondCode = String(data);
            document.getElementById('camera-container').classList.add('hidden');
            
            let result = firstCode === secondCode ? "✅ 一致" : "❌ 不一致";
            resultDiv.innerHTML = `
                <p>第一个条形码：${firstCode}</p>
                <p>第二个条形码：${secondCode}</p>
                <h2>${result}</h2>
            `;
            resultDiv.classList.remove('hidden');
            statusDiv.textContent = "扫描结果";

            setTimeout(() => {
                firstCode = null;
                secondCode = null;
                resultDiv.classList.add('hidden');
                statusDiv.textContent = "请扫描第一个条形码";
                scannerActive = true;
                document.getElementById('camera-container').classList.remove('hidden');
            }, 3000);
        }

        // 初始化
        window.addEventListener('load', () => {
            initScanner();
            scannerActive = true;
        });
    </script>
</body>
</html>
