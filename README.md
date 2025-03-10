# qrcodematching3
圣昌电子专用
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>二维码扫描</title>
    <style>
        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
            background-color: #f7f7f7;
        }
        #camera {
            width: 100%;
            max-width: 600px;
            margin-bottom: 20px;
        }
        #confirm-button {
            padding: 10px 20px;
            font-size: 16px;
            display: none; /* Initially hidden */
        }
    </style>
</head>
<body>
    <div id="camera"></div>
    <button id="confirm-button">确认第二个二维码</button>
    <div id="result"></div>

    <script>
        let scannedQRCode1 = null;
        let scannedQRCode2 = null;

        const html5QrCode = new Html5Qrcode("camera");

        function onScanSuccess(qrCodeMessage) {
            if (!scannedQRCode1) {
                scannedQRCode1 = qrCodeMessage;
                document.getElementById("result").innerText = `第一个二维码: ${scannedQRCode1}`;
                document.getElementById("confirm-button").style.display = "block"; // Show confirm button
                html5QrCode.stop();
            } else if (!scannedQRCode2) {
                scannedQRCode2 = qrCodeMessage;
                if (scannedQRCode1 === scannedQRCode2) {
                    document.getElementById("result").innerText = "两个二维码一致!";
                } else {
                    document.getElementById("result").innerText = "二维码不一致!";
                }
                html5QrCode.stop();
            }
        }

        const startScanning = () => {
            html5QrCode.start(
                { facingMode: "environment" }, // Use the device camera
                {
                    fps: 10,
                    qrbox: 250
                },
                onScanSuccess)
                .catch(err => {
                    console.error("Unable to start scanning.", err);
                });
        };

        document.getElementById("confirm-button").onclick = function () {
            startScanning();
            this.style.display = "none"; // Hide confirm button
        };

        startScanning(); // Start scanning the first QR code
    </script>
</body>
</html>
