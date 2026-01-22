// linguist-language: JavaScript
import React, { useState, useEffect } from 'react';
import { 
  Plus, 
  Trash2, 
  BarChart3, 
  CheckCircle2, 
  Clock, 
  Settings2,
  ChevronRight,
  Archive,
  Lock,
  Unlock,
  AlertCircle,
  X
} from 'lucide-react';

// --- Supabase Configuration ---
const SUPABASE_URL = 'https://zhwjrgkkucmifdalcgsg.supabase.co';
const SUPABASE_ANON_KEY = 'sb_publishable_T83Yr-DRdL2ZiqCSu2Fnjw_soVm-wMG';

const getSupabase = () => {
  if (window.supabase) {
    return window.supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
  }
  return null;
};

export default function App() {
  const [supabase, setSupabase] = useState(null);
  const [polls, setPolls] = useState([]);
  const [loading, setLoading] = useState(true);
  const [isAdmin, setIsAdmin] = useState(false);
  const [showAdminLogin, setShowAdminLogin] = useState(false);
  const [adminPassword, setAdminPassword] = useState('');
  const [errorState, setErrorState] = useState(null);
  const [votedPolls, setVotedPolls] = useState(new Set());
  
  const [newQuestion, setNewQuestion] = useState('');
  const [newOptions, setNewOptions] = useState(['', '']);
  const [view, setView] = useState('active');

  const handleAdminAuth = (e) => {
    e.preventDefault();
    if (adminPassword === 'Smith123') {
      setIsAdmin(true);
      setShowAdminLogin(false);
      setAdminPassword('');
      setErrorState(null);
    } else {
      setErrorState("Incorrect Admin Password");
    }
  };

  useEffect(() => {
    const script = document.createElement('script');
    script.src = 'https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2';
    script.async = true;
    script.onload = () => {
      setSupabase(getSupabase());
    };
    document.body.appendChild(script);
  }, []);

  useEffect(() => {
    if (!supabase) return;

    fetchPolls();

    const channel = supabase
      .channel('schema-db-changes')
      .on(
        'postgres_changes',
        { event: '*', schema: 'public', table: 'polls' },
        () => fetchPolls()
      )
      .subscribe();

    return () => {
      supabase.removeChannel(channel);
    };
  }, [supabase]);

  const fetchPolls = async () => {
    if (!supabase) return;
    const { data, error } = await supabase
      .from('polls')
      .select('*')
      .order('created_at', { ascending: false });

    if (error) {
      console.error('Error fetching:', error);
      setErrorState(error.message);
    } else {
      setPolls(data || []);
      setErrorState(null);
    }
    setLoading(false);
  };

  const createPoll = async () => {
    if (!supabase || !newQuestion.trim() || newOptions.some(opt => !opt.trim())) return;

    const pollData = {
      question: newQuestion,
      options: newOptions.filter(o => o.trim()).map(text => ({ text, votes: 0 })),
      active: true,
      total_votes: 0,
      month: new Date().toLocaleString('default', { month: 'long', year: 'numeric' })
    };

    const { error } = await supabase.from('polls').insert([pollData]);
    if (error) {
      setErrorState("Failed to create poll.");
    } else {
      setNewQuestion('');
      setNewOptions(['', '']);
      fetchPolls();
    }
  };

  const handleVote = async (pollId, optionIndex) => {
    if (!supabase || votedPolls.has(pollId)) return;

    const poll = polls.find(p => p.id === pollId);
    const updatedOptions = [...poll.options];
    updatedOptions[optionIndex].votes += 1;

    const { error } = await supabase
      .from('polls')
      .update({ 
        options: updatedOptions, 
        total_votes: (poll.total_votes || 0) + 1 
      })
      .eq('id', pollId);

    if (!error) {
      setVotedPolls(prev => new Set(prev).add(pollId));
    }
  };

  const togglePollStatus = async (pollId, currentStatus) => {
    if (!supabase) return;
    await supabase
      .from('polls')
      .update({ active: !currentStatus })
      .eq('id', pollId);
  };

  const deletePoll = async (pollId) => {
    if (!supabase) return;
    await supabase.from('polls').delete().eq('id', pollId);
  };

  const addOptionField = () => setNewOptions([...newOptions, '']);
  const updateOptionText = (index, val) => {
    const next = [...newOptions];
    next[index] = val;
    setNewOptions(next);
  };

  const activePolls = polls.filter(p => p.active);
  const archivedPolls = polls.filter(p => !p.active);

  if (!supabase && loading) {
    return (
      <div className="min-h-screen bg-slate-50 flex items-center justify-center">
        <div className="text-center">
          <div className="animate-spin rounded-full h-10 w-10 border-b-2 border-emerald-600 mx-auto mb-4"></div>
          <p className="text-slate-500 font-medium">Connecting to services...</p>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-slate-50 text-slate-900 font-sans p-4 md:p-8">
      {/* Header */}
      <header className="max-w-4xl mx-auto flex flex-col md:flex-row md:items-center justify-between gap-4 mb-10">
        <div>
          <h1 className="text-3xl font-bold tracking-tight text-emerald-600 flex items-center gap-2">
            <BarChart3 className="w-8 h-8" />
            MonthlyPolls
          </h1>
          <p className="text-slate-500 mt-1">Manage your community engagement.</p>
        </div>
        
        <div className="flex items-center gap-3">
          <button 
            onClick={() => setView('active')}
            className={`px-4 py-2 rounded-full text-sm font-medium transition-all ${view === 'active' ? 'bg-emerald-600 text-white shadow-md' : 'bg-white text-slate-600 shadow-sm border border-slate-200'}`}
          >
            Active
          </button>
          <button 
            onClick={() => setView('archive')}
            className={`px-4 py-2 rounded-full text-sm font-medium transition-all ${view === 'archive' ? 'bg-emerald-600 text-white shadow-md' : 'bg-white text-slate-600 shadow-sm border border-slate-200'}`}
          >
            Archives
          </button>
          <div className="h-6 w-[1px] bg-slate-300 mx-1" />
          <button 
            onClick={() => isAdmin ? setIsAdmin(false) : setShowAdminLogin(true)}
            className={`p-2 rounded-full transition-all border ${isAdmin ? 'bg-orange-100 border-orange-200 text-orange-600' : 'bg-white border-slate-200 text-slate-400 hover:border-emerald-500'}`}
            title={isAdmin ? "Logout Admin" : "Admin Login"}
          >
            {isAdmin ? <Unlock className="w-5 h-5" /> : <Lock className="w-5 h-5" />}
          </button>
        </div>
      </header>

      <main className="max-w-4xl mx-auto space-y-8">
        {/* Admin Login Modal */}
        {showAdminLogin && (
          <div className="fixed inset-0 bg-slate-900/40 backdrop-blur-sm z-50 flex items-center justify-center p-4">
            <div className="bg-white rounded-2xl p-6 w-full max-w-sm shadow-2xl animate-in zoom-in-95 duration-200">
              <div className="flex justify-between items-center mb-6">
                <h3 className="text-lg font-bold text-slate-800 flex items-center gap-2">
                  <Lock className="w-5 h-5 text-emerald-600" /> Admin Access
                </h3>
                <button onClick={() => setShowAdminLogin(false)} className="text-slate-400 hover:text-slate-600">
                  <X className="w-5 h-5" />
                </button>
              </div>
              <form onSubmit={handleAdminAuth} className="space-y-4">
                <div>
                  <label className="text-xs font-bold text-slate-400 uppercase tracking-wider mb-1 block">Secret Password</label>
                  <input 
                    type="password"
                    autoFocus
                    placeholder="Enter Password"
                    className="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:ring-2 focus:ring-emerald-500"
                    value={adminPassword}
                    onChange={(e) => setAdminPassword(e.target.value)}
                  />
                </div>
                <button 
                  type="submit"
                  className="w-full bg-emerald-600 text-white py-3 rounded-xl font-bold hover:bg-emerald-700 transition-colors shadow-lg shadow-emerald-100"
                >
                  Unlock Admin Tools
                </button>
              </form>
            </div>
          </div>
        )}

        {/* Database Error Alert */}
        {errorState && (
          <div className="bg-rose-50 border border-rose-200 p-4 rounded-xl flex items-start gap-3 text-rose-700">
            <AlertCircle className="w-5 h-5 mt-0.5 flex-shrink-0" />
            <div>
              <p className="font-bold">Database Message</p>
              <p className="text-sm opacity-90">{errorState}</p>
            </div>
          </div>
        )}

        {/* Admin Section: Create Poll */}
        {isAdmin && (
          <section className="bg-white border-2 border-orange-100 rounded-2xl p-6 shadow-sm animate-in fade-in slide-in-from-top-4 duration-500">
            <h2 className="text-xl font-semibold mb-4 text-orange-700 flex items-center gap-2">
              <Plus className="w-5 h-5" />
              Launch New Monthly Poll
            </h2>
            <div className="space-y-4">
              <input 
                type="text" 
                placeholder="What's the big question this month?"
                className="w-full p-4 bg-slate-50 border border-slate-200 rounded-xl focus:ring-2 focus:ring-emerald-500 outline-none transition-all"
                value={newQuestion}
                onChange={(e) => setNewQuestion(e.target.value)}
              />
              <div className="grid grid-cols-1 md:grid-cols-2 gap-3">
                {newOptions.map((opt, i) => (
                  <input 
                    key={i}
                    type="text" 
                    placeholder={`Option ${i + 1}`}
                    className="p-3 bg-slate-50 border border-slate-200 rounded-xl focus:ring-2 focus:ring-emerald-500 outline-none transition-all"
                    value={opt}
                    onChange={(e) => updateOptionText(i, e.target.value)}
                  />
                ))}
              </div>
              <div className="flex justify-between items-center pt-2">
                <button 
                  onClick={addOptionField}
                  className="text-sm text-emerald-600 font-semibold hover:text-emerald-700 flex items-center gap-1"
                >
                  <Plus className="w-4 h-4" /> Add Option
                </button>
                <button 
                  onClick={createPoll}
                  disabled={!newQuestion || newOptions.filter(o => o.trim()).length < 2}
                  className="bg-emerald-600 text-white px-8 py-3 rounded-xl font-bold hover:bg-emerald-700 shadow-lg shadow-emerald-100 disabled:opacity-50 disabled:cursor-not-allowed transition-all"
                >
                  Post to Live Site
                </button>
              </div>
            </div>
          </section>
        )}

        {/* Poll Listing */}
        <div className="space-y-6">
          {loading ? (
            <div className="flex flex-col items-center py-20 text-slate-400">
              <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-emerald-600 mb-4"></div>
              Refreshing polls...
            </div>
          ) : (
            (view === 'active' ? activePolls : archivedPolls).map((poll) => (
              <div key={poll.id} className="bg-white border border-slate-200 rounded-2xl shadow-sm overflow-hidden transition-all hover:border-emerald-200">
                <div className="p-6">
                  <div className="flex justify-between items-start mb-6">
                    <div>
                      <span className="text-xs font-bold uppercase tracking-widest text-emerald-500 mb-1 block">
                        {poll.month || 'Poll'}
                      </span>
                      <h3 className="text-2xl font-bold text-slate-800 leading-tight">{poll.question}</h3>
                    </div>
                    {isAdmin && (
                      <div className="flex gap-1">
                        <button 
                          onClick={() => togglePollStatus(poll.id, poll.active)}
                          className={`p-2 rounded-lg transition-colors ${poll.active ? 'text-slate-400 hover:bg-slate-100' : 'text-emerald-600 hover:bg-emerald-50'}`}
                          title={poll.active ? "Archive Poll" : "Re-activate Poll"}
                        >
                          {poll.active ? <Archive className="w-5 h-5" /> : <CheckCircle2 className="w-5 h-5" />}
                        </button>
                        <button 
                          onClick={() => deletePoll(poll.id)}
                          className="p-2 text-rose-400 hover:bg-rose-50 hover:text-rose-600 rounded-lg transition-colors"
                        >
                          <Trash2 className="w-5 h-5" />
                        </button>
                      </div>
                    )}
                  </div>

                  <div className="space-y-4">
                    {poll.options.map((option, idx) => {
                      const percentage = poll.total_votes > 0 ? Math.round((option.votes / poll.total_votes) * 100) : 0;
                      const hasVoted = votedPolls.has(poll.id) || !poll.active;

                      return (
                        <div key={idx} className="relative h-14">
                          <button
                            disabled={hasVoted}
                            onClick={() => handleVote(poll.id, idx)}
                            className={`w-full h-full relative z-10 px-5 rounded-xl border text-left transition-all flex items-center justify-between font-semibold ${
                              hasVoted 
                                ? 'border-transparent cursor-default' 
                                : 'border-slate-200 hover:border-emerald-500 hover:bg-emerald-50'
                            }`}
                          >
                            <span className={hasVoted ? 'text-slate-800' : 'text-slate-600'}>
                              {option.text}
                            </span>
                            {hasVoted && (
                              <span className="text-sm font-black text-emerald-700 bg-white/50 px-2 py-0.5 rounded-md">{percentage}%</span>
                            )}
                            {!hasVoted && (
                              <ChevronRight className="w-4 h-4 text-emerald-300" />
                            )}
                          </button>
                          
                          {/* Progress Bar Background */}
                          {hasVoted && (
                            <div className="absolute inset-0 z-0 bg-slate-100 rounded-xl overflow-hidden">
                              <div 
                                className={`h-full transition-all duration-700 ease-out rounded-r-lg ${poll.active ? 'bg-emerald-200' : 'bg-slate-300'}`}
                                style={{ width: `${percentage}%` }}
                              />
                            </div>
                          )}
                        </div>
                      );
                    })}
                  </div>

                  <div className="mt-8 pt-4 border-t border-slate-50 flex items-center justify-between text-slate-400 text-sm">
                    <div className="flex items-center gap-4">
                      <div className="flex items-center gap-1.5">
                        <Clock className="w-4 h-4" />
                        <span className="font-medium text-slate-500">{poll.total_votes || 0} participants</span>
                      </div>
                    </div>
                    
                    {!poll.active ? (
                      <span className="flex items-center gap-1 text-slate-500 font-bold bg-slate-100 px-3 py-1 rounded-full uppercase text-[10px]">
                        Results Finalized
                      </span>
                    ) : votedPolls.has(poll.id) ? (
                      <span className="text-emerald-600 font-bold flex items-center gap-1 bg-emerald-50 px-3 py-1 rounded-full uppercase text-[10px]">
                        <CheckCircle2 className="w-3 h-3" /> Thank you for voting
                      </span>
                    ) : (
                      <span className="text-emerald-500 font-bold animate-pulse text-[10px] uppercase">
                        Vote Now
                      </span>
                    )}
                  </div>
                </div>
              </div>
            ))
          )}

          {!loading && (view === 'active' ? activePolls : archivedPolls).length === 0 && !errorState && (
            <div className="text-center py-24 bg-white border-2 border-dashed border-slate-200 rounded-3xl">
              <BarChart3 className="w-16 h-16 text-slate-200 mx-auto mb-4" />
              <p className="text-slate-400 font-medium text-lg">No {view} polls for this month.</p>
              {isAdmin && view === 'active' && (
                <button 
                   onClick={() => window.scrollTo({top: 0, behavior: 'smooth'})}
                   className="mt-4 text-emerald-600 font-bold hover:underline"
                >
                  Create one to start the conversation!
                </button>
              )}
            </div>
          )}
        </div>
      </main>

      <footer className="max-w-4xl mx-auto mt-20 pb-10 text-center text-slate-400 text-xs border-t border-slate-200 pt-8">
        &copy; {new Date().getFullYear()} MonthlyPolls • Powered by Supabase Real-time
      </footer>
    </div>
  );
}
