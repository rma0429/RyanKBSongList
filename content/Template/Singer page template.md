<%*
// 1. 取得目前筆記的標題 (歌手名字)
var singerName = tp.file.title;

// 2. 生成標題
tR += "# 🎤 " + singerName + " 的所有歌曲\n\n";

// 3. 生成 Dataview 查詢語法
tR += "```dataview\n";
tR += "TABLE without id \n";
tR += "	file.link as \"歌名\",\n";

// --- 自動抓取 Key (保持逗號分隔，通常 Key 只有一個，若多個想換行也可改 <br>) ---
tR += "	join(map(filter(file.etags, (t) => startswith(t, \"#Singer/" + singerName + "\")), (t) => default(split(t, \"/\")[2], \"-\")), \", \") as \"Key\",\n";

// --- 自動抓取 節奏 (Style) [修改處：逗號改為 <br>] ---
tR += "	join(map(filter(file.etags, (t) => startswith(t, \"#Style/\")), (t) => split(t, \"/\")[1]), \"<br>\") as \"節奏\",\n";

// --- 自動抓取 BPM [修改處：逗號改為 <br>] ---
tR += "	join(map(filter(file.etags, (t) => startswith(t, \"#Style/\")), (t) => split(t, \"/\")[2]), \"<br>\") as \"BPM\"\n";

// 4. 設定來源與篩選
tR += "FROM #Singer/" + singerName + "\n";
tR += "WHERE contains(file.folder, \"Ryan KB-Song List\")\n";
tR += "SORT file.name ASC\n";
tR += "```";
%>
 