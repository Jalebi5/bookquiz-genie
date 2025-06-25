import React, { useState, useCallback, useMemo, useEffect, useRef } from 'react';
import { useDropzone } from 'react-dropzone';
import { UploadCloud, Settings, BrainCircuit, Sparkles, Loader, X, Copy, Clock, Info, Sun, Moon, History, Download, Home, LogOut } from 'lucide-react';

// --- HELPER HOOK for Dark Mode & Local Storage ---
const useStickyState = (defaultValue, key) => {
    const [value, setValue] = useState(() => {
        try {
            const stickyValue = window.localStorage.getItem(key);
            return stickyValue !== null ? JSON.parse(stickyValue) : defaultValue;
        } catch (error) {
            console.error("Error reading from localStorage", error);
            return defaultValue;
        }
    });
    useEffect(() => {
        try {
            window.localStorage.setItem(key, JSON.stringify(value));
        } catch (error) {
            console.error("Error writing to localStorage", error);
        }
    }, [key, value]);
    return [value, setValue];
};

// --- UTILITY for rendering bolded explanations ---
const renderExplanation = (text) => {
    if (!text) return null;
    const parts = text.split(/\*\*(.*?)\*\*/g);
    return parts.map((part, index) => 
        index % 2 === 1 ? <strong key={index} className="text-indigo-500 dark:text-indigo-400">{part}</strong> : <span key={index}>{part}</span>
    );
};
const renderExplanationForHTML = (text) => {
    if (!text) return '';
    return text.replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>');
}


// =================================================================================
// --- UI Components ---
// =================================================================================

const Sidebar = ({ theme, setTheme, goTo }) => (
    <>
        <div className="hidden md:flex flex-col w-64 h-screen bg-white dark:bg-slate-800/80 border-r border-slate-200 dark:border-slate-700 p-4 fixed top-0 left-0">
            <div className="flex items-center gap-3 mb-8">
                <div className="bg-indigo-100 dark:bg-indigo-900/80 p-2 rounded-lg"><BrainCircuit className="text-indigo-600 dark:text-indigo-400 w-8 h-8" /></div>
                <h1 className="text-xl font-bold text-slate-800 dark:text-white">BookQuiz Genie</h1>
            </div>
            <nav className="flex-1 space-y-2">
                <SidebarItem icon={<Home />} onClick={() => goTo('uploader')}>Home</SidebarItem>
                <SidebarItem icon={<History />} onClick={() => goTo('history')}>History</SidebarItem>
            </nav>
            <div>
                <p className="px-3 text-xs font-semibold text-slate-400 uppercase mb-2">Theme</p>
                <SidebarItem icon={<Sun />} onClick={() => setTheme('light')} isActive={theme === 'light'}>Light Mode</SidebarItem>
                <SidebarItem icon={<Moon />} onClick={() => setTheme('dark')} isActive={theme === 'dark'}>Dark Mode</SidebarItem>
                <div className="border-t border-slate-200 dark:border-slate-700 my-2"></div>
                <SidebarItem icon={<LogOut />} onClick={() => goTo('landing')}>Exit to Welcome</SidebarItem>
            </div>
        </div>
        <div className="md:hidden fixed bottom-0 left-0 right-0 h-16 bg-white dark:bg-slate-800 border-t border-slate-200 dark:border-slate-700 flex justify-around items-center z-50">
            <MobileNavItem icon={<Home />} onClick={() => goTo('uploader')} />
            <MobileNavItem icon={<History />} onClick={() => goTo('history')} />
            <MobileNavItem icon={theme === 'light' ? <Moon /> : <Sun />} onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')} />
            <MobileNavItem icon={<LogOut />} onClick={() => goTo('landing')} />
        </div>
    </>
);

const SidebarItem = ({ icon, children, onClick, isActive }) => (
    <button onClick={onClick} className={`w-full flex items-center gap-3 px-3 py-2 text-sm rounded-md transition-colors ${isActive ? 'bg-indigo-50 dark:bg-indigo-900/80 text-indigo-600 dark:text-indigo-300 font-semibold' : 'text-slate-700 dark:text-slate-300 hover:bg-slate-100 dark:hover:bg-slate-700'}`}>
        {icon}<span>{children}</span>
    </button>
);

const MobileNavItem = ({ icon, onClick }) => (
     <button onClick={onClick} className="p-3 text-slate-600 dark:text-slate-300 hover:text-indigo-600 dark:hover:text-indigo-400 transition-colors">
        {React.cloneElement(icon, { className: 'w-6 h-6' })}
    </button>
);

const LandingPage = ({ onStart }) => (
    <div className="text-center flex flex-col items-center min-h-screen justify-center">
        <Sparkles className="w-16 h-16 mx-auto text-indigo-500 mb-4" />
        <h2 className="text-3xl font-bold text-slate-900 dark:text-white mb-2">Turn Any Book into a Quiz!</h2>
        <p className="text-slate-600 dark:text-slate-300 mb-8 max-w-prose">Snap a photo of your notes or a book page, and I'll magically generate a practice quiz for you.</p>
        <button onClick={onStart} className="bg-indigo-600 text-white font-bold py-3 px-8 rounded-lg shadow-lg hover:bg-indigo-700 transition-all transform hover:scale-105">Let's Get Started</button>
    </div>
);

const ImageUploader = ({ files, setFiles, onUpload, loading }) => {
    const onDrop = useCallback(acceptedFiles => {
        onUpload(acceptedFiles);
    }, [onUpload]);
    const { getRootProps, getInputProps, isDragActive } = useDropzone({ onDrop, accept: { 'image/*': ['.jpeg', '.jpg', '.png'] } });

    return (
        <div>
            <h2 className="text-xl font-bold mb-2 text-slate-900 dark:text-white">Step 1: Upload Your Pages</h2>
            <p className="text-slate-600 dark:text-slate-300 mb-6">Drag & drop or click to select your book pages or notes.</p>
            <div {...getRootProps()} className={`p-8 border-2 border-dashed rounded-xl text-center cursor-pointer transition-colors ${isDragActive ? `bg-indigo-50 dark:bg-indigo-900/50 border-indigo-400` : `bg-slate-50 dark:bg-slate-700/50 border-slate-300 dark:border-slate-600 hover:border-indigo-400 dark:hover:border-indigo-500`}`}>
                <input {...getInputProps()} />
                <UploadCloud className="w-12 h-12 mx-auto text-slate-400 dark:text-slate-500 mb-4" />
                <p className={`${isDragActive ? 'text-indigo-600 dark:text-indigo-400' : 'text-slate-500 dark:text-slate-400'} font-semibold`}>{isDragActive ? 'Drop the files here...' : 'Drag & drop images, or click to select'}</p>
            </div>
            
            {files.length > 0 && !loading && (
                 <div className="mt-6">
                    <div className="flex justify-between items-center mb-2">
                        <h3 className="font-semibold text-slate-800 dark:text-slate-200">Selected files:</h3>
                        <button onClick={() => setFiles([])} className="text-sm font-semibold text-red-500 hover:text-red-700 dark:hover:text-red-400 transition-colors flex items-center gap-1"><X className="w-4 h-4" /> Clear All</button>
                    </div>
                    <ul className="space-y-2">
                        {files.map((file, i) => 
                            <li key={i} className="p-2 bg-slate-50 dark:bg-slate-700/50 rounded-lg flex items-center justify-between">
                                <span className="text-sm text-slate-700 dark:text-slate-300 truncate">{file.name}</span>
                                <span className="text-xs text-slate-500 dark:text-slate-400 ml-2 flex-shrink-0">({Math.round(file.size / 1024)} KB)</span>
                            </li>
                        )}
                    </ul>
                </div>
            )}
            {loading && <div className="mt-6 text-center"><Loader className="animate-spin w-8 h-8 mx-auto text-indigo-500"/><p className="mt-2 font-semibold text-slate-700 dark:text-slate-200">Analyzing images and extracting text...</p></div>}
        </div>
    );
};

const TextPreview = ({ text, setText, onNext, loading, settings, setSettings }) => {
    const questionOptions = [5, 10, 15, 20, 30, 40, 50, 75, 100, 125, 150, 175, 200, 225];
    return (
    <div>
        <h2 className="text-xl font-bold text-slate-900 dark:text-white mb-2">Step 2: Review & Generate Quiz</h2>
        <p className="text-slate-600 dark:text-slate-300 mb-4">The text from your images is below. You can edit it before generating the quiz.</p>
        <textarea value={text} onChange={(e) => setText(e.target.value)} className="w-full h-64 p-3 border border-slate-300 dark:border-slate-600 rounded-lg bg-slate-50 dark:bg-slate-700 dark:text-slate-100 focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-shadow" placeholder="Extracted text will appear here..."/>
        <div className="mt-6 p-4 bg-slate-100 dark:bg-slate-700/50 rounded-lg border border-slate-200 dark:border-slate-700">
            <h3 className="font-semibold mb-4 flex items-center gap-2 text-slate-900 dark:text-white"><Settings className="w-5 h-5 text-slate-600 dark:text-slate-400"/> Quiz Settings</h3>
            <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div><label htmlFor="numQuestions" className="block text-sm font-medium text-slate-700 dark:text-slate-300 mb-1">Number of Questions</label><select id="numQuestions" value={settings.numQuestions} onChange={(e) => setSettings({...settings, numQuestions: parseInt(e.target.value)})} className="w-full p-2 border border-slate-300 dark:border-slate-600 rounded-lg bg-white dark:bg-slate-800 focus:ring-indigo-500 focus:border-indigo-500">{questionOptions.map(num => <option key={num} value={num}>{num}</option>)}</select></div>
                <div><label htmlFor="timePerQuestion" className="block text-sm font-medium text-slate-700 dark:text-slate-300 mb-1">Time per Question</label><select id="timePerQuestion" value={settings.timePerQuestion} onChange={(e) => setSettings({...settings, timePerQuestion: parseInt(e.target.value)})} className="w-full p-2 border border-slate-300 dark:border-slate-600 rounded-lg bg-white dark:bg-slate-800 focus:ring-indigo-500 focus:border-indigo-500"><option value="15">15 seconds</option><option value="30">30 seconds</option><option value="60">60 seconds</option></select></div>
                <div className="flex items-end"><label className="flex items-center gap-2 p-3 bg-white dark:bg-slate-800 border border-slate-300 dark:border-slate-600 rounded-lg cursor-pointer hover:bg-indigo-50 dark:hover:bg-slate-700 w-full h-full"><input type="checkbox" checked={settings.useInternet} onChange={(e) => setSettings({...settings, useInternet: e.target.checked})} className="h-4 w-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500"/><span className="text-sm font-medium text-slate-700 dark:text-slate-300">Enrich Explanations</span></label></div>
            </div>
        </div>
        <button onClick={onNext} disabled={loading || !text} className="w-full mt-6 bg-indigo-600 text-white font-bold py-3 px-6 rounded-lg shadow-md hover:bg-indigo-700 transition-all disabled:opacity-50 disabled:cursor-not-allowed flex items-center justify-center gap-2">{loading ? <><Loader className="animate-spin w-5 h-5"/> Generating...</> : 'Generate Quiz'}</button>
    </div>
    )
};

const QuizView = ({ quiz, setQuiz, onFinish, settings }) => {
    const [currentIndex, setCurrentIndex] = useState(0);
    const [showResults, setShowResults] = useState(false);
    const [timer, setTimer] = useState(settings.timePerQuestion);
    const quizTopRef = useRef(null);

    const handleNext = useCallback(() => {
        setShowResults(false);
        if (currentIndex < quiz.length - 1) {
            setCurrentIndex(prev => prev + 1);
        } else {
            onFinish(quiz);
        }
    }, [currentIndex, quiz, onFinish]);

    const handleAnswer = useCallback((option) => {
        if (showResults) return;
        const newQuiz = [...quiz];
        newQuiz[currentIndex].userAnswer = option;
        setQuiz(newQuiz);
        setShowResults(true);
    }, [currentIndex, quiz, setQuiz, showResults]);

    useEffect(() => {
        if (showResults) return;
        setTimer(settings.timePerQuestion);
        const interval = setInterval(() => setTimer(prev => (prev <= 1 ? 0 : prev - 1)), 1000);
        return () => clearInterval(interval);
    }, [currentIndex, showResults, settings.timePerQuestion]);

    useEffect(() => {
        if (timer === 0 && !showResults) {
            handleAnswer(null);
        }
    }, [timer, showResults, handleAnswer]);
    
    useEffect(() => {
        if (quizTopRef.current) {
            quizTopRef.current.scrollIntoView({ behavior: 'smooth', block: 'start' });
        }
    }, [currentIndex]);

    const currentQuestion = quiz[currentIndex];
    if (!currentQuestion) return <div className="text-center p-8"><Loader className="animate-spin w-8 h-8 mx-auto text-indigo-500" /><p className="mt-2">Loading quiz...</p></div>;

    return (
        <div ref={quizTopRef} className="pb-16 md:pb-0">
            <div className="bg-slate-50 dark:bg-slate-700/50 p-4 rounded-lg border border-slate-200 dark:border-slate-700">
                <div className="flex justify-between items-center mb-4"><p className="font-bold text-indigo-600 dark:text-indigo-400">Question {currentIndex + 1} of {quiz.length}</p><div className={`flex items-center gap-2 font-bold ${timer <= 5 ? 'text-red-500' : 'text-slate-500 dark:text-slate-400'}`}><Clock className="w-5 h-5" /> {timer}s</div></div>
                <p className="text-lg font-semibold mb-6 text-slate-900 dark:text-white">{currentQuestion.question}</p>
                <div className="space-y-3">{currentQuestion.options.map((option, i) => { const isSelected = currentQuestion.userAnswer === option; const isCorrect = currentQuestion.correctAnswer === option; let buttonClass = 'w-full text-left p-4 rounded-lg border-2 transition-all disabled:opacity-90'; if (showResults) { if (isCorrect) buttonClass += ' bg-green-100 dark:bg-green-900/50 border-green-400 text-green-800 dark:text-green-300 font-semibold'; else if (isSelected) buttonClass += ' bg-red-100 dark:bg-red-900/50 border-red-400 text-red-800 dark:text-red-300 font-semibold'; else buttonClass += ' bg-white dark:bg-slate-800 border-slate-300 dark:border-slate-600'; } else buttonClass += ` bg-white dark:bg-slate-800 border-slate-300 dark:border-slate-600 hover:bg-indigo-50 dark:hover:bg-slate-700 hover:border-indigo-400 dark:hover:border-indigo-500`; return <button key={i} onClick={() => handleAnswer(option)} disabled={showResults} className={buttonClass}>{option}</button>; })}</div>
            </div>
            {showResults && (<div className="mt-4 p-4 bg-slate-100 dark:bg-slate-700/50 rounded-lg"><h4 className="font-bold mb-2 flex items-center gap-2 text-indigo-600 dark:text-indigo-400"><Info className="w-5 h-5" /> Explanation</h4><p className="text-sm text-slate-700 dark:text-slate-300">{renderExplanation(currentQuestion.explanation)}</p></div>)}
            <div className="mt-6 flex justify-end">{showResults && <button onClick={handleNext} className="bg-indigo-600 text-white font-bold py-3 px-8 rounded-lg shadow-md hover:bg-indigo-700 transition-all">{currentIndex < quiz.length - 1 ? 'Next Question' : 'Finish Quiz'}</button>}</div>
        </div>
    );
};

const Confetti = () => {
    const [particles, setParticles] = useState([]);
    useEffect(() => { setParticles(Array.from({ length: 100 }, (_, i) => ({ id: i, x: Math.random() * 100, y: -20 - Math.random() * 100, r: Math.random() * 360, d: Math.random() * 2 + 1, color: `hsl(${Math.random() * 360}, 70%, 60%)`, }))); }, []);
    return (<div className="absolute top-0 left-0 w-full h-full overflow-hidden pointer-events-none z-50">{particles.map(p => (<div key={p.id} className="absolute w-2 h-4" style={{ background: p.color, top: `${p.y}%`, left: `${p.x}%`, transform: `rotate(${p.r}deg)`, animation: `fall ${p.d}s linear forwards`, }} />))}<style>{`@keyframes fall { to { transform: translateY(120vh) rotate(720deg); } }`}</style></div>);
};

const WorkHardAnimation = () => (
    <div className="flex justify-center items-center my-4"><svg width="150" height="100" viewBox="0 0 150 100"><style>{`.pencil-lead { animation: write 3s linear infinite; } .line { stroke-dasharray: 200; stroke-dashoffset: 200; animation: draw 3s linear infinite; } @keyframes write { 0%, 100% { transform: translateX(0); } 50% { transform: translateX(-5px); } } @keyframes draw { to { stroke-dashoffset: 0; } }`}</style><path d="M20 10 H 130 V 90 H 20 Z" fill="#fff" className="dark:fill-slate-700" stroke="#ccc" strokeWidth="1" /><path d="M30 30 H 120" stroke="#e0e0e0" className="dark:stroke-slate-600" strokeWidth="1" /><path d="M30 50 H 120" stroke="#e0e0e0" className="dark:stroke-slate-600" strokeWidth="1" /><path d="M30 70 q 20 -10, 40 0 t 40 0" stroke="#4a5568" strokeWidth="2" fill="none" className="dark:stroke-slate-400 line" /><g transform="translate(10, 65) rotate(-20)"><path d="M0 0 L 80 0 L 80 10 L 0 10 Z" fill="#f6e05e" /><path d="M80 0 L 90 5 L 80 10 Z" fill="#d69e2e" /><path d="M90 5 L 88 5 L 80 2 Z" fill="#2d3748" className="pencil-lead"/></g></svg></div>
);

const ResultsPage = ({ quiz, onRestart, onDownload }) => {
    const score = useMemo(() => quiz.reduce((acc, q) => (q.userAnswer === q.correctAnswer ? acc + 1 : acc), 0), [quiz]);
    const percentage = quiz.length > 0 ? Math.round((score / quiz.length) * 100) : 0;
    return (
        <div className="text-center relative pb-16 md:pb-0">{percentage >= 60 && <Confetti />} {percentage < 60 && <WorkHardAnimation />}<h2 className="text-2xl font-bold mb-2 text-slate-900 dark:text-white">Quiz Results</h2><div className="bg-indigo-50 dark:bg-slate-700 inline-flex flex-col items-center justify-center p-8 rounded-full border-8 border-indigo-200 dark:border-indigo-800 aspect-square w-48 h-48 mx-auto my-6"><span className="text-5xl font-bold text-indigo-600 dark:text-indigo-400">{score}/{quiz.length}</span><span className="text-lg text-slate-600 dark:text-slate-300">Correct</span><span className="font-bold text-2xl text-indigo-600 dark:text-indigo-400 mt-1">{percentage}%</span></div><div className="my-6 flex justify-center gap-2"><button onClick={() => onDownload(quiz, score, percentage)} className="flex items-center gap-2 px-4 py-2 bg-slate-100 dark:bg-slate-700 border border-slate-300 dark:border-slate-600 rounded-lg shadow-sm hover:bg-slate-200 dark:hover:bg-slate-600 transition-colors"><Download className="w-5 h-5"/> Download</button></div><button onClick={onRestart} className="mt-4 bg-indigo-600 text-white font-bold py-3 px-8 rounded-lg shadow-lg hover:bg-indigo-700 transition-all transform hover:scale-105">Create Another Quiz</button></div>
    );
};

const HistoryPage = ({ history, onRestart, onDownload }) => {
    if (history.length === 0) { return (<div className="text-center py-12"><h2 className="text-xl font-bold mb-2">No History Yet</h2><p className="text-slate-600 dark:text-slate-400 mb-6">Complete a quiz to see your results here.</p><button onClick={onRestart} className="bg-indigo-600 text-white font-bold py-3 px-8 rounded-lg shadow-lg hover:bg-indigo-700 transition-all">Start a New Quiz</button></div>); }
    return (
        <div className="pb-16 md:pb-0"><h2 className="text-2xl font-bold mb-4 text-slate-900 dark:text-white">Quiz History</h2><div className="space-y-4">{history.map(item => (<div key={item.id} className="p-4 bg-slate-50 dark:bg-slate-700/50 rounded-lg border border-slate-200 dark:border-slate-700 flex items-center justify-between"><div><p className="font-semibold text-slate-800 dark:text-slate-100">Quiz from {new Date(item.date).toLocaleDateString()}</p><p className="text-sm text-slate-600 dark:text-slate-300">Score: <span className="font-bold">{item.score}/{item.total}</span></p></div><button onClick={() => onDownload(item.quiz, item.score, Math.round((item.score / item.total) * 100))} className="flex items-center gap-2 px-3 py-2 bg-white dark:bg-slate-800 border border-slate-300 dark:border-slate-600 rounded-lg shadow-sm hover:bg-slate-100 dark:hover:bg-slate-700 transition-colors"><Download className="w-5 h-5"/><span className="hidden sm:inline">Download</span></button></div>))}</div></div>
    );
};

const ErrorDisplay = ({ message, onDismiss }) => (
    <div className="bg-red-100 border-l-4 border-red-500 text-red-700 dark:bg-red-900/30 dark:text-red-300 dark:border-red-700 p-4 rounded-lg mb-6 flex justify-between items-center" role="alert">
        <div><p className="font-bold">An Error Occurred</p><p className="text-sm">{message}</p></div>
        <button onClick={onDismiss} className="p-1 rounded-full hover:bg-red-200 dark:hover:bg-red-800/50"><X className="w-5 h-5" /></button>
    </div>
);


// =================================================================================
// --- Main App Wrapper ---
// =================================================================================
const App = () => {
    const [theme, setTheme] = useStickyState('light', 'theme');
    const [quizHistory, setQuizHistory] = useStickyState([], 'quizHistory');
    const [currentPage, setCurrentPage] = useState('landing');
    const [files, setFiles] = useState([]);
    const [extractedText, setExtractedText] = useState('');
    const [quiz, setQuiz] = useState([]);
    const [quizSettings, setQuizSettings] = useState({ numQuestions: 5, useInternet: false, timePerQuestion: 30 });
    const [loading, setLoading] = useState({ extracting: false, generating: false });
    const [error, setError] = useState(null);

    useEffect(() => {
        const root = window.document.documentElement;
        root.classList.remove('light', 'dark');
        root.classList.add(theme);
    }, [theme]);

    const goTo = (page) => {
        setError(null);
        setCurrentPage(page);
    };
    
    const handleExtractText = useCallback(async (uploadedFiles) => {
        if (uploadedFiles.length === 0 || loading.extracting) return;
        setFiles(uploadedFiles);
        setLoading(prev => ({ ...prev, extracting: true }));
        setError(null);
        setExtractedText('');

        try {
            const textPromises = uploadedFiles.map(file => new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.onload = async (e) => {
                    try {
                        const base64ImageData = e.target.result.split(',')[1];
                        const prompt = "Extract all text from this image. Return only the raw text, without any introductory phrases, comments, or formatting.";
                        const payload = { contents: [{ role: "user", parts: [ { text: prompt }, { inlineData: { mimeType: file.type, data: base64ImageData } } ] }] };
                        const apiKey = ""; 
                        const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
                        const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                        if (!response.ok) throw new Error(`API Error: ${response.statusText}`);
                        const result = await response.json();
                        if (result.candidates?.[0]?.content?.parts?.[0]?.text) {
                            resolve(result.candidates[0].content.parts[0].text);
                        } else {
                            resolve(`[Could not extract text from ${file.name}]`);
                        }
                    } catch (err) { reject(err); }
                };
                reader.onerror = () => reject(new Error(`FileReader error for ${file.name}`));
                reader.readAsDataURL(file);
            }));
            
            const texts = await Promise.all(textPromises);
            setExtractedText(texts.join('\n\n---\n\n'));
            goTo('textPreview');
        } catch (err) {
            setError(`Failed to extract text. Please try again. Details: ${err.message}`);
        } finally {
            setLoading(prev => ({ ...prev, extracting: false }));
        }
    }, [loading.extracting]);
    
    const handleGenerateQuiz = async () => {
        if (!extractedText || extractedText.trim().length < 20) {
            setError("The extracted text is too short to generate a meaningful quiz.");
            return;
        }
        setLoading(prev => ({ ...prev, generating: true }));
        setError(null);
        setQuiz([]);
        
        try {
            const prompt = `Based on the following text, generate exactly ${quizSettings.numQuestions} multiple-choice questions (MCQs). For each question, provide: the question, exactly 4 options, and the correct answer. Also provide a detailed explanation of 40-60 words. In the explanation, identify and wrap the most important keywords or concepts in Markdown-style asterisks for bolding (e.g., **keyword**). The explanation should directly clarify the topic and provide richer context. ${quizSettings.useInternet ? ' You can also use general knowledge to enrich the explanation.' : ''} Here is the text: --- ${extractedText} ---`;
            
            const payload = {
                contents: [{ role: "user", parts: [{ text: prompt }] }],
                generationConfig: { responseMimeType: "application/json",
                    responseSchema: { type: 'ARRAY', items: { type: 'OBJECT', properties: { question: { type: 'STRING' }, options: { type: 'ARRAY', items: { type: 'STRING' } }, correctAnswer: { type: 'STRING' }, explanation: { type: 'STRING' } }, required: ['question', 'options', 'correctAnswer', 'explanation'] } }
                }
            };
            
            const apiKey = "";
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
            const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });

            if (!response.ok) throw new Error(`API Error: ${response.statusText} (${response.status})`);
            
            const result = await response.json();
            if (result.candidates?.[0]?.content?.parts?.[0]?.text) {
                const quizData = JSON.parse(result.candidates[0].content.parts[0].text);
                const processedQuiz = quizData.slice(0, quizSettings.numQuestions).map((q, index) => ({
                    ...q, id: `${Date.now()}-${index}`, options: [...q.options].sort(() => Math.random() - 0.5), userAnswer: null,
                }));
                setQuiz(processedQuiz);
                goTo('quiz');
            } else {
                throw new Error("The API returned an unexpected response structure.");
            }
        } catch (err) {
             setError(`Failed to generate quiz. The AI may have had trouble with the text. Please try again. Details: ${err.message}`);
        } finally {
            setLoading(prev => ({ ...prev, generating: false }));
        }
    };

    const handleFinishQuiz = (finalQuiz) => {
        const score = finalQuiz.reduce((acc, q) => (q.userAnswer === q.correctAnswer ? acc + 1 : acc), 0);
        setQuizHistory(prev => [{ id: Date.now(), date: new Date().toISOString(), quiz: finalQuiz, score: score, total: finalQuiz.length }, ...prev]);
        setQuiz(finalQuiz);
        goTo('results');
    };
    
    const handleDownload = (quizData, score, percentage) => {
        const QuizPDF = `
        <html><head><title>Quiz Results</title><script src="https://cdn.tailwindcss.com"></script><style>body { font-family: sans-serif; -webkit-print-color-adjust: exact; } .question-block { page-break-inside: avoid; margin-bottom: 1.5rem; border-radius: 0.5rem; border: 1px solid #e2e8f0; padding: 1rem; }</style></head>
        <body class="p-8 bg-white"><h1 class="text-3xl font-bold mb-2 text-gray-900">Quiz Results</h1><h2 class="text-2xl font-bold mb-6 text-gray-800">Score: ${score}/${quizData.length} (${percentage}%)</h2>
            ${quizData.map((q, i) => `<div class="question-block"><p class="text-lg font-semibold mb-2 text-gray-900">${i + 1}. ${q.question}</p><p class="font-bold text-sm text-gray-700">Correct Answer: <span class="text-green-600">${q.correctAnswer}</span></p>${q.userAnswer ? `<p class="font-bold text-sm text-gray-700">Your Answer: <span class="${q.userAnswer === q.correctAnswer ? 'text-green-600' : 'text-red-600'}">${q.userAnswer}</span></p>` : ''}<p class="mt-3 text-gray-700 text-sm"><span class="font-semibold">Explanation:</span> ${renderExplanationForHTML(q.explanation)}</p></div>`).join('')}
        </body></html>`;
        
        const blob = new Blob([QuizPDF], { type: 'text/html' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a'); a.href = url; a.target = '_blank'; document.body.appendChild(a); a.click(); document.body.removeChild(a); URL.revokeObjectURL(url);
    };

    const renderPage = () => {
        switch (currentPage) {
            case 'uploader': return <ImageUploader files={files} setFiles={setFiles} onUpload={handleExtractText} loading={loading.extracting} />;
            case 'textPreview': return <TextPreview text={extractedText} setText={setExtractedText} onNext={handleGenerateQuiz} loading={loading.generating} settings={quizSettings} setSettings={setQuizSettings} />;
            case 'quiz': return <QuizView quiz={quiz} setQuiz={setQuiz} onFinish={handleFinishQuiz} settings={quizSettings} />;
            case 'results': return <ResultsPage quiz={quiz} onRestart={() => setCurrentPage('uploader') } onDownload={handleDownload} />;
            case 'history': return <HistoryPage history={quizHistory} onRestart={() => goTo('uploader')} onDownload={handleDownload} />;
            case 'landing':
            default: return <LandingPage onStart={() => goTo('uploader')} />;
        }
    };
    
    return (
        <div className="bg-slate-100 dark:bg-slate-900 min-h-screen font-sans text-slate-800 dark:text-slate-200 transition-colors duration-300">
             {currentPage === 'landing' ? renderPage() : (
                <div className="flex">
                    <Sidebar theme={theme} setTheme={setTheme} goTo={goTo} />
                    <div className="flex-1 transition-all duration-300 md:ml-64">
                        <div className="p-4 sm:p-6 md:p-8">
                            <main className="bg-white dark:bg-slate-800/80 backdrop-blur-sm p-4 sm:p-6 md:p-8 rounded-2xl shadow-lg border border-slate-200 dark:border-slate-700">
                                {error && <ErrorDisplay message={error} onDismiss={() => setError(null)} />}
                                {renderPage()}
                            </main>
                            <footer className="text-center mt-8 text-sm text-slate-500 dark:text-slate-400"><p>Powered by Google Gemini.</p></footer>
                        </div>
                    </div>
                </div>
            )}
        </div>
    );
};

export default App;
