# Contributing

デジタル源氏物語 動画字幕プロジェクトへの貢献を歓迎します。

## How to Contribute

### 字幕の修正・改善

字幕の内容に誤りや改善点がある場合：

1. このリポジトリをフォーク
2. `docs/videos/<video-name>/ja.vtt` または `en.vtt` を修正
3. Pull Requestを作成

### 新しい動画の追加

新しい動画に字幕を追加する場合：

1. `docs/videos/<new-video-name>/` フォルダを作成
2. 以下のファイルを用意:
   - `video.mp4` - 動画ファイル
   - `ja.vtt` - 日本語字幕
   - `en.vtt` - 英語字幕
   - `manifest.json` - IIIF v3マニフェスト
3. `docs/index.html` に動画カードを追加
4. Pull Requestを作成

### 字幕作成の手順

字幕の作成手順は [blog.md](docs/blog.md) を参照してください。

### マニフェストのテンプレート

既存の `manifest.json` をコピーして、以下を変更してください:

- `id` - 新しいURL
- `label` - 動画タイトル（日英）
- `summary` - 動画の説明（日英）
- `duration` - 動画の長さ（秒）
- VTTファイルへのパス

## Issues

バグ報告や機能リクエストは [Issues](https://github.com/nakamura196/genji-movie/issues) からお願いします。

## Code of Conduct

- 丁寧なコミュニケーションを心がけてください
- 字幕の内容は正確性を重視してください
- 他の貢献者の作業を尊重してください
