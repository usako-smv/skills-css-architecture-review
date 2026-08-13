# Issueトラッカー: GitHub

このリポジトリのissueや仕様書はGitHub issuesとして管理します。すべての操作に`gh` CLIを使用してください。

## 利用方法

- **issueの作成**: `gh issue create --title "..." --body "..."`。複数行の本文にはheredocを使用してください。
- **issueの読み取り**: `gh issue view <number> --comments`。コメントは`jq`でフィルタし、ラベルも合わせて取得してください。
- **issue一覧の取得**: `gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`。必要に応じて`--label`や`--state`で絞り込んでください。
- **issueへのコメント**: `gh issue comment <number> --body "..."`
- **ラベルの付与・削除**: `gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **クローズ**: `gh issue close <number> --comment "..."`

リポジトリは`git remote -v`から推測されます。クローン内で実行すれば`gh`が自動的に判別します。

## triage対象としてのプルリクエスト

**外部PRをリクエストとして扱う: しない。**（このリポジトリで外部PRを機能要望として扱う場合は`yes`に設定してください。`/triage`がこのフラグを参照します。）

`yes`に設定した場合、PRはissueと同じラベル・状態を使い、対応する`gh pr`コマンドで扱います。

- **PRの読み取り**: `gh pr view <number> --comments`、差分は`gh pr diff <number>`
- **triage対象の外部PR一覧**: `gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments`を実行し、`authorAssociation`が`CONTRIBUTOR`・`FIRST_TIME_CONTRIBUTOR`・`NONE`のものだけ残す(`OWNER`/`MEMBER`/`COLLABORATOR`は除外)。
- **コメント・ラベル・クローズ**: `gh pr comment`、`gh pr edit --add-label`/`--remove-label`、`gh pr close`

GitHubはissueとPRで番号空間を共有しているため、`#42`のような表記はどちらか判別できません。`gh pr view 42`を試し、失敗したら`gh issue view 42`にフォールバックしてください。

## スキルが「issueトラッカーに公開」と言った場合

GitHub issueを作成する。

## スキルが「関連チケットを取得」と言った場合

`gh issue view <number> --comments`を実行する。

## Wayfinding操作

`/wayfinder`が使用。**map**は1つのissueで、**child**issueがチケットに対応します。

- **map**: `wayfinder:map`ラベルの付いた単一issue。Notes / Decisions-so-far / Fogの本文を持つ。`gh issue create --label wayfinder:map`で作成。
- **child ticket**: mapとGitHub sub-issueとして紐付いたissue(`gh api`のsub-issuesエンドポイント使用)。sub-issue機能が無効な環境ではmap本文のタスクリストに追加し、child本文冒頭に`Part of #<map>`を記載。ラベルは`wayfinder:<type>`(`research`/`prototype`/`grilling`/`task`)。着手時は担当開発者をアサイン。
- **ブロッキング**: GitHubの**native issue dependencies**が正式表現(UI上でも確認可)。`gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`でエッジ追加。`<blocker-db-id>`はブロッカーの数値**database id**(`gh api repos/<owner>/<repo>/issues/<n> --jq .id`で取得。`#number`や`node_id`ではない点に注意)。GitHubは`issue_dependencies_summary.blocked_by`を報告(オープン中のブロッカーのみ→現在有効なゲート)。dependencies機能が使えない場合はchild本文冒頭に`Blocked by: #<n>, #<n>`と記載してフォールバック。全ブロッカーがクローズされればブロック解除。
- **frontierクエリ**: mapのオープン中child一覧を取得(`gh issue list --state open`、mapのsub-issue/タスクリストに限定)。オープン中ブロッカーがあるもの(`issue_dependencies_summary.blocked_by > 0`、または`Blocked by`行に未クローズissueがある場合)・アサイン済みのものは除外。map順で先頭が対象。
- **claim**: `gh issue edit <n> --add-assignee @me` — セッション最初の書き込み。
- **resolve**: `gh issue comment <n> --body "<answer>"` → `gh issue close <n>` → mapのDecisions-so-farにコンテキストへのポインタ(gist + リンク)を追記。
