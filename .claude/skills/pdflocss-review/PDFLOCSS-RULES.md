# PDFLOCSS ルールリファレンス

出典: [明日から使えるCSS設計【PDFLOCSS】](https://zenn.dev/wagashi_osushi/books/94efd21a66ccaa)（wagashi氏、FLOCSS拡張）。評価ステップで参照する採点基準・チェックリスト。

## 全体思想

- **シングルクラス設計**: 1要素1クラスにほぼ全スタイルを書く。Modifierで2クラス以上になるのはok。
- **クラスセレクタ以外は原則使わない**: ID・タグセレクタ・インラインスタイル禁止。`p.text {}`のようなタグ+クラスの組合せも禁止(そのタグに依存してしまうため)。例外: `<img>`/`<iframe>`など内容非依存の要素を`<div class="wrapper">`で囲んで`.wrapper img {}`とするのは許容。
- **全てのタグにクラスを入れる**: スタイルが無くても入れる。理由は(a)HTML構造変更時にクラス名が不自然にならない、(b)クラス名で役割を明示できるため。`<img>`/`<iframe>`など明示が重要でない要素は例外。
- **要素はモジュールでなくページとセクションで分割**(PDFLOCSS独自): 本家FLOCSSの「component/projectどっちか問題」を、ページ名+セクション名の疑似スコープで解消。
- 詳細度は全クラスセレクタで統一 → 上書きは「後に読み込まれる方が勝つ」順序で制御(layout→component→project→utility)。

## 命名規則(共通)

- MindBEMding: `.Block__Element--Modifier`。Modifierはハイフン**2つ**。
- Elementの2階層目(`.Block__Element__Element`)は非推奨。1階層に留める。
- 単語はハイフンでつなぐ(ケバブケース)。キャメルケースでも良いが**混在は不可**。
- 連番はゼロパディング2桁(`01`,`02`...)。**連番の前にハイフンを入れない**(`item01`は○、`item-01`は✕)。ただし値そのものを表す数字(例: `opacity-06`)はハイフンを入れる。
- レイヤー接頭辞は本家FLOCSSと同じ: `l-`(layout) / `c-`(component) / `p-`(project) / `u-`(utility)。

## レイヤー別ルール

### Foundation(接頭辞なし)
初期化・変数・ベーススタイル。ファイル分割の目安: `_color.scss`(色変数) / `_variable.scss`(色以外の変数) / `_font.scss`(フォント定義) / `_mixin.scss`(`@mixin`とメディアクエリ) / `_function.scss`(`@function`) / `_reset.scss`(リセットCSSそのまま、**自分で編集しない**) / `_base.scss`(リセットしきれなかった分+ベーススタイル)。
- 色変数はできるだけセマンティックな命名(`$c-primary`)を優先し、無理なら色名(`$c-gray`)にフォールバック。
- ブレイクポイントは`min-width`/`max-width`どちらかに**統一**(混在禁止)。

### Layout(`l-`)
- クラス名: `.l-[ブロック名]`。ファイル名: `_[ブロック名].scss`。
- 対象基準: **複数ページ(または全ページ)に共通して現れる大きい要素**(header/footer/main/sidebar等)。ほぼ全ページに出るが小さい要素(パンくずリスト等)はcomponentへ。
- 内側の要素まで**全て`.l-*`で書く**(外側だけlayout、中身はproject、という分割は不可)。
- 量産ページ/テンプレートページの**共通スタイル**はlayoutに置き、命名は`.l-[量産ページ名]-[セクション名]`(projectと同じ構造)。固有スタイルはprojectへ。

### Component(`c-`)
- クラス名: `.c-[コンポーネント名]` / `.c-[コンポーネント名]-[ユニーク名]`(bottun-primary等、種類が複数ある場合) / `.c-[コンポーネント名]+連番`。ファイル名: `_[コンポーネント名].scss`(ユニーク名は含めない)。
- 対象基準: **複数ページで使われる共通モジュールか**。かつ **rule of three**(3回出てきてから化)。「スタイルが同じ」だけでは化しない — 役割も同じであること。
- **独立したモジュールとして定義**: 特定タグ・特定の親クラスに依存する書き方(`a.c-button`、`.contact-button .c-button`)は禁止。クラスセレクタ単体・ネスト無しで定義。
- BEMの階層関係(`.c-breadcrumbs__item`)は可。ただし必ず`.c-[コンポーネント名]`で全体を囲むこと(Blockなしでいきなり`__Element`は不可)。
- **marginを持たせない**(理由: 使い回すたびに余白が邪魔になり再利用性が下がるため)。階層を持つcomponentの`__Element`部分がmarginを持つのは許容。違反はこのカテゴリ(Componentの独立性)でのみ計上する — Project節にも関連記述があるが、あれは「余白はproject側で取る」という対処法の説明であり、別の違反ではない。
- 固定`width`/`height`はできるだけ持たせない(必要なら`min-width`/`min-height`)。
- 共通スタイル(最小)のみをcomponent本体に持たせ、差分はModifierで拡張。
- **上書きの優先順位**(この順で検討): ①componentにprojectクラスを追加してそこで上書き ②componentにModifierを用意 ③utilityクラスで上書き。**projectからcomponentのスタイルを直接上書きする(`.p-profile > .c-media__image`のような子孫セレクタでの上書き)のはPDFLOCSSでは禁止**(本家FLOCSSは許容するが、詳細度が上がるため)。

### Project(`p-`)
- クラス名: `.p-[ページ名]-[セクション名]`。ファイル名: `_[ページ名].scss`。**モジュール単位ではなくページ×セクション単位で分割**するのがPDFLOCSSの核。
- 各セクションの最上位クラスは`.p-[ページ名]-[セクション名]`自体(Element/Modifier無し)。その配下は`__Element`。
- **セクションをまたいで別セクションのクラスを流用するのは禁止**(スタイルが同じでも別クラスとして定義しコピペ、または`@extend`)。
- ハイフンでBEMのネストを深くしない(`&__prof { &-text {} }`のような入れ子は不可。`&__prof-text`とフラットに)。
- コンポーネントの余白はproject側のクラスで取る(Component節の「marginを持たせない」の対処法。同じ違反をここでも二重に計上しない)。
- ページ内で複数セクションにまたがる共通スタイルは、**プレースホルダー(`%p-[ページ名]-[モジュール名]`、Element/Modifier無し、ファイル先頭に定義)を`@extend`**してグループ化。rule of threeと最小スタイルの原則も適用。
  - **クラスそのものを`@extend`するのは禁止**(プレースホルダー以外への`@extend`)。判別できない暗黙のスタイル共有を防ぐため。
- JS連携: 「JS連携(弊社独自ルール)」セクション参照(PDFLOCSS本来の`.is-○○`状態classルールは弊社では採用しない)。
- **タグセレクタは影響範囲が広くても使わない**(`.p-xxx p {}`のような子孫タグセレクタは禁止)。スタイルが同じ要素には同じクラスを付与する。
- `@keyframes`はクラスセレクタ内にそのまま書いてよい。

### Utility(`u-`)
- クラス名: `.u-[Emmetのプロパティ+値]`(例: `u-mt10`, `u-w50p`) / `.u-[同]-[ブレイクポイント]-[min|max]`(例: `u-dn-md-max`)。ファイル名: `_utility.scss`または`_[プロパティ名].scss`。
- 値を変えたらクラス名も追従して変える(`u-mt10`の値を15pxにしたら`u-mt15`にリネーム)。値とクラス名の乖離は不可。
- **むやみに使わない**: 「1箇所だけ他と違う」かつ「component/projectのModifierで拡張するのが不適切」な場合のみ。
- **`!important`は入れない**(utilityクラスも含め全面禁止)。CSSの読み込み順(layout→component→project→utility)で詳細度統一のままutilityが最終的に勝つ設計のため不要。

### Lib
外部ライブラリCSSをそのまま配置。任意。

### style.scss(エントリ)
foundation→layout→object(component→project→utility)→libの順で`@import`。foundation内は変数定義→利用の順序に注意(font/color/variable/function/mixin→reset→base)。globでの一括importは`foundation/`以外に使える。

## JS連携(弊社独自ルール)

出典: [website-starter-kit — JSの記述規則](https://github.com/sonicmoov/website-starter-kit#js%E3%81%AE%E8%A8%98%E8%BF%B0%E8%A6%8F%E5%89%87)。PDFLOCSS本来の`.is-○○`状態classルールを置き換える弊社標準。評価はこちらを正とする。

- **起点クラス**: JS操作でHTMLに付与するclassは`js-[ブロック](__[エレメント] ※任意)`(例: `js-modal`, `js-tab__panel`)。スタイルとは別にJS的機能が紐づくことを明示する接頭辞。**`.js-○○`にスタイルを入れない**(ここはPDFLOCSS本来のルールと同じ)。
- **状態操作**: `.is-○○`のようなclass付与ではなく、以下の優先順位で操作する: ①HTML由来の属性(`disabled`等) → ②ARIA属性(`aria-expanded`等) → ③カスタムデータ属性(`data-state`等)。状態に応じたスタイルはこれらの属性セレクタ(例: `[aria-expanded="true"]`)に対して書く。
- **共通スタイルの置き場所**: foundation/layout/objectを分離したフルCSS構成では、サイト/ページ横断で使う処理前後の共通スタイルは`foundation/_mixin.scss`(または`_mixin-js.scss`)に`@mixin`定義し各セレクタで`@include`する(PDFLOCSS本来のルール通り)。**Next.js/Nuxt/Rails/Astro等のコンポーネント指向フレームワークでは、foundationへの集約は必須とせずコンポーネント内にスタイルを定義してよい**(コンポーネントが再利用の単位であり、集約するとかえって見通しが悪くなるため)。

## 適用範囲の調整(重要)

- **ディレクトリ構成はフレームワーク準拠でよい**: Next.js/Nuxt/Rails等の規約とPDFLOCSSの`foundation/layout/object/...`ディレクトリ名が衝突する場合、物理ディレクトリ名の一致は問わない。判定するのは「役割の分離(初期化/構造/再利用部品/ページ固有/微調整)が命名やファイル構成のどこかで区別できているか」。Astroの場合は下記「Astro(弊社独自ルール)」を正とする。
- **Scoped CSS(CSS Modules, Vue/Svelte `<style scoped>`, styled-components等)はコンポーネント内で完結**: ページ横断的なスコープ管理はフレームワークが肩代わりしているため、下表の8カテゴリのうち**ディレクトリ/ファイル構成**と**レイヤー原則の遵守**の2カテゴリ(計25点)はN/Aとし、他の6カテゴリ(計75点)は通常通り適用する。
  - 適用する6カテゴリの読み替え: 命名規則の一貫性(レイヤー接頭辞`l-`/`c-`/`p-`/`u-`はコンポーネント単体でも付与する — 単一コンポーネントの大半は再利用パーツなら`c-`、レイアウト用なら`l-`に該当。BEMのElement/Modifier表記・ケバブケース・連番規則も通常通り適用) / **全タグへのクラス付与(緩めない。スコープに関係なく必須)** / セレクタ規律 / Componentの独立性(margin・固定幅高さ回避はコンポーネント内でも同じ理由で有効) / JS連携規約(「JS連携(弊社独自ルール)」参照。スコープに関係なく適用) / タブー事項(`!important`不使用等はスコープに関係なく適用)。
  - **実装方法**: スタイルを追加する場合は`<style scoped>`内にクラスを宣言して実装する(インラインstyle属性や場当たり的な追加は不可)。Tailwindを導入している場合は、タグにユーティリティクラスを直接大量付与するのではなく、`<style scoped>`内で宣言したクラスに対し`@apply`でユーティリティを合成して実装する(シングルクラス設計を保つため)。
- N/Aにした2カテゴリは採点対象から除外し、残り75点を100点に比例配分で再スケールする(粒度はカテゴリ単位に固定し、実行者による判断のブレを無くす)。

### Astro(弊社独自ルール)

出典: [website-starter-kit](https://github.com/sonicmoov/website-starter-kit)。

- ディレクトリ構成は[Astro公式のディレクトリ構成](https://docs.astro.build/ja/basics/project-structure/)に準拠する(PDFLOCSSの`foundation/layout/object/...`という物理ディレクトリを新設しない)。
- 以下はPDFLOCSSから意図的に外れている割当てであり、減点対象にしない:
  - `src/components/component/`・`src/components/layout/` = PDFLOCSSのComponent・Layoutに相当。
  - `src/layouts/` = ページ全体を包むBaseLayout的なもの。PDFLOCSSのLayoutレイヤーとは別概念なので混同しない(命名が同じ「layout」でも役割が違う)。
  - `src/styles/foundation/tokens/` = デザインシステムのトークン置き場。
  - Layout・ComponentのCSS実体は`src/styles/layout/`や`src/styles/object/component/`に集約するのではなく、`src/components/`配下の各コンポーネントディレクトリに内包する想定。集約用の`index.scss`でまとめて`@import`する。

## 採点基準(フル評価、100点満点)

| カテゴリ | 配点 | チェック内容 |
|---|---|---|
| ディレクトリ/ファイル構成 | 10 | 役割別分離ができているか(フレームワーク準拠可) |
| 命名規則の一貫性 | 20 | 接頭辞・MindBEMding・連番ゼロパディング・ケバブ統一 |
| 全タグへのクラス付与 | 15 | スタイル無しでも役割を示すクラスがあるか |
| セレクタ規律 | 20 | クラスセレクタのみ・ID/タグセレクタ不使用・詳細度統一 |
| レイヤー原則の遵守 | 15 | 各層の役割分離、component/project境界の混同がないか |
| Componentの独立性 | 10 | margin無し・固定幅高さ回避・Modifier拡張・rule of three |
| JS連携規約 | 5 | `.js-○○`命名・`.js-○○`にスタイルを入れていないか、状態操作がclassでなくHTML/ARIA/データ属性の優先順位に従っているか(「JS連携(弊社独自ルール)」参照) |
| タブー事項 | 5 | `!important`不使用、クラス直接`@extend`禁止、projectからcomponentの子孫セレクタ上書き禁止(Component節参照) |

N/Aカテゴリがある場合はそのカテゴリを除外し、残り配点の合計を100になるよう比例配分する。

## 重大度の目安

- **must-fix**: クラスセレクタ以外の使用、`!important`、component/projectの子孫セレクタ上書き、タグセレクタでのスタイリングなど、原則を直接破るもの。
- **should-fix**: 命名規則の逸脱(連番のハイフン、Element2階層化等)、rule of three違反のコンポーネント化/コピペ判断ミス、状態操作を`.is-○○`のようなclassで行っている(「JS連携(弊社独自ルール)」の属性優先順位に従っていない)。
- **suggestion**: セマンティックでない変数名、utilityの乱用気味な使用など、改善の余地はあるが原則違反ではないもの。
