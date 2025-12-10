---
title: "{{title}}"
tags: 
style_number: ""   # 輸入節奏編號 (例: 61)
bpm: ""            # 輸入速度 (例: 120)
original_artist: "" # 輸入原唱 (例: 周杰倫)
original_key: ""    # 輸入原 Key (例: G)
cover_artist: ""    # 輸入翻唱 (例: Ryan)
cover_key: ""       # 輸入翻唱 Key (例: C)
style_name: ""      # (自動填入，勿動)
---

<%*
// --- 自動讀取節奏資料庫 ---
const dbPath = "content/E-A7_Styles.md"; 
const styleFile = app.vault.getAbstractFileByPath(dbPath);
let styleName = "未定義";

if (styleFile) {
  const fileCache = app.metadataCache.getFileCache(styleFile);
  if (fileCache?.frontmatter?.E_A7_Styles) {
    const data = fileCache.frontmatter.E_A7_Styles;
    const styleNum = tp.frontmatter.style_number;
    if (data && data[styleNum]) {
      styleName = data[styleNum].name;
      // 將名稱寫回屬性，方便未來查詢
      await tp.file.updateFrontmatter({ style_name: styleName });
    }
  }
}
// --- 結束 ---
%>

# <%= tp.file.title %>

> [!info] 歌曲資訊
> - **🎹 原唱：** [[歌手/原唱/<% tp.frontmatter.original_artist %>]]  (原 Key: <% tp.frontmatter.original_key %>)
> - **🎤 演唱：** [[歌手/演唱/<% tp.frontmatter.cover_artist %>]]  (#<% tp.frontmatter.cover_key %>)
> - **🥁 節奏設定：** <%= styleName %> - <%= tp.frontmatter.style_number %>  (BPM: <% tp.frontmatter.bpm %>)

<div style="display: flex; gap: 2em; align-items: start;">

  <div style="flex: 1; padding-right: 1em; border-right: 1px solid #3d3d3d;">
    ### 📄 Lyric
    
  </div>

  <div style="flex: 1; padding-left: 1em;">
    ### 🎵 Note
    - in：
    - V.：
    - C.：
    - Brdg:
    - out:
  </div>

</div>
