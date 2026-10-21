---
name: "ai-video-camera-work"
description: "AI動画生成のプロンプトにカメラワークを組み込むときに使う。57種のカメラ手法、感情からの逆引き、Seedance/Kling 3.0/Wan 3.0/MiniMax H3別の書き方を提供。"
---

# AI動画 カメラワーク・スキル

AI動画生成のプロンプトに、意図どおりのカメラワークを組み込むためのスキル。主な対象モデルは Seedance / Kling 3.0 / Wan 3.0 / MiniMax H3（Hailuo 3.0）。

## ワークフロー

1. **使うモデルと生成モードを確認する**（テキスト→動画 / 画像→動画 / 参照→動画、尺、単一ショットかマルチショットか）。
2. **シーンの意図を確認する**：何を見せたいか（状況・感情・人物・物）。
3. **手法を選ぶ**：「感情・目的からの逆引き」で候補を絞り、「カタログ」で用語と例文を確認する。
4. **共通テンプレートで組み立て、「モデル別の書き方」でそのモデルの形式に合わせる。**
5. **破綻チェック**：「原則」と「避けるべき組み合わせ」で見直す。
6. 出力は英語プロンプトが基本。必要に応じて日本語で意図を添える。

## プロンプトの原則（全モデル共通）

- **1ショットにカメラの動きは1つ**。動き（カタログ0・A〜C・F）は1つ、構図（D・E）は組み合わせてよい。
- **速度を必ず書く**：very slow / slow / smooth / fast / sudden。
- **方向を明示する**：left / right / up / down / toward / away from / clockwise。
- **始点と終点を書く**：例「slow push-in from a 45° angle, ending in a medium close-up」。
- **被写体の動き・カメラの動き・カットを分けて書く**：「She walks forward. The camera tracks backward at the same speed, maintaining a waist-up medium shot.」
- **雰囲気語（cinematic, epic, dynamic）に頼らない**。具体的なカメラ用語・レンズ・光で書く。
- **固定したいときは明記**：「Fixed shot, camera static, position unchanged.」
- **画像→動画では画像の内容を再描写しない**。言葉は動き・カメラ・音に使う。
- **顔の同一性が重要なショットはミディアム〜クローズアップ**にする（顔が小さいと崩れやすい）。
- **複数ビートはタイムスタンプで区切る**：`[0-3s] ... [3-6s] ...`

### 共通テンプレート

```
[Shot size / angle], [subject + action], [camera movement + direction + speed + end point], [lens / focus], [time effect], [lighting], [audio], [constraints].
```

## モデル別の書き方（2026年10月時点の情報）

### Seedance（2.0 / 2.5）

- 推奨構造：`Subject + Action + Environment + Camera + Style + Constraints`
- カメラ指示は「1つの仕事」。開始点と終了点つきで書く。
- 「cinematic」「moody」などの雰囲気語はほぼ効かない。光の方向・質など物理的に描ける指示にする。
- **複雑なカメラワークは参照動画で転写するのが最も確実**：
  `Use @Video1 only for the camera movement and pacing. Do not transfer the performer, room, or logo.`
- 複数ビートは `[0s-3s]: Extreme wide shot, static camera. [3s-6s]: ...` のタイムライン形式が有効。
- 揺れを避けたい場合は Constraints に `no camera shake`。

### Kling 3.0

- **カメラ指示をプロンプトの先頭に置く**（配置位置が効きやすい）。
- dolly zoom / tracking shot / rack focus などの正式な撮影用語が実撮影に近い挙動をするので、用語を正確に使う。
- 矛盾するカメラ指示を入れるとカメラがガタつく（jitter）。
- **マルチショットはUIで設定する**：Multi-Shot をオンにして Custom Multi-Shot でショットごとの内容と尺を入力する。プロンプトに「Shot 1」と書くだけでは代わりにならない。オフなら単一ショット。
- 尺は3〜15秒。各ショットに「見える動作1つ＋カメラの意図1つ」。
- ネガティブプロンプト対応：`warped faces, jittery camera` など。
- 人物の一貫性は Elements で固定する（形容詞の繰り返しでは固定されない）。

### Wan 3.0（Prime も同じ書き方）

- 公式の構造（不要な項目は省略可）：
  `[全体の説明] + [参照: Image N / Video N / Audio N] + [Shot N (開始-終了秒): 被写体+シーン+動き+レンズ・カメラ] + [台詞] + [効果音/BGM] + [スタイル/ムード] + [ネガティブ]`
- マルチショットは `Shot 1 [0-4s]: ... Hard cut. Shot 2 [4-8s]: ...` の形式。隙間・重なりなく、1ショット2〜5秒目安。トランジションは各ショットの末尾に書く。
- **勝手にカットさせたくない場合は1行目に `One continuous shot` または `Generate single shot`**。
- レンズ指定が効く：`85mm telephoto, shallow depth of field` / `24mm wide-angle, deep focus`。
- スローは `High-speed photography (1000fps slow motion)`。
- 尺は2〜30秒。下書きは5〜8秒で作り、ショットが安定してから本番の長さにする。
- prompt_extend（自動リライト）はデフォルトでオン。タイムスタンプが崩れることがあるので、マルチショットではオン/オフ両方を試す。
- 台詞やBGMが不要なら `No dialogue.` `No BGM.` と明記する（書かないとモデルが勝手に入れる）。

### MiniMax H3（Hailuo 3.0）

- 推奨構造：`References → Retention → Scene → Timeline → Camera → Audio → Constraints`
- **旧Hailuoのブラケット命令（`[Push in]` など）はH3の主な制御方法ではない**。自然言語の撮影用語で書く：`Slowly dolly toward the product while maintaining eye-level framing.`
- 被写体の動き・カメラの動き・カットを明確に分ける。
- タイムラインは `[0-3s] ... [3-6s] ...`。カメラ欄で「Slow dolly-in, then locked close-up. Hard cut at 6 seconds.」のように順序を書く。
- 参照動画には役割を限定して書く：`Video 1 defines the movement timing only. Do not copy its actor, setting, or camera movement.`
- 尺は5〜15秒。
- 参考：旧モデル（Hailuo 02 / 2.3）のAPIでは15種のブラケット命令が使える。`[Truck left]` `[Truck right]` `[Pan left]` `[Pan right]` `[Push in]` `[Pull out]` `[Pedestal up]` `[Pedestal down]` `[Tilt up]` `[Tilt down]` `[Zoom in]` `[Zoom out]` `[Shake]` `[Tracking shot]` `[Static shot]`。1つのカッコ内に並べると同時、文中に順に置くと順番に実行される。

### モデルの使い分けの目安

| やりたいこと | 向いているモデル |
|---|---|
| 既存映像のカメラワークを再現したい | Seedance（@Video1でカメラだけ転写） |
| ショットごとの尺とカメラをUIで厳密に指定したい | Kling 3.0（Custom Multi-Shot） |
| 長回し・最大30秒・台詞入りのマルチショット | Wan 3.0 |
| 人物・商品・動き・音を別々の参照素材から組み合わせたい | MiniMax H3 |

## 感情・目的からの逆引き

| 目的・感情 | 推奨手法 |
|---|---|
| 冒頭で場所を見せる | エスタブリッシング、ドローン空撮、フライオーバー、エクストリームワイド |
| 緊張・気づき・決意 | ドリーイン、クラッシュズーム、ドリーズーム |
| 孤独・余韻・終幕 | ドリーアウト、クレーン上昇、コズミックズーム、ハイアングル |
| 驚き・ショック | クラッシュズーム、ドリーズーム、カメラシェイク、ウィップパン |
| 威厳・英雄感 | ローアングル、ワームズアイ、ペデスタルアップ |
| 不安・狂気・異常 | ダッチアングル、スノリカム、カメラロール、ハンドヘルド |
| 臨場感・ドキュメンタリー | ハンドヘルド、POV、トラッキング |
| 疾走感・アクション | FPVドローン、カーマウント、ウィップパン、スピードランプ |
| 夢・回想・幻想 | フローティングカム、スローモーション、浅い被写界深度 |
| ロマンス・関係性 | オービット、ツーショット、オーバーザショルダー、ラックフォーカス |
| 商品・ディテール | マクロ、インサート、オービット、トラック |
| 謎・正体を明かす | リビール、ラックフォーカス、ティルトアップ、プルバック |
| 時間経過 | タイムラプス、ハイパーラプス |
| 決定的瞬間 | バレットタイム、スローモーション、スピードランプ |
| 場面転換 | ウィップパン、フライスルー、クラッシュズーム |

## カタログ（57種）

### 0. 基本7種

| 手法 | 用語 | 用途・効果 | 例文 |
|---|---|---|---|
| ドリーイン | dolly in, push in | 被写体に迫り、緊張や感情を高める | slow dolly in toward her face as she realizes the truth |
| パン | pan left / right | 固定位置から水平に振り、空間や視線の移動を見せる | slow pan right across the empty office |
| ティルト | tilt up / down | 固定位置から縦に振り、高さや全身を見せる | slow tilt up from his shoes to his face |
| トラッキング | tracking shot | 動く被写体を並走・追従する | smooth tracking shot following her through the crowded street |
| クレーン | crane shot, crane up | 大きく上昇・下降し、壮大さや終幕を出す | crane up from the couple to reveal the whole city at dusk |
| オービット | orbit shot, 360 orbit | 被写体の周りを回り、存在感や関係性を強調する | slow 180-degree orbit around the dancer |
| ドリーズーム | dolly zoom, vertigo effect | 背景だけが伸縮し、衝撃や眩暈を表す | dolly zoom on his face as the background stretches away |

### A. カメラ移動系

| 手法 | 用語 | 用途・効果 | 例文 |
|---|---|---|---|
| ドリーアウト | dolly out, pull back | 離れて状況を明かし、孤独や余韻を出す | slow pull back from the girl to reveal the empty stadium |
| トラック（横移動） | truck left / right | 真横に平行移動し、並んだ対象を見せる | camera trucks right past a row of desks |
| ペデスタル | pedestal up / down | 角度を保って垂直に移動する | camera pedestals up from the table to eye level |
| リードショット | leading shot | 被写体の前を後退しながら撮る（歩きながらの会話） | camera moves backward in front of two friends walking and talking |
| スパイラル | spiral shot, corkscrew rise | 回り込みつつ上昇し、高揚感やクライマックスを出す | camera spirals upward around the hero on the rooftop |
| フライスルー | fly-through | 窓や隙間を通り抜ける（シームレスな移動） | camera flies through the window into the living room |
| フライオーバー | flyover | 上空を滑空し、地理や規模を見せる | smooth flyover above a misty forest |
| ドローン空撮 | aerial drone shot | 高所から広く捉える（冒頭、ロケーション） | aerial drone shot of a coastal town at sunrise |
| FPVドローン | FPV drone shot | 高速の急旋回・急降下でスピード感を出す | fast FPV drone diving down a waterfall |
| リビール | reveal shot | 遮蔽物を外して対象を見せる（サプライズ） | camera slides past a pillar to reveal the bride |
| カーマウント | car-mounted shot | 車両に固定して並走・追走する | car-mounted shot alongside a red sports car on the highway |
| グラウンドレベル | ground-level glide | 地面すれすれを滑り、迫力を出す | ground-level glide toward approaching footsteps |

### B. リグ・手持ち・質感系

| 手法 | 用語 | 用途・効果 | 例文 |
|---|---|---|---|
| ハンドヘルド | handheld camera | 揺れによる臨場感や生っぽさ | handheld camera following a nurse down the hallway |
| ステディカム | steadicam, gimbal shot | 滑らかな長回しの追従 | steadicam shot following the chef through the kitchen |
| スノリカム | snorricam, body-mounted | 人物を固定して背景を揺らす（酩酊・パニック） | snorricam on a panicking man running |
| POV | POV shot, first-person view | 主観視点で没入させる | POV shot opening a door into a dark room |
| カメラシェイク | camera shake | 衝撃・爆発・地震 | heavy camera shake as the building collapses |
| フローティングカム | floating camera, slow drift | 浮遊感（夢・回想） | floating camera slowly drifting through a sunlit room |
| 固定ショット | static shot, locked-off | 動きは被写体だけ（会話・観察） | static shot, locked-off, a woman sits alone on a bench |

### C. レンズ・ズーム・フォーカス系

| 手法 | 用語 | 用途・効果 | 例文 |
|---|---|---|---|
| ズーム | slow zoom in / out | 位置固定で画角を変える（監視・観察感） | slow zoom in on the man in the crowd |
| クラッシュズーム | crash zoom, snap zoom | 急激なズーム（驚き・コミカル） | sudden crash zoom on her shocked face |
| ウィップパン | whip pan | 高速パン＋ブラー（転換・テンポ） | fast whip pan to the door as it bursts open |
| ラックフォーカス | rack focus | ピントを移して注目点を変える | rack focus from the ring in the foreground to her face |
| 浅い被写界深度 | shallow depth of field, bokeh | 背景をぼかし主役を際立たせる | shallow depth of field, creamy bokeh city lights |
| ディープフォーカス | deep focus | 全面にピントを合わせ空間全体を見せる | deep focus, foreground and background both sharp |
| スプリットディオプター | split diopter | 手前と奥の2人を同時に見せる（対立） | split diopter shot, a man close in foreground, rival far behind, both in focus |
| ティルトシフト | tilt-shift, miniature effect | 実景をミニチュア化（可愛さ・俯瞰） | tilt-shift view of a busy intersection |
| 魚眼 | fisheye lens | 歪みでストリート感やMV感 | fisheye lens close to a skateboarder |
| アナモルフィック | anamorphic lens, lens flare | 映画的質感と横長フレア | anamorphic lens, horizontal blue flares |
| マクロ | macro shot | 極小被写体・質感の超接写 | macro shot of a water droplet on a leaf |
| コズミックズーム | cosmic zoom out, earth zoom | 人物から地球規模まで一気に引く | cosmic zoom out from her eye to the city, then to Earth from space |

### D. アングル系（動きと併用可）

| 手法 | 用語 | 用途・効果 | 例文 |
|---|---|---|---|
| ローアングル | low angle | 威圧感・英雄感 | low angle shot of the CEO standing |
| ハイアングル | high angle | 無力感・孤独 | high angle shot of a child alone in the rain |
| ワームズアイ | worm's-eye view | 極端な見上げ（巨大さ） | worm's-eye view of skyscrapers |
| トップダウン | top-down, bird's-eye view | 真上から（配置・料理・群衆） | top-down shot of hands preparing sushi |
| ダッチアングル | dutch angle | 傾きで不安・異常を表す | dutch angle of a man in a dim corridor |
| カメラロール | camera roll, barrel roll | 画面回転（混乱・ダイナミック） | slow camera roll as he loses consciousness |

### E. ショットサイズ・構図系（動きと併用可）

| 手法 | 用語 | 用途・効果 | 例文 |
|---|---|---|---|
| エスタブリッシング | establishing shot | 冒頭で場所・状況を提示 | establishing shot of a Tokyo office tower at night |
| エクストリームワイド | extreme wide shot | 広大さの中の小さな人物 | extreme wide shot, a tiny figure on a vast salt flat |
| ミディアムショット | medium shot | 会話の基本（腰上） | medium shot of two colleagues talking |
| クローズアップ | close-up | 表情と感情 | close-up of her teary eyes |
| エクストリームCU | extreme close-up | 目・唇・指先など細部 | extreme close-up of a finger pressing a button |
| オーバーザショルダー | over-the-shoulder shot | 会話・対峙 | over-the-shoulder shot from the interviewer |
| ツーショット | two-shot | 2人の関係性 | two-shot of a couple sitting side by side |
| インサート | insert shot | 手元・小道具の強調 | insert shot of a letter sliding under the door |

### F. 時間表現系

| 手法 | 用語 | 用途・効果 | 例文 |
|---|---|---|---|
| スローモーション | slow motion | 決定的瞬間・情緒 | slow motion, her hair flowing as she turns |
| スピードランプ | speed ramp | 通常→スロー→通常（アクションの見せ場） | speed ramp as the punch lands, slowing to slow motion |
| タイムラプス | timelapse | 時間経過の圧縮 | timelapse of clouds over the city from day to night |
| ハイパーラプス | hyperlapse | 移動しながらのタイムラプス | hyperlapse moving through Shibuya crossing |
| バレットタイム | bullet time, frozen moment | 時間停止＋回り込み | bullet time, camera orbits as the glass shatters, frozen in midair |

## 組み合わせ例

- **ショートドラマの見せ場（気づき）**：`Close-up, the woman reads a message on her phone, slow dolly in ending on her eyes, shallow depth of field, cold side light, no dialogue.`
- **対立シーン**：`Over-the-shoulder shot, two men face each other in a dim office, static shot, low key lighting.`
- **オープニング**：`Establishing shot, aerial drone shot slowly flying over a seaside town at sunrise, warm golden light from frame left.`
- **エンディング**：`Extreme wide shot, she stands alone on the beach, slow crane up and pull back, overcast soft light.`
- **商品カット**：`Macro shot, a perfume bottle on black glass, slow orbit, rack focus to the logo, soft studio lighting.`
- **Wan 3.0 マルチショット**：`A tense office drama. Shot 1 [0-4s]: Wide shot, static camera, she enters the meeting room. Hard cut. Shot 2 [4-8s]: Close-up, 85mm telephoto, slow push-in as she reads the document. She says: "This can't be right." Lip sync. No BGM.`

## 避けるべき組み合わせ（破綻しやすい）

- 動きの重ね掛け（例：orbit + dolly in + tilt up）は1つに絞る。必要ならショットを分ける。
- ドリーインとズームインを同時に指定すると意図が曖昧になる。どちらか一方にする（両者を逆方向に組むのがドリーズーム）。
- 固定ショットとカメラ移動は矛盾する。
- マクロ／エクストリームCUと大きな移動（FPV、フライオーバー）は破綻しやすい。
- 短い尺（5秒前後）に大きな移動や複数段階の動きを入れない。
- 人物の顔に寄るカメラワークでは、人物の動作を最小限にすると顔が崩れにくい。
- 速すぎる動き（「3秒に3回首を振る」など高頻度の動作）は制御できない。動作は回数ではなく仕草として書く。