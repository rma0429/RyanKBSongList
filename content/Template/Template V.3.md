<%*
// ==========================================
//  Templater V.9: 視覺分隔優化版 (加入分隔線)
// ==========================================

// --- 1. 變數初始化 ---
var orgArtist = "未設定";
var orgKey = "未設定";
var singerListMarkdown = ""; 
var styleListMarkdown = ""; 

// --- 2. 讀取 Frontmatter ---
var fm = tp.frontmatter || {};
var tags = fm.tags || [];
var globalBpm = fm.bpm; 

// 防呆
if (typeof tags === 'string') { tags = [tags]; }

if (Array.isArray(tags)) {
    
    // A. 解析原唱
    var tOrg = tags.find(function(t) { return t.indexOf("原唱/") >= 0; });
    if (tOrg) orgArtist = tOrg.replace("#", "").replace("原唱/", "");

    var tKey = tags.find(function(t) { return t.indexOf("原Key/") >= 0; });
    if (tKey) orgKey = tKey.replace("#", "").replace("原Key/", "");

    // B. 解析演唱者
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

    // C. 解析節奏
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
//  4. 輸出 HTML (加入分隔線 <hr>)
// ==========================================

tR += "> [!info] 歌曲資訊\n";
tR += "> - ** 原唱：** [[歌手/原唱/" + orgArtist + "|" + orgArtist + "]] (原 Key: " + orgKey + ")\n";

// 分隔線區塊 1 (上下加 > \n)
tR += "> \n"; 
tR += "> <hr style=\"margin: 0.5em 0; border-color: rgba(255,255,255,0.2);\">\n";
tR += "> \n";

tR += singerListMarkdown;

// 分隔線區塊 2 (上下加 > \n)
tR += "> \n";
tR += "> <hr style=\"margin: 0.5em 0; border-color: rgba(255,255,255,0.2);\">\n";
tR += "> \n";

tR += styleListMarkdown; 
tR += "\n";
%>
<div style="display: flex; gap: 2em; align-items: flex-start; width: 100%;">

<div style="flex: 1; padding-right: 1em; border-right: 1px solid #3d3d3d; min-width: 0;">

📄 Lyric


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
