# aws-wordpress-portfolio
AWS EC2, Apache, PHP, RDS, and WordPressを使った構成。CloudShellから構築しました。
# AWS WordPress Portfolio

## 📌 構成図
インターネット
│
[インターネットゲートウェイ]
│
[VPC]
├── [パブリックサブネット]──[EC2（Apache+PHP）]
└── [プライベートサブネット]──[RDS（MySQL）]


---

## 🛠 セットアップ手順

### 1. EC2インスタンス作成（Amazon Linux 2 または 2023）
### 2. Apache + PHP インストール
（OSにより yum または dnf を使用）
### 3. RDSインスタンス作成（MySQL）
### 4. セキュリティグループ設定（ポート80, 3306）
### 5. WordPressダウンロードと展開
### 6. wp-config.php 編集（RDS情報記入）
### 7. Apache再起動して動作確認

---
SSHでec2にログインしたら


 コマンド例（Amazon Linux 2）(yum仕様)
# 1. パッケージ更新
sudo yum update -y

# 2. Apache & PHPインストール（ExtrasからPHPを有効化）
sudo amazon-linux-extras enable php7.4
sudo yum clean metadata
sudo yum install -y php php-mysqlnd http

Amazon Linux 2023 の場合（dnf使用）

# 2. Apache & PHPインストール（Extrasは使わない）
sudo dnf install -y httpd php php-mysqlnd php-fpm php-cli php-json php-common

# 2.. Apache起動
sudo systemctl start httpd
sudo systemctl enable httpd

# 3. WordPressダウンロード
cd /var/www/html
sudo curl -O https://wordpress.org/latest.tar.gz
sudo tar -xzf latest.tar.gz
sudo cp -r wordpress/* .
# 4. 権限設定
sudo chown -R apache:apache /var/www/html

# 5. wp-config編集（RDS情報を記入）
sudo cp wp-config-sample.php wp-config.php
sudo vi wp-config.php

wp-config.php の記述例（RDS接続部分）

/** WordPressのためのデータベース設定 */
define( 'DB_NAME', 'your_db_name' );
define( 'DB_USER', 'your_db_user' );
define( 'DB_PASSWORD', 'your_password' );
define( 'DB_HOST', 'your-rds-endpoint.ap-northeast-1.rds.amazonaws.com' );

# 6. Apache再起動
sudo systemctl restart httpd

詰まったポイント①：RDSに接続できなかった

RDSのセキュリティグループのインバウンド設定に、EC2のセキュリティグループを許可していなかった
解決方法：

RDSのセキュリティグループ設定を開き、インバウンドルールに以下を追加：
	•	タイプ：MySQL/Aurora
	•	ポート範囲：3306
	•	ソース：EC2のセキュリティグループ（sg-xxxxxxxx）を指定

 
 詰まったポイント②：PHPのインストールで amazon-linux-extras が使えなかった

原因：Amazon Linux 2023 は amazon-linux-extras が廃止されており、代わりに dnf を使用する必要がある

解決方法（Amazon Linux 2023の場合）：sudo dnf install -y httpd php php-mysqlnd php-cli php-common php-fpm
