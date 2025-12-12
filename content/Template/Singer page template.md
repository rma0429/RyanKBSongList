<%*
// 1. 取得目前筆記的標題 (也就是歌手名字，例如 "Ryan")
var singerName = tp.file.title;

// 2. 生成標題
tR += "# 🎤 " + singerName + " 的所有歌曲\n\n";

// 3. 生成 Dataview 查詢語法
// 注意：這裡我們用字串拼接的方式，避開語法解析錯誤
tR += "```dataview\n";
tR += "TABLE without id \n";
tR += "	file.link as \"歌名\",\n";
tR += "	style_name as \"節奏\",\n";
tR += "	bpm as \"BPM\"\n";

// 關鍵修改：使用 FROM #Singer/名字
// 這樣才能自動抓到所有子標籤 (例如 #Singer/Ryan/Ab, #Singer/Ryan/C)
tR += "FROM #Singer/" + singerName + "\n";

tR += "SORT file.name ASC\n";
tR += "```";
%>
