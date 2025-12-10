---
title: Template test
tags:
  - 原唱/周杰倫
  - 原Key/E
  - Singer/小丰/G
style_number: "110"
bpm: "100"
---

<%*
// --- 1. 讀取目前的 Tags (Templater 版本) ---
// 注意：如果是新建檔案，有時候 tags 還沒寫入 cache，建議填完 tag 後再手動執行模板
const currentFile = tp.file.find_tfile(tp.file.path(true));
const cache = app.metadataCache.getFileCache(currentFile);
const tags = cache?.tags?.map(t => t.tag) || [];

// --- 2. 工具函式：從 Tag 抓資料 ---
function getTagVal(prefix) {
    const found = tags.find(t => t.startsWith(prefix));
    return found ? found.substring(prefix.length) : "未設定";
}

function parseNestedTag(prefix) {
    const found = tags.find(t => t.startsWith(prefix));
    if (!found) return { name: "未設定", key: "無" };
    const content = found.substring(prefix.length);
    const parts = content.split("/"); 
    return { name: parts[0] || "未設定", key: parts[1] || "無" };
}

// 執行抓取
const orgArtist = getTagVal("#原唱/");
const orgKey    = getTagVal("#原Key/");
const coverInfo = parseNestedTag("#Singer/"); // 抓取 #Singer/名字/Key

// --- 3. 讀取節奏資料庫 ---
const styleNum = tp.frontmatter.style_number;
const bpm = tp.frontmatter.bpm;
const dbPath = "content/E-A7_Styles.md"; 
let styleName = "未選擇";

const styleFile = app.vault.getAbstractFileByPath(dbPath);
if (styleFile) {
    const fileCache = app.metadataCache.getFileCache(styleFile);
    if (fileCache?.frontmatter?.E_A7_Styles) {
        const data = fileCache.frontmatter.E_A7_Styles;
        if (data && data[styleNum]) {
            styleName = data[styleNum].name;
            // 順便把自動抓到的名稱寫回屬性
            await tp.file.updateFrontmatter({ style_name: styleName });
        }
    }
}
%>

# <%= tp.file.title %>

> [!info] 歌曲資訊
> - **🎹 原唱：** [[歌手/原唱/<% orgArtist %>|<% orgArtist %>]] (原 Key: <% orgKey %>)
> - **🎤 演唱：** [[歌手/演唱/<% coverInfo.name %>|<% coverInfo.name %>]] (#<% coverInfo.key %>)
> - **🥁 節奏設定：** <% styleName %> - <% styleNum %> (BPM: <% bpm %>)

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
