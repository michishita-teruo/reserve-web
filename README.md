# 🚀【超速設計書】施設予約＆備品管理WEBシステム（reserve-web）

> GitHubリポジトリ名：`reserve-web`  
> 公開・mainブランチ  
> 技術：Next.js + MySQL（Xserver対応）  
> ロール：管理者 / 一般ユーザー  
> 納期：3時間以内に設計書完成（※開発は別）

---

## 📋 1. システム概要

| 項目 | 内容 |
|------|------|
| 名称 | 施設予約＆備品管理WEBシステム |
| 目的 | 施設の重複予約を防ぎ、備品の貸出状況を一元管理 |
| ターゲット | 管理者（施設・備品登録・承認）、ユーザー（予約・貸出申請） |
| 動作環境 | ローカル開発 → Xserverへデプロイ（PHP8.1/MySQL8） |

---

## 🧩 2. 機能一覧（最小MVP）

### 🔹 一般ユーザー機能
- [ ] 施設一覧表示（予約可能/不可を明示）
- [ ] 施設予約（日時指定、重複チェック）
- [ ] 自分の予約一覧・キャンセル
- [ ] 備品一覧表示（在庫・貸出状況）
- [ ] 備品貸出申請（管理者承認待ち）

### 🔹 管理者機能
- [ ] 施設登録・編集・削除
- [ ] 備品登録・在庫数管理
- [ ] 予約一覧閲覧（全ユーザー）
- [ ] 貸出申請の承認/却下
- [ ] 利用者管理（簡易：ユーザーIDのみ）

---

## 🗃 3. データベース設計（MySQL）

### テーブル構成（最低限）

```sql
-- ユーザー
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(50) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  role ENUM('user', 'admin') DEFAULT 'user',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 施設
CREATE TABLE facilities (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  capacity INT,
  is_active BOOLEAN DEFAULT true
);

-- 予約
CREATE TABLE reservations (
  id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  facility_id INT NOT NULL,
  start_datetime DATETIME NOT NULL,
  end_datetime DATETIME NOT NULL,
  status ENUM('pending', 'confirmed', 'cancelled') DEFAULT 'confirmed',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (facility_id) REFERENCES facilities(id),
  UNIQUE KEY no_overlap (facility_id, start_datetime, end_datetime) -- 重複防止
);

-- 備品
CREATE TABLE items (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  stock INT DEFAULT 0,
  description TEXT
);

-- 貸出申請
CREATE TABLE borrow_requests (
  id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  item_id INT NOT NULL,
  quantity INT DEFAULT 1,
  status ENUM('pending', 'approved', 'rejected') DEFAULT 'pending',
  requested_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (item_id) REFERENCES items(id)
);
```

> ※ 実際の開発では、重複予約チェックはアプリケーション層でも実装必須

---

## 🛠 4. 技術スタック & デプロイ方針

| 項目 | 内容 |
|------|------|
| フロントエンド | Next.js (App Router) + TypeScript + Tailwind CSS |
| バックエンド | Next.js API Routes（Node.js）→ でもXserverはNode.js非対応！⚠️ |
| → 修正案 | **Next.jsは静的サイト生成（SSG/ISR）のみ利用**。APIはPHPで別実装（後述） |
| データベース | MySQL 8（Xserver） |
| デプロイ | `next build` → `out/` をXserverのpublic_htmlへFTPアップロード |
| 認証 | 簡易：セッション or JWT（PHP側で実装） |

> ❗ **重要：XserverはNode.jsランタイムを提供していません**  
> → Next.jsのAPI RoutesやServer Actionsは使えない  
> → API部分は**PHPで別途実装**し、Next.jsフロントがフェッチする形に変更必須

---

## 🔄 5. システムアーキテクチャ（簡易）

```
[ユーザー] 
   ↓
[Next.js フロント（静的ビルド）] → ビルド後HTML/CSS/JSをXserverへ配置
   ↓ fetch
[PHP API（Xserver上）] ← MySQLと通信
   ↓
[MySQL データベース]
```

---

## 📁 6. GitHubリポジトリ作成手順（今すぐ実行！）

```bash
# ローカルで実行
mkdir reserve-web
cd reserve-web
git init
echo "# 施設予約＆備品管理WEBシステム" > README.md
echo "node_modules/" > .gitignore
echo ".env" >> .gitignore

# GitHubで reserve-web リポジトリを新規作成（Public / mainブランチ）
# リモート追加（YOUR_GITHUB_USERNAMEを置き換えて）
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/reserve-web.git
git add .
git commit -m "初期コミット: 設計書ベース構成"
git branch -M main
git push -u origin main
```

> ✅ GitHubリポジトリ：https://github.com/YOUR_GITHUB_USERNAME/reserve-web  
> （↑ 作成後、README.mdにこの設計書を貼り付けてください）

---

## ⏱ 7. 3時間でやるべきこと（設計書フェーズ）

| 時間 | タスク |
|------|--------|
| 0:00-0:15 | GitHubリポジトリ作成 + .gitignore/README初期化 |
| 0:15-0:45 | DB設計SQLを `schema.sql` としてリポジトリに保存 |
| 0:45-1:30 | Next.jsアプリ雛形作成（npx create-next-app）＋静的ビルド確認 |
| 1:30-2:30 | フロント画面のワイヤーフレーム（Figma or 手書きスキャン）を `docs/` へ保存 |
| 2:30-3:00 | この設計書をREADME.mdに整形してコミット＆プッシュ |

---

## ✅ 最終成果物（3時間後）

- GitHubリポジトリ `reserve-web` 完成（公開）
- `README.md` に設計書（この文書）が記載済み
- `schema.sql` でDB設計確定
- `docs/` に画面イメージ（ワイヤーフレーム）
- Next.js雛形プロジェクト（`/app` ディレクトリ構成まで）

---

## 🚨 注意点（Xserver制約）

- Next.jsのAPI Routes（`/app/api/`）は**使えない**
- Server Componentsの動的処理も**使えない**（静的生成のみ）
- 認証・予約処理などのAPIは**別途PHPで実装** → `/api/` 以下にPHPファイルを配置
- フロントはあくまで「静的サイト」→ データはすべて `fetch()` でPHP APIから取得

---

## ▶️ 次ステップ（設計書提出後）

1. Next.js静的サイトの画面実装（トップ、施設一覧、予約フォームなど）
2. PHP API開発（Xserver上動作確認）
3. MySQL接続テスト
4. ローカルで統合テスト → ビルド → Xserverアップロード

---