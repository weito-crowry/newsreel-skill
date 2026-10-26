# AIニュース動画の脚本化

## 対象と前提

- この手順は、「AIニュース収集」シートで選択済みになったトピックを脚本（production.json）に仕上げ、動画生成キューに載せるまでの作業を定める。

- 制作パッケージは `~/workspace/news-video-package/` にあり、機械的な手順は `tools/scripting.py` が担う。脚本の中身を書くのは Muse で、最終的な選択と読みの判断は Tanaka が行う。

- 脚本の事実は、素材に束ねられた記事本文からのみ構成する。要約やタイトルだけで内容を膨らませない。

- 詳細な作法の全文は [references/scripting-full.md](references/scripting-full.md) を正とし、このスキルと食い違うときは全文側を優先する。

## 手順

- 対象を洗い出す。`python3 tools/scripting.py list` を実行し、選択済みで未脚本化のトピックを確認する。脚本化してよいか曖昧なものは、着手前に Tanaka に確認する。

- 素材を束ねる。`python3 tools/scripting.py materials <topicId>` でメンバー全記事の本文を集め、発表主体、記事の著者、掲載媒体を分けて把握する。

- 器を作る。`python3 tools/scripting.py scaffold <topicId>` で土台を生成する。ここでのタイトルは管理用の仮タイトルであり、様式の標準値（style）は scaffold が書き出す。

- 脚本を書く。全体構成を先に決め、本タイトルは構成の要約として後に付ける。研究・技術系はフックを置かず、タイトルカードに冒頭サマリーの1文目を載せ、直後に3文の冒頭サマリーを置く。

- 読みをチェックする。`python3 tools/scripting.py readings <production.json>` で未登録の固有名詞を洗い出し、生成の前に辞書登録まで済ませる。

- 絵コンテで確認する。`runner.py --timing-only` で timing を作り、`tools/storyboard.py` で絵コンテを出す。初見レビューにかける場合は、一般的な30代会社員男性で IT リテラシーは ChatGPT で質問する程度というペルソナを使う。

- 完了処理をする。`python3 tools/scripting.py finalize <topicId>` でシートを脚本済みに進め、Drive の `scripts/<YYYY-MM-DD_topicId>/` に production.json と script.md を正規のファイル名で置き、準備時間ログに記録する。

- 完了の定義は、production.json と script.md が完成し、読みチェックと辞書登録が済み、finalize の反映まで終わった状態とする。ここまで済めばトピックは worker のキューに自動で載る。

## 脚本の作法（要点）

- 並びはタイトルカード、本文、まとめ、エンドカードの順とし、研究・技術系以外では本文の前にフックを置く。エンドカードは出典のみとし、上部は YouTube のエンドスクリーン用に空ける。

- 本文は冒頭サマリー（研究系）または動画の中心主張の展開として作る。事実には役割を付け、理解に必要か、根拠になるか、限定する場合だけ残し、依存順に並べる。まとめで新しい事実は出さず、まとめのシーンは「まとめです。」から始める。

- 発表主体、記事の著者、掲載媒体を正確に書き分ける。プラットフォームを発表主体にせず、一次ソースの数値は著者側の報告として扱う。

- ナレーションで触れる名詞や主語は、画面のどこかに必ず存在させる。画面がナレーションに先行するのはよいが、主語が画面に無いシーンは作らない。

- 図解のテキストは単語の途中で切らず、文節の切れ目に改行を入れる。箇条書きは表示領域を画面幅いっぱいに取り、行は長く書く。

- 表示タイミングや様式は脚本データが決める。標準の見せ方は「シーン開始から全部出す」で、データの省略時の既定として表す。例外は脚本に atUnit などを書いて表し、プログラムは直さない。

- アクセントテロップは付けない（試行中）。用語解説ブロックはフックと冒頭サマリーには付けず、初見で引っかかる固有名詞にだけ付ける。

- 言い回しはニュースで普通に使う表現にし、英語の直訳にあたる聞き慣れない表現は避ける。

## 読みの登録基準

- 登録するのは読みが固定する固有名詞に限る。一般の英単語を固有名詞として登録しない。部分一致で将来の脚本に誤って効くためである。

- 文脈で読みが変わる漢字はフレーズ単位で登録し、それでも難しい箇所は合成用テキストだけをひらがな化する。表示テキストは変えない。

- 読みが資料で確定しないものは、候補を Tanaka に提示して判断を仰いでから登録する。登録後は readings を再実行し、候補が想定どおり減ったことを確認する。

## 境界

- 選択の最終判断は Tanaka が行う。自動化が担うのは収集、要約、スコア付け、脚本化の準備までであり、選択済みへの決定はしない。

- プログラムの変更は、既存機能ではできない新しい表現の追加とバグ修正のときだけ行う。様式の標準値を変えたいときは、このスキルと scaffold とデータ側で変更する。

- GitHub へのプッシュは Tanaka の承認が都度必要なので、変更はまとめて一度に済ませる。脚本と読み辞書はプログラムではなく Drive で共有する。

## Available connectors

- [Google Sheets](/opt/hatch/connectors/google_sheets/CONNECTOR.md) for reading and updating the news sheet that holds the topic queue and statuses.

- [Google Drive](/opt/hatch/connectors/google_drive/CONNECTOR.md) for storing finished scripts and the shared reading dictionary.

- Use `muse.connector_search` to find other connectors and `muse.skill_search` to find related skills.

- If nothing here fits, look on the web for a maintained API or MCP server and set it up with [Custom Connectors and MCPs](/opt/muse/skills/custom-connector-setup/SKILL.md), or build a custom connector.
