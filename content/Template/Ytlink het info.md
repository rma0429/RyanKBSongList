<%*
// ==========================================
//  YouTube Meta Fetcher (原生直連版)
//  策略：不依賴任何 API，直接請求網頁原始碼並挖取資料
// ==========================================

// 1. 取得連結
var file = tp.file.find_tfile(tp.file.path(true));
var rawUrl = tp.frontmatter.yt_link;

if (!rawUrl || rawUrl === "") {
    new Notice("❌ 錯誤：找不到 yt_link");
} else {
    try {
        new Notice("⏳ 正在連線至 YouTube...");

        // --- 步驟 A: 提取 ID (最穩定的第一步) ---
        const idRegex = /(?:v=|youtu\.be\/|\/embed\/)([a-zA-Z0-9_-]{11})/;
        const match = rawUrl.match(idRegex);
        if (!match) throw new Error("無法識別影片 ID");
        const videoId = match[1];

        // --- 步驟 B: 建構標準網址並請求 (使用 Obsidian 內部請求) ---
        // 我們不請求 music.youtube，改請求 www.youtube
        const targetUrl = `https://www.youtube.com/watch?v=${videoId}`;
        
        // 使用 requestUrl (Obsidian API) 而非 fetch，這能避開某些 CORS 限制
        const response = await requestUrl({ url: targetUrl });
        const html = response.text;

        // --- 步驟 C: 暴力解析 HTML (Regex) ---
        // YouTube 的標題通常藏在 <meta name="title"> 或 <title> 標籤中
        
        // 1. 嘗試抓取標題
        let songName = "";
        const titleMatch = html.match(/<meta name="title" content="(.*?)">/);
        if (titleMatch && titleMatch[1]) {
            songName = titleMatch[1];
        } else {
            // 備用方案：抓 <title> 標籤
            const titleTagMatch = html.match(/<title>(.*?) - YouTube<\/title>/);
            if (titleTagMatch && titleTagMatch[1]) {
                songName = titleTagMatch[1];
            } else {
                // 如果都抓不到，嘗試抓取 JSON 資料結構 (最底層的資料)
                const jsonTitle = html.match(/"title":\{"runs":\[\{"text":"(.*?)"\}\]/);
                if (jsonTitle) songName = jsonTitle[1];
            }
        }

        if (!songName) throw new Error("無法從網頁中解析出標題");

        // 2. 嘗試抓取歌手 (頻道名稱)
        let artistName = "Unknown";
        const authorMatch = html.match(/<link itemprop="name" content="(.*?)">/);
        if (authorMatch && authorMatch[1]) {
            artistName = authorMatch[1];
        } else {
            // 備用方案：抓取 JSON 中的 owner
            const jsonAuthor = html.match(/"owner":\{.*?"title":\{"runs":\[\{"text":"(.*?)"\}\]/);
            if (jsonAuthor) artistName = jsonAuthor[1];
        }

        // --- 步驟 D: 資料清理 ---
        // 1. 移除歌手名稱中的雜訊
        artistName = artistName.replace(" - Topic", "").trim();

        // 2. 移除歌名中的雜訊
        songName = songName.replace(/\(Official.*?\)/gi, "")
                           .replace(/\[Official.*?\]/gi, "")
                           .replace(/\(Music Video\)/gi, "")
                           .replace(/\(Lyrics\)/gi, "")
                           .replace(/\(Audio\)/gi, "")
                           .replace(/&quot;/g, '"') // 修正 HTML 引號
                           .replace(/&#39;/g, "'")  // 修正 HTML 撇號
                           .trim();

        // --- 步驟 E: 組合結果 ---
        let finalTitle = "";
        if (songName.toLowerCase().includes(artistName.toLowerCase())) {
             finalTitle = songName;
        } else {
             finalTitle = `${songName} - ${artistName}`;
        }

        // --- 步驟 F: 寫入 ---
        await app.fileManager.processFrontMatter(file, (frontmatter) => {
            frontmatter["title"] = finalTitle;
            if (!frontmatter["org_artist"] || frontmatter["org_artist"] === "Unknown") {
                frontmatter["org_artist"] = artistName;
            }
        });

        new Notice(`✅ 抓取成功：${finalTitle}`);

    } catch (error) {
        console.error("Direct Parse Error:", error);
        new Notice("⚠️ 自動抓取失敗，請手動輸入");
        
        // 只有真的完全失敗時，才跳出這個框
        const manualTitle = await tp.system.prompt("請輸入 [歌名 - 歌手]");
        if (manualTitle) {
            await app.fileManager.processFrontMatter(file, (frontmatter) => {
                frontmatter["title"] = manualTitle;
            });
        }
    }
}
_%>
