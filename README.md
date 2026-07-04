# ApexPlanet-tast3

Author- Prem Somnath Sonawane

TASK3:

Advanced Features Implementation
Enhance the application with advanced features like search and pagination.
Improve the user interface.
Add a search form to allow users to search posts by title or content.
Implement PHP code to perform the search query and display results.
Search Functionality :
Objective :
Implement pagination for the posts listing page.
Ensure the page displays a limited number of posts per page with navigation links to other pages.
Pagination :
Use CSS to style the application for better UX.
Optionally, integrate a front-end framework like Bootstrap for responsive design.
User Interface Improvements :
Updated application with search and pagination.
Improved user interface design.



import React, { useState, useMemo } from 'react';
import { 
  FileText, 
  Terminal, 
  Database, 
  Lock, 
  Eye, 
  Search, 
  RefreshCw, 
  Layers, 
  Plus, 
  Check, 
  Copy, 
  Download, 
  ShieldCheck, 
  AlertTriangle, 
  Clock, 
  ArrowRight,
  BookOpen,
  Info
} from 'lucide-react';
import { INITIAL_POSTS, BlogPost } from './data/postsData';
import { sourceFiles, CodeFile } from './data/sourceCodes';

export default function App() {
  // Navigation & Sub-tab controls
  const [activeTab, setActiveTab] = useState<'live' | 'code' | 'security'>('live');
  const [activeCodeFile, setActiveCodeFile] = useState<number>(1); // Index of index.php in sourceFiles (1)
  
  // Blog State Engine (replicates index.php logic client-side)
  const [posts, setPosts] = useState<BlogPost[]>(INITIAL_POSTS);
  const [searchQuery, setSearchQuery] = useState<string>('');
  const [tempSearchQuery, setTempSearchQuery] = useState<string>('');
  const [currentPage, setCurrentPage] = useState<number>(1);
  const postsPerPage = 5;

  // New Post Creator modal/form state
  const [showAddModal, setShowAddModal] = useState<boolean>(false);
  const [newTitle, setNewTitle] = useState<string>('');
  const [newContent, setNewContent] = useState<string>('');
  const [newCategory, setNewCategory] = useState<string>('Security');
  
  // Custom notification state
  const [notification, setNotification] = useState<string | null>(null);

  // Copy status map for files
  const [copiedFileIndex, setCopiedFileIndex] = useState<number | null>(null);

  // Security Labs states
  const [sqlPayload, setSqlPayload] = useState<string>("' OR '1'='1");
  const [xssPayload, setXssPayload] = useState<string>("<script>alert('Hacked!');</script>");
  const [testPostTitle, setTestPostTitle] = useState<string>("Hacker's Article <img src=x onerror=alert('XSS!')>");

  // --- Real-time Blog Logic Simulation ---
  // Replicates filtering in PHP: WHERE title LIKE :search OR content LIKE :search
  const filteredPosts = useMemo(() => {
    if (!searchQuery.trim()) {
      return posts;
    }
    const query = searchQuery.toLowerCase().trim();
    return posts.filter(
      p => p.title.toLowerCase().includes(query) || p.content.toLowerCase().includes(query)
    );
  }, [posts, searchQuery]);

  const totalPages = useMemo(() => {
    const pages = Math.ceil(filteredPosts.length / postsPerPage);
    return pages < 1 ? 1 : pages;
  }, [filteredPosts]);

  // Handle auto-adjusting page boundary
  const validatedPage = useMemo(() => {
    if (currentPage > totalPages) return totalPages;
    if (currentPage < 1) return 1;
    return currentPage;
  }, [currentPage, totalPages]);

  const paginatedPosts = useMemo(() => {
    const startIndex = (validatedPage - 1) * postsPerPage;
    return filteredPosts.slice(startIndex, startIndex + postsPerPage);
  }, [filteredPosts, validatedPage]);

  // Execute search (simulates form submission in index.php)
  const handleSearchSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    setSearchQuery(tempSearchQuery);
    setCurrentPage(1); // Reset to page 1 upon search
  };

  // Reset search
  const handleResetSearch = () => {
    setSearchQuery('');
    setTempSearchQuery('');
    setCurrentPage(1);
  };

  // Create custom post (simulates INSERT INTO database)
  const handleCreatePost = (e: React.FormEvent) => {
    e.preventDefault();
    if (!newTitle.trim() || !newContent.trim()) {
      alert("Please fill in all fields.");
      return;
    }

    const newPost: BlogPost = {
      id: posts.length > 0 ? Math.max(...posts.map(p => p.id)) + 1 : 1,
      title: newTitle,
      content: newContent,
      created_at: new Date().toISOString(),
      category: newCategory
    };

    setPosts([newPost, ...posts]);
    setNewTitle('');
    setNewContent('');
    setShowAddModal(false);
    triggerNotification(`Successfully inserted post #${newPost.id} into simulated database table!`);
  };

  const triggerNotification = (message: string) => {
    setNotification(message);
    setTimeout(() => {
      setNotification(null);
    }, 4000);
  };

  const handleCopyCode = (content: string, index: number) => {
    navigator.clipboard.writeText(content);
    setCopiedFileIndex(index);
    setTimeout(() => {
      setCopiedFileIndex(null);
    }, 2000);
  };

  const handleDownloadAll = () => {
    // Generate a shell script, composite files, or explain how to retrieve
    const zipExplanation = `
# How to run this PHP blogging application locally:
1. Ensure PHP 8.x and MySQL (e.g. XAMPP, MAMP, or Docker) are installed on your machine.
2. Create a database called 'blog_db' in phpMyAdmin or MySQL terminal.
3. Import the contents of 'schema.sql' to create the 'posts' table and seed data.
4. Place 'db.php' and 'index.php' into your server root directory (e.g., htdocs/).
5. Open 'db.php' and adjust the connection constants (DB_HOST, DB_NAME, DB_USER, DB_PASS) to match your local environment.
6. Open your browser and navigate to: http://localhost/index.php
`;
    
    // Create element to download explanation text + files
    const blob = new Blob(
      [
        "=== PHP BLOGGING APP INSTALLATION GUIDE ===\n",
        zipExplanation,
        "\n\n=== FILE 1: db.php ===\n",
        sourceFiles[0].content,
        "\n\n=== FILE 2: index.php ===\n",
        sourceFiles[1].content,
        "\n\n=== FILE 3: schema.sql ===\n",
        sourceFiles[2].content
      ],
      { type: "text/plain;charset=utf-8" }
    );
    const url = URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.href = url;
    link.download = "php-blog-application.txt";
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  };

  return (
    <div className="min-h-screen bg-slate-50 text-slate-800 flex flex-col font-sans antialiased">
      
      {/* Top Main Navigation Header */}
      <header className="bg-white border-b border-slate-200 sticky top-0 z-50 px-6 py-4">
        <div className="max-w-7xl mx-auto flex flex-col md:flex-row items-center justify-between gap-4">
          
          <div className="flex items-center gap-3">
            <div className="p-2.5 bg-indigo-600 rounded-none text-white flex items-center justify-center">
              <Layers className="w-6 h-6" />
            </div>
            <div>
              <div className="flex flex-wrap items-center gap-2">
                <span className="font-mono text-[9px] font-bold tracking-widest uppercase bg-emerald-50 text-emerald-700 px-2 py-0.5 rounded-none border border-emerald-200">
                  Full-Stack PHP
                </span>
                <span className="font-mono text-[9px] font-bold tracking-widest uppercase bg-slate-100 text-slate-700 px-2 py-0.5 rounded-none border border-slate-200">
                  Bootstrap 5
                </span>
              </div>
              <h1 className="text-base font-black tracking-widest text-slate-900 uppercase mt-1">PHP Blogging Sandbox</h1>
            </div>
          </div>

          {/* Tab Navigation Controls */}
          <nav className="flex bg-slate-100 p-1 rounded-none border border-slate-200">
            <button
              onClick={() => setActiveTab('live')}
              className={`flex items-center gap-2 px-4 py-2 rounded-none text-xs font-bold uppercase tracking-wider transition-all cursor-pointer ${
                activeTab === 'live'
                  ? 'bg-indigo-600 text-white shadow-none'
                  : 'text-slate-600 hover:text-slate-950'
              }`}
            >
              <Eye className="w-3.5 h-3.5" />
              <span>Interactive App Sandbox</span>
            </button>
            <button
              onClick={() => setActiveTab('code')}
              className={`flex items-center gap-2 px-4 py-2 rounded-none text-xs font-bold uppercase tracking-wider transition-all cursor-pointer ${
                activeTab === 'code'
                  ? 'bg-indigo-600 text-white shadow-none'
                  : 'text-slate-600 hover:text-slate-950'
              }`}
            >
              <FileText className="w-3.5 h-3.5" />
              <span>Source Files</span>
            </button>
            <button
              onClick={() => setActiveTab('security')}
              className={`flex items-center gap-2 px-4 py-2 rounded-none text-xs font-bold uppercase tracking-wider transition-all cursor-pointer ${
                activeTab === 'security'
                  ? 'bg-indigo-600 text-white shadow-none'
                  : 'text-slate-600 hover:text-slate-950'
              }`}
            >
              <Lock className="w-3.5 h-3.5" />
              <span>PHP Security Labs</span>
            </button>
          </nav>

          {/* Quick Actions */}
          <div className="hidden lg:flex items-center gap-3">
            <button 
              onClick={handleDownloadAll}
              className="flex items-center gap-2 px-4 py-2 bg-slate-900 hover:bg-slate-800 active:bg-black border border-slate-900 rounded-none text-xs font-bold uppercase tracking-wider text-white transition-all cursor-pointer"
            >
              <Download className="w-3.5 h-3.5" />
              <span>Download PHP Bundle</span>
            </button>
          </div>

        </div>
      </header>

      {/* Real-time Event Notification Toast */}
      {notification && (
        <div className="fixed bottom-6 right-6 z-50 bg-white text-slate-900 px-5 py-4 rounded-none font-bold shadow-lg border-l-4 border-emerald-500 border border-slate-200 flex items-center gap-3 animate-slide-in">
          <ShieldCheck className="w-5 h-5 flex-shrink-0 text-emerald-600" />
          <span className="text-xs uppercase tracking-wide">{notification}</span>
        </div>
      )}

      {/* Main Sandbox Container */}
      <main className="flex-1 max-w-7xl w-full mx-auto px-4 md:px-6 py-8">
        
        {/* TAB 1: INTERACTIVE LIVE APP SANDBOX */}
        {activeTab === 'live' && (
          <div className="grid grid-cols-1 lg:grid-cols-12 gap-8 items-start">
            
            {/* LEFT COLUMN: THE SIMULATED BROWSER / IFRAME */}
            <div className="lg:col-span-8 flex flex-col gap-6">
              
              {/* Virtual Browser Mock Frame */}
              <div className="bg-white rounded-none border-2 border-slate-300 overflow-hidden shadow-md">
                
                {/* Browser Controls Header */}
                <div className="bg-slate-100 px-4 py-3 flex items-center justify-between border-b border-slate-200">
                  <div className="flex items-center gap-1.5">
                    <span className="w-2.5 h-2.5 rounded-full bg-red-400"></span>
                    <span className="w-2.5 h-2.5 rounded-full bg-yellow-400"></span>
                    <span className="w-2.5 h-2.5 rounded-full bg-emerald-400"></span>
                  </div>
                  <div className="flex-1 mx-4 max-w-xl">
                    <div className="bg-white rounded-none py-1.5 px-3 border border-slate-300 flex items-center justify-between text-xs font-mono text-slate-600 select-all">
                      <div className="flex items-center gap-2 truncate">
                        <span className="text-indigo-600 font-bold">http</span>
                        <span className="text-slate-400">://</span>
                        <span>localhost/blog/index.php</span>
                        {searchQuery && (
                          <span className="text-indigo-500 truncate font-semibold">
                            ?search={encodeURIComponent(searchQuery)}&page={validatedPage}
                          </span>
                        )}
                        {!searchQuery && validatedPage > 1 && (
                          <span className="text-indigo-500 font-semibold">?page={validatedPage}</span>
                        )}
                      </div>
                      <span className="text-[9px] font-bold bg-indigo-50 text-indigo-700 px-1.5 py-0.5 rounded-none border border-indigo-200">
                        GET
                      </span>
                    </div>
                  </div>
                  <div className="flex items-center gap-2">
                    <button 
                      onClick={handleResetSearch}
                      title="Reload application"
                      className="p-1.5 text-slate-500 hover:text-slate-850 hover:bg-slate-200 rounded-none transition cursor-pointer"
                    >
                      <RefreshCw className="w-3.5 h-3.5" />
                    </button>
                  </div>
                </div>

                {/* --- VIRTUAL APP BODY (Simulating Bootstrap 5 Frontend) --- */}
                <div className="bg-slate-50 text-slate-800 min-h-[580px] flex flex-col font-sans">
                  
                  {/* Virtual Bootstrap Nav */}
                  <nav className="bg-white border-b border-slate-200 px-6 py-4 flex items-center justify-between">
                    <div className="flex items-center gap-2 select-none">
                      <div className="text-slate-900 font-extrabold tracking-widest uppercase flex items-center gap-1.5 text-xs">
                        <div className="w-3 h-3 bg-indigo-600 rounded-none"></div>
                        <span>NexusCMS Blog</span>
                      </div>
                      <span className="text-[9px] font-bold tracking-widest bg-slate-100 text-slate-600 px-2 py-0.5 rounded-none border border-slate-200 font-mono">
                        V5.3
                      </span>
                    </div>
                    <div className="flex items-center gap-4 text-xs">
                      <button 
                        onClick={handleResetSearch}
                        className="text-slate-600 hover:text-indigo-600 font-bold uppercase tracking-wider transition cursor-pointer"
                      >
                        Home
                      </button>
                      <button 
                        onClick={() => setShowAddModal(true)}
                        className="bg-indigo-600 hover:bg-indigo-700 active:bg-indigo-800 text-white px-4 py-2 rounded-none flex items-center gap-1.5 font-bold uppercase tracking-wider text-[10px] transition cursor-pointer"
                      >
                        <Plus className="w-3.5 h-3.5" />
                        <span>New Post</span>
                      </button>
                    </div>
                  </nav>

                  {/* Virtual Bootstrap App Content Container */}
                  <div className="flex-1 p-6 max-w-3xl mx-auto w-full">
                    
                    {/* Welcome Jumbotron */}
                    <div className="text-center mb-8 mt-2">
                      <h2 className="text-xl font-black uppercase text-slate-900 tracking-widest">Developer Knowledge Hub</h2>
                      <p className="text-xs text-slate-500 tracking-wide mt-1">Insights, secure code snippets, and modern web engineering articles.</p>
                    </div>

                    {/* Simulated Search Form Container */}
                    <div className="bg-white p-5 rounded-none border border-slate-300 mb-6">
                      <form onSubmit={handleSearchSubmit} className="flex flex-col sm:flex-row gap-3">
                        <div className="flex-1 relative">
                          <span className="absolute inset-y-0 left-0 pl-3.5 flex items-center text-slate-400 pointer-events-none">
                            <Search className="w-4 h-4" />
                          </span>
                          <input 
                            type="text"
                            placeholder="Search articles by title or content..."
                            className="w-full bg-slate-50 border border-slate-300 text-slate-900 text-sm rounded-none pl-10 pr-4 py-2.5 focus:outline-none focus:border-indigo-600 focus:bg-white transition"
                            value={tempSearchQuery}
                            onChange={(e) => setTempSearchQuery(e.target.value)}
                          />
                        </div>
                        <div className="flex gap-2">
                          <button 
                            type="submit"
                            className="bg-indigo-600 hover:bg-indigo-700 active:bg-indigo-800 text-white font-bold text-xs uppercase tracking-wider px-6 py-2.5 rounded-none transition flex items-center gap-1 cursor-pointer"
                          >
                            <span>Search</span>
                          </button>
                          {searchQuery && (
                            <button
                              type="button"
                              onClick={handleResetSearch}
                              title="Reset and clear search query"
                              className="bg-slate-100 hover:bg-slate-200 active:bg-slate-300 text-slate-600 border border-slate-300 p-2.5 rounded-none transition cursor-pointer"
                            >
                              <RefreshCw className="w-4 h-4" />
                            </button>
                          )}
                        </div>
                      </form>

                      {/* Display Search Metadata badge */}
                      {searchQuery && (
                        <div className="mt-4 flex items-center justify-between text-xs text-slate-500 border-t border-slate-200 pt-3">
                          <div className="tracking-wide">
                            Found <span className="font-bold text-slate-900">{filteredPosts.length}</span> matches for <span className="font-mono bg-slate-100 px-1.5 py-0.5 rounded-none text-red-600 border border-slate-200 font-semibold">"{searchQuery}"</span>
                          </div>
                          <span className="bg-slate-100 text-slate-700 px-2.5 py-0.5 rounded-none text-[10px] border border-slate-200 uppercase font-bold tracking-wide">
                            Page {validatedPage} of {totalPages}
                          </span>
                        </div>
                      )}
                    </div>

                    {/* Article Cards Grid */}
                    <div className="flex flex-col gap-5">
                      {paginatedPosts.length === 0 ? (
                        <div className="text-center bg-white border border-slate-300 rounded-none p-10">
                          <div className="w-12 h-12 bg-slate-100 rounded-none border border-slate-200 flex items-center justify-center mx-auto mb-3 text-slate-400">
                            <Search className="w-6 h-6" />
                          </div>
                          <h4 className="font-black text-slate-900 uppercase tracking-widest text-sm">No articles found</h4>
                          <p className="text-slate-500 text-xs mt-1 max-w-md mx-auto leading-relaxed">
                            We couldn't find any articles matching "{searchQuery}". Try modifying your search keywords or resetting the filter.
                          </p>
                          <button
                            onClick={handleResetSearch}
                            className="mt-5 bg-indigo-600 text-white text-[10px] px-5 py-2.5 rounded-none font-bold uppercase tracking-wider hover:bg-indigo-700 transition cursor-pointer"
                          >
                            Show All Articles
                          </button>
                        </div>
                      ) : (
                        paginatedPosts.map((post) => (
                          <article key={post.id} className="bg-white border border-slate-300 rounded-none p-6 hover:border-indigo-600 hover:shadow-lg transition-all duration-200 flex flex-col justify-between">
                            <div>
                              <div className="flex items-center justify-between text-xs text-slate-400 mb-3">
                                <span className="flex items-center gap-1.5 font-mono text-[10px]">
                                  <Clock className="w-3.5 h-3.5 text-slate-400" />
                                  <span>{new Date(post.created_at).toLocaleDateString('en-US', {month: 'long', day: 'numeric', year: 'numeric'})}</span>
                                </span>
                                <span className="bg-indigo-50 text-indigo-600 px-2.5 py-0.5 rounded-none font-bold border border-indigo-200 text-[9px] tracking-wider uppercase">
                                  {post.category}
                                </span>
                              </div>

                              <h3 className="text-lg font-black text-slate-950 mb-3 hover:text-indigo-600 uppercase tracking-tight transition">
                                {post.title}
                              </h3>

                              <p className="text-xs text-slate-600 leading-relaxed mb-4">
                                {post.content.length > 200 ? `${post.content.substring(0, 200)}...` : post.content}
                              </p>
                            </div>

                            <div className="flex items-center justify-between border-t border-slate-100 pt-3 mt-1">
                              <span className="text-[10px] font-extrabold uppercase tracking-widest text-indigo-600 hover:text-indigo-850 transition flex items-center gap-1 cursor-pointer">
                                <span>Read full article</span>
                                <ArrowRight className="w-3 h-3 text-indigo-600" />
                              </span>
                              <span className="text-[9px] font-mono text-slate-400 uppercase font-bold tracking-wider">
                                ID: <span className="bg-slate-100 px-1.5 py-0.5 rounded-none border border-slate-200 font-mono text-slate-600">#{post.id}</span>
                              </span>
                            </div>
                          </article>
                        ))
                      )}
                    </div>

                    {/* Virtual Pagination Component */}
                    {totalPages > 1 && (
                      <nav className="mt-8 flex justify-center" aria-label="Page navigation">
                        <ul className="inline-flex bg-white border border-slate-300 rounded-none p-1 space-x-1">
                          
                          {/* Previous Button */}
                          <li>
                            <button
                              disabled={validatedPage === 1}
                              onClick={() => setCurrentPage(validatedPage - 1)}
                              className={`px-4 py-2 rounded-none text-xs font-bold uppercase tracking-wider flex items-center transition ${
                                validatedPage === 1
                                  ? 'text-slate-300 cursor-not-allowed bg-transparent'
                                  : 'text-slate-600 hover:bg-slate-100 active:bg-slate-200 cursor-pointer'
                              }`}
                            >
                              Previous
                            </button>
                          </li>

                          {/* Numeric pages */}
                          {Array.from({ length: totalPages }, (_, idx) => {
                            const pNum = idx + 1;
                            const isActive = pNum === validatedPage;
                            return (
                              <li key={pNum}>
                                <button
                                  onClick={() => setCurrentPage(pNum)}
                                  className={`w-9 h-9 rounded-none text-xs font-bold transition border cursor-pointer ${
                                    isActive
                                      ? 'bg-indigo-600 text-white border-indigo-600 font-black'
                                      : 'text-slate-600 hover:bg-slate-100 active:bg-slate-200 border-transparent'
                                  }`}
                                >
                                  {pNum}
                                </button>
                              </li>
                            );
                          })}

                          {/* Next Button */}
                          <li>
                            <button
                              disabled={validatedPage === totalPages}
                              onClick={() => setCurrentPage(validatedPage + 1)}
                              className={`px-4 py-2 rounded-none text-xs font-bold uppercase tracking-wider flex items-center transition ${
                                validatedPage === totalPages
                                  ? 'text-slate-300 cursor-not-allowed bg-transparent'
                                  : 'text-slate-600 hover:bg-slate-100 active:bg-slate-200 cursor-pointer'
                              }`}
                            >
                              Next
                            </button>
                          </li>

                        </ul>
                      </nav>
                    )}

                  </div>

                  {/* Virtual Footer */}
                  <footer className="bg-slate-900 text-slate-400 text-center py-5 border-t border-slate-850 text-[10px] tracking-wider uppercase font-bold mt-auto">
                    <p>&copy; 2026 Developer Knowledge Hub. Protected against SQLi and XSS.</p>
                  </footer>

                </div>
              </div>

              {/* PHP Script Execution Console Logs Panel */}
              <div className="bg-white border border-slate-300 rounded-none p-6">
                <div className="flex items-center justify-between border-b border-slate-200 pb-3 mb-4">
                  <div className="flex items-center gap-2">
                    <Terminal className="w-5 h-5 text-indigo-600" />
                    <h3 className="font-bold text-sm text-slate-900 uppercase tracking-widest">PHP Server & Database Process Tracer</h3>
                  </div>
                  <span className="font-mono text-[9px] bg-slate-50 text-slate-600 px-2 py-0.5 rounded-none border border-slate-200 uppercase font-semibold">
                    Live PHP SQL Tracing
                  </span>
                </div>

                <div className="space-y-3 font-mono text-xs text-slate-300 bg-slate-950 p-4 rounded-none border border-slate-900 max-h-[220px] overflow-y-auto">
                  <div className="text-blue-400">
                    <span className="text-slate-500">[2026-07-03 23:10:48]</span> PHP INIT: Require_once "db.php" successful. Established PDO connection to blog_db mysql host.
                  </div>
                  <div>
                    <span className="text-slate-500">[2026-07-03 23:10:48]</span> PHP GETPARAMS: <span className="text-yellow-400">page = {validatedPage}</span>, <span className="text-yellow-400">search = "{searchQuery}"</span>
                  </div>
                  
                  {searchQuery ? (
                    <>
                      <div className="text-emerald-400">
                        <span className="text-slate-500">[2026-07-03 23:10:48]</span> SQL PREPARE (Count Filtered): 
                        <span className="text-slate-100 select-all block mt-1 bg-slate-900 p-2 rounded-none text-[11px] border border-slate-800">
                          "SELECT COUNT(*) FROM `posts` WHERE `title` LIKE :search OR `content` LIKE :search"
                        </span>
                      </div>
                      <div className="text-purple-400">
                        <span className="text-slate-500">[2026-07-03 23:10:48]</span> PDO BIND: <span className="text-yellow-400">:search</span> = <span className="text-emerald-300">"%{searchQuery}%"</span>
                      </div>
                      <div className="text-indigo-400">
                        <span className="text-slate-500">[2026-07-03 23:10:48]</span> SQL EXECUTE COUNT: Returned <span className="font-bold text-slate-100">{filteredPosts.length}</span> records. Calculated Total Pages: <span className="text-white font-bold">{totalPages}</span>.
                      </div>
                      <div className="text-emerald-400">
                        <span className="text-slate-500">[2026-07-03 23:10:48]</span> SQL PREPARE (Fetch Filtered Data): 
                        <span className="text-slate-100 select-all block mt-1 bg-slate-900 p-2 rounded-none text-[11px] border border-slate-800">
                          "SELECT `id`, `title`, `content`, `created_at` FROM `posts` WHERE `title` LIKE :search OR `content` LIKE :search ORDER BY `created_at` DESC LIMIT :limit OFFSET :offset"
                        </span>
                      </div>
                      <div className="text-purple-400">
                        <span className="text-slate-500">[2026-07-03 23:10:48]</span> PDO BIND PARAMETERS: 
                        <ul className="list-disc list-inside ml-2 text-[11px] text-slate-300">
                          <li><span className="text-yellow-400">:search</span> = <span className="text-emerald-300">"%{searchQuery}%"</span></li>
                          <li><span className="text-yellow-400">:limit</span> = <span className="text-orange-300">{postsPerPage}</span> (INT)</li>
                          <li><span className="text-yellow-400">:offset</span> = <span className="text-orange-300">{(validatedPage - 1) * postsPerPage}</span> (INT)</li>
                        </ul>
                      </div>
                    </>
                  ) : (
                    <>
                      <div className="text-indigo-400">
                        <span className="text-slate-500">[2026-07-03 23:10:48]</span> SQL EXECUTE (Count Unfiltered): 
                        <span className="text-slate-100 block bg-slate-900 p-1.5 rounded-none text-[11px] border border-slate-800 font-mono">
                          "SELECT COUNT(*) FROM `posts`"
                        </span>
                        Returned <span className="font-bold text-slate-100">{filteredPosts.length}</span> total records. Calculated Total Pages: <span className="text-white font-bold">{totalPages}</span>.
                      </div>
                      <div className="text-emerald-400">
                        <span className="text-slate-500">[2026-07-03 23:10:48]</span> SQL PREPARE (Fetch Unfiltered Pagination): 
                        <span className="text-slate-100 select-all block mt-1 bg-slate-900 p-2 rounded-none text-[11px] border border-slate-800">
                          "SELECT `id`, `title`, `content`, `created_at` FROM `posts` ORDER BY `created_at` DESC LIMIT :limit OFFSET :offset"
                        </span>
                      </div>
                      <div className="text-purple-400">
                        <span className="text-slate-500">[2026-07-03 23:10:48]</span> PDO BIND PARAMETERS: 
                        <ul className="list-disc list-inside ml-2 text-[11px] text-slate-300">
                          <li><span className="text-yellow-400">:limit</span> = <span className="text-orange-300">{postsPerPage}</span> (INT)</li>
                          <li><span className="text-yellow-400">:offset</span> = <span className="text-orange-300">{(validatedPage - 1) * postsPerPage}</span> (INT)</li>
                        </ul>
                      </div>
                    </>
                  )}
                  
                  <div className="text-emerald-500 font-bold">
                    <span className="text-slate-500">[2026-07-03 23:10:48]</span> PHP HTML RENDER: Rendering {paginatedPosts.length} post cards safely with htmlspecialchars() to protect user inputs.
                  </div>
                </div>
              </div>

            </div>

            {/* RIGHT COLUMN: SANDBOX MANAGEMENT & EXPLANATORY TEXT */}
            <div className="lg:col-span-4 flex flex-col gap-6">
              
              {/* Database Schema Status Inspector */}
              <div className="bg-white border border-slate-300 rounded-none p-6">
                <div className="flex items-center gap-2 border-b border-slate-200 pb-3 mb-4">
                  <Database className="w-5 h-5 text-indigo-600" />
                  <h3 className="font-bold text-sm text-slate-900 uppercase tracking-widest">Database Status Inspector</h3>
                </div>

                <div className="space-y-4">
                  <div>
                    <span className="text-[10px] text-slate-500 font-bold uppercase tracking-wider block">Active Database Engine</span>
                    <span className="text-sm font-extrabold text-slate-900 mt-1 block">MySQL (InnoDB)</span>
                  </div>
                  <div>
                    <span className="text-[10px] text-slate-500 font-bold uppercase tracking-wider block">Primary Database Table</span>
                    <span className="text-sm font-mono font-bold text-indigo-600 mt-0.5 block">posts</span>
                  </div>
                  <div className="grid grid-cols-2 gap-3 bg-slate-50 p-3 rounded-none border border-slate-200">
                    <div>
                      <span className="text-[9px] text-slate-500 font-bold block uppercase tracking-wide">Total Rows</span>
                      <span className="text-lg font-black text-slate-900">{posts.length}</span>
                    </div>
                    <div>
                      <span className="text-[9px] text-slate-500 font-bold block uppercase tracking-wide">Filtered Rows</span>
                      <span className="text-lg font-black text-indigo-600">{filteredPosts.length}</span>
                    </div>
                  </div>

                  <div className="border-t border-slate-200 pt-4">
                    <h4 className="text-xs font-bold text-slate-900 uppercase tracking-wider mb-2">Simulated SQL Schema Properties</h4>
                    <ul className="space-y-2 text-xs text-slate-600 font-mono">
                      <li className="flex items-center justify-between">
                        <span className="font-bold">id</span>
                        <span className="text-slate-500 text-[11px] font-medium">INT UNSIGNED (PK, AI)</span>
                      </li>
                      <li className="flex items-center justify-between">
                        <span className="font-bold">title</span>
                        <span className="text-slate-500 text-[11px] font-medium">VARCHAR(255) (NOT NULL)</span>
                      </li>
                      <li className="flex items-center justify-between">
                        <span className="font-bold">content</span>
                        <span className="text-slate-500 text-[11px] font-medium">TEXT (NOT NULL)</span>
                      </li>
                      <li className="flex items-center justify-between">
                        <span className="font-bold">created_at</span>
                        <span className="text-slate-500 text-[11px] font-medium">DATETIME (DEFAULT CURRENT_TIMESTAMP)</span>
                      </li>
                    </ul>
                  </div>

                  <button 
                    onClick={() => setShowAddModal(true)}
                    className="w-full bg-slate-900 hover:bg-slate-800 active:bg-black text-white font-bold text-xs uppercase tracking-wider py-3 rounded-none transition border border-slate-900 flex items-center justify-center gap-2 cursor-pointer"
                  >
                    <Plus className="w-4 h-4" />
                    <span>Insert Simulated Post</span>
                  </button>

                </div>
              </div>

              {/* Developer Concept Checklist card */}
              <div className="bg-white border border-slate-300 rounded-none p-6 text-slate-700">
                <div className="flex items-center gap-2 border-b border-slate-200 pb-3 mb-4">
                  <ShieldCheck className="w-5 h-5 text-indigo-600" />
                  <h3 className="font-bold text-sm text-slate-900 uppercase tracking-widest">Security & Code Compliance</h3>
                </div>
                <p className="text-xs text-slate-500 leading-relaxed mb-4">
                  The accompanying PHP code provides robust defenses that you can test live inside the sandbox:
                </p>
                <div className="space-y-4">
                  <div className="flex items-start gap-2.5">
                    <div className="p-1 bg-emerald-50 text-emerald-600 rounded-none border border-emerald-200 mt-0.5">
                      <Check className="w-3.5 h-3.5" />
                    </div>
                    <div>
                      <h4 className="text-xs font-bold text-slate-900 uppercase tracking-wide">Parameterized Search</h4>
                      <p className="text-[11px] text-slate-500 leading-normal mt-0.5">
                        Prepared statements with <code className="font-mono text-indigo-600 font-bold">LIKE :search</code> bind variables securely. The SQL processor never interprets text payloads as code commands.
                      </p>
                    </div>
                  </div>
                  <div className="flex items-start gap-2.5">
                    <div className="p-1 bg-emerald-50 text-emerald-600 rounded-none border border-emerald-200 mt-0.5">
                      <Check className="w-3.5 h-3.5" />
                    </div>
                    <div>
                      <h4 className="text-xs font-bold text-slate-900 uppercase tracking-wide">Dynamic Pagination Range</h4>
                      <p className="text-[11px] text-slate-500 leading-normal mt-0.5">
                        Limits datasets to <code className="font-mono text-indigo-600 font-bold">LIMIT 5</code>. Prevents server strain and denial of service attacks by dynamically sanitizing <code className="font-mono text-indigo-600 font-bold">page</code> offsets.
                      </p>
                    </div>
                  </div>
                  <div className="flex items-start gap-2.5">
                    <div className="p-1 bg-emerald-50 text-emerald-600 rounded-none border border-emerald-200 mt-0.5">
                      <Check className="w-3.5 h-3.5" />
                    </div>
                    <div>
                      <h4 className="text-xs font-bold text-slate-900 uppercase tracking-wide">Cross-Site Scripting (XSS) Shield</h4>
                      <p className="text-[11px] text-slate-500 leading-normal mt-0.5">
                        Using PHP's native <code className="font-mono text-indigo-600 font-bold">htmlspecialchars()</code> escapes dangerous tags like <code className="font-mono text-red-600">&lt;script&gt;</code> before output.
                      </p>
                    </div>
                  </div>
                </div>
              </div>

            </div>

          </div>
        )}

        {/* TAB 2: CODE EXPLORER */}
        {activeTab === 'code' && (
          <div className="grid grid-cols-1 lg:grid-cols-12 gap-8">
            
            {/* LEFT SIDEBAR: FILE SELECTOR */}
            <div className="lg:col-span-3 flex flex-col gap-4">
              <div className="bg-white border border-slate-300 rounded-none p-5 text-slate-700">
                <h3 className="font-bold text-sm text-slate-900 uppercase tracking-wider mb-3 px-1">PHP Code Files</h3>
                <div className="space-y-2">
                  {sourceFiles.map((file, idx) => (
                    <button
                      key={file.name}
                      onClick={() => setActiveCodeFile(idx)}
                      className={`w-full flex items-start gap-3 p-3 rounded-none transition-all text-left cursor-pointer ${
                        activeCodeFile === idx
                          ? 'bg-indigo-600 text-white font-bold'
                          : 'bg-slate-50 text-slate-700 hover:bg-slate-100 hover:text-slate-900 border border-slate-200'
                      }`}
                    >
                      <div className="mt-0.5">
                        {file.language === 'sql' ? (
                          <Database className="w-4 h-4 flex-shrink-0" />
                        ) : (
                          <FileText className="w-4 h-4 flex-shrink-0" />
                        )}
                      </div>
                      <div>
                        <div className="text-xs font-bold font-mono">{file.name}</div>
                        <p className={`text-[10px] mt-1 leading-relaxed ${activeCodeFile === idx ? 'text-indigo-100' : 'text-slate-500'}`}>
                          {file.description}
                        </p>
                      </div>
                    </button>
                  ))}
                </div>

                <div className="border-t border-slate-200 mt-4 pt-4 px-1">
                  <h4 className="text-xs font-bold text-slate-800 uppercase tracking-wider mb-1">Locally Saved Path</h4>
                  <p className="text-[10px] text-slate-500 leading-relaxed mb-3">
                    These code files are physically persisted in the workspace folder for you:
                  </p>
                  <div className="bg-slate-50 p-2 rounded-none border border-slate-200 font-mono text-[10px] text-slate-600 select-all">
                    ./php/{sourceFiles[activeCodeFile].name}
                  </div>
                </div>

              </div>
            </div>

            {/* RIGHT MAIN BOX: SOURCE VIEWER & DISK DETAILS */}
            <div className="lg:col-span-9 flex flex-col gap-4">
              <div className="bg-white border border-slate-300 rounded-none overflow-hidden flex flex-col">
                
                {/* File Header Details */}
                <div className="bg-slate-50 px-5 py-4 flex items-center justify-between border-b border-slate-200">
                  <div className="flex items-center gap-3">
                    <span className="font-mono text-xs bg-white text-indigo-600 px-3 py-1.5 rounded-none border border-slate-300 font-bold">
                      {sourceFiles[activeCodeFile].name}
                    </span>
                    <span className="text-xs text-slate-500 font-medium hidden sm:block font-mono">
                      {sourceFiles[activeCodeFile].path}
                    </span>
                  </div>
                  <div className="flex items-center gap-2">
                    <button
                      onClick={() => handleCopyCode(sourceFiles[activeCodeFile].content, activeCodeFile)}
                      className="flex items-center gap-1.5 px-4 py-2 bg-slate-900 hover:bg-slate-800 active:bg-black border border-slate-900 rounded-none text-xs font-bold uppercase tracking-wider text-white transition-all cursor-pointer"
                    >
                      {copiedFileIndex === activeCodeFile ? (
                        <>
                          <Check className="w-3.5 h-3.5 text-emerald-400" />
                          <span className="text-emerald-400">Copied!</span>
                        </>
                      ) : (
                        <>
                          <Copy className="w-3.5 h-3.5" />
                          <span>Copy Code</span>
                        </>
                      )}
                    </button>
                  </div>
                </div>

                {/* Preformatted Code Content */}
                <div className="bg-slate-950 p-6 overflow-x-auto font-mono text-xs text-slate-300 max-h-[600px] border-t border-slate-900 leading-relaxed select-text">
                  <pre className="select-text whitespace-pre">
                    {sourceFiles[activeCodeFile].content}
                  </pre>
                </div>

              </div>
            </div>

          </div>
        )}

        {/* TAB 3: PHP SECURITY LABS */}
        {activeTab === 'security' && (
          <div className="flex flex-col gap-8">
            
            {/* LAB 1: PREPARED STATEMENTS VS SQL INJECTION */}
            <section className="bg-white border border-slate-300 rounded-none p-6 text-slate-700">
              <div className="flex flex-col md:flex-row items-start md:items-center justify-between border-b border-slate-200 pb-4 mb-6 gap-3">
                <div className="flex items-center gap-2.5">
                  <div className="p-2 bg-red-50 text-red-600 rounded-none border border-red-200">
                    <ShieldCheck className="w-5 h-5" />
                  </div>
                  <div>
                    <h3 className="font-black text-lg text-slate-900 uppercase tracking-wide">Lab #1: Prepared Statements (PDO) vs SQL Injection</h3>
                    <p className="text-xs text-slate-500 mt-0.5">Explore how PHP PDO separates query structure from payload data.</p>
                  </div>
                </div>
                <span className="text-[10px] bg-red-50 text-red-700 px-3 py-1 rounded-none font-mono font-bold border border-red-200 uppercase tracking-wider">
                  SQL Injection Defense
                </span>
              </div>

              <div className="grid grid-cols-1 lg:grid-cols-12 gap-8">
                
                {/* Inputs & Settings */}
                <div className="lg:col-span-4 flex flex-col gap-4">
                  <div className="bg-slate-50 p-4 rounded-none border border-slate-200">
                    <label className="text-xs font-bold text-slate-850 uppercase tracking-wider block mb-2">
                      Simulate Search Input Payload
                    </label>
                    <input 
                      type="text"
                      className="w-full bg-white border border-slate-300 rounded-none px-3 py-2 text-xs font-mono text-red-600 focus:border-indigo-600 focus:outline-none"
                      value={sqlPayload}
                      onChange={(e) => setSqlPayload(e.target.value)}
                    />
                    <p className="text-[10px] text-slate-500 mt-2 leading-relaxed">
                      Type standard text (e.g. <code className="text-slate-700">PHP</code>) or a payload (e.g. <code className="text-red-600">' OR '1'='1</code>).
                    </p>
                  </div>

                  <div className="bg-slate-50 p-4 rounded-none border border-slate-200">
                    <h4 className="text-xs font-bold text-slate-850 uppercase tracking-wider mb-2">Payload Pre-Sets</h4>
                    <div className="flex flex-col gap-2">
                      <button 
                        onClick={() => setSqlPayload("PHP")}
                        className="w-full text-left bg-white hover:bg-slate-100 px-3 py-2 rounded-none text-[10px] font-mono border border-slate-300 text-slate-700 font-semibold cursor-pointer"
                      >
                        "PHP" (Safe search)
                      </button>
                      <button 
                        onClick={() => setSqlPayload("' OR '1'='1")}
                        className="w-full text-left bg-red-50 hover:bg-red-100 px-3 py-2 rounded-none text-[10px] font-mono border border-red-200 text-red-700 font-semibold cursor-pointer"
                      >
                        "' OR '1'='1" (Auth Bypass)
                      </button>
                      <button 
                        onClick={() => setSqlPayload("'; DROP TABLE posts; --")}
                        className="w-full text-left bg-red-50 hover:bg-red-100 px-3 py-2 rounded-none text-[10px] font-mono border border-red-200 text-red-700 font-semibold cursor-pointer"
                      >
                        "'; DROP TABLE posts; --"
                      </button>
                    </div>
                  </div>
                </div>

                {/* Simulated Query Engine Display */}
                <div className="lg:col-span-8 flex flex-col gap-4">
                  
                  {/* Vulnerable Method */}
                  <div className="bg-red-50/50 rounded-none p-5 border border-red-200">
                    <div className="flex items-center justify-between mb-2">
                      <span className="text-xs font-bold text-red-700 flex items-center gap-1.5 uppercase tracking-wide">
                        <AlertTriangle className="w-3.5 h-3.5" />
                        Vulnerable PHP SQL Concatenation (DO NOT USE)
                      </span>
                      <span className="text-[9px] bg-red-100 text-red-700 px-1.5 py-0.5 rounded-none border border-red-200 font-mono font-bold">
                        INSECURE
                      </span>
                    </div>
                    <code className="text-[11px] text-slate-700 block bg-white p-2.5 rounded-none font-mono border border-slate-300">
                      $sql = "SELECT * FROM posts WHERE title LIKE '%" . $searchQuery . "%'";
                    </code>
                    
                    <div className="mt-3">
                      <span className="text-[10px] font-bold text-slate-500 block mb-1 uppercase tracking-wider">Resulting Evaluated SQL Query:</span>
                      <code className="text-xs text-red-700 font-mono bg-white px-2.5 py-1.5 rounded-none border border-red-200 block leading-relaxed select-all">
                        SELECT * FROM posts WHERE title LIKE '%<span className="font-bold underline text-red-600">{sqlPayload}</span>%'
                      </code>
                    </div>
                    <div className="text-[10px] text-red-700 mt-2 leading-relaxed bg-white p-2.5 rounded-none border border-red-200">
                      {sqlPayload.includes("'") ? (
                        <strong>⚠️ VULNERABILITY DETECTED:</strong>
                      ) : (
                        <strong>Info:</strong>
                      )}{' '}
                      {sqlPayload.includes("'") 
                        ? "The single quote breaks out of the SQL string delimiter. The SQL compiler will compile and execute the rest of your string input as SQL operations, leading to critical database compromise!"
                        : "While this payload doesn't break out of quotes, direct string addition exposes the codebase to syntax failures and injection vulnerabilities the moment special characters enter variables."
                      }
                    </div>
                  </div>

                  {/* Secure Prepared Statement Method */}
                  <div className="bg-emerald-50/40 rounded-none p-5 border border-emerald-200">
                    <div className="flex items-center justify-between mb-2">
                      <span className="text-xs font-bold text-emerald-800 flex items-center gap-1.5 uppercase tracking-wide">
                        <ShieldCheck className="w-3.5 h-3.5" />
                        Secure PHP PDO Prepared Statement (Best Practice)
                      </span>
                      <span className="text-[9px] bg-emerald-100 text-emerald-800 px-1.5 py-0.5 rounded-none border border-emerald-200 font-mono font-bold">
                        SECURE
                      </span>
                    </div>
                    <code className="text-[11px] text-slate-700 block bg-white p-2.5 rounded-none font-mono border border-slate-300 leading-normal">
                      {"$stmt = $pdo->prepare(\"SELECT * FROM posts WHERE title LIKE :search\");\n$stmt->bindValue(':search', '%' . $searchQuery . '%', PDO::PARAM_STR);\n$stmt->execute();"}
                    </code>

                    <div className="mt-3">
                      <span className="text-[10px] font-bold text-slate-500 block mb-1 uppercase tracking-wider">Resulting Pre-Compiled SQL & Parameter Binding:</span>
                      <div className="bg-white p-3 rounded-none border border-slate-250 space-y-2 text-xs font-mono">
                        <div>
                          <span className="text-slate-400">Query: </span> 
                          <span className="text-slate-900 font-bold">SELECT * FROM posts WHERE title LIKE <span className="text-emerald-600 font-black">:search</span></span>
                        </div>
                        <div className="border-t border-slate-100 pt-1.5">
                          <span className="text-slate-400">Bindings: </span>
                          <span className="text-indigo-600 font-bold">:search</span> = <span className="text-emerald-700">"%{sqlPayload}%"</span>
                        </div>
                      </div>
                    </div>
                    <div className="text-[10px] text-emerald-800 mt-2 leading-relaxed bg-white p-2.5 rounded-none border border-emerald-200">
                      <strong>✅ SECURE PROTECTION ACTIVE:</strong> PDO separates the SQL instructions from the data values. The value <code className="font-mono bg-slate-50 text-indigo-700 px-1 rounded border border-slate-200 font-bold">"{sqlPayload}"</code> is bound strictly as a literal string payload, meaning any SQL syntax inside it is completely neutralized and cannot execute.
                    </div>
                  </div>

                </div>

              </div>
            </section>

            {/* LAB 2: XSS PREVENTION LAB */}
            <section className="bg-white border border-slate-300 rounded-none p-6 text-slate-700">
              <div className="flex flex-col md:flex-row items-start md:items-center justify-between border-b border-slate-200 pb-4 mb-6 gap-3">
                <div className="flex items-center gap-2.5">
                  <div className="p-2 bg-yellow-50 text-yellow-600 rounded-none border border-yellow-200">
                    <ShieldCheck className="w-5 h-5" />
                  </div>
                  <div>
                    <h3 className="font-black text-lg text-slate-900 uppercase tracking-wide">Lab #2: Cross-Site Scripting (XSS) Prevention with htmlspecialchars</h3>
                    <p className="text-xs text-slate-500 mt-0.5">Understand how escaping characters protects browser sessions from rogue scripts.</p>
                  </div>
                </div>
                <span className="text-[10px] bg-yellow-50 text-yellow-700 px-3 py-1 rounded-none font-mono font-bold border border-yellow-200 uppercase tracking-wider">
                  XSS Defense
                </span>
              </div>

              <div className="grid grid-cols-1 lg:grid-cols-12 gap-8">
                
                {/* Inputs & Settings */}
                <div className="lg:col-span-4 flex flex-col gap-4">
                  <div className="bg-slate-50 p-4 rounded-none border border-slate-200">
                    <label className="text-xs font-bold text-slate-850 uppercase tracking-wider block mb-2">
                      Input Dangerous String
                    </label>
                    <textarea 
                      rows={3}
                      className="w-full bg-white border border-slate-300 rounded-none px-3 py-2 text-xs font-mono text-yellow-600 focus:border-indigo-600 focus:outline-none leading-relaxed"
                      value={xssPayload}
                      onChange={(e) => setXssPayload(e.target.value)}
                    />
                    <p className="text-[10px] text-slate-500 mt-2 leading-relaxed">
                      This simulates raw user content submitted as a post title or comment containing script anchors.
                    </p>
                  </div>

                  <div className="bg-slate-50 p-4 rounded-none border border-slate-200">
                    <h4 className="text-xs font-bold text-slate-850 uppercase tracking-wider mb-2">Preset Exploits</h4>
                    <div className="flex flex-col gap-2">
                      <button 
                        onClick={() => setXssPayload("<script>alert('Cookie stolen: ' + document.cookie);</script>")}
                        className="w-full text-left bg-white hover:bg-slate-100 px-3 py-2 rounded-none text-[10px] font-mono border border-slate-300 text-yellow-600 font-semibold truncate cursor-pointer"
                        title="Script tag cookie theft"
                      >
                        Script Tag Exploit
                      </button>
                      <button 
                        onClick={() => setXssPayload("<img src=x onerror=\"alert('XSS execution via onerror')\">")}
                        className="w-full text-left bg-white hover:bg-slate-100 px-3 py-2 rounded-none text-[10px] font-mono border border-slate-300 text-yellow-600 font-semibold truncate cursor-pointer"
                        title="Image onerror trigger"
                      >
                        Image OnError Exploit
                      </button>
                    </div>
                  </div>
                </div>

                {/* Simulated XSS Results */}
                <div className="lg:col-span-8 flex flex-col gap-4">
                  
                  {/* Unescaped Output */}
                  <div className="bg-red-50/50 rounded-none p-5 border border-red-200">
                    <div className="flex items-center justify-between mb-2">
                      <span className="text-xs font-bold text-red-700 flex items-center gap-1.5 uppercase tracking-wide">
                        <AlertTriangle className="w-3.5 h-3.5" />
                        Unescaped Output (Insecure)
                      </span>
                      <span className="text-[9px] bg-red-100 text-red-700 px-1.5 py-0.5 rounded-none border border-red-200 font-mono font-bold">
                        VULNERABLE
                      </span>
                    </div>
                    <code className="text-[11px] text-slate-700 block bg-white p-2.5 rounded-none font-mono border border-slate-300">
                      echo $post['title'];
                    </code>

                    <div className="grid grid-cols-1 md:grid-cols-2 gap-4 mt-3">
                      <div>
                        <span className="text-[10px] font-bold text-slate-500 block mb-1 uppercase tracking-wider">Raw HTML Sent:</span>
                        <code className="text-[11px] text-red-600 font-mono bg-white p-2 rounded-none border border-red-200 block truncate" title={xssPayload}>
                          {xssPayload}
                        </code>
                      </div>
                      <div>
                        <span className="text-[10px] font-bold text-slate-500 block mb-1 uppercase tracking-wider">Rendered Visual:</span>
                        <div className="bg-white border border-red-200 text-red-600 font-bold p-2.5 rounded-none text-xs min-h-[44px] flex items-center shadow-inner">
                          {/* Alert symbol to show execution */}
                          <div className="flex items-center gap-2 animate-bounce text-red-600">
                            <AlertTriangle className="w-4 h-4 flex-shrink-0" />
                            <span>Exploit Triggered!</span>
                          </div>
                        </div>
                      </div>
                    </div>
                    <p className="text-[10px] text-red-700 mt-2 leading-relaxed bg-white p-2.5 rounded-none border border-red-200">
                      ⚠️ **CRITICAL XSS EXPOSURE:** Because the HTML parser in the browser reads raw <code className="text-slate-700">&lt;script&gt;</code> tags, it instantly runs the JavaScript. An attacker can hijack user sessions, steal administrative session keys, or modify page interfaces.
                    </p>
                  </div>

                  {/* Sanitized Escaped Output */}
                  <div className="bg-emerald-50/40 rounded-none p-5 border border-emerald-200">
                    <div className="flex items-center justify-between mb-2">
                      <span className="text-xs font-bold text-emerald-800 flex items-center gap-1.5 uppercase tracking-wide">
                        <ShieldCheck className="w-3.5 h-3.5" />
                        Escaped PHP htmlspecialchars Output (Secure)
                      </span>
                      <span className="text-[9px] bg-emerald-100 text-emerald-800 px-1.5 py-0.5 rounded-none border border-emerald-200 font-mono font-bold">
                        PROTECTED
                      </span>
                    </div>
                    <code className="text-[11px] text-slate-700 block bg-white p-2.5 rounded-none font-mono border border-slate-300">
                      echo htmlspecialchars($post['title'], ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5, 'UTF-8');
                    </code>

                    <div className="grid grid-cols-1 md:grid-cols-2 gap-4 mt-3">
                      <div>
                        <span className="text-[10px] font-bold text-slate-500 block mb-1 uppercase tracking-wider">Raw Safe Entities:</span>
                        <code className="text-[11px] text-emerald-700 font-mono bg-white p-2 rounded-none border border-emerald-200 block overflow-x-auto whitespace-pre select-all">
                          {xssPayload
                            .replace(/&/g, "&amp;")
                            .replace(/</g, "&lt;")
                            .replace(/>/g, "&gt;")
                            .replace(/"/g, "&quot;")
                            .replace(/'/g, "&#039;")}
                        </code>
                      </div>
                      <div>
                        <span className="text-[10px] font-bold text-slate-500 block mb-1 uppercase tracking-wider">Safe Visual Render:</span>
                        <div className="bg-slate-50 border border-slate-200 text-slate-700 p-2.5 rounded-none text-xs min-h-[44px] flex items-center shadow-inner overflow-hidden select-text">
                          {xssPayload}
                        </div>
                      </div>
                    </div>
                    <p className="text-[10px] text-emerald-800 mt-2 leading-relaxed bg-white p-2.5 rounded-none border border-emerald-200">
                      ✅ **SAFE RENDERING:** By converting brackets into <code className="text-indigo-600 font-mono font-bold">&amp;lt;</code> and <code className="text-indigo-600 font-mono font-bold">&amp;gt;</code>, the browser renders the tags literally as document text instead of compiling them. The script never executes.
                    </p>
                  </div>

                </div>

              </div>
            </section>

          </div>
        )}

      </main>

      {/* Global Sandbox Footer */}
      <footer className="bg-white border-t border-slate-200 py-6 px-6 mt-auto">
        <div className="max-w-7xl mx-auto flex flex-col md:flex-row items-center justify-between gap-4 text-xs text-slate-500">
          <div className="flex items-center gap-2 font-bold uppercase tracking-wider">
            <ShieldCheck className="w-4 h-4 text-indigo-600" />
            <span>PHP Full-Stack Blogging Environment Simulator &bull; 2026</span>
          </div>
          <div className="flex items-center gap-4 font-bold uppercase tracking-wider text-[10px]">
            <span>PDO Prepared Statements Enabled</span>
            <span>&bull;</span>
            <span>XSS Filtering Active</span>
          </div>
        </div>
      </footer>

      {/* --- ADD NEW POST MODAL (SIMULATED INSERT) --- */}
      {showAddModal && (
        <div className="fixed inset-0 z-50 bg-slate-900/65 flex items-center justify-center p-4">
          <div className="bg-white border-2 border-slate-900 max-w-lg w-full overflow-hidden shadow-2xl rounded-none animate-fade-in">
            
            <div className="bg-slate-900 text-white px-6 py-4 flex items-center justify-between rounded-none">
              <div className="flex items-center gap-2">
                <Database className="w-5 h-5 text-indigo-400" />
                <h3 className="font-bold uppercase tracking-widest text-sm text-white">MySQL Table: Insert New Row</h3>
              </div>
              <button 
                onClick={() => setShowAddModal(false)}
                className="text-slate-400 hover:text-white text-sm font-semibold p-1 cursor-pointer"
              >
                ✕
              </button>
            </div>

            <form onSubmit={handleCreatePost} className="p-6 space-y-4 text-slate-700">
              
              <div>
                <label className="text-xs font-bold text-slate-800 uppercase tracking-wider block mb-1.5">
                  Article Title (PDO Param: :title)
                </label>
                <input 
                  type="text"
                  required
                  placeholder="e.g., Best Practices for MySQL Query Optimization"
                  className="w-full bg-slate-50 border border-slate-300 rounded-none px-3 py-2 text-sm text-slate-900 focus:border-indigo-600 focus:bg-white focus:outline-none"
                  value={newTitle}
                  onChange={(e) => setNewTitle(e.target.value)}
                />
              </div>

              <div className="grid grid-cols-2 gap-4">
                <div>
                  <label className="text-xs font-bold text-slate-800 uppercase tracking-wider block mb-1.5">
                    Category Tag
                  </label>
                  <select 
                    className="w-full bg-slate-50 border border-slate-300 rounded-none px-3 py-2 text-sm text-slate-900 focus:border-indigo-600 focus:bg-white focus:outline-none"
                    value={newCategory}
                    onChange={(e) => setNewCategory(e.target.value)}
                  >
                    <option value="Security">Security</option>
                    <option value="Frontend">Frontend</option>
                    <option value="Backend">Backend</option>
                    <option value="Database">Database</option>
                    <option value="Architecture">Architecture</option>
                    <option value="Tools">Tools</option>
                  </select>
                </div>
                <div>
                  <label className="text-xs font-bold text-slate-800 uppercase tracking-wider block mb-1.5">
                    Date Created (Default)
                  </label>
                  <input 
                    type="text" 
                    disabled 
                    value="CURRENT_TIMESTAMP" 
                    className="w-full bg-slate-100 border border-slate-200 rounded-none px-3 py-2 text-sm text-slate-400 cursor-not-allowed font-mono font-bold"
                  />
                </div>
              </div>

              <div>
                <label className="text-xs font-bold text-slate-800 uppercase tracking-wider block mb-1.5">
                  Content Body (PDO Param: :content)
                </label>
                <textarea 
                  required
                  rows={4}
                  placeholder="Insert post content..."
                  className="w-full bg-slate-50 border border-slate-300 rounded-none px-3 py-2 text-sm text-slate-900 focus:border-indigo-600 focus:bg-white focus:outline-none leading-relaxed"
                  value={newContent}
                  onChange={(e) => setNewContent(e.target.value)}
                />
              </div>

              <div className="bg-slate-50 p-3 rounded-none border border-slate-200 space-y-2 text-xs font-mono text-slate-600">
                <span className="text-[10px] text-slate-500 font-bold uppercase block">Simulated PHP Statement:</span>
                <div>
                  <span className="text-emerald-600 font-bold">INSERT INTO</span> `posts` (`title`, `content`, `created_at`) 
                  <span className="text-emerald-600 font-bold"> VALUES</span> (:title, :content, NOW())
                </div>
              </div>

              <div className="flex justify-end gap-3 pt-4 border-t border-slate-200">
                <button
                  type="button"
                  onClick={() => setShowAddModal(false)}
                  className="px-4 py-2 bg-slate-100 hover:bg-slate-200 active:bg-slate-300 border border-slate-300 rounded-none text-xs font-bold uppercase tracking-wider text-slate-700 transition-all cursor-pointer"
                >
                  Cancel
                </button>
                <button
                  type="submit"
                  className="px-6 py-2 bg-indigo-600 hover:bg-indigo-700 active:bg-indigo-800 text-white rounded-none text-xs font-bold uppercase tracking-widest transition-all cursor-pointer"
                >
                  Execute Statement
                </button>
              </div>

            </form>

          </div>
        </div>
      )}

    </div>
  );
}
