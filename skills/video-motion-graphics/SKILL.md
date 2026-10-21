---
name: "video-motion-graphics"
description: "ショートドラマ等のAI生成動画にテロップ・タイトル・画像・トランジションを重ねるときに使う。手法カタログ、縦型の安全域と文字量、Remotion/FFmpeg実装、生成モデルへの文字指示を提供。"
---

# 動画モーショングラフィックス・スキル

AIで生成したカットに、テロップ・タイトル・画像・トランジションなどを重ねて仕上げるためのスキル。
主な用途は縦型（9:16）ショートドラマ。カメラワーク自体は `ai-video-camera-work` スキルを使う。

## 進め方

1. **どこで重ねるかを決める**（下の「3つの重ね方」）。指定がなければコード合成（Remotion）を既定にする。
2. **演出の目的から手法を選ぶ**（「目的からの逆引き」→「手法カタログ」）。
3. **縦型の基本ルールを当てる**：安全域・文字量・表示時間・フォント。
4. **実装する**：Remotion か FFmpeg の節を参照。テロップは必ずデータ（JSON）と部品に分け、作品ごとにデータだけ差し替える。
5. **最後にチェックリストで検証**し、満たせない項目は理由を添えて報告する。

## 3つの重ね方

| 方式 | 向いているもの | 注意 |
|---|---|---|
| **コードで合成（Remotion）** | 量産、テロップの型を使い回す、Webアプリから自動で書き出す | React。Vercel上でも書き出せる（後述） |
| **コードで合成（FFmpeg）** | 字幕の焼き込み、ロゴや局IDの重ね、単純な結合 | 凝った動きは不向き |
| **編集ソフト**（CapCut、Premiere、After Effects） | 1本ずつ手で詰める作品 | 手法名と数値（秒数・イーズ）はこのスキルのものがそのまま使える |
| **生成モデルに描かせる** | 看板・画面内の小物など「世界の中にある文字」 | 正確さが要る文字には使わない |

**原則：正確さが必要な文字は全部後乗せにする。** タイトル、役名、台詞の字幕、局ID、ロゴ、CTA、数字は生成させず、合成で重ねる。後乗せする前提で動画を生成するときは、生成プロンプトに必ず文字の禁止を入れる（「生成モデルとの併用」参照）。

## 縦型ショートの基本ルール

### 画面と安全域（1080×1920）

TikTok・Reels・Shorts 共通で使える控えめな余白（公式仕様ではなく実務上の目安。最終的に各アプリのプレビューで確認する）：

| 辺 | 割合 | px |
|---|---|---|
| 上 | 約15% | 約288px |
| 下 | 25〜35% | 480〜672px |
| 左右 | 10〜15% | 108〜162px（右はいいね等のボタンがあるので広めに） |

- 字幕・会話テロップは**中央より下、下端の安全域より上**（目安 y=1150〜1400px）。
- 顔・手・商品にかかるなら上に逃がす。最も長い2行テロップで確認する。
- タイトル・局ID・ロゴは上部の安全域の内側。

### 文字量と表示時間（日本語）

| 項目 | 目安 |
|---|---|
| 1行の文字数 | 13〜15字（縦型の強調テロップは8〜10字） |
| 行数 | 最大2行 |
| 読む速さ | 1秒あたり4〜6字 |
| 1枚の表示時間 | 1行で3〜5秒。短い強調テロップでも最低0.8秒 |
| 改行位置 | 文節の切れ目。助詞だけを次の行に送らない |

表示時間の計算：`秒 = max(0.8, 文字数 / 5)`。台詞の字幕は音声の始まりと終わりに合わせ、音声より短くしない。

### 文字の見た目

- フォントは作品で**2種類まで**（本文用ゴシック＋演出用ディスプレイ書体）。商用利用可のライセンスを確認する。
- 文字サイズは1080幅で**会話56〜72px、強調90〜140px**。太字（700以上）。
- 背景に負けないよう、**縁取り（6〜10px）か座布団（半透明の帯）**を付ける。影だけでは明るい背景で読めない。
- 話者ごとに色を固定する場合も、色だけに頼らず位置や名前でも区別できるようにする。

## 目的からの逆引き

| 目的 | 推奨手法 |
|---|---|
| 冒頭2秒で止める（フック） | 強調テロップ（ポップ）、ズームパンチ、フラッシュ |
| 登場人物を紹介する | フリーズフレーム＋人物紹介、ロワーサード |
| 心の声・本音 | 心の声テロップ（斐体・吹き出し）、タイプライター |
| 衝撃・驚き | 画面揺れ、ズームパンチ、オノマトペ、ホワイトフラッシュ |
| 回想・過去 | 回想ルック（彩度落とし・ビネット）、クロスディゾルブ |
| 場面・時間が変わる | 章タイトル（アイキャッチ）、ブラックフェード、場所・時刻テロップ |
| スマホのやり取りを見せる | チャット画面風オーバーレイ、ピクチャーインピクチャー |
| 対比・同時進行 | 分割画面 |
| テンポを上げる | ビート同期カット、ウィップトランジション、スライド |
| 続きを見たくさせる（引き） | フリーズ＋「…」テロップ、次回予告カード、エンドカード |
| 作品の身元を示す | タイトルカード、局ID・ウォーターマーク |

## 手法カタログ

時間は30fps前提の目安。イーズは「入場 ease-out／退場 ease-in」が基本。

### A. テロップ・文字

| 手法 | 英語 | 内容・用途 | 動きの目安 |
|---|---|---|---|
| 会話テロップ（字幕） | subtitles, captions | 台詞を文字で出す。無音視聴対策 | フェード6〜8f、または切り替えのみ |
| 強調テロップ | emphasis text, pop-up text | キーワードを大きく出す | scale 0.6→1.08→1（spring、10〜14f） |
| キネティックタイポグラフィ | kinetic typography | 文字そのものを動かして感情を表す | 語ごとにstagger 2〜4f |
| 1文字ずつポップ | per-character pop | 可愛い・弾む印象 | 文字ごとに2fずらす、上に8px→元位置 |
| タイプライター | typewriter | 心の声、メッセージ入力、緊張 | 1文字あたり2〜3f、カーソル点滅 |
| ワードハイライト | karaoke captions, word highlight | 話している語だけ色を変える | 音声の単語タイミングに同期 |
| 心の声テロップ | inner voice caption | 口に出さない本音。通常テロップと見た目を変える | ゆっくりフェード＋わずかに浮遊 |
| ロワーサード | lower third | 名前・役職・関係性の帯 | 帯が横から伸び→文字フェード。入3秒表示 |
| 人物紹介（フリーズ） | freeze-frame intro | 止めた画面に名前＋一言説明 | フリーズ＋彩度落とし、文字スライドイン |
| タイトルカード | title card | 作品名・話数 | 入場は速く（10f）、1.5〜2秒保持 |
| 章タイトル（アイキャッチ） | chapter title, eyecatch | 場面の区切り | 画面暗転→文字1秒→次の場面 |
| 場所・時刻テロップ | location / time stamp | 「3年前」「深夜2時」 | 小さく左上、タイプライターかフェード |
| オノマトペ | onomatopoeia text | 「ドンッ」「ザワ…」 | 強いスケールパンチ＋揺れ、SEと同時 |
| 数字カウントアップ | number count-up | 金額・日数・順位 | 0.6〜1.2秒 ease-out、等幅数字 |
| エンドカード | end card, CTA | フォロー・次回への誘導 | 最後の2〜3秒、安全域の内側 |

### B. 画像・素材の合成

| 手法 | 英語 | 内容・用途 | 動きの目安 |
|---|---|---|---|
| オーバーレイ | overlay | 画像・動画を上に重ねる | フェードかスケールイン 8〜12f |
| ピクチャーインピクチャー | picture-in-picture | 小窓で別映像。電話の相手、回想 | 角丸＋影、スライドイン |
| 写真のスローズーム | Ken Burns effect | 静止画に動きを付ける | 3〜5秒で scale 1→1.08＋わずかな移動 |
| チャット画面風 | chat UI overlay | メッセージのやり取りを見せる | 吹き出しが下から順に出る、既読表示 |
| キーイング | chroma key, background removal | 人物を切り抜いて背景を差し替え | — |
| マスク・リビール | mask reveal | 形に沿って映像や文字を見せる | ワイプ、円形マスク拡大 |
| ステッカー・アイコン | stickers, emoji overlay | 感情やリアクション | バウンス付きスケールイン |
| 局ID・ウォーターマーク | channel ID, watermark | 作品・チャンネルの身元 | 常時。不透明度60〜80% |
| 光・粒子素材 | light leak, particles, film grain | 情緒、質感の統一 | スクリーン合成、控えめに |
| 分割画面 | split screen | 同時進行、対比 | 境界線がスライドして分割 |

### C. 動きの付け方

| 手法 | 英語 | 要点 |
|---|---|---|
| キーフレーム | keyframe | 開始と終了の値を決め、間を補間する |
| イーズ | easing | 入場 ease-out、退場 ease-in、移動 ease-in-out。等速（linear）はスクロール系だけ |
| スプリング | spring | 少し行き過ぎて戻る弾力。ポップ系に使う |
| スタッガー | stagger | 複数要素を時間差で出す。2〜4f差 |
| トラッキング追従 | motion tracking | 物や人物に文字を追従させる（編集ソフト向き） |
| パララックス | parallax | 文字と背景を別の速度で動かし奥行きを出す |
| コマ打ち風 | stepped animation | 2〜3fごとに更新して手作り感を出す |

### D. つなぎ（トランジション）

| 手法 | 英語 | 用途 | 時間の目安 |
|---|---|---|---|
| カット | hard cut | 基本。迷ったらカット | 0f |
| クロスディゾルブ | cross dissolve | 時間経過、回想への入り | 12〜20f |
| ブラック／ホワイトフェード | fade to black / white | 場面の終わり／衝撃・回想 | 8〜15f |
| ズームパンチ | zoom punch | 強調・テンポ。カットの頭で一瞬拡大 | 4〜6fで scale 1.15→1 |
| ウィップトランジション | whip transition | 勢いよく次へ | 6〜8f＋モーションブラー |
| スライド／ワイプ | slide / wipe | 話題・場所の切り替え | 10〜15f |
| グリッチ | glitch | 異常・デジタル・不穩 | 4〜8f。多用しない |
| マッチカット | match cut | 形や動きが似たカットをつなぐ | 生成時に構図を合わせておく |
| Jカット／Lカット | J-cut / L-cut | 音を映像より先／後につなぐ。会話が自然になる | 0.3〜1秒ずらす |

### E. 演出

| 手法 | 英語 | 要点 |
|---|---|---|
| 回想ルック | flashback look | 彩度を落とす、セピア、ビネット、軽いぼかし |
| 画面揺れ（後付け） | screen shake | 衝撃の瞬間に6〜10f。振幅は小さく |
| フリーズフレーム | freeze frame | 決定的瞬間、引き、人物紹介 |
| ビート同期 | beat sync | BGMの拍にカットやテロップの出現を合わせる |
| プログレスバー | progress bar | 画面端で残り時間を示し離脱を減らす |
| レターボックス | letterbox | 上下に黒帯を入れて映画的な場面にする |

## ショートドラマの定番構成

| 時間 | 要素 | 手法 |
|---|---|---|
| 0〜2秒 | フック | 一番強い台詞を強調テロップ＋ズームパンチ |
| 2〜5秒 | 状況 | 場所・時刻テロップ、人物紹介（フリーズ） |
| 本編 | 会話と展開 | 会話テロップ、心の声、オノマトペ、カット中心 |
| 山場 | 衝撃 | 画面揺れ、ホワイトフラッシュ、フリーズ |
| 最後2〜3秒 | 引き | フリーズ＋「…」、次回予告、エンドカード、局ID |

演出を盛りすぎない。**強調系（ポップ・パンチ・揺れ）は1本につ見せ場の数だけ**。全部のテロップを弾ませると疲れて強調が効かなくなる。

## 生成モデルとの併用

### 後乗せする前提で生成するとき（基本）

生成プロンプトの最後に文字の禁止を入れる。入れないと勝手に字幕や看板の文字が入り、後乗せのテロップとぶつかる。

```
No subtitles, no captions, no on-screen text, no logos.
```

- Seedance / MiniMax H3：Constraints 欄に入れる
- Kling 3.0：ネガティブプロンプトに `subtitles, text, watermark`
- Wan 3.0：末尾のネガティブに書く。台詞音声を生成する場合も字幕は後乗せにする

テロップを置く余白も生成時に指示しておくと合成が楽になる：`leave empty space in the lower third of the frame` / `subject framed in the upper two-thirds`。

### 生成モデルに文字を描かせるとき

- 対象は看板・メモ・画面内の小物など、多少崩れても物語が壊れない文字に限る。
- 文字列は短く（数語）、引用符で囲んでそのまま書く：`a neon sign that reads "OPEN"`。
- 位置と書体の雰囲気も書く：`bold white sans-serif text centered on the wall`。
- 日本語の文字は崩れやすい前提で、必ず短いテスト生成で確かめてから使う。崩れたら後乗せに切り替える。
- 商品ラベル・ロゴ・法的表記・CTAは描かせない。

## Remotion での実装

### 前提

- Reactで動画を組むライブラリ（v4系）。新規は `npx create-video@latest`。
- **ライセンス**：個人・非営利・3人以下の組織は無償。4人以上の会社で使う場合は会社全体の人数で判定され、Company License が必要。使う立場を確認してから導入する。
- フレーム単位で考える。`useCurrentFrame()` と `useVideoConfig()` から値を計算し、CSSの transition / animation は使わない（書き出し時にズレる）。

### 構成（9:16）

```tsx
// src/Root.tsx
import {Composition} from 'remotion';
import {Episode, episodeSchema} from './Episode';

export const RemotionRoot = () => (
  <Composition
    id="Episode"
    component={Episode}
    width={1080}
    height={1920}
    fps={30}
    durationInFrames={30 * 60}
    schema={episodeSchema}
    defaultProps={{clips: [], telops: [], title: '', channelId: ''}}
    calculateMetadata={({props}) => ({
      durationInFrames: props.clips.reduce((s, c) => s + c.durationInFrames, 0) || 30,
    })}
  />
);
```

### テロップはデータで持つ

```ts
// 作品ごとに差し替えるJSON。秒で書き、部品側でフレームに変換する
type Telop = {
  start: number;          // 秒
  end: number;            // 秒
  text: string;           // 改行は \n
  kind: 'dialogue' | 'emphasis' | 'inner' | 'place' | 'sfx' | 'lowerThird';
  speaker?: string;       // 会話テロップの話者（色分け用）
  sub?: string;           // ロワーサードの2行目（役職など）
};
```

### 日本語フォント

書き出しを安定させるには、**フォントファイルをプロジェクトに同梱する**のが確実（Google Fonts 経由は日本語だとリクエスト数が多く、タイムアウトの原因になる）。

```tsx
// public/fonts/ に woff2 を置く。使う太さだけ
import {loadFont} from '@remotion/fonts';
import {staticFile} from 'remotion';

export const FONT = 'TelopSans';
loadFont({family: FONT, url: staticFile('fonts/NotoSansJP-Black.woff2'), weight: '900'});
loadFont({family: FONT, url: staticFile('fonts/NotoSansJP-Bold.woff2'), weight: '700'});
```

Google Fonts を使う場合は `@remotion/google-fonts/NotoSansJP` の `loadFont` で `weights` と `subsets` を必要なものだけに絞る。

### 部品：会話テロップ（縁取り・安全域内）

```tsx
import {AbsoluteFill, interpolate, useCurrentFrame} from 'remotion';
import {FONT} from './fonts';

export const DialogueTelop: React.FC<{text: string; color?: string; durationInFrames: number}> = ({
  text, color = '#FFFFFF', durationInFrames,
}) => {
  const f = useCurrentFrame();
  const opacity = interpolate(f, [0, 6, durationInFrames - 6, durationInFrames], [0, 1, 1, 0], {
    extrapolateLeft: 'clamp', extrapolateRight: 'clamp',
  });
  return (
    <AbsoluteFill style={{justifyContent: 'flex-start', alignItems: 'center', paddingTop: 1180, paddingInline: 140}}>
      <div style={{
        opacity, fontFamily: FONT, fontWeight: 900, fontSize: 64, lineHeight: 1.35, color,
        textAlign: 'center', whiteSpace: 'pre-wrap',
        WebkitTextStroke: '10px #111', paintOrder: 'stroke fill',
      }}>{text}</div>
    </AbsoluteFill>
  );
};
```

### 部品：強調テロップ（スプリングでポップ）

```tsx
import {spring, useCurrentFrame, useVideoConfig, interpolate} from 'remotion';

export const EmphasisTelop: React.FC<{text: string; durationInFrames: number}> = ({text, durationInFrames}) => {
  const f = useCurrentFrame();
  const {fps} = useVideoConfig();
  const pop = spring({frame: f, fps, config: {damping: 11, stiffness: 180, mass: 0.6}});
  const scale = interpolate(pop, [0, 1], [0.6, 1]);
  const out = interpolate(f, [durationInFrames - 5, durationInFrames], [1, 0], {extrapolateLeft: 'clamp', extrapolateRight: 'clamp'});
  return (
    <div style={{
      position: 'absolute', top: 760, left: 0, right: 0, textAlign: 'center',
      transform: `scale(${scale})`, opacity: out,
      fontFamily: FONT, fontWeight: 900, fontSize: 120, color: '#FFE14D',
      WebkitTextStroke: '14px #000', paintOrder: 'stroke fill',
    }}>{text}</div>
  );
};
```

### 部品：1文字ずつ出す（タイプライター／スタッガー）

```tsx
export const Typewriter: React.FC<{text: string; framesPerChar?: number}> = ({text, framesPerChar = 2}) => {
  const f = useCurrentFrame();
  const chars = Array.from(text); // 絵文字やサロゲートペアを壊さない
  const shown = Math.min(chars.length, Math.floor(f / framesPerChar));
  const cursorOn = Math.floor(f / 15) % 2 === 0;
  return (
    <span style={{fontFamily: FONT, fontWeight: 700}}>
      {chars.slice(0, shown).join('')}
      <span style={{opacity: shown < chars.length || cursorOn ? 1 : 0}}>｜</span>
    </span>
  );
};
```

スタッガー版は文字ごとに `spring({frame: f - i * 2, fps})` で translateY と opacity を動かす。

### 部品：ロワーサード（帯が伸びてから文字）

```tsx
export const LowerThird: React.FC<{name: string; role: string}> = ({name, role}) => {
  const f = useCurrentFrame();
  const {fps} = useVideoConfig();
  const bar = spring({frame: f, fps, config: {damping: 200}});            // 弾まない
  const txt = interpolate(f, [8, 16], [0, 1], {extrapolateLeft: 'clamp', extrapolateRight: 'clamp'});
  return (
    <div style={{position: 'absolute', left: 120, top: 1300}}>
      <div style={{height: 8, width: 360, background: '#E8443A', transformOrigin: 'left', transform: `scaleX(${bar})`}} />
      <div style={{opacity: txt, transform: `translateY(${(1 - txt) * 12}px)`, fontFamily: FONT, color: '#fff'}}>
        <div style={{fontSize: 64, fontWeight: 900}}>{name}</div>
        <div style={{fontSize: 36, fontWeight: 700, opacity: 0.85}}>{role}</div>
      </div>
    </div>
  );
};
```

### テロップをタイムラインに並べる

```tsx
import {Sequence, useVideoConfig} from 'remotion';

export const TelopTrack: React.FC<{telops: Telop[]}> = ({telops}) => {
  const {fps} = useVideoConfig();
  return (
    <>
      {telops.map((t, i) => {
        const from = Math.round(t.start * fps);
        const dur = Math.max(Math.round(0.8 * fps), Math.round((t.end - t.start) * fps));
        return (
          <Sequence key={i} from={from} durationInFrames={dur} layout="none">
            {t.kind === 'emphasis' ? <EmphasisTelop text={t.text} durationInFrames={dur} />
              : t.kind === 'lowerThird' ? <LowerThird name={t.text} role={t.sub ?? ''} />
              : <DialogueTelop text={t.text} durationInFrames={dur} />}
          </Sequence>
        );
      })}
    </>
  );
};
```

### カットのつなぎ（@remotion/transitions）

```tsx
// npx remotion add @remotion/transitions
import {TransitionSeries, linearTiming, springTiming} from '@remotion/transitions';
import {fade} from '@remotion/transitions/fade';
import {slide} from '@remotion/transitions/slide';
import {OffthreadVideo, staticFile} from 'remotion';

export const Clips: React.FC<{clips: {src: string; durationInFrames: number; transition?: 'fade' | 'slide' | 'cut'}[]}> = ({clips}) => (
  <TransitionSeries>
    {clips.flatMap((c, i) => {
      const seq = (
        <TransitionSeries.Sequence key={`s${i}`} durationInFrames={c.durationInFrames}>
          <OffthreadVideo src={c.src} />
        </TransitionSeries.Sequence>
      );
      const next = clips[i + 1];
      if (!next || !c.transition || c.transition === 'cut') return [seq];
      const t = (
        <TransitionSeries.Transition
          key={`t${i}`}
          presentation={c.transition === 'fade' ? fade() : slide({direction: 'from-right'})}
          timing={c.transition === 'fade' ? linearTiming({durationInFrames: 15}) : springTiming({config: {damping: 200}})}
        />
      );
      return [seq, t];
    })}
  </TransitionSeries>
);
```

**注意**：トランジションは前後のカットを重ねるので、その分だけ全体が短くなる。テロップの開始秒をカット基準で持つ場合は、重なり分を差し引いて計算する。

### その他のレシピ

- **ズームパンチ**：カット頭で `scale = interpolate(f, [0, 5], [1.15, 1], {extrapolateRight: 'clamp'})` を映像の親に掛ける。
- **画面揺れ**：`random(seed + f)`（remotion の `random`）で ±8px の translate を6〜10f。`Math.random` は書き出しで結果が変わるので使わない。
- **回想ルック**：映像に `filter: 'saturate(0.3) sepia(0.4) contrast(1.05)'`＋放射状グラデーションのビネット。
- **フリーズフレーム**：`<Freeze frame={n}>` で映像を止め、上にテロップ。
- **写真のスローズーム**：`<Img>` に `scale = interpolate(f, [0, dur], [1, 1.08])`。
- **数字カウントアップ**：`interpolate` の結果を `Intl.NumberFormat('ja-JP')` で整形、`fontVariantNumeric: 'tabular-nums'`。
- **局ID**：全体に敷く `<AbsoluteFill>` の右上（top 300, right 170 前後）、`opacity: 0.7`。
- **チャット画面風**：吹き出しを `Sequence` で順に出す。**実在アプリのロゴ・配色・画面を模倣しない**（架空のUIにする）。

### Vercel 上で書き出す（@remotion/vercel）

Webアプリから自動で書き出す場合：

1. `npx create-video@latest --template vercel` のテンプレートを基にする。Vercel で Blob ストアを作って接続する。
2. `createSandbox()` → `addBundleToSandbox()` → `renderMediaOnVercel({detached: true})` → `getRenderProgress()` で完了を待つ。
3. 終わったら Sandbox を止め、不要な動画と Blob を消す（残り続けて課金される）。

制約：Sandbox の上限は Hobby 45分／Pro 5時間。関数の実行上限を超えないよう、書き出しは必ず detached で。公開するならレート制限と利用額の上限設定を入れる。

## FFmpeg での実装（軽い合成）

### テロップを焼き込む（ASS字幕）

動きのあるテロップは drawtext より ASS 字幕が向いている。縁取り・フェード・移動・拡大が書ける。

```
[Script Info]
PlayResX: 1080
PlayResY: 1920

[V4+ Styles]
Format: Name, Fontname, Fontsize, PrimaryColour, OutlineColour, BackColour, Bold, BorderStyle, Outline, Shadow, Alignment, MarginL, MarginR, MarginV
Style: Dialogue,Noto Sans JP Black,64,&H00FFFFFF,&H00111111,&H00000000,1,1,8,0,2,140,140,560
Style: Emphasis,Noto Sans JP Black,120,&H004DE1FF,&H00000000,&H00000000,1,1,12,0,5,140,140,0

[Events]
Format: Layer, Start, End, Style, Name, MarginL, MarginR, MarginV, Effect, Text
Dialogue: 0,0:00:01.00,0:00:03.80,Dialogue,,0,0,0,,{\fad(150,150)}それ、本当にあなたがやったの？
Dialogue: 0,0:00:04.00,0:00:05.20,Emphasis,,0,0,0,,{\fscx60\fscy60\t(0,180,\fscx100\fscy100)\fad(0,120)}嘘だよね
```

- 色は `&HAABBGGRR`（青緑赤の順）。`MarginV: 560` で下端の安全域を空ける。

```bash
ffmpeg -i input.mp4 -vf "subtitles=telop.ass:fontsdir=./fonts" -c:a copy output.mp4
```

### 画像・局IDを重ねる

```bash
# 右上に局ID（常時）、1〜4秒だけ画像を中央に
ffmpeg -i input.mp4 -i channel_id.png -i insert.png -filter_complex \
"[0][1]overlay=W-w-170:300[a];[a][2]overlay=(W-w)/2:(H-h)/2:enable='between(t,1,4)'" \
-c:a copy output.mp4
```

## 避けること

- 安全域の外（特に下端と右端）に大事な文字を置く。
- 1枚に3行以上、1行16字以上のテロップ。読み切れない。
- 全部のテロップをポップさせる。強調が効かなくなり疲れる。
- フォント3種類以上、色の意味が作品中で変わる。
- 顔の上にテロップを重ねる。
- **点滅（フラッシュ、グリッチ）を短時間に繰り返す**。1秒に3回を超える点滅は光過敏の視聴者に危険。
- 実在のアプリ画面、ロゴ、ニュース番組のテロップを模倣した素材。
- 生成モデルに役名・タイトル・商品名を描かせる（崩れる）。
- Remotion で CSS transition / `Math.random` / 現在時刻に依存した動きを使う（書き出すたびに結果が変わる）。

## チェックリスト

- [ ] 正確さが必要な文字（タイトル・名前・台詞・局ID・CTA）はすべて後乗せ
- [ ] 生成プロンプトに `No subtitles, no on-screen text` を入れている
- [ ] すべての文字が安全域（上288px・下480px以上・左右108px以上）の内側
- [ ] 1行15字以内、2行以内、表示時間が `文字数÷5` 秒以上（最低0.8秒）
- [ ] 縁取りか座布団があり、明るい背景でも読める
- [ ] フォントは2種類まで、商用利用可
- [ ] 入場 ease-out／退場 ease-in、強調演出は見せ場だけ
- [ ] 台詞字幕が音声とずれていない
- [ ] 点滅が1秒に3回を超えない
- [ ] テロップがデータ（JSON）と部品に分かれていて、作品ごとに差し替えられる
- [ ] 実際のTikTok・Reels・Shortsのプレビューで文字が隠れていない