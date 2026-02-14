# SNS配布時のセキュリティガイドライン

## はじめに：「クローズド」と「オープン配布」のリスク差

| 運用シーン | 対象ユーザー | リスク度 | 対応レベル |
|---|---|---|---|
| **ローカル実行** | 本人のみ | 🟢 低 | 最低限（現在のレベル） |
| **社内イントラネット** | 社員のみ | 🟡 中 | 基本的対応 ✅ 実装済 |
| **GitHub 限定公開** | 招待者のみ | 🟡 中 | 基本的対応 ✅ 実装済 |
| **GitHub パブリック** | 全世界 | 🟠 高 | ⬜ **今ここ** (このドキュメント) |
| **SNS 拡散配布** | 全世界＋無知識層 | 🔴 最高 | **本ドキュメント推奨** |

---

## SNS配布時に追加で気をつけるべき 11 項目

### 🔴 優先度A：必ず実装（セキュリティホール防止）

#### A-1: Content Security Policy を厳格化

**現在の状態（最小限）:**
```html
<meta http-equiv="Content-Security-Policy" content="
  default-src 'self'; 
  style-src 'self' 'unsafe-inline'; 
  script-src 'self' 'unsafe-inline'
">
```

**問題点：** `'unsafe-inline'` を許可しているため、インラインスクリプトの XSS 脆弱性に対してほぼ無防備です。

**推奨: インラインスクリプトを外部化**

```html
<!-- ❌ 現在: HTML に JavaScript が埋め込まれている -->
<script>
  // 1000行のコード...
  function updateAll() { ... }
</script>

<!-- ✅ 推奨: 外部ファイルに分離 -->
<script src="app.js"></script>
```

**そのうえで CSP を厳格化:**
```html
<meta http-equiv="Content-Security-Policy" content="
  default-src 'self';
  style-src 'self';
  script-src 'self';
  font-src 'self' https:;
  img-src 'self' data:;
  connect-src 'none'
">
```

**新しい CSP のポリシー解説:**

| ディレクティブ | 内容 | 効果 |
|---|---|---|
| `default-src 'self'` | デフォルトは同一オリジンのみ | 外部リソースの無認可読み込みを防ぐ |
| `style-src 'self'` | CSS は同一オリジンのみ | インラインスタイルを禁止 |
| `script-src 'self'` | JS は同一オリジンのみ | インラインスクリプト＋外部悪意スクリプトを禁止 |
| `font-src 'self' https:` | フォントは同一＆HTTPS | Google Fonts 等を許可しつつ安全に |
| `connect-src 'none'` | ネットワーク通信禁止 | fetch / XMLHttpRequest を禁止 |

**実装手順:**

1. **`app.js` に JavaScript を分離**
   ```
   clamp-calculator.html
   ├── app.js          ← すべての JavaScript をここに
   └── styles.css      ← CSS もあれば外部化
   ```

2. **HTML から `<script>` タグを削除**
   ```html
   </head>
   <body>
     <!-- ... -->
     <script src="app.js"></script>  ← これだけ残す
   </body>
   </html>
   ```

3. **`app.js` に `'use strict'` を追加**
   ```javascript
   'use strict';  ← これを最初に
   
   // グローバル変数を最小化
   const baseFontSize = 16;
   // ...
   ```

**なぜ重要か:**
- SNS 経由で多くの「コード理解度が低いユーザー」が使う
- 開発マシンに侵入されて、`clamp-calculator.html` が改ざんされるリスク
- 改ざんされたファイルが配布されると、ユーザーのブラウザで悪意スクリプトが実行される

**新人向けポイント:**
> これは「カプセル化」と同じです。JavaScript を外部ファイル化することで、「何を実行すべきか」が明確になり、セキュリティレビューも簡単になります。

---

#### A-2: 外部リソース（フォント、アイコン等）の Subresource Integrity (SRI) チェック

**リスク状況:**

SNS 配布時、不特定多数がファイルをさまざまな環境でホストする可能性があります。その過程で：

```
❌ 危険なシナリオ例:
1. ユーザーが「Google Fonts」から Inter フォントをダウンロード
2. 途中で中間者（ISP、WiFi ルータ等）に改ざん
3. 改ざんされたフォント内に悪意スクリプトが埋め込まれている
4. ユーザーが知らないうちに悪意スクリプトが実行

✅ SRI で防止:
1. HTML に「このフォントのハッシュ値は xxxx であるべき」と記載
2. ブラウザが「ダウンロード時のハッシュ値」と比較
3. 改ざんされていたら、ブラウザが読み込みを拒否
```

**現在のコード確認:**

[clamp-calculator.html](clamp-calculator.html) を見ると、フォントはシステムフォントのみ（外部ダウンロードなし）：

```html
font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", ...
```

このため **現状では SRI 不要**です。ただし将来的に外部リソースを追加する場合：

```html
<!-- ❌ 非推奨: SRI なし -->
<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Inter">

<!-- ✅ 推奨: SRI ハッシュ付き -->
<link 
  rel="stylesheet" 
  href="https://fonts.googleapis.com/css?family=Inter"
  integrity="sha384-NXmAk4cUPCQlKOUIZDG0jKsAg0G07tN+7U6/6w6j41N6X1/HO76oKt7xw7n7a0T5"
  crossorigin="anonymous"
>
```

**SRI ハッシュの取得方法:**

1. [SRI Hash Generator](https://www.srihash.org/) にアクセス
2. 外部リソースの URL を入力
3. 生成されたハッシュをコピ

**チェックリスト:**
```
☐ このアプリで外部リソース（CDN、Google Fonts等）を使用していないか確認
☐ 将来的に使う場合は、SRI ハッシュを必ず付ける
☐ package.json があれば npm でのパッケージ依存も確認
```

**新人向けポイント:**
> 目に見えませんが、ネットワークは「信頼できない」という原則（Zero Trust）が業界標準になっています。SRI はその実装の一例です。

---

#### A-3: HTTPS 強制（ブラウザ警告の抑止）

**リスク状況:**

ユーザーが HTTP（暗号化なし）でアクセスすると、ブラウザに警告が表示されます：

```
🔴 警告: 「このサイトは安全ではありません」
　 （クリップボード API が HTTPS を要求するため）
```

**対応方法:**

| ホスト先 | 対応 | 設定 |
|---|---|---|
| **GitHub Pages** | ✅ 自動 HTTPS | `https://username.github.io/repo` |
| **Vercel / Netlify** | ✅ 自動 HTTPS | `https://yourapp.vercel.app` |
| **自前サーバー** | 🟡 手作業 | Let's Encrypt で SSL 証明書取得 |
| **AWS S3 + CloudFront** | ✅ 手作業 | CloudFront で HTTPS 配信 |

**最も簡単な方法: GitHub Pages を使う**

```bash
# GitHub にアップロード後、
# リポジトリ Settings → Pages → Branch を main に設定
# 自動的に https://tatsu471.github.io/clamp_calculator でホスト可能
```

**クリップボード API を使っている場合:**

```javascript
// クリップボードコピー機能（現在のコードに含まれます）
navigator.clipboard.writeText(textToCopy)
  .then(() => { /* ... */ })
  .catch(err => console.error(err));

// このコードは HTTPS (またはローカルhost) でのみ動作
// HTTP では自動的にエラー
```

**チェックリスト:**
```
☐ GitHub Pages で無料ホスト設定済み (HTTPS自動)
☐ または Vercel/Netlify を使用 (HTTPS自動)
☐ または自前サーバーで Let's Encrypt 設定済み
☐ ユーザー向けドキュメントに「HTTPS でアクセスしてください」と記載
```

**新人向けポイント:**
> **HTTP は「郵便はがき」です。誰でも内容を読める。HTTPS は「手紙を鍵付き封筒」に入れるイメージ。** 特に機密設計データを扱う際はこだわるべきです。

---

#### A-4: 入力値検証の強化（integer overflow / DDoS 防止）

**現在の実装:**
```javascript
const min = parseFloat(document.getElementById('mobileMin').value);
if (isNaN(min)) { /* skip */ }
```

**問題点:** 非常に大きな数値（9999999999999999）を入力されて：
1. 計算が遅くなる
2. ブラウザが hang（反応停止）する
3. モバイルデバイスで battery が急消費

**推奨: より厳格な検証を追加**

```javascript
function validateFontSize(px, minVal = 4, maxVal = 500) {
  const num = parseFloat(px);
  
  // Step 1: 数値か?
  if (isNaN(num) || !isFinite(num)) {
    return { valid: false, error: '有効な数値を入力してください' };
  }
  
  // Step 2: 整数か?（小数点以下は許可）
  if (num !== Math.floor(num * 10) / 10) {
    return { valid: false, error: '小数第一位までにしてください' };
  }
  
  // Step 3: 正の数か?
  if (num <= 0) {
    return { valid: false, error: 'フォントサイズは正の数を入力してください' };
  }
  
  // Step 4: 現実的な範囲か? (4px 〜 500px)
  if (num < minVal || num > maxVal) {
    return { 
      valid: false, 
      error: `フォントサイズは ${minVal}px 〜 ${maxVal}px の範囲で入力してください` 
    };
  }
  
  return { valid: true };
}

// 使用例
const result = validateFontSize(document.getElementById('mobileMin').value);
if (!result.valid) {
  showError(result.error);  // ユーザーに理由を教える
  return;
}
```

**各項目の意義:**

| チェック | なぜ? |
|---|---|
| `isFinite()` | Infinity を検出（極端な計算ミス防止） |
| `小数第一位まで` | precision を制限（丸め誤差防止） |
| `正の数` | 負の値で計算が破綻するのを防ぐ |
| `4px 〜 500px` | 非現実的な値を拒否（計算負荷軽減） |

**新人向けポイント:**
> 「検証は面倒」と思うかもしれませんが、ユーザー（特に意図せず大きい値を入力した人）を保護することになります。

---

### 🟡 優先度B：できれば実装（リスク低減）

#### B-1: エラーハンドリングの詳細化

**現在:**
```javascript
if (isNaN(width) || width === 0 || width === 0) {
  resultEl.textContent = '...';
  return;
}
```

**問題:** ユーザーには何が悪いのか分からない。

**推奨:**
```javascript
function calculateVw() {
  const px = parseFloat(document.getElementById('vwPx').value);
  const width = parseFloat(document.getElementById('vwWidth').value);
  const resultEl = document.getElementById('vwResult');
  const errorEl = document.getElementById('vwError');  // エラー表示用

  // エラーをクリア
  errorEl.style.display = 'none';

  // チェック: 入力されているか?
  if (!document.getElementById('vwPx').value.trim()) {
    showErrorMessage('目標ピクセルサイズを入力してください');
    return;
  }
  
  if (!document.getElementById('vwWidth').value.trim()) {
    showErrorMessage('デザイン幅を入力してください');
    return;
  }

  // チェック: 有効な数値か?
  if (isNaN(px) || isNaN(width)) {
    showErrorMessage('数値を入力してください');
    return;
  }

  // チェック: ゼロ除算?
  if (width === 0) {
    showErrorMessage('デザイン幅は 0 以外の値を入力してください');
    return;
  }

  // ここに来たら安全に計算できる
  const vwVal = ((px / width) * 100).toFixed(4);
  resultEl.textContent = `${vwVal}vw`;
}

function showErrorMessage(message) {
  const errorEl = document.getElementById('vwError');
  errorEl.textContent = '⚠️ ' + message;
  errorEl.style.display = 'block';
  errorEl.style.color = '#ff6b6b';
}
```

**ユーザーへの表示例:**
```
⚠️ 目標ピクセルサイズを入力してください
```

**メリット:**
- ユーザーが「何をすべきか」が明確
- デバッグが簡単（エラーログから何が起きたか推測可能）
- 新人プログラマでも修正しやすい

---

#### B-2: パフォーマンス監視（意図しない hang 検知）

**リスク:** 計算が重くなってブラウザが反応しなくなるケース

```javascript
// 計算開始時刻を記録
const startTime = performance.now();

// 計算実行
const result = calculateClamp(min, max, minWidth, maxWidth);

// 計算時間を測定
const endTime = performance.now();
const duration = endTime - startTime;

// 1秒以上かかったら warning
if (duration > 1000) {
  console.warn(`⚠️ 計算に ${duration.toFixed(0)}ms かかりました（重い）`);
}

console.log(`✓ 計算完了: ${duration.toFixed(2)}ms`);
```

**ローカル開発での確認:**

```javascript
// DevTools コンソール（F12）で確認可能
performance.mark('start');
// 処理
performance.mark('end');
performance.measure('my-measurement', 'start', 'end');

// DevTools → Performance タブで可視化
```

**本番での使用（オプション）:**

```javascript
// エラートラッキングサービスに送る場合
if (duration > 1000) {
  sendToAnalytics({
    event: 'slow_calculation',
    duration_ms: duration,
    user_agent: navigator.userAgent
  });
}
```

---

#### B-3: ブラウザ互換性テスト

**対象ブラウザ:**

| ブラウザ | シェア | テスト優先度 |
|---|---|---|
| **Chrome** | 65% | 🔴必須 |
| **Safari** | 20% | 🟡推奨 |
| **Firefox** | 10% | 🟡推奨 |
| **Edge** | 4% | 🟢オプション |
| **IE 11** | <1% | スキップ可 |

**テスト項目:**

```
デスクトップ:
  ☐ Chrome (最新)
  ☐ Firefox (最新)
  ☐ Safari (最新)
  ☐ Edge (最新)

モバイル:
  ☐ Chrome for Android
  ☐ Safari for iOS
  ☐ Firefox for Android

古いバージョン:
  ☐ Chrome -2 (1-2年前)
  ☐ Safari -2 (1-2年前)
```

**テスト時のチェック:**
```
見た目:
  ☐ UI が壊れていないか
  ☐ フォントが正しく表示されるか
  ☐ レイアウトが崩れていないか

機能:
  ☐ 計算が正しく動作するか
  ☐ クリップボードコピーが動くか
  ☐ プレビューがリサイズできるか

パフォーマンス:
  ☐ 計算が遅くないか
  ☐ メモリ使用量が増え続けないか（メモリリーク）
```

**メモリリークの簡単なチェック:**

DevTools → Memory タブで「Heap Snapshots」を撮って、操作前後を比較。増え続けていたら問題。

---

### 🟢 優先度C：参考（知識レベル）

#### C-1: プライバシーポリシー（法的に必要な場合）

**ご自身のアプリの状態:**
このアプリは：
- ✅ クッキーなし
- ✅ localStorage なし
- ✅ 分析ツール（Google Analytics）なし
- ✅ サーバー通信なし

このため **プライバシーポリシー不要**です。

ただし将来的に以下を追加する場合：
- ❌ Google Analytics
- ❌ Mixpanel（使用統計）
- ❌ ユーザー認証
- ❌ クラウド保存

...上記の場合は **プライバシーポリシーが必須**（GDPR, CCPA 対応）。

**簡方針の例:**
```markdown
## プライバシーポリシー

### データ収集
本アプリケーションは以下のデータを収集・保存します：
- アナリティクス: Google Analytics を使用
- クッキー: ユーザー設定の保存用（ローカルのみ）

### データ共有
収集データは第三者と共有されません。

### ユーザー権利
ユーザーはいつでもローカルストレージをクリアして、
すべての設定をリセットできます。
```

---

#### C-2: バージョニング・ロールバック戦略

**重要な脆弱性が見つかった場合:**

```bash
# 現在のバージョン
git tag v3.1

# セキュリティ修正版を作成
git checkout -b hotfix/security-xss
# 修正を加える
git commit -m "Fix: XSS vulnerability in reverse calculator"

# 修正をマージ
git checkout main
git merge hotfix/security-xss
git tag v3.1.1  # パッチバージョンアップ

# ユーザーに通知
# README に「v3.1.1 で脆弱性修正」と記載
```

**バージョン番号の意味:**
```
v3.1.1
  ↑ パッチ: バグ修正（後方互換性あり）
   ↑ マイナー: 新機能（後方互換性あり）
    ↑ メジャー: 破壊的変更
```

---

#### C-3: issue テンプレート（ユーザー報告用）

GitHub リポジトリに以下を作成：

`.github/ISSUE_TEMPLATE/bug_report.md`:
```markdown
---
name: Bug Report
about: バグを報告する
title: "[BUG] "
labels: bug
assignees: ''

---

**バグの説明**
発生した問題を簡潔に説明してください。

**再現手順**
1. '...' に移動
2. '...' をクリック
3. '...' と入力
4. エラーが表示される

**期待される動作**
どのような結果になるはずだったか説明してください。

**環境**
- ブラウザ: (例: Chrome 120.0)
- OS: (例: macOS 14.2)
- デバイス: (例: MacBook Pro 2023)

**スクリーンショット**
問題を説明するのに役立つスクリーンショットがあれば、説明してください。
```

---

## 実装チェックリスト（SNS配布前に必須）

### Phase 1: セキュリティ強化（1-2 日）
```
コード構成:
  ☐ JavaScript を app.js に分離
  ☐ CSS を style.css に分離（必要なら）
  ☐ インラインスクリプト・スタイルをすべて削除
  ☐ 'use strict' を app.js に追加

CSP:
  ☐ CSP を厳格化（外部リソース限定）
  ☐ SRI ハッシュを外部リソースに追加（あれば）
  ☐ CSP 違反の browser console テスト

入力値検証:
  ☐ すべての数値入力に isFinite() チェック追加
  ☐ 境界値チェック（最小値・最大値）を実装
  ☐ ゼロ除算などのエッジケース処理

エラーハンドリング:
  ☐ try-catch で予期しないエラーを catch（必要なら）
  ☐ ユーザーフレンドリーなエラーメッセージ
  ☐ console.error でログ出力
```

### Phase 2: 配信準備（1 日）
```
ホスティング:
  ☐ GitHub Pages 設定完了（HTTPS自動）
  ☐ または Vercel/Netlify 設定完了
  ☐ カスタムドメイン設定（不要なら OK）

ドキュメント:
  ☐ README に「このアプリは本番環境では... 」と記載
  ☐ SECURITY.md を最新化
  ☐ Changelog 追加（バージョン履歴）
  ☐ Code of Conduct 追加（コミュニティ規約）

テスト:
  ☐ Chrome でテスト
  ☐ Safari でテスト
  ☐ Firefox でテスト
  ☐ モバイル Safari / Chrome でテスト
  ☐ HTTPS で全機能動作確認
  ☐ DevTools console にエラーがないか確認

SNS:
  ☐ テンプレート文を作成（LinkedIn, Twitter)
  ☐ スクリーンショット・デモ動画を準備
  ☐ 「セキュリティ、プライバシーは安全」を明記
```

### Phase 3: 監視・保守（継続）
```
問題追跡:
  ☐ GitHub Issues を定期的に確認
  ☐ 報告されたバグに 1週間以内に返信
  ☐ セキュリティ報告は「非公開」で対応

アップデート:
  ☐ ブラウザの新機能チェック（四半期ごと）
  ☐ セキュリティアラート監視（GitHub Security tab）
  ☐ 脆弱性報告の対応（最優先）

ユーザーサポート:
  ☐ よくある質問を README/Discussions に集約
  ☐ 使い方動画を YouTube に投稿
```

---

## SNS 投稿例（参考）

### Twitter / X 向け
```
🎨 Clamp Calculator v3.1 を公開しました

✨ CSS clamp() を計算・プレビューできるツールです
- 📱 Mobile / Desktop の自動スケーリング
- 🎯 SCSS mixin 生成
- 🔒 ローカル実行で安全
- 📥 コピー1クリック

ソースコード
https://github.com/Tatsu471/clamp_calculator

#CSS #Typography #デザイン発剤 #TypeScript
```

### LinkedIn 向け
```
I've published a responsive typography calculator that 
pairs with SCSS mixins and clamp() functions.

🔧 Features:
- Auto-scale calculations (Mobile/Desktop)
- Real-time preview with resize handle
- VW converter with context awareness
- Fully local-running (no external API calls)
- MIT licensed, open source

👉 [Link]

Security note: All calculations run in-browser. 
No data is sent to external servers.
```

### GitHub Discussions（コミュニティ）
```
title: Share your use case

Have you used Clamp Calculator? We'd love to hear about it!

Please share:
- What design system do you use it for?
- Any feature requests?
- Did you find any bugs?

Feel free to open an issue or discussion thread.
```

---

## 最後に：SNS配布時のマインドセット

### 「不特定多数」の想定
SNS配布では、以下のようなユーザーが使います：

```
✅ 期待通りのユーザー:
  - フロントエンドエンジニア
  - デザイナー
  - セキュリティに関心のある人

⚠️ 予想外のユーザー:
  - コードを読めない人
  - セキュリティ知識のない人
  - 「なぜ安全なのか」を理解していない人
  - 「改変して再配布」する人
  - 脆弱性を探そうとする人（善意と悪意の両方）
```

### セキュリティの責任
ユーザーに多いほど：
- 報告される脆弱性も多い
- エラー報告も増える
- 対応のコストも高まる

このため、事前の対応が重要です。

### コミュニティになる
SNS配布で数百～数千のユーザーが使い始めると、単なる「ツール」から「コミュニティ」になります。

その時に必要なのは：
- **透明性**: セキュリティ上の判断を理由とともに説明
- **反応性**: 報告に対して迅速に対応
- **親切さ**: エラーメッセージやドキュメントをユーザーフレンドリーに

---

## 参考資料

### セキュリティ関連
- [OWASP Top 10](https://owasp.org/www-project-top-ten/) - Web アプリケーション脆弱性 Top 10
- [Content Security Policy (MDN)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [Subresource Integrity (MDN)](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)
- [Mozilla Observatory](https://observatory.mozilla.org/) - セキュリティスコア測定

### パフォーマンス
- [Web.dev: Performance](https://web.dev/performance/)
- [Performance API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Performance)

### ホスティング（HTTPS 無料）
- [GitHub Pages](https://pages.github.com/)
- [Vercel](https://vercel.com/)
- [Netlify](https://netlify.com/)

### テスト関連
- [BrowserStack](https://www.browserstack.com/) - マルチブラウザテスト（有料）
- [LambdaTest](https://www.lambdatest.com/) - クロスブラウザテスト（有料）

---

**最後に一言:**

SNS配布は「あなたのコードが世界で使われる」ということです。責任は大きいですが、それだけやりがいもあります。

このガイドを参考に、「安全で、使いやすく、信頼できるツール」を目指してください。

---

**このドキュメント:**
- **対象**: Clamp Calculator をSNS配布する開発者
- **作成日**: 2026年2月15日
- **バージョン**: 1.0
- **参考**: OWASP, MDN, GitHub Security Best Practices
