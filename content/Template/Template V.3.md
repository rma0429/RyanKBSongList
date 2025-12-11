<%*
// ==========================================
//  Templater V.5: 直接讀取屬性版 (最速)
// ==========================================

// 1. 初始化變數
var styleName = "未選擇";
var displayRhythm = "";
var orgArtist = "未設定";
var orgKey = "未設定";
var coverName = "未設定";
var coverKey = "無";

// 2. 直接抓取當前檔案的屬性 (這是您剛剛填完 V2 後留下的)
var fm = tp.frontmatter || {};
var tags = fm.tags || [];
var styleNum = fm.style_number;
var bpm = fm.bpm;

// 防呆：如果只有一個 tag，把它轉成陣列
if (typeof tags === 'string') { tags = [tags]; }

// --- A. 解析歌手與 Key (直接分析文字) ---
if (Array.isArray(tags)) {
    // 找原唱
    var tOrg = tags.find(function(t) { return t.indexOf("原唱/") >= 0; });
    if (tOrg) orgArtist = tOrg.replace("#", "").replace("原唱/", "");

    // 找原Key
    var tKey = tags.find(function(t) { return t.indexOf("原Key/") >= 0; });
    if (tKey) orgKey = tKey.replace("#", "").replace("原Key/", "");

    // 找演唱者 (Singer/名字/Key)
    var tSinger = tags.find(function(t) { return t.indexOf("Singer/") >= 0; });
    if (tSinger) {
        var clean = tSinger.replace("#", "").replace("Singer/", "");
        var parts = clean.split("/");
        coverName = parts[0] || "未設定";
        coverKey = parts[1] || "無";
    }
}

// --- B. 處理節奏 (優先看 style_number 欄位) ---
var targetID = null;

if (styleNum) {
    targetID = styleNum;
} 
// 如果屬性沒填，嘗試從 Tag 找 #Style/110
else if (Array.isArray(tags)) {
    var tStyle = tags.find(function(t) { return t.indexOf("Style/") >= 0; });
    if (tStyle) {
        targetID = tStyle.replace("#", "").replace("Style/", "");
        styleNum = targetID; // 同步一下
    }
}

// --- C. 查資料庫 (維持您成功的路徑) ---
if (targetID) {
    // 狀況 1: 純文字 (Piano)
    if (isNaN(parseInt(targetID))) {
        styleName = "🎹 " + targetID;
        displayRhythm = styleName;
    } 
    // 狀況 2: 數字編號 (110)
    else {
        var dbPath = "content/Metadata/E-A7_Styles.md"; 
        var styleFile = app.vault.getAbstractFileByPath(dbPath);
        
        if (styleFile) {
            var meta = app.metadataCache.getFileCache(styleFile);
            if (meta && meta.frontmatter && meta.frontmatter.E_A7_Styles) {
                var db = meta.frontmatter.E_A7_Styles;
                var key = "" + targetID;
                if (db[key]) {
                    styleName = db[key].name;
                    // 順手更新屬性
                    var currentFile = tp.file.find_tfile(tp.file.path(true));
                    if (currentFile) {
                        await app.fileManager.processFrontMatter(currentFile, (fm) => {
                            fm['style_name'] = styleName;
                        });
                    }
                }
            }
        }
        displayRhythm = styleName + " - " + targetID;
    }
} else {
    displayRhythm = "未設定";
}

// 加上 BPM
if (bpm) {
    displayRhythm += " (BPM: " + bpm + ")";
}

// ==========================================
//  4. 輸出 HTML 內容
// ==========================================
%>
> [!info] 歌曲資訊
> - **🎹 原唱：** [[歌手/原唱/<% orgArtist %>|<% orgArtist %>]] (原 Key: <% orgKey %>)
> - **🎤 演唱：** [[歌手/演唱/<% coverName %>|<% coverName %>]] (#<% coverKey %>)
> - **🥁 節奏設定：** <% displayRhythm %>

<div style="display: flex; gap: 2em; align-items: start;">

  <div style="flex: 1; padding-right: 1em; border-right: 1px solid #3d3d3d;">
    ### 📄 歌詞
    ![[assets/歌詞截圖.png]]
  </div>

  <div style="flex: 1; padding-left: 1em;">
    ### 🎵 樂譜筆記
    - 前奏：
    - 間奏：
    - 尾奏：
  </div>

</div>