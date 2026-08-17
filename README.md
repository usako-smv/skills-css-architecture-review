# skills-css-architecture-review

PDFLOCSSに準拠したCSS/SCSS設計を評価・改善するClaude Codeスキル `pdflocss-review` を提供するリポジトリ。

## 使い方

### 評価

- **特定PRを評価する場合**: レビュー対象のPR番号を伝えて呼び出す(例: 「PR #12をpdflocss-reviewでレビューして」)。結果はそのPRへコメントとして投稿される。
- **プロジェクト全体を評価する場合**: PR番号を指定せず呼び出す。評価前にissueが作成され、結果はそのissueへコメントとして投稿される。
- CSS/SCSS/Sass/Less、CSS-in-JS、scoped styleなど評価できるファイルが見つからない場合は、評価を実行せずその旨を報告する。

評価結果には、100点満点のスコア・カテゴリ別内訳・重大度別(must-fix / should-fix / suggestion)の問題一覧が含まれる。

### 改善

評価後に「直して」のように修正を依頼すると、must-fix→should-fixの順に該当ファイルを編集し、修正前後のスコアを再報告する。**明示的に依頼しない限りコードは編集されない。**

## 前提

- GitHub CLI(`gh`)が認証済みであること
- レビュー対象リポジトリにissue/PRを作成・コメントする権限があること

## ルールの中身

評価基準は[`.claude/skills/pdflocss-review/references/PDFLOCSS-RULES.md`](.claude/skills/pdflocss-review/references/PDFLOCSS-RULES.md)に定義されている。ベースは[PDFLOCSS](https://zenn.dev/wagashi_osushi/books/94efd21a66ccaa)(wagashi氏、FLOCSS拡張)。以下は弊社独自の上書き・追加ルール:

- 単語連結はケバブケースのみ(キャメルケース不可)
- 連番は2番目から付与、ゼロパディング無し(`item`、`item2`、`item3`...)
- Foundationの変数はSassの`$variable`でなく、`tokens`ディレクトリのCSS変数(カスタムプロパティ)に集約
- JS連携の状態操作は`.is-○○`classでなく、HTML属性→ARIA属性→データ属性の優先順位で行う([website-starter-kit](https://github.com/sonicmoov/website-starter-kit)準拠)
- ディレクトリ構成・ファイル名はwebsite-starter-kit準拠プロジェクトのみPDFLOCSS本来の規則を厳密適用し、それ以外(Next.js/Nuxt/Rails/Astro単体等)はフレームワーク準拠でよい
- Astro + website-starter-kitの場合は専用のディレクトリ構成ルールがある

## スキルの構成

```
.claude/skills/pdflocss-review/
├── SKILL.md                     # 評価・改善の手順
└── references/
    └── PDFLOCSS-RULES.md         # 採点基準・詳細ルール
```

[Agent Skills](https://github.com/agentskills/agentskills)形式に準拠。
