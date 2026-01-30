<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Metron AI - Intelligent Content Moderation</title>
    
    <!-- 1. Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- 2. React & ReactDOM (UMD - Local Execution Compatible) -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    
    <!-- 3. Babel (Compiles JSX in browser) -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

    <!-- Global Styles -->
    <style>
        body {
            /* Changed from pitch black to a deep slate for better contrast */
            background-color: #020617; 
            background-image: 
                radial-gradient(circle at 15% 50%, rgba(79, 70, 229, 0.15), transparent 25%),
                radial-gradient(circle at 85% 30%, rgba(6, 182, 212, 0.15), transparent 25%);
            color: white;
            font-family: 'Inter', system-ui, -apple-system, sans-serif;
            overflow-x: hidden;
        }
        
        /* Custom Scrollbar */
        ::-webkit-scrollbar { width: 8px; }
        ::-webkit-scrollbar-track { background: #020617; }
        ::-webkit-scrollbar-thumb { background: #334155; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #475569; }

        /* Animations */
        @keyframes fade-in-up {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .animate-fade-in-up { animation: fade-in-up 0.8s ease-out forwards; }

        @keyframes pulse {
            0%, 100% { opacity: 1; transform: scaleY(1); }
            50% { opacity: 0.5; transform: scaleY(0.7); }
        }
        
        @keyframes spin-slow {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        .animate-spin-slow { animation: spin-slow 3s linear infinite; }

        /* Glass Panel */
        .glass-panel {
            background: rgba(30, 41, 59, 0.4);
            backdrop-filter: blur(12px);
            -webkit-backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
        }
        
        /* Gradient Text */
        .text-gradient {
            background: linear-gradient(to right, #22d3ee, #818cf8);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useRef } = React;
        const { createRoot } = ReactDOM;

        // --- ICON DEFINITIONS ---
        const createIcon = (path) => ({ className }) => (
            <svg 
                xmlns="http://www.w3.org/2000/svg" 
                width="24" height="24" viewBox="0 0 24 24" 
                fill="none" stroke="currentColor" strokeWidth="2" 
                strokeLinecap="round" strokeLinejoin="round" 
                className={className}
            >
                {path}
            </svg>
        );

        const Shield = createIcon(<path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z" />);
        const AlertTriangle = createIcon(<path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0zM12 9v4m0 4h.01" />);
        const CheckCircle = createIcon(<path d="M22 11.08V12a10 10 0 1 1-5.93-9.14M22 4L12 14.01l-3-3" />);
        const Activity = createIcon(<polyline points="22 12 18 12 15 21 9 3 6 12 2 12" />);
        const Lock = createIcon(<><rect x="3" y="11" width="18" height="11" rx="2" ry="2" /><path d="M7 11V7a5 5 0 0 1 10 0v4" /></>);
        const Upload = createIcon(<><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" /><polyline points="17 8 12 3 7 8" /><line x1="12" y1="3" x2="12" y2="15" /></>);
        const FileText = createIcon(<><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z" /><polyline points="14 2 14 8 20 8" /><line x1="16" y1="13" x2="8" y2="13" /><line x1="16" y1="17" x2="8" y2="17" /><polyline points="10 9 9 9 8 9" /></>);
        const Code = createIcon(<><polyline points="16 18 22 12 16 6" /><polyline points="8 6 2 12 8 18" /></>);
        const Cpu = createIcon(<><rect x="4" y="4" width="16" height="16" rx="2" ry="2" /><rect x="9" y="9" width="6" height="6" /><line x1="9" y1="1" x2="9" y2="4" /><line x1="15" y1="1" x2="15" y2="4" /><line x1="9" y1="20" x2="9" y2="23" /><line x1="15" y1="20" x2="15" y2="23" /><line x1="20" y1="9" x2="23" y2="9" /><line x1="20" y1="15" x2="23" y2="15" /><line x1="1" y1="9" x2="4" y2="9" /><line x1="1" y1="15" x2="4" y2="15" /></>);
        const Zap = createIcon(<polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2" />);
        const Eye = createIcon(<><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z" /><circle cx="12" cy="12" r="3" /></>);
        const Mic = createIcon(<><path d="M12 1a3 3 0 0 0-3 3v8a3 3 0 0 0 6 0V4a3 3 0 0 0-3-3z" /><path d="M19 10v2a7 7 0 0 1-14 0v-2" /><line x1="12" y1="19" x2="12" y2="23" /><line x1="8" y1="23" x2="16" y2="23" /></>);
        const RefreshCw = createIcon(<><polyline points="23 4 23 10 17 10" /><polyline points="1 20 1 14 7 14" /><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15" /></>);
        const X = createIcon(<><line x1="18" y1="6" x2="6" y2="18" /><line x1="6" y1="6" x2="18" y2="18" /></>);
        const Database = createIcon(<><ellipse cx="12" cy="5" rx="9" ry="3" /><path d="M21 12c0 1.66-4 3-9 3s-9-1.34-9-3" /><path d="M3 5v14c0 1.66 4 3 9 3s9-1.34 9-3V5" /></>);
        const Sliders = createIcon(<><line x1="4" y1="21" x2="4" y2="14" /><line x1="4" y1="10" x2="4" y2="3" /><line x1="12" y1="21" x2="12" y2="12" /><line x1="12" y1="8" x2="12" y2="3" /><line x1="20" y1="21" x2="20" y2="16" /><line x1="20" y1="12" x2="20" y2="3" /><line x1="1" y1="14" x2="7" y2="14" /><line x1="9" y1="8" x2="15" y2="8" /><line x1="17" y1="16" x2="23" y2="16" /></>);
        const Send = createIcon(<><line x1="22" y1="2" x2="11" y2="13" /><polygon points="22 2 15 22 11 13 2 9 22 2" /></>);
        const MapPin = createIcon(<><path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z" /><circle cx="12" cy="10" r="3" /></>);
        const LifeBuoy = createIcon(<><circle cx="12" cy="12" r="10" /><circle cx="12" cy="12" r="4" /><line x1="4.93" y1="4.93" x2="9.17" y2="9.17" /><line x1="14.83" y1="14.83" x2="19.07" y2="19.07" /><line x1="14.83" y1="9.17" x2="19.07" y2="4.93" /><line x1="14.83" y1="9.17" x2="18.36" y2="5.64" /><line x1="4.93" y1="19.07" x2="9.17" y2="14.83" /></>);
        const ArrowRight = createIcon(<><line x1="5" y1="12" x2="19" y2="12" /><polyline points="12 5 19 12 12 19" /></>);
        const Smartphone = createIcon(<><rect x="5" y="2" width="14" height="20" rx="2" ry="2" /><line x1="12" y1="18" x2="12.01" y2="18" /></>);
        const Check = createIcon(<polyline points="20 6 9 17 4 12" />);
        const AlertOctagon = createIcon(<polygon points="7.86 2 16.14 2 22 7.86 22 16.14 16.14 22 7.86 22 2 16.14 2 7.86 7.86 2" />);
        const Globe = createIcon(<><circle cx="12" cy="12" r="10" /><line x1="2" y1="12" x2="22" y2="12" /><path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z" /></>);
        const MessageSquare = createIcon(<path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z" />);

        // --- COMPONENTS ---

        const Waveform = ({ active }) => (
            <div className="flex items-center justify-center gap-1 h-16 w-full">
                {[...Array(45)].map((_, i) => (
                    <div
                        key={i}
                        className={`w-1 rounded-full transition-all duration-300 ${
                            active ? (i > 15 && i < 25 ? "bg-red-500 h-12" : "bg-cyan-500 h-4") : "bg-slate-700 h-1"
                        }`}
                        style={{
                            height: active ? `${Math.random() * 40 + 10}%` : '4px',
                            opacity: active ? 1 : 0.3,
                            animation: active ? `pulse 0.4s infinite ${i * 0.05}s` : 'none'
                        }}
                    />
                ))}
            </div>
        );

        const SafetyMeter = ({ score, size = "large" }) => {
            let color = "text-red-500";
            let status = "HIGH RISK";
            let bg = "border-red-500/50 bg-red-950/30";

            if (score >= 50) {
                color = "text-emerald-400";
                status = "SAFE";
                bg = "border-emerald-500/50 bg-emerald-950/30";
            } else if (score >= 20) {
                color = "text-amber-400";
                status = "MODERATE";
                bg = "border-amber-500/50 bg-amber-950/30";
            }

            const dimensions = size === "large" ? "w-48 h-48" : "w-32 h-32";
            const textSize = size === "large" ? "text-5xl" : "text-3xl";
            const stroke = size === "large" ? "6" : "4";

            return (
                <div className={`relative ${dimensions} rounded-full border-4 flex flex-col items-center justify-center transition-all duration-500 ${bg}`}>
                    <span className={`${textSize} font-bold ${color}`}>{score}</span>
                    <span className={`text-[10px] tracking-widest mt-1 font-bold ${color}`}>{status}</span>
                    <svg className="absolute inset-0 w-full h-full -rotate-90 pointer-events-none">
                        <circle cx="50%" cy="50%" r="45%" fill="none" stroke="#1e293b" strokeWidth={stroke} />
                        <circle
                            cx="50%"
                            cy="50%"
                            r="45%"
                            fill="none"
                            stroke="currentColor"
                            strokeWidth={stroke}
                            strokeDasharray="283"
                            strokeDashoffset={283 - (283 * score) / 100}
                            className={`${color} transition-all duration-500`}
                            strokeLinecap="round"
                        />
                    </svg>
                </div>
            );
        };

        const AccurateHeatmap = () => {
            // Simulated heatmap data with timestamps
            const segments = [
                { id: 1, type: 'safe', label: 'Intro', start: '0:00', end: '0:35', width: '25%', color: 'bg-emerald-500' },
                { id: 2, type: 'safe', label: 'Gameplay', start: '0:35', end: '1:12', width: '30%', color: 'bg-emerald-500' },
                { id: 3, type: 'danger', label: 'Violation', start: '1:12', end: '1:24', width: '15%', color: 'bg-red-500 animate-pulse' },
                { id: 4, type: 'warning', label: 'Slang', start: '1:24', end: '1:45', width: '15%', color: 'bg-amber-500' },
                { id: 5, type: 'safe', label: 'Outro', start: '1:45', end: '2:10', width: '15%', color: 'bg-emerald-500' },
            ];

            return (
                <div className="w-full">
                    {/* Visual Bar */}
                    <div className="h-14 w-full flex rounded-lg overflow-hidden mb-2 border border-white/10 shadow-lg relative">
                        {segments.map((seg) => (
                            <div 
                                key={seg.id} 
                                className={`h-full ${seg.color} flex flex-col items-center justify-center relative hover:opacity-90 transition-opacity cursor-pointer group border-r border-black/20`}
                                style={{ width: seg.width }}
                            >
                                <span className="text-[9px] font-bold uppercase tracking-wider text-black/80 drop-shadow-sm">{seg.label}</span>
                                
                                {/* Hover Tooltip */}
                                <div className="absolute -top-10 left-1/2 -translate-x-1/2 bg-black text-white text-[10px] py-1 px-2 rounded opacity-0 group-hover:opacity-100 transition-opacity whitespace-nowrap border border-white/20 pointer-events-none z-10">
                                    {seg.start} - {seg.end}
                                </div>
                            </div>
                        ))}
                        
                        {/* Playhead Indicator (Static for demo) */}
                        <div className="absolute top-0 bottom-0 left-[60%] w-0.5 bg-white shadow-[0_0_10px_rgba(255,255,255,0.8)] z-20">
                            <div className="absolute -top-1 -left-1.5 w-3 h-3 bg-white rounded-full"></div>
                        </div>
                    </div>

                    {/* Accurate Time Scale */}
                    <div className="flex justify-between text-[9px] text-slate-500 font-mono px-1">
                        <span>0:00</span>
                        <span>0:30</span>
                        <span>1:00</span>
                        <span>1:30</span>
                        <span>2:00</span>
                        <span>2:10</span>
                    </div>
                </div>
            );
        };

        function App() {
            const [activePage, setActivePage] = useState('home'); 
            const [scrollY, setScrollY] = useState(0);

            // Simulation State
            const [isProcessing, setIsProcessing] = useState(false);
            const [processProgress, setProcessProgress] = useState(0);
            const [processingStage, setProcessingStage] = useState("Initializing...");
            const [uploadedFile, setUploadedFile] = useState(null);
            const [analysisResult, setAnalysisResult] = useState(null);
            const [demoSequenceIndex, setDemoSequenceIndex] = useState(0); 
            const fileInputRef = useRef(null);

            useEffect(() => {
                const handleScroll = () => setScrollY(window.scrollY);
                window.addEventListener('scroll', handleScroll);
                return () => window.removeEventListener('scroll', handleScroll);
            }, []);

            // --- ANALYSIS LOGIC ---
            const handleFileUpload = (e) => {
                const file = e.target.files[0];
                if (file) {
                    setUploadedFile(file);
                    setAnalysisResult(null);
                    startAnalysis();
                }
            };

            const startAnalysis = () => {
                setIsProcessing(true);
                setProcessProgress(0);
                setProcessingStage("Initializing upload stream...");

                let progress = 0;
                const interval = setInterval(() => {
                    progress += 1;
                    setProcessProgress(progress);
                    
                    if (progress < 20) setProcessingStage("Extracting audio waveforms...");
                    else if (progress < 50) setProcessingStage("Scanning visual frames for violence...");
                    else if (progress < 80) setProcessingStage("Cross-referencing regional slang DB...");
                    else setProcessingStage("Finalizing S-Score calculation...");

                    if (progress >= 100) {
                        clearInterval(interval);
                        finishAnalysis();
                    }
                }, 30);
            };

            const finishAnalysis = () => {
                setIsProcessing(false);
                
                // DEMO LOOP LOGIC
                const step = demoSequenceIndex % 3;
                let score;
                let verdict = "";
                let action = "";
                let colorClass = "";
                let message = "";

                if (step === 0) {
                    // SAFE (50-100)
                    score = 85 + Math.floor(Math.random() * 15);
                    verdict = "UPLOAD APPROVED";
                    action = "SAFE";
                    colorClass = "text-emerald-400";
                    message = "Content meets all community guidelines. Safe to publish.";
                } else if (step === 1) {
                    // MODERATE (20-49) - Updated Message
                    score = 30 + Math.floor(Math.random() * 15);
                    verdict = "CAN'T UPLOAD";
                    action = "UNSAFE";
                    colorClass = "text-amber-400";
                    message = "Content contains explicit language or graphical references. Blurring or muting segments recommended.";
                } else {
                    // STRIKE (0-19) - Updated Message
                    score = Math.floor(Math.random() * 15);
                    verdict = "STRIKE TRIGGERED";
                    action = "HIGH RISK";
                    colorClass = "text-red-500";
                    message = "SEVERE RISK: Graphic violence and explicit content detected. Upload rejected.";
                }

                setDemoSequenceIndex(prev => prev + 1);

                setAnalysisResult({
                    score,
                    verdict,
                    action,
                    colorClass,
                    message
                });
            };

            const navigateTo = (page) => {
                setActivePage(page);
                window.scrollTo(0, 0);
            };

            // --- PAGES ---

            const HomePage = () => (
                <React.Fragment>
                    {/* HERO */}
                    <section className="relative pt-32 pb-20 px-6 min-h-[70vh] flex items-center justify-center overflow-hidden">
                        {/* Background blobs for color */}
                        <div className="absolute top-20 left-1/4 w-96 h-96 bg-purple-600/20 rounded-full blur-[100px] pointer-events-none" />
                        <div className="absolute bottom-20 right-1/4 w-96 h-96 bg-cyan-500/10 rounded-full blur-[100px] pointer-events-none" />

                        <div className="relative z-10 text-center max-w-4xl mx-auto">
                            <div className="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-slate-800/50 border border-slate-700 text-cyan-400 text-xs font-bold mb-6 tracking-wide animate-fade-in-up">
                                <span className="w-2 h-2 rounded-full bg-cyan-400 animate-pulse"></span>
                                SYSTEM ONLINE V2.4
                            </div>
                            <h1 className="text-5xl md:text-7xl font-bold leading-tight mb-6">
                                Intelligent Content <br />
                                <span className="text-gradient">Moderation Infrastructure</span>
                            </h1>
                            <p className="text-lg text-slate-400 mb-10 max-w-2xl mx-auto leading-relaxed">
                                Metron analyzes audio, video, and text in real-time to detect policy violations before they strike. 
                                The standard for safe digital ecosystems.
                            </p>
                            <div className="flex flex-col sm:flex-row gap-4 justify-center">
                                <button 
                                    onClick={() => navigateTo('metron-ai')}
                                    className="px-8 py-4 bg-white text-black font-bold rounded-lg hover:bg-slate-200 transition-all active:scale-95 flex items-center justify-center gap-2 shadow-[0_0_20px_rgba(255,255,255,0.3)]"
                                >
                                    <Zap className="w-4 h-4" /> Try Metron AI
                                </button>
                                <button className="px-8 py-4 border border-slate-700 hover:border-cyan-400/50 hover:bg-slate-800/50 rounded-lg transition-all text-slate-300">
                                    View Documentation
                                </button>
                            </div>
                        </div>
                    </section>

                    {/* PIPELINE */}
                    <section id="pipeline" className="py-20 px-6 border-y border-white/5 bg-[#020617]">
                        <div className="max-w-6xl mx-auto">
                            <div className="glass-panel rounded-3xl p-10 relative overflow-hidden">
                                <div className="text-center mb-12">
                                    <h2 className="text-2xl font-bold mb-2">Processing Pipeline</h2>
                                    <p className="text-slate-500 text-sm">End-to-end safety architecture</p>
                                </div>
                                <div className="flex flex-col md:flex-row justify-center items-center gap-8 md:gap-16 mb-16 relative z-10">
                                    <div className="flex flex-col items-center gap-4 group">
                                        <div className="w-20 h-20 rounded-2xl bg-cyan-900/20 border border-cyan-500/30 flex items-center justify-center group-hover:bg-cyan-900/40 transition-colors">
                                            <Database className="w-8 h-8 text-cyan-400" />
                                        </div>
                                        <div className="text-center">
                                            <h3 className="font-bold text-lg">Ingest</h3>
                                            <p className="text-xs text-slate-500">Multi-modal input</p>
                                        </div>
                                    </div>
                                    <ArrowRight className="text-slate-700 hidden md:block w-6 h-6" />
                                    <div className="flex flex-col items-center gap-4 group">
                                        <div className="w-20 h-20 rounded-2xl bg-purple-900/20 border border-purple-500/30 flex items-center justify-center group-hover:bg-purple-900/40 transition-colors">
                                            <Cpu className="w-8 h-8 text-purple-400" />
                                        </div>
                                        <div className="text-center">
                                            <h3 className="font-bold text-lg">Analyze</h3>
                                            <p className="text-xs text-slate-500">AI processing</p>
                                        </div>
                                    </div>
                                    <ArrowRight className="text-slate-700 hidden md:block w-6 h-6" />
                                    <div className="flex flex-col items-center gap-4 group">
                                        <div className="w-20 h-20 rounded-2xl bg-emerald-900/20 border border-emerald-500/30 flex items-center justify-center group-hover:bg-emerald-900/40 transition-colors">
                                            <Shield className="w-8 h-8 text-emerald-400" />
                                        </div>
                                        <div className="text-center">
                                            <h3 className="font-bold text-lg">Decide</h3>
                                            <p className="text-xs text-slate-500">Policy enforcement</p>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </section>

                    {/* NEW SECTION: IMAGE & AUDIO THREAT DETECTION (Side by Side) */}
                    <section className="py-24 px-6">
                        <div className="max-w-7xl mx-auto grid lg:grid-cols-12 gap-8 items-stretch">
                            
                            {/* Left: Image Threat Visual (Span 5) */}
                            <div className="lg:col-span-5 relative group h-full min-h-[500px]">
                                <div className="absolute -inset-1 bg-gradient-to-r from-red-600 to-orange-600 rounded-2xl blur opacity-30 group-hover:opacity-50 transition duration-1000"></div>
                                <div className="relative bg-slate-900 rounded-2xl overflow-hidden border border-white/10 h-full flex flex-col">
                                    {/* Chaos/Fire Image with heavy blur */}
                                    <div className="absolute inset-0 bg-[url('https://images.unsplash.com/photo-1542281286-9e0a16bb7366?q=80&w=2000')] bg-cover bg-center filter blur-xl scale-110 opacity-70"></div>
                                    <div className="absolute inset-0 bg-red-950/60 mix-blend-overlay"></div>
                                    
                                    {/* Privacy Shield Overlay */}
                                    <div className="absolute inset-0 flex items-center justify-center bg-black/50 backdrop-blur-sm z-10 flex-col p-6 text-center">
                                        <div className="w-20 h-20 bg-red-500/20 rounded-full flex items-center justify-center mb-4 border border-red-500/50 animate-pulse shadow-[0_0_30px_rgba(239,68,68,0.4)]">
                                            <AlertTriangle className="w-10 h-10 text-red-500" />
                                        </div>
                                        <h3 className="text-2xl font-bold text-white mb-2 uppercase tracking-wider">Restricted</h3>
                                        <p className="text-red-400 font-mono text-xs uppercase tracking-widest bg-black/60 px-3 py-1 rounded">Graphic Violence Detected</p>
                                    </div>
                                    
                                    {/* Corner Label */}
                                    <div className="absolute top-4 left-4 bg-black/60 backdrop-blur-md px-3 py-1.5 rounded-lg border border-white/10 flex items-center gap-2 z-20">
                                        <Eye className="w-4 h-4 text-cyan-400" />
                                        <span className="text-xs font-bold text-slate-200">Visual Analysis</span>
                                    </div>
                                </div>
                            </div>

                            {/* Right: Audio & Text Threat Stack (Span 7) */}
                            <div className="lg:col-span-7 flex flex-col gap-6">
                                
                                {/* Audio Threat Info */}
                                <div className="glass-panel rounded-2xl p-8 relative overflow-hidden">
                                    <div className="flex items-center gap-3 mb-6">
                                        <div className="p-3 bg-purple-500/10 rounded-xl border border-purple-500/20">
                                            <Mic className="w-6 h-6 text-purple-500" />
                                        </div>
                                        <h3 className="text-2xl font-bold">Audio threat detection</h3>
                                    </div>
                                    
                                    <div className="flex flex-col md:flex-row gap-8 items-center">
                                        <div className="w-full md:w-1/2">
                                            <Waveform active={true} />
                                        </div>
                                        <div className="w-full md:w-1/2 space-y-4">
                                            <p className="text-slate-400 text-sm leading-relaxed">
                                                Detects hate speech, harassment, and localized slang in real-time audio streams.
                                            </p>
                                            <div className="flex flex-wrap gap-2">
                                                <span className="px-2 py-1 rounded bg-purple-500/10 border border-purple-500/20 text-purple-300 text-xs font-mono">
                                                    99.8% Accuracy
                                                </span>
                                                <span className="px-2 py-1 rounded bg-purple-500/10 border border-purple-500/20 text-purple-300 text-xs font-mono">
                                                    &lt; 200ms Latency
                                                </span>
                                            </div>
                                        </div>
                                    </div>
                                </div>

                                {/* Text Threat Info */}
                                <div className="glass-panel rounded-2xl p-8 flex-1 flex flex-col justify-center">
                                    <div className="flex items-center gap-3 mb-6">
                                        <div className="p-3 bg-amber-500/10 rounded-xl border border-amber-500/20">
                                            <FileText className="w-6 h-6 text-amber-500" />
                                        </div>
                                        <h3 className="text-2xl font-bold">Text threat detection</h3>
                                    </div>
                                    
                                    <div className="space-y-4">
                                        <div className="bg-slate-900/50 p-4 rounded-xl border border-white/5 flex items-center gap-4">
                                            <Globe className="w-5 h-5 text-cyan-400 shrink-0" />
                                            <span className="text-sm font-medium text-slate-200">Multilingual intent classification</span>
                                        </div>
                                        <div className="bg-slate-900/50 p-4 rounded-xl border border-white/5 flex items-center gap-4">
                                            <AlertTriangle className="w-5 h-5 text-amber-400 shrink-0" />
                                            <span className="text-sm font-medium text-slate-200">Hate/harassment pattern detection</span>
                                        </div>
                                        <div className="bg-slate-900/50 p-4 rounded-xl border border-white/5 flex items-center gap-4">
                                            <Sliders className="w-5 h-5 text-emerald-400 shrink-0" />
                                            <span className="text-sm font-medium text-slate-200">Adaptive thresholds per platform</span>
                                        </div>
                                    </div>
                                </div>

                            </div>

                        </div>
                    </section>

                    {/* DEEP ANALYSIS CAPABILITIES (Heatmap/Context) */}
                    <section className="py-24 px-6 border-t border-white/5 bg-[#020617]">
                        <div className="max-w-7xl mx-auto">
                            <div className="text-center mb-16">
                                <h2 className="text-3xl font-bold mb-4">Deep Analysis Capabilities</h2>
                                <p className="text-slate-400">Advanced visualization tools for enterprise moderation teams.</p>
                            </div>

                            <div className="grid lg:grid-cols-12 gap-8">
                                <div className="lg:col-span-4 glass-panel rounded-2xl p-6 flex flex-col">
                                    <div className="mb-6">
                                        <h3 className="font-bold text-lg mb-2">Context-aware detection</h3>
                                        <p className="text-xs text-slate-400 leading-relaxed">
                                            Surface exactly where risk appears. Reviewers get timestamps and model rationale.
                                        </p>
                                    </div>
                                    <div className="space-y-3 mb-8">
                                        <div className="bg-slate-800/50 p-3 rounded-lg border border-white/5 flex items-center gap-3">
                                            <Activity className="w-4 h-4 text-cyan-400" />
                                            <span className="text-sm text-slate-300">Segment-level confidence</span>
                                        </div>
                                        <div className="bg-slate-800/50 p-3 rounded-lg border border-white/5 flex items-center gap-3">
                                            <Eye className="w-4 h-4 text-cyan-400" />
                                            <span className="text-sm text-slate-300">Explainable highlights</span>
                                        </div>
                                        <div className="bg-slate-800/50 p-3 rounded-lg border border-white/5 flex items-center gap-3">
                                            <ArrowRight className="w-4 h-4 text-cyan-400" />
                                            <span className="text-sm text-slate-300">One-click escalation</span>
                                        </div>
                                    </div>
                                    <div className="mt-auto">
                                        <div className="flex justify-between text-xs mb-2">
                                            <span className="text-slate-500">Detection confidence</span>
                                            <span className="text-cyan-400 font-bold">94.7%</span>
                                        </div>
                                        <div className="w-full bg-slate-800 h-1.5 rounded-full overflow-hidden">
                                            <div className="h-full bg-cyan-400 w-[94.7%] shadow-[0_0_10px_rgba(34,211,238,0.5)]"></div>
                                        </div>
                                    </div>
                                </div>

                                <div className="lg:col-span-8 glass-panel rounded-2xl p-6 flex flex-col relative overflow-hidden">
                                    <div className="flex items-center gap-3 mb-6">
                                        <div className="p-2 bg-red-500/10 rounded-lg">
                                            <Activity className="w-5 h-5 text-red-500" />
                                        </div>
                                        <h3 className="font-bold text-lg">Violation heatmap analysis</h3>
                                    </div>
                                    <div className="h-32 w-full bg-black/50 rounded-xl mb-6 flex items-center justify-center border border-white/5 relative overflow-hidden">
                                        <div className="absolute inset-0 opacity-20 bg-[linear-gradient(rgba(255,255,255,0.05)_1px,transparent_1px),linear-gradient(90deg,rgba(255,255,255,0.05)_1px,transparent_1px)] bg-[size:20px_20px]"></div>
                                        <Waveform active={true} />
                                    </div>
                                    <div className="mb-2 flex items-center gap-2 text-xs text-slate-500 font-mono uppercase tracking-widest">
                                        <RefreshCw className="w-3 h-3" /> Timeline Analysis
                                    </div>
                                    
                                    {/* UPDATED ACCURATE HEATMAP VISUAL */}
                                    <AccurateHeatmap />
                                    
                                    <div className="bg-slate-900/50 rounded-xl p-4 border border-white/5 flex justify-between items-center mt-4">
                                        <div>
                                            <span className="text-slate-500 text-xs block mb-1">Selected segment</span>
                                            <span className="text-white font-bold block">Violation</span>
                                        </div>
                                        <div className="text-center">
                                            <span className="text-slate-500 text-xs block mb-1">Duration</span>
                                            <span className="text-cyan-400 font-mono block">1:12 - 1:24</span>
                                        </div>
                                        <div className="text-right">
                                            <span className="text-slate-500 text-xs block mb-1">Risk level</span>
                                            <span className="text-red-500 font-bold block">High</span>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </section>

                    {/* S-SCORE */}
                    <section className="py-24 px-6 relative overflow-hidden">
                        <div className="max-w-6xl mx-auto grid md:grid-cols-2 gap-16 items-center">
                            <div>
                                <div className="mb-8">
                                    <h2 className="text-3xl font-bold mb-4">S-Score interpretation</h2>
                                    <p className="text-slate-400 text-sm leading-relaxed">
                                        A single normalized score derived from image, text, and audio signals—calibrated to your platform's risk tolerance.
                                    </p>
                                </div>
                                <div className="space-y-4">
                                    <div className="glass-panel rounded-xl p-4 flex items-center justify-between hover:border-emerald-500/50 transition-colors">
                                        <div className="flex items-start gap-3">
                                            <CheckCircle className="w-5 h-5 text-emerald-400 mt-0.5" />
                                            <div>
                                                <h4 className="text-slate-200 font-medium">Low to No Risk</h4>
                                                <span className="text-emerald-400 text-xs font-bold uppercase">Safe to Publish</span>
                                            </div>
                                        </div>
                                        <span className="text-slate-500 font-mono">50–100</span>
                                    </div>
                                    <div className="glass-panel rounded-xl p-4 flex items-center justify-between hover:border-amber-500/50 transition-colors">
                                        <div className="flex items-start gap-3">
                                            <AlertTriangle className="w-5 h-5 text-amber-400 mt-0.5" />
                                            <div>
                                                <h4 className="text-slate-200 font-medium">Moderate Risk</h4>
                                                <span className="text-amber-400 text-xs font-bold uppercase">Unsafe to Publish</span>
                                            </div>
                                        </div>
                                        <span className="text-slate-500 font-mono">20–49</span>
                                    </div>
                                    <div className="glass-panel rounded-xl p-4 flex items-center justify-between hover:border-red-500/50 transition-colors">
                                        <div className="flex items-start gap-3">
                                            <X className="w-5 h-5 text-red-500 mt-0.5" />
                                            <div>
                                                <h4 className="text-slate-200 font-medium">High Risk</h4>
                                                <span className="text-red-500 text-xs font-bold uppercase">Immediate Strike</span>
                                            </div>
                                        </div>
                                        <span className="text-slate-500 font-mono">0–19</span>
                                    </div>
                                </div>
                            </div>
                            <div className="flex flex-col items-center justify-center p-8 bg-black/30 rounded-3xl border border-white/5 backdrop-blur-sm shadow-2xl">
                                <SafetyMeter score={15} /> 
                                <div className="mt-8 text-center">
                                    <div className="inline-flex items-center gap-2 px-4 py-2 bg-red-500/10 border border-red-500/20 rounded-lg text-red-400 text-sm font-bold animate-pulse">
                                        <AlertTriangle className="w-4 h-4" /> ACTION REQUIRED
                                    </div>
                                </div>
                            </div>
                        </div>
                    </section>

                    {/* CONTACT */}
                    <section className="py-24 px-6 border-t border-white/5 bg-[#020617]">
                        <div className="max-w-6xl mx-auto grid md:grid-cols-2 gap-16">
                            <div>
                                <h2 className="text-3xl font-bold mb-8">Request a demo</h2>
                                <form className="space-y-6">
                                    <div className="space-y-2">
                                        <input type="text" placeholder="Your name" className="w-full bg-slate-900 border border-white/10 rounded-lg p-3 text-sm focus:border-cyan-500 focus:outline-none transition-colors" />
                                    </div>
                                    <div className="space-y-2">
                                        <input type="email" placeholder="you@company.com" className="w-full bg-slate-900 border border-white/10 rounded-lg p-3 text-sm focus:border-cyan-500 focus:outline-none transition-colors" />
                                    </div>
                                    <button className="w-full bg-cyan-500 hover:bg-cyan-400 text-black font-bold py-3 rounded-lg flex items-center justify-center gap-2 transition-transform active:scale-95 shadow-[0_0_15px_rgba(34,211,238,0.4)]">
                                        <Send className="w-4 h-4" /> Send message
                                    </button>
                                </form>
                            </div>
                            <div>
                                <h2 className="text-3xl font-bold mb-8">Get in touch</h2>
                                <div className="space-y-8 mb-10">
                                    <div className="flex items-start gap-4">
                                        <div className="w-10 h-10 rounded-lg bg-slate-800 flex items-center justify-center text-cyan-400">
                                            <Send className="w-5 h-5" />
                                        </div>
                                        <div>
                                            <h4 className="text-xs font-bold text-slate-500 uppercase mb-1">Email</h4>
                                            <p className="text-slate-200">hello@metron.ai</p>
                                        </div>
                                    </div>
                                    <div className="flex items-start gap-4">
                                        <div className="w-10 h-10 rounded-lg bg-slate-800 flex items-center justify-center text-cyan-400">
                                            <MapPin className="w-5 h-5" />
                                        </div>
                                        <div>
                                            <h4 className="text-xs font-bold text-slate-500 uppercase mb-1">Location</h4>
                                            <p className="text-slate-200">Bengaluru, India</p>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </section>
                </React.Fragment>
            );

            const MetronAIPage = () => (
                <div className="min-h-screen pt-28 pb-20 px-6">
                    <div className="max-w-4xl mx-auto">
                        <div className="flex items-center gap-4 mb-8">
                            <button 
                                onClick={() => navigateTo('home')}
                                className="p-2 rounded-full bg-white/5 hover:bg-white/10 transition-colors"
                            >
                                <ArrowRight className="w-5 h-5 rotate-180" />
                            </button>
                            <div>
                                <h1 className="text-3xl font-bold">Metron AI</h1>
                                <p className="text-slate-400 text-sm">Automated Content Moderation Engine</p>
                            </div>
                        </div>

                        {/* UPLOAD & SCAN SECTION */}
                        <div className={`glass-panel p-8 rounded-3xl mb-8 relative overflow-hidden transition-all duration-500 ${isProcessing ? 'h-[400px]' : 'h-auto'}`}>
                            <div className="absolute top-0 left-0 w-full h-1 bg-gradient-to-r from-cyan-500 to-blue-500 opacity-50" />
                            
                            {!uploadedFile ? (
                                <div className="flex flex-col items-center justify-center py-12 text-center">
                                    <input 
                                        type="file" 
                                        ref={fileInputRef} 
                                        onChange={handleFileUpload} 
                                        className="hidden" 
                                        accept="video/*" 
                                    />
                                    <div 
                                        onClick={() => fileInputRef.current.click()}
                                        className="w-24 h-24 bg-slate-800 rounded-full flex items-center justify-center mb-6 hover:bg-slate-700 transition-colors cursor-pointer border border-white/5 hover:border-cyan-500/50 group"
                                    >
                                        <Upload className="w-10 h-10 text-cyan-400 group-hover:scale-110 transition-transform" />
                                    </div>
                                    <h3 className="text-2xl font-bold mb-2">Upload Content</h3>
                                    <p className="text-slate-400 text-sm max-w-md mx-auto mb-6">
                                        Drag & drop or select a video file to begin automated analysis. 
                                        Metron will scan for audio, visual, and textual violations.
                                    </p>
                                    <button 
                                        onClick={() => fileInputRef.current.click()}
                                        className="px-8 py-3 bg-white text-black font-bold rounded-lg hover:bg-slate-200 transition-colors"
                                    >
                                        Select File
                                    </button>
                                </div>
                            ) : isProcessing ? (
                                <div className="flex flex-col items-center justify-center py-12 text-center h-full">
                                    {/* Dynamic Loading Visualization */}
                                    <div className="w-32 h-32 relative mb-8">
                                        <div className="absolute inset-0 border-4 border-slate-800 rounded-full"></div>
                                        <div className="absolute inset-0 border-4 border-cyan-500 rounded-full border-t-transparent animate-spin"></div>
                                        <div className="absolute inset-2 border-4 border-slate-800 rounded-full opacity-50"></div>
                                        <div className="absolute inset-2 border-4 border-purple-500 rounded-full border-b-transparent animate-spin-slow"></div>
                                        <div className="absolute inset-0 flex items-center justify-center flex-col">
                                            <span className="font-bold text-3xl text-white tabular-nums">{processProgress}%</span>
                                        </div>
                                    </div>
                                    <h3 className="text-2xl font-bold mb-3 animate-pulse text-cyan-400">{processingStage}</h3>
                                    
                                    <div className="flex gap-2 text-xs text-slate-500 mt-4">
                                        <div className={`flex items-center gap-1 ${processProgress > 30 ? 'text-emerald-400' : ''}`}>
                                            <CheckCircle className="w-3 h-3" /> Audio
                                        </div>
                                        <span className="text-slate-700">•</span>
                                        <div className={`flex items-center gap-1 ${processProgress > 60 ? 'text-emerald-400' : ''}`}>
                                            <CheckCircle className="w-3 h-3" /> Visuals
                                        </div>
                                        <span className="text-slate-700">•</span>
                                        <div className={`flex items-center gap-1 ${processProgress > 90 ? 'text-emerald-400' : ''}`}>
                                            <CheckCircle className="w-3 h-3" /> Context
                                        </div>
                                    </div>
                                </div>
                            ) : (
                                <div className="flex items-center justify-between">
                                    <div className="flex items-center gap-4">
                                        <div className="w-16 h-16 bg-cyan-900/20 rounded-xl flex items-center justify-center border border-cyan-500/20">
                                            <FileText className="w-8 h-8 text-cyan-400" />
                                        </div>
                                        <div>
                                            <h3 className="font-bold text-lg">{uploadedFile.name}</h3>
                                            <p className="text-emerald-400 text-xs flex items-center gap-1">
                                                <CheckCircle className="w-3 h-3" /> Analysis Complete
                                            </p>
                                        </div>
                                    </div>
                                    <button 
                                        onClick={() => { setUploadedFile(null); setAnalysisResult(null); }}
                                        className="text-slate-400 hover:text-white text-sm underline"
                                    >
                                        Analyze Another
                                    </button>
                                </div>
                            )}
                        </div>

                        {/* MODERATION RESULTS SECTION */}
                        {analysisResult && (
                            <div className="animate-fade-in-up">
                                <div className="flex items-center justify-between mb-6">
                                    <h2 className="text-xl font-bold flex items-center gap-2">
                                        <Sliders className="w-5 h-5 text-cyan-400" /> Moderation Result
                                    </h2>
                                    <div className="px-3 py-1 bg-slate-800 rounded-full text-xs text-slate-400 border border-white/5">
                                        Feedback Generated
                                    </div>
                                </div>

                                <div className="grid md:grid-cols-2 gap-8">
                                    
                                    {/* Left: Verdict Card */}
                                    <div className="glass-panel rounded-3xl p-8 flex flex-col items-center justify-center text-center relative overflow-hidden shadow-2xl">
                                        <div className={`absolute top-0 inset-x-0 h-1 ${analysisResult.score >= 50 ? 'bg-emerald-500' : analysisResult.score >= 20 ? 'bg-amber-500' : 'bg-red-500'}`} />
                                        
                                        <SafetyMeter score={analysisResult.score} />

                                        <div className="mt-8 space-y-3">
                                            <div className={`text-3xl font-bold tracking-tight ${analysisResult.colorClass}`}>
                                                {analysisResult.verdict}
                                            </div>
                                            
                                            {/* Feedback Box */}
                                            <div className="bg-white/5 p-4 rounded-xl border border-white/5 mt-4">
                                                <div className="flex items-start gap-3 text-left">
                                                    <MessageSquare className="w-5 h-5 text-cyan-400 mt-1 shrink-0" />
                                                    <div>
                                                        <p className="text-xs font-bold text-slate-400 uppercase mb-1">Metron Feedback</p>
                                                        <p className="text-slate-200 text-sm leading-relaxed">
                                                            {analysisResult.message}
                                                        </p>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                    </div>

                                    {/* Right: Detailed Breakdown */}
                                    <div className="glass-panel rounded-3xl p-8 flex flex-col justify-center">
                                        <h3 className="font-bold text-lg mb-6">Score Breakdown</h3>
                                        
                                        <div className="space-y-6">
                                            <div>
                                                <div className="flex justify-between text-sm mb-2">
                                                    <span className="text-slate-400">Audio Analysis</span>
                                                    <span className={analysisResult.score > 40 ? "text-emerald-400" : "text-amber-400"}>
                                                        {analysisResult.score > 40 ? "Clean" : "Flags Detected"}
                                                    </span>
                                                </div>
                                                <div className="w-full bg-slate-800 h-2 rounded-full overflow-hidden">
                                                    <div 
                                                        className={`h-full ${analysisResult.score > 40 ? "bg-emerald-500" : "bg-amber-500"}`} 
                                                        style={{ width: `${Math.min(100, analysisResult.score + 20)}%` }}
                                                    ></div>
                                                </div>
                                            </div>

                                            <div>
                                                <div className="flex justify-between text-sm mb-2">
                                                    <span className="text-slate-400">Visual Safety</span>
                                                    <span className={analysisResult.score > 20 ? "text-emerald-400" : "text-red-500"}>
                                                        {analysisResult.score > 20 ? "Pass" : "Severe Risk"}
                                                    </span>
                                                </div>
                                                <div className="w-full bg-slate-800 h-2 rounded-full overflow-hidden">
                                                    <div 
                                                        className={`h-full ${analysisResult.score > 20 ? "bg-emerald-500" : "bg-red-500"}`} 
                                                        style={{ width: `${Math.max(10, analysisResult.score)}%` }}
                                                    ></div>
                                                </div>
                                            </div>

                                            <div>
                                                <div className="flex justify-between text-sm mb-2">
                                                    <span className="text-slate-400">Text/Context</span>
                                                    <span className="text-emerald-400">Verified</span>
                                                </div>
                                                <div className="w-full bg-slate-800 h-2 rounded-full overflow-hidden">
                                                    <div className="h-full bg-emerald-500" style={{ width: '95%' }}></div>
                                                </div>
                                            </div>
                                        </div>

                                        <div className="mt-8 pt-6 border-t border-white/5 flex gap-4">
                                            <button className="flex-1 py-3 bg-white/5 hover:bg-white/10 rounded-lg text-sm font-bold transition-colors">
                                                Download Report
                                            </button>
                                            <button className="flex-1 py-3 bg-cyan-500/10 hover:bg-cyan-500/20 text-cyan-400 rounded-lg text-sm font-bold transition-colors border border-cyan-500/30">
                                                View Details
                                            </button>
                                        </div>
                                    </div>

                                </div>
                            </div>
                        )}

                    </div>
                </div>
            );

            return (
                <div className="min-h-screen font-sans selection:bg-cyan-500/30 overflow-x-hidden">
                    
                    {/* --- NAVBAR --- */}
                    <nav className={`fixed top-0 w-full z-50 transition-all duration-300 ${scrollY > 50 ? 'bg-[#020617]/90 backdrop-blur-md border-b border-white/10' : 'bg-transparent'} h-20 flex items-center`}>
                        <div className="max-w-7xl mx-auto w-full px-6 flex items-center justify-between">
                            <div className="flex items-center gap-2 cursor-pointer group" onClick={() => navigateTo('home')}>
                                <div className="text-cyan-400 group-hover:rotate-12 transition-transform">
                                    <Shield className="w-8 h-8" />
                                </div>
                                <span className="text-xl font-bold tracking-widest text-white">METRON</span>
                            </div>
                            <div className="hidden md:flex gap-8 text-sm font-medium text-slate-400">
                                <button 
                                    onClick={() => navigateTo('home')} 
                                    className={`hover:text-cyan-400 transition-colors ${activePage === 'home' ? 'text-cyan-400' : ''}`}
                                >
                                    HOME
                                </button>
                                <button 
                                    onClick={() => navigateTo('metron-ai')}
                                    className={`hover:text-cyan-400 transition-colors ${activePage === 'metron-ai' ? 'text-cyan-400' : ''}`}
                                >
                                    METRON AI
                                </button>
                            </div>
                            <button 
                                onClick={() => navigateTo('metron-ai')}
                                className="bg-cyan-500 hover:bg-cyan-400 text-black px-6 py-2 rounded-full font-bold text-sm transition-transform hover:scale-105 active:scale-95 shadow-[0_0_10px_rgba(6,182,212,0.5)]"
                            >
                                GET ACCESS
                            </button>
                        </div>
                    </nav>

                    {/* --- PAGE CONTENT --- */}
                    {activePage === 'home' ? <HomePage /> : <MetronAIPage />}

                    {/* --- FOOTER --- */}
                    <footer className="py-8 text-center text-slate-600 text-xs border-t border-white/5 bg-[#020617]">
                        <p>&copy; 2026 Metron Systems. All rights reserved.</p>
                    </footer>
                </div>
            );
        }

        const root = createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
