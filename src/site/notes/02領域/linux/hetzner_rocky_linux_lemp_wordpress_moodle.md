---
{"tags":["devops","git","github-actions","nginx","server"],"created":"2026-11-19T15:54:37.404+08:00","status":"completed","dg-publish":true,"permalink":"/02領域/linux/hetzner_rocky_linux_lemp_wordpress_moodle/","dgPassFrontmatter":true,"noteIcon":"","updated":"2026-11-19T15:59:43.202+08:00","dg-note-properties":{"tags":["devops","git","github-actions","nginx","server"],"created":"2026-11-19","status":"completed"}}
---

# Hetzner Rocky Linux：LEMP + WordPress (Multisite) + Moodle 完整部署手冊

本手冊適用於在 **Hetzner Cloud** 虛擬主機或獨立伺服器上，使用 **Rocky Linux** 架設高相容性、高安全性的 LEMP 堆疊，並同時運行 **WordPress 多站點模式 (Multisite Network)** 與 **Moodle 4.x**。

---

## 目錄
1. [架構規劃與版本相容性需求](#1-架構規劃與版本相容性需求)
2. [Hetzner 伺服器初始化與基礎防護](#2-hetzner-伺服器初始化與基礎防護)
3. [安裝與配置 MariaDB 11.4 LTS](#3-安裝與配置-mariadb-114-lts)
4. [安裝與配置 PHP 8.2 / 8.3 與必要擴充模組](#4-安裝與配置-php-82--83-與必要擴充模組)
5. [安裝與配置 Nginx 核心服務](#5-安裝與配置-nginx-核心服務)
6. [部署 WordPress 多站點模式 (Multisite)](#6-部署-wordpress-多站點模式-multisite)
7. [部署 Moodle 4.x (隔離埠口與安全性路徑)](#7-部署-moodle-4x-隔離埠口與安全性路徑)
8. [SELinux 與權限深度調教重點](#8-selinux-與權限深度調教重點)
9. [日常維護與 Cron 定時任務](#9-日常維護與-cron-定時任務)

---

## 1. 架構規劃與版本相容性需求

| 元件 | 推薦版本 | 說明 |
| :--- | :--- | :--- |
| **作業系統** | Rocky Linux 9 / 10 | 企業級 RHEL 衍生發行版，預設啟用強制 SELinux |
| **Web 伺服器** | Nginx (Mainline 或 Stable) | 高併發處理靜態資源與反向代理 |
| **資料庫** | MariaDB 11.4 LTS | Moodle 4.x 嚴格要求 MariaDB $\ge$ 10.6.7（且支援 11.4） |
| **PHP 運行環境** | PHP 8.2 或 8.3 (Remi Repo) | 同時兼顧 WordPress 最新版與 Moodle 4.4+ 的相容支援 |
| **WordPress** | 根目錄 `/` 或特定子目錄 | 啟用 Sub-directory / Sub-domain 多站點網路 |
| **Moodle** | 獨立埠口 (Port 8080) 或獨立子網域 | 根目錄必須指向 `public/` 目錄，避免全域洩漏安全性檢查報錯 |

---

## 2. Hetzner 伺服器初始化與基礎防護

### 2.1 系統更新與基礎工具安裝
以 SSH 登入 Hetzner 實例後，更新所有既有套件：

```bash
sudo dnf update -y
sudo dnf install -y epel-release vim curl wget tar bzip2 unzip git firewalld policycoreutils-python-utils
```

### 2.2 防火牆設定 (Firewalld)
啟用 Rocky Linux 內建防火牆，並放行 HTTP、HTTPS 及後續 Moodle 預計使用的連接埠：

```bash
sudo systemctl enable --now firewalld
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

> **注意：安全防護備註**  
> Hetzner 控制台內建「Cloud Firewall」，建議搭配伺服器本機的 `firewalld` 雙層過濾。若 Hetzner 防火牆未放行 8080 埠口，外部連線將無法觸達本機。

---

## 3. 安裝與配置 MariaDB 11.4 LTS

Moodle 4.x 對資料庫版本與字元集有嚴格規範。Rocky Linux 內建 AppStream 的 MariaDB 版本往往過舊，因此我們直接引入 MariaDB 官方 Repository。

### 3.1 引入 MariaDB 11.4 官方儲存庫
使用 `vim` 建立官方套件來源：

```bash
sudo vim /etc/yum.repos.d/mariadb.repo
```

在 `vim` 中按 `i` 進入編輯模式，貼上以下內容：

```ini
[mariadb]
name = MariaDB
baseurl = https://rpm.mariadb.org/11.4/rhel/$releasever/$basearch
module_hotfixes = 1
gpgkey = https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck = 1
```

按 `Esc`，輸入 `:wq` 儲存並離開。

### 3.2 安裝並啟動 MariaDB
```bash
sudo dnf clean all
sudo dnf install -y MariaDB-server MariaDB-client
sudo systemctl enable --now mariadb
```

### 3.3 初始化安全設定
執行安全初始化腳本，設定 Root 密碼並移除測試庫：

```bash
sudo mariadb-secure-installation
```
*(依序依照指示設定強密碼，並移除 anonymous users、禁止 remote root 登入、刪除 test 資料庫。)*

### 3.4 配置 Moodle 必備資料庫參數
Moodle 要求資料庫必須支援 Barracuda 格式與全字元 UTF-8（`utf8mb4`）。

```bash
sudo vim /etc/my.cnf.d/server.cnf
```

在 `[mysqld]` 區段下方加入以下參數：

```ini
[mysqld]
innodb_file_per_table = 1
innodb_large_prefix = 1
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
innodb_buffer_pool_size = 512M
max_allowed_packet = 64M
```

按 `Esc`，輸入 `:wq` 儲存離開，並重新啟動資料庫：

```bash
sudo systemctl restart mariadb
```

### 3.5 建立 WordPress 與 Moodle 資料庫
登入 MariaDB 控制台：

```bash
sudo mariadb -u root -p
```

在 SQL 提示字元下執行（請自行替換 `YourSecurePasswordHere`）：

```sql
-- 建立 WordPress 資料庫與使用者
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_520_ci;
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'Wp_Password_123!';
GRANT ALL ON wordpress.* TO 'wp_user'@'localhost';

-- 建立 Moodle 資料庫與使用者
CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'moodle_user'@'localhost' IDENTIFIED BY 'Moodle_Password_123!';
GRANT ALL ON moodle.* TO 'moodle_user'@'localhost';

FLUSH PRIVILEGES;
EXIT;
```

---

## 4. 安裝與配置 PHP 8.2 / 8.3 與必要擴充模組

WordPress 與 Moodle 皆需要特定的 PHP 擴充模組（特別是 Moodle 必須的 `sodium`, `intl`, `max_input_vars` 等）。

### 4.1 啟用 Remi PHP 儲存庫
```bash
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
sudo dnf module reset php -y
sudo dnf module enable php:remi-8.2 -y
```
*(若需使用 PHP 8.3，可將指令改為 `sudo dnf module enable php:remi-8.3 -y`)*

### 4.2 安裝 PHP-FPM 及所有必要模組
```bash
sudo dnf install -y php-fpm php-cli php-mysqlnd php-gd php-xml php-mbstring \
php-zip php-soap php-intl php-opcache php-curl php-sodium php-bcmath php-imagick
```

### 4.3 調整 php.ini 全域設定
針對 Moodle 表單與大型檔案上傳調優：

```bash
sudo vim /etc/php.ini
```

搜尋並修改下列參數（在 `vim` 普通模式下可按 `/` 搜尋關鍵字）：

```ini
memory_limit = 512M
upload_max_filesize = 100M
post_max_size = 100M
max_execution_time = 300
max_input_vars = 5000
date.timezone = Asia/Taipei
```

### 4.4 設定 PHP-FPM 連線池（Pool）權限
在 Rocky Linux 中，Nginx 與 PHP-FPM 預設的使用者分工不同。為了與 Nginx 協同運作，需將運行權限指定給 `nginx` 使用者。

```bash
sudo vim /etc/php-fpm.d/www.conf
```

將以下設定修改或確認：

```ini
user = nginx
group = nginx
listen = /run/php-fpm/www.sock
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
```

啟動並啟用 PHP-FPM：

```bash
sudo systemctl enable --now php-fpm
```

---

## 5. 安裝與配置 Nginx 核心服務

```bash
sudo dnf install -y nginx
sudo systemctl enable --now nginx
```

---

## 6. 部署 WordPress 多站點模式 (Multisite)

### 6.1 下載並配置 WordPress 檔案
```bash
sudo mkdir -p /var/www/html/wordpress
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -zxvf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/wordpress/
```

### 6.2 建立與設定 wp-config.php
```bash
cd /var/www/html/wordpress
sudo cp wp-config-sample.php wp-config.php
sudo vim wp-config.php
```

填寫資料庫連線資訊：

```php
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wp_user' );
define( 'DB_PASSWORD', 'Wp_Password_123!' );
define( 'DB_HOST', 'localhost' );
define( 'DB_CHARSET', 'utf8mb4' );
define( 'DB_COLLATE', '' );

// 啟用多站點網路 (WP Multisite) 核心支援開關
define( 'WP_ALLOW_MULTISITE', true );
```

### 6.3 建立 WordPress Nginx 設定檔 (Port 80)
```bash
sudo vim /etc/nginx/conf.d/wordpress.conf
```

寫入針對 **WordPress 多站點（子目錄 Sub-directory 模式）相容的重寫規則**：

```nginx
server {
    listen 80;
    server_name _; # 或填寫您的 WordPress 網域名稱

    root /var/www/html/wordpress;
    index index.php index.html index.htm;

    client_max_body_size 100M;

    # 全域阻擋隱藏檔案存取
    location ~ /\. {
        deny all;
    }

    # WordPress 多站點（子目錄）專用重寫規則
    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    # 針對多站點 uploads 虛擬路徑的靜態資源重定向
    location ~ ^(/[^/]+)?/files/(.+) {
        try_files /wp-content/blogs.dir/$blogid/files/$2 /wp-includes/ms-files.php?file=$2;
        access_log off;
        log_not_found off;
        expires max;
    }

    # 子目錄多站點靜態資源轉發
    if (!-e $request_filename) {
        rewrite ^/[_0-9a-zA-Z-]+(/wp-(content|admin|includes).*) $1 last;
        rewrite ^/[_0-9a-zA-Z-]+(/.*\.php)$ $1 last;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_read_timeout 300;
    }
}
```

---

## 7. 部署 Moodle 4.x (隔離埠口與安全性路徑)

> **重要安全架構說明：Moodle 4.x 根目錄限制**  
> Moodle 4.x 要求網頁伺服器的 Document Root **不得** 直接指向程式庫根目錄，必須指向其子目錄 `/public`。此外，為了避免與 WordPress 產生路徑衝突，建議將 Moodle 部署在獨立埠口（如 `8080`）或獨立子網域。

### 7.1 下載 Moodle
```bash
sudo mkdir -p /var/www/html/moodle
sudo mkdir -p /var/www/moodledata
cd /tmp
wget https://download.moodle.org/download.php/direct/stable404/moodle-latest-404.tgz -O moodle.tgz
tar -zxvf moodle.tgz
sudo cp -r moodle/* /var/www/html/moodle/
```

### 7.2 建立 Moodle Nginx 設定檔 (Port 8080)
```bash
sudo vim /etc/nginx/conf.d/moodle.conf
```

寫入完整設定內容：

```nginx
server {
    listen 8080;
    server_name _; # 或填寫您的 Moodle 網域名稱

    # 關鍵：指向 public 目錄符合安全標準
    root /var/www/html/moodle/public;
    index index.php index.html index.htm;

    client_max_body_size 100M;

    # 路由導向
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # FastCGI 處理 Moodle 複雜的路由請求
    location ~ [^/]\.php(/|$) {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        if (!-f $document_root$fastcgi_script_name) {
            return 404;
        }
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_read_timeout 300;
    }

    location ~ /\. {
        deny all;
    }
}
```

---

## 8. SELinux 與權限深度調教重點

在 Rocky Linux / RHEL 架構下，最常見的 `500 Internal Server Error` 或 `Permission Denied` 幾乎都是由 **檔案權限** 與 **SELinux 標籤** 引起的。

### 8.1 檔案與目錄所有權配置
```bash
# WordPress 權限
sudo chown -R nginx:nginx /var/www/html/wordpress
sudo find /var/www/html/wordpress -type d -exec chmod 755 {} \;
sudo find /var/www/html/wordpress -type f -exec chmod 644 {} \;

# Moodle 程式碼與數據目錄權限
sudo chown -R nginx:nginx /var/www/html/moodle
sudo chown -R nginx:nginx /var/www/moodledata
sudo chmod -R 775 /var/www/moodledata
```

### 8.2 SELinux 安全標籤與規則配置 (不可跳過)
Rocky Linux 預設開啟強制模式（Enforcing）。若未放行網路連線與設定正確上下文，網頁將無法連線資料庫或對目錄進行寫入。

```bash
# 允許 Nginx 與 PHP 發送網路請求 (用於外掛更新、下載語言包等)
sudo setsebool -P httpd_can_network_connect 1
sudo setsebool -P httpd_can_network_connect_db 1

# 設定 WordPress 內容與上傳目錄的 SELinux 標籤
sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html/wordpress/wp-content(/.*)?"
sudo restorecon -Rv /var/www/html/wordpress/wp-content

# 設定 Moodle 專用資料目錄 moodledata 的寫入標籤
sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/moodledata(/.*)?"
sudo restorecon -Rv /var/www/moodledata

# 設定 Moodle 本體設定檔允許動態建立
sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html/moodle/config.php"
sudo restorecon -v /var/www/html/moodle/config.php 2>/dev/null || true
```

### 8.3 測試 Nginx 配置並重啟
```bash
sudo nginx -t
sudo systemctl restart nginx
sudo systemctl restart php-fpm
```

---

## 9. 日常維護與 Cron 定時任務

WordPress 與 Moodle 均高度依賴定時排程執行排隊信件、快取清除與備份任務。

```bash
sudo vim /etc/crontab
```

在檔案末尾加入以下排程：

```ini
# 每小時執行一次 WordPress 排程工作 (避免依賴訪客點擊觸發)
*/15 * * * * nginx /usr/bin/php /var/www/html/wordpress/wp-cron.php >/dev/null 2>&1

# 每分鐘執行一次 Moodle 後台排程維護
* * * * * nginx /usr/bin/php /var/www/html/moodle/admin/cli/cron.php >/dev/null 2>&1
```

儲存離開後，重啟 Crond 服務：

```bash
sudo systemctl restart crond
```

---

## 結語與驗證

1. **WordPress 驗證**：在瀏覽器中造訪 `http://<您的伺服器IP>/` 完成基本站點安裝，接著進入後台「工具」>「網路設定」正式啟動多站點網路。
2. **Moodle 驗證**：在瀏覽器中造訪 `http://<您的伺服器IP>:8080/`，系統將引導進入環境檢測畫面，由於環境擴充與資料庫均已調優，檢測將全數亮起綠燈。