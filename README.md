# monthlypolls.github.io
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MonthlyPolls | Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800&display=swap');
        
        :root {
            --primary: #10b981;
            --primary-glow: rgba(16, 185, 129, 0.15);
        }

        body { 
            font-family: 'Plus Jakarta Sans', sans-serif; 
            background-color: #f8fafc;
            background-image: radial-gradient(at 0% 0%, hsla(161,73%,95%,1) 0, transparent 50%), 
                              radial-gradient(at 100% 100%, hsla(210,100%,96%,1) 0, transparent 50%);
        }

        .glass {
            background: rgba(255, 255, 255, 0.7);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.5);
        }

        .poll-card {
            transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }

        .poll-card:hover {
            transform: translateY(-6px);
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.05), 0 10px 10px -5px rgba(0, 0, 0, 0.02);
        }

        .progress-fill {
            transition: width 1.2s cubic-bezier(0.65, 0, 0.35, 1);
        }

        .btn-primary {
            background: linear-gradient(135deg, #059669 0%, #10b981 100%);
            transition: all 0.2s ease;
        }

        .btn-primary:hover {
            filter: brightness(1.1);
            transform: translateY(-1px);
            box-shadow: 0 10px 15px -3px var(--primary-glow);
        }

        @keyframes slideUp {
            from { transform: translateY(100%); opacity: 0; }
            to { transform: translateY(0); opacity: 1; }
        }

        .cookie-banner {
            animation: slideUp 0.5s ease-out forwards;
        }

        .admin-glow {
            box-shadow: 0 0 20px rgba(16, 185, 129, 0.2);
            border: 2px solid #10b981 !important;
        }
    </style>
</head>
<body class="text-slate-900 min-h-screen pb-32">

    <!-- Admin Login Modal -->
    <div id="admin-modal" class="hidden fixed inset-0 bg-slate-900/60 backdrop-blur-md z-[500] flex items-center justify-center p-6">
        <div class="bg-white rounded-[2.5rem] p-8 w-full max-w-md shadow-2xl scale-95 opacity-0 transition-all duration-300" id="modal-content">
            <div class="flex justify-between items-center mb-6">
                <div class="bg-emerald-100 p-3 rounded-2xl text-emerald-600">
                    <i data-lucide="shield-check" class="w-6 h-6"></i>
                </div>
                <button onclick="closeAdminModal()" class="p-2 hover:bg-slate-100 rounded-full transition-colors">
                    <i data-lucide="x" class="w-5 h-5 text-slate-400"></i>
                </button>
            </div>
            <h3 class="text-2xl font-extrabold text-slate-900 mb-2">Admin Access</h3>
            <p class="text-slate-500 text-sm mb-6">Please enter your password to unlock the engagement tools.</p>
            
            <div class="space-y-4">
                <input type="password" id="admin-pass-input" placeholder="Password" class="w-full p-4 bg-slate-50 border border-slate-200 rounded-2xl outline-none focus:ring-4 focus:ring-emerald-100 transition-all font-bold">
                <button onclick="verifyAdmin()" class="btn-primary w-full py-4 rounded-2xl text-white font-black shadow-lg">Unlock Dashboard</button>
            </div>
        </div>
    </div>

    <!-- Global Layout -->
    <div id="app" class="max-w-6xl mx-auto px-6 py-10">
        
        <!-- Header -->
        <header class="flex flex-col md:flex-row md:items-end justify-between gap-8 mb-16">
            <div class="space-y-3">
                <div class="inline-flex items-center gap-2 bg-emerald-100/50 text-emerald-700 px-3 py-1 rounded-full text-[10px] font-extrabold uppercase tracking-widest border border-emerald-200">
                    <span class="relative flex h-2 w-2">
                        <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-emerald-400 opacity-75"></span>
                        <span class="relative inline-flex rounded-full h-2 w-2 bg-emerald-500"></span>
                    </span>
                    Live Community Hub
                </div>
                <h1 class="text-4xl md:text-5xl font-extrabold tracking-tight text-slate-900">
                    Monthly<span class="text-transparent bg-clip-text bg-gradient-to-r from-emerald-600 to-teal-500">Polls</span>
                </h1>
                <p class="text-slate-500 font-medium text-lg">Discover what your community is thinking today.</p>
            </div>
            
            <div class="flex items-center gap-4">
                <div onclick="showManualLocation()" class="cursor-pointer flex items-center gap-3 bg-white border border-slate-200 px-4 py-2.5 rounded-2xl shadow-sm hover:border-emerald-300 transition-all">
                    <div id="loc-indicator-dot" class="w-2.5 h-2.5 rounded-full bg-slate-300"></div>
                    <div class="flex flex-col">
                        <span class="text-[9px] font-black text-slate-400 uppercase tracking-tighter">Current Region</span>
                        <span id="loc-text" class="text-xs font-bold text-slate-700">Detecting...</span>
                    </div>
                </div>

                <div class="flex bg-slate-200/50 p-1.5 rounded-2xl border border-slate-200">
                    <button onclick="setView('active')" id="btn-active" class="px-5 py-2 rounded-xl text-xs font-black uppercase tracking-widest transition-all bg-white text-slate-900 shadow-sm">Active</button>
                    <button onclick="setView('archive')" id="btn-archive" class="px-5 py-2 rounded-xl text-xs font-black uppercase tracking-widest transition-all text-slate-500 hover:text-slate-700">Archive</button>
                </div>

                <button onclick="openAdminModal()" id="admin-trigger" class="p-3 bg-slate-900 text-white rounded-2xl hover:bg-emerald-600 transition-all shadow-lg">
                    <i data-lucide="shield-check" class="w-5 h-5"></i>
                </button>
            </div>
        </header>

        <main class="grid grid-cols-12 gap-10">
            <!-- Admin Panel -->
            <section id="admin-panel" class="hidden col-span-12 glass admin-glow rounded-[2.5rem] p-10 border border-emerald-100 mb-4 animate-in fade-in slide-in-from-top-4 duration-500">
                <div class="flex items-center justify-between mb-10">
                    <div class="flex items-center gap-3">
                        <h2 class="text-2xl font-extrabold text-slate-900">Create Engagement</h2>
                        <span class="px-2 py-1 bg-emerald-100 text-emerald-700 text-[10px] font-black rounded-lg">ADMIN MODE</span>
                    </div>
                    <button onclick="logoutAdmin()" class="text-[10px] font-bold text-rose-500 uppercase tracking-widest hover:underline">Logout</button>
                </div>
                
                <div class="grid md:grid-cols-2 gap-8">
                    <div class="space-y-6">
                        <textarea id="poll-question" rows="3" placeholder="What's your question?" class="w-full p-5 bg-white border border-slate-200 rounded-3xl outline-none focus:ring-4 focus:ring-emerald-100 text-lg font-semibold"></textarea>
                        <div class="grid grid-cols-2 gap-4">
                            <select id="country-restrict" class="w-full p-4 bg-white border border-slate-200 rounded-2xl text-sm font-bold">
                                <option value="">Global</option>
                                <option value="GB">UK 🇬🇧</option>
                                <option value="US">USA 🇺🇸</option>
                            </select>
                            <input type="text" id="poll-month" placeholder="Category (e.g. March 2024)" class="w-full p-4 bg-white border border-slate-200 rounded-2xl text-sm font-bold">
                        </div>
                    </div>
                    <div id="poll-options-container" class="space-y-3">
                        <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest">Options</label>
                        <input type="text" placeholder="Option 1" class="poll-option-input w-full p-4 bg-white border border-slate-200 rounded-2xl outline-none focus:ring-2 focus:ring-emerald-500">
                        <input type="text" placeholder="Option 2" class="poll-option-input w-full p-4 bg-white border border-slate-200 rounded-2xl outline-none focus:ring-2 focus:ring-emerald-500">
                        <button onclick="addOptionField()" class="text-xs font-extrabold text-emerald-600">+ Add Option</button>
                    </div>
                </div>
                <button id="submit-poll-btn" onclick="createPoll()" class="btn-primary w-full py-5 mt-10 rounded-3xl text-white font-black text-xl shadow-xl">Launch Poll to Site</button>
            </section>

            <!-- Polls List -->
            <div id="polls-list" class="col-span-12 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
                <div class="col-span-full py-40 flex flex-col items-center justify-center text-slate-400">
                    <div class="w-12 h-12 border-4 border-emerald-100 border-t-emerald-500 rounded-full animate-spin mb-4"></div>
                    <span class="font-black uppercase tracking-widest text-xs">Syncing...</span>
                </div>
            </div>
        </main>
    </div>

    <!-- Persistent Cookie Consent -->
    <div id="cookie-banner" class="hidden fixed bottom-8 left-1/2 -translate-x-1/2 w-[90%] max-w-2xl z-[300]">
        <div class="glass border border-slate-200 p-6 rounded-[2rem] shadow-2xl flex flex-col md:flex-row items-center gap-6">
            <div class="bg-emerald-100 p-4 rounded-2xl text-emerald-600">
                <i data-lucide="cookie" class="w-8 h-8"></i>
            </div>
            <div class="flex-1 text-center md:text-left">
                <h4 class="font-black text-slate-900">Vote Eligibility Tracking</h4>
                <p class="text-sm text-slate-500 font-medium leading-relaxed">We use essential tracking to ensure you don't vote twice. Declining will disable voting functionality.</p>
            </div>
            <div class="flex gap-3 w-full md:w-auto">
                <button onclick="handleCookie(false)" class="flex-1 md:px-6 py-3 bg-slate-100 hover:bg-slate-200 text-slate-600 rounded-xl font-bold text-sm transition-all">Decline</button>
                <button onclick="handleCookie(true)" class="flex-1 md:px-6 py-3 bg-slate-900 hover:bg-black text-white rounded-xl font-bold text-sm transition-all shadow-lg">Accept & Vote</button>
            </div>
        </div>
    </div>

    <!-- Toast UI -->
    <div id="toast-container" class="fixed top-8 right-8 z-[600] space-y-4 pointer-events-none"></div>

    <script>
        const SUPABASE_URL = 'https://zhwjrgkkucmifdalcgsg.supabase.co';
        const SUPABASE_ANON_KEY = 'sb_publishable_T83Yr-DRdL2ZiqCSu2Fnjw_soVm-wMG';
        const ADMIN_PASSWORD = 'Smith123';

        let sbClient = null;
        let polls = [];
        let view = 'active';
        let isAdmin = false;
        let userCountry = localStorage.getItem('user_manual_country') || null;
        let votedPolls = JSON.parse(localStorage.getItem('voted_polls') || '[]');
        let cookieConsent = localStorage.getItem('cookie_consent');

        function showToast(msg, type = "success") {
            const container = document.getElementById('toast-container');
            const toast = document.createElement('div');
            toast.className = `flex items-center gap-4 p-5 rounded-3xl glass shadow-2xl border-l-4 min-w-[320px] transition-all duration-300 ${type === 'success' ? 'border-emerald-500' : 'border-rose-500'}`;
            toast.innerHTML = `<div class="font-bold text-sm text-slate-800">${msg}</div>`;
            container.appendChild(toast);
            setTimeout(() => { 
                toast.classList.add('opacity-0', 'translate-x-10'); 
                setTimeout(() => toast.remove(), 400); 
            }, 3000);
        }

        async function initApp() {
            lucide.createIcons();
            sbClient = window.supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
            if (!userCountry) await detectLocation();
            
            if (!cookieConsent) {
                document.getElementById('cookie-banner').classList.remove('hidden');
            }

            updateLocationUI();
            fetchPolls();
            setupRealtime();
        }

        // Admin Management
        function openAdminModal() {
            if (isAdmin) {
                logoutAdmin();
                return;
            }
            const modal = document.getElementById('admin-modal');
            const content = document.getElementById('modal-content');
            modal.classList.remove('hidden');
            setTimeout(() => {
                content.classList.remove('scale-95', 'opacity-0');
                document.getElementById('admin-pass-input').focus();
            }, 10);
        }

        function closeAdminModal() {
            const modal = document.getElementById('admin-modal');
            const content = document.getElementById('modal-content');
            content.classList.add('scale-95', 'opacity-0');
            setTimeout(() => modal.classList.add('hidden'), 300);
        }

        function verifyAdmin() {
            const pass = document.getElementById('admin-pass-input').value;
            if (pass === ADMIN_PASSWORD) {
                isAdmin = true;
                document.getElementById('admin-panel').classList.remove('hidden');
                document.getElementById('admin-trigger').classList.add('bg-emerald-600');
                closeAdminModal();
                showToast("Admin access granted.");
                document.getElementById('admin-pass-input').value = '';
                renderPolls();
            } else {
                showToast("Incorrect password", "error");
                document.getElementById('admin-pass-input').classList.add('border-rose-500');
                setTimeout(() => document.getElementById('admin-pass-input').classList.remove('border-rose-500'), 1000);
            }
        }

        function logoutAdmin() {
            isAdmin = false;
            document.getElementById('admin-panel').classList.add('hidden');
            document.getElementById('admin-trigger').classList.remove('bg-emerald-600');
            showToast("Logged out from admin");
            renderPolls();
        }

        // Business Logic
        function handleCookie(accept) {
            cookieConsent = accept ? 'accepted' : 'declined';
            localStorage.setItem('cookie_consent', cookieConsent);
            document.getElementById('cookie-banner').classList.add('hidden');
            renderPolls();
        }

        async function detectLocation() {
            try {
                const res = await fetch('https://ipapi.co/json/');
                const data = await res.json();
                userCountry = data.country_code || null;
                updateLocationUI();
            } catch (e) { }
        }

        async function fetchPolls() {
            const { data, error } = await sbClient.from('polls').select('*').order('created_at', { ascending: false });
            if (!error) { 
                polls = data || []; 
                renderPolls(); 
            }
        }

        function setupRealtime() {
            sbClient.channel('db-changes').on('postgres_changes', { event: '*', schema: 'public', table: 'polls' }, () => fetchPolls()).subscribe();
        }

        async function vote(pollId, idx) {
            if (cookieConsent === 'declined') return showToast("Please accept cookies to vote", "error");
            if (!cookieConsent) return document.getElementById('cookie-banner').classList.remove('hidden');
            
            const poll = polls.find(p => p.id === pollId);
            if (votedPolls.includes(pollId)) return;

            const updatedOptions = [...poll.options];
            updatedOptions[idx].votes += 1;
            
            const { error } = await sbClient.from('polls').update({ 
                options: updatedOptions, 
                total_votes: (poll.total_votes || 0) + 1 
            }).eq('id', pollId);

            if (!error) {
                votedPolls.push(pollId);
                localStorage.setItem('voted_polls', JSON.stringify(votedPolls));
                renderPolls();
                showToast("Vote recorded!");
            }
        }

        async function toggleStatus(id, current) {
            if (!isAdmin) return;
            await sbClient.from('polls').update({ active: !current }).eq('id', id);
        }

        async function deletePoll(id) {
            if (!isAdmin) return;
            if (confirm("Delete this poll permanently?")) {
                await sbClient.from('polls').delete().eq('id', id);
            }
        }

        function renderPolls() {
            const container = document.getElementById('polls-list');
            const filtered = polls.filter(p => view === 'active' ? p.active : !p.active);

            if (filtered.length === 0) {
                container.innerHTML = `<div class="col-span-full py-20 text-center text-slate-400 font-bold">No ${view} polls found.</div>`;
                return;
            }

            container.innerHTML = filtered.map(poll => {
                const total = poll.total_votes || 0;
                const hasVoted = votedPolls.includes(poll.id);
                const isBlocked = cookieConsent === 'declined';

                return `
                    <div class="poll-card glass rounded-[2.5rem] p-8 flex flex-col border border-slate-200 relative group">
                        ${isAdmin ? `
                            <div class="absolute top-4 right-4 flex gap-2 opacity-0 group-hover:opacity-100 transition-opacity">
                                <button onclick="toggleStatus('${poll.id}', ${poll.active})" class="p-2 bg-white shadow-sm border rounded-xl hover:text-emerald-500">
                                    <i data-lucide="${poll.active ? 'archive' : 'check-circle'}" class="w-4 h-4"></i>
                                </button>
                                <button onclick="deletePoll('${poll.id}')" class="p-2 bg-white shadow-sm border rounded-xl hover:text-rose-500">
                                    <i data-lucide="trash-2" class="w-4 h-4"></i>
                                </button>
                            </div>
                        ` : ''}
                        
                        <div class="mb-6 space-y-1">
                            <span class="text-[10px] font-black uppercase text-emerald-600 tracking-widest">${poll.month || 'Poll'}</span>
                            <h3 class="text-xl font-extrabold text-slate-900 leading-tight">${poll.question}</h3>
                        </div>

                        <div class="space-y-3 flex-1">
                            ${poll.options.map((opt, i) => {
                                const perc = total > 0 ? Math.round((opt.votes / total) * 100) : 0;
                                const showResults = hasVoted || !poll.active;
                                return `
                                    <button onclick="vote('${poll.id}', ${i})" ${showResults || isBlocked ? 'disabled' : ''} class="group relative w-full h-14 rounded-2xl overflow-hidden border border-slate-100 bg-white text-left transition-all">
                                        <div class="progress-fill absolute inset-0 bg-emerald-500/10" style="width: ${showResults ? perc : 0}%"></div>
                                        <div class="relative z-10 px-5 flex items-center justify-between font-bold text-sm">
                                            <span class="${showResults ? 'text-slate-800' : 'text-slate-500 group-hover:text-emerald-600'}">${opt.text}</span>
                                            ${showResults ? `
                                                <span class="bg-emerald-100 text-emerald-700 px-2 py-0.5 rounded-lg text-[11px] font-black">${perc}%</span>
                                            ` : ''}
                                        </div>
                                    </button>
                                `;
                            }).join('')}
                        </div>

                        <div class="mt-8 pt-6 border-t border-slate-100 flex items-center justify-between">
                            <span class="text-[10px] font-black uppercase tracking-widest text-slate-400">${total} Participants</span>
                            ${hasVoted ? `<span class="text-[9px] font-bold text-emerald-600 uppercase tracking-tighter">Vote Recorded</span>` : ''}
                        </div>
                    </div>
                `;
            }).join('');
            lucide.createIcons();
        }

        function setView(v) { 
            view = v; 
            document.getElementById('btn-active').className = v === 'active' ? 'px-5 py-2 rounded-xl text-xs font-black uppercase tracking-widest transition-all bg-white text-slate-900 shadow-sm' : 'px-5 py-2 rounded-xl text-xs font-black uppercase tracking-widest transition-all text-slate-500';
            document.getElementById('btn-archive').className = v === 'archive' ? 'px-5 py-2 rounded-xl text-xs font-black uppercase tracking-widest transition-all bg-white text-slate-900 shadow-sm' : 'px-5 py-2 rounded-xl text-xs font-black uppercase tracking-widest transition-all text-slate-500';
            renderPolls(); 
        }

        function addOptionField() {
            const container = document.getElementById('poll-options-container');
            const i = document.createElement('input');
            i.className = "poll-option-input w-full p-4 bg-white border border-slate-200 rounded-2xl outline-none focus:ring-2 focus:ring-emerald-500";
            i.placeholder = "New Option";
            container.appendChild(i);
        }

        async function createPoll() {
            const question = document.getElementById('poll-question').value;
            const month = document.getElementById('poll-month').value;
            const opts = Array.from(document.querySelectorAll('.poll-option-input')).map(i => ({text: i.value, votes: 0})).filter(o => o.text);
            
            if (!question || opts.length < 2) return showToast("Need question and 2 options", "error");

            const { error } = await sbClient.from('polls').insert([{ 
                question, 
                options: opts, 
                active: true, 
                total_votes: 0, 
                month 
            }]);
            
            if (!error) {
                showToast("Poll launched successfully!");
                document.getElementById('poll-question').value = '';
                document.getElementById('poll-month').value = '';
                document.querySelectorAll('.poll-option-input').forEach((input, index) => {
                    if (index > 1) input.remove();
                    else input.value = '';
                });
            }
        }

        function updateLocationUI() { document.getElementById('loc-text').innerText = userCountry || "Global"; }
        function showManualLocation() { const l = prompt("Enter Country Code (e.g. US, GB):"); if(l) { userCountry = l.toUpperCase(); localStorage.setItem('user_manual_country', userCountry); updateLocationUI(); renderPolls(); } }

        window.onload = initApp;
    </script>
</body>
</html>
