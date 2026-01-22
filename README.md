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
        }
        [v-cloak] { display: none; }
        .progress-bar-fill { 
            transition: width 1s cubic-bezier(0.34, 1.56, 0.64, 1); 
        }
        .glass-effect {
            background: rgba(255, 255, 255, 0.8);
            backdrop-filter: blur(12px);
            -webkit-backdrop-filter: blur(12px);
        }
        .poll-card {
            transition: transform 0.2s ease, border-color 0.2s ease;
        }
        .poll-card:hover {
            transform: translateY(-2px);
        }
    </style>
</head>
<body class="bg-[#F8FAFC] text-slate-900 antialiased">
    <div id="app" class="min-h-screen pb-20">
        <!-- Navigation -->
        <nav class="sticky top-0 z-40 glass-effect border-b border-slate-200">
            <div class="max-w-5xl mx-auto px-4 h-20 flex items-center justify-between">
                <div class="flex items-center gap-3">
                    <div class="bg-emerald-600 p-2 rounded-xl shadow-lg shadow-emerald-200">
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="20" x2="18" y2="10"></line><line x1="12" y1="20" x2="12" y2="4"></line><line x1="6" y1="20" x2="6" y2="14"></line></svg>
                    </div>
                    <div>
                        <h1 class="text-xl font-bold tracking-tight text-slate-900">MonthlyPolls</h1>
                        <p class="text-[10px] uppercase tracking-widest font-bold text-slate-400">By Monthly</p>
                    </div>
                </div>
                
                <div class="flex items-center gap-2 bg-slate-100 p-1 rounded-2xl border border-slate-200">
                    <button id="view-active" class="px-5 py-2 rounded-xl text-sm font-semibold transition-all bg-white text-emerald-600 shadow-sm">Active</button>
                    <button id="view-archive" class="px-5 py-2 rounded-xl text-sm font-semibold transition-all text-slate-500 hover:text-slate-700">Archives</button>
                </div>

                <button id="admin-toggle" class="ml-2 p-2.5 rounded-xl border border-slate-200 bg-white text-slate-400 hover:text-emerald-600 hover:border-emerald-200 transition-all">
                    <svg id="lock-icon" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect><path d="M7 11V7a5 5 0 0 1 10 0v4"></path></svg>
                </button>
            </div>
        </nav>

        <main class="max-w-3xl mx-auto px-4 pt-12 space-y-10">
            <!-- Admin Login Modal -->
            <div id="admin-modal" class="hidden fixed inset-0 bg-slate-900/60 backdrop-blur-md z-50 flex items-center justify-center p-4">
                <div class="bg-white rounded-3xl p-8 w-full max-w-sm shadow-2xl animate-in zoom-in-95 duration-200">
                    <div class="text-center mb-8">
                        <div class="w-16 h-16 bg-emerald-50 text-emerald-600 rounded-2xl flex items-center justify-center mx-auto mb-4">
                            <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect><path d="M7 11V7a5 5 0 0 1 10 0v4"></path></svg>
                        </div>
                        <h3 class="text-xl font-extrabold text-slate-800">Admin Login</h3>
                        <p class="text-slate-400 text-sm">Enter password to manage polls</p>
                    </div>
                    <form id="admin-form" class="space-y-4">
                        <input type="password" id="admin-pass" placeholder="••••••••" class="w-full p-4 bg-slate-50 border border-slate-200 rounded-2xl outline-none focus:ring-4 focus:ring-emerald-500/10 focus:border-emerald-500 transition-all text-center text-lg">
                        <div class="flex gap-3">
                            <button type="button" id="close-modal" class="flex-1 bg-slate-100 text-slate-600 py-4 rounded-2xl font-bold hover:bg-slate-200 transition-all">Cancel</button>
                            <button type="submit" class="flex-[2] bg-emerald-600 text-white py-4 rounded-2xl font-bold shadow-lg shadow-emerald-200 hover:bg-emerald-700 transition-all">Unlock</button>
                        </div>
                    </form>
                </div>
            </div>

            <!-- Create Poll Section -->
            <section id="create-poll-section" class="hidden bg-white border border-emerald-100 rounded-[32px] p-8 shadow-xl shadow-emerald-900/5 animate-in slide-in-from-top-4 duration-500">
                <div class="flex items-center gap-3 mb-6">
                    <span class="p-2 bg-emerald-50 text-emerald-600 rounded-lg">
                        <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M12 5v14M5 12h14"/></svg>
                    </span>
                    <h2 class="text-xl font-bold text-slate-800">New Monthly Poll</h2>
                </div>
                <div class="space-y-6">
                    <div>
                        <label class="text-xs font-bold text-slate-400 uppercase tracking-widest mb-2 block">The Question</label>
                        <input type="text" id="poll-question" placeholder="e.g. What should our next event be?" class="w-full p-4 bg-slate-50 border border-slate-200 rounded-2xl focus:ring-4 focus:ring-emerald-500/10 focus:border-emerald-500 outline-none transition-all">
                    </div>
                    <div>
                        <label class="text-xs font-bold text-slate-400 uppercase tracking-widest mb-2 block">Options</label>
                        <div id="options-container" class="grid grid-cols-1 gap-3">
                            <input type="text" placeholder="Option 1" class="poll-opt p-4 bg-slate-50 border border-slate-200 rounded-2xl focus:border-emerald-500 outline-none">
                            <input type="text" placeholder="Option 2" class="poll-opt p-4 bg-slate-50 border border-slate-200 rounded-2xl focus:border-emerald-500 outline-none">
                        </div>
                        <button id="add-opt" class="mt-4 text-sm text-emerald-600 font-bold flex items-center gap-2 hover:bg-emerald-50 p-2 px-4 rounded-xl transition-all">
                            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><path d="M12 5v14M5 12h14"/></svg>
                            Add another choice
                        </button>
                    </div>
                    <button id="submit-poll" class="w-full bg-emerald-600 text-white py-5 rounded-[20px] font-extrabold text-lg shadow-xl shadow-emerald-200 hover:bg-emerald-700 transition-all active:scale-[0.98]">Launch Poll</button>
                </div>
            </section>

            <!-- Poll List -->
            <div id="poll-list" class="space-y-8">
                <!-- Polls injected here -->
            </div>
        </main>
    </div>

    <script>
        // Use unique naming to prevent global scope conflicts
        const db = window.supabase.createClient(
            'https://zhwjrgkkucmifdalcgsg.supabase.co', 
            'sb_publishable_T83Yr-DRdL2ZiqCSu2Fnjw_soVm-wMG'
        );

        let isAdmin = false;
        let currentView = 'active';
        let votedPolls = JSON.parse(localStorage.getItem('votedPolls') || '[]');

        const pollList = document.getElementById('poll-list');
        const adminModal = document.getElementById('admin-modal');
        const createSection = document.getElementById('create-poll-section');

        async function fetchPolls() {
            const { data, error } = await db
                .from('polls')
                .select('*')
                .order('created_at', { ascending: false });

            if (error) return console.error(error);
            renderPolls(data);
        }

        function renderPolls(polls) {
            const filtered = polls.filter(p => currentView === 'active' ? p.active : !p.active);
            
            if (filtered.length === 0) {
                pollList.innerHTML = `
                    <div class="text-center py-20 bg-white rounded-[40px] border border-dashed border-slate-200">
                        <div class="bg-slate-50 w-20 h-20 rounded-3xl flex items-center justify-center mx-auto mb-4 text-slate-300">
                            <svg xmlns="http://www.w3.org/2000/svg" width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"></circle><line x1="12" y1="8" x2="12" y2="12"></line><line x1="12" y1="16" x2="12.01" y2="16"></line></svg>
                        </div>
                        <p class="text-slate-400 font-bold">No ${currentView} polls at the moment.</p>
                    </div>
                `;
                return;
            }

            pollList.innerHTML = filtered.map(poll => {
                const totalVotes = poll.total_votes || 0;
                const hasVoted = votedPolls.includes(poll.id) || !poll.active;

                return `
                    <div class="poll-card bg-white border border-slate-200 rounded-[32px] p-8 shadow-sm hover:border-emerald-200 transition-all">
                        <div class="flex justify-between items-start mb-8">
                            <div class="space-y-1">
                                <span class="text-[10px] font-black uppercase tracking-[0.2em] text-emerald-500 bg-emerald-50 px-3 py-1 rounded-full">Monthly Insight</span>
                                <h3 class="text-2xl font-extrabold text-slate-800 leading-tight pt-2">${poll.question}</h3>
                            </div>
                            ${isAdmin ? `
                                <div class="flex gap-2">
                                    <button onclick="toggleArchive('${poll.id}', ${poll.active})" class="p-2 text-slate-400 hover:text-emerald-600 hover:bg-emerald-50 rounded-xl transition-all">
                                        <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 8V20a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8"/><path d="M1 3h22v5H1z"/><path d="M10 12h4"/></svg>
                                    </button>
                                    <button onclick="deletePoll('${poll.id}')" class="p-2 text-slate-400 hover:text-rose-500 hover:bg-rose-50 rounded-xl transition-all">
                                        <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg>
                                    </button>
                                </div>
                            ` : ''}
                        </div>
                        <div class="space-y-4">
                            ${poll.options.map((opt, idx) => {
                                const pct = totalVotes > 0 ? Math.round((opt.votes / totalVotes) * 100) : 0;
                                return `
                                    <div class="relative group">
                                        <button onclick="vote('${poll.id}', ${idx})" ${hasVoted ? 'disabled' : ''} 
                                            class="w-full py-4 px-6 relative z-10 border-2 rounded-2xl flex justify-between items-center transition-all ${
                                                hasVoted 
                                                ? 'border-transparent' 
                                                : 'border-slate-100 hover:border-emerald-500 bg-white hover:shadow-md'
                                            }">
                                            <span class="font-bold ${hasVoted ? 'text-slate-900' : 'text-slate-600'}">${opt.text}</span>
                                            ${hasVoted ? `<span class="text-sm font-black text-emerald-700 bg-white/80 px-2 py-1 rounded-lg">${pct}%</span>` : `
                                                <svg xmlns="http://www.w3.org/2000/svg" class="text-slate-200 group-hover:text-emerald-500 transition-colors" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12h14M12 5l7 7-7 7"/></svg>
                                            `}
                                        </button>
                                        ${hasVoted ? `
                                            <div class="absolute inset-0 bg-slate-100 rounded-2xl overflow-hidden">
                                                <div class="h-full bg-emerald-200/60 progress-bar-fill rounded-r-xl" style="width: ${pct}%"></div>
                                            </div>
                                        ` : ''}
                                    </div>
                                `;
                            }).join('')}
                        </div>
                        <div class="mt-8 pt-6 border-t border-slate-50 flex items-center justify-between text-slate-400 font-bold text-[11px] uppercase tracking-widest">
                            <div class="flex items-center gap-2">
                                <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path><circle cx="9" cy="7" r="4"></circle><path d="M23 21v-2a4 4 0 0 0-3-3.87"></path><path d="M16 3.13a4 4 0 0 1 0 7.75"></path></svg>
                                ${totalVotes} Votes Recorded
                            </div>
                            <div class="flex items-center gap-2">
                                ${!poll.active ? 'Results Finalized' : votedPolls.includes(poll.id) ? 'Vote Counted ✓' : 'Live Now'}
                            </div>
                        </div>
                    </div>
                `;
            }).join('');
        }

        async function vote(pollId, idx) {
            if (votedPolls.includes(pollId)) return;
            
            const { data: poll } = await db.from('polls').select('*').eq('id', pollId).single();
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
            if (!confirm('Are you sure you want to delete this poll?')) return;
            await db.from('polls').delete().eq('id', id);
            fetchPolls();
        }

        document.getElementById('admin-form').onsubmit = (e) => {
            e.preventDefault();
            if (document.getElementById('admin-pass').value === 'Smith123') {
                isAdmin = true;
                createSection.classList.remove('hidden');
                adminModal.classList.add('hidden');
                document.getElementById('admin-toggle').classList.add('bg-emerald-50', 'text-emerald-600', 'border-emerald-200');
                fetchPolls();
            } else {
                alert('Invalid password');
            }
        };

        document.getElementById('admin-toggle').onclick = () => {
            if (isAdmin) {
                isAdmin = false;
                createSection.classList.add('hidden');
                document.getElementById('admin-toggle').classList.remove('bg-emerald-50', 'text-emerald-600', 'border-emerald-200');
                fetchPolls();
            } else {
                adminModal.classList.remove('hidden');
                document.getElementById('admin-pass').focus();
            }
        };
        
        document.getElementById('close-modal').onclick = () => adminModal.classList.add('hidden');

        document.getElementById('view-active').onclick = () => { 
            currentView = 'active'; 
            updateNav();
            fetchPolls(); 
        };
        
        document.getElementById('view-archive').onclick = () => { 
            currentView = 'archive'; 
            updateNav();
            fetchPolls(); 
        };

        function updateNav() {
            const activeBtn = document.getElementById('view-active');
            const archiveBtn = document.getElementById('view-archive');
            if (currentView === 'active') {
                activeBtn.className = 'px-5 py-2 rounded-xl text-sm font-semibold transition-all bg-white text-emerald-600 shadow-sm';
                archiveBtn.className = 'px-5 py-2 rounded-xl text-sm font-semibold transition-all text-slate-500 hover:text-slate-700';
            } else {
                archiveBtn.className = 'px-5 py-2 rounded-xl text-sm font-semibold transition-all bg-white text-emerald-600 shadow-sm';
                activeBtn.className = 'px-5 py-2 rounded-xl text-sm font-semibold transition-all text-slate-500 hover:text-slate-700';
            }
        }

        document.getElementById('add-opt').onclick = () => {
            const container = document.getElementById('options-container');
            const input = document.createElement('input');
            input.type = 'text';
            input.placeholder = `Option ${container.children.length + 1}`;
            input.className = 'poll-opt p-4 bg-slate-50 border border-slate-200 rounded-2xl focus:border-emerald-500 outline-none animate-in fade-in slide-in-from-left-2 duration-300';
            container.appendChild(input);
        };

        document.getElementById('submit-poll').onclick = async () => {
            const question = document.getElementById('poll-question').value;
            const optionElements = document.querySelectorAll('.poll-opt');
            const options = Array.from(optionElements)
                .map(el => el.value.trim())
                .filter(val => val !== '')
                .map(text => ({ text, votes: 0 }));

            if (!question || options.length < 2) return alert('Please enter a question and at least 2 options.');

            const btn = document.getElementById('submit-poll');
            btn.disabled = true;
            btn.innerText = 'Launching...';

            await db.from('polls').insert([{
                question,
                options,
                active: true,
                total_votes: 0,
                created_at: new Date()
            }]);

            document.getElementById('poll-question').value = '';
            document.getElementById('options-container').innerHTML = `
                <input type="text" placeholder="Option 1" class="poll-opt p-4 bg-slate-50 border border-slate-200 rounded-2xl focus:border-emerald-500 outline-none">
                <input type="text" placeholder="Option 2" class="poll-opt p-4 bg-slate-50 border border-slate-200 rounded-2xl focus:border-emerald-500 outline-none">
            `;
            
            btn.disabled = false;
            btn.innerText = 'Launch Poll';
            fetchPolls();
        };

        db.channel('polls-all').on('postgres_changes', { event: '*', schema: 'public', table: 'polls' }, fetchPolls).subscribe();
        window.onload = fetchPolls;
    </script>
</body>
</html>
