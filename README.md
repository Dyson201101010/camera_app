<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="theme-color" content="#000000">
    <title>EPhoto Pro</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- React & ReactDOM -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    
    <!-- Babel for JSX -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;800&family=Space+Mono:wght@400;700&display=swap');

        /* Custom Scrollbar hide */
        .no-scrollbar::-webkit-scrollbar {
            display: none;
        }
        .no-scrollbar {
            -ms-overflow-style: none;
            scrollbar-width: none;
        }

        body {
            background-color: #000;
            color: white;
            font-family: 'Inter', sans-serif;
            overflow: hidden; /* Prevent scrolling */
            touch-action: none; /* Prevent bounce */
        }

        /* iOS 26 Liquid Glass Effect */
        .liquid-glass {
            background: rgba(20, 20, 20, 0.4);
            backdrop-filter: blur(20px) saturate(180%);
            -webkit-backdrop-filter: blur(20px) saturate(180%);
            border: 1px solid rgba(255, 255, 255, 0.08);
            box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.37);
        }

        .liquid-glass-accent {
            background: rgba(34, 211, 238, 0.1); /* Cyan hint */
            backdrop-filter: blur(12px);
            -webkit-backdrop-filter: blur(12px);
            border: 1px solid rgba(34, 211, 238, 0.3);
        }

        /* Apple Log Simulation Filter */
        .filter-apple-log {
            filter: contrast(0.85) saturate(0.7) brightness(1.1) sepia(0.1);
        }

        /* Range Slider Styling */
        input[type=range] {
            -webkit-appearance: none;
            background: transparent;
        }
        input[type=range]::-webkit-slider-thumb {
            -webkit-appearance: none;
            height: 20px;
            width: 20px;
            border-radius: 50%;
            background: #22d3ee; /* Cyan-400 */
            cursor: pointer;
            margin-top: -8px;
            box-shadow: 0 0 10px rgba(34, 211, 238, 0.5);
        }
        input[type=range]::-webkit-slider-runnable-track {
            width: 100%;
            height: 4px;
            cursor: pointer;
            background: rgba(255,255,255,0.2);
            border-radius: 2px;
        }
        
        /* Shutter Animation */
        @keyframes flash {
            0% { opacity: 0; }
            10% { opacity: 1; }
            100% { opacity: 0; }
        }
        .shutter-flash {
            animation: flash 0.2s ease-out;
            background-color: black;
            position: absolute;
            top: 0; left: 0; right: 0; bottom: 0;
            pointer-events: none;
            z-index: 50;
        }

        /* Orientation Indicator */
        .level-indicator {
            transition: transform 0.2s ease;
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useRef, useMemo } = React;

        // --- Inline Icons to avoid CDN errors ---
        const IconBase = ({ children, size = 24, className = "", ...props }) => (
            <svg 
                xmlns="http://www.w3.org/2000/svg" 
                width={size} 
                height={size} 
                viewBox="0 0 24 24" 
                fill="none" 
                stroke="currentColor" 
                strokeWidth="2" 
                strokeLinecap="round" 
                strokeLinejoin="round" 
                className={className} 
                {...props}
            >
                {children}
            </svg>
        );

        const Zap = (props) => <IconBase {...props}><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"></polygon></IconBase>;
        const Sliders = (props) => <IconBase {...props}><line x1="4" y1="21" x2="4" y2="14"></line><line x1="4" y1="10" x2="4" y2="3"></line><line x1="12" y1="21" x2="12" y2="12"></line><line x1="12" y1="8" x2="12" y2="3"></line><line x1="20" y1="21" x2="20" y2="16"></line><line x1="20" y1="12" x2="20" y2="3"></line><line x1="1" y1="14" x2="7" y2="14"></line><line x1="9" y1="8" x2="15" y2="8"></line><line x1="17" y1="16" x2="23" y2="16"></line></IconBase>;
        const RotateCcw = (props) => <IconBase {...props}><path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 12" /><path d="M3 3v9h9" /></IconBase>;
        const Sun = (props) => <IconBase {...props}><circle cx="12" cy="12" r="4"></circle><path d="M12 2v2"></path><path d="M12 20v2"></path><path d="M4.93 4.93l1.41 1.41"></path><path d="M17.66 17.66l1.41 1.41"></path><path d="M2 12h2"></path><path d="M20 12h2"></path><path d="M6.34 17.66l-1.41-1.41"></path><path d="M19.07 4.93l-1.41 1.41"></path></IconBase>;
        const Grid = (props) => <IconBase {...props}><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><line x1="3" y1="9" x2="21" y2="9"></line><line x1="3" y1="15" x2="21" y2="15"></line><line x1="9" y1="3" x2="9" y2="21"></line><line x1="15" y1="3" x2="15" y2="21"></line></IconBase>;
        const ImageIcon = (props) => <IconBase {...props}><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><circle cx="8.5" cy="8.5" r="1.5"></circle><polyline points="21 15 16 10 5 21"></polyline></IconBase>;
        const Aperture = (props) => <IconBase {...props}><circle cx="12" cy="12" r="10"></circle><line x1="14.31" y1="8" x2="20.05" y2="17.94"></line><line x1="9.69" y1="8" x2="21.17" y2="8"></line><line x1="7.38" y1="12" x2="13.12" y2="2.06"></line><line x1="9.69" y1="16" x2="3.95" y2="6.06"></line><line x1="14.31" y1="16" x2="2.83" y2="16"></line><line x1="16.62" y1="12" x2="10.88" y2="21.94"></line></IconBase>;

        const App = () => {
            // --- State Management ---
            const videoRef = useRef(null);
            const [stream, setStream] = useState(null);
            const [hasPermission, setHasPermission] = useState(false);
            const [deviceInfo, setDeviceInfo] = useState({ model: 'Unknown', isPro: false, isMax: false });
            
            // Camera Features
            const [cameraMode, setCameraMode] = useState('PHOTO'); // PHOTO, VIDEO
            const [format, setFormat] = useState('HEIF'); // HEIF, RAW, LOG
            const [zoom, setZoom] = useState(1);
            const [lens, setLens] = useState('1x'); // 0.5x, 1x, 2x, 5x
            const [gridVisible, setGridVisible] = useState(true);
            const [flashMode, setFlashMode] = useState('off');
            
            // Manual Controls
            const [exposure, setExposure] = useState(0); // -2 to 2
            const [iso, setIso] = useState(400); // Simulated
            const [showControls, setShowControls] = useState(false);

            // UX
            const [isShutterPressed, setIsShutterPressed] = useState(false);
            const [galleryPreview, setGalleryPreview] = useState(null);
            const [orientation, setOrientation] = useState(0);

            // --- Device Detection Logic ---
            useEffect(() => {
                const detectDevice = () => {
                    const width = window.screen.width;
                    const height = window.screen.height;
                    const ratio = window.devicePixelRatio;
                    
                    // Simple heuristic for iPhone models based on logical resolution
                    let model = 'iPhone';
                    let isPro = false;
                    let isMax = false;

                    // iPhone 12/13/14/15/16 Pro Max or Plus
                    if (width >= 430) {
                        model = 'iPhone Pro Max / Plus';
                        isMax = true;
                        isPro = true; // Assuming newer large phones are powerful
                    } 
                    // iPhone 12/13/14/15/16 Pro / Base
                    else if (width >= 390) {
                        model = 'iPhone Pro';
                        isPro = true;
                    } 
                    // iPhone X/XS/11 Pro
                    else if (width >= 375) {
                        model = 'iPhone X/XS';
                    }
                    
                    // User Agent Check for iOS version (Rough check)
                    const ua = navigator.userAgent;
                    const isIOS = /iPhone|iPad|iPod/.test(ua);
                    
                    setDeviceInfo({ model, isPro, isMax, isIOS });
                };

                detectDevice();
                
                // Orientation listener
                const handleOrientation = (event) => {
                    if(event.beta) {
                         // Simple tilt logic just for UI feedback
                    }
                };
                window.addEventListener('deviceorientation', handleOrientation);
                return () => window.removeEventListener('deviceorientation', handleOrientation);
            }, []);

            // --- Camera Initialization ---
            useEffect(() => {
                startCamera();
                return () => {
                    if (stream) {
                        stream.getTracks().forEach(track => track.stop());
                    }
                };
            }, [lens]); // Restart camera if lens "physically" changes (simulated)

            const startCamera = async () => {
                try {
                    // Logic to request different cameras based on "lens" state
                    const constraints = {
                        video: {
                            facingMode: 'environment',
                            width: { ideal: 4096 }, // Try to get 4K
                            height: { ideal: 2160 },
                        },
                        audio: false
                    };

                    const newStream = await navigator.mediaDevices.getUserMedia(constraints);
                    setStream(newStream);
                    setHasPermission(true);
                    if (videoRef.current) {
                        videoRef.current.srcObject = newStream;
                    }
                } catch (err) {
                    console.error("Camera Error:", err);
                    // alert("Unable to access camera. Please allow permission in Safari settings.");
                    // Fallback for demo if camera fails (e.g. desktop without camera)
                }
            };

            // --- Capture Logic ---
            const handleShutter = () => {
                setIsShutterPressed(true);
                
                // Simulate Capture delay
                setTimeout(() => {
                    setIsShutterPressed(false);
                    captureImage();
                }, 150);
            };

            const captureImage = () => {
                if (!videoRef.current) return;
                
                const canvas = document.createElement('canvas');
                const video = videoRef.current;
                canvas.width = video.videoWidth;
                canvas.height = video.videoHeight;
                
                const ctx = canvas.getContext('2d');
                
                // Apply "Apple Log" filter if active
                if (format === 'LOG') {
                    ctx.filter = 'contrast(0.85) saturate(0.7) brightness(1.1) sepia(0.1)';
                }
                // Apply "Exposure" simulation
                if (exposure !== 0) {
                     const bright = 100 + (exposure * 10);
                     ctx.filter = (ctx.filter !== 'none' ? ctx.filter + ' ' : '') + `brightness(${bright}%)`;
                }

                ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
                
                // Save to state
                const dataUrl = canvas.toDataURL('image/jpeg', 0.9);
                setGalleryPreview(dataUrl);
            };

            // --- Lens Switching Logic (Simulation) ---
            const switchLens = (newLens) => {
                setLens(newLens);
                // In a real Native app, this calls hardware. 
                // In Web, we simulate zoom via CSS scale transform or track constraints
                if (newLens === '0.5x') setZoom(0.5);
                if (newLens === '1x') setZoom(1);
                if (newLens === '2x') setZoom(2);
                if (newLens === '5x') setZoom(5);
            };

            // --- Render Helper: Active Style ---
            const activeStyle = (isActive) => 
                isActive ? "text-cyan-400 drop-shadow-[0_0_8px_rgba(34,211,238,0.8)]" : "text-white/60";

            if (!hasPermission) {
                return (
                    <div className="h-screen w-full bg-black flex flex-col items-center justify-center text-center p-6 space-y-6">
                        <div className="w-20 h-20 rounded-full border-2 border-cyan-400 flex items-center justify-center animate-pulse">
                            <Aperture className="w-10 h-10 text-cyan-400" />
                        </div>
                        <h1 className="text-2xl font-bold font-mono tracking-tighter">EPhoto</h1>
                        <p className="text-gray-400 text-sm max-w-xs">
                            EPhoto 需要相機權限以啟動 Liquid Glass 引擎。
                            <br/>專為 iPhone XS 及後續機型設計。
                        </p>
                        <button 
                            onClick={startCamera}
                            className="px-8 py-3 bg-cyan-900/30 border border-cyan-400/50 rounded-full text-cyan-400 font-bold hover:bg-cyan-400/20 transition-all"
                        >
                            啟動相機
                        </button>
                    </div>
                );
            }

            return (
                <div className="relative h-[100dvh] w-full bg-black overflow-hidden select-none">
                    
                    {/* Shutter Flash Animation */}
                    {isShutterPressed && <div className="shutter-flash"></div>}

                    {/* --- Viewfinder Layer --- */}
                    <div className="absolute inset-0 z-0 flex items-center justify-center bg-black">
                        <video 
                            ref={videoRef}
                            autoPlay 
                            playsInline 
                            muted
                            className={`h-full w-full object-cover transition-all duration-500 ease-out 
                                ${format === 'LOG' ? 'filter-apple-log' : ''}
                            `}
                            style={{ 
                                transform: `scale(${Math.max(1, zoom)})`, // Digital zoom simulation for Web
                                filter: `brightness(${1 + exposure/5})` // Exposure simulation
                            }}
                        />
                        
                        {/* Grid Lines */}
                        {gridVisible && (
                            <div className="absolute inset-0 pointer-events-none opacity-30">
                                <div className="w-full h-full border-white/20 grid grid-cols-3 grid-rows-3">
                                    <div className="border-r border-b border-white/20"></div>
                                    <div className="border-r border-b border-white/20"></div>
                                    <div className="border-b border-white/20"></div>
                                    <div className="border-r border-b border-white/20"></div>
                                    <div className="border-r border-b border-white/20"></div>
                                    <div className="border-b border-white/20"></div>
                                    <div className="border-r border-white/20"></div>
                                    <div className="border-r border-white/20"></div>
                                    <div></div>
                                </div>
                            </div>
                        )}
                        
                        {/* Center Point */}
                        <div className="absolute top-1/2 left-1/2 w-2 h-2 bg-yellow-400/50 rounded-full -translate-x-1/2 -translate-y-1/2 pointer-events-none border border-black/20"></div>
                    </div>

                    {/* --- UI Layer: Top Bar --- */}
                    <div className="absolute top-0 left-0 right-0 pt-safe-top z-20">
                        <div className="liquid-glass mx-4 mt-2 rounded-3xl p-3 flex justify-between items-center h-16 shadow-lg border-b border-white/5">
                            {/* Flash */}
                            <button onClick={() => setFlashMode(f => f === 'on' ? 'off' : 'on')} className="p-2 active:scale-95 transition-transform">
                                <Zap className={`w-5 h-5 ${activeStyle(flashMode === 'on')}`} />
                            </button>

                            {/* Format Toggles (Dynamic based on iPhone Model) */}
                            <div className="flex gap-2 text-[10px] font-mono font-bold">
                                {deviceInfo.isPro && (
                                    <>
                                        <button 
                                            onClick={() => setFormat(f => f === 'RAW' ? 'HEIF' : 'RAW')}
                                            className={`px-2 py-1 rounded border transition-colors ${format === 'RAW' ? 'bg-cyan-500/20 border-cyan-400 text-cyan-400' : 'border-white/20 text-gray-400'}`}
                                        >
                                            RAW
                                        </button>
                                        <button 
                                            onClick={() => setFormat(f => f === 'LOG' ? 'HEIF' : 'LOG')}
                                            className={`px-2 py-1 rounded border transition-colors ${format === 'LOG' ? 'bg-amber-500/20 border-amber-400 text-amber-400' : 'border-white/20 text-gray-400'}`}
                                        >
                                            LOG
                                        </button>
                                    </>
                                )}
                                {!deviceInfo.isPro && (
                                    <span className="px-2 py-1 border border-white/10 rounded text-gray-500">HEIF</span>
                                )}
                            </div>

                            {/* Settings Toggle */}
                            <button onClick={() => setShowControls(!showControls)} className="p-2 active:scale-95 transition-transform">
                                <Sliders className={`w-5 h-5 ${activeStyle(showControls)}`} />
                            </button>
                        </div>

                        {/* Dropdown Manual Controls */}
                        {showControls && (
                            <div className="mx-4 mt-2 p-4 liquid-glass rounded-2xl animate-in slide-in-from-top-4 fade-in duration-300 space-y-4">
                                {/* Exposure */}
                                <div className="space-y-1">
                                    <div className="flex justify-between text-xs text-gray-400 font-mono">
                                        <span className="flex items-center gap-1"><Sun size={12}/> 曝光補償</span>
                                        <span>{exposure > 0 ? '+' : ''}{exposure} EV</span>
                                    </div>
                                    <input 
                                        type="range" min="-2" max="2" step="0.1" 
                                        value={exposure} 
                                        onChange={(e) => setExposure(parseFloat(e.target.value))}
                                        className="w-full"
                                    />
                                </div>
                                {/* Grid Toggle */}
                                <div className="flex items-center justify-between">
                                    <span className="text-xs text-gray-400 font-mono flex items-center gap-1"><Grid size={12}/> 構圖網格</span>
                                    <button 
                                        onClick={() => setGridVisible(!gridVisible)}
                                        className={`w-10 h-5 rounded-full relative transition-colors ${gridVisible ? 'bg-cyan-500' : 'bg-gray-700'}`}
                                    >
                                        <div className={`absolute top-1 w-3 h-3 bg-white rounded-full transition-all ${gridVisible ? 'left-6' : 'left-1'}`}></div>
                                    </button>
                                </div>
                                {/* Device Info */}
                                <div className="pt-2 border-t border-white/10 text-[10px] text-gray-600 font-mono flex justify-between">
                                    <span>DETECTED: {deviceInfo.model}</span>
                                    <span>iOS 26 GLASS ENGINE</span>
                                </div>
                            </div>
                        )}
                    </div>

                    {/* --- UI Layer: Bottom Controls --- */}
                    <div className="absolute bottom-0 left-0 right-0 pb-safe-bottom z-20">
                        <div className="bg-gradient-to-t from-black via-black/90 to-transparent pt-12 pb-8 px-6">
                            
                            {/* Lens Selector */}
                            <div className="flex justify-center items-center gap-6 mb-8">
                                <button onClick={() => switchLens('0.5x')} className={`w-10 h-10 rounded-full flex items-center justify-center text-xs font-bold transition-all border ${lens === '0.5x' ? 'bg-black/50 border-cyan-400 text-cyan-400 scale-110 shadow-[0_0_15px_rgba(34,211,238,0.3)]' : 'bg-black/30 border-transparent text-gray-400'}`}>.5</button>
                                <button onClick={() => switchLens('1x')} className={`w-10 h-10 rounded-full flex items-center justify-center text-xs font-bold transition-all border ${lens === '1x' ? 'bg-black/50 border-cyan-400 text-cyan-400 scale-110 shadow-[0_0_15px_rgba(34,211,238,0.3)]' : 'bg-black/30 border-transparent text-gray-400'}`}>1x</button>
                                <button onClick={() => switchLens('2x')} className={`w-10 h-10 rounded-full flex items-center justify-center text-xs font-bold transition-all border ${lens === '2x' ? 'bg-black/50 border-cyan-400 text-cyan-400 scale-110 shadow-[0_0_15px_rgba(34,211,238,0.3)]' : 'bg-black/30 border-transparent text-gray-400'}`}>2</button>
                                {(deviceInfo.isPro || deviceInfo.isMax) && (
                                    <button onClick={() => switchLens('5x')} className={`w-10 h-10 rounded-full flex items-center justify-center text-xs font-bold transition-all border ${lens === '5x' ? 'bg-black/50 border-cyan-400 text-cyan-400 scale-110 shadow-[0_0_15px_rgba(34,211,238,0.3)]' : 'bg-black/30 border-transparent text-gray-400'}`}>5</button>
                                )}
                            </div>

                            {/* Mode Selector */}
                            <div className="flex justify-center gap-6 mb-6 text-sm font-bold tracking-widest uppercase overflow-x-auto no-scrollbar mask-gradient">
                                <span className={`cursor-pointer transition-colors ${cameraMode === 'VIDEO' ? 'text-cyan-400' : 'text-gray-600'}`} onClick={() => setCameraMode('VIDEO')}>Video</span>
                                <span className={`cursor-pointer transition-colors ${cameraMode === 'PHOTO' ? 'text-cyan-400 drop-shadow-[0_0_5px_rgba(34,211,238,0.8)]' : 'text-gray-600'}`} onClick={() => setCameraMode('PHOTO')}>Photo</span>
                                <span className="text-gray-800 cursor-not-allowed">Portrait</span>
                                <span className="text-gray-800 cursor-not-allowed">Pano</span>
                            </div>

                            {/* Main Footer Row */}
                            <div className="flex items-center justify-between">
                                {/* Gallery Preview */}
                                <div className="w-14 h-14 rounded-lg overflow-hidden border border-white/20 bg-gray-900 relative">
                                    {galleryPreview ? (
                                        <img src={galleryPreview} className="w-full h-full object-cover" alt="Last capture" />
                                    ) : (
                                        <div className="w-full h-full flex items-center justify-center text-gray-700">
                                            <ImageIcon size={20} />
                                        </div>
                                    )}
                                </div>

                                {/* Shutter Button */}
                                <button 
                                    onClick={handleShutter}
                                    className={`
                                        w-20 h-20 rounded-full border-4 flex items-center justify-center transition-all duration-100
                                        ${cameraMode === 'VIDEO' ? 'border-red-500' : 'border-white'}
                                        ${isShutterPressed ? 'scale-90 opacity-80' : 'scale-100'}
                                    `}
                                >
                                    <div className={`
                                        rounded-full transition-all duration-300
                                        ${cameraMode === 'VIDEO' ? 'bg-red-500 w-16 h-16 hover:bg-red-600' : 'bg-white w-18 h-18 hover:bg-gray-200'}
                                    `}></div>
                                </button>

                                {/* Switch Camera */}
                                <div className="w-14 h-14 flex items-center justify-center rounded-full bg-white/10 backdrop-blur-md active:bg-white/20 transition-all">
                                    <RotateCcw className="text-white w-6 h-6" />
                                
                                </div>
                            </div>
                        </div>
                    </div>
                    
                    {/* iOS Safe Area Spacers (Visual logic only as CSS env() handles this) */}
                    <style>{`
                        .pb-safe-bottom { padding-bottom: env(safe-area-inset-bottom, 20px); }
                        .pt-safe-top { padding-top: env(safe-area-inset-top, 40px); }
                    `}</style>
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
