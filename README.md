<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>相機APP - iOS 18+</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            -webkit-tap-highlight-color: transparent;
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
            background-color: #000;
            color: #fff;
            overflow: hidden;
            height: 100vh;
            position: relative;
            padding: env(safe-area-inset-top) env(safe-area-inset-right) env(safe-area-inset-bottom) env(safe-area-inset-left);
        }
        
        #device-check {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            padding: 20px;
            text-align: center;
            background-color: #000;
        }
        
        #device-check h2 {
            margin-bottom: 20px;
            color: #ff3b30;
        }
        
        #device-check p {
            margin-bottom: 10px;
            color: #ccc;
        }
        
        #camera-app {
            display: none;
            height: 100vh;
            flex-direction: column;
            background-color: #000;
        }
        
        /* 預覽區域 */
        #preview-container {
            flex: 1;
            position: relative;
            overflow: hidden;
            background-color: #111;
            border-radius: 10px;
            margin: 10px;
        }
        
        #video-preview {
            width: 100%;
            height: 100%;
            object-fit: cover;
            border-radius: 10px;
        }
        
        /* 控制區域 */
        #controls {
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            background-color: #000;
        }
        
        /* 模式切換 */
        #mode-switch {
            display: flex;
            background-color: #222;
            border-radius: 20px;
            padding: 4px;
            margin-bottom: 20px;
        }
        
        .mode-btn {
            padding: 8px 20px;
            border-radius: 16px;
            background: transparent;
            color: #fff;
            border: none;
            font-size: 16px;
            transition: all 0.3s ease;
        }
        
        .mode-btn.active {
            background-color: #007AFF;
            color: #000;
        }
        
        /* 鏡頭選擇 */
        #lens-selector {
            display: flex;
            justify-content: center;
            margin-bottom: 15px;
            gap: 10px;
        }
        
        .lens-btn {
            padding: 6px 12px;
            border-radius: 12px;
            background-color: #333;
            color: #fff;
            border: none;
            font-size: 14px;
            transition: all 0.3s ease;
        }
        
        .lens-btn.active {
            background-color: #007AFF;
            color: #000;
        }
        
        /* 錄影參數控制 */
        #video-controls {
            display: none;
            flex-direction: column;
            width: 100%;
            margin-bottom: 15px;
            padding: 10px;
            background-color: #1a1a1a;
            border-radius: 10px;
        }
        
        .control-group {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
        }
        
        .control-label {
            font-size: 14px;
            color: #ccc;
        }
        
        .control-value {
            font-size: 14px;
            color: #007AFF;
        }
        
        .control-slider {
            width: 100%;
            height: 4px;
            -webkit-appearance: none;
            background: #333;
            border-radius: 2px;
            outline: none;
        }
        
        .control-slider::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background: #007AFF;
            cursor: pointer;
        }
        
        /* 底部控制欄 */
        #bottom-controls {
            display: flex;
            justify-content: space-between;
            align-items: center;
            width: 100%;
            max-width: 400px;
        }
        
        /* 查看照片按鈕 */
        #gallery-btn {
            width: 50px;
            height: 50px;
            border-radius: 10px;
            background-color: #333;
            border: none;
            display: flex;
            justify-content: center;
            align-items: center;
            color: #fff;
            font-size: 20px;
        }
        
        /* 拍攝按鈕 */
        #capture-btn {
            width: 70px;
            height: 70px;
            border-radius: 50%;
            background-color: #007AFF;
            border: 4px solid rgba(255, 255, 255, 0.2);
            cursor: pointer;
            transition: all 0.2s ease;
        }
        
        #capture-btn:active {
            transform: scale(0.95);
            background-color: #0056CC;
        }
        
        /* 切換鏡頭按鈕 */
        #flip-camera-btn {
            width: 50px;
            height: 50px;
            border-radius: 10px;
            background-color: #333;
            border: none;
            display: flex;
            justify-content: center;
            align-items: center;
            color: #fff;
            font-size: 20px;
        }
        
        /* 錄影指示器 */
        #recording-indicator {
            display: none;
            align-items: center;
            margin-bottom: 15px;
        }
        
        #recording-dot {
            width: 12px;
            height: 12px;
            background-color: #ff3b30;
            border-radius: 50%;
            margin-right: 8px;
            animation: pulse 1.5s infinite;
        }
        
        @keyframes pulse {
            0% { opacity: 1; }
            50% { opacity: 0.3; }
            100% { opacity: 1; }
        }
        
        #recording-timer {
            font-size: 14px;
            color: #ff3b30;
        }
        
        /* 解析度顯示 */
        #resolution-info {
            font-size: 12px;
            color: #888;
            margin-top: 5px;
        }
        
        /* 儲存確認對話框 */
        #save-dialog {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.8);
            z-index: 1000;
            justify-content: center;
            align-items: center;
        }
        
        #save-dialog-content {
            background-color: #1c1c1e;
            border-radius: 14px;
            padding: 20px;
            width: 80%;
            max-width: 300px;
            text-align: center;
        }
        
        #save-dialog h3 {
            margin-bottom: 15px;
            font-size: 18px;
        }
        
        #save-dialog p {
            margin-bottom: 20px;
            color: #aaa;
            font-size: 14px;
        }
        
        #save-dialog-buttons {
            display: flex;
            justify-content: space-between;
        }
        
        .dialog-btn {
            flex: 1;
            padding: 12px;
            border-radius: 10px;
            border: none;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
        }
        
        #save-cancel {
            background-color: #333;
            color: #fff;
            margin-right: 10px;
        }
        
        #save-confirm {
            background-color: #007AFF;
            color: #fff;
            margin-left: 10px;
        }
        
        /* 狀態訊息 */
        #status-message {
            position: fixed;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background-color: rgba(0, 0, 0, 0.7);
            color: #fff;
            padding: 10px 20px;
            border-radius: 20px;
            font-size: 14px;
            display: none;
            z-index: 999;
        }
        
        /* 相簿預覽 */
        #gallery-preview {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: #000;
            z-index: 1001;
            flex-direction: column;
        }
        
        #gallery-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 15px;
            background-color: rgba(0, 0, 0, 0.7);
        }
        
        #gallery-close {
            background: none;
            border: none;
            color: #007AFF;
            font-size: 16px;
        }
        
        #gallery-content {
            flex: 1;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }
        
        #gallery-image {
            max-width: 100%;
            max-height: 100%;
            border-radius: 10px;
        }
    </style>
</head>
<body>
    <!-- 設備檢測頁面 -->
    <div id="device-check">
        <h2>設備不兼容</h2>
        <p>此應用程式僅支援iOS 18及以上版本的iPhone設備</p>
        <p>請使用相容設備開啟此應用程式</p>
    </div>
    
    <!-- 相機應用主界面 -->
    <div id="camera-app">
        <!-- 預覽區域 -->
        <div id="preview-container">
            <video id="video-preview" autoplay playsinline muted></video>
        </div>
        
        <!-- 控制區域 -->
        <div id="controls">
            <!-- 模式切換 -->
            <div id="mode-switch">
                <button id="photo-mode" class="mode-btn active">拍照</button>
                <button id="video-mode" class="mode-btn">錄影</button>
            </div>
            
            <!-- 鏡頭選擇 -->
            <div id="lens-selector">
                <!-- 鏡頭按鈕將由JavaScript動態生成 -->
            </div>
            
            <!-- 錄影參數控制 -->
            <div id="video-controls">
                <div class="control-group">
                    <span class="control-label">解析度</span>
                    <span class="control-value" id="resolution-value">4K</span>
                </div>
                <input type="range" min="0" max="2" value="2" class="control-slider" id="resolution-slider">
                
                <div class="control-group">
                    <span class="control-label">幀率</span>
                    <span class="control-value" id="fps-value">60 fps</span>
                </div>
                <input type="range" min="0" max="2" value="2" class="control-slider" id="fps-slider">
                
                <div class="control-group">
                    <span class="control-label">比特率</span>
                    <span class="control-value" id="bitrate-value">高</span>
                </div>
                <input type="range" min="0" max="2" value="1" class="control-slider" id="bitrate-slider">
            </div>
            
            <!-- 錄影指示器 -->
            <div id="recording-indicator">
                <div id="recording-dot"></div>
                <div id="recording-timer">00:00</div>
            </div>
            
            <!-- 底部控制欄 -->
            <div id="bottom-controls">
                <!-- 查看照片按鈕 -->
                <button id="gallery-btn">📷</button>
                
                <!-- 拍攝按鈕 -->
                <button id="capture-btn"></button>
                
                <!-- 切換鏡頭按鈕 -->
                <button id="flip-camera-btn">🔄</button>
            </div>
            
            <!-- 解析度資訊 -->
            <div id="resolution-info">4K 60fps</div>
        </div>
    </div>
    
    <!-- 儲存確認對話框 -->
    <div id="save-dialog">
        <div id="save-dialog-content">
            <h3>儲存媒體</h3>
            <p>是否要將拍攝的內容儲存到相簿？</p>
            <div id="save-dialog-buttons">
                <button id="save-cancel" class="dialog-btn">取消</button>
                <button id="save-confirm" class="dialog-btn">儲存</button>
            </div>
        </div>
    </div>
    
    <!-- 相簿預覽 -->
    <div id="gallery-preview">
        <div id="gallery-header">
            <button id="gallery-close">關閉</button>
            <h3>相簿</h3>
            <div></div> <!-- 佔位元素 -->
        </div>
        <div id="gallery-content">
            <img id="gallery-image" src="" alt="預覽圖片">
        </div>
    </div>
    
    <!-- 狀態訊息 -->
    <div id="status-message"></div>
    
    <script>
        // 檢測是否為iOS 18+的iPhone設備
        function isCompatibleDevice() {
            const userAgent = navigator.userAgent;
            const isiPhone = /iPhone/.test(userAgent) && !/iPad/.test(userAgent);
            
            // 檢測iOS版本 (簡化版本，實際應用中可能需要更精確的檢測)
            const iosVersionMatch = userAgent.match(/OS (\d+)_/);
            const iosVersion = iosVersionMatch ? parseInt(iosVersionMatch[1]) : 0;
            
            return isiPhone && iosVersion >= 18;
        }
        
        // 顯示狀態訊息
        function showStatusMessage(message, duration = 3000) {
            const statusEl = document.getElementById('status-message');
            statusEl.textContent = message;
            statusEl.style.display = 'block';
            
            setTimeout(() => {
                statusEl.style.display = 'none';
            }, duration);
        }
        
        // 初始化應用程式
        function initApp() {
            const deviceCheckEl = document.getElementById('device-check');
            const cameraAppEl = document.getElementById('camera-app');
            
            if (isCompatibleDevice()) {
                deviceCheckEl.style.display = 'none';
                cameraAppEl.style.display = 'flex';
                initCamera();
            } else {
                deviceCheckEl.style.display = 'flex';
                cameraAppEl.style.display = 'none';
            }
        }
        
        // 相機相關變數
        let stream = null;
        let mediaRecorder = null;
        let recordedChunks = [];
        let isRecording = false;
        let recordingTimer = null;
        let recordingStartTime = null;
        let currentMode = 'photo'; // 'photo' 或 'video'
        let currentCamera = 'environment'; // 'user' 或 'environment'
        let availableLenses = [];
        let currentLens = 'wide';
        
        // 錄影參數
        let videoSettings = {
            resolution: 2, // 0: 1080p, 1: 2.7K, 2: 4K
            fps: 2, // 0: 24, 1: 30, 2: 60
            bitrate: 1 // 0: 低, 1: 中, 2: 高
        };
        
        // 初始化相機
        async function initCamera() {
            try {
                // 獲取可用鏡頭
                await getAvailableCameras();
                
                // 設置預設鏡頭
                await switchCamera('environment');
                
                // 設置事件監聽器
                setupEventListeners();
                
                // 初始化錄影參數控制
                initVideoControls();
                
                showStatusMessage('相機已就緒');
            } catch (error) {
                console.error('無法訪問相機:', error);
                showStatusMessage('無法訪問相機，請檢查權限設定', 5000);
            }
        }
        
        // 獲取可用鏡頭
        async function getAvailableCameras() {
            try {
                const devices = await navigator.mediaDevices.enumerateDevices();
                const videoDevices = devices.filter(device => device.kind === 'videoinput');
                
                // 模擬不同鏡頭類型 (實際應用中需要更精確的檢測)
                availableLenses = [
                    { id: 'ultrawide', label: '0.5x', facing: 'environment' },
                    { id: 'wide', label: '1x', facing: 'environment' },
                    { id: 'telephoto', label: '2x', facing: 'environment' }
                ];
                
                // 生成鏡頭選擇按鈕
                const lensSelector = document.getElementById('lens-selector');
                lensSelector.innerHTML = '';
                
                availableLenses.forEach(lens => {
                    const button = document.createElement('button');
                    button.className = `lens-btn ${lens.id === currentLens ? 'active' : ''}`;
                    button.textContent = lens.label;
                    button.dataset.lensId = lens.id;
                    button.addEventListener('click', () => switchLens(lens.id));
                    lensSelector.appendChild(button);
                });
                
            } catch (error) {
                console.error('獲取鏡頭失敗:', error);
            }
        }
        
        // 切換鏡頭
        async function switchLens(lensId) {
            currentLens = lensId;
            
            // 更新按鈕狀態
            document.querySelectorAll('.lens-btn').forEach(btn => {
                btn.classList.toggle('active', btn.dataset.lensId === lensId);
            });
            
            // 重新啟動相機以應用新的鏡頭設置
            await restartCamera();
            
            showStatusMessage(`已切換到${lensId === 'ultrawide' ? '超廣角' : lensId === 'wide' ? '廣角' : '望遠'}鏡頭`);
        }
        
        // 切換前後鏡頭
        async function switchCamera(facingMode) {
            currentCamera = facingMode;
            await restartCamera();
            showStatusMessage(`已切換到${facingMode === 'environment' ? '後置' : '前置'}鏡頭`);
        }
        
        // 重新啟動相機
        async function restartCamera() {
            if (stream) {
                stream.getTracks().forEach(track => track.stop());
            }
            
            try {
                // 設置約束條件
                const constraints = {
                    video: {
                        facingMode: currentCamera,
                        width: { ideal: 3840 },
                        height: { ideal: 2160 },
                        frameRate: { ideal: 60 }
                    },
                    audio: false // 禁用麥克風，避免雜音
                };
                
                // 請求相機權限
                stream = await navigator.mediaDevices.getUserMedia(constraints);
                
                // 設置視頻預覽
                const videoPreview = document.getElementById('video-preview');
                videoPreview.srcObject = stream;
                
            } catch (error) {
                console.error('重新啟動相機失敗:', error);
                showStatusMessage('相機啟動失敗', 5000);
            }
        }
        
        // 初始化錄影參數控制
        function initVideoControls() {
            const resolutionSlider = document.getElementById('resolution-slider');
            const fpsSlider = document.getElementById('fps-slider');
            const bitrateSlider = document.getElementById('bitrate-slider');
            
            resolutionSlider.addEventListener('input', updateVideoSettings);
            fpsSlider.addEventListener('input', updateVideoSettings);
            bitrateSlider.addEventListener('input', updateVideoSettings);
            
            updateVideoSettings();
        }
        
        // 更新錄影參數
        function updateVideoSettings() {
            const resolutionSlider = document.getElementById('resolution-slider');
            const fpsSlider = document.getElementById('fps-slider');
            const bitrateSlider = document.getElementById('bitrate-slider');
            
            videoSettings.resolution = parseInt(resolutionSlider.value);
            videoSettings.fps = parseInt(fpsSlider.value);
            videoSettings.bitrate = parseInt(bitrateSlider.value);
            
            // 更新顯示值
            document.getElementById('resolution-value').textContent = 
                ['1080p', '2.7K', '4K'][videoSettings.resolution];
            document.getElementById('fps-value').textContent = 
                ['24 fps', '30 fps', '60 fps'][videoSettings.fps];
            document.getElementById('bitrate-value').textContent = 
                ['低', '中', '高'][videoSettings.bitrate];
            
            // 更新解析度資訊
            document.getElementById('resolution-info').textContent = 
                `${['1080p', '2.7K', '4K'][videoSettings.resolution]} ${['24', '30', '60'][videoSettings.fps]}fps`;
        }
        
        // 設置事件監聽器
        function setupEventListeners() {
            // 模式切換按鈕
            document.getElementById('photo-mode').addEventListener('click', () => switchMode('photo'));
            document.getElementById('video-mode').addEventListener('click', () => switchMode('video'));
            
            // 拍攝按鈕
            document.getElementById('capture-btn').addEventListener('click', handleCapture);
            
            // 切換鏡頭按鈕
            document.getElementById('flip-camera-btn').addEventListener('click', () => {
                switchCamera(currentCamera === 'environment' ? 'user' : 'environment');
            });
            
            // 查看照片按鈕
            document.getElementById('gallery-btn').addEventListener('click', showGallery);
            
            // 儲存對話框按鈕
            document.getElementById('save-cancel').addEventListener('click', closeSaveDialog);
            document.getElementById('save-confirm').addEventListener('click', confirmSave);
            
            // 相簿關閉按鈕
            document.getElementById('gallery-close').addEventListener('click', closeGallery);
        }
        
        // 切換模式
        function switchMode(mode) {
            currentMode = mode;
            
            // 更新按鈕狀態
            document.getElementById('photo-mode').classList.toggle('active', mode === 'photo');
            document.getElementById('video-mode').classList.toggle('active', mode === 'video');
            
            // 顯示/隱藏錄影控制
            document.getElementById('video-controls').style.display = mode === 'video' ? 'flex' : 'none';
            
            // 更新解析度顯示
            updateVideoSettings();
            
            // 如果正在錄影，停止錄影
            if (mode === 'photo' && isRecording) {
                stopRecording();
            }
            
            showStatusMessage(`已切換到${mode === 'photo' ? '拍照' : '錄影'}模式`);
        }
        
        // 處理拍攝
        function handleCapture() {
            if (currentMode === 'photo') {
                takePhoto();
            } else {
                if (isRecording) {
                    stopRecording();
                } else {
                    startRecording();
                }
            }
        }
        
        // 拍照
        function takePhoto() {
            const videoPreview = document.getElementById('video-preview');
            const canvas = document.createElement('canvas');
            const context = canvas.getContext('2d');
            
            // 設置canvas尺寸與視頻相同
            canvas.width = videoPreview.videoWidth;
            canvas.height = videoPreview.videoHeight;
            
            // 繪製當前視頻幀到canvas
            context.drawImage(videoPreview, 0, 0, canvas.width, canvas.height);
            
            // 將canvas轉換為圖片數據URL
            const imageDataURL = canvas.toDataURL('image/jpeg');
            
            // 顯示儲存對話框
            showSaveDialog(imageDataURL, 'photo');
            
            showStatusMessage('照片已拍攝');
        }
        
        // 開始錄影
        function startRecording() {
            if (!stream) return;
            
            try {
                recordedChunks = [];
                
                // 創建MediaRecorder實例
                const options = {
                    mimeType: 'video/mp4; codecs="avc1.42E01E"',
                    videoBitsPerSecond: [1000000, 3000000, 5000000][videoSettings.bitrate] // 根據設置調整比特率
                };
                
                mediaRecorder = new MediaRecorder(stream, options);
                
                // 收集錄影數據
                mediaRecorder.ondataavailable = (event) => {
                    if (event.data.size > 0) {
                        recordedChunks.push(event.data);
                    }
                };
                
                // 錄影結束時的處理
                mediaRecorder.onstop = () => {
                    const blob = new Blob(recordedChunks, { type: 'video/mp4' });
                    const videoURL = URL.createObjectURL(blob);
                    
                    // 顯示儲存對話框
                    showSaveDialog(videoURL, 'video');
                };
                
                // 開始錄影
                mediaRecorder.start(1000); // 每1秒收集一次數據
                isRecording = true;
                
                // 顯示錄影指示器
                document.getElementById('recording-indicator').style.display = 'flex';
                
                // 開始計時器
                recordingStartTime = Date.now();
                updateRecordingTimer();
                recordingTimer = setInterval(updateRecordingTimer, 1000);
                
                showStatusMessage('錄影已開始');
            } catch (error) {
                console.error('開始錄影失敗:', error);
                showStatusMessage('開始錄影失敗', 5000);
            }
        }
        
        // 停止錄影
        function stopRecording() {
            if (mediaRecorder && isRecording) {
                mediaRecorder.stop();
                isRecording = false;
                
                // 隱藏錄影指示器
                document.getElementById('recording-indicator').style.display = 'none';
                
                // 清除計時器
                clearInterval(recordingTimer);
                
                showStatusMessage('錄影已停止');
            }
        }
        
        // 更新錄影計時器
        function updateRecordingTimer() {
            if (!recordingStartTime) return;
            
            const elapsedTime = Math.floor((Date.now() - recordingStartTime) / 1000);
            const minutes = Math.floor(elapsedTime / 60).toString().padStart(2, '0');
            const seconds = (elapsedTime % 60).toString().padStart(2, '0');
            
            document.getElementById('recording-timer').textContent = `${minutes}:${seconds}`;
        }
        
        // 顯示相簿
        function showGallery() {
            // 這裡應該顯示實際的相簿內容
            // 目前僅顯示示例圖片
            document.getElementById('gallery-image').src = 'https://via.placeholder.com/800x600/333333/007AFF?text=相簿預覽';
            document.getElementById('gallery-preview').style.display = 'flex';
        }
        
        // 關閉相簿
        function closeGallery() {
            document.getElementById('gallery-preview').style.display = 'none';
        }
        
        // 顯示儲存對話框
        function showSaveDialog(mediaURL, mediaType) {
            const saveDialog = document.getElementById('save-dialog');
            saveDialog.style.display = 'flex';
            
            // 儲存媒體URL和類型供確認時使用
            saveDialog.dataset.mediaUrl = mediaURL;
            saveDialog.dataset.mediaType = mediaType;
        }
        
        // 關閉儲存對話框
        function closeSaveDialog() {
            const saveDialog = document.getElementById('save-dialog');
            saveDialog.style.display = 'none';
            
            // 清理URL對象
            if (saveDialog.dataset.mediaType === 'video') {
                URL.revokeObjectURL(saveDialog.dataset.mediaUrl);
            }
            
            // 清除儲存的數據
            delete saveDialog.dataset.mediaUrl;
            delete saveDialog.dataset.mediaType;
        }
        
        // 確認儲存
        function confirmSave() {
            const saveDialog = document.getElementById('save-dialog');
            const mediaURL = saveDialog.dataset.mediaUrl;
            const mediaType = saveDialog.dataset.mediaType;
            
            // 這裡應該實現實際的儲存邏輯
            // 由於瀏覽器限制，無法直接存取相簿
            // 實際應用中需要使用Cordova、Capacitor等框架
            
            if (mediaType === 'photo') {
                // 模擬照片儲存
                const link = document.createElement('a');
                link.download = `photo_${Date.now()}.jpg`;
                link.href = mediaURL;
                link.click();
                showStatusMessage('照片已儲存');
            } else {
                // 模擬影片儲存
                const link = document.createElement('a');
                link.download = `video_${Date.now()}.mp4`;
                link.href = mediaURL;
                link.click();
                showStatusMessage('影片已儲存');
            }
            
            closeSaveDialog();
        }
        
        // 初始化應用
        document.addEventListener('DOMContentLoaded', initApp);
    </script>
</body>
</html>
