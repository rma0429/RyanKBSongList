<%*
// ==========================================
//  0. 增強版：完整支援多位歌手 (陣列/字串通吃)
// ==========================================
var file = tp.file.find_tfile(tp.file.path(true));

// --- 步驟 A: 取得歌手名單 (標準化為陣列) ---
var rawArtist = tp.frontmatter.org_artist; 
var targetArtists = [];

// 1. 判斷現有的 org_artist 格式
if (Array.isArray(rawArtist)) {
    // 狀況一：已經是陣列 (如截圖中的多位歌手)
    targetArtists = rawArtist;
} else if (typeof rawArtist === 'string' && rawArtist.trim() !== "") {
    // 狀況二：是單一字串，嘗試切分 (支援 , & 、)
    targetArtists = rawArtist.split(/[,&、]/).map(s => s.trim());
}

// 2. 如果上面沒抓到，嘗試從標題抓 (後備方案)
if (targetArtists.length === 0) {
    var sourceTitle = tp.frontmatter.title || tp.file.title; 
    var separatorRegex = /[-–—－]/; 
    
    if (sourceTitle && separatorRegex.test(sourceTitle)) {
        var parts = sourceTitle.split(separatorRegex);
        var suffix = parts[parts.length - 1].trim();
        // 將抓到的後綴再次切分 (例如 "蔡依林&陶喆" -> ["蔡依林", "陶喆"])
        targetArtists = suffix.split(/[,&、]/).map(s => s.trim());
    }
}

// --- 步驟 B: 執行寫入 (將名單同步到 Tags) ---
if (targetArtists.length > 0) {
    await app.fileManager.processFrontMatter(file, (frontmatter) => {
        // 1. 確保 org_artist 格式統一 (建議存回陣列，方便 Obsidian 管理)
        frontmatter["org_artist"] = targetArtists;

        // 2. 處理 Tags
        let currentTags = frontmatter["tags"] || [];
        if (typeof currentTags === 'string') currentTags = [currentTags]; 
        if (!Array.isArray(currentTags)) currentTags = []; 

        // 3. 【關鍵修正】迴圈處理每一位歌手，避免塞入陣列物件
        targetArtists.forEach(artist => {
            if (artist && artist !== "") {
                // 防呆：避免重複加入
                if (!currentTags.includes(artist)) {
                    currentTags.push(artist);
                }
            }
        });
        
        // 寫回 Tags
        frontmatter["tags"] = currentTags;
    });

    // C. 手動更新記憶體變數 (讓下方顯示能立刻生效)
    if (!tp.frontmatter) tp.frontmatter = {};
    tp.frontmatter.org_artist = targetArtists;
}

// ==========================================
//  下方為顯示邏輯 (配合多歌手微調)
// ==========================================

// --- 1. 變數初始化 ---
var fm = tp.frontmatter || {};
var tags = fm.tags || [];
if (typeof tags === 'string') { tags = [tags]; }
var globalBpm = fm.bpm; 
var ytLink = fm.yt_link || "";
var orgKey = "未設定";

// --- 2. 處理歌手顯示 (轉成 #標籤) ---
var artistList = [];
var orgArtistDisplay = "未設定";
var isDuet = false; 

// 優先使用剛剛整理好的 targetArtists
if (targetArtists.length > 0) {
    artistList = targetArtists;
} else if (fm.org_artist) {
    // 雙重保險：如果上面沒跑(例如已經有資料)，這裡再抓一次
    var temp = fm.org_artist;
    if (Array.isArray(temp)) {
        artistList = temp;
    } else {
        artistList = temp.split(/[,&、]/).map(s => s.trim());
    }
}

// 產生顯示內容
if (artistList.length > 0) {
    orgArtistDisplay = artistList.map(function(name) {
        var safeTag = name.replace(/\s+/g, "_");
        // 強制指定完整路徑，繞過 Quartz 的自動計算
        return '<a href="/RyanKBSongList/tags/' + safeTag + '.html">' + name + '</a>';
    }).join(" & ");
    
    if (artistList.length > 1) { isDuet = true; }
}

// --- 3. 解析其他 Tag (Singer, Style...) ---
var singerListMarkdown = ""; 
var styleListMarkdown = ""; 

if (Array.isArray(tags)) {
    // 抓 Key
    var tKey = tags.find(t => t.indexOf("原Key/") >= 0);
    if (tKey) orgKey = tKey.replace("#", "").replace("原Key/", "");

    // 抓 Singer
    var singerTags = tags.filter(t => t.indexOf("Singer/") >= 0);
    if (singerTags.length > 0) {
        singerTags.forEach(function(t) {
            var clean = t.replace("#", "").replace("Singer/", "");
            var parts = clean.split("/");
            var name = parts[0] || "未設定";
            var key = parts[1] || "無";
            singerListMarkdown += "> - **🎤 演唱：** [[" + name + "]] (" + key + ")\n";
        });
    } else {
        singerListMarkdown = "> - **🎤 演唱：** 未設定\n";
    }

    // 抓 Style
    var styleTags = tags.filter(t => t.indexOf("Style/") >= 0);
    if (fm.style_number) { styleTags.push("Style/" + fm.style_number); }

    if (styleTags.length > 0) {
        var dbPath = "content/Metadata/E-A7_Styles.md"; 
        var styleFile = app.vault.getAbstractFileByPath(dbPath);
        var dbData = null;
        if (styleFile) {
            var meta = app.metadataCache.getFileCache(styleFile);
            if (meta && meta.frontmatter && meta.frontmatter.E_A7_Styles) {
                dbData = meta.frontmatter.E_A7_Styles;
            }
        }
        styleTags.forEach(function(t) {
            var raw = t.replace("#", "").replace("Style/", "");
            var parts = raw.split("/");
            var sID = parts[0]; 
            var sBpm = parts[1] || globalBpm || ""; 
            var displayText = isNaN(parseInt(sID)) ? "🎹 " + sID : "🥁 " + (dbData && dbData[sID] ? dbData[sID].name : "未知節奏") + " - " + sID;
            if (sBpm) displayText += " (BPM: " + sBpm + ")";
            styleListMarkdown += "> - " + displayText + "\n";
        });
    } else {
        styleListMarkdown = "> - 🥁 節奏設定：未選擇\n";
    }
}

// --- 4. 輸出 HTML ---
var ytDisplay = "";
if (ytLink && ytLink.length > 0) {
    ytDisplay = " &nbsp; [🎧 原曲聆聽](" + ytLink + ")";
}

var duetTag = "";
if (isDuet) {
    duetTag = " #合唱"; 
}

tR += "> [!info] \n";
tR += "> -  原唱： " + orgArtistDisplay + duetTag + " (原 Key: " + orgKey + ")" + ytDisplay + "\n";
tR += "> \n> <hr style=\"margin: 0.5em 0; border-color: rgba(255,255,255,0.2);\">\n> \n";
tR += singerListMarkdown;
tR += "> \n> <hr style=\"margin: 0.5em 0; border-color: rgba(255,255,255,0.2);\">\n> \n";
tR += styleListMarkdown; 
tR += "\n";
%>