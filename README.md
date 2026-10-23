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
- `pack.toml` の `[versions]` から Minecraft / Mod Loader を読み取り、Modrinth の候補バージョンを API 結果と照合して互換性を二重確認（判定不能・非対応は追加拒否）
- `index.toml` に登録された任意ファイルの一覧表示、テキスト/TOML/JSON編集、選択削除
- ローカルの JAR / 任意ファイルを直接アップロード予定に追加し、SHA-256 と `index.toml` エントリーを生成（最大25 MiB/ファイル。JAR は互換性を手動確認するチェックが必須）
- Mod メタデータの index.toml 登録、pack.toml の index ハッシュ更新
- pack.toml / index.toml / .pw.toml の TOML 編集と構文検証
- 変更一覧、差分確認、Git Trees API による複数ファイル一括コミット
- Mod メタデータの index からの削除と .pw.toml の Git 削除

## 注意 / 制限

- Packwiz CLI 自体はブラウザーでは実行しません。`packwiz refresh`, `packwiz update`, エクスポート等の CLI と完全に同等ではありません。
- Modrinth 対応のメタデータを生成します。CurseForge は API キーや配布条件などの事情があるため、このプロトタイプの対象外です。
- 一覧は `index.toml` に登録された配布対象ファイルです。リポジトリ内の README や `.github` など、index に登録されていない開発用ファイルは配布対象一覧には表示しません。
- TOML 編集は変更対象の TOML 構文を検証しますが、Packwiz 仕様全体のスキーマ検証はしません。実際に配布する前にローカルの Packwiz CLI で `packwiz refresh` / `packwiz validate` 等、利用バージョンで提供される検証コマンドを実行してください。
- Modrinth 更新では `pack.toml` の `[versions].minecraft` と loader キーを基準に、候補の `game_versions` と `loaders` を厳密照合します。両方を確認できない場合は追加・更新を拒否します。Quilt は Fabric、NeoForge は Forge の候補も Packwiz の互換性ルールに沿って許容します。

## Sinytra Connector と Forge 環境での Fabric Mod

`index.toml` の登録ファイル一覧に Sinytra Connector（Modrinth project ID `u58R1TMW`、名前に `Sinytra Connector` を含むメタデータ、または `mods/connector-*.jar`）を検出した場合、`pack.toml` のローダーが `forge` のときに限り、Modrinth の検索・配布バージョン互換性チェックで `fabric` も許可します。Minecraft バージョンは引き続き `pack.toml` の値と完全一致する必要があります。Connector がない Forge 環境では Fabric Mod は許可されません。

この判定は Connector が `index.toml` のファイル一覧に含まれていることを前提とします。JAR の中身を解析して Connector かどうかを判定するものではなく、ファイル名だけの場合は `mods/connector-*.jar` を検出条件とします。
