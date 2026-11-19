---
{"tags":["ssh","linux","macos","hetzner","rocky-linux","security"],"status":"completed","created":"2026-11-18T19:36:14.378+08:00","dg-publish":true,"permalink":"/02領域/linux/multi_device_ssh_setup_manual/","dgPassFrontmatter":true,"noteIcon":"","updated":"2026-11-19T15:18:41.672+08:00","dg-note-properties":{"tags":["ssh","linux","macos","hetzner","rocky-linux","security"],"status":"completed","created":"2026-11-18"}}
---


# 多設備（Mac mini / MacBook Pro / MX Linux）連線至 Hetzner Rocky Linux 之 SSH 金鑰建置手冊

本手冊指導如何採用**「一機一鑰（One Key Per Device）」**的最佳資安實踐，讓手邊的三台主力設備（Mac mini、MacBook Pro、MX Linux）各自擁有獨立的私鑰，並將三把公鑰統一註冊到 Hetzner 的 Rocky Linux 伺服器上，實現安全、免密碼的快速登入。

---

## 🗺️ 架構概念圖

每一台電腦各自產生一對金鑰，**私鑰留在自己電腦裡**，**公鑰（`.pub`）全部貼到伺服器的 `authorized_keys` 清單中**：

```
[ Mac mini ]    持有私鑰 A ──┐
                            ├──> 驗證通過 ──> [ Hetzner Rocky Linux ]
[ MacBook Pro ] 持有私鑰 B ──┤               (/root/.ssh/authorized_keys)
                            │                ├── 紀錄：公鑰 A.pub (mac-mini)
[ MX Linux ]    持有私鑰 C ──┘                ├── 紀錄：公鑰 B.pub (macbook-pro)
                                             └── 紀錄：公鑰 C.pub (mx-linux)
```

> [!TIP] 為什麼要給每台機器設定不同的 `-C` 註解？
> 在產生金鑰時加上 `-C "機器名稱"`，會顯示在公鑰文字的最後面。日後打開伺服器上的 `authorized_keys` 時，就能一眼看出哪一行屬於哪一台電腦，若某台電腦停用，直接刪除該行即可。

---

## 🖥️ 第一階段：在三台機器上各自產生並上傳金鑰

### 1️⃣ 設備一：Mac mini (macOS)

#### 1. 檢查或產生金鑰
打開「終端機 (Terminal)」，先檢查是否已有預設金鑰：
```bash
ls -al ~/.ssh/id_ed25519*
```
* **如果已經有 `id_ed25519` 與 `id_ed25519.pub`**（如先前設定 GitHub 時已建立）：可直接使用，**不需重新產生**，請直接跳至「步驟 2」。
* **如果沒有**：執行以下指令建立：
  ```bash
  ssh-keygen -t ed25519 -C "linkc@mac-mini"
  ```
  *（連按三次 `Enter` 使用預設路徑與無密碼即可）*

#### 2. 載入至 macOS 鑰匙圈
```bash
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

#### 3. 將公鑰上傳至 Hetzner Rocky Linux
在 Mac mini 終端機執行一鍵上傳指令（請將 IP 替換為你的 Hetzner 伺服器 IP，如 `95.217.152.250`）：
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub root@95.217.152.250
```
*（系統會要求輸入最後一次伺服器的 root 密碼，輸入成功後即完成綁定）*

---

### 2️⃣ 設備二：MacBook Pro (macOS)

#### 1. 檢查或產生金鑰
打開 MacBook Pro 的「終端機 (Terminal)」，檢查是否已有金鑰：
```bash
ls -al ~/.ssh/id_ed25519*
```
* **若無金鑰**，執行以下指令產生專屬金鑰：
  ```bash
  ssh-keygen -t ed25519 -C "linkc@macbook-pro"
  ```
  *（連按三次 `Enter` 完成建立）*

#### 2. 載入至 macOS 鑰匙圈
```bash
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

#### 3. 將公鑰上傳至 Hetzner Rocky Linux
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub root@95.217.152.250
```

---

### 3️⃣ 設備三：MX Linux (Linux)

#### 1. 檢查或產生金鑰
打開 MX Linux 的終端機，檢查是否已有金鑰：
```bash
ls -al ~/.ssh/id_ed25519*
```
* **若無金鑰**，執行以下指令產生 MX Linux 專屬金鑰：
  ```bash
  ssh-keygen -t ed25519 -C "linkc@mx-linux"
  ```
  *（連按三次 `Enter` 完成建立）*

#### 2. 將公鑰上傳至 Hetzner Rocky Linux
Linux 系統內建完整的 `ssh-copy-id` 工具，直接執行：
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub root@95.217.152.250
```

---

## 🛠️ 備用方案：如果伺服器已關閉密碼登入（手動貼上公鑰法）

如果你的 Hetzner Rocky Linux 為了安全，已經關閉了密碼登入（執行 `ssh-copy-id` 會出現 `Permission denied (publickey)`），請使用以下手動方式：

1. **在要新增的機器上**印出公鑰內容並複製：
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
2. **使用已經能連上伺服器的機器**（或透過 Hetzner Cloud 網頁後台的 Console）登入 Rocky Linux。
3. 打開授權清單檔案：
   ```bash
   nano ~/.ssh/authorized_keys
   ```
4. 捲動到最下方，**新增一行**，將剛剛複製的 `ssh-ed25519 AAAA...` 貼上，按 `Ctrl + O` 存檔、`Ctrl + X` 離開。

---

## 🛡️ 第二階段：Hetzner Rocky Linux 伺服器端權限檢查

全部上傳完成後，任選一台機器登入 Hetzner 伺服器，確認檔案結構與權限是否符合 Rocky Linux (SELinux) 的嚴格安全規範：

```bash
# 1. 登入伺服器
ssh root@95.217.152.250

# 2. 查看目前已註冊的所有公鑰（應該會看到三台機器與 GitHub Actions 的公鑰）
cat ~/.ssh/authorized_keys
```

請確保伺服器上的權限與 SELinux 標籤正確（若權限過大，SSH 會拒絕讀取公鑰）：
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
restorecon -R -v ~/.ssh
```

---

## ⚡ 第三階段：進階設定 — 建立 `~/.ssh/config` 快速連線別名

為了不用每次都背誦與輸入長長的 IP（`root@95.217.152.250`），建議在 **Mac mini、MacBook Pro 與 MX Linux** 三台機器的 `~/.ssh/config` 中加入別名設定。

在各台機器的終端機執行：
```bash
nano ~/.ssh/config
```

貼上以下內容並存檔：

```ssh-config
Host hetzner
    HostName 95.217.152.250
    User root
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
    ServerAliveCountMax 3
```
> [!NOTE] Mac 專屬額外設定
> 在 **Mac mini** 與 **MacBook Pro** 的 `~/.ssh/config` 中，可以在最下方多加兩行，讓系統重開機後依然自動從鑰匙圈載入金鑰：
> ```ssh-config
>     AddKeysToAgent yes
>     UseKeychain yes
> ```

確保 config 檔案權限正確：
```bash
chmod 600 ~/.ssh/config
```

### 🎉 最終成果測試
設定完成後，在三台電腦的終端機中，只需要輸入短短一行指令：
```bash
ssh hetzner
```
就能在一秒鐘內免密碼直接進入 Hetzner 的 Rocky Linux 伺服器！