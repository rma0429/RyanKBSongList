<%*
// 1. 取得歌手名字與頁面資料
const singerName = tp.file.title;
const dv = app.plugins.plugins.dataview.api;
const pages = dv.pages(`#Singer/${singerName}`)
    .filter(p => p.file.folder.includes("Ryan KB-Song List"))
    .sort(p => p.file.name, "asc");

const songCount = pages.length;

// 2. 準備變數存儲
let tableBody = "";
let referenceLinks = "";

// 3. 處理每一首歌
pages.forEach((p, i) => {
    // 優先使用屬性 title，若無則用檔名
    let displayTitle = p.title ? p.title : p.file.name;
    
    // 獲取標籤資料 (Key, 節奏, BPM)
    let key = p.file.etags
        .filter(t => t.startsWith(`#Singer/${singerName}`))
        .map(t => t.split("/")[2] || "-").join(", ");
    let styles = p.file.etags.filter(t => t.startsWith("#Style/"));
    let rhythm = styles.map(t => t.split("/")[1] || "-").join("<br>");
    let bpm = styles.map(t => t.split("/")[2] || "-").join("<br>");

    // 構建 Quartz 絕對路徑
    let rawPath = p.file.path
        .replace(".md", "")
        .replace(/^content\//, "")
        .replace(/ /g, "-");
    let quartzUrl = "https://rma0429.github.io/RyanKBSongList/" + encodeURI(rawPath);

    // 生成表格行：使用 [歌名][song-i] 格式避開符號衝突
    tableBody += `| [${displayTitle}][song-${i}] | ${key} | ${rhythm} | ${bpm} |\n`;
    
    // 生成頁尾引用定義
    referenceLinks += `[song-${i}]: ${quartzUrl}\n`;
});

// 4. 組合最終輸出
tR += `# 🎤 ${singerName} 的所有歌曲 (共 ${songCount} 首)\n\n`;
tR += "| 歌名 | Key | 節奏 | BPM |\n";
tR += "| :--- | :--- | :--- | :--- |\n";
tR += tableBody;
tR += "\n\n" + referenceLinks;
%>