# 簡易的な Web アプリケーション開発ガイドライン

## 対象者

- **新人エンジニア**（1-2年目のレベルを想定）
- **フロントエンド初心者**
- **プロトタイプをリリースしたい人**

このガイドでは「小規模な Webアプリを安全に公開する際に気をつけるべきこと」を、優先度順にまとめました。

---

## はじめに：「簡易的な Webアプリ」とは

このガイドでいう「簡易的な Webアプリ」は、以下のような特徴があります：

| 特徴 | 例 |
|---|---|
| **サーバーレス** | ファイルをブラウザで開くだけで動作 |
| **ローカル処理** | データ送信・外部 API なし |
| **単一ファイル** | HTML + CSS + JavaScript を 1 ファイルに統合 |
| **開発期間** | 数日～数週間 |
| **対象ユーザー** | 個人・チーム内・限定公開 |

**例:** 計算機、単位変換、デザインプレビュー、チェックリスト、など

---

## 優先度別：開発時の注意点

### 🔴 優先度 1：絶対に忘れるな

#### 1.1 セキュリティ宣言（README に明記）

**何をするか:**

「このアプリは安全か？」という質問を事前に防ぐために、README に明確に書きましょう。

**具体的な書き方:**

```markdown
## セキュリティ & プライバシー

✅ **安全な点:**
- すべての計算がブラウザ内で実行
- データはサーバーに送信されません
- ローカルストレージなし、クッキーなし
- オフラインで使用可能

⚠️ **注意事項:**
- 無制限インターネット公開は推奨しません
- 本番環境では HTTPS が必要です（クリップボード連携時）

✅ **安全に使える場面:**
- ローカルファイルとして開く
- 社内イントラネット
- 自分のサーバーにホスト
```

**なぜ必要か:**
- ユーザーが「データを盗まれるかも」という不安を感じず、安心して使える
- 「これ、重要な情報入力しても大丈夫?」という GitHub issue を減らせる
- 企業導入時に「セキュリティ確認」が簡単になる

**新人向けポイント:**
> 「安全です」と書くのではなく、「どのような点が安全なのか」を具体的に説明しましょう。ユーザーは「あなたの確認」ではなく「事実」を信じます。

---

#### 1.2 ライセンスの明記（LICENSE ファイル Required）

**何をするか:**

GitHub に公開するなら、`LICENSE` ファイルを必ず作成してください。

**推奨: MIT ライセンスを 3 行で説明**

```
◆ MIT ライセンスとは？
  「このコードは自由に使ってOK。ただし著作者表示は忘れずに」

◆ どうして必要か？
  ❌ ライセンス明記なし → 「使ってもいい？」という相談が増える
  ✅ ライセンス明記あり → 「自由に使えますね」と相手が判断できる

◆ 企業が導入するときも、法務チェックが簡単
  MIT ライセンス = 企業友好的（Apache, React, Vue など大企業の OSS で採用）
```

**ライセンス選択フローチャート:**

```
あなたのプロジェクトを、
他の人が改変・商用化・再配布するのを許可したい？

  ┌─ YES → MIT ライセンス（最も無制限）
  │         （個人・企業・商用・改変 すべてOK）
  │
  ├─ 改変は OK だが、改変版でも同じライセンスを強制したい
  │ └─ GPL ライセンス（やや厳しい）
  │
  └─ NO → Proprietary（全部禁止）
           （オープンソース化しない場合）
```

**ファイル作成方法:**

**方法A: GitHub が自動生成（推奨）**
1. GitHub のリポジトリ画面で「Add file」→「Create new file」
2. ファイル名を `LICENSE` と入力
3. テンプレートボタンが出現 → MIT を選択

**方法B: テンプレートをコピペ**
```
MIT License

Copyright (c) [西暦] [Your Name]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
[以下省略]
```

**新人向けポイント:**
> **「ライセンスなし = 宙ぶらりん状態」です。** 万が一トラブルになったときに、どちらが悪いかも分からなくなります。必ず決めておきましょう。

---

#### 1.3 入力値検証（バグと攻撃を防ぐ）

**何をするか:**

ユーザーが入力するあらゆる値に対して「それって有効な値?」を確認する処理を書きましょう。

**よくある失敗：**

```javascript
// ❌ ダメ: 何にもチェックしない
const minSize = document.getElementById('minSize').value;
const result = clamp(minSize, maxSize, minWidth, maxWidth);

// 問題が出ます:
// - ユーザーが何も入力しない (undefined)
// - 負の数を入力 (計算が意図と異なる)
// - 文字を入力 (NaN になって UI が壊れる)
// - 極端な数を入力 (ブラウザがクラッシュ)
```

**改善版:**

```javascript
// ✅ 実装例 1: 基本的なチェック
function validateFontSize(px) {
  const num = parseFloat(px);
  
  // チェック 1: 数値か?
  if (isNaN(num)) {
    return false;  // 無効
  }
  
  // チェック 2: 正の数か?
  if (num <= 0) {
    return false;  // 無効（フォントサイズに負の値はない）
  }
  
  // チェック 3: 現実的な範囲か? (4px 〜 500px)
  if (num < 4 || num > 500) {
    return false;  // 無効
  }
  
  return true;  // 有効！
}

// ✅ 実装例 2: 使用方法
const minInput = document.getElementById('minSize').value;
if (!validateFontSize(minInput)) {
  console.log('エラー: 4 〜 500 の数値を入力してください');
  return;  // 処理を中止
}

const minSize = parseFloat(minInput);
// あとは安心して計算できる
```

**各項目での検証例:**

| 項目 | 何をチェック? | 例 |
|---|---|---|
| **フォントサイズ** | 正の数か、範囲内か | `4 <= px <= 500` |
| **ビューポート幅** | 正の数か、範囲内か | `320 <= width <= 2560` |
| **カラーコード** | 16進数か | `^#([0-9A-F]{3,6})?$/i` |
| **数学式** | 括弧は対応してるか | `countParens(expr) === 0` |

**正規表現（regex）を使った検証:**

```javascript
// カラーコード検証の例
function isValidColor(color) {
  // #fff, #ffffff のような形式を許可
  return /^#([0-9A-F]{3}|[0-9A-F]{6})$/i.test(color);
}

// 使用例
console.log(isValidColor('#fff'));      // true
console.log(isValidColor('#ff0000'));   // true
console.log(isValidColor('red'));       // false
console.log(isValidColor('xyz'));       // false
```

**新人向けポイント:**
> 「入力値検証は面倒だ」と思うかもしれませんが、**後でバグ報告を受けるより、今 10 分かけて実装しましょう。** 「その値は入力できません」とユーザーに教えるだけで、信頼性が大幅に上がります。

---

### 🟡 優先度 2：公開前に確認

#### 2.1 セキュリティメタタグ（最低限の保護）

**何をするか:**

HTML の `<head>` セクションに以下の 4 つのメタタグを追加してください。**ひな形をコピペするだけです。**

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  
  <!-- セキュリティメタタグ（ここから） -->
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <meta http-equiv="X-Content-Type-Options" content="nosniff">
  <meta http-equiv="X-Frame-Options" content="SAMEORIGIN">
  <meta http-equiv="X-XSS-Protection" content="1; mode=block">
  <!-- セキュリティメタタグ（ここまで） -->
  
  <title>Your App</title>
</head>
```

**各タグの意味（新人向け簡潔版）:**

| メタタグ | 説明 | どういう攻撃を防ぐ |
|---|---|---|
| `X-Content-Type-Options: nosniff` | 「このファイルは HTML です。.js に変えないで」とブラウザに指示 | ファイル形式の偽装 |
| `X-Frame-Options: SAMEORIGIN` | 「このページは iframe で読み込むのは、同じサイトからだけOK」と指示 | ページの不正な埋め込み（クリックジャッキング） |
| `X-XSS-Protection: 1; mode=block` | 古いブラウザの XSS フィルタを有効化 | XSS（クロスサイトスクリプティング）攻撃 |
| `X-UA-Compatible: ie=edge` | Internet Explorer に「最新エンジンで動かして」と指示 | IE の互換性バグ |

**「これで完全に安全」ではない（重要）:**

このメタタグは最小限の保護です。必要に応じて、さらに強い設定があります：

```html
<!-- 上級: Content Security Policy (CSP)
     これを設定すると、インラインスクリプト/スタイルが禁止される
     単一ファイルアプリではやや complexになるため、まずはメタタグから -->
<meta http-equiv="Content-Security-Policy" content="
  default-src 'self'; 
  style-src 'self' 'unsafe-inline'; 
  script-src 'self' 'unsafe-inline'
">
```

**新人向けポイント:**
> メタタグは「保険」です。100% の安全を保証しませんが、ないよりあるほうがずっともいいです。テンプレートをコピペして、変更は不要です。

---

#### 2.2 GitHub での公開準備

**何をするか:**

以下のファイルが揃っているか確認してから push しましょう。

| ファイル | 必須？ | 用途 |
|---|---|---|
| **README.md** | ✅必須 | プロジェクト説明、使い方 |
| **LICENSE** | ✅必須 | 著作権・利用条件 |
| **.gitignore** | ✅必須 | Git から除外するファイル |
| **SECURITY.md** | ⭐推奨 | セキュリティガイドライン |

**チェックリスト:**

```
ReadMe:
  ☐ 機能説明（箇条書き）があるか?
  ☐ インストール/実行方法が明確か?
  ☐ スクリーンショットがあるか?
  ☐ セキュリティ/プライバシーについて書いてあるか?
  ☐ ライセンス情報があるか?

License:
  ☐ LICENSE ファイルがあるか?
  ☐ MIT か Apache 2.0 など標準的なライセンスか?
  ☐ README で LICENSE について言及しているか?

.gitignore:
  ☐ OS ファイル（.DS_Store など）を除外しているか?
  ☐ IDE 設定（.vscode/ など）を除外しているか?
  ☐ ログ・ temp ファイルを除外しているか?
  ☐ 機密情報（.env など）を除外しているか?
```

**git push の前に:**

```bash
# 最終確認
git status

# 以下が表示されるべき:
# - README.md
# - LICENSE
# - .gitignore
# - clamp-calculator.html
# などの主要ファイルのみ

# node_modules/, *.log, .env などが含まれていたら除外する
git rm --cached node_modules/  # 誤ってコミットした場合
```

**新人向けポイント:**
> GitHub は「あなたのプロジェクト」です。最初の印象が大事です。README がなく、LICENSE もないリポジトリは、誰もフォークしません。**公開前の 30 分の準備が、後の信頼性を左右します**。

---

#### 2.3 エラーメッセージはユーザーフレンドリーに

**何をするか:**

「エラーが発生した」ことを、ユーザーが理解できる言葉で説明しましょう。

**ダメな例：**

```javascript
// ❌ 最悪: エラーメッセージがない
const result = clamp(a, b, c);
// 計算に失敗しても、何も表示されない...

// ❌ 悪い: テクニカルすぎる
if (isNaN(value)) {
  console.error("NaN detected at parseFloat(input.value) in calculateClamp()");
}

// ❌ やや悪い: 冷たい
alert("Invalid input");
```

**良い例：**

```javascript
// ✅ 良い: ユーザーが理解できる
function validateAndCalculate(minInput, maxInput) {
  const min = parseFloat(minInput);
  const max = parseFloat(maxInput);
  
  if (isNaN(min) || isNaN(max)) {
    showError("数値を入力してください");
    return;
  }
  
  if (min >= max) {
    showError("最小値は最大値より小さくしてください");
    return;
  }
  
  if (min < 4 || max > 500) {
    showError("フォントサイズは 4px 〜 500px の範囲を入力してください");
    return;
  }
  
  // すべことがOK → 計算実行
  return calculateClamp(min, max);
}

// エラーを UI に表示
function showError(message) {
  const errorEl = document.getElementById('error-message');
  errorEl.style.color = 'red';
  errorEl.textContent = message;
}
```

**良いエラーメッセージの 3 要素:**

1. **何が間違ったのか** - 「フォントサイズが無効です」
2. **なぜ間違ったのか** - 「4〜500px の範囲が必要です」
3. **どう直したらいいのか** - 「正の数を入力してください」

**新人向けポイント:**
> ユーザーは『プログラマ』ではありません。「NaN」「undefined」理解していません。**「この項目に数値を入力してください」という日本語で十分です。**

---

### 🟢 優先度 3：余裕があれば やると good

#### 3.1 イベントハンドラを最新化（インライン属性を削除）

**現在の（やや古い）書き方:**

```html
<!-- ❌ これは動作しますが、ベストプラクティスではない -->
<button onclick="handleClick()">Click me</button>
<input type="text" oninput="updateValue(this.value)">
```

**推奨される書き方:**

```html
<!-- ✅ HTML は構造のみ -->
<button id="myButton">Click me</button>
<input type="text" id="myInput">

<script>
// JavaScript で「どうするか」を指定
document.getElementById('myButton').addEventListener('click', function() {
  handleClick();
});

document.getElementById('myInput').addEventListener('input', function(event) {
  updateValue(event.target.value);
});
</script>
```

**メリット:**

| 観点 | インライン | addEventListener |
|---|---|---|
| **可読性** | ❌ HTML と JS が混在 | ✅ 役割が分離 |
| **複数ハンドラ** | ❌ 1 つだけ登録可能 | ✅ 複数登録可能 |
| **CSP 対応** | ❌ `'unsafe-inline'` 必須 | ✅ より安全 |
| **保守性** | ❌ 修正時に HTML を触る | ✅ JS のみ修正 |

**新人向けポイント:**
> **今すぐ修正は不要です**。新規プロジェクトで最初から `addEventListener` を使う習慣をつけましょう。

---

#### 3.2 ローカルストレージで設定を保存（オプション）

**何をするか:**

ユーザーが前回設定した値を、次回開いたときに自動復元できます。

```javascript
// 設定を保存
function saveSettings() {
  const settings = {
    baseFontSize: document.getElementById('baseFontSize').value,
    mobileMin: document.getElementById('mobileMin').value,
    mobileMax: document.getElementById('mobileMax').value,
  };
  localStorage.setItem('clampCalculatorSettings', JSON.stringify(settings));
}

// 設定を読み込む
function loadSettings() {
  const saved = localStorage.getItem('clampCalculatorSettings');
  if (saved) {
    const settings = JSON.parse(saved);
    document.getElementById('baseFontSize').value = settings.baseFontSize || 16;
    document.getElementById('mobileMin').value = settings.mobileMin || '';
    document.getElementById('mobileMax').value = settings.mobileMax || '';
  }
}

// ページ読み込み時に復元
window.addEventListener('load', loadSettings);

// ユーザーが値を変更したら保存
document.getElementById('mobileMin').addEventListener('change', saveSettings);
```

**メリット・デメリット:**

| 観点 | メリット | デメリット |
|---|---|---|
| **UX** | ✅ 前回の設定が自動復元される | パソコンが変わるとリセット |
| **プライバシー** | ✅ データはローカルのみ | キャッシュクリア時に削除される |
| **実装難度** | ✅ 数行で実装 | 複数デバイス間で同期されない |

**セキュリティ注意:**
- 機密情報（API キーなど）は localStorage に保存しない
- このアプリではクッキーより安全（XSS リスクが低い）

**新人向けポイント:**
> 便利ですが、「なくても動作する」機能です。基本機能が完成したあとに追加しましょう。

---

## まとめ：開発フロー（Step by Step）

新人が初めて Webアプリを作る場合、このオーダーで進めることをお勧めします：

### Step 1: 機能開発（1 週間）
```html
<input id="input1">
<div id="output">...</div>

<script>
document.getElementById('input1').addEventListener('input', () => {
  const result = calculate(...);
  document.getElementById('output').textContent = result;
});
</script>
```

### Step 2: セキュリティの基本（30 分）
```html
<head>
  <meta http-equiv="X-Content-Type-Options" content="nosniff">
  <meta http-equiv="X-Frame-Options" content="SAMEORIGIN">
</head>
```

### Step 3: 入力値検証（1 時間）
```javascript
if (!validateInput(value)) {
  showError("有効な値を入力してください");
  return;
}
```

### Step 4: ドキュメント作成（1 時間）
- README.md: 機能説明 + セキュリティ明記
- LICENSE: MIT を選択
- SECURITY.md: ガイドライン

### Step 5: GitHub 公開（30 分）
```bash
git add .
git commit -m "Release v1.0"
git push -u origin main
```

### Bonus: ポーランド（あれば）
- localStorage で設定保存
- addEventListener への移行
- より詳細な検証ロジック

---

## Q&A：よくある質問

### Q: 「安全です」と書いてだけで大丈夫?

**A:** いいえ。具体的に説明しましょう。

```markdown
❌ 「このアプリは安全です」

✅ 「このアプリはすべての処理をブラウザで実行しており、
   サーバーへの通信はなく、ローカルストレージも使用していません。
   ご利用いただく通知信が外部に送信されることはありません」
```

---

### Q: どのライセンスを選べばいい?

**A:** 迷ったなら **MIT**。

- MIT: 企業友好的、最も一般的（React, Vue など）
- Apache 2.0: 同じく企業向け（より厳格）
- GPL: 改変版も同じライセンス強制（学術向け）

---

### Q: 「ローカル実行は安全」「インターネット公開は非推奨」の違いは?

**A:** クリップボード API とクッキーの差。

```
ローカル実行（file://）
  ✅ 安全 - 外部との通信がない
  ✅ オフラインでも動作
  ⚠️ 一部ブラウザ API（クリップボード）が制限される

HTTP/HTTPS 公開
  ✅ すべての API が使える（クリップボード、Geolocation など）
  ⚠️ HTTPS が必須（クッキーなど）
  ⚠️ DDoS 対策がいる
  ⚠️ ユーザーがデータ入力時、中間者攻撃のリスク

内部イントラネット
  ✅ HTTPS 不要（閉じた環境なら HTTP OK）
  ✅ ユーザーが社内の人のみ
  ✅ バックアップが簡単

```

---

### Q: 外部 API（Google Fonts など）を使いたい。セキュリティは?

**A:** Subresource Integrity (SRI) ハッシュを使いましょう。

```html
<!-- ❌ 危険: SRI なし（フォントが改ざんされる可能性） -->
<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Inter">

<!-- ✅ 安全: SRI ハッシュで改ざん検知 -->
<link 
  rel="stylesheet" 
  href="https://fonts.googleapis.com/css?family=Inter"
  integrity="sha384-xxxxxxxxxx"
  crossorigin="anonymous"
>
```

[SRI Hash Generator](https://www.srihash.org/) でハッシュを生成できます。

---

### Q: 大きな数値（999999999）を入力されたらどうする?

**A:** 境界値チェックで防ぎましょう。

```javascript
function validateFontSize(px) {
  const num = parseFloat(px);
  
  // 現実的な範囲内か確認
  if (num < 4 || num > 500) {
    return false;
  }
  
  return true;
}
```

**なぜ 500px?** → 一般的なフォントサイズの最大値より十分大きいが、計算が破綻しない「現実的な上限」

---

## チェックリスト：公開グリーンライト

本番公開の前に、以下がすべてチェック済みですか？

```
セキュリティ:
  ☐ 入力値検証が実装されている
  ☐ セキュリティメタタグが HTML に含まれている
  ☐ localStorage に機密情報が保存されていない

ドキュメント:
  ☐ README.md に機能説明がある
  ☐ README に「何が安全か」を具体的に書いてある
  ☐ LICENSE ファイルが存在し、ライセンスが明記されている
  ☐ SECURITY.md に注意事項が書いてある（推奨）

GitHub:
  ☐ README に GitHub へのリンクがある
  ☐ .gitignore が設定されている
  ☐ node_modules などが contain されていない
  ☐ .env ファイルが .gitignore に含まれている

テスト:
  ☐ 正常入力で正しく計算される
  ☐ 負の数を入力しても error handling されている
  ☐ 非常に大きい数を入力しても crash しない
  ☐ 空入力が graceful に handled されている

ブラウザテスト:
  ☐ Chrome / Firefox / Safari で動作確認
  ☐ スマートフォンでも UI が壊れていない
  ☐ JavaScript エラーが console に出ていない
```

---

## 今後の学習方向

このガイドを読んだあと、さらに学びたい場合：

### 次のステップ:
1. **CSP（Content Security Policy）**を深掘り
   - [MDN: Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
   
2. **OWASP Top 10** 脆弱性を理解
   - [OWASP Top 10](https://owasp.org/www-project-top-ten/)
   
3. **テスト駆動開発（TDD）**の実装
   - Jest などのテストフレームワーク

4. **本格的な Webアプリケーションフレームワーク**
   - React, Vue, Svelte など

---

## 参考書籍・リソース

| タイトル | レベル | リンク |
|---|---|---|
| **MDN - Web Security** | 初心者〜中級 | https://developer.mozilla.org/docs/Web/Security |
| **OWASP Top 10** | 初心者 | https://owasp.org/www-project-top-ten/ |
| **Choose a License** | 初心者 | https://choosealicense.com/ |
| **JavaScript Best Practices** | 中級 | https://developer.mozilla.org/docs/Web/JavaScript/Guide |
| **Web Performance APIs** | 中級 | https://developer.mozilla.org/docs/Web/API/Performance |

---

## 最後に

Webアプリケーション開発は「楽しい」ですが「責任がある」です。

ユーザーが信頼してくれるアプリを作るために：

1. **セキュリティを後付けではなく、最初から考える**
2. **ドキュメントは「自分のため」ではなく「ユーザーのため」に書く**
3. **エラーが起きたときは、ユーザーを責めない。詳しく説明する**
4. **「完全な安全」はない。「できる限りの安全」を目指す**

このガイドが、あなたの開発の助けになれば幸いです。

---

**このガイドについて:**
- **対象**: 新人エンジニア
- **最終更新**: 2026 年 2 月 15 日  
- **バージョン**: 1.0
- **参考**: Clamp Calculator v3.1 の開発から学んだベストプラクティス
