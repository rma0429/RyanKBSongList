
<%*
// ==========================================
//  Templater V.11: 原唱屬性 + YT按鈕 + 視覺分隔
// ==========================================

// --- 1. 變數初始化 ---
var fm = tp.frontmatter || {};
var tags = fm.tags || [];
var globalBpm = fm.bpm; 

// A. 讀取屬性：原唱 (支援空格) 與 YT連結
var orgArtist = fm.org_artist || "未設定";
var ytLink = fm.yt_link || "";
var orgKey = "未設定";

var singerListMarkdown = ""; 
var styleListMarkdown = ""; 

// 防呆：轉陣列
if (typeof tags === 'string') { tags = [tags]; }

if (Array.isArray(tags)) {
    
    // B. 解析原Key (維持 Tag 抓取)
    var tKey = tags.find(function(t) { return t.indexOf("原Key/") >= 0; });
    if (tKey) orgKey = tKey.replace("#", "").replace("原Key/", "");

    // C. 解析演唱者
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

    // D. 解析節奏
    var styleTags = tags.filter(function(t) { return t.indexOf("Style/") >= 0; });
    if (fm.style_number) {
        styleTags.push("Style/" + fm.style_number);
    }

    if (styleTags.length > 0) {
        // 讀取資料庫
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
            var displayText = "";

            if (isNaN(parseInt(sID))) {
                displayText = "🎹 " + sID; 
            } else {
                var sName = "未知節奏";
                if (dbData && dbData[sID]) {
                    sName = dbData[sID].name;
                }
                displayText = "🥁 " + sName + " - " + sID;
            }

            if (sBpm) {
                displayText += " (BPM: " + sBpm + ")";
            }
            styleListMarkdown += "> - " + displayText + "\n";
        });

    } else {
        styleListMarkdown = "> - 🥁 節奏設定：未選擇\n";
    }
}

// ==========================================
//  4. 輸出 HTML
// ==========================================

// 處理 YT 按鈕顯示邏輯
var ytDisplay = "";
if (ytLink && ytLink.length > 0) {
    // 增加一個空格與按鈕
    ytDisplay = " &nbsp; [📺 原曲連結](" + ytLink + ")";
}

tR += "> [!info] \n";

// 輸出：原唱 + Key + YT按鈕
tR += "> -  原唱： [[歌手/原唱/" + orgArtist + "|" + orgArtist + "]] (原 Key: " + orgKey + ")" + ytDisplay + "\n";

// 分隔線 1
tR += "> \n"; 
tR += "> <hr style=\"margin: 0.5em 0; border-color: rgba(255,255,255,0.2);\">\n";
tR += "> \n";

tR += singerListMarkdown;

// 分隔線 2
tR += "> \n";
tR += "> <hr style=\"margin: 0.5em 0; border-color: rgba(255,255,255,0.2);\">\n";
tR += "> \n";

tR += styleListMarkdown; 
tR += "\n";
%>