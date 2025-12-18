<%*
// ==========================================
//  0. 自動抓取歌手邏輯 (增強版)
//  針對格式： "歌名 - 歌手" (抓取最後面的文字)
// ==========================================
var file = tp.file.find_tfile(tp.file.path(true));

// [修正1] 改用 tp.file.title (讀檔名)，比讀屬性更穩定
var fTitle = tp.file.title; 
var fArtist = tp.frontmatter.org_artist;

// [修正2] 定義分隔符號的正則表達式 (同時支援 - 和 – 和 －)
var separatorRegex = /[-–—－]/;

// 條件：只有當 "org_artist" 為空，且標題包含分隔符號時才運作
if ((!fArtist || fArtist === "") && fTitle && separatorRegex.test(fTitle)) {
    
    // 使用正則表達式切割，抓取最後一段
    var parts = fTitle.split(separatorRegex);
    var extractedArtist = parts[parts.length - 1].trim();
    
    // 1. 寫入檔案屬性 (永久儲存)
    await app.fileManager.processFrontMatter(file, (frontmatter) => {
        frontmatter["org_artist"] = extractedArtist;
    });
    
    // 2. 更新記憶體中的變數 (讓下方的 Info 面板立刻顯示剛抓到的歌手)
    if (!tp.frontmatter) tp.frontmatter = {};
    tp.frontmatter.org_artist = extractedArtist;
}

// ==========================================
//  Templater V.14: 修正合唱 Tag 版 (保持不變)
// ==========================================

// --- 1. 變數初始化 ---
// 重新抓取一次 frontmatter 確保拿到最新寫入的資料
var fm = tp.frontmatter || {};
var tags = fm.tags || [];
if (typeof tags === 'string') { tags = [tags]; }
var globalBpm = fm.bpm; 
var ytLink = fm.yt_link || "";
var orgKey = "未設定";

// --- 2. 智慧解析原唱 (改良版) ---
var rawArtist = fm.org_artist; 
var artistList = [];
var orgArtistDisplay = "未設定";
var isDuet = false; 

// A. 處理輸入資料 (支援逗號與 & 符號混用)
if (rawArtist) {
    if (Array.isArray(rawArtist)) {
        artistList = rawArtist;
    } else if (typeof rawArtist === 'string' && rawArtist.trim() !== "") {
        // 使用正則表達式：同時支援 "," 和 "&" 切割，並過濾掉空白項目
        artistList = rawArtist.split(/[,&]/)
            .map(function(s) { return s.trim(); })
            .filter(function(s) { return s !== ""; });
    }
}

// B. 產生顯示字串
if (artistList.length > 0) {
    orgArtistDisplay = artistList.map(function(name) {
        return "[[" + "歌手/原唱/" + name + "|" + name + "]]";
    }).join(" & ");
    
    // 判斷是否超過一人
    if (artistList.length > 1) {
        isDuet = true;
    }
}

// --- 3. 解析其他 Tag (Singer, Style...) ---
var singerListMarkdown = ""; 
var styleListMarkdown = ""; 

if (Array.isArray(tags)) {
    // 抓 Key
    var tKey = tags.find(function(t) { return t.indexOf("原Key/") >= 0; });
    if (tKey) orgKey = tKey.replace("#", "").replace("原Key/", "");

    // 抓 Singer
    var singerTags = tags.filter(function(t) { return t.indexOf("Singer/") >= 0; });
    if (singerTags.length > 0) {
        singerTags.forEach(function(t) {
            var clean = t.replace("#", "").replace("Singer/", "");
            var parts = clean.split("/");
            var name = parts[0] || "未設定";
            var key = parts[1] || "無";
            singerListMarkdown += "> - **🎤 演唱：** [[" + "歌手/演唱/" + name + "|" + name + "]] (" + key + ")\n";
        });
    } else {
        singerListMarkdown = "> - **🎤 演唱：** 未設定\n";
    }

    // 抓 Style
    var styleTags = tags.filter(function(t) { return t.indexOf("Style/") >= 0; });
    if (fm.style_number) { styleTags.push("Style/" + fm.style_number); }

    if (styleTags.length > 0) {
        // 讀取 Style 資料庫
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

// ==========================================
//  4. 輸出 HTML
// ==========================================

// 處理 YT 按鈕
var ytDisplay = "";
if (ytLink && ytLink.length > 0) {
    ytDisplay = " &nbsp; [🎧 原曲聆聽](" + ytLink + ")";
}

// 處理 合唱 Tag
var duetTag = "";
if (isDuet) {
    duetTag = " #合唱"; 
}

tR += "> [!info] \n";
tR += "> -  原唱： " + orgArtistDisplay + duetTag + " (原 Key: " + orgKey + ")" + ytDisplay + "\n";

// 分隔線
tR += "> \n> <hr style=\"margin: 0.5em 0; border-color: rgba(255,255,255,0.2);\">\n> \n";

tR += singerListMarkdown;
tR += "> \n> <hr style=\"margin: 0.5em 0; border-color: rgba(255,255,255,0.2);\">\n> \n";

tR += styleListMarkdown; 
tR += "\n";
%>
<div style="display: flex; gap: 2em; align-items: flex-start; width: 100%;">

<div style="flex: 1; padding-right: 1em; border-right: 1px solid #3d3d3d; min-width: 0; white-space: pre-wrap;">📄 Lyric

</div>

<div style="flex: 1; padding-left: 1em; min-width: 0; white-space: pre-wrap;">🎵 Note 

ＩＮＴ：
Ｖ．： 
ＰＣ： 
Ｃ﹒： 
Ｂ．： 
ＯＵＴ：

</div>

</div>
