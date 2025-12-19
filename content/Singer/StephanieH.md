<!-- QueryToSerialize: LIST FROM #foo  -->
# 🎤 StephanieH 的所有歌曲

```dataview
TABLE without id 
	file.link as "歌名",
	join(map(filter(file.etags, (t) => startswith(t, "#Singer/StephanieH")), (t) => default(split(t, "/")[2], "-")), ", ") as "Key",
	join(map(filter(file.etags, (t) => startswith(t, "#Style/")), (t) => split(t, "/")[1]), "<br>") as "節奏",
	join(map(filter(file.etags, (t) => startswith(t, "#Style/")), (t) => split(t, "/")[2]), "<br>") as "BPM"
FROM #Singer/StephanieH
WHERE contains(file.folder, "Ryan KB-Song List")
SORT file.name ASC
```