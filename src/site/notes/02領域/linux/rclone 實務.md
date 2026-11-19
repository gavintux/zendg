---
{"tags":["rclone","backup"],"created":"2023-12-24T22:49:01.856+08:00","status":"completed","dg-publish":true,"permalink":"/02領域/linux/rclone 實務/","dgPassFrontmatter":true,"noteIcon":"","updated":"2026-11-19T15:23:02.004+08:00","dg-note-properties":{"tags":["rclone","backup"],"created":"2025-11-18","status":"completed"}}
---


- ## 常用指令
	- ### [rclone dedupe](https://rclone.org/commands/rclone_dedupe)- 合併重複檔案
		- `--dedupe-mode newest` - removes identical files then keeps the newest one.移除相同的檔案，並保留最新的檔案。
	-
- ## 觀念
- ### copy 與sync 的差別
	- sync , 會將目的區檔案與來區的檔案致。即 sync A-->B ==> B=A
	- copy,會將來來源區檔案複製到目的區，目標區若有相同的檔案會不處理。 即copy A--> B ==> B=A+B
-