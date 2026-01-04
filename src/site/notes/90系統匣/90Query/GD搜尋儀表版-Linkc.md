---
{"dg-publish":true,"permalink":"/90系統匣/90Query/GD搜尋儀表版-Linkc/","dgPassFrontmatter":true,"noteIcon":"","created":"2025-12-13T14:27:28.210+08:00","updated":"2025-12-13T19:36:34.736+08:00"}
---


```dataviewjs
// ================= 設定區 =================
const indexFile = "/Drive_Index/Drive_Master_Index.md";  // 您的索引檔名
// 移除 targetFolders 設定，改為自動抓取所有內容
// ==========================================

const container = this.container;
container.innerHTML = "";
container.style.fontFamily = "var(--font-interface)";

// 1. 讀取並檢查檔案
let content;
try {
    content = await dv.io.load(indexFile);
} catch (e) {
    content = null;
}

if (!content) {
    container.createEl("div", { 
        text: `⚠️ 無法讀取 "${indexFile}"。請確認 GAS 腳本已執行，且檔案已同步。`, 
        attr: { style: "color: var(--text-error); padding: 20px; border: 1px dashed red;" } 
    });
} else {
    // 2. 抓取更新時間
    let updateTime = "未知";
    const timeMatch = content.match(/🔄 最後同步時間：(.*)/);
    if (timeMatch) updateTime = timeMatch[1].trim();

    // 3. 建立標題列
    const headerEl = container.createEl("div", { attr: { style: "display: flex; justify-content: space-between; align-items: flex-end; margin-bottom: 15px; border-bottom: 2px solid var(--interactive-accent); padding-bottom: 5px;" } });
    headerEl.createEl("h2", { text: "🔍 檔案搜尋儀表板", attr: { style: "margin: 0;" } });
    headerEl.createEl("span", { text: `📅 資料時間: ${updateTime}`, attr: { style: "font-size: 0.8em; color: var(--text-muted);" } });

    // 4. 搜尋框
    const inputEl = container.createEl("input", {
        type: "text",
        placeholder: "輸入關鍵字... (支援: 南澳 預算 | pdf -2023)",
        attr: { style: "width: 100%; padding: 10px; font-size: 1.1em; border: 1px solid var(--background-modifier-border); border-radius: 5px; background: var(--background-primary); margin-bottom: 10px;" }
    });

    const resultsEl = container.createEl("div");

    // 5. 解析資料 (不再過濾目錄，有什麼就顯示什麼)
    const files = [];
    const regex = /file_name: (.*)\nurl: (.*)\npath: (.*)/g;
    let match;
    
    // 統計根目錄，用來顯示摘要
    const rootFolders = new Set();

    while ((match = regex.exec(content)) !== null) {
        const fName = match[1].trim();
        const fUrl = match[2].trim();
        const fPath = match[3].trim(); // 完整路徑，例如 "00PARA/01專案/..."

        // 切割路徑
        const parts = fPath.split("/").filter(p => p && p.trim() !== "");
        
        // 記錄第一層目錄名稱 (為了統計用)
        if (parts.length > 0) rootFolders.add(parts[0]);

        files.push({
            name: fName,
            url: fUrl,
            parts: parts,
            fullPath: fPath
        });
    }

    // 初始化顯示 (目錄樹)
    renderView("");

    // 監聽輸入
    inputEl.addEventListener("input", (e) => {
        renderView(e.target.value);
    });

    // --- 渲染控制 ---
    function renderView(keyword) {
        resultsEl.innerHTML = "";
        if (!keyword || keyword.trim() === "") {
            renderTreeMode();
        } else {
            renderSearchMode(keyword);
        }
    }

    // --- 模式 A: 搜尋清單 ---
    function renderSearchMode(keyword) {
        const rawTerms = keyword.toLowerCase().split(" ");
        const matched = files.filter(f => {
            const text = (f.name + " " + f.fullPath).toLowerCase();
            return rawTerms.every(term => {
                if (term.startsWith("-")) return !text.includes(term.substring(1)); // 排除
                if (term.includes("|")) return term.split("|").some(t => text.includes(t)); // OR
                return text.includes(term); // AND
            });
        });

        if (matched.length === 0) {
            resultsEl.createEl("div", { text: "❌ 找不到符合條件的檔案", attr: { style: "padding: 10px; color: var(--text-muted);" } });
            return;
        }

        resultsEl.createEl("div", { text: `📊 找到 ${matched.length} 筆資料`, attr: { style: "font-weight: bold; margin-bottom: 5px; color: var(--text-accent);" } });

        const limit = 100; // 顯示限制
        const listEl = resultsEl.createEl("div");
        
        matched.slice(0, limit).forEach(f => {
            const row = listEl.createEl("div", { attr: { style: "padding: 5px 0; border-bottom: 1px solid var(--background-modifier-border);" } });
            const icon = getIcon(f.name);
            row.innerHTML = `${icon} <a href="${f.url}" target="_blank" style="font-weight: 500; font-size: 1.05em;">${f.name}</a>`;
            row.createEl("div", { text: f.fullPath, attr: { style: "font-size: 0.8em; color: var(--text-muted); margin-left: 22px;" } });
        });
        
        if (matched.length > limit) {
            resultsEl.createEl("div", { text: `(還有 ${matched.length - limit} 筆未顯示...)`, attr: { style: "color: var(--text-muted); font-style: italic; margin-top: 5px;" } });
        }
    }

    // --- 模式 B: 目錄樹 ---
    function renderTreeMode() {
        // 顯示統計資訊
        const rootList = Array.from(rootFolders).sort().join(", ");
        resultsEl.createEl("div", { 
            text: `🗂️ 索引範圍：包含 [${rootList}] 等共 ${files.length} 個檔案`, 
            attr: { style: "font-size: 0.85em; color: var(--text-muted); margin-bottom: 10px; border-bottom: 1px dashed var(--background-modifier-border); padding-bottom: 5px;" } 
        });

        const root = { _sub: {}, _files: [], _count: 0 };
        for (const f of files) {
            let curr = root;
            for (const folder of f.parts) {
                if (!curr._sub[folder]) curr._sub[folder] = { _sub: {}, _files: [], _count: 0 };
                curr = curr._sub[folder];
                curr._count++;
            }
            curr._files.push(f);
        }

        let html = "";
        for (const key of Object.keys(root._sub).sort()) {
            html += renderNode(root._sub[key], key, 1);
        }
        
        const treeContent = resultsEl.createEl("div");
        treeContent.innerHTML = html || "<p style='padding:10px;'>沒有資料，請確認索引檔內容。</p>";
    }

    function renderNode(node, name, depth) {
        // 第一層強制展開
        const openAttr = depth < 2 ? "open" : "";
        
        // 視覺樣式優化
        let summaryStyle = "cursor: pointer; font-weight: 600; color: var(--text-accent); padding: 4px 8px;";
        let borderStyle = "1px solid rgba(130,130,130,0.2)";
        
        // 如果是根目錄 (例如 00PARA)，樣式加強
        if (depth === 1) {
             borderStyle = "3px solid var(--interactive-accent)";
             summaryStyle += " font-size: 1.1em;";
        }

        const count = node._count > 0 ? `<span style="font-size:0.8em; color:gray; margin-left:5px;">(${node._count})</span>` : "";
        
        let html = `<details ${openAttr} style="margin-left: 10px; border-left: ${borderStyle}; margin-bottom: 2px;">
            <summary style="${summaryStyle}">📂 ${name} ${count}</summary>`;

        for (const sub of Object.keys(node._sub).sort()) {
            html += renderNode(node._sub[sub], sub, depth + 1);
        }

        if (node._files.length > 0) {
            html += `<div style="margin-left: 24px; padding: 2px 0;">`;
            for (const f of node._files) {
                html += `<div style="padding: 2px 0;">${getIcon(f.name)} <a href="${f.url}" target="_blank" style="text-decoration: none; color: var(--text-normal);">${f.name}</a></div>`;
            }
            html += `</div>`;
        }
        return html + "</details>";
    }

    function getIcon(n) {
        if (n.endsWith(".pdf")) return "📕";
        if (n.match(/\.(doc|docx)$/i)) return "📝";
        if (n.match(/\.(xls|xlsx|csv)$/i)) return "📊";
        if (n.match(/\.(ppt|pptx)$/i)) return "📢";
        if (n.match(/\.(jpg|png|jpeg|gif)$/i)) return "🖼️";
        if (n.match(/\.(mp4|mov)$/i)) return "🎬";
        if (n.match(/\.(zip|rar|7z)$/i)) return "📦";
        return "📄";
    }
}