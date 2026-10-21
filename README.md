# AI動画 カメラワーク図鑑・モーショングラフィックス図鑑

AI動画を作るときに使う、カメラワーク（57種）とモーショングラフィックス（47種）の図鑑と、
その中身を Claude に覚えさせるスキルです。無料で使えます。

- 図鑑を見る: https://ai-video-zukan.dhack-ai.workers.dev/
- 作り方と使い方の解説: https://note.com/kazuya_dhack_ai

## 中身

| ファイル | 何か |
|---|---|
| `camera-work.html` | カメラワーク図鑑 |
| `motion-graphics.html` | モーショングラフィックス図鑑 |
| `download/ai-video-camera-work.zip` | カメラワークのスキル。Seedance / Kling 3.0 / Wan 3.0 / MiniMax H3 ごとにプロンプトの書き方を変える |
| `download/video-motion-graphics.zip` | モーショングラフィックスのスキル。テロップ・タイトル・トランジションの手法と、Remotion / FFmpeg での実装 |
| `skills/` | ZIP の中身と同じもの（GitHub 上で読むため） |

## スキルの入れ方

**claude.ai**: 設定のスキルの画面から、`download/` の ZIP をそのままアップロードする。

**Claude Code**: `skills/` の中のフォルダを `~/.claude/skills/` に置く。

入れたあとは、チャットで「Seedance で、カフェで振り向く女性を10秒。緊張感のあるカメラワークで」のように頼むと、
スキルが手法を選んで、そのモデルの書き方で英語のプロンプトを組みます。

## 使ってよい範囲

個人・仕事どちらの動画づくりにも自由に使ってください。
再配布するときは、このページへのリンクを添えてください。

モデルの書き方は 2026年10月時点の情報です。モデルの更新で効き方が変わることがあります。
