---
name: pdflocss-review
description: Evaluate whether a project's CSS/SCSS (or CSS-in-JS / scoped CSS) architecture follows PDFLOCSS — layer separation (foundation/layout/component/project/utility), MindBEMding naming, single-class-per-element, class-on-every-tag — and score it with actionable fixes. Then, only if asked, edit the source to apply those fixes. Use when the user wants a PDFLOCSS or CSS-design review of a specific PR (by number) or the whole project, or asks to fix/improve CSS to match PDFLOCSS.
---

コードベースのCSSをPDFLOCSSと照合してスコア化し、具体的な修正点を報告する。編集は依頼された時のみ行う。

ルール一覧・採点表: [PDFLOCSS-RULES.md](PDFLOCSS-RULES.md) — Step 3で読み込む(それより前には読まない)。

## 手順

### 1. 対象を特定する

- PR番号が指定された場合(ユーザーから、またはこのskillの引数として) → そのPRのdiffが対象。`gh pr diff <number>`と`gh pr view <number> --json files`で取得する。`docs/agents/issue-tracker.md`があれば、そのPR/issue取得コマンドの部分だけ従う(triage専用の`authorAssociation`除外フィルタ等、triageフロー固有のルールはこのskillには適用しない — ここはユーザーが直接指名したPRのレビューであり、triageの受付可否とは無関係)。
- PR番号が無い場合 → プロジェクト全体のCSSが対象。
- `gh pr diff <number>`が失敗する場合、まず`gh issue view <number>`を試す。issueとして存在すれば「#<number>はissueでありPRではないためレビュー対象なし」と報告して停止する(`docs/agents/issue-tracker.md`にある、issue/PRが番号空間を共有するリポジトリ向けのフォールバックと同じ扱い)。issueとしても存在しない、または`gh`が未認証などの場合は、そのエラーをそのまま報告して停止する — Step 2の「評価対象ファイルが無い」とは別の停止理由として区別する。

### 2. 評価対象ファイルを収集する

CSS設計上の判断を含むファイルを探す: `.css`/`.scss`/`.sass`/`.less`、加えてCSS-in-JS(styled-components/emotionのテンプレートリテラル)やscoped styleブロック(Vue/Svelte SFCの`<style>`、CSS Modulesの`*.module.css`)。PR対象の場合はdiffで変更されたファイルに限定する。

**ディレクトリ構成・ファイル配置はプロジェクトのフレームワークが定める形に従ってよい** — Next.js/Nuxt/Rails等のレイアウトが、PDFLOCSSの`foundation/layout/object/...`という物理ディレクトリ名と一致しないことを理由に減点しない。見るべきは、役割の分離(初期化/構造/再利用部品/ページ固有/微調整)が命名・ファイル分割・フォルダのいずれかで判別できるかどうか。**Astroの場合はPDFLOCSS-RULES.mdの「Astro(弊社独自ルール)」を正とする**(読み込みはStep 3)。

Scoped CSS(CSS Modules・`<style scoped>`・CSS-in-JS)の場合、PDFLOCSSがページ名+セクション名の命名で得ている疑似スコープはフレームワーク側が既に提供している状態にあたる → コンポーネント*内*で適用されるべき事項のみを評価する。ただし**全てのタグにクラスを入れるルールは緩めない**(詳細・実装ルールはPDFLOCSS-RULES.mdを読み込んだ後の「適用範囲の調整」セクション参照)。

**評価できる対象ファイルが無い場合**(プロジェクト全体を走査しても該当なし、またはPRがCSSを含むファイルに触れていない) → ここで停止する。何も評価対象が無いこととその理由を平易に報告する(例: 「対象PRにCSS変更なし」「CSS/SCSSファイルが見つからない」)。採点にも進まず、PRへのコメントも行わない。

### 3. 評価する

[PDFLOCSS-RULES.md](PDFLOCSS-RULES.md)をここで読み込む。収集した各ファイルを、そのスコープ(ファイル全体のCSSか、scoped component内のCSSか — 適用範囲の注記を参照)に該当する全ルールカテゴリと照合する。違反ごとに以下を記録する: file:line、該当コードの短い引用、違反しているルール、重大度(must-fix / should-fix / suggestion。ルールファイルの重大度指針に従う)。

ルールファイルの採点表に従いカテゴリ別スコアと合計(0〜100)を算出する。該当プロジェクトのCSSアプローチ上N/Aのカテゴリがあれば、それを除いて再スケールする。

対象ファイルが全て同じスコープ(全てフルCSS、または全てscoped CSS)なら合計は1本でよい。**フルCSSとscoped CSSが混在する場合は無理に1本の数値に統合せず、「フルCSS対象: n点」「scoped CSS対象: m点」と2本に分けて報告する**(異なる採点表を適用した数値を平均・合算すると意味を持たない数値になるため)。

### 4. 報告する

- **PR対象の場合** → 報告本文をファイルに書き出し、`gh pr comment <number> --body-file <ファイル>`で投稿する(引用するCSSセレクタにはバッククォート・`$`・引用符が含まれ得るため、シェルのインライン`--body`文字列には直接埋め込まない)。構成: 合計スコア → カテゴリごとのサブスコアを1行ずつ → 重大度別(must-fixを先頭に)にグループ化した問題一覧、各項目にfile:lineと1行の修正案を添える。
- **プロジェクト全体対象の場合** → 同じ構成をこのセッション内にそのまま出力する(ファイルは作成せず、コメントも投稿しない) — 会話ログから確認する前提。

スコアだけ、問題一覧だけでは半分の仕事にしかならない。両方揃って初めて実用的になる。

### 5. 改善する(修正を依頼された時のみ)

依頼されていない限りここは実施しない — ユーザーが別途「修正して」と依頼しない限り、手順1〜4が成果物の全てとなる。

依頼された場合: must-fixから着手し、次にshould-fixに取り組む。修正はルールファイルが示す正しいPDFLOCSSパターンに沿って行う(単に違反箇所を削除するだけでなく、正しい接頭辞・正しいセレクタ・正しいレイヤーへと置き換える)。suggestionはユーザーが含めるよう指示しない限り対象外とする。

編集後、変更したファイルに対してStep 3の評価を再実行し、修正前後のスコアと、どの問題を修正/スキップしたか(スキップした場合はその理由 — 例えばフレームワーク側の変更が必要でスコープ外、など)を報告する。報告先はStep 4と同じ基準で分ける: PR対象なら追加の`gh pr comment`(Step 4のコメントへの返信・追記であることが分かるようタイトルに「修正後」等を明示)、プロジェクト全体対象ならこのセッション内に出力する。
