
# 🎤 Brandon 的所有歌曲

```dataview
TABLE without id 
	file.link as "歌名",
	join(map(filter(file.etags, (t) => startswith(t, "#Singer/Brandon")), (t) => default(split(t, "/")[2], "-")), ", ") as "Key",
	join(map(filter(file.etags, (t) => startswith(t, "#Style/")), (t) => split(t, "/")[1]), "<br>") as "節奏",
	join(map(filter(file.etags, (t) => startswith(t, "#Style/")), (t) => split(t, "/")[2]), "<br>") as "BPM"
FROM #Singer/Brandon
WHERE contains(file.folder, "Ryan KB-Song List")
SORT file.name ASC
```