# qrcodematching3
圣昌电子专用 测试6
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>双二维码验证系统</title>
    <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.min.js"></script>
    <style>
        /* 原有样式保持不变，新增以下样式 */
        #reset-btn {
            position: fixed;
            bottom: 80px;
            left: 50%;
            transform: translateX(-50%);
            padding: 10px 20px;
            background: #28a745;
            color: white;
            border: none;
            border-radius: 20px;
            font-size: 1em;
            cursor: pointer;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            z-index: 100;
        }
    </style>
</head>
<body>
    <!-- 原有HTML结构保持不变 -->
    <button id="reset-btn" class="hidden" onclick="location.reload()">重新开始</button>

    <script>
        // 修改后的处理函数
        function handleSecondCode(data) {
            secondCode = data;
            scanning = false;
            video.classList.add('hidden');
            
            // 标准化比较（去除首尾空白）
            const code1 = firstCode.trim();
            const code2 = secondCode.trim();
            
            let result = code1 === code2 ? "匹配成功 ✅" : "匹配失败 ❌";
            resultDiv.innerHTML = `
                <p>第一个二维码：<span class="code-content">${firstCode}</span></p>
                <p>第二个二维码：<span class="code-content">${secondCode}</span></p>
                <h2 style="color: ${code1 === code2 ? 'green' : 'red'}">${result}</h2>
            `;
            resultDiv.classList.remove('hidden');
            statusDiv.textContent = "扫描结果";
            
            // 显示重置按钮
            document.getElementById('reset-btn').classList.remove('hidden');
        }

        // 移除原有的setTimeout重置逻辑
    </script>
</body>
</html>
