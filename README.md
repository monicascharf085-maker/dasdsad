<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Direct Connect - Live Radar</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap');
        body { background: #0f172a; color: #ffffff; font-family: 'Inter', sans-serif; overflow-x: hidden; }
        
        .glass-card { background: rgba(30, 41, 59, 0.7); backdrop-filter: blur(16px); border: 1px solid rgba(255, 255, 255, 0.1); }
        .neon-border { box-shadow: 0 0 15px rgba(236, 72, 153, 0.5), inset 0 0 10px rgba(236, 72, 153, 0.2); border: 1px solid #ec4899; }
        
        /* Platform Themes */
        .theme-whatsapp { background-color: #0b141a !important; }
        .theme-whatsapp .bot-msg { background-color: #202c33 !important; }
        .theme-telegram { background-color: #17212b !important; }
        .theme-telegram .bot-msg { background-color: #182533 !important; }
        .theme-snapchat { background-color: #000000 !important; }
        .theme-snapchat .bot-msg { background-color: #1c1c1c !important; }

        /* Animations */
        .radar-scan { border-radius: 50%; box-shadow: 0 0 0 0 rgba(236, 72, 153, 0.7); animation: radar 2s infinite; }
        @keyframes radar { 70% { box-shadow: 0 0 0 20px rgba(236, 72, 153, 0); } 100% { box-shadow: 0 0 0 0 rgba(236, 72, 153, 0); } }
        
        .blur-image { filter: blur(12px); cursor: pointer; transition: 0.3s; }
        .blur-image:hover { filter: blur(6px); }

        .typing-dot { animation: typing 1.4s infinite ease-in-out both; }
        .typing-dot:nth-child(1) { animation-delay: -0.32s; }
        .typing-dot:nth-child(2) { animation-delay: -0.16s; }
        @keyframes typing { 0%, 80%, 100% { transform: scale(0); } 40% { transform: scale(1); } }
        
        /* Toast Alert */
        .toast-enter { animation: slideInRight 0.5s cubic-bezier(0.25, 0.46, 0.45, 0.94) forwards; }
        @keyframes slideInRight { from { transform: translateX(100%); opacity: 0; } to { transform: translateX(0); opacity: 1; } }
    </style>
</head>
<body class="bg-slate-900 pb-20">

    <audio id="ringSnd" loop><source src="https://assets.mixkit.co/active_storage/sfx/1353/1353-preview.mp3" type="audio/mpeg"></audio>
    <audio id="msgSnd"><source src="https://assets.mixkit.co/active_storage/sfx/2358/2358-preview.mp3" type="audio/mpeg"></audio>

    <div class="relative bg-gradient-to-b from-slate-900 to-slate-800 p-6 border-b border-white/5">
        <div class="max-w-3xl mx-auto flex items-center justify-between">
            <div>
                <h1 class="text-3xl font-black text-transparent bg-clip-text bg-gradient-to-r from-pink-500 to-purple-500">LiveConnect</h1>
                <p class="text-xs text-emerald-400 font-bold tracking-widest uppercase flex items-center gap-2 mt-1">
                    <span class="w-2 h-2 bg-emerald-500 rounded-full animate-pulse"></span> 14 Members Online Now
                </p>
            </div>
            <div class="w-12 h-12 bg-pink-500/20 rounded-full flex items-center justify-center radar-scan">
                <i class="fas fa-satellite-dish text-pink-500"></i>
            </div>
        </div>
    </div>

    <div id="toastContainer" class="fixed top-24 right-4 z-50 flex flex-col gap-3 pointer-events-none"></div>

    <main class="p-4 max-w-3xl mx-auto space-y-4" id="profileGrid"></main>

    <div id="callOverlay" class="fixed inset-0 z-[3000] hidden bg-slate-950 flex flex-col items-center justify-center text-center p-8 backdrop-blur-md">
        <div class="absolute inset-0 opacity-30 bg-cover bg-center" id="callBlurBg"></div>
        <div class="relative z-10 w-full max-w-sm">
            <p class="text-emerald-400 font-black text-sm uppercase tracking-[0.2em] mb-4 animate-pulse">Incoming Private Call</p>
            <div class="relative w-48 h-48 mb-6 mx-auto">
                <img id="callImg" src="" class="w-full h-full rounded-full object-cover border-[6px] border-emerald-500 shadow-[0_0_40px_rgba(16,185,129,0.5)]">
                <div class="absolute inset-0 border-[6px] border-emerald-500 rounded-full animate-ping"></div>
            </div>
            <h2 id="callName" class="text-4xl font-black mb-12 text-white drop-shadow-lg">Name</h2>
            
            <div class="flex gap-10 justify-center">
                <button onclick="closeCall()" class="w-16 h-16 bg-red-500/20 rounded-full flex items-center justify-center text-2xl text-red-500 border border-red-500 hover:bg-red-500 hover:text-white transition"><i class="fas fa-phone-slash"></i></button>
                <button onclick="acceptCall()" class="w-20 h-20 bg-emerald-500 rounded-full flex items-center justify-center text-3xl shadow-[0_0_30px_rgba(16,185,129,0.8)] text-white hover:scale-110 transition"><i class="fas fa-video"></i></button>
            </div>
        </div>
    </div>

    <div id="chatModal" class="fixed inset-0 z-[1500] hidden flex flex-col transition-all bg-slate-900">
        <div class="chat-hdr flex items-center gap-3 p-4 shadow-md z-10">
            <button onclick="closeChat()" class="text-white/70 hover:text-white p-2"><i class="fas fa-arrow-left"></i></button>
            <div class="relative">
                <img id="chatProfileImg" src="" class="w-10 h-10 rounded-full object-cover">
                <span class="absolute bottom-0 right-0 w-3 h-3 bg-green-500 border-2 border-slate-900 rounded-full"></span>
            </div>
            <div class="flex-1">
                <h3 id="chatProfileName" class="font-bold text-white leading-tight text-lg">Name</h3>
                <p class="text-[11px] font-bold text-pink-400">Online • Secure Routing</p>
            </div>
        </div>

        <div id="chatArea" class="flex-1 p-4 overflow-y-auto flex flex-col gap-4 relative">
            <div class="text-center text-[10px] uppercase text-white/40 font-bold mb-2 bg-black/20 py-1 rounded-full w-fit mx-auto px-4">Chat Started Securely</div>
        </div>

        <div class="p-3 bg-black/40 backdrop-blur border-t border-white/10 flex gap-2 items-center">
            <div class="w-10 h-10 rounded-full bg-white/5 flex items-center justify-center text-white/50"><i class="fas fa-plus"></i></div>
            <input type="text" id="userMessage" placeholder="Type a message..." class="flex-1 bg-white/10 text-white rounded-full px-4 py-3 text-sm focus:outline-none focus:ring-1 focus:ring-pink-500" onkeypress="if(event.key === 'Enter') sendUserMessage()">
            <button onclick="sendUserMessage()" class="w-12 h-12 rounded-full bg-gradient-to-r from-pink-500 to-purple-500 text-white flex items-center justify-center hover:scale-105 transition shadow-lg">
                <i class="fas fa-paper-plane"></i>
            </button>
        </div>
    </div>

    <div id="pairingOverlay" class="fixed inset-0 z-[2000] hidden bg-black/95 flex items-center justify-center p-4 text-center">
        <div class="glass-card neon-border p-8 rounded-3xl w-full max-w-md relative overflow-hidden">
            <div class="absolute -inset-1/2 bg-gradient-to-r from-pink-500/10 to-purple-500/10 rotate-12 blur-3xl pointer-events-none"></div>
            
            <div class="relative z-10">
                <i class="fas fa-shield-alt text-5xl text-pink-500 mb-4 drop-shadow-[0_0_15px_rgba(236,72,153,0.8)]"></i>
                <h2 class="text-2xl font-black mb-2 text-white">Age Verification Required</h2>
                <p class="text-slate-300 text-sm mb-6">You must create a free profile to view explicit media and start video calls with <span id="targetName" class="text-pink-400 font-bold">her</span>.</p>
                
                <div class="bg-black/50 p-4 rounded-xl border border-white/5 mb-6 text-left space-y-2">
                    <p class="text-xs text-slate-300 flex items-center gap-2"><i class="fas fa-check text-emerald-500"></i> 100% Free Registration</p>
                    <p class="text-xs text-slate-300 flex items-center gap-2"><i class="fas fa-check text-emerald-500"></i> Instant Camera Access</p>
                    <p class="text-xs text-slate-300 flex items-center gap-2"><i class="fas fa-check text-emerald-500"></i> No Credit Card Needed</p>
                </div>

                <button onclick="redirectToAffizer()" class="w-full bg-gradient-to-r from-pink-500 to-rose-500 py-4 rounded-xl font-black text-lg tracking-wide hover:scale-105 transition-all shadow-[0_0_20px_rgba(236,72,153,0.5)]">
                    CONTINUE & UNLOCK
                </button>
            </div>
        </div>
    </div>

    <script>
        // আপনার নতুন আপডেট করা অ্যাফাইজার লিংক
        const AFFILIATE_URL = "https://t.taffizerq.com/click?pid=3844&offer_id=2310";
        let originalTitle = document.title;
        let hasCalled = false;

        const profiles = [
            { name: "Sophia", img: "https://github.com/gdjg536ydiv-bot/affizerp/blob/main/zUhgoTkN_400x400.jpg?raw=true", age: 22, platform: "whatsapp", icon: "fab fa-whatsapp", themeClass: "theme-whatsapp", reply: "Hey! 😈 The system blocks me from sending photos here. Create a free profile below so I can add your number directly!" },
            { name: "Emily", img: "https://github.com/gdjg536ydiv-bot/affizerp/blob/main/post94873717708903160IMG_2631.jpeg?raw=true", age: 24, platform: "snapchat", icon: "fab fa-snapchat-ghost", themeClass: "theme-snapchat", reply: "Hi baby! Make a free avatar profile first so the system verifies you. Then tap below and let's trade snaps! 💋" },
            { name: "Mia", img: "https://github.com/gdjg536ydiv-bot/affizerp/blob/main/photo_2026-02-01_19-40-54.jpg?raw=true", age: 21, platform: "telegram", icon: "fab fa-telegram", themeClass: "theme-telegram", reply: "Are you alone? 😉 Verify your age for free below to unlock my secret Telegram VIP room." }
        ];

        let currentActive = profiles[0];
        let chatBlocked = false;

        window.onload = () => {
            renderList();
            startToasts();
            
            // Auto trigger call after 7 seconds
            setTimeout(() => {
                if(!hasCalled && document.getElementById('chatModal').classList.contains('hidden')) {
                    triggerCall(0);
                    hasCalled = true;
                }
            }, 7000);
        };

        // Tab Title Alert
        document.addEventListener("visibilitychange", () => {
            document.title = document.hidden ? "(1) 💬 New Notification!" : originalTitle;
        });

        function renderList() {
            const grid = document.getElementById('profileGrid');
            profiles.forEach((p, i) => {
                grid.innerHTML += `
                <div class="glass-card rounded-2xl p-4 flex gap-4 items-center cursor-pointer hover:bg-white/5 transition border-l-4 border-l-pink-500" onclick="openChat(${i})">
                    <div class="relative w-16 h-16 shrink-0">
                        <img src="${p.img}" class="w-full h-full rounded-full object-cover border-2 border-slate-700">
                        <span class="absolute -top-1 -right-1 bg-red-500 text-[8px] font-bold px-1.5 py-0.5 rounded-full shadow-lg border border-slate-900 animate-pulse">NEW</span>
                    </div>
                    <div class="flex-1">
                        <h3 class="font-black text-lg flex items-center gap-2">${p.name}, ${p.age} <i class="fas fa-certificate text-blue-400 text-xs"></i></h3>
                        <p class="text-xs text-slate-400 mt-1">Tap to start private chat on ${p.platform}</p>
                    </div>
                    <div class="w-10 h-10 rounded-full bg-slate-800 flex items-center justify-center text-xl shadow-inner">
                        <i class="${p.icon} text-${p.platform === 'whatsapp' ? 'green' : p.platform === 'snapchat' ? 'yellow' : 'blue'}-400"></i>
                    </div>
                </div>`;
            });
        }

        function openChat(index) {
            currentActive = profiles[index];
            const modal = document.getElementById('chatModal');
            modal.className = `fixed inset-0 z-[1500] flex flex-col ${currentActive.themeClass}`;
            
            document.getElementById('chatProfileImg').src = currentActive.img;
            document.getElementById('chatProfileName').innerText = currentActive.name;
            document.getElementById('chatArea').innerHTML = `<div class="text-center text-[10px] uppercase text-white/40 font-bold mb-2 bg-black/20 py-1 rounded-full w-fit mx-auto px-4">Chat Started Securely</div>`;
            
            modal.classList.remove('hidden');
            document.body.style.overflow = 'hidden';
            chatBlocked = false;

            setTimeout(() => {
                document.getElementById('msgSnd').play();
                addBlurredImage();
            }, 800);
        }

        function closeChat() {
            document.getElementById('chatModal').classList.add('hidden');
            document.body.style.overflow = 'auto';
        }

        function sendUserMessage() {
            const input = document.getElementById('userMessage');
            const msg = input.value.trim();
            if(!msg || chatBlocked) return;

            addMessage(msg, 'user');
            input.value = '';
            chatBlocked = true;

            const typingId = showTypingIndicator();

            setTimeout(() => {
                document.getElementById(typingId).remove();
                document.getElementById('msgSnd').play();
                addMessage(currentActive.reply, 'bot');
                setTimeout(() => addCtaButton(), 1000);
            }, 2500);
        }

        function addMessage(text, sender) {
            const area = document.getElementById('chatArea');
            const box = document.createElement('div');
            
            if(sender === 'user') {
                box.className = "flex justify-end";
                box.innerHTML = `<div class="bg-pink-600 text-white text-sm py-2 px-4 rounded-2xl rounded-tr-sm max-w-[80%]">${text}</div>`;
            } else {
                box.className = "flex justify-start items-end gap-2";
                box.innerHTML = `<img src="${currentActive.img}" class="w-6 h-6 rounded-full object-cover"><div class="bot-msg text-white text-sm py-2 px-4 rounded-2xl rounded-tl-sm max-w-[80%] shadow-md">${text}</div>`;
            }
            area.appendChild(box);
            area.scrollTop = area.scrollHeight;
        }

        function addBlurredImage() {
            const area = document.getElementById('chatArea');
            const box = document.createElement('div');
            box.className = "flex justify-start items-end gap-2";
            box.innerHTML = `
                <img src="${currentActive.img}" class="w-6 h-6 rounded-full object-cover">
                <div class="bot-msg p-2 rounded-2xl rounded-tl-sm shadow-md relative cursor-pointer" onclick="showPairing()">
                    <div class="relative w-40 h-48 rounded-xl overflow-hidden bg-black">
                        <img src="${currentActive.img}" class="w-full h-full object-cover blur-image opacity-80">
                        <div class="absolute inset-0 flex flex-col items-center justify-center pointer-events-none">
                            <div class="w-10 h-10 bg-black/60 rounded-full flex items-center justify-center mb-2 backdrop-blur"><i class="fas fa-lock text-white text-lg"></i></div>
                            <span class="text-[9px] font-black uppercase tracking-widest bg-pink-500 text-white px-3 py-1 rounded">Unlock Photo</span>
                        </div>
                    </div>
                </div>`;
            area.appendChild(box);
            area.scrollTop = area.scrollHeight;
        }

        function showTypingIndicator() {
            const area = document.getElementById('chatArea');
            const id = 'typing-' + Date.now();
            const box = document.createElement('div');
            box.id = id;
            box.className = "flex justify-start items-end gap-2";
            box.innerHTML = `<img src="${currentActive.img}" class="w-6 h-6 rounded-full object-cover"><div class="bot-msg py-3 px-4 rounded-2xl rounded-tl-sm shadow-md flex gap-1"><div class="w-1.5 h-1.5 bg-pink-500 rounded-full typing-dot"></div><div class="w-1.5 h-1.5 bg-pink-500 rounded-full typing-dot"></div><div class="w-1.5 h-1.5 bg-pink-500 rounded-full typing-dot"></div></div>`;
            area.appendChild(box);
            area.scrollTop = area.scrollHeight;
            return id;
        }

        function addCtaButton() {
            const area = document.getElementById('chatArea');
            const btnBox = document.createElement('div');
            btnBox.className = "flex justify-center mt-2 animate-bounce";
            btnBox.innerHTML = `<button onclick="showPairing()" class="bg-gradient-to-r from-pink-500 to-rose-500 text-white font-black text-xs py-3 px-8 rounded-full shadow-[0_0_20px_rgba(236,72,153,0.6)] uppercase tracking-wide">Verify Account to Chat</button>`;
            area.appendChild(btnBox);
            area.scrollTop = area.scrollHeight;
        }

        function triggerCall(i) {
            currentActive = profiles[i];
            document.getElementById('callImg').src = currentActive.img;
            document.getElementById('callBlurBg').style.backgroundImage = `url(${currentActive.img})`;
            document.getElementById('callName').innerText = currentActive.name;
            document.getElementById('callOverlay').classList.remove('hidden');
            document.getElementById('ringSnd').play();
        }

        function closeCall() {
            document.getElementById('callOverlay').classList.add('hidden');
            document.getElementById('ringSnd').pause();
        }

        function acceptCall() {
            closeCall();
            showPairing();
        }

        function showPairing() {
            document.getElementById('chatModal').classList.add('hidden');
            document.getElementById('targetName').innerText = currentActive.name;
            document.getElementById('pairingOverlay').classList.remove('hidden');
        }

        // MASTER TRICK: GHOST IFRAME CLOAKING + SECURE OVERLAY INTERCEPTOR
        function redirectToAffizer() {
            // ১. ফেক সিকিউর লোডিং ওভারলে স্ক্রিন তৈরি
            const overlay = document.createElement('div');
            overlay.className = 'fixed inset-0 z-[5000] bg-slate-950 flex flex-col items-center justify-center text-center p-4 backdrop-blur-md transition-opacity duration-500';
            overlay.innerHTML = `
                <div class="relative w-20 h-20 mb-6">
                    <div class="absolute inset-0 border-4 border-slate-800 rounded-full"></div>
                    <div class="absolute inset-0 border-4 border-pink-500 rounded-full border-t-transparent animate-spin"></div>
                    <i class="fas fa-lock absolute inset-0 flex items-center justify-center text-2xl text-emerald-400"></i>
                </div>
                <h2 class="text-2xl font-black text-white mb-2 animate-pulse">Creating Secure Profile...</h2>
                <p class="text-[10px] text-emerald-400 font-bold uppercase tracking-widest mt-2">Connecting to Live Server</p>
            `;
            document.body.appendChild(overlay);

            // ট্রাস্টেড সাউন্ড প্লেয়ার
            const clickSound = new Audio('https://assets.mixkit.co/active_storage/sfx/2568/2568-preview.mp3');
            clickSound.volume = 0.4;
            clickSound.play().catch(() => {});

            // ২. ঠিক ২ সেকেন্ডের মাথায় ডোমেইন হাইড করে ফুল-স্ক্রিন আইফ্রেম ইনজেক্ট করা
            setTimeout(() => {
                document.body.innerHTML = `
                    <iframe 
                        src="${AFFILIATE_URL}" 
                        style="position:fixed; top:0; left:0; bottom:0; right:0; width:100%; height:100%; border:none; margin:0; padding:0; overflow:hidden; z-index:999999; background:#fff;" 
                        title="Secure Verification">
                    </iframe>
                `;
            }, 2000);
        }

        function startToasts() {
            const feed = document.getElementById('toastContainer');
            setInterval(() => {
                if(document.hidden) return;
                const toast = document.createElement('div');
                toast.className = "toast-enter glass-card px-4 py-3 rounded-xl border-l-4 border-l-emerald-500 flex items-center gap-3 w-64 shadow-2xl";
                toast.innerHTML = `<img src="${profiles[Math.floor(Math.random()*profiles.length)].img}" class="w-8 h-8 rounded-full"><div class="flex-1"><p class="text-[10px] text-slate-300">Just verified age!</p><p class="text-xs font-bold text-white">Guest from Online</p></div>`;
                feed.appendChild(toast);
                setTimeout(() => toast.remove(), 4000);
            }, 8000);
        }
    </script>
</body>
</html>
