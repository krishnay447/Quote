<head>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/heic2any/0.0.4/heic2any.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; transition: background 0.5s ease; }      
        .quote-item {
            background: rgba(255, 255, 255, 0.03);
            border: 1px solid rgba(255, 255, 255, 0.1);
            transition: all 0.3s ease;
        }
        .quote-item:hover {
            border-color: #6366f1;
            background: rgba(99, 102, 241, 0.05);
            transform: translateY(-2px);
        }
    </style>
</head>
<body class="bg-slate-950 text-slate-200 min-h-screen p-4 md:p-8">

    <div class="max-w-6xl mx-auto grid grid-cols-1 lg:grid-cols-2 gap-10">
        
        <div class="space-y-6 bg-slate-900/60 p-6 md:p-8 rounded-3xl border border-slate-800 shadow-2xl">

            <header>
                <h1 class="text-3xl font-extrabold text-white tracking-tight">SoulScript <span class="text-indigo-500">AI</span></h1>
                <p class="text-slate-500 text-xs mt-1 uppercase tracking-widest">Image to 3-Quote Generator</p>
            </header>

            <div class="space-y-2">
                <label class="text-xs font-bold text-slate-500 uppercase tracking-tighter">1. Upload Full Image</label>
                <div id="dropzone" class="relative min-h-[350px] w-full border-2 border-dashed border-slate-700 rounded-2xl flex items-center justify-center bg-black/40 overflow-hidden hover:border-indigo-500 transition-all cursor-pointer">
                    <img id="previewImg" class="hidden w-full h-full max-h-[500px] object-contain z-10 p-2" />
                    
                    <div id="uploadUI" class="text-center">
                        <span class="text-4xl opacity-50">🖼️</span>
                        <p class="text-sm mt-3 text-slate-400">Tap to upload your photo</p>
                        <p class="text-[10px] text-slate-600 mt-1 uppercase">JPG, PNG, WebP, HEIC</p>
                    </div>
                    <input type="file" id="imageInput" accept="image/*,.heic,.heif" class="absolute inset-0 opacity-0 cursor-pointer z-20">
                </div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                    <label class="text-xs font-bold text-slate-500 uppercase block mb-2 tracking-tighter">2. Choose Mood</label>
                    <select id="moodSelect" class="w-full bg-slate-800 border border-slate-700 rounded-xl p-3 text-sm outline-none focus:ring-2 focus:ring-indigo-500 transition-all appearance-none cursor-pointer">
                        <option value="Any">✨ Any / Surprise Me</option>
                        <option value="Friendship">🤝 Friendship & Unity</option>
                        <option value="Devotion">🙏 Devotion & Prayer</option>
                        <option value="Romantic">❤️ Romantic & Deep</option>
                        <option value="Motivational">🚀 Motivational & Power</option>
                        <option value="Nature">🌲 Peace & Nature</option>
                        <option value="Travel">🌍 Wanderlust & Adventure</option>
                        <option value="Success">🏆 Ambition & Hustle</option>
                        <option value="Sad">🌧️ Melancholy & Reflection</option>
                        <option value="Birthday">🎂 Celebration & Joy</option>
                        <option value="Attitude">😎 Attitude & Confidence</option>
                        <option value="Wisdom">📖 Wisdom & Lessons</option>
                    </select>
                </div>
                <div>
                    <label class="text-xs font-bold text-slate-500 uppercase block mb-2 tracking-tighter">3. Language</label>
                    <select id="langSelect" class="w-full bg-slate-800 border border-slate-700 rounded-xl p-3 text-sm outline-none focus:ring-2 focus:ring-indigo-500 transition-all cursor-pointer">
                        <option value="English">English</option>
                        <option value="Hindi">Hindi (हिंदी)</option>
                        <option value="Punjabi">Punjabi (ਪੰਜਾਬੀ)</option>
                    </select>
                </div>
            </div>

            <div>
                <label class="text-xs font-bold text-slate-500 uppercase block mb-2 tracking-tighter">4. Personal Context</label>
                <textarea id="userContext" placeholder="Who is in the photo? What is the special occasion?" class="w-full bg-slate-800 border border-slate-700 rounded-xl p-3 text-sm h-24 outline-none focus:ring-2 focus:ring-indigo-500 resize-none font-light transition-all"></textarea>
            </div>

            <button onclick="generateQuotes()" id="genBtn" class="w-full bg-indigo-600 hover:bg-indigo-500 text-white font-bold py-4 rounded-2xl transition-all shadow-lg active:scale-[0.98] flex items-center justify-center gap-2">
                Generate 3 Quotes <span id="btnIcon">✨</span>
            </button>
        </div>

        <div class="flex flex-col">
            <div id="resultsArea" class="hidden space-y-4">
                <div class="flex items-center justify-between mb-4">
                    <h3 class="text-slate-500 text-[10px] font-bold uppercase tracking-[0.2em]">Generated Options</h3>
                    <button onclick="generateQuotes()" class="text-indigo-400 text-[10px] uppercase font-bold tracking-widest hover:text-white transition-colors">🔄 New Batch</button>
                </div>
                <div id="quotesContainer" class="space-y-4">
                    </div>
            </div>

            <div id="placeholderUI" class="flex-grow flex flex-col items-center justify-center text-center p-10 border-2 border-slate-800 border-dashed rounded-3xl min-h-[400px]">
                <div id="loader" class="hidden h-12 w-12 border-4 border-indigo-600 border-t-transparent rounded-full animate-spin mb-4"></div>
                <p id="placeholderText" class="text-slate-600 text-sm italic max-w-[250px]">Upload a photo and choose a mood to see the AI magic.</p>
            </div>
        </div>
    </div>

    <script>
        // Image Processing Logic
        document.getElementById('imageInput').addEventListener('change', async function(e) {
            let file = e.target.files[0];
            if (!file) return;

            const preview = document.getElementById('previewImg');
            const uploadUI = document.getElementById('uploadUI');

            // Handle iPhone format
            if (file.name.toLowerCase().endsWith(".heic") || file.name.toLowerCase().endsWith(".heif")) {
                uploadUI.innerHTML = "<p class='text-xs text-indigo-400 animate-pulse font-bold tracking-widest'>Converting HEIC...</p>";
                file = await heic2any({ blob: file, toType: "image/jpeg" });
            }

            const url = URL.createObjectURL(file);
            preview.src = url;
            preview.classList.remove('hidden');
            uploadUI.classList.add('hidden');
        });

        async function generateQuotes() {
            if (!document.getElementById('imageInput').files[0]) return alert("Please upload an image first!");

            const btn = document.getElementById('genBtn');
            const loader = document.getElementById('loader');
            const placeholderUI = document.getElementById('placeholderUI');
            const resultsArea = document.getElementById('resultsArea');
            const container = document.getElementById('quotesContainer');

            btn.disabled = true;
            loader.classList.remove('hidden');
            document.getElementById('placeholderText').classList.add('hidden');
            resultsArea.classList.add('hidden');

            // AI Simulation Logic
            setTimeout(() => {
                const lang = document.getElementById('langSelect').value;
                const mood = document.getElementById('moodSelect').value;
                container.innerHTML = ''; 

                // Extended Database reflecting the 12 Moods
                const quotesDatabase = {
                    English: [
                        "The soul speaks through the eyes of a moment.",
                        "Every path taken has its own beautiful destination.",
                        "Simplicity is the ultimate form of elegance.",
                        "Your energy defines your reality, keep it radiant.",
                        "Life is a collection of chapters, make this one a masterpiece.",
                        "The greatest journey is the one that brings you home.",
                        "Authenticity is the most magnetic trait one can possess."
                    ],
                    Hindi: [
                        "वक्त ठहर सा जाता है, जब यादें खूबसूरत होती हैं।",
                        "सपनों की उड़ान भरने के लिए पंख नहीं, हौसलों की ज़रूरत होती है।",
                        "मंज़िल उन्हें मिलती है जिनके सपनों में जान होती है।",
                        "सादगी से बड़ा कोई श्रृंगार नहीं होता।",
                        "रिश्ते दिल से जुड़ते हैं, कागजों से नहीं।"
                    ],
                    Punjabi: [
                        "ਖੁਸ਼ੀਆਂ ਦਾ ਕੋਈ ਮੁੱਲ ਨਹੀਂ ਹੁੰਦਾ, ਇਹ ਸਿਰਫ਼ ਮਹਿਸੂਸ ਕੀਤੀਆਂ ਜਾਂਦੀਆਂ ਨੇ।",
                        "ਰੱਬ ਦੀ ਰਜ਼ਾ ਵਿੱਚ ਰਹਿਣਾ ਹੀ ਸਭ ਤੋਂ ਵੱਡਾ ਸਕੂਨ ਹੈ।",
                        "ਯਾਰੀਆਂ ਦੇ ਰੰਗ ਕਦੇ ਫਿੱਕੇ ਨਹੀਂ ਪੈਂਦੇ।",
                        "ਸਬਰ ਰੱਖ ਮਨਾਂ, ਤੇਰਾ ਵਕਤ ਵੀ ਜ਼ਰੂਰ ਆਵੇਗਾ।",
                        "ਜ਼ਿੰਦਗੀ ਦੇ ਰਾਹਾਂ ਵਿੱਚ ਹਮੇਸ਼ਾ ਮੁਸਕਰਾਉਂਦੇ ਰਹੋ।"
                    ]
                };

                // Pick 3 random quotes
                const selection = [...quotesDatabase[lang]].sort(() => 0.5 - Math.random()).slice(0, 3);

                selection.forEach((text) => {
                    const div = document.createElement('div');
                    div.className = "quote-item p-6 rounded-2xl flex items-center justify-between gap-4 group";
                    div.innerHTML = `
                        <p class="text-sm md:text-base text-slate-100 font-light leading-relaxed tracking-wide">"${text}"</p>
                        <button onclick="copyToClipboard('${text}', this)" class="shrink-0 p-2 bg-slate-800 hover:bg-indigo-600 rounded-lg text-xs transition-all opacity-100 lg:opacity-0 group-hover:opacity-100">📋 Copy</button>
                    `;
                    container.appendChild(div);
                });

                loader.classList.add('hidden');
                placeholderUI.classList.add('hidden');
                resultsArea.classList.remove('hidden');
                btn.disabled = false;
            }, 1500);
        }

        function copyToClipboard(text, btn) {
            navigator.clipboard.writeText(text);
            const original = btn.innerText;
            btn.innerText = "✅ Saved";
            setTimeout(() => btn.innerText = original, 2000);
        }
    </script>
</body>
</html>
