<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>相機APP</title>
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
        }
        
        #video-preview {
            width: 100%;
            height: 100%;
            object-fit: cover;
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
        
        /* 拍攝按鈕 */
        #capture-btn {
            width: 70px;
            height: 70px;
            border-radius: 50%;
            background-color: #007AFF;
            border: 4px solid rgba(255, 255, 255, 0.2);
            margin-bottom: 10px;
            cursor: pointer;
            transition: all 0.2s ease;
        }
        
        #capture-btn:active {
            transform: scale(0.95);
            background-color: #0056CC;
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
    </style>
</head>
<body>
    <!-- 設備檢測頁面 -->
    <div id="device-check">
        <h2>設備不兼容</h2>
        <p>此應用程式僅支援iPhone設備使用</p>
        <p>請使用iPhone開啟此應用程式</p>
    </div>
    
    <!-- 相機應用主界面 -->
    <div id="camera-app">
        <!-- 預覽區域 -->
        <div id="preview-container">
            <video id="video-preview" autoplay playsinline></video>
        </div>
        
        <!-- 控制區域 -->
        <div id="controls">
            <!-- 模式切換 -->
            <div id="mode-switch">
                <button id="photo-mode" class="mode-btn active">拍照</button>
                <button id="video-mode" class="mode-btn">錄影</button>
            </div>
            
            <!-- 錄影指示器 -->
            <div id="recording-indicator">
                <div id="recording-dot"></div>
                <div id="recording-timer">00:00</div>
            </div>
            
            <!-- 拍攝按鈕 -->
            <button id="capture-btn"></button>
            
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
    
    <!-- 狀態訊息 -->
    <div id="status-message"></div>
    
    <script>
        // 檢測是否為iPhone設備
        function isiPhone() {
            return /iPhone/.test(navigator.userAgent) && !/iPad/.test(navigator.userAgent);
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
            
            if (isiPhone()) {
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
        
        // 初始化相機
        async function initCamera() {
            try {
                // 請求相機權限
                stream = await navigator.mediaDevices.getUserMedia({ 
                    video: { 
                        width: { ideal: 3840 }, // 4K
                        height: { ideal: 2160 },
                        frameRate: { ideal: 60 }
                    }, 
                    audio: true 
                });
                
                // 設置視頻預覽
                const videoPreview = document.getElementById('video-preview');
                videoPreview.srcObject = stream;
                
                // 設置事件監聽器
                setupEventListeners();
                
                showStatusMessage('相機已就緒');
            } catch (error) {
                console.error('無法訪問相機:', error);
                showStatusMessage('無法訪問相機，請檢查權限設定', 5000);
            }
        }
        
        // 設置事件監聽器
        function setupEventListeners() {
            // 模式切換按鈕
            document.getElementById('photo-mode').addEventListener('click', () => switchMode('photo'));
            document.getElementById('video-mode').addEventListener('click', () => switchMode('video'));
            
            // 拍攝按鈕
            document.getElementById('capture-btn').addEventListener('click', handleCapture);
            
            // 儲存對話框按鈕
            document.getElementById('save-cancel').addEventListener('click', closeSaveDialog);
            document.getElementById('save-confirm').addEventListener('click', confirmSave);
        }
        
        // 切換模式
        function switchMode(mode) {
            currentMode = mode;
            
            // 更新按鈕狀態
            document.getElementById('photo-mode').classList.toggle('active', mode === 'photo');
            document.getElementById('video-mode').classList.toggle('active', mode === 'video');
            
            // 更新解析度顯示
            const resolutionInfo = document.getElementById('resolution-info');
            resolutionInfo.textContent = mode === 'photo' ? '高解析度照片' : '4K 60fps';
            
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
                mediaRecorder = new MediaRecorder(stream, {
                    mimeType: 'video/mp4; codecs="avc1.42E01E"'
                });
                
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
