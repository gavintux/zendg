---
{"tags":["rclone","backup"],"created":"2022-06-12T08:52:06.810+08:00","status":"completed","dg-publish":true,"permalink":"/02領域/linux/Rclone 安裝及基本指令/","dgPassFrontmatter":true,"noteIcon":"","updated":"2026-11-19T15:23:41.572+08:00","dg-note-properties":{"tags":["rclone","backup"],"created":"2025-11-18","status":"completed"}}
---

## rclone 基本指令

1. 檔案上傳/下載  
	`rclone copy -P --drive-chunk-size=64M --transfers=40 --checkers=40 --stats=10s -v 來源:/ 目的地:/`  
	→ rclone copy（傳新檔案、忽略舊檔。速度比 sync 快）  
	→ drive-chunk-size（預設是 8M太慢）  
	→ transfers（預設是 4 ）  
	→ stats(固定時間顯示傳送資訊)  
	→ v（顯示傳送時間）  
	→ P（顯示即時傳送速度）
	
	伺服器之間轉傳（不會下載到電腦）多加這個  
	`--drive-server-side-across-configs`

2. 資料合併
 #### dedupe
 
> rclone dedupe drive:dupe --dedupe-mode newest

> rclone dedupe 雲端名稱:路徑
	
	`--dedupe-mode` 是用來設定遇到相同名稱的檔案要觸發的行為，支援傳入以下值：
		-   `interactive`：每次都詢問（預設）
		-   `skip`：刪除一樣的檔案，且後續的檔案全部略過
		-   `first`：保留第一個發現的檔案
		-   `newest`：保留最新的檔案
		-   `oldest`：保留最舊的檔案
		-   `largest`：保留檔案大小最大的檔案
		-   `smallest`：保留檔案大小最小的檔案
		-   `rename`：依照順序於檔案名稱後綴加入編號
		-   `list`：僅列出清單，不做任何事（使用上面的值如果有發現一樣名稱的檔案，也都會列出清單）

### 使用gui
> rclone rcd --rc-web-gui --rc-user=gavin --rc-pass=cljt0123







## 安裝
### macOS
	1. 安裝 Rclone [[2\|2]](https://blog.brownsugar.tw/transfer-g-suite-files/#s3-r2)，這邊使用 macOS 示範，打開終端機執行指令。

```bash
> curl https://rclone.org/install.sh | sudo bash
```

### Windows
- 下載[最新版](https://rclone.org/downloads/)，順便下載[winfsp](https://github.com/billziss-gh/winfsp/releases)，方便 win10 掛載
-   解壓縮 rclone，找個好地方放，安裝 winfsp（這個只要下一步安裝法）
## 設定
- 設定雲端資訊Google Drive

	```bash
> 	rclone config
	
	# 建立新的雲端設定檔
	n/r/c/s/q> n
	
	# 自訂這個雲端的名稱，之後指令會用到
	name> drive
	
	# 選擇雲端服務
	Storage> drive
	
	# 設定 Google API 用戶端資訊，可以留空用預設，也可以按照官網說明步驟自行產生 [4]
	client_id> 
	client_secret> 
	
	# 設定要請求的存取權限，稍後登入會用到
	scope> drive
	```

3. 接下來幾個設定都持續 Enter 使用預設值，直到瀏覽器打開 Google 應用程式授權頁面，選擇要合併資料夾的帳號、勾選存取範圍的選項並點擊「繼續」後即可授權。
### 參考文件
- [win10 Rclone 不思考ㄉ安裝與掛載 - HackMD](https://hackmd.io/@adeliae/H1ZRCUfaU)
- [拯救雲端硬碟資料，將 G Suite 教育版帳號中的檔案轉移至其他 Google 帳號 | LAY ‧ Blog](https://blog.brownsugar.tw/transfer-g-suite-files/#s3-r5)
- [將您的學校帳戶內容複製到另一個帳戶](https://support.google.com/accounts/answer/6386856)
- [將檔案上傳至 Google 雲端硬碟](https://support.google.com/a/answer/172541)
- [修正網頁搜尋範圍](https://support.google.com/websearch/answer/2466433)