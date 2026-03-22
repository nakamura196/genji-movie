# Digital Tale of Genji - Video Subtitle Project

デジタル源氏物語の機能紹介動画に多言語字幕（日本語・英語）を付与し、IIIF v3マニフェストで公開するプロジェクトです。

## Demo

https://nakamura196.github.io/genji-movie/

## Features

- IIIF Presentation API 3.0 準拠のマニフェストファイル
- 日本語・英語の二言語字幕（WebVTT形式）
- Web Speech APIによる音声読み上げ機能
- IIIF対応ビューア（RAMP等）でも利用可能

## Videos

| Video | Duration | Description |
|---|---|---|
| 画像とテキストを一緒にみる | 2:42 | TEI & IIIFを活用したParallel Text Viewer |
| AI画像検索（改訂版） | 4:19 | くずし字OCRと類似度計算による画像横断検索 |
| パタパタ顔比較 | 1:38 | vdiff.jsによる源氏百人一首の挿絵比較 |

## Structure

```
articles/
└── blog.md                 # How-to article
docs/
├── index.html              # Video list page
├── player.html             # IIIF v3 manifest-based player
├── favicon.svg / .ico
├── ogp.png
└── videos/
    ├── image-and-text/     # manifest.json, video.mp4, ja.vtt, en.vtt
    ├── ai-image-search/    # manifest.json, video.mp4, ja.vtt, en.vtt
    └── patapata-face/      # manifest.json, video.mp4, ja.vtt, en.vtt
```

## IIIF Manifests

各動画のマニフェストURLは以下の通りです。IIIF対応ビューア（RAMP、Clover等）で直接読み込むことができます。

- `https://nakamura196.github.io/genji-movie/videos/image-and-text/manifest.json`
- `https://nakamura196.github.io/genji-movie/videos/ai-image-search/manifest.json`
- `https://nakamura196.github.io/genji-movie/videos/patapata-face/manifest.json`

## How Subtitles Were Generated

1. ffmpegでシーンチェンジを検出し、正確な画面切り替えタイムスタンプを取得
2. シーンチェンジポイントのフレームを画像として抽出
3. Claude Codeのマルチモーダル機能で各フレームの内容を読み取り
4. タイムスタンプに合わせた1文単位のVTTファイルを日本語・英語で作成

詳細は [blog.md](articles/blog.md) をご覧ください。

## Technology

- **ffmpeg** - Scene change detection & frame extraction
- **Claude Code** - Multimodal frame analysis & subtitle generation
- **IIIF Presentation API 3.0** - Interoperable manifest format
- **WebVTT** - Subtitle format
- **Web Speech API** - Browser-native text-to-speech

## Related

- [デジタル源氏物語](https://genji.dl.itc.u-tokyo.ac.jp/)
- [IIIF Audio/Visual: 複数のVTTファイルの記述](https://zenn.dev/nakamura196/articles/ba1fc1624c0503)

## License

MIT License - see [LICENSE](LICENSE) for details.
