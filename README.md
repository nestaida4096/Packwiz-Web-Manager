# Packwiz Web Manager

静的ホスティング（GitHub Pages など）で動く、ブラウザー完結型の Packwiz メタデータ管理 UI のプロトタイプです。専用バックエンドやビルド工程は不要です。

## 起動

`index.html` をローカルサーバーまたは GitHub Pages で開きます。TOML ライブラリは `@iarna/toml@2.2.5` を esm.sh から読み込みます。オフライン運用が必要ならライブラリをローカルに同梱して import 先を変更してください。

## GitHub 接続

1. GitHub の Settings → Developer settings → Personal access tokens → Fine-grained tokens で対象リポジトリだけを選び、Contents: Read and write を付与します。
2. PAT、所有者、リポジトリ名、ブランチを入力して接続します。
3. トークンは JavaScript のメモリ内だけに保持され、localStorage / sessionStorage / Cookie には保存されません。切断またはページ再読み込みで破棄します。
4. 作業終了後は GitHub でトークンを revoke してください。

## セキュリティ上の制約

- このアプリはバックエンドを持たないため、GitHub OAuth のクライアントシークレットを安全に保持できません。PAT を使う方式です。PAT はブラウザー上の JavaScript から利用可能なので、信頼できるホストからのみ開き、ブラウザー拡張機能や第三者スクリプトに注意してください。
- トークンは DOM に表示せず、ブラウザーの永続ストレージに保存しません。ネットワーク通信先は GitHub API と Modrinth API、および TOML ライブラリ CDN です。
- GitHub Pages は HTTPS を使用してください。公開端末では PAT を入力しないでください。

## 現在の機能

- GitHub リポジトリとブランチの読み込み
- Modrinth の Mod 検索、Minecraft バージョン / ローダーフィルター
- Modrinth の .pw.toml メタデータ追加、登録済み Mod の更新確認・更新候補作成
- Mod メタデータの index.toml 登録、pack.toml の index ハッシュ更新
- pack.toml / index.toml / .pw.toml の TOML 編集と構文検証
- 変更一覧、差分確認、Git Trees API による複数ファイル一括コミット
- Mod メタデータの index からの削除と .pw.toml の Git 削除

## 注意 / 制限

- Packwiz CLI 自体はブラウザーでは実行しません。`packwiz refresh`, `packwiz update`, エクスポート等の CLI と完全に同等ではありません。
- Modrinth 対応のメタデータを生成します。CurseForge は API キーや配布条件などの事情があるため、このプロトタイプの対象外です。
- GitHub の内容 API から既存の全ファイルを再帰取得する汎用ファイルブラウザーではなく、pack.toml、index.toml、index が参照する `.pw.toml` を中心に読み込みます。
- TOML 編集は変更対象の TOML 構文を検証しますが、Packwiz 仕様全体のスキーマ検証はしません。実際に配布する前にローカルの Packwiz CLI で `packwiz refresh` / `packwiz validate` 等、利用バージョンで提供される検証コマンドを実行してください。
- Mod 更新は Modrinth API が返す先頭の互換バージョンを選びます。ゲーム版とローダーを指定し、コミット前にファイル名・バージョン・差分を必ず確認してください。
