# デジタル源氏物語 動画字幕プロジェクト

## 概要

デジタル源氏物語の機能紹介動画に、日本語・英語の二言語字幕を付与し、IIIF v3マニフェストとして公開するプロジェクトです。

Demo: https://nakamura196.github.io/genji-movie/

GitHub: https://github.com/nakamura196/genji-movie

## 対象動画

以下の3本の機能紹介動画に字幕を付与しています。

| 動画 | 時間 | 内容 |
|---|---|---|
| 画像とテキストを一緒にみる | 2:42 | TEI & IIIFを活用したParallel Text Viewerの使い方 |
| AI画像検索（改訂版） | 4:19 | くずし字OCRと類似度計算による写本画像の横断検索 |
| パタパタ顔比較 | 1:38 | vdiff.jsによる源氏百人一首の挿絵比較ツール |

## 機能

### 多言語字幕

各動画に日本語・英語のWebVTT字幕ファイルを用意しています。字幕は1文単位で分割されており、読みやすさを重視しています。

VTTファイルはそのままYouTubeの字幕としてもアップロードできます。

### IIIF v3マニフェスト

各動画にIIIF Presentation API 3.0準拠のマニフェストファイルを作成しています。動画は`painting`のAnnotation、字幕は`supplementing`のAnnotationとして記述しています。

```json
{
  "annotations": [{
    "type": "AnnotationPage",
    "items": [
      {
        "type": "Annotation",
        "motivation": "supplementing",
        "label": { "ja": ["日本語"] },
        "body": {
          "id": "https://example.com/ja.vtt",
          "type": "Text",
          "format": "text/vtt",
          "language": "ja"
        }
      },
      {
        "type": "Annotation",
        "motivation": "supplementing",
        "label": { "en": ["English"] },
        "body": {
          "id": "https://example.com/en.vtt",
          "type": "Text",
          "format": "text/vtt",
          "language": "en"
        }
      }
    ]
  }]
}
```

### 複数のIIIF対応ビューアで表示可能

IIIF v3マニフェストを採用しているため、以下のIIIF対応ビューアで直接表示できます。

| ビューア | 動画再生 | 字幕表示 | 言語切替 | 備考 |
|---|---|---|---|---|
| Player（本プロジェクト） | ○ | ○ | ○ | Web Speech APIによる読み上げ機能付き |
| [RAMP](https://ramp.avalonmediasystem.org/) | ○ | ○ | ○ | AV資料に最も強い |
| [Theseus](https://theseusviewer.org/) | ○ | ○ | ○ | IIIF対応の汎用ビューア |
| [Clover](https://samvera-labs.github.io/clover-iiif/) | ○ | ○ | △ | Samvera/Northwestern開発 |
| [Universal Viewer](https://universalviewer.io/) | ○ | △ | - | v4でAV対応改善 |

各ビューアへのリンクは、デモページの各動画カードに用意されています。

### 独自プレーヤーの機能

本プロジェクトのPlayerは、IIIF v3マニフェストを読み込んで動画と字幕を再生する独自のHTMLプレーヤーです。

- **マニフェスト読み込み**: URLパラメータ `?manifest=path/to/manifest.json` で指定
- **日英字幕切替**: ボタンまたはキーボード `L` で切り替え
- **音声読み上げ**: Web Speech API（無料・ブラウザ内蔵）による自動読み上げ
- **字幕一覧パネル**: 現在の字幕をハイライト＆自動スクロール
- **クリックジャンプ**: 字幕一覧をクリックしてその時間にジャンプ
- **再生速度**: 0.5x〜2xの速度調整
- **音声設定**: 読み上げ速度・声の高さ・音声の選択
- **キーボード操作**: Space（再生/停止）、←→（3秒スキップ）、T（読み上げ）、L（言語切替）

### YouTubeへの字幕アップロード

各動画のVTTファイルは、YouTubeの字幕としてそのままアップロードできます。

1. YouTube Studio → 動画を選択 → 字幕
2. 「字幕を追加」→「ファイルをアップロード」→「字幕あり」を選択
3. `ja.vtt` をアップロード → 言語を「日本語」に設定
4. 同様に `en.vtt` を「英語」でアップロード

## 字幕の生成方法

字幕はClaude Codeを使って以下の手順で生成しました。

1. **シーンチェンジ検出**: ffmpegで動画の画面切り替えタイミングを自動検出
2. **フレーム抽出**: シーンチェンジポイントのフレームを画像として抽出
3. **画像読み取り**: Claude Codeのマルチモーダル機能で各フレームの内容を読み取り
4. **VTT作成**: タイムスタンプに合わせた1文単位のVTTファイルを日英で作成

詳細な手順は [blog.md](blog.md) をご覧ください。

## 技術スタック

- **IIIF Presentation API 3.0** - 相互運用可能なマニフェスト形式
- **WebVTT** - 字幕ファイル形式
- **Web Speech API** - ブラウザ内蔵の無料テキスト読み上げ
- **ffmpeg** - シーンチェンジ検出・フレーム抽出
- **Claude Code** - マルチモーダルなフレーム分析・字幕生成
- **GitHub Pages** - 静的サイトホスティング

## 関連リンク

- [デジタル源氏物語](https://genji.dl.itc.u-tokyo.ac.jp/)
- [IIIF Audio/Visual: 複数のVTTファイルの記述](https://zenn.dev/nakamura196/articles/ba1fc1624c0503)
- [IIIF Presentation API 3.0](https://iiif.io/api/presentation/3.0/)
