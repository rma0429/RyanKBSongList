<%*
// 1. 取得歌手名字
var singerName = tp.file.title;
const dv = app.plugins.plugins.dataview.api;

// 2. 獲取符合條件的所有頁面以計算總數
const pages = dv.pages(`#Singer/${singerName}`)
    .filter(p => p.file.folder.includes("Ryan KB-Song List"));
const songCount = pages.length;

// 3. 生成標題 (包含計數)
tR += "# 🎤 " + singerName + " 的所有歌曲 ( " + songCount + " )\n\n";

// 4. 定義 Dataview 查詢語法
const query = `TABLE without id 
	file.link as "歌名",
	join(map(filter(file.etags, (t) => startswith(t, "#Singer/${singerName}")), (t) => default(split(t, "/")[2], "-")), ", ") as "Key",
	join(map(filter(file.etags, (t) => startswith(t, "#Style/")), (t) => split(t, "/")[1]), "<br>") as "節奏",
	join(map(filter(file.etags, (t) => startswith(t, "#Style/")), (t) => split(t, "/")[2]), "<br>") as "BPM"
FROM #Singer/${singerName}
WHERE contains(file.folder, "Ryan KB-Song List")
SORT file.name ASC`;

// 5. 直接抓取 Dataview 的 Markdown 結果並印出
const result = await dv.queryMarkdown(query);

if (result.successful) {
    tR += result.value;
} else {
    tR += "❌ 無法找到歌曲資料，請檢查標籤是否正確。";
}
%>