---
title: Test song title
tags:
  - 原唱/周杰倫
  - 原Key/C
  - Singer/小丰/D
style_number: "110"
bpm: "100"
---


`$= 
// --- 1. 初始設定 ---
const page = dv.current();
const tags = page.file.tags || [];

// --- 2. 工具函式：抓取單層標籤 (給原唱用) ---
function getTagVal(prefix) {
    // 找到以 prefix 開頭的 tag (例如 "#原唱/")
    const found = tags.find(t => t.startsWith(prefix));
    return found ? found.substring(prefix.length) : "未設定";
}

// --- 3. 工具函式：抓取巢狀標籤 (給演唱者用：#Singer/名字/Key) ---
function parseNestedTag(prefix) {
    // 找到以 prefix 開頭的 tag (例如 "#Singer/")
    const found = tags.find(t => t.startsWith(prefix));
    
    if (!found) return { name: "未設定", key: "無" };

    // 移除前綴，剩下的字串 (例如 "小丰/Ab")
    const content = found.substring(prefix.length);
    const parts = content.split("/"); // 用斜線切開

    return {
        name: parts[0] || "未設定", // 第一段是名字
        key: parts[1] || "無"      // 第二段是 Key
    };
}

// --- 4. 執行抓取 ---
// 原唱部分：分開抓
const orgArtist = getTagVal("#原唱/");
const orgKey    = getTagVal("#原Key/");

// 演唱部分：一起抓 (使用 #Singer/ 前綴)
const coverInfo = parseNestedTag("#Singer/"); 

// --- 5. 讀取節奏資料 ---
const styleNum = page.style_number;
const bpm = page.bpm;
const db = dv.page("content/E-A7_Styles.md"); 
let styleName = "未選擇";

if (styleNum && db && db.E_A7_Styles && db.E_A7_Styles[styleNum]) {
    styleName = db.E_A7_Styles[styleNum].name;
}

// --- 6. 輸出顯示 (HTML) ---
dv.paragraph(`
> [!info] 歌曲資訊
> - **🎹 原唱：** [[歌手/原唱/${orgArtist}|${orgArtist}]] (原 Key: ${orgKey})
> - **🎤 演唱：** [[歌手/演唱/${coverInfo.name}|${coverInfo.name}]] (#${coverInfo.key})
> - **🥁 節奏設定：** ${styleName} - ${styleNum || "?"} (BPM: ${bpm || "?"})
`);
`

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
