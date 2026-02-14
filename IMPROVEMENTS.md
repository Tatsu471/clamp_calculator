# セキュリティ・公開対応の改善内容

## 概要

このドキュメントは、Clamp Calculator v3.1 を安全に公開するための改善内容をまとめたものです。GitHub での public リリース前に実施した改善点と、その目的を記載しています。

---

## 実施した改善

### 1. README.md の充実（プロジェクト説明の整備）

**何をしたのか:**
初期状態では単に「# clamp_calculator」とだけ書かれていた README を、実用的な説明に拡充しました。

**変更内容:**
- ✅ 機能紹介：5つの主要機能を箇条書きで説明
- ✅ クイックスタート：「ファイルをブラウザで開くだけで動作」という簡潔な手順
- ✅ 使い方ガイド：各パネルの役割（Mobile, Desktop, VW Converter など）
- ✅ セキュリティ・プライバシークレーム：「ローカル実行のみ」「データ送信なし」を明記
- ✅ デプロイ時の注意書き：「ローカル開発・内部利用は安全」「無制限公開は非推奨」
- ✅ ライセンス・技術スタック：著作権とビルド不要性を記載

**なぜ必要か:**
GitHub を見た人が「このツールは何か」「安全か」「どう使うか」を 30 秒で理解できるようにするため。特にセキュリティ関連の質問を事前に防ぐ効果があります。

**新人へのアドバイス:**
> README は「プロジェクトの顔」です。コード品質がよくても、説明がなければ誰も使ってくれません。**同僚や後輩が 1 分で理解できる内容に** しましょう。

---

### 2. LICENSE ファイルの追加（著作権・利用条件の明記）

**何をしたのか:**
新たに `LICENSE` ファイルを作成し、**MIT ライセンス**を採用しました。

**ファイル内容:**
```
MIT License

Copyright (c) 2026 Clamp Calculator Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy...
[著作権と利用条件の法的テキスト]
```

**なぜ必要か:**

| 問題 | ライセンスなし | MIT ライセンスあり |
|------|---|---|
| **法的効力** | 曖昧、トラブルリスク | 明確、安全 |
| **再利用可否** | 不明確 | 「改変・商用利用OK、著名義表示必須」と明記 |
| **企業導入** | 法務チェックでNG | チェック OK（MIT は企業友好的） |
| **GitHub警告** | ⚠️ 警告が出る | ✅ 警告なし |

**MIT ライセンスを選んだ理由:**
- 制限が少ない（企業導入しやすい）
- 実績が多い（React, Vue, Node.js など）
- シンプル（テンプレートが豊富）

**新人へのアドバイス:**
> GitHub に公開するなら **必ずライセンスを選びましょう**。ライセンスなしだと「使ってもいい？」という相談が増えます。個人・企業・商用利用を許可したいなら **MIT か Apache 2.0** がおすすめです。

---

### 3. SECURITY.md の新規作成（セキュリティガイドラインの提供）

**何をしたのか:**
セキュリティに関する考慮事項をまとめた `SECURITY.md` ファイルを新規作成しました。

**ファイル構成:**

| セクション | 内容 | 例 |
|---|---|---|
| **Security Properties（安全な点）** | このツールの安全性根拠 | 「外部ネットワーク通信なし」「localStorageなし」 |
| **When It's Safe to Use（推奨用途）** | 使用可能な環境 | ✅ ローカル開発、✅ 社内イントラネット |
| **Deployment Considerations（デプロイ時の注意）** | サーバー公開時の対応 | HTTPS 必須、CSP ヘッダー設定 |
| **Input Validation（入力値検証）** | 現状と改善案 | `isNaN()` チェック、境界値検証 |
| **Vulnerability Disclosure（脆弱性報告）** | セキュリティ問題の報告方法 | 「GitHub issue ではなくメールで報告」 |
| **Compliance（コンプライアンス）** | 規制対応状況 | GDPR/HIPAA 不要（個人情報なし） |

**なぜ必要か：**
- GitHub issue で「これ安全？」という質問を事前に防ぐ
- ユーザーが社内導入時に「セキュリティ確認」をしやすくする
- 開発者に「改善のロードマップ」を提示できる

**新人へのアドバイス:**
> **「セキュリティ = 黒魔術」ではありません**。何が安全で何が危険かを明記するだけで、ユーザーの信頼が大きく向上します。「データ送信なし」「ローカルのみ」という事実を、まず明確に書きましょう。

---

### 4. HTML の `<head>` 部分に セキュリティメタタグを追加

**何をしたのか:**
HTML ファイルの `<head>` セクションに、以下の 5 種類のセキュリティメタタグを追加しました。

#### 4.1 Content Security Policy (CSP)
```html
<meta http-equiv="Content-Security-Policy" content="
  default-src 'self'; 
  style-src 'self' 'unsafe-inline'; 
  script-src 'self' 'unsafe-inline';
">
```

**役割:** ブラウザに「どのリソース（スクリプト・スタイル）を読み込んでいいか」を指示する

**保護内容:**
- XSS 攻撃（悪意あるスクリプト注入）を軽減
- 外部スクリプトの無認可読み込みを防止
- ブラウザキャッシュの悪用を防ぐ

#### 4.2 X-Content-Type-Options
```html
<meta http-equiv="X-Content-Type-Options" content="nosniff">
```

**役割:** ブラウザに「MIME タイプを勝手に変えるな」と指示

**保護内容:**
- `.html` ファイルを `.js` として実行させられるのを防ぐ
- Content-Type ヘッダーが間違っていても、ブラウザが勝手に修正しない

**例:**
```
❌ 危険: Content-Type が text/plain でも、ブラウザが勝手に .js と判定
✅ 安全: nosniff により、指定された MIME タイプを厳格に守る
```

#### 4.3 X-Frame-Options
```html
<meta http-equiv="X-Frame-Options" content="SAMEORIGIN">
```

**役割:** ブラウザに「このページを iframe で読み込んでいいのは同一オリジンのみ」と指示

**保護内容:**
- クリックジャッキング（ユーザーが気付かないうちに操作させる攻撃）を防止
- 悪質なサイトにこのツールが内包されるのを防ぐ

**例:**
```
❌ 危険: https://evil.com が <iframe src="https://yoursite.com/app"> で内包
✅ 安全: SAMEORIGIN により、https://yoursite.com からのみ iframe 可能
```

#### 4.4 X-XSS-Protection
```html
<meta http-equiv="X-XSS-Protection" content="1; mode=block">
```

**役割:** 古いブラウザ（IE, 古い Chrome）の XSS フィルタを有効化

**保護内容:**
- ブラウザ組み込みの XSS 検出と防止を有効化
- 検出時にページをブロック（mode=block）

**注:** 最新ブラウザでは CSP で対応されるため、レガシー対応です

#### 4.5 X-UA-Compatible
```html
<meta http-equiv="X-UA-Compatible" content="ie=edge">
```

**役割:** Internet Explorer に「最新エンジンモードで動かせ」と指示

**保護内容:**
- IE の互換モードバグを回避
- 予期しない動作を防止

---

### 5. メタデータの充実（SEO・機能説明）

**追加したメタタグ:**

```html
<meta name="keywords" content="CSS, clamp, typography, responsive design, SCSS, mixin, calculator">
<meta name="author" content="Clamp Calculator Contributors">
<meta name="application-name" content="Clamp Calculator">
```

**効果:**
- Google 検索に「clamp calculator」で引っかかるようになる
- GitHub の「About」セクションに説明が表示される
- ブックマーク時にアプリ名が正確に表示される

---

### 6. .gitignore ファイルの追加（バージョン管理の整備）

**何をしたのか:**
`.gitignore` ファイルを作成し、Git リポジトリに含めるべきでないファイルを指定しました。

**指定した除外対象:**

| カテゴリ | ファイル例 | 理由 |
|---|---|---|
| **macOS** | `.DS_Store`, `.AppleDouble` | OS が自動生成する非本質ファイル |
| **IDE** | `.vscode/`, `.idea/`, `*.sublime-*` | 個人のエディタ設定（チーム内で異なる） |
| **ログ** | `*.log` | 実行時の一時ログ（再現不可能） |
| **Node** | `node_modules/`, `dist/`, `build/` | パッケージ（`package-lock.json` で復元可能） |
| **環境** | `.env`, `.env.local` | API キーなどの機密情報 |

**なぜ必要か：**
- チーム開発時に「自分の PC 設定がコミットされた!」というトラブルを防ぐ
- リポジトリサイズを削る（ダウンロードを高速化）
- 誤って機密情報をコミットするのを防ぐ

**新人へのアドバイス:**
> **`.gitignore` は、プロジェクト開始時に必ず設定しましょう。** あとから除外するのは大変です。GitHub が提供する [gitignore テンプレート](https://github.com/github/gitignore) を参考にするのがおすすめです。

---

## 改善前後の比較

### ディレクトリ構造

#### 改善前
```
clamp_calculator/
├── clamp-calculator.html
├── clamp_calculator_development_log.md
└── README.md  (空)
```

#### 改善後
```
clamp_calculator/
├── clamp-calculator.html            ✅ セキュリティメタタグ追加
├── README.md                        ✅ 詳細説明に拡充
├── LICENSE                          ✨ 新規作成（MIT）
├── SECURITY.md                      ✨ 新規作成（ガイドライン）
├── IMPROVEMENTS.md                  ✨ このファイル
├── GUIDELINES_FOR_WEBAPP_DEVELOPMENT.md  ✨ 新規作成（新人向けガイド）
├── clamp_calculator_development_log.md
└── .gitignore                       ✨ 新規作成（重要）
```

---

## 改善のインパクト

### セキュリティ面
| 対応項目 | 改善前 | 改善後 |
|---|---|---|
| **CSP ヘッダー** | なし | ✅ あり（inline スクリプト許可） |
| **XSS 対策** | 暗黙的 | ✅ 明示的 |
| **クリックジャッキング対策** | なし | ✅ あり |
| **ユーザー取説** | なし | ✅ SECURITY.md で明記 |

### 信頼性・透明性
| 観点 | 改善前 | 改善後 |
|---|---|---|
| **ライセンス明記** | 不明確 | ✅ MIT で明確 |
| **利用条件** | 不明確 | ✅ README で説明 |
| **セキュリティポリシー** | ゼロ | ✅ SECURITY.md で公開 |
| **コンプライアンス** | 不明 | ✅ GDPR/HIPAA など記載 |

### 保守性
| 項目 | 改善前 | 改善後 |
|---|---|---|
| **ファイル管理** | 手作業 | ✅ .gitignore で自動化 |
| **ドキュメント** | minimal | ✅ 充実 |
| **新人オンボーディング** | 難しい | ✅ ガイドで簡単 |

---

## 次のステップ（推奨）

### Phase 1: ニアタームで実施推奨
- [ ] **入力値検証の厳格化**: 負の値・極端な値（9999px など）を拒否
- [ ] **イベントハンドラの最新化**: `onclick=""` → `addEventListener()`に移行
- [ ] **エラーメッセージの改善**: ユーザーが問題を理解しやすく

### Phase 2: 将来のリリース
- [ ] **外部デプロイ用 CSP**: `'unsafe-inline'` を削除可能な設計
- [ ] **ユニットテスト**: 計算ロジックの動作確認
- [ ] **多言語対応**: 英語 UI オプション

---

## チェックリスト（次回のプロジェクトで参照）

Webアプリを公開する際は、以下をチェックしましょう：

- [ ] **README.md** - 機能説明、クイックスタート、セキュリティクレーム
- [ ] **LICENSE** - ライセンスファイル（MIT/Apache など）
- [ ] **SECURITY.md** - セキュリティガイドライン
- [ ] **HTML メタタグ** - CSP, X-Frame-Options など最低 4 つ
- [ ] **.gitignore** - OS・IDE・機密ファイルの除外
- [ ] **入力値検証** - isNaN, 境界値チェック
- [ ] **エラーハンドリング** - ユーザーフレンドリーなメッセージ
- [ ] **HTTPS 対応** - 本番公開時は必須

---

## 参考リソース

**セキュリティ関連:**
- [OWASP Top 10](https://owasp.org/www-project-top-ten/) - Web アプリケーション脆弱性トップ 10
- [MDN: Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) - CSP 詳細
- [nginx セキュリティヘッダー設定](https://www.nginx.com/blog/http-security-headers/) - 本番環境例

**ライセンス関連:**
- [Choose a License](https://choosealicense.com/) - ライセンス選択ガイド
- [GitHub: Adding a license to a repository](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/adding-a-license-to-a-repository)

**ドキュメント関連:**
- [README テンプレート](https://github.com/othneildrew/Best-README-Template)
- [Keep a Changelog](https://keepachangelog.com/) - 変更ログの書き方

---

**最終更新**: 2026年2月15日  
**作成者**: Clamp Calculator Security & Documentation Team
