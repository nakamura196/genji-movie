# Claude Codeを使って動画に多言語字幕を自動生成し、IIIF v3マニフェストで公開する

## はじめに

動画コンテンツに字幕をつける作業は手間がかかります。本記事では、Claude Code（CLI版Claude）を使い、動画のフレーム分析から多言語字幕（VTT）の生成、IIIF v3マニフェストの作成までを効率的に行う方法を紹介します。

## 全体の流れ

```
1. 動画ファイル（mp4）を用意する
2. ffmpegでシーンチェンジを検出
3. シーンチェンジポイントのフレームを抽出
4. Claude Codeでフレーム画像を読み取り、内容を把握
5. シーンチェンジのタイムスタンプに基づいてVTTファイルを作成
6. 英語字幕も同様に作成
7. IIIF v3マニフェストを作成
8. HTMLプレーヤーで動画・字幕・読み上げを同期
```

## 前提条件

- Claude Code（CLI版）
- ffmpeg / ffprobe
- 字幕をつけたい動画ファイル（mp4）

```bash
# macOSの場合
brew install ffmpeg
```

## Step 1: シーンチェンジの検出

動画の画面が切り替わるタイミングを自動検出します。これが字幕のタイムスタンプの基準になります。

```bash
ffmpeg -i "video.mp4" \
  -vf "select='gt(scene,0.15)',showinfo" \
  -vsync vfr -f null - 2>&1 \
  | grep "pts_time" \
  | sed 's/.*pts_time:\([0-9.]*\).*/\1/'
```

出力例：
```
3.033333
8.066667
20.066667
25.066667
32.100000
...
```

### なぜシーンチェンジ検出が重要か

最初は3秒間隔でフレームを抽出していましたが、実際の画面切り替えとずれが生じました。シーンチェンジ検出を使うことで、**実際に画面が変わるタイミング**に基づいた正確な字幕タイミングが得られます。

## Step 2: シーンチェンジポイントのフレーム抽出

```bash
mkdir -p scenes
ffmpeg -i "video.mp4" \
  -vf "select='gt(scene,0.15)'" \
  -vsync vfr -q:v 2 \
  scenes/scene_%03d.jpg
```

## Step 3: Claude Codeでフレーム画像の内容を読み取る

Claude Codeのマルチモーダル機能で、抽出したフレーム画像の内容を読み取ります。

```
# Claude Codeのプロンプト例
各シーンの画像を読み取って、動画の内容を把握してください。
日本語のテキストも含めて、画面に表示されている内容を詳細に記述してください。
```

Claude Codeは画像を直接読み取れるため、各シーンの内容（タイトル、説明テキスト、UI要素など）を正確に把握できます。

## Step 4: VTTファイルの作成

シーンチェンジのタイムスタンプと画像の内容に基づいて、VTTファイルを作成します。

### 字幕作成のポイント

- **1文単位で分割する**：長い字幕は読みにくい
- **1キュー2〜5秒**：読みやすい長さ
- **シーンチェンジ境界を尊重**：画面とテキストがずれないようにする
- **シーン内の文を均等配分**：各シーンの持続時間に合わせて文を配置

```
WEBVTT

00:00:00.000 --> 00:00:01.500
デジタル源氏物語、機能紹介。

00:00:01.500 --> 00:00:03.000
「画像とテキストを一緒にみる」についてご説明します。

00:00:03.000 --> 00:00:05.500
メニュー画面から「画像とテキストを一緒にみる」にアクセスします。
```

## Step 5: 英語字幕の作成

日本語VTTと同じタイムスタンプで英語版を作成します。

```
WEBVTT

00:00:00.000 --> 00:00:01.500
Digital Tale of Genji - Feature Introduction.

00:00:01.500 --> 00:00:03.000
We will explain "Viewing Images and Text Together."
```

## Step 6: IIIF v3マニフェストの作成

IIIF Presentation API 3.0に準拠したマニフェストファイルを作成します。多言語字幕は`annotations`としてマニフェストに記述します。

```json
{
  "@context": "http://iiif.io/api/presentation/3/context.json",
  "id": "https://example.com/manifest.json",
  "type": "Manifest",
  "label": {
    "ja": ["動画タイトル（日本語）"],
    "en": ["Video Title (English)"]
  },
  "items": [
    {
      "id": "https://example.com/canvas",
      "type": "Canvas",
      "duration": 162.27,
      "width": 1920,
      "height": 1080,
      "items": [
        {
          "id": "https://example.com/canvas/page",
          "type": "AnnotationPage",
          "items": [
            {
              "id": "https://example.com/canvas/page/annotation",
              "type": "Annotation",
              "motivation": "painting",
              "body": {
                "id": "https://example.com/video.mp4",
                "type": "Video",
                "format": "video/mp4",
                "duration": 162.27,
                "width": 1920,
                "height": 1080
              },
              "target": "https://example.com/canvas"
            }
          ]
        }
      ],
      "annotations": [
        {
          "id": "https://example.com/canvas/annotations",
          "type": "AnnotationPage",
          "items": [
            {
              "id": "https://example.com/canvas/annotations/ja",
              "type": "Annotation",
              "motivation": "supplementing",
              "label": { "ja": ["日本語"] },
              "body": {
                "id": "https://example.com/ja.vtt",
                "type": "Text",
                "format": "text/vtt",
                "language": "ja"
              },
              "target": "https://example.com/canvas"
            },
            {
              "id": "https://example.com/canvas/annotations/en",
              "type": "Annotation",
              "motivation": "supplementing",
              "label": { "en": ["English"] },
              "body": {
                "id": "https://example.com/en.vtt",
                "type": "Text",
                "format": "text/vtt",
                "language": "en"
              },
              "target": "https://example.com/canvas"
            }
          ]
        }
      ]
    }
  ]
}
```

### ポイント

- 動画は`items`内の`Annotation`で`motivation: "painting"`として記述
- 字幕は`annotations`内で`motivation: "supplementing"`として記述
- 各言語のVTTファイルを個別の`Annotation`として追加
- RAMPビューアなどのIIIF対応ビューアで言語切り替えが可能

## Step 7: HTMLプレーヤー

IIIF v3マニフェストを読み込んで再生するHTMLプレーヤーを作成しました。主な機能：

- マニフェストのURLをクエリパラメータで指定（`player.html?manifest=path/to/manifest.json`）
- マニフェストから動画URL・字幕トラックを動的にロード
- 日本語/英語の字幕切り替え
- Web Speech APIによる読み上げ（無料・ブラウザ内蔵）
- 字幕一覧パネルとの同期スクロール
- カスタム字幕レンダリング（黒背景で視認性確保）

## フォルダ構成

```
docs/
├── index.html              # 動画一覧ページ
├── player.html             # 共通プレーヤー（IIIF v3対応）
├── blog.md                 # 本記事
└── videos/
    ├── image-and-text/     # 画像とテキストを一緒にみる
    │   ├── manifest.json   # IIIF v3マニフェスト
    │   ├── video.mp4
    │   ├── ja.vtt          # 日本語字幕
    │   └── en.vtt          # 英語字幕
    ├── ai-image-search/    # AI画像検索（改訂版）
    │   ├── manifest.json
    │   ├── video.mp4
    │   ├── ja.vtt
    │   └── en.vtt
    └── patapata-face/      # パタパタ顔比較
        ├── manifest.json
        ├── video.mp4
        ├── ja.vtt
        └── en.vtt
```

## GitHub Pagesでの公開

```bash
# リポジトリの設定でGitHub Pagesのソースを docs/ に設定
# マニフェスト内のURLをGitHub PagesのURLに合わせて更新
```

## まとめ

- **シーンチェンジ検出**（`ffmpeg -vf "select='gt(scene,0.15)'"...`）により、正確なタイムスタンプを取得
- **Claude Codeのマルチモーダル機能**で、フレーム画像から内容を読み取り、字幕テキストを生成
- **1文単位の字幕**が読みやすく、音声読み上げとの同期も取りやすい
- **IIIF v3マニフェスト**で多言語字幕をAnnotationとして記述し、相互運用性を確保
- **Web Speech API**を活用することで、無料で音声読み上げ機能を実現

### 参考

- [IIIF Audio/Visual: 複数のVTTファイルの記述](https://zenn.dev/nakamura196/articles/ba1fc1624c0503)
- [yt-dlp](https://github.com/yt-dlp/yt-dlp)
- [IIIF Presentation API 3.0](https://iiif.io/api/presentation/3.0/)
- [Web Speech API](https://developer.mozilla.org/ja/docs/Web/API/Web_Speech_API)
