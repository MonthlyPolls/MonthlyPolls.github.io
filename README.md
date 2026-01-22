<!-- linguist-detectable=false -->
<!-- linguist-documentation=false -->
<!-- linguist-language=JavaScript -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MonthlyPolls - Community Engagement</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800&display=swap" rel="stylesheet">
    <style>
        body { 
            font-family: 'Plus Jakarta Sans', sans-serif; 
            scroll-behavior: smooth;
            background-color: #f1f5f9;
            background-image: radial-gradient(at 0% 0%, rgba(16, 185, 129, 0.05) 0, transparent 50%), 
                              radial-gradient(at 50% 0%, rgba(59, 130, 246, 0.05) 0, transparent 50%);
        }
        [v-cloak] { display: none; }
        .progress-bar-fill { 
            transition: width 1s cubic-bezier(0.34, 1.56, 0.64, 1); 
        }
        .glass-effect {
            background: rgba(255, 255, 255, 0.7);
            backdrop-filter: blur(16px);
            -webkit-backdrop-filter: blur(16px);
        }
        .poll-card {
            transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        }
        .poll-card:hover {
            transform: translateY(-4px);
            box-shadow: 0 20px 25px -5px rgb(0 0 0 / 0.05), 0 8px 10px -6px rgb(0 0 0 / 0.05);
        }
        .btn-hover-effect {
            transition: transform 0.1s active;
        }
        .btn-hover-effect:active {
            transform: scale(0.96);
        }
    </style>
</head>
<body class="text-slate-900 antialiased min-h-screen">
    <div id="app" class="pb-20">
        <!-- Navigation -->
        <nav class="sticky top-0 z-40 glass-effect border-b border-slate-200/60">
            <div class="max-w-5xl mx-auto px-4 h-20 flex items-center justify-between">
                <div class="flex items-center gap-3">
                    <div class="bg-emerald-600 p-2.5 rounded-2xl shadow-lg shadow-emerald-200">
                        <svg xmlns="http://www.w3.org/2000/svg" width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="20" x2="18" y2="10"></line><line x1="12" y1="20" x2="12" y2="4"></line><line x1="6" y1="20" x2="6" y2="14"></line></svg>
                    </div>
                    <div>
                        <h1 class="text-xl font-extrabold tracking-tight text-slate-900 leading-none">MonthlyPolls</h1>
                        <p class="text-[10px] uppercase tracking-[0.2em] font-black text-slate-400 mt-1">By Monthly</p>
                    </div>
                </div>
                
                <div class="hidden md:flex items-center gap-2 bg-slate-200/50 p-1.5 rounded-2xl border border-slate-200/50">
                    <button id="view-active" class="px-6 py-2 rounded-xl text-sm font-bold transition-all bg-white text-emerald-600 shadow-sm border border-slate-100">Active</button>
                    <button id="view-archive" class="px-6 py-2 rounded-xl text-sm font-bold transition-all text-slate-500 hover:text-slate-700">Archives</button>
                </div>

                <div class="flex items-center gap-3">
                    <button id="admin-toggle" class="p-3 rounded-2xl border border-slate-200 bg-white text-slate-400 hover:text-emerald-600 hover:border-emerald-200 transition-all shadow-sm group">
                        <svg id="lock-icon" class="group-hover:scale-110 transition-transform" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect><path d="M7 11V7a5 5 0 0 1 10 0v4"></path></svg>
                    </button>
                </div>
            </div>
        </nav>

        <header class="max-w-3xl mx-auto px-4 pt-16 text-center">
            <h2 class="text-4xl md:text-5xl font-black text-slate-900 tracking-tight mb-4">Shape our future together.</h2>
            <p class="text-slate-500 text-lg font-medium max-w-xl mx-auto">Cast your vote on this month's community topics and see real-time insights from your peers.</p>
        </header>

        <main class="max-w-3xl mx-auto px-4 pt-12 space-y-10">
            <!-- Search & Filter Mobile Toggle -->
            <div class="flex flex-col md:flex-row gap-4 items-center">
                <div class="relative w-full">
                    <div class="absolute inset-y-0 left-0 pl-4 flex items-center pointer-events-none text-slate-400">
                        <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg>
                    </div>
                    <input type="text" id="poll-search" placeholder="Search polls..." class="w-full pl-11 pr-4 py-4 bg-white border border-slate-200 rounded-[22px] outline-none focus:ring-4 focus:ring-emerald-500/10 focus:border-emerald-500 transition-all font-medium">
                </div>
                <div class="md:hidden flex w-full gap-2 p-1 bg-slate-200/50 rounded-2xl border border-slate-200/50">
                    <button onclick="setView('active')" class="flex-1 py-3 text-sm font-bold rounded-xl transition-all" id="m-view-active">Active</button>
                    <button onclick="setView('archive')" class="flex-1 py-3 text-sm font-bold rounded-xl transition-all" id="m-view-archive">Archives</button>
                </div>
            </div>

            <!-- Admin Login Modal -->
            <div id="admin-modal" class="hidden fixed inset-0 bg-slate-900/60 backdrop-blur-md z-50 flex items-center justify-center p-4">
                <div class="bg-white rounded-[40px] p-10 w-full max-w-sm shadow-2xl animate-in zoom-in-95 duration-200">
                    <div class="text-center mb-8">
                        <div class="w-20 h-20 bg-emerald-50 text-emerald-600 rounded-3xl flex items-center justify-center mx-auto mb-6">
                            <svg xmlns="http://www.w3.org/2000/svg" width="36" height="36" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect><path d="M7 11V7a5 5 0 0 1 10 0v4"></path></svg>
                        </div>
                        <h3 class="text-2xl font-black text-slate-900">Admin Control</h3>
                        <p class="text-slate-400 font-medium mt-1">Hello Smith, please verify.</p>
                    </div>
                    <form id="admin-form" class="space-y-4">
                        <input type="password" id="admin-pass" placeholder="Enter Password" class="w-full p-5 bg-slate-50 border border-slate-200 rounded-[22px] outline-none focus:ring-4 focus:ring-emerald-500/10 focus:border-emerald-500 transition-all text-center text-xl font-bold tracking-widest">
                        <div class="flex flex-col gap-3 pt-2">
                            <button type="submit" class="w-full bg-emerald-600 text-white py-5 rounded-[22px] font-black text-lg shadow-xl shadow-emerald-200 hover:bg-emerald-700 transition-all active:scale-95">Verify Identity</button>
                            <button type="button" id="close-modal" class="w-full text-slate-400 py-2 text-sm font-bold hover:text-slate-600 transition-all">Go Back</button>
                        </div>
                    </form>
                </div>
            </div>

            <!-- Create Poll Section -->
            <section id="create-poll-section" class="hidden bg-white border border-emerald-100 rounded-[40px] p-10 shadow-2xl shadow-emerald-900/5 animate-in slide-in-from-top-6 duration-500">
                <div class="flex items-center gap-4 mb-8">
                    <div class="p-3 bg-emerald-50 text-emerald-600 rounded-2xl">
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><path d="M12 5v14M5 12h14"/></svg>
                    </div>
                    <h2 class="text-2xl font-black text-slate-900 tracking-tight">Launch New Poll</h2>
                </div>
                <div class="space-y-8">
                    <div class="relative">
                        <label class="text-[11px] font-black text-slate-400 uppercase tracking-[0.2em] mb-3 block">Topic / Question</label>
                        <textarea id="poll-question" rows="2" placeholder="What's on the community's mind?" class="w-full p-5 bg-slate-50 border border-slate-200 rounded-[22px] focus:ring-4 focus:ring-emerald-500/10 focus:border-emerald-500 outline-none transition-all font-semibold resize-none"></textarea>
                    </div>
                    <div>
                        <label class="text-[11px] font-black text-slate-400 uppercase tracking-[0.2em] mb-3 block">Choices</label>
                        <div id="options-container" class="space-y-3">
                            <div class="relative">
                                <input type="text" placeholder="Option 1" class="poll-opt w-full p-5 bg-slate-50 border border-slate-200 rounded-[22px] focus:border-emerald-500 outline-none font-semibold transition-all">
                            </div>
                            <div class="relative">
                                <input type="text" placeholder="Option 2" class="poll-opt w-full p-5 bg-slate-50 border border-slate-200 rounded-[22px] focus:border-emerald-500 outline-none font-semibold transition-all">
                            </div>
                        </div>
                        <button id="add-opt" class="mt-5 text-sm text-emerald-600 font-extrabold flex items-center gap-2 hover:bg-emerald-50 py-3 px-6 rounded-2xl transition-all">
                            <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><path d="M12 5v14M5 12h14"/></svg>
                            Add another choice
                        </button>
                    </div>
                    <button id="submit-poll" class="w-full bg-emerald-600 text-white py-6 rounded-[24px] font-black text-xl shadow-2xl shadow-emerald-200 hover:bg-emerald-700 transition-all active:scale-[0.98]">Publish to Community</button>
                </div>
            </section>

            <!-- Poll List -->
            <div id="poll-list" class="space-y-10">
                <!-- Polls injected here -->
            </div>
        </main>
    </div>

    <script>
        const db = window.supabase.createClient(
            'https://zhwjrgkkucmifdalcgsg.supabase.co', 
            'sb_publishable_T83Yr-DRdL2ZiqCSu2Fnjw_soVm-wMG'
        );

        let isAdmin = false;
        let currentView = 'active';
        let votedPolls = JSON.parse(localStorage.getItem('votedPolls') || '[]');
        let allPolls = [];

        const pollList = document.getElementById('poll-list');
        const adminModal = document.getElementById('admin-modal');
        const createSection = document.getElementById('create-poll-section');
        const searchInput = document.getElementById('poll-search');

        async function fetchPolls() {
            const { data, error } = await db
                .from('polls')
                .select('*')
                .order('created_at', { ascending: false });

            if (error) return console.error(error);
            allPolls = data;
            renderPolls();
        }

        searchInput.addEventListener('input', renderPolls);

        function setView(view) {
            currentView = view;
            updateNav();
            renderPolls();
        }

        function renderPolls() {
            const query = searchInput.value.toLowerCase();
            const filtered = allPolls.filter(p => {
                const matchesView = currentView === 'active' ? p.active : !p.active;
                const matchesSearch = p.question.toLowerCase().includes(query);
                return matchesView && matchesSearch;
            });
            
            if (filtered.length === 0) {
                pollList.innerHTML = `
                    <div class="text-center py-24 bg-white/50 rounded-[48px] border-2 border-dashed border-slate-200/60 animate-in fade-in duration-700">
                        <div class="bg-slate-100 w-24 h-24 rounded-[32px] flex items-center justify-center mx-auto mb-6 text-slate-300">
                            <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"></circle><line x1="12" y1="8" x2="12" y2="12"></line><line x1="12" y1="16" x2="12.01" y2="16"></line></svg>
                        </div>
                        <h4 class="text-xl font-black text-slate-800">Nothing found</h4>
                        <p class="text-slate-400 font-bold mt-1">Try adjusting your filters or search.</p>
                    </div>
                `;
                return;
            }

            pollList.innerHTML = filtered.map(poll => {
                const totalVotes = poll.total_votes || 0;
                const hasVoted = votedPolls.includes(poll.id) || !poll.active;
                const dateString = new Date(poll.created_at).toLocaleDateString('en-US', { month: 'long', day: 'numeric' });

                return `
                    <div class="poll-card bg-white border border-slate-200/80 rounded-[44px] p-10 shadow-sm relative overflow-hidden group">
                        <!-- Decorative bg -->
                        <div class="absolute -right-16 -top-16 w-48 h-48 bg-emerald-50/50 rounded-full blur-3xl group-hover:bg-emerald-100 transition-colors"></div>

                        <div class="relative z-10">
                            <div class="flex flex-col md:flex-row md:justify-between md:items-start gap-4 mb-8">
                                <div class="space-y-3">
                                    <div class="flex items-center gap-3">
                                        <span class="text-[10px] font-black uppercase tracking-[0.2em] text-emerald-600 bg-emerald-50 px-4 py-1.5 rounded-full border border-emerald-100">
                                            ${poll.active ? 'Active Topic' : 'Closed Poll'}
                                        </span>
                                        <span class="text-[10px] font-bold text-slate-400 uppercase tracking-widest">${dateString}</span>
                                    </div>
                                    <h3 class="text-3xl font-black text-slate-900 leading-[1.15] tracking-tight max-w-lg transition-colors group-hover:text-emerald-950">${poll.question}</h3>
                                </div>
                                
                                ${isAdmin ? `
                                    <div class="flex gap-2 self-start bg-slate-50 p-1.5 rounded-2xl border border-slate-100">
                                        <button onclick="toggleArchive('${poll.id}', ${poll.active})" title="Toggle Status" class="p-3 text-slate-400 hover:text-emerald-600 hover:bg-white hover:shadow-sm rounded-xl transition-all">
                                            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M21 8V20a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8"/><path d="M1 3h22v5H1z"/><path d="M10 12h4"/></svg>
                                        </button>
                                        <button onclick="deletePoll('${poll.id}')" title="Delete Permanent" class="p-3 text-slate-400 hover:text-rose-600 hover:bg-white hover:shadow-sm rounded-xl transition-all">
                                            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg>
                                        </button>
                                    </div>
                                ` : ''}
                            </div>

                            <div class="grid grid-cols-1 gap-4">
                                ${poll.options.map((opt, idx) => {
                                    const pct = totalVotes > 0 ? Math.round((opt.votes / totalVotes) * 100) : 0;
                                    const isWinner = !poll.active && pct === Math.max(...poll.options.map(o => (o.votes/totalVotes)*100));
                                    
                                    return `
                                        <div class="relative group/opt">
                                            <button onclick="vote('${poll.id}', ${idx})" ${hasVoted ? 'disabled' : ''} 
                                                class="w-full py-5 px-8 relative z-10 border-2 rounded-[24px] flex justify-between items-center transition-all ${
                                                    hasVoted 
                                                    ? 'border-transparent cursor-default' 
                                                    : 'border-slate-100 bg-slate-50/50 hover:border-emerald-500 hover:bg-white hover:shadow-xl hover:shadow-emerald-900/5'
                                                }">
                                                <div class="flex items-center gap-4">
                                                    ${isWinner ? '<span class="text-xl">👑</span>' : ''}
                                                    <span class="font-extrabold text-lg ${hasVoted ? 'text-slate-900' : 'text-slate-700'}">${opt.text}</span>
                                                </div>
                                                
                                                ${hasVoted ? `
                                                    <div class="flex items-center gap-4">
                                                        <span class="text-xs font-black text-slate-400">${opt.votes} votes</span>
                                                        <span class="text-lg font-black text-emerald-600 tabular-nums">${pct}%</span>
                                                    </div>
                                                ` : `
                                                    <div class="w-8 h-8 rounded-full border-2 border-slate-200 group-hover/opt:border-emerald-500 group-hover/opt:bg-emerald-500 transition-all flex items-center justify-center">
                                                        <svg xmlns="http://www.w3.org/2000/svg" class="text-transparent group-hover/opt:text-white transition-colors" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="4" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12"></polyline></svg>
                                                    </div>
                                                `}
                                            </button>
                                            ${hasVoted ? `
                                                <div class="absolute inset-0 bg-slate-100/50 rounded-[24px] overflow-hidden">
                                                    <div class="h-full bg-emerald-600/10 progress-bar-fill" style="width: ${pct}%"></div>
                                                </div>
                                            ` : ''}
                                        </div>
                                    `;
                                }).join('')}
                            </div>

                            <div class="mt-10 pt-8 border-t border-slate-100 flex flex-wrap items-center justify-between gap-4">
                                <div class="flex items-center gap-6">
                                    <div class="flex items-center gap-2">
                                        <div class="flex -space-x-2">
                                            <div class="w-7 h-7 rounded-full bg-blue-100 border-2 border-white"></div>
                                            <div class="w-7 h-7 rounded-full bg-emerald-100 border-2 border-white"></div>
                                            <div class="w-7 h-7 rounded-full bg-rose-100 border-2 border-white"></div>
                                        </div>
                                        <span class="text-xs font-black text-slate-500 uppercase tracking-widest pl-2">${totalVotes} Voices</span>
                                    </div>
                                    <div class="h-4 w-[1px] bg-slate-200"></div>
                                    <div class="text-xs font-black text-slate-400 uppercase tracking-widest">
                                        Status: <span class="${poll.active ? 'text-emerald-500' : 'text-rose-400'}">${poll.active ? 'Open for voting' : 'Archived Results'}</span>
                                    </div>
                                </div>
                                
                                ${votedPolls.includes(poll.id) ? `
                                    <div class="bg-emerald-50 text-emerald-700 text-[10px] font-black uppercase tracking-widest px-4 py-2 rounded-xl flex items-center gap-2">
                                        <svg xmlns="http://www.w3.org/2000/svg" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="4" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12"></polyline></svg>
                                        Your Choice is Recorded
                                    </div>
                                ` : ''}
                            </div>
                        </div>
                    </div>
                `;
            }).join('');
        }

        async function vote(pollId, idx) {
            if (votedPolls.includes(pollId)) return;
            
            const { data: poll } = await db.from('polls').select('*').eq('id', pollId).single();
            if (!poll.active) return;

            const opts = [...poll.options];
            opts[idx].votes++;
            
            await db.from('polls').update({ 
                options: opts, 
                total_votes: (poll.total_votes || 0) + 1 
            }).eq('id', pollId);

            votedPolls.push(pollId);
            localStorage.setItem('votedPolls', JSON.stringify(votedPolls));
            fetchPolls();
        }

        async function toggleArchive(id, currentStatus) {
            await db.from('polls').update({ active: !currentStatus }).eq('id', id);
            fetchPolls();
        }

        async function deletePoll(id) {
            const confirmed = confirm('Delete this poll permanently? This action is irreversible.');
            if (!confirmed) return;
            await db.from('polls').delete().eq('id', id);
            fetchPolls();
        }

        document.getElementById('admin-form').onsubmit = (e) => {
            e.preventDefault();
            if (document.getElementById('admin-pass').value === 'Smith123') {
                isAdmin = true;
                createSection.classList.remove('hidden');
                adminModal.classList.add('hidden');
                document.getElementById('admin-toggle').classList.add('bg-emerald-600', 'text-white', 'border-emerald-600', 'shadow-emerald-200');
                document.getElementById('admin-toggle').innerHTML = `
                    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect><path d="M7 11V7a5 5 0 0 1 10 0v4"></path></svg>
                `;
                fetchPolls();
            } else {
                const input = document.getElementById('admin-pass');
                input.classList.add('border-rose-500', 'bg-rose-50');
                setTimeout(() => input.classList.remove('border-rose-500', 'bg-rose-50'), 1000);
            }
        };

        document.getElementById('admin-toggle').onclick = () => {
            if (isAdmin) {
                isAdmin = false;
                createSection.classList.add('hidden');
                const btn = document.getElementById('admin-toggle');
                btn.classList.remove('bg-emerald-600', 'text-white', 'border-emerald-600', 'shadow-emerald-200');
                btn.innerHTML = `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect><path d="M7 11V7a5 5 0 0 1 10 0v4"></path></svg>`;
                fetchPolls();
            } else {
                adminModal.classList.remove('hidden');
                document.getElementById('admin-pass').focus();
            }
        };
        
        document.getElementById('close-modal').onclick = () => adminModal.classList.add('hidden');

        document.getElementById('view-active').onclick = () => setView('active');
        document.getElementById('view-archive').onclick = () => setView('archive');

        function updateNav() {
            const activeBtn = document.getElementById('view-active');
            const archiveBtn = document.getElementById('view-archive');
            const mActiveBtn = document.getElementById('m-view-active');
            const mArchiveBtn = document.getElementById('m-view-archive');

            const activeClass = 'bg-white text-emerald-600 shadow-sm border border-slate-100';
            const inactiveClass = 'text-slate-500 hover:text-slate-700';

            if (currentView === 'active') {
                activeBtn.className = `px-6 py-2 rounded-xl text-sm font-bold transition-all ${activeClass}`;
                archiveBtn.className = `px-6 py-2 rounded-xl text-sm font-bold transition-all ${inactiveClass}`;
                mActiveBtn.className = 'flex-1 py-3 text-sm font-bold rounded-xl transition-all bg-white text-emerald-600 shadow-sm';
                mArchiveBtn.className = 'flex-1 py-3 text-sm font-bold rounded-xl transition-all text-slate-500';
            } else {
                archiveBtn.className = `px-6 py-2 rounded-xl text-sm font-bold transition-all ${activeClass}`;
                activeBtn.className = `px-6 py-2 rounded-xl text-sm font-bold transition-all ${inactiveClass}`;
                mArchiveBtn.className = 'flex-1 py-3 text-sm font-bold rounded-xl transition-all bg-white text-emerald-600 shadow-sm';
                mActiveBtn.className = 'flex-1 py-3 text-sm font-bold rounded-xl transition-all text-slate-500';
            }
        }

        document.getElementById('add-opt').onclick = () => {
            const container = document.getElementById('options-container');
            const wrapper = document.createElement('div');
            wrapper.className = 'animate-in fade-in slide-in-from-left-4 duration-300';
            wrapper.innerHTML = `
                <input type="text" placeholder="Option ${container.children.length + 1}" class="poll-opt w-full p-5 bg-slate-50 border border-slate-200 rounded-[22px] focus:border-emerald-500 outline-none font-semibold transition-all">
            `;
            container.appendChild(wrapper);
        };

        document.getElementById('submit-poll').onclick = async () => {
            const question = document.getElementById('poll-question').value.trim();
            const optionElements = document.querySelectorAll('.poll-opt');
            const options = Array.from(optionElements)
                .map(el => el.value.trim())
                .filter(val => val !== '')
                .map(text => ({ text, votes: 0 }));

            if (!question || options.length < 2) return alert('Please enter a topic and at least 2 choices.');

            const btn = document.getElementById('submit-poll');
            const originalText = btn.innerText;
            btn.disabled = true;
            btn.innerText = 'Publishing...';

            await db.from('polls').insert([{
                question,
                options,
                active: true,
                total_votes: 0,
                created_at: new Date()
            }]);

            document.getElementById('poll-question').value = '';
            document.getElementById('options-container').innerHTML = `
                <input type="text" placeholder="Option 1" class="poll-opt w-full p-5 bg-slate-50 border border-slate-200 rounded-[22px] focus:border-emerald-500 outline-none font-semibold">
                <input type="text" placeholder="Option 2" class="poll-opt w-full p-5 bg-slate-50 border border-slate-200 rounded-[22px] focus:border-emerald-500 outline-none font-semibold">
            `;
            
            btn.disabled = false;
            btn.innerText = originalText;
            fetchPolls();
        };

        db.channel('polls-live').on('postgres_changes', { event: '*', schema: 'public', table: 'polls' }, fetchPolls).subscribe();
        window.onload = () => {
            updateNav();
            fetchPolls();
        };
    </script>
</body>
</html>
