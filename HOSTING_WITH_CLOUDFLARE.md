# セキュリティ＆ホスティングガイド（自前サーバーホスト版）

## 概要：限定公開 vs SNS拡散のセキュリティ差

あなたが希望する構成：
```
🔹 GitHub: ソースコード公開（オープン）
🔹 ホスティング: Cloudflare など自前サーバーで限定公開
🔹 アクセス: URL を知っている人のみ
```

このモデルは **GitHub パブリック + 運用上の制限** という形で、セキュリティ負荷が異なります。

---

## 推奨ホスティング方法の比較

### 1️⃣ Cloudflare Pages（最も推奨）

**特徴：**
- ✅ 無料 tier あり
- ✅ HTTPS 自動
- ✅ 世界中の CDN（高速）
- ✅ GitHub 連携（自動デプロイ）
- ✅ セキュリティ設定も豊富

**セットアップ手順：**

```bash
# Step 1: GitHub にアップロード
git push origin main

# Step 2: Cloudflare アカウント作成
# https://dash.cloudflare.com/ にアクセス

# Step 3: Pages に接続
# Cloudflare Dashboard → Pages → Connect to Git
# → GitHub 選択 → clamp_calculator リポジトリ選択
# → Build settings: "No build command" (単一 HTML ファイルのため)
# → Deploy

# Step 4: カスタムドメイン設定（オプション）
# Cloudflare で管理するドメインがあれば、
# Pages → Custom domain で設定可能
```

**セキュリティ設定（Cloudflare ダッシュボード）：**

```
1. Settings → Browsing Security
   ☐ Bot Fight Mode: ON（スパムボット対策）
   ☐ DDoS Protection: ON

2. SSL/TLS → Overview
   ☐ Encryption mode: "Full" または "Full (strict)"
   ☐ Always Use HTTPS: ON

3. Security → Rate limiting
   ☐ リクエスト数の制限（オプション）
```

**URL 形式：**
```
https://clamp-calculator-xxx.pages.dev/

または

https://your-domain.com/  （カスタムドメイン利用時）
```

**コスト：**
- 無料 tier: 十分（月 25,000 PV まで）
- 有料: $20/月（より多くの機能）

---

### 2️⃣ Vercel（第二選択肢）

**特徴：**
- ✅ 無料 tier あり
- ✅ HTTPS 自動
- ✅ 高速で信頼性高い
- ✅ デプロイが簡単

**セットアップ手順（Cloudflare と同じ）：**

```bash
# Vercel にアクセス
https://vercel.com/

# GitHub アカウントでログイン
# → Import Project → clamp_calculator 選択
# → Deploy

# URL: https://clamp-calculator.vercel.app/
```

**コスト：**
- 無料 tier: 十分
- 有料: $20/月（Pro）

---

### 3️⃣ Netlify（第三選択肢）

**特徴：**
- ✅ 無料 tier あり
- ✅ HTTPS 自動
- ✅ ビルドスクリプト不要

**セットアップ：** Vercel とほぼ同じ

**URL：** `https://clamp-calculator-xxx.netlify.app/`

---

### 4️⃣ 自前サーバー + Let's Encrypt（最高度なカスタマイズ）

**特徴：**
- ✅ 完全なカスタマイズ
- ✅ IP アドレス制限可能
- ❌ 自分で保守
- ❌ セキュリティは自己責任

**推奨スタック：**

```
サーバー OS: Ubuntu 22.04
Web サーバー: nginx
SSL: Let's Encrypt（certbot）
ファイアウォール: ufw
```

**セットアップ例：**

```bash
# VPS 上（DigitalOcean, Linode など）
sudo apt update && sudo apt install nginx certbot python3-certbot-nginx

# Let's Encrypt で証明書取得
sudo certbot certonly --standalone -d your-domain.com

# nginx 設定
# /etc/nginx/sites-available/default を編集
# ssl_certificate や server 設定を追加

# HTTPS リダイレクト
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    root /var/www/clamp-calculator;
    index clamp-calculator.html;
}

# リロード
sudo systemctl reload nginx
```

**コスト：**
- VPS: $5-20/月（DigitalOcean など）
- SSL: 無料（Let's Encrypt）

---

## 限定公開（URL 制限）の実装方法

### 方法A: Cloudflare Access（推奨）

Cloudflare Pages でホストした場合、**Cloudflare Access** を使って簡単にアクセス制限できます。

**設定手順：**

```
1. Cloudflare Dashboard → Zero Trust → Access
2. "Create Access policy"
3. Application: https://clamp-calculator.example.com
4. Policies:
   - Allow: Emails match "user@example.com"
   - Block: Everyone else
5. Save
```

**実装後：**
- ユーザーがアクセス
- ↓
- Cloudflare ログイン画面が表示
- ↓
- 許可されたメール or One-Time PIN で認証
- ↓
- アプリに進める

**コスト：**
- 無料 tier: 1 つのアプリケーション
- 有料: 月 $7/ユーザー

**メリット：**
- ✅ コード側の変更不要
- ✅ パスワード管理が簡単
- ✅ ログイン履歴が記録される

---

### 方法B: Basic 認証（パスワード保護）

HTML に直接実装する場合：

```html
<!-- ❌ セキュリティが低い -->
<script>
  const password = prompt('パスワードを入力してください');
  if (password !== 'secret123') {
    document.body.innerHTML = '<h1>Access Denied</h1>';
  }
</script>

<!-- ✅ より安全 -->
```

**より安全な実装：**

nginx で HTTP Basic Auth を設定

```nginx
# /etc/nginx/sites-available/default に追加

server {
    listen 443 ssl http2;
    server_name your-domain.com;
    
    # Basic 認証の設定
    auth_basic "Restricted Area";
    auth_basic_user_file /etc/nginx/.htpasswd;
    
    # パスワードファイルを作成（htpasswd コマンド）
    # sudo htpasswd -c /etc/nginx/.htpasswd username
}
```

**パスワードファイルの作成：**

```bash
# ユーザー 1 人目を作成
sudo htpasswd -c /etc/nginx/.htpasswd tatsu
# パスワードを入力

# ユーザーを追加する場合
sudo htpasswd /etc/nginx/.htpasswd another_user
```

**メリット：**
- ✅ nginx 側で完結
- ✅ HTTP Basic Auth は標準的
- ❌ ただし HTTPS 必須（平文送信のため）

---

### 方法C: URL に Token を埋め込む（シンプル）

```
https://your-domain.com/app/?token=abc123xyz

ブラウザが自動的に token をチェック
```

**実装例：**

```javascript
// URL から token を抽出
const params = new URLSearchParams(window.location.search);
const token = params.get('token');

// トークンがない、または無効なら表示しない
const VALID_TOKENS = [
  'abc123xyz',
  'def456uvw'
];

if (!token || !VALID_TOKENS.includes(token)) {
  document.body.innerHTML = `
    <h1>⛔ Access Denied</h1>
    <p>有効なトークンが必要です</p>
  `;
  // 以下のコードは実行されない
  throw new Error('Invalid token');
}

// ここからアプリのコードが実行される
```

**メリット：**
- ✅ 実装が簡単
- ✅ パスワード管理不要
- ❌ URL がシェアされると誰でもアクセス可

**コスト：** 無料

---

## 推奨構成（最適解）

あなたの用途に最適な構成：

```
┌─────────────────────────────────────────┐
│ ホスティング: Cloudflare Pages          │
│ （HTTPS 自動、世界中 CDN 高速）         │
└─────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────┐
│ アクセス制御: Cloudflare Access         │
│ （メール認証で URL 制限）               │
└─────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────┐
│ アプリケーション: clamp-calculator.html │
│ （単一 HTML ファイル、セキュアなコード） │
└─────────────────────────────────────────┘
```

**コスト：** 無料 tier で十分（フリーユーザー向け Cloudflare Access含む）

**セットアップ時間：** 約 30 分

---

## Cloudflare Pages でのセットアップ（具体的手順）

### Step 1: GitHub に push

```bash
cd /Users/tatsu/Desktop/仕事/05.表現活動/aiプロダクト/clamp_calculator

git add .
git commit -m "feat: finalize security and documentation"
git push origin main
```

### Step 2: Cloudflare Pages に接続

1. **Cloudflare アカウント作成**
   ```
   https://dash.cloudflare.com/sign-up
   ```

2. **Pages に接続**
   ```
   ダッシュボード左側 → Pages
   → Connect to Git → GitHub 選択
   → "Authorize Cloudflare Pages" をクリック
   → tatsu471/clamp_calculator を選択
   ```

3. **Build Settings**
   ```
   Build command:              （空白 - ビルド不要）
   Build output directory:     ./ （ルートフォルダ）
   Root directory:             /  （ルート）
   ```

4. **Deploy**
   - 自動的に `https://clamp-calculator-xxx.pages.dev` でデプロイされます

### Step 3: セキュリティ設定

**Cloudflare Zero Trust (Access) を有効化：**

1. **Zero Trust ダッシュボードへ移動**
   ```
   https://one.dash.cloudflare.com/
   ```

2. **Access → Applications → Add an application**
   ```
   Application name: Clamp Calculator
   Subdomain: clamp-calculator
   Domain: your-domain.com（カスタムドメイン使用時）
          または pages.dev （デフォルト）
   ```

3. **Authentication policy を作成**
   ```
   Policy name: Allow only my team
   
   Require:
   ├─ Emails ending with: @your-company.com
   ├─ または One-Time PIN（誰でも OK）
   ├─ または Google 認証
   ```

4. **Save**

**設定後：**
- ユーザーがアクセス → Cloudflare ログイン画面
- 認証後 → アプリが表示される

---

## セキュリティ考慮事項の見直し

「限定公開」であるため、**SNS拡散配布よりリスクは低い**です：

### 優先度を再評価

| 項目 | SNS拡散時 | 限定公開時 | 実装優先度 |
|---|---|---|---|
| **CSP 厳格化** | 🔴 必須 | 🟡 推奨 | 中 |
| **HTTPS** | 🔴 必須 | 🔴 必須 | 高 |
| **入力検証** | 🔴 必須 | 🟡 推奨 | 中 |
| **アクセス制御** | ❌ 不可能 | 🔴 必須 | 高 |
| **ログイン** | ❌ 不可能 | 🟡 推奨 | 中 |
| **エラーハンドリング** | 🔴 必須 | 🟢 オプション | 低 |

### 限定公開向けの推奨実装

**必須（Phase 1: 1-2日）:**
```
☑ Cloudflare Pages でホスト
☑ Cloudflare Access でログイン設定
☑ HTTPS で運用開始
☑ README に「限定公開」と記載
```

**推奨（Phase 2: 1-2日、あれば）:**
```
☑ CSP を厳格化（任意）
☑ 入力値検証を強化（任意）
☑ エラーハンドリング改善（任意）
```

**オプション（Phase 3: 余裕があれば）:**
```
☑ ログイン履歴の記録（Cloudflare がやる）
☑ API レート制限（Cloudflare が可能）
☑ DDoS 防止（Cloudflare が自動）
```

---

## 実装チェックリスト（限定公開版）

### 最小限（これだけはやる）

```
ホスティング:
  ☐ Cloudflare Pages アカウント作成
  ☐ GitHub リポジトリ接続
  ☐ 自動デプロイ設定完了
  ☐ HTTPS で動作確認

アクセス制限:
  ☐ Cloudflare Access 設定完了
  ☐ ログイン画面が表示されることを確認
  ☐ 認証ユーザーのみアクセス可能を確認

ドキュメント:
  ☐ README に「限定公開」と記載
  ☐ Cloudflare Access のログイン方法を説明
  ☐ SECURITY.md を「限定公開向け」に修正
```

### あると良い（追加で 1-2日）

```
セキュリティ:
  ☐ CSP を厳格化
  ☐ Security headers を設定

テスト:
  ☐ Chrome で動作確認
  ☐ Safari で動作確認
  ☐ ログイン認証が機能することを確認

監視:
  ☐ Cloudflare Analytics 有効化
  ☐ アクセス数・エラー数の監視開始
```

---

## よくある質問：限定公開版

### Q1: Cloudflare Access を使うと料金がかかる？

**A:** 
- Cloudflare Pages (無料) + Cloudflare Access (無料 tier で 1 アプリ)
- つまり **完全無料**
- メール認証は回数制限なし

有料になるケース：
- 複数アプリを管理したい
- 多数のユーザーを管理したい（Pro 以上）

---

### Q2: 「限定公開」だけど、どれくらいセキュアか？

**A:**
```
Cloudflare Access を使った場合:

✅ 安全:
  - Cloudflare のサーバーで認証（Google level の堅牢性）
  - ブルートフォース攻撃検知
  - セッション管理が自動

❌ 注意:
  - アクセス可能なユーザーが増えると、内部漏洩リスク増加
  - ユーザーのパスワード管理は各自に依存
  - アプリ内に機密データを入れないこと
```

---

### Q3: URL を知っている人は誰でもアクセス可能？

**A:** いいえ。Cloudflare Access を使う場合：
- URL を知っていても、ログイン認証が必須
- ログイン方法は設定で選択可能：
  - メールアドレス（One-Time PIN）
  - Google/GitHub 認証
  - 企業 SSO（有料）

---

### Q4: 自前サーバーと Cloudflare、どちらが安全？

**A:**
```
Cloudflare Pages:
  ✅ 自動で HTTPS, DDoS 対策, CDN キャッシュ
  ✅ Cloudflare が保守（セキュリティ更新も自動）
  ❌ コントロールが限定的

自前サーバー:
  ✅ 完全なコントロール
  ✅ IP 制限など細かい設定可能
  ❌ セキュリティ保守は自己責任
  ❌ パッチ管理が大変
  ❌ 24/7 監視が必要

→ 限定公開なら Cloudflare推奨
  （保守コスト低く、セキュリティは Cloudflare に任せられる）
```

---

### Q5: GitHub のコードは公開なのに、サイトは限定公開？

**A:** はい、その通り。実装可能です：

```
GitHub (パブリック):
  - コード は誰でも見える
  - ただし知識がないと使えない

Cloudflare Pages (限定公開):
  - URL を知っていても、パスワード必須
  - 一般人はアクセス不可

メリット:
  - セキュリティ分野の人がコードレビュー可能
  - 教育目的で参考にできる
  - でも通常ユーザーは保護される
```

---

## Cloudflare Pages へのデプロイ（ステップバイステップ）

### 準備（初回のみ）

```bash
# 現在のディレクトリ
cd /Users/tatsu/Desktop/仕事/05.表現活動/aiプロダクト/clamp_calculator

# GitHub に最新版をアップロード
git add .
git commit -m "Ready for Cloudflare deployment"
git push origin main

# GitHub を確認 → clamp-calculator.html が見える ✓
```

### Cloudflare 設定（5 分）

1. **https://dash.cloudflare.com/sign-up**
   - メールアドレスで登録
   - パスワード確認

2. **左メニュー:** Pages (ないなら検索)
   - **「Connect to Git」** をクリック
   - GitHub を選択
   - 「Authorize Cloudflare Pages」
   - GitHub でのみ使用するアプリ認可

3. **リポジトリ選択**
   - `Tatsu471/clamp_calculator` を選択

4. **Build 設定**
   ```
   Build command: （空のまま）
   Build output directory: ./ または ./
   Root directory: /
   ```
   → **Save & Deploy** をクリック

5. **デプロイ完了**
   ```
   Your site is live at:
   https://clamp-calculator-abc123.pages.dev
   ```

### Cloudflare Access 設定（10 分）

1. **https://one.dash.cloudflare.com/**
   - Cloudflare Zero Trust へログイン

2. **左メニュー:** Settings → Domains
   - Pages デプロイ時のドメインを追加

3. **左メニュー:** Access → Applications
   - **Add an application** をクリック
   
4. **Application 設定**
   ```
   Application name: Clamp Calculator
   Session duration: 24 hours
   Subdomain: clamp-calculator-public
   Domain: pages.dev （または your-domain.com）
   ```

5. **Authentication Policies**
   ```
   ☐ Include (Allow)
   
   ┌─────────────────────┐
   │ Require             │
   │ ☑ Emails ending in  │
   │  [your-email@...]   │
   └─────────────────────┘
   
   → Add your teammates' emails
   ```

6. **Save**

### アクセステスト

```
1. ページを開く
2. Cloudflare ログイン画面が出る ✓
3. メールアドレス入力
4. Cloudflare が One-Time PIN を送信
5. PIN を入力 → アプリが表示される ✓
```

---

## セキュリティドキュメントの修正（推奨）

現在の `SECURITY_FOR_PUBLIC_DISTRIBUTION.md` は「SNS拡散向け」のため、限定公開用に修正を入れることをお勧めします：

**修正ポイント：**

```markdown
## このアプリの配布モデル

🔹 **ソースコード**: GitHub パブリック
   - 技術者がコードレビュー可能
   - セキュリティ脆弱性の報告も可能

🔹 **ホスティング**: Cloudflare Pages（限定公開）
   - URL を知っていても、Cloudflare Access で認証が必須
   - Cloudflare がメール認証を管理
   - IP 制限や DDoS 対策は自動

🔹 **対象ユーザー**: 招待ユーザーのみ
   - 企業内部、チーム内での利用を想定
   - SNS での無制限配布は対象外
```

---

## 自前サーバーを使いたい場合の参考

Cloudflare ではなく自分でサーバーを運用する場合：

### 推奨: DigitalOcean App Platform

```bash
# git を DigitalOcean に接続
# DigitalOcean → App Platform → Create App
# → GitHub リポジトリ選択
# → Build command: 空（単一 HTML）
# → Deploy

# URL: https://clamp-calculator-xyz.ondigitalocean.app/
# コスト: 月 $5-12
```

### 推奨: Heroku （廃止予定）

```
※ 2023年11月に Heroku の無料 tier が廃止されたため、現在は非推奨
  代わりに Railway, Render などを検討
```

### 推奨: Railway

```bash
# https://railway.app/ にアクセス
# GitHub 連携 → Deploy

# コスト: 月 $5~ （使用量に応じた従量課金）
```

---

## 最終推奨構成（あなた向け）

```
┌────────────────────────────────────────────┐
│ Step 1: Cloudflare Pages にデプロイ        │
│ （HTTPS 自動、世界中 CDN で高速配信）      │
│ 所要時間: 5 分                              │
└────────────────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────┐
│ Step 2: Cloudflare Access で認証設定       │
│ （メール認証で限定公開）                   │
│ 所要時間: 10 分                             │
└────────────────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────┐
│ Step 3: README に接続情報を記載            │
│ （招待ユーザーへの説明）                   │
│ 所要時間: 5 分                              │
└────────────────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────┐
│ 完成! 限定公開 Web アプリ                  │
│ コスト: 無料 tier で十分                    │
│ セキュリティ: 企業レベル                   │
└────────────────────────────────────────────┘
```

**総セットアップ時間: 約 20 分**
**月額コスト: ¥0（無料 tier で十分）**

---

## 参考：Cloudflare の全体構成

```
   ユーザー
      ↓
   インターネット
      ↓
「Cloudflare Access」（認証層）
  ├─ Email: メール認証
  ├─ GitHub: GitHub 認証
  └─ Google: Google 認証
      ↓
   「Cloudflare Pages」（ホスト）
  ├─ HTTPS 自動
  ├─ DDoS 防止
  ├─ CDN（高速化）
  └─ Analytics
      ↓
「clamp-calculator.html」（アプリ）
  ├─ セキュアなコード
  ├─ 入力検証
  └─ エラーハンドリング
```

---

## まとめ

**あなたの用途（限定公開）に最適な選択肢：**

| 観点 | 評価 |
|---|---|
| **簡単さ** | ⭐⭐⭐⭐⭐ （20分で完成） |
| **セキュリティ** | ⭐⭐⭐⭐ （企業レベル） |
| **コスト** | ⭐⭐⭐⭐⭐ （無料） |
| **スケーラビリティ** | ⭐⭐⭐⭐ （数千ユーザーまで対応） |
| **カスタマイズ性** | ⭐⭐⭐（基本的な設定は可能） |

---

**このドキュメント:**
- **対象**: 自前サーバーで Cloudflare を使う開発者
- **作成日**: 2026年2月15日
- **バージョン**: 1.0
