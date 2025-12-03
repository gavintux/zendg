---
{"created":"2025-12-03T11:30:03.459+08:00","dg-publish":true,"modify":"2025-11-21T11:58:00","permalink":"/04檔案/app/Mindomo search tools 程式碼/","dgPassFrontmatter":true,"noteIcon":"","updated":"2025-12-03T14:20:20.703+08:00"}
---


```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta name="author" content="Lin Kuang Chang"> 
    <meta name="description" content="Mindomo search tools">
    <meta name="keywords" content="Mindomo,Mindmap">
    <meta name="revised" content="2025/11/21">
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mindomo 檔案分享區</title>
    
    <!-- 設定瀏覽器分頁圖示 (Favicon) -->
    <link rel="icon" href="https://2blog.ilc.edu.tw/linkc/wp-content/uploads/sites/141/2024/12/linkc-logo.png" />
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- React & ReactDOM -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    
    <!-- Babel -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@300;400;500;700&display=swap');
        body {
            font-family: 'Noto Sans TC', sans-serif;
            background-color: #f3f4f6;
        }
        .card-hover:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(-5px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .animate-fade-in {
            animation: fadeIn 0.3s ease-out forwards;
        }
        /* 自訂捲軸樣式 */
        .custom-scrollbar::-webkit-scrollbar {
            width: 6px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
            background: transparent;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
            background-color: #cbd5e1;
            border-radius: 20px;
        }
        .appearance-none {
            -webkit-appearance: none;
            -moz-appearance: none;
            appearance: none;
        }
        /* Tooltip 樣式 */
        .tooltip {
            visibility: hidden;
            position: absolute;
            z-index: 50;
            right: 0;
            top: 100%;
            margin-top: 0.5rem;
            width: 280px;
            background-color: #1f2937;
            color: white;
            text-align: left;
            border-radius: 0.5rem;
            padding: 0.75rem;
            font-size: 0.75rem;
            opacity: 0;
            transition: opacity 0.3s;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
        }
        .has-tooltip:hover .tooltip {
            visibility: visible;
            opacity: 1;
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useMemo } = React;

        // 定義主類別架構
        const PARA_ROOTS = ['00inbox', '01專案', '02領域', '03資源', '04檔案'];

        // 模擬資料
        const INITIAL_MOCK_DATA = [
            { id: "1", title: "2025 年度目標規劃", description: "包含 #工作 #學習 的年度大計畫 #share", modified_at: "2025-01-21T09:00:00Z" },
            { id: "2", title: "網站改版架構圖", description: "公司官網設計 #工作 #專案 #急件", modified_at: "2025-01-15T14:30:00Z" },
            { id: "3", title: "個人財務報表", description: "每季回顧 #健康 #財務", modified_at: "2025-01-10T11:00:00Z" },
            { id: "4", title: "Python 學習筆記", description: "爬蟲與數據分析 #學習 #程式 #Coding #share", modified_at: "2025-01-18T16:00:00Z" },
            { id: "5", title: "健身菜單", description: "重訓與飲食控制 #健康 #運動", modified_at: "2024-12-25T10:15:00Z" },
            { id: "6", title: "雜七雜八的想法", description: "還沒整理的靈感", modified_at: "2025-01-22T10:00:00Z" }, 
            { id: "7", title: "日本旅遊攻略", description: "行程安排 #旅遊 #休閒 #2025 #share", modified_at: "2024-11-05T09:00:00Z" },
            { id: "8", title: "讀書筆記：原子習慣", description: "#學習 #閱讀 #習慣 #share", modified_at: "2024-11-01T09:00:00Z" },
            { id: "9", title: "Q1 專案檢討", description: "#工作 #專案 #會議", modified_at: "2025-02-01T09:00:00Z" }
        ];

        const extractTags = (text) => {
            if (!text || typeof text !== 'string') return [];
            const cleanText = text.replace(/<[^>]*>?/gm, '');
            const matches = cleanText.match(/#[\S]+/g);
            if (!matches) return [];
            return matches.map(tag => tag.substring(1)).filter(t => t.length > 0);
        };
        
        const formatDate = (isoString) => {
            if (!isoString) return "無資料";
            const date = new Date(isoString);
            if (isNaN(date.getTime())) return "日期錯誤";
            return new Intl.DateTimeFormat('zh-TW', {
                year: 'numeric', month: '2-digit', day: '2-digit',
                hour: '2-digit', minute: '2-digit'
            }).format(date);
        };

        // Icons
        const Icons = {
            File: () => <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M14.5 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7.5L14.5 2z"/><polyline points="14 2 14 8 20 8"/></svg>,
            Clock: () => <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><circle cx="12" cy="12" r="10"/><polyline points="12 6 12 12 16 14"/></svg>,
            Tag: () => <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M12 2H2v10l9.29 9.29c.94.94 2.48.94 3.42 0l6.58-6.58c.94-.94.94-2.48 0-3.42L12 2Z"/><path d="M7 7h.01"/></svg>,
            Hash: () => <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><line x1="4" y1="9" x2="20" y2="9"/><line x1="4" y1="15" x2="20" y2="15"/><line x1="10" y1="3" x2="8" y2="21"/><line x1="16" y1="3" x2="14" y2="21"/></svg>,
            Grid: () => <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><rect x="3" y="3" width="7" height="7"></rect><rect x="14" y="3" width="7" height="7"></rect><rect x="14" y="14" width="7" height="7"></rect><rect x="3" y="14" width="7" height="7"></rect></svg>,
            Menu: () => <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><line x1="3" y1="12" x2="21" y2="12"></line><line x1="3" y1="6" x2="21" y2="6"></line><line x1="3" y1="18" x2="21" y2="18"></line></svg>,
            Search: () => <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg>,
            Settings: () => <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><circle cx="12" cy="12" r="3"></circle><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"></path></svg>,
            Refresh: ({ spin }) => <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={spin ? "animate-spin" : ""}><path d="M21 12a9 9 0 1 1-9-9c2.52 0 4.93 1 6.74 2.74L21 8"/><path d="M21 3v5h-5"/></svg>,
            ChevronLeft: () => <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polyline points="15 18 9 12 15 6"></polyline></svg>,
            ChevronRight: () => <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polyline points="9 18 15 12 9 6"></polyline></svg>,
            ChevronDown: () => <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polyline points="6 9 12 15 18 9"></polyline></svg>,
            SortAsc: () => <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="m3 16 4 4 4-4"/><path d="M7 20V4"/><path d="M11 4h10"/><path d="M11 8h7"/><path d="M11 12h4"/></svg>,
            SortDesc: () => <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="m3 8 4-4 4 4"/><path d="M7 4v16"/><path d="M11 12h4"/><path d="M11 16h7"/><path d="M11 20h10"/></svg>,
            ArrowDown: () => <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polyline points="6 9 12 15 18 9"></polyline></svg>,
            Bug: () => <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="m8 2 1.88 1.88"/><path d="M14.12 3.88 16 2"/><path d="M9 7.13v-1a3.003 3.003 0 1 1 6 0v1"/><path d="M12 20c-3.3 0-6-2.7-6-6v-3a4 4 0 0 1 4-4h4a4 4 0 0 1 4 4v3c0 3.3-2.7 6-6 6"/><path d="M12 20v-9"/><path d="M6.53 9C4.6 8.8 3 7.1 3 5"/><path d="M6 13H2"/><path d="M3 21c0-2.1 1.7-3.9 3.8-4"/><path d="M20.97 5c0 2.1-1.6 3.8-3.5 4"/><path d="M22 13h-4"/><path d="M17.2 17c2.1.1 3.8 1.9 3.8 4"/></svg>,
            AlignLeft: () => <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><line x1="17" y1="10" x2="3" y2="10"/><line x1="21" y1="6" x2="3" y2="6"/><line x1="21" y1="14" x2="3" y2="14"/><line x1="17" y1="18" x2="3" y2="18"/></svg>,
            Download: () => <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>,
            Info: () => <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><circle cx="12" cy="12" r="10"/><line x1="12" y1="16" x2="12" y2="12"/><line x1="12" y1="8" x2="12.01" y2="8"/></svg>,
            Filter: () => <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polygon points="22 3 2 3 10 12.46 10 19 14 21 14 12.46 22 3"/></svg>,
            Lock: () => <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><rect x="3" y="11" width="18" height="11" rx="2" ry="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/></svg>,
            Unlock: () => <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><rect x="3" y="11" width="18" height="11" rx="2" ry="2"/><path d="M7 11V7a5 5 0 0 1 9.9-1"/></svg>
        };

        const ITEMS_PER_PAGE = 9;
        // 設定硬寫入的 Token 與 密碼
        const HARDCODED_TOKEN = 'KgWdA3krt3dkuS3yQ0TWiOghaYeSGR_tp7u_SV9k_go';
        const ACCESS_PASSWORD = '590123';

        // 密碼輸入視窗
        const PasswordModal = ({ onSubmit, onClose }) => {
            const [input, setInput] = useState('');
            const [error, setError] = useState('');

            const handleSubmit = () => {
                if (input === ACCESS_PASSWORD) {
                    onSubmit();
                } else {
                    setError('密碼錯誤');
                }
            };

            return (
                <div className="fixed inset-0 z-50 flex items-center justify-center bg-black bg-opacity-50 animate-fade-in p-4">
                    <div className="bg-white rounded-xl shadow-2xl w-full max-w-sm overflow-hidden p-6">
                        <h3 className="text-lg font-bold text-gray-800 mb-4">管理員權限驗證</h3>
                        <p className="text-sm text-gray-600 mb-4">請輸入密碼以查看所有檔案：</p>
                        <input 
                            type="password" 
                            value={input}
                            onChange={(e) => { setInput(e.target.value); setError(''); }}
                            onKeyDown={(e) => e.key === 'Enter' && handleSubmit()}
                            className="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none mb-2"
                            placeholder="輸入密碼"
                            autoFocus
                        />
                        {error && <p className="text-xs text-red-500 mb-2">{error}</p>}
                        <div className="flex justify-end gap-2 mt-4">
                            <button onClick={onClose} className="px-3 py-2 text-sm font-medium text-gray-600 hover:bg-gray-100 rounded-lg">取消</button>
                            <button onClick={handleSubmit} className="px-3 py-2 text-sm font-medium text-white bg-blue-600 hover:bg-blue-700 rounded-lg">確認</button>
                        </div>
                    </div>
                </div>
            );
        };

        const App = () => {
            // 使用硬寫入的 Token
            const [apiToken] = useState(HARDCODED_TOKEN);
            
            // 權限狀態：預設 false (訪客模式)
            const [isAuthorized, setIsAuthorized] = useState(false);
            const [showPasswordModal, setShowPasswordModal] = useState(false);

            const [diagrams, setDiagrams] = useState([]);
            const [loading, setLoading] = useState(false);
            const [error, setError] = useState(null);
            
            // 預設開啟 Real API 模式，因為 Token 已內建
            const [useRealApi, setUseRealApi] = useState(true);
            const [showDebug, setShowDebug] = useState(false);
            const [searchTerm, setSearchTerm] = useState('');
            const [rawData, setRawData] = useState(null);
            const [downloadingState, setDownloadingState] = useState(null);
            
            // 預設顯示 'all' (但在訪客模式下，底層資料已被過濾為 share)
            const [activeCategory, setActiveCategory] = useState('all'); 
            const [expandedFolders, setExpandedFolders] = useState(PARA_ROOTS.reduce((acc, root) => ({...acc, [root]: true}), {}));
            const [currentPage, setCurrentPage] = useState(1);
            const [isMobileMenuOpen, setIsMobileMenuOpen] = useState(false);
            const [tagSearchQuery, setTagSearchQuery] = useState('');
            const [isTagExpanded, setIsTagExpanded] = useState(false);
            const [sortField, setSortField] = useState('modified_at');
            const [sortOrder, setSortOrder] = useState('desc');

            // 初始化 Demo 資料 (僅當 useRealApi 為 false 時)
            useEffect(() => {
                if (!useRealApi) {
                     const normalizedMock = INITIAL_MOCK_DATA.map(item => ({
                         ...item,
                         tags: extractTags(item.description) 
                     }));
                     setDiagrams(normalizedMock);
                     setRawData(INITIAL_MOCK_DATA);
                } else {
                    // 若使用 Real API，組件掛載時直接抓取
                    fetchDiagrams();
                }
            }, [useRealApi]);

            useEffect(() => {
                setCurrentPage(1);
            }, [searchTerm, activeCategory, sortField, sortOrder]);

            const fetchDiagrams = async () => {
                if (!apiToken) { setError("程式碼中未設定 Token"); return; }
                setLoading(true); setError(null);
                setRawData(null);
                try {
                    const response = await fetch('https://www.mindomo.com/api/v1/diagrams', {
                        method: 'GET',
                        headers: { 'Authorization': `Bearer ${apiToken}`, 'Content-Type': 'application/json' }
                    });
                    if (!response.ok) {
                        if (response.status === 401) throw new Error("認證失敗：Token 無效");
                        if (response.status === 403) throw new Error("權限不足");
                        throw new Error(`API 錯誤: ${response.status}`);
                    }
                    const data = await response.json();
                    setRawData(data);
                    let items = Array.isArray(data) ? data : (data.diagrams || []);
                    
                    const normalizedData = items.map(item => {
                        const modDate = item.modified || item.lastModified || item.modified_at || item.updated || item.updated_at || item.dateModified || item.last_modified;
                        const createDate = item.created_at || item.creationDate || item.created || item.dateCreated;
                        const description = item.description || item.note || ""; 
                        const tags = extractTags(description);

                        return {
                            id: item.id,
                            title: item.title || item.name || "未命名檔案",
                            description: description,
                            tags: tags, 
                            created_at: createDate || new Date().toISOString(),
                            modified_at: modDate || createDate,
                            _raw: item
                        };
                    });
                    setDiagrams(normalizedData);
                } catch (err) {
                    console.error(err);
                    setError(err.message + " (若是 CORS 錯誤，請嘗試 Demo 模式)");
                } finally {
                    setLoading(false);
                }
            };

            const handleDownload = async (e, item, format) => {
                e.stopPropagation();
                if (!apiToken) { alert("API Token 設定錯誤"); return; }
                setDownloadingState({ id: item.id, format: format });
                try {
                    const apiFormat = format === 'md' ? 'txt' : format;
                    if (!useRealApi) {
                        await new Promise(resolve => setTimeout(resolve, 1000));
                        alert(`(Demo 模式) 模擬下載 ${item.title}.${format}`);
                        setDownloadingState(null);
                        return;
                    }
                    const response = await fetch(`https://www.mindomo.com/api/v1/diagrams/${item.id}.${apiFormat}`, {
                        headers: { 'Authorization': `Bearer ${apiToken}` }
                    });
                    if (!response.ok) throw new Error(`API 錯誤: ${response.status} ${response.statusText}`);
                    const blob = await response.blob();
                    const url = window.URL.createObjectURL(blob);
                    const a = document.createElement('a');
                    a.href = url;
                    a.download = `${item.title}.${format}`;
                    document.body.appendChild(a);
                    a.click();
                    window.URL.revokeObjectURL(url);
                    document.body.removeChild(a);
                } catch (err) {
                    console.error(err);
                    alert('下載失敗: ' + err.message);
                } finally {
                    setDownloadingState(null);
                }
            };

            // 權限過濾核心：如果未登入，只顯示包含 share 標籤的檔案
            const permittedDiagrams = useMemo(() => {
                if (isAuthorized) return diagrams;
                // 訪客模式：過濾含有 'share' 標籤的檔案 (不分大小寫)
                return diagrams.filter(item => 
                    item.tags && item.tags.some(t => t.toLowerCase() === 'share')
                );
            }, [diagrams, isAuthorized]);

            // 標籤統計 (基於 permittedDiagrams)
            const sortedTags = useMemo(() => {
                const tagsCount = {};
                permittedDiagrams.forEach(d => {
                    if (d.tags && d.tags.length > 0) {
                        d.tags.forEach(tag => {
                            tagsCount[tag] = (tagsCount[tag] || 0) + 1;
                        });
                    }
                });
                return Object.entries(tagsCount)
                    .sort(([, a], [, b]) => b - a) 
                    .map(([tag, count]) => ({ name: tag, count }));
            }, [permittedDiagrams]);

            const visibleTags = useMemo(() => {
                let tags = sortedTags;
                if (tagSearchQuery) {
                    tags = tags.filter(t => t.name.toLowerCase().includes(tagSearchQuery.toLowerCase()));
                }
                if (!isTagExpanded && !tagSearchQuery) {
                    return tags.slice(0, 10);
                }
                return tags;
            }, [sortedTags, tagSearchQuery, isTagExpanded]);

            const checkBooleanSearch = (item, query) => {
                try {
                    let evalStr = query.toLowerCase();
                    evalStr = evalStr.replace(/\s*&\s*/g, ' && ').replace(/\s*\|\s*/g, ' || ').replace(/\s*!\s*/g, ' ! ');
                    const itemTags = new Set((item.tags || []).map(t => t.toLowerCase()));
                    evalStr = evalStr.replace(/#([a-zA-Z0-9\u4e00-\u9fa5_]+)/g, (match, tagName) => {
                        return itemTags.has(tagName) ? 'true' : 'false';
                    });
                    if (!/^(true|false|&&|\|\||!|\(|\)|\s)+$/.test(evalStr)) return null;
                    return new Function(`return (${evalStr})`)();
                } catch (e) {
                    return null;
                }
            };

            const processedData = useMemo(() => {
                let baseList = [...permittedDiagrams];
                const trimmedTerm = searchTerm.trim();

                if (trimmedTerm) {
                    const isBooleanQuery = /[&|!]/.test(trimmedTerm) && /#/.test(trimmedTerm);
                    if (isBooleanQuery) {
                         baseList = baseList.filter(item => {
                             const result = checkBooleanSearch(item, trimmedTerm);
                             if (result === null) {
                                 const lower = trimmedTerm.toLowerCase();
                                 return item.title.toLowerCase().includes(lower) || 
                                        (item.description && item.description.toLowerCase().includes(lower)) ||
                                        (item.tags && item.tags.some(t => t.toLowerCase().includes(lower)));
                             }
                             return result;
                         });
                    } else {
                         const lower = trimmedTerm.toLowerCase();
                         baseList = baseList.filter(item => 
                            item.title.toLowerCase().includes(lower) || 
                            (item.description && item.description.toLowerCase().includes(lower)) ||
                            (item.tags && item.tags.some(t => t.toLowerCase().includes(lower)))
                        );
                    }
                }
                
                let filtered = [];
                if (activeCategory === 'recent') {
                    filtered = [...baseList]
                        .filter(item => item.modified_at)
                        .sort((a, b) => new Date(b.modified_at) - new Date(a.modified_at))
                        .slice(0, 20);
                } else if (activeCategory === 'all') {
                    filtered = baseList;
                } else if (activeCategory === 'uncategorized') {
                    filtered = baseList.filter(item => !item.tags || item.tags.length === 0);
                } else {
                    filtered = baseList.filter(item => item.tags && item.tags.includes(activeCategory));
                }

                if (activeCategory !== 'recent') {
                    filtered.sort((a, b) => {
                        let valA = a[sortField];
                        let valB = b[sortField];
                        if (!valA && !valB) return 0;
                        if (!valA) return 1;
                        if (!valB) return -1;
                        let comparison = 0;
                        if (sortField === 'title') comparison = valA.localeCompare(valB, 'zh-Hant');
                        else comparison = new Date(valA) - new Date(valB);
                        return sortOrder === 'asc' ? comparison : -comparison;
                    });
                }
                const totalItems = filtered.length;
                const totalPages = Math.ceil(totalItems / ITEMS_PER_PAGE);
                const items = filtered.slice((currentPage - 1) * ITEMS_PER_PAGE, currentPage * ITEMS_PER_PAGE);
                return { items, totalItems, totalPages };
            }, [permittedDiagrams, searchTerm, activeCategory, currentPage, sortField, sortOrder]);

            const Pagination = () => {
                if (processedData.totalPages <= 1) return null;
                return (
                    <div className="flex justify-center items-center gap-2 mt-8 pt-6 border-t border-gray-200">
                        <button onClick={() => setCurrentPage(p => Math.max(1, p - 1))} disabled={currentPage === 1} className="p-2 rounded-md hover:bg-gray-100 disabled:opacity-30"><Icons.ChevronLeft /></button>
                        <span className="text-sm text-gray-600">第 {currentPage} / {processedData.totalPages} 頁</span>
                        <button onClick={() => setCurrentPage(p => Math.min(processedData.totalPages, p + 1))} disabled={currentPage === processedData.totalPages} className="p-2 rounded-md hover:bg-gray-100 disabled:opacity-30"><Icons.ChevronRight /></button>
                    </div>
                );
            };

            const SidebarItem = ({ id, label, icon, count, isActiveOverride }) => (
                <button onClick={() => { setActiveCategory(id); setIsMobileMenuOpen(false); }} className={`w-full flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium transition-colors mb-1 ${(activeCategory === id || isActiveOverride) ? 'bg-blue-50 text-blue-700' : 'text-gray-600 hover:bg-gray-100 hover:text-gray-900'}`}>
                    {icon}
                    <span className="flex-grow text-left truncate">{label}</span>
                    {count !== undefined && <span className="text-xs text-gray-400 bg-white px-1.5 py-0.5 rounded border border-gray-100">{count}</span>}
                </button>
            );

            return (
                <div className="min-h-screen bg-gray-50 flex flex-col">
                    <header className="bg-white border-b border-gray-200 sticky top-0 z-30 h-16 flex items-center px-4 justify-between lg:hidden">
                        <div className="flex items-center gap-2">
                            <img src="https://2blog.ilc.edu.tw/linkc/wp-content/uploads/sites/141/2024/12/linkc-logo.png" alt="Logo" className="h-10 w-10 rounded-lg object-contain" />
                            <h1 className="font-bold text-gray-800">Mindomo Tags</h1>
                        </div>
                        <button onClick={() => setIsMobileMenuOpen(!isMobileMenuOpen)} className="p-2 text-gray-600 hover:bg-gray-100 rounded-md"><Icons.Menu /></button>
                    </header>

                    <div className="flex flex-1 overflow-hidden relative">
                        <aside className={`fixed inset-y-0 left-0 z-20 w-64 bg-white border-r border-gray-200 transform transition-transform duration-200 ease-in-out lg:translate-x-0 lg:static lg:h-auto ${isMobileMenuOpen ? 'translate-x-0' : '-translate-x-full'}`}>
                            <div className="h-full flex flex-col">
                                <div className="hidden lg:flex h-16 items-center px-6 border-b border-gray-100">
                                    <div className="flex items-center gap-3 text-blue-700">
                                        <img src="https://2blog.ilc.edu.tw/linkc/wp-content/uploads/sites/141/2024/12/linkc-logo.png" alt="Logo" className="h-10 w-10 rounded-lg object-contain" />
                                        <span className="font-bold text-xl">Mindomo</span>
                                    </div>
                                </div>
                                <div className="flex-1 overflow-y-auto custom-scrollbar p-4">
                                    <div className="mb-6">
                                        <p className="px-3 text-xs font-semibold text-gray-400 uppercase tracking-wider mb-2">儀表板</p>
                                        <SidebarItem id="recent" label="最近開啟" icon={<Icons.Clock />} />
                                        <SidebarItem id="all" label="全部檔案" icon={<Icons.Grid />} count={permittedDiagrams.length} />
                                    </div>
                                    
                                    <div className="mb-6">
                                        <div className="flex items-center justify-between px-3 mb-2">
                                            <p className="text-xs font-semibold text-gray-400 uppercase tracking-wider">標籤分類 (#Tag)</p>
                                            {(isTagExpanded || sortedTags.length > 8) && (
                                                <div className="relative">
                                                    <input 
                                                        type="text" 
                                                        placeholder="篩選..." 
                                                        value={tagSearchQuery}
                                                        onChange={(e) => setTagSearchQuery(e.target.value)}
                                                        className="w-24 text-xs bg-gray-100 border border-transparent focus:bg-white focus:border-blue-300 rounded px-1.5 py-0.5 outline-none transition-colors"
                                                    />
                                                </div>
                                            )}
                                        </div>

                                        <div className="px-3 flex flex-wrap gap-2">
                                            {visibleTags.length === 0 && <p className="text-xs text-gray-400 italic w-full">沒有發現標籤</p>}
                                            {visibleTags.map(tag => {
                                                const isActive = activeCategory === tag.name;
                                                return (
                                                    <button 
                                                        key={tag.name}
                                                        onClick={() => { setActiveCategory(tag.name); setIsMobileMenuOpen(false); }}
                                                        className={`inline-flex items-center px-2.5 py-1 rounded-full text-xs font-medium transition-colors border ${
                                                            isActive 
                                                                ? 'bg-blue-600 text-white border-blue-600 shadow-sm' 
                                                                : 'bg-white text-gray-600 border-gray-200 hover:border-blue-300 hover:text-blue-600'
                                                        }`}
                                                        title={`${tag.name} (${tag.count})`}
                                                    >
                                                        <span className="truncate max-w-[80px]">{tag.name}</span>
                                                        <span className={`ml-1.5 text-[10px] ${isActive ? 'text-blue-100' : 'text-gray-400'}`}>{tag.count}</span>
                                                    </button>
                                                );
                                            })}
                                        </div>
                                        
                                        {sortedTags.length > 10 && !tagSearchQuery && (
                                            <button 
                                                onClick={() => setIsTagExpanded(!isTagExpanded)}
                                                className="w-full text-center text-xs text-blue-500 hover:text-blue-700 mt-2 py-1 transition-colors"
                                            >
                                                {isTagExpanded ? '收合標籤' : `顯示全部 (${sortedTags.length})`}
                                            </button>
                                        )}
                                    </div>

                                    <div>
                                        <p className="px-3 text-xs font-semibold text-gray-400 uppercase tracking-wider mb-2">其他</p>
                                        <SidebarItem id="uncategorized" label="未分類 (無標籤)" icon={<Icons.File />} count={permittedDiagrams.filter(d => !d.tags || d.tags.length === 0).length} />
                                    </div>
                                </div>
                                <div className="p-4 border-t border-gray-100 bg-gray-50 space-y-2">
                                    {/* 權限切換按鈕 */}
                                    <button 
                                        onClick={() => isAuthorized ? setIsAuthorized(false) : setShowPasswordModal(true)} 
                                        className={`w-full flex items-center justify-center gap-2 text-xs py-2 px-2 border rounded transition-colors ${isAuthorized ? 'bg-red-50 border-red-200 text-red-600 hover:bg-red-100' : 'bg-white border-gray-300 text-gray-600 hover:bg-gray-50'}`}
                                    >
                                        {isAuthorized ? <Icons.Unlock /> : <Icons.Lock />}
                                        {isAuthorized ? '登出管理員' : '管理員登入'}
                                    </button>

                                    <button onClick={() => setUseRealApi(!useRealApi)} className="w-full text-xs py-1.5 px-2 border border-gray-300 rounded bg-white text-gray-600 hover:bg-gray-50">
                                        {useRealApi ? '切換演示模式' : '切換 API 模式'}
                                    </button>
                                    
                                    {useRealApi && isAuthorized && (
                                        <div className="grid grid-cols-1 gap-2">
                                            <button onClick={() => setShowDebug(!showDebug)} className={`flex items-center justify-center gap-1 text-xs py-1.5 px-2 border border-gray-200 rounded bg-white transition-colors ${showDebug ? 'text-green-600 bg-green-50 border-green-200' : 'text-gray-500 hover:text-gray-700'}`}>
                                                <Icons.Bug />
                                                {showDebug ? '關閉除錯' : '除錯模式'}
                                            </button>
                                        </div>
                                    )}
                                </div>
                            </div>
                        </aside>
                        {isMobileMenuOpen && <div onClick={() => setIsMobileMenuOpen(false)} className="fixed inset-0 z-10 bg-gray-800 bg-opacity-50 lg:hidden"></div>}

                        <main className="flex-1 overflow-y-auto custom-scrollbar h-full bg-gray-50/50 w-full">
                            <div className="max-w-5xl mx-auto px-4 sm:px-8 py-8">
                                <div className="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-8">
                                    <div>
                                        <div className="flex items-center gap-2 mb-1">
                                            <h2 className="text-2xl font-bold text-gray-900 flex items-center gap-2">
                                                {activeCategory === 'recent' ? '最近修改' : 
                                                 activeCategory === 'all' ? '全部檔案' : 
                                                 activeCategory === 'uncategorized' ? '未分類檔案' : 
                                                 `# ${activeCategory}`}
                                            </h2>
                                            {!isAuthorized && (
                                                <span className="inline-flex items-center px-2 py-0.5 rounded text-xs font-medium bg-yellow-100 text-yellow-800">
                                                    訪客模式 (#share)
                                                </span>
                                            )}
                                        </div>
                                        <p className="text-sm text-gray-500">共 {processedData.totalItems} 個項目</p>
                                    </div>
                                    <div className="flex flex-col sm:flex-row gap-3 items-stretch sm:items-center">
                                        <div className="relative group flex-1">
                                            <div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none"><Icons.Search /></div>
                                            <input 
                                                type="text" 
                                                value={searchTerm} 
                                                onChange={(e) => setSearchTerm(e.target.value)} 
                                                className="block w-full pl-10 pr-10 py-2 border border-gray-200 rounded-lg bg-white focus:ring-2 focus:ring-blue-500 outline-none text-sm" 
                                                placeholder="搜尋... (如 #工作 & #急件)" 
                                            />
                                            <div className="absolute inset-y-0 right-0 pr-3 flex items-center cursor-help has-tooltip">
                                                <div className="text-gray-400 hover:text-blue-500"><Icons.Info /></div>
                                                <div className="tooltip">
                                                    <p className="font-bold mb-1 text-blue-300">進階標籤搜尋語法：</p>
                                                    <ul className="list-disc list-inside space-y-1">
                                                        <li><code className="bg-gray-700 px-1 rounded">&</code> (AND): 同時包含</li>
                                                        <li><code className="bg-gray-700 px-1 rounded">|</code> (OR): 包含任一</li>
                                                        <li><code className="bg-gray-700 px-1 rounded">!</code> (NOT): 排除</li>
                                                        <li><code className="bg-gray-700 px-1 rounded">()</code>: 優先權</li>
                                                    </ul>
                                                </div>
                                            </div>
                                        </div>
                                        {activeCategory !== 'recent' && (
                                            <>
                                                <div className="h-8 w-px bg-gray-300 hidden sm:block mx-1"></div>
                                                <div className="flex gap-2">
                                                    <div className="relative">
                                                        <select value={sortField} onChange={(e) => setSortField(e.target.value)} className="appearance-none w-full sm:w-32 bg-white border border-gray-200 text-gray-700 text-sm py-2 pl-3 pr-8 rounded-lg focus:ring-2 focus:ring-blue-500 cursor-pointer outline-none">
                                                            <option value="modified_at">最近修改</option>
                                                            <option value="created_at">創建時間</option>
                                                            <option value="title">檔案名稱</option>
                                                        </select>
                                                        <div className="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-500"><Icons.ArrowDown /></div>
                                                    </div>
                                                    <button onClick={() => setSortOrder(o => o === 'asc' ? 'desc' : 'asc')} className="p-2 bg-white border border-gray-200 rounded-lg text-gray-600 hover:text-blue-600">{sortOrder === 'asc' ? <Icons.SortAsc /> : <Icons.SortDesc />}</button>
                                                </div>
                                            </>
                                        )}
                                        {useRealApi && <button onClick={fetchDiagrams} disabled={loading} className="p-2 bg-white border border-gray-200 rounded-lg text-blue-600 hover:bg-blue-50"><Icons.Refresh spin={loading} /></button>}
                                    </div>
                                </div>
                                
                                {/* Main Content Area - Grid/Cards */}
                                {error && <div className="mb-6 p-4 bg-red-50 text-red-700 rounded-lg border border-red-100">{error}</div>}
                                
                                {processedData.totalItems === 0 ? (
                                    <div className="text-center py-20 animate-fade-in">
                                        <div className="inline-flex items-center justify-center w-20 h-20 rounded-full bg-gray-100 mb-4 text-gray-300"><Icons.Grid /></div>
                                        <h3 className="text-lg font-medium text-gray-900">沒有檔案</h3>
                                        <p className="text-gray-500 mt-2 text-sm">
                                            {isAuthorized ? "確認是否有在 Mindomo 描述欄位加上 #標籤 ?" : "目前沒有公開分享 (#share) 的檔案"}
                                        </p>
                                    </div>
                                ) : (
                                    <div className="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-6">
                                        {processedData.items.map((item) => (
                                            <div key={item.id} className="group bg-white rounded-xl border border-gray-200 p-5 transition-all duration-200 card-hover flex flex-col h-full relative overflow-hidden">
                                                <div className="absolute top-0 left-0 w-1 h-full bg-blue-500 opacity-0 group-hover:opacity-100 transition-opacity"></div>
                                                <div className="flex justify-between items-start mb-3">
                                                    <div className="p-2.5 bg-blue-50 text-blue-600 rounded-lg group-hover:bg-blue-600 group-hover:text-white transition-colors"><Icons.File /></div>
                                                    <div className="flex gap-2">
                                                        {useRealApi && <a href={`https://www.mindomo.com/mindmap/${item.id}`} target="_blank" className="text-xs font-medium text-blue-600 bg-blue-50 px-2.5 py-1.5 rounded-full hover:bg-blue-100 transition-colors flex items-center">開啟</a>}
                                                    </div>
                                                </div>
                                                
                                                <h3 className="text-lg font-bold text-gray-900 mb-2 line-clamp-2 leading-snug group-hover:text-blue-600 transition-colors">{item.title}</h3>
                                                
                                                {/* 顯示標籤區塊 */}
                                                <div className="flex flex-wrap gap-1 mb-4 min-h-[24px]">
                                                    {item.tags && item.tags.length > 0 ? (
                                                        item.tags.map(tag => (
                                                            <span key={tag} onClick={(e) => { e.stopPropagation(); setActiveCategory(tag); }} className="cursor-pointer inline-flex items-center px-2 py-0.5 rounded-full text-xs font-medium bg-gray-100 text-gray-600 hover:bg-blue-100 hover:text-blue-600 transition-colors">
                                                                #{tag}
                                                            </span>
                                                        ))
                                                    ) : (
                                                        <span className="text-xs text-gray-400 italic">無標籤</span>
                                                    )}
                                                </div>

                                                <div className="mt-auto pt-4 border-t border-gray-50 space-y-2">
                                                    <div className="flex items-center text-xs text-gray-500">
                                                        <Icons.Clock />
                                                        <span className="ml-2">
                                                            {item.modified_at ? `最後修改於 ${formatDate(item.modified_at)}` : '無修改日期'}
                                                        </span>
                                                    </div>
                                                    
                                                    <div className="flex gap-2 pt-2 border-t border-gray-100 justify-end">
                                                        <button onClick={(e) => handleDownload(e, item, 'md')} disabled={downloadingState && downloadingState.id === item.id} className="flex items-center gap-1 px-2 py-1 text-xs font-medium text-gray-600 bg-gray-50 hover:bg-gray-100 rounded transition-colors disabled:opacity-50" title="下載 Markdown (.md)">{downloadingState && downloadingState.id === item.id && downloadingState.format === 'md' ? <Icons.Refresh spin={true} /> : <span>MD</span>}</button>
                                                        <button onClick={(e) => handleDownload(e, item, 'mm')} disabled={downloadingState && downloadingState.id === item.id} className="flex items-center gap-1 px-2 py-1 text-xs font-medium text-gray-600 bg-gray-50 hover:bg-gray-100 rounded transition-colors disabled:opacity-50" title="下載 Freemind (.mm)">{downloadingState && downloadingState.id === item.id && downloadingState.format === 'mm' ? <Icons.Refresh spin={true} /> : <span>MM</span>}</button>
                                                        <button onClick={(e) => handleDownload(e, item, 'pdf')} disabled={downloadingState && downloadingState.id === item.id} className="flex items-center gap-1 px-2 py-1 text-xs font-medium text-gray-600 bg-gray-50 hover:bg-gray-100 rounded transition-colors disabled:opacity-50" title="下載 PDF (.pdf)">{downloadingState && downloadingState.id === item.id && downloadingState.format === 'pdf' ? <Icons.Refresh spin={true} /> : <span>PDF</span>}</button>
                                                    </div>
                                                </div>
                                            </div>
                                        ))}
                                    </div>
                                )}
                                <Pagination />
                                
                                {/* 密碼輸入視窗 */}
                                {showPasswordModal && (
                                    <PasswordModal 
                                        onSubmit={() => {
                                            setIsAuthorized(true);
                                            setShowPasswordModal(false);
                                        }}
                                        onClose={() => setShowPasswordModal(false)}
                                    />
                                )}

                                {useRealApi && rawData && showDebug && (
                                    <div className="mt-16 p-6 bg-gray-900 text-gray-300 rounded-xl shadow-lg animate-fade-in">
                                        <div className="flex items-center justify-between mb-4 border-b border-gray-700 pb-4">
                                            <div className="flex items-center gap-2">
                                                <Icons.Bug />
                                                <h3 className="text-lg font-bold text-white">API 資料除錯 (Debug Info)</h3>
                                            </div>
                                            <button onClick={() => setShowDebug(false)} className="text-xs text-gray-500 hover:text-white">關閉</button>
                                        </div>
                                        <p className="text-sm mb-4 text-gray-400">
                                            請檢查 JSON 中 <code>description</code> 欄位是否有包含 #標籤。
                                        </p>
                                        
                                        <div className="space-y-6">
                                            <div>
                                                <h4 className="text-xs font-bold text-blue-400 uppercase tracking-wider mb-2">原始資料 (前 1 筆範例)</h4>
                                                <pre className="bg-black p-4 rounded-lg text-xs overflow-x-auto font-mono border border-gray-800 text-green-400">
                                                    {JSON.stringify(Array.isArray(rawData) ? rawData[0] : (rawData.diagrams ? rawData.diagrams[0] : rawData), null, 2)}
                                                </pre>
                                            </div>
                                            
                                            <div>
                                                <h4 className="text-xs font-bold text-blue-400 uppercase tracking-wider mb-2">資料結構統計</h4>
                                                <div className="bg-black p-4 rounded-lg text-xs font-mono border border-gray-800">
                                                    <p>總筆數: {Array.isArray(rawData) ? rawData.length : (rawData.diagrams ? rawData.diagrams.length : 0)}</p>
                                                    <p>包含 'description' 欄位的筆數: {(Array.isArray(rawData) ? rawData : (rawData.diagrams || [])).filter(i => i.description).length}</p>
                                                    <p>偵測到的標籤總數: {sortedTags.length}</p>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                )}
                            </div>
                        </main>
                    </div>
                </div>
            );
        }

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mindomo 檔案分享區</title>
    
    <!-- 設定瀏覽器分頁圖示 (Favicon) -->
    <link rel="icon" href="https://2blog.ilc.edu.tw/linkc/wp-content/uploads/sites/141/2024/12/linkc-logo.png" />
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- React & ReactDOM -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    
    <!-- Babel -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@300;400;500;700&display=swap');
        body {
            font-family: 'Noto Sans TC', sans-serif;
            background-color: #f3f4f6;
        }
        .card-hover:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(-5px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .animate-fade-in {
            animation: fadeIn 0.3s ease-out forwards;
        }
        /* 自訂捲軸樣式 */
        .custom-scrollbar::-webkit-scrollbar {
            width: 6px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
            background: transparent;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
            background-color: #cbd5e1;
            border-radius: 20px;
        }
        .appearance-none {
            -webkit-appearance: none;
            -moz-appearance: none;
            appearance: none;
        }
        /* Tooltip 樣式 */
        .tooltip {
            visibility: hidden;
            position: absolute;
            z-index: 50;
            right: 0;
            top: 100%;
            margin-top: 0.5rem;
            width: 280px;
            background-color: #1f2937;
            color: white;
            text-align: left;
            border-radius: 0.5rem;
            padding: 0.75rem;
            font-size: 0.75rem;
            opacity: 0;
            transition: opacity 0.3s;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
        }
        .has-tooltip:hover .tooltip {
            visibility: visible;
            opacity: 1;
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useMemo } = React;

        // 定義主類別架構
        const PARA_ROOTS = ['00inbox', '01專案', '02領域', '03資源', '04檔案'];

        // 模擬資料
        const INITIAL_MOCK_DATA = [
            { id: "1", title: "2025 年度目標規劃", description: "包含 #工作 #學習 的年度大計畫 #share", modified_at: "2025-01-21T09:00:00Z" },
            { id: "2", title: "網站改版架構圖", description: "公司官網設計 #工作 #專案 #急件", modified_at: "2025-01-15T14:30:00Z" },
            { id: "3", title: "個人財務報表", description: "每季回顧 #健康 #財務", modified_at: "2025-01-10T11:00:00Z" },
            { id: "4", title: "Python 學習筆記", description: "爬蟲與數據分析 #學習 #程式 #Coding #share", modified_at: "2025-01-18T16:00:00Z" },
            { id: "5", title: "健身菜單", description: "重訓與飲食控制 #健康 #運動", modified_at: "2024-12-25T10:15:00Z" },
            { id: "6", title: "雜七雜八的想法", description: "還沒整理的靈感", modified_at: "2025-01-22T10:00:00Z" }, 
            { id: "7", title: "日本旅遊攻略", description: "行程安排 #旅遊 #休閒 #2025 #share", modified_at: "2024-11-05T09:00:00Z" },
            { id: "8", title: "讀書筆記：原子習慣", description: "#學習 #閱讀 #習慣 #share", modified_at: "2024-11-01T09:00:00Z" },
            { id: "9", title: "Q1 專案檢討", description: "#工作 #專案 #會議", modified_at: "2025-02-01T09:00:00Z" }
        ];

        const extractTags = (text) => {
            if (!text || typeof text !== 'string') return [];
            const cleanText = text.replace(/<[^>]*>?/gm, '');
            const matches = cleanText.match(/#[\S]+/g);
            if (!matches) return [];
            return matches.map(tag => tag.substring(1)).filter(t => t.length > 0);
        };
        
        const formatDate = (isoString) => {
            if (!isoString) return "無資料";
            const date = new Date(isoString);
            if (isNaN(date.getTime())) return "日期錯誤";
            return new Intl.DateTimeFormat('zh-TW', {
                year: 'numeric', month: '2-digit', day: '2-digit',
                hour: '2-digit', minute: '2-digit'
            }).format(date);
        };

        // Icons
        const Icons = {
            File: () => <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M14.5 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7.5L14.5 2z"/><polyline points="14 2 14 8 20 8"/></svg>,
            Clock: () => <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><circle cx="12" cy="12" r="10"/><polyline points="12 6 12 12 16 14"/></svg>,
            Tag: () => <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M12 2H2v10l9.29 9.29c.94.94 2.48.94 3.42 0l6.58-6.58c.94-.94.94-2.48 0-3.42L12 2Z"/><path d="M7 7h.01"/></svg>,
            Hash: () => <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><line x1="4" y1="9" x2="20" y2="9"/><line x1="4" y1="15" x2="20" y2="15"/><line x1="10" y1="3" x2="8" y2="21"/><line x1="16" y1="3" x2="14" y2="21"/></svg>,
            Grid: () => <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><rect x="3" y="3" width="7" height="7"></rect><rect x="14" y="3" width="7" height="7"></rect><rect x="14" y="14" width="7" height="7"></rect><rect x="3" y="14" width="7" height="7"></rect></svg>,
            Menu: () => <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><line x1="3" y1="12" x2="21" y2="12"></line><line x1="3" y1="6" x2="21" y2="6"></line><line x1="3" y1="18" x2="21" y2="18"></line></svg>,
            Search: () => <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg>,
            Settings: () => <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><circle cx="12" cy="12" r="3"></circle><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"></path></svg>,
            Refresh: ({ spin }) => <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={spin ? "animate-spin" : ""}><path d="M21 12a9 9 0 1 1-9-9c2.52 0 4.93 1 6.74 2.74L21 8"/><path d="M21 3v5h-5"/></svg>,
            ChevronLeft: () => <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polyline points="15 18 9 12 15 6"></polyline></svg>,
            ChevronRight: () => <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polyline points="9 18 15 12 9 6"></polyline></svg>,
            ChevronDown: () => <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polyline points="6 9 12 15 18 9"></polyline></svg>,
            SortAsc: () => <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="m3 16 4 4 4-4"/><path d="M7 20V4"/><path d="M11 4h10"/><path d="M11 8h7"/><path d="M11 12h4"/></svg>,
            SortDesc: () => <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="m3 8 4-4 4 4"/><path d="M7 4v16"/><path d="M11 12h4"/><path d="M11 16h7"/><path d="M11 20h10"/></svg>,
            ArrowDown: () => <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polyline points="6 9 12 15 18 9"></polyline></svg>,
            Bug: () => <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="m8 2 1.88 1.88"/><path d="M14.12 3.88 16 2"/><path d="M9 7.13v-1a3.003 3.003 0 1 1 6 0v1"/><path d="M12 20c-3.3 0-6-2.7-6-6v-3a4 4 0 0 1 4-4h4a4 4 0 0 1 4 4v3c0 3.3-2.7 6-6 6"/><path d="M12 20v-9"/><path d="M6.53 9C4.6 8.8 3 7.1 3 5"/><path d="M6 13H2"/><path d="M3 21c0-2.1 1.7-3.9 3.8-4"/><path d="M20.97 5c0 2.1-1.6 3.8-3.5 4"/><path d="M22 13h-4"/><path d="M17.2 17c2.1.1 3.8 1.9 3.8 4"/></svg>,
            AlignLeft: () => <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><line x1="17" y1="10" x2="3" y2="10"/><line x1="21" y1="6" x2="3" y2="6"/><line x1="21" y1="14" x2="3" y2="14"/><line x1="17" y1="18" x2="3" y2="18"/></svg>,
            Download: () => <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>,
            Info: () => <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><circle cx="12" cy="12" r="10"/><line x1="12" y1="16" x2="12" y2="12"/><line x1="12" y1="8" x2="12.01" y2="8"/></svg>,
            Filter: () => <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polygon points="22 3 2 3 10 12.46 10 19 14 21 14 12.46 22 3"/></svg>,
            Lock: () => <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><rect x="3" y="11" width="18" height="11" rx="2" ry="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/></svg>,
            Unlock: () => <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><rect x="3" y="11" width="18" height="11" rx="2" ry="2"/><path d="M7 11V7a5 5 0 0 1 9.9-1"/></svg>
        };

        const ITEMS_PER_PAGE = 9;
        // 設定硬寫入的 Token 與 密碼
        const HARDCODED_TOKEN = 'KgWdA3krt3dkuS3yQ0TWiOghaYeSGR_tp7u_SV9k_go';
        const ACCESS_PASSWORD = '590123';

        // 密碼輸入視窗
        const PasswordModal = ({ onSubmit, onClose }) => {
            const [input, setInput] = useState('');
            const [error, setError] = useState('');

            const handleSubmit = () => {
                if (input === ACCESS_PASSWORD) {
                    onSubmit();
                } else {
                    setError('密碼錯誤');
                }
            };

            return (
                <div className="fixed inset-0 z-50 flex items-center justify-center bg-black bg-opacity-50 animate-fade-in p-4">
                    <div className="bg-white rounded-xl shadow-2xl w-full max-w-sm overflow-hidden p-6">
                        <h3 className="text-lg font-bold text-gray-800 mb-4">管理員權限驗證</h3>
                        <p className="text-sm text-gray-600 mb-4">請輸入密碼以查看所有檔案：</p>
                        <input 
                            type="password" 
                            value={input}
                            onChange={(e) => { setInput(e.target.value); setError(''); }}
                            onKeyDown={(e) => e.key === 'Enter' && handleSubmit()}
                            className="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none mb-2"
                            placeholder="輸入密碼"
                            autoFocus
                        />
                        {error && <p className="text-xs text-red-500 mb-2">{error}</p>}
                        <div className="flex justify-end gap-2 mt-4">
                            <button onClick={onClose} className="px-3 py-2 text-sm font-medium text-gray-600 hover:bg-gray-100 rounded-lg">取消</button>
                            <button onClick={handleSubmit} className="px-3 py-2 text-sm font-medium text-white bg-blue-600 hover:bg-blue-700 rounded-lg">確認</button>
                        </div>
                    </div>
                </div>
            );
        };

        const App = () => {
            // 使用硬寫入的 Token
            const [apiToken] = useState(HARDCODED_TOKEN);
            
            // 權限狀態：預設 false (訪客模式)
            const [isAuthorized, setIsAuthorized] = useState(false);
            const [showPasswordModal, setShowPasswordModal] = useState(false);

            const [diagrams, setDiagrams] = useState([]);
            const [loading, setLoading] = useState(false);
            const [error, setError] = useState(null);
            
            // 預設開啟 Real API 模式，因為 Token 已內建
            const [useRealApi, setUseRealApi] = useState(true);
            const [showDebug, setShowDebug] = useState(false);
            const [searchTerm, setSearchTerm] = useState('');
            const [rawData, setRawData] = useState(null);
            const [downloadingState, setDownloadingState] = useState(null);
            
            // 預設顯示 'all' (但在訪客模式下，底層資料已被過濾為 share)
            const [activeCategory, setActiveCategory] = useState('all'); 
            const [expandedFolders, setExpandedFolders] = useState(PARA_ROOTS.reduce((acc, root) => ({...acc, [root]: true}), {}));
            const [currentPage, setCurrentPage] = useState(1);
            const [isMobileMenuOpen, setIsMobileMenuOpen] = useState(false);
            const [tagSearchQuery, setTagSearchQuery] = useState('');
            const [isTagExpanded, setIsTagExpanded] = useState(false);
            const [sortField, setSortField] = useState('modified_at');
            const [sortOrder, setSortOrder] = useState('desc');

            // 初始化 Demo 資料 (僅當 useRealApi 為 false 時)
            useEffect(() => {
                if (!useRealApi) {
                     const normalizedMock = INITIAL_MOCK_DATA.map(item => ({
                         ...item,
                         tags: extractTags(item.description) 
                     }));
                     setDiagrams(normalizedMock);
                     setRawData(INITIAL_MOCK_DATA);
                } else {
                    // 若使用 Real API，組件掛載時直接抓取
                    fetchDiagrams();
                }
            }, [useRealApi]);

            useEffect(() => {
                setCurrentPage(1);
            }, [searchTerm, activeCategory, sortField, sortOrder]);

            const fetchDiagrams = async () => {
                if (!apiToken) { setError("程式碼中未設定 Token"); return; }
                setLoading(true); setError(null);
                setRawData(null);
                try {
                    const response = await fetch('https://www.mindomo.com/api/v1/diagrams', {
                        method: 'GET',
                        headers: { 'Authorization': `Bearer ${apiToken}`, 'Content-Type': 'application/json' }
                    });
                    if (!response.ok) {
                        if (response.status === 401) throw new Error("認證失敗：Token 無效");
                        if (response.status === 403) throw new Error("權限不足");
                        throw new Error(`API 錯誤: ${response.status}`);
                    }
                    const data = await response.json();
                    setRawData(data);
                    let items = Array.isArray(data) ? data : (data.diagrams || []);
                    
                    const normalizedData = items.map(item => {
                        const modDate = item.modified || item.lastModified || item.modified_at || item.updated || item.updated_at || item.dateModified || item.last_modified;
                        const createDate = item.created_at || item.creationDate || item.created || item.dateCreated;
                        const description = item.description || item.note || ""; 
                        const tags = extractTags(description);

                        return {
                            id: item.id,
                            title: item.title || item.name || "未命名檔案",
                            description: description,
                            tags: tags, 
                            created_at: createDate || new Date().toISOString(),
                            modified_at: modDate || createDate,
                            _raw: item
                        };
                    });
                    setDiagrams(normalizedData);
                } catch (err) {
                    console.error(err);
                    setError(err.message + " (若是 CORS 錯誤，請嘗試 Demo 模式)");
                } finally {
                    setLoading(false);
                }
            };

            const handleDownload = async (e, item, format) => {
                e.stopPropagation();
                if (!apiToken) { alert("API Token 設定錯誤"); return; }
                setDownloadingState({ id: item.id, format: format });
                try {
                    const apiFormat = format === 'md' ? 'txt' : format;
                    if (!useRealApi) {
                        await new Promise(resolve => setTimeout(resolve, 1000));
                        alert(`(Demo 模式) 模擬下載 ${item.title}.${format}`);
                        setDownloadingState(null);
                        return;
                    }
                    const response = await fetch(`https://www.mindomo.com/api/v1/diagrams/${item.id}.${apiFormat}`, {
                        headers: { 'Authorization': `Bearer ${apiToken}` }
                    });
                    if (!response.ok) throw new Error(`API 錯誤: ${response.status} ${response.statusText}`);
                    const blob = await response.blob();
                    const url = window.URL.createObjectURL(blob);
                    const a = document.createElement('a');
                    a.href = url;
                    a.download = `${item.title}.${format}`;
                    document.body.appendChild(a);
                    a.click();
                    window.URL.revokeObjectURL(url);
                    document.body.removeChild(a);
                } catch (err) {
                    console.error(err);
                    alert('下載失敗: ' + err.message);
                } finally {
                    setDownloadingState(null);
                }
            };

            // 權限過濾核心：如果未登入，只顯示包含 share 標籤的檔案
            const permittedDiagrams = useMemo(() => {
                if (isAuthorized) return diagrams;
                // 訪客模式：過濾含有 'share' 標籤的檔案 (不分大小寫)
                return diagrams.filter(item => 
                    item.tags && item.tags.some(t => t.toLowerCase() === 'share')
                );
            }, [diagrams, isAuthorized]);

            // 標籤統計 (基於 permittedDiagrams)
            const sortedTags = useMemo(() => {
                const tagsCount = {};
                permittedDiagrams.forEach(d => {
                    if (d.tags && d.tags.length > 0) {
                        d.tags.forEach(tag => {
                            tagsCount[tag] = (tagsCount[tag] || 0) + 1;
                        });
                    }
                });
                return Object.entries(tagsCount)
                    .sort(([, a], [, b]) => b - a) 
                    .map(([tag, count]) => ({ name: tag, count }));
            }, [permittedDiagrams]);

            const visibleTags = useMemo(() => {
                let tags = sortedTags;
                if (tagSearchQuery) {
                    tags = tags.filter(t => t.name.toLowerCase().includes(tagSearchQuery.toLowerCase()));
                }
                if (!isTagExpanded && !tagSearchQuery) {
                    return tags.slice(0, 10);
                }
                return tags;
            }, [sortedTags, tagSearchQuery, isTagExpanded]);

            const checkBooleanSearch = (item, query) => {
                try {
                    let evalStr = query.toLowerCase();
                    evalStr = evalStr.replace(/\s*&\s*/g, ' && ').replace(/\s*\|\s*/g, ' || ').replace(/\s*!\s*/g, ' ! ');
                    const itemTags = new Set((item.tags || []).map(t => t.toLowerCase()));
                    evalStr = evalStr.replace(/#([a-zA-Z0-9\u4e00-\u9fa5_]+)/g, (match, tagName) => {
                        return itemTags.has(tagName) ? 'true' : 'false';
                    });
                    if (!/^(true|false|&&|\|\||!|\(|\)|\s)+$/.test(evalStr)) return null;
                    return new Function(`return (${evalStr})`)();
                } catch (e) {
                    return null;
                }
            };

            const processedData = useMemo(() => {
                let baseList = [...permittedDiagrams];
                const trimmedTerm = searchTerm.trim();

                if (trimmedTerm) {
                    const isBooleanQuery = /[&|!]/.test(trimmedTerm) && /#/.test(trimmedTerm);
                    if (isBooleanQuery) {
                         baseList = baseList.filter(item => {
                             const result = checkBooleanSearch(item, trimmedTerm);
                             if (result === null) {
                                 const lower = trimmedTerm.toLowerCase();
                                 return item.title.toLowerCase().includes(lower) || 
                                        (item.description && item.description.toLowerCase().includes(lower)) ||
                                        (item.tags && item.tags.some(t => t.toLowerCase().includes(lower)));
                             }
                             return result;
                         });
                    } else {
                         const lower = trimmedTerm.toLowerCase();
                         baseList = baseList.filter(item => 
                            item.title.toLowerCase().includes(lower) || 
                            (item.description && item.description.toLowerCase().includes(lower)) ||
                            (item.tags && item.tags.some(t => t.toLowerCase().includes(lower)))
                        );
                    }
                }
                
                let filtered = [];
                if (activeCategory === 'recent') {
                    filtered = [...baseList]
                        .filter(item => item.modified_at)
                        .sort((a, b) => new Date(b.modified_at) - new Date(a.modified_at))
                        .slice(0, 20);
                } else if (activeCategory === 'all') {
                    filtered = baseList;
                } else if (activeCategory === 'uncategorized') {
                    filtered = baseList.filter(item => !item.tags || item.tags.length === 0);
                } else {
                    filtered = baseList.filter(item => item.tags && item.tags.includes(activeCategory));
                }

                if (activeCategory !== 'recent') {
                    filtered.sort((a, b) => {
                        let valA = a[sortField];
                        let valB = b[sortField];
                        if (!valA && !valB) return 0;
                        if (!valA) return 1;
                        if (!valB) return -1;
                        let comparison = 0;
                        if (sortField === 'title') comparison = valA.localeCompare(valB, 'zh-Hant');
                        else comparison = new Date(valA) - new Date(valB);
                        return sortOrder === 'asc' ? comparison : -comparison;
                    });
                }
                const totalItems = filtered.length;
                const totalPages = Math.ceil(totalItems / ITEMS_PER_PAGE);
                const items = filtered.slice((currentPage - 1) * ITEMS_PER_PAGE, currentPage * ITEMS_PER_PAGE);
                return { items, totalItems, totalPages };
            }, [permittedDiagrams, searchTerm, activeCategory, currentPage, sortField, sortOrder]);

            const Pagination = () => {
                if (processedData.totalPages <= 1) return null;
                return (
                    <div className="flex justify-center items-center gap-2 mt-8 pt-6 border-t border-gray-200">
                        <button onClick={() => setCurrentPage(p => Math.max(1, p - 1))} disabled={currentPage === 1} className="p-2 rounded-md hover:bg-gray-100 disabled:opacity-30"><Icons.ChevronLeft /></button>
                        <span className="text-sm text-gray-600">第 {currentPage} / {processedData.totalPages} 頁</span>
                        <button onClick={() => setCurrentPage(p => Math.min(processedData.totalPages, p + 1))} disabled={currentPage === processedData.totalPages} className="p-2 rounded-md hover:bg-gray-100 disabled:opacity-30"><Icons.ChevronRight /></button>
                    </div>
                );
            };

            const SidebarItem = ({ id, label, icon, count, isActiveOverride }) => (
                <button onClick={() => { setActiveCategory(id); setIsMobileMenuOpen(false); }} className={`w-full flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium transition-colors mb-1 ${(activeCategory === id || isActiveOverride) ? 'bg-blue-50 text-blue-700' : 'text-gray-600 hover:bg-gray-100 hover:text-gray-900'}`}>
                    {icon}
                    <span className="flex-grow text-left truncate">{label}</span>
                    {count !== undefined && <span className="text-xs text-gray-400 bg-white px-1.5 py-0.5 rounded border border-gray-100">{count}</span>}
                </button>
            );

            return (
                <div className="min-h-screen bg-gray-50 flex flex-col">
                    <header className="bg-white border-b border-gray-200 sticky top-0 z-30 h-16 flex items-center px-4 justify-between lg:hidden">
                        <div className="flex items-center gap-2">
                            <img src="https://2blog.ilc.edu.tw/linkc/wp-content/uploads/sites/141/2024/12/linkc-logo.png" alt="Logo" className="h-10 w-10 rounded-lg object-contain" />
                            <h1 className="font-bold text-gray-800">Mindomo Tags</h1>
                        </div>
                        <button onClick={() => setIsMobileMenuOpen(!isMobileMenuOpen)} className="p-2 text-gray-600 hover:bg-gray-100 rounded-md"><Icons.Menu /></button>
                    </header>

                    <div className="flex flex-1 overflow-hidden relative">
                        <aside className={`fixed inset-y-0 left-0 z-20 w-64 bg-white border-r border-gray-200 transform transition-transform duration-200 ease-in-out lg:translate-x-0 lg:static lg:h-auto ${isMobileMenuOpen ? 'translate-x-0' : '-translate-x-full'}`}>
                            <div className="h-full flex flex-col">
                                <div className="hidden lg:flex h-16 items-center px-6 border-b border-gray-100">
                                    <div className="flex items-center gap-3 text-blue-700">
                                        <img src="https://2blog.ilc.edu.tw/linkc/wp-content/uploads/sites/141/2024/12/linkc-logo.png" alt="Logo" className="h-10 w-10 rounded-lg object-contain" />
                                        <span className="font-bold text-xl">Mindomo</span>
                                    </div>
                                </div>
                                <div className="flex-1 overflow-y-auto custom-scrollbar p-4">
                                    <div className="mb-6">
                                        <p className="px-3 text-xs font-semibold text-gray-400 uppercase tracking-wider mb-2">儀表板</p>
                                        <SidebarItem id="recent" label="最近開啟" icon={<Icons.Clock />} />
                                        <SidebarItem id="all" label="全部檔案" icon={<Icons.Grid />} count={permittedDiagrams.length} />
                                    </div>
                                    
                                    <div className="mb-6">
                                        <div className="flex items-center justify-between px-3 mb-2">
                                            <p className="text-xs font-semibold text-gray-400 uppercase tracking-wider">標籤分類 (#Tag)</p>
                                            {(isTagExpanded || sortedTags.length > 8) && (
                                                <div className="relative">
                                                    <input 
                                                        type="text" 
                                                        placeholder="篩選..." 
                                                        value={tagSearchQuery}
                                                        onChange={(e) => setTagSearchQuery(e.target.value)}
                                                        className="w-24 text-xs bg-gray-100 border border-transparent focus:bg-white focus:border-blue-300 rounded px-1.5 py-0.5 outline-none transition-colors"
                                                    />
                                                </div>
                                            )}
                                        </div>

                                        <div className="px-3 flex flex-wrap gap-2">
                                            {visibleTags.length === 0 && <p className="text-xs text-gray-400 italic w-full">沒有發現標籤</p>}
                                            {visibleTags.map(tag => {
                                                const isActive = activeCategory === tag.name;
                                                return (
                                                    <button 
                                                        key={tag.name}
                                                        onClick={() => { setActiveCategory(tag.name); setIsMobileMenuOpen(false); }}
                                                        className={`inline-flex items-center px-2.5 py-1 rounded-full text-xs font-medium transition-colors border ${
                                                            isActive 
                                                                ? 'bg-blue-600 text-white border-blue-600 shadow-sm' 
                                                                : 'bg-white text-gray-600 border-gray-200 hover:border-blue-300 hover:text-blue-600'
                                                        }`}
                                                        title={`${tag.name} (${tag.count})`}
                                                    >
                                                        <span className="truncate max-w-[80px]">{tag.name}</span>
                                                        <span className={`ml-1.5 text-[10px] ${isActive ? 'text-blue-100' : 'text-gray-400'}`}>{tag.count}</span>
                                                    </button>
                                                );
                                            })}
                                        </div>
                                        
                                        {sortedTags.length > 10 && !tagSearchQuery && (
                                            <button 
                                                onClick={() => setIsTagExpanded(!isTagExpanded)}
                                                className="w-full text-center text-xs text-blue-500 hover:text-blue-700 mt-2 py-1 transition-colors"
                                            >
                                                {isTagExpanded ? '收合標籤' : `顯示全部 (${sortedTags.length})`}
                                            </button>
                                        )}
                                    </div>

                                    <div>
                                        <p className="px-3 text-xs font-semibold text-gray-400 uppercase tracking-wider mb-2">其他</p>
                                        <SidebarItem id="uncategorized" label="未分類 (無標籤)" icon={<Icons.File />} count={permittedDiagrams.filter(d => !d.tags || d.tags.length === 0).length} />
                                    </div>
                                </div>
                                <div className="p-4 border-t border-gray-100 bg-gray-50 space-y-2">
                                    {/* 權限切換按鈕 */}
                                    <button 
                                        onClick={() => isAuthorized ? setIsAuthorized(false) : setShowPasswordModal(true)} 
                                        className={`w-full flex items-center justify-center gap-2 text-xs py-2 px-2 border rounded transition-colors ${isAuthorized ? 'bg-red-50 border-red-200 text-red-600 hover:bg-red-100' : 'bg-white border-gray-300 text-gray-600 hover:bg-gray-50'}`}
                                    >
                                        {isAuthorized ? <Icons.Unlock /> : <Icons.Lock />}
                                        {isAuthorized ? '登出管理員' : '管理員登入'}
                                    </button>

                                    <button onClick={() => setUseRealApi(!useRealApi)} className="w-full text-xs py-1.5 px-2 border border-gray-300 rounded bg-white text-gray-600 hover:bg-gray-50">
                                        {useRealApi ? '切換演示模式' : '切換 API 模式'}
                                    </button>
                                    
                                    {useRealApi && isAuthorized && (
                                        <div className="grid grid-cols-1 gap-2">
                                            <button onClick={() => setShowDebug(!showDebug)} className={`flex items-center justify-center gap-1 text-xs py-1.5 px-2 border border-gray-200 rounded bg-white transition-colors ${showDebug ? 'text-green-600 bg-green-50 border-green-200' : 'text-gray-500 hover:text-gray-700'}`}>
                                                <Icons.Bug />
                                                {showDebug ? '關閉除錯' : '除錯模式'}
                                            </button>
                                        </div>
                                    )}
                                </div>
                            </div>
                        </aside>
                        {isMobileMenuOpen && <div onClick={() => setIsMobileMenuOpen(false)} className="fixed inset-0 z-10 bg-gray-800 bg-opacity-50 lg:hidden"></div>}

                        <main className="flex-1 overflow-y-auto custom-scrollbar h-full bg-gray-50/50 w-full">
                            <div className="max-w-5xl mx-auto px-4 sm:px-8 py-8">
                                <div className="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-8">
                                    <div>
                                        <div className="flex items-center gap-2 mb-1">
                                            <h2 className="text-2xl font-bold text-gray-900 flex items-center gap-2">
                                                {activeCategory === 'recent' ? '最近修改' : 
                                                 activeCategory === 'all' ? '全部檔案' : 
                                                 activeCategory === 'uncategorized' ? '未分類檔案' : 
                                                 `# ${activeCategory}`}
                                            </h2>
                                            {!isAuthorized && (
                                                <span className="inline-flex items-center px-2 py-0.5 rounded text-xs font-medium bg-yellow-100 text-yellow-800">
                                                    訪客模式 (#share)
                                                </span>
                                            )}
                                        </div>
                                        <p className="text-sm text-gray-500">共 {processedData.totalItems} 個項目</p>
                                    </div>
                                    <div className="flex flex-col sm:flex-row gap-3 items-stretch sm:items-center">
                                        <div className="relative group flex-1">
                                            <div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none"><Icons.Search /></div>
                                            <input 
                                                type="text" 
                                                value={searchTerm} 
                                                onChange={(e) => setSearchTerm(e.target.value)} 
                                                className="block w-full pl-10 pr-10 py-2 border border-gray-200 rounded-lg bg-white focus:ring-2 focus:ring-blue-500 outline-none text-sm" 
                                                placeholder="搜尋... (如 #工作 & #急件)" 
                                            />
                                            <div className="absolute inset-y-0 right-0 pr-3 flex items-center cursor-help has-tooltip">
                                                <div className="text-gray-400 hover:text-blue-500"><Icons.Info /></div>
                                                <div className="tooltip">
                                                    <p className="font-bold mb-1 text-blue-300">進階標籤搜尋語法：</p>
                                                    <ul className="list-disc list-inside space-y-1">
                                                        <li><code className="bg-gray-700 px-1 rounded">&</code> (AND): 同時包含</li>
                                                        <li><code className="bg-gray-700 px-1 rounded">|</code> (OR): 包含任一</li>
                                                        <li><code className="bg-gray-700 px-1 rounded">!</code> (NOT): 排除</li>
                                                        <li><code className="bg-gray-700 px-1 rounded">()</code>: 優先權</li>
                                                    </ul>
                                                </div>
                                            </div>
                                        </div>
                                        {activeCategory !== 'recent' && (
                                            <>
                                                <div className="h-8 w-px bg-gray-300 hidden sm:block mx-1"></div>
                                                <div className="flex gap-2">
                                                    <div className="relative">
                                                        <select value={sortField} onChange={(e) => setSortField(e.target.value)} className="appearance-none w-full sm:w-32 bg-white border border-gray-200 text-gray-700 text-sm py-2 pl-3 pr-8 rounded-lg focus:ring-2 focus:ring-blue-500 cursor-pointer outline-none">
                                                            <option value="modified_at">最近修改</option>
                                                            <option value="created_at">創建時間</option>
                                                            <option value="title">檔案名稱</option>
                                                        </select>
                                                        <div className="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-500"><Icons.ArrowDown /></div>
                                                    </div>
                                                    <button onClick={() => setSortOrder(o => o === 'asc' ? 'desc' : 'asc')} className="p-2 bg-white border border-gray-200 rounded-lg text-gray-600 hover:text-blue-600">{sortOrder === 'asc' ? <Icons.SortAsc /> : <Icons.SortDesc />}</button>
                                                </div>
                                            </>
                                        )}
                                        {useRealApi && <button onClick={fetchDiagrams} disabled={loading} className="p-2 bg-white border border-gray-200 rounded-lg text-blue-600 hover:bg-blue-50"><Icons.Refresh spin={loading} /></button>}
                                    </div>
                                </div>
                                
                                {/* Main Content Area - Grid/Cards */}
                                {error && <div className="mb-6 p-4 bg-red-50 text-red-700 rounded-lg border border-red-100">{error}</div>}
                                
                                {processedData.totalItems === 0 ? (
                                    <div className="text-center py-20 animate-fade-in">
                                        <div className="inline-flex items-center justify-center w-20 h-20 rounded-full bg-gray-100 mb-4 text-gray-300"><Icons.Grid /></div>
                                        <h3 className="text-lg font-medium text-gray-900">沒有檔案</h3>
                                        <p className="text-gray-500 mt-2 text-sm">
                                            {isAuthorized ? "確認是否有在 Mindomo 描述欄位加上 #標籤 ?" : "目前沒有公開分享 (#share) 的檔案"}
                                        </p>
                                    </div>
                                ) : (
                                    <div className="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-6">
                                        {processedData.items.map((item) => (
                                            <div key={item.id} className="group bg-white rounded-xl border border-gray-200 p-5 transition-all duration-200 card-hover flex flex-col h-full relative overflow-hidden">
                                                <div className="absolute top-0 left-0 w-1 h-full bg-blue-500 opacity-0 group-hover:opacity-100 transition-opacity"></div>
                                                <div className="flex justify-between items-start mb-3">
                                                    <div className="p-2.5 bg-blue-50 text-blue-600 rounded-lg group-hover:bg-blue-600 group-hover:text-white transition-colors"><Icons.File /></div>
                                                    <div className="flex gap-2">
                                                        {useRealApi && <a href={`https://www.mindomo.com/mindmap/${item.id}`} target="_blank" className="text-xs font-medium text-blue-600 bg-blue-50 px-2.5 py-1.5 rounded-full hover:bg-blue-100 transition-colors flex items-center">開啟</a>}
                                                    </div>
                                                </div>
                                                
                                                <h3 className="text-lg font-bold text-gray-900 mb-2 line-clamp-2 leading-snug group-hover:text-blue-600 transition-colors">{item.title}</h3>
                                                
                                                {/* 顯示標籤區塊 */}
                                                <div className="flex flex-wrap gap-1 mb-4 min-h-[24px]">
                                                    {item.tags && item.tags.length > 0 ? (
                                                        item.tags.map(tag => (
                                                            <span key={tag} onClick={(e) => { e.stopPropagation(); setActiveCategory(tag); }} className="cursor-pointer inline-flex items-center px-2 py-0.5 rounded-full text-xs font-medium bg-gray-100 text-gray-600 hover:bg-blue-100 hover:text-blue-600 transition-colors">
                                                                #{tag}
                                                            </span>
                                                        ))
                                                    ) : (
                                                        <span className="text-xs text-gray-400 italic">無標籤</span>
                                                    )}
                                                </div>

                                                <div className="mt-auto pt-4 border-t border-gray-50 space-y-2">
                                                    <div className="flex items-center text-xs text-gray-500">
                                                        <Icons.Clock />
                                                        <span className="ml-2">
                                                            {item.modified_at ? `最後修改於 ${formatDate(item.modified_at)}` : '無修改日期'}
                                                        </span>
                                                    </div>
                                                    
                                                    <div className="flex gap-2 pt-2 border-t border-gray-100 justify-end">
                                                        <button onClick={(e) => handleDownload(e, item, 'md')} disabled={downloadingState && downloadingState.id === item.id} className="flex items-center gap-1 px-2 py-1 text-xs font-medium text-gray-600 bg-gray-50 hover:bg-gray-100 rounded transition-colors disabled:opacity-50" title="下載 Markdown (.md)">{downloadingState && downloadingState.id === item.id && downloadingState.format === 'md' ? <Icons.Refresh spin={true} /> : <span>MD</span>}</button>
                                                        <button onClick={(e) => handleDownload(e, item, 'mm')} disabled={downloadingState && downloadingState.id === item.id} className="flex items-center gap-1 px-2 py-1 text-xs font-medium text-gray-600 bg-gray-50 hover:bg-gray-100 rounded transition-colors disabled:opacity-50" title="下載 Freemind (.mm)">{downloadingState && downloadingState.id === item.id && downloadingState.format === 'mm' ? <Icons.Refresh spin={true} /> : <span>MM</span>}</button>
                                                        <button onClick={(e) => handleDownload(e, item, 'pdf')} disabled={downloadingState && downloadingState.id === item.id} className="flex items-center gap-1 px-2 py-1 text-xs font-medium text-gray-600 bg-gray-50 hover:bg-gray-100 rounded transition-colors disabled:opacity-50" title="下載 PDF (.pdf)">{downloadingState && downloadingState.id === item.id && downloadingState.format === 'pdf' ? <Icons.Refresh spin={true} /> : <span>PDF</span>}</button>
                                                    </div>
                                                </div>
                                            </div>
                                        ))}
                                    </div>
                                )}
                                <Pagination />
                                
                                {/* 密碼輸入視窗 */}
                                {showPasswordModal && (
                                    <PasswordModal 
                                        onSubmit={() => {
                                            setIsAuthorized(true);
                                            setShowPasswordModal(false);
                                        }}
                                        onClose={() => setShowPasswordModal(false)}
                                    />
                                )}

                                {useRealApi && rawData && showDebug && (
                                    <div className="mt-16 p-6 bg-gray-900 text-gray-300 rounded-xl shadow-lg animate-fade-in">
                                        <div className="flex items-center justify-between mb-4 border-b border-gray-700 pb-4">
                                            <div className="flex items-center gap-2">
                                                <Icons.Bug />
                                                <h3 className="text-lg font-bold text-white">API 資料除錯 (Debug Info)</h3>
                                            </div>
                                            <button onClick={() => setShowDebug(false)} className="text-xs text-gray-500 hover:text-white">關閉</button>
                                        </div>
                                        <p className="text-sm mb-4 text-gray-400">
                                            請檢查 JSON 中 <code>description</code> 欄位是否有包含 #標籤。
                                        </p>
                                        
                                        <div className="space-y-6">
                                            <div>
                                                <h4 className="text-xs font-bold text-blue-400 uppercase tracking-wider mb-2">原始資料 (前 1 筆範例)</h4>
                                                <pre className="bg-black p-4 rounded-lg text-xs overflow-x-auto font-mono border border-gray-800 text-green-400">
                                                    {JSON.stringify(Array.isArray(rawData) ? rawData[0] : (rawData.diagrams ? rawData.diagrams[0] : rawData), null, 2)}
                                                </pre>
                                            </div>
                                            
                                            <div>
                                                <h4 className="text-xs font-bold text-blue-400 uppercase tracking-wider mb-2">資料結構統計</h4>
                                                <div className="bg-black p-4 rounded-lg text-xs font-mono border border-gray-800">
                                                    <p>總筆數: {Array.isArray(rawData) ? rawData.length : (rawData.diagrams ? rawData.diagrams.length : 0)}</p>
                                                    <p>包含 'description' 欄位的筆數: {(Array.isArray(rawData) ? rawData : (rawData.diagrams || [])).filter(i => i.description).length}</p>
                                                    <p>偵測到的標籤總數: {sortedTags.length}</p>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                )}
                            </div>
                        </main>
                    </div>
                </div>
            );
        }

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
```