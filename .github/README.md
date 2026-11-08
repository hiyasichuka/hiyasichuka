# GitHub Actions の更新と検証

このリポジトリの workflow を変更する人向けの手順です。GitHub CLI の認証を済ませてから、リポジトリのルートで実行してください。

PR と main への push では、GitHub 公式の `gh-actions-lock` が依存 Action の固定先を検証し、zizmor が workflow のセキュリティ上の問題を検出します。検証ジョブの権限は `contents: read` に限定しています。

## Action を変更したら lockfile を生成する

```sh
gh extension install github/gh-actions-lock --pin v0.4.1
gh actions-lock --no-narrow
gh actions-lock --verify --no-narrow
```

インストール済みの場合は、最初のコマンドに `--force` を付けて指定バージョンへ更新してください。`--no-narrow` は workflow の SHA 固定を維持するために指定します。

生成された `.github/workflows/actions.lock` を workflow と一緒にコミットしてください。lockfile は手で編集せず、依存先のコミットやリポジトリ ID の差分を確認します。pinact は使用しません。

## 手元で zizmor を実行する

```sh
uvx zizmor==1.32.0 --pedantic --offline .
```

オフライン実行はローカルの静的解析に使えます。CI は固定した zizmor-action に含まれるハッシュ検証付きの本体を使います。導入時の本体は 1.32.0 です。Action を更新する際は、ローカル検証のバージョンも合わせて更新してください。

CI ではオンライン監査も実行し、結果をログとアノテーションに表示します。SARIF のアップロードは無効にしているため、`security-events: write` は不要です。

## Renovate の更新 PR を確認する

Renovate は Action の SHA とバージョン、およびコメントで指定した `gh-actions-lock` のバージョンを更新対象にします。zizmor 本体は固定した Action に同梱のバージョンへ追随します。

lockfile の自動更新には [Renovate 44.24.0 以降](https://github.com/renovatebot/renovate/issues/45190)が必要です。古い環境では前述のコマンドで生成してください。Action 更新後は workflow の SHA 固定と lockfile の差分を確認し、検証 CI の成功を確認してからマージします。

手動でツールのバージョンを変更した場合は、この文書のコマンド例も合わせて更新します。検証エラーを解消できない場合は、実行ログを添えてこのリポジトリの Issue に記録してください。

## スネークは毎日自動で更新する

`snake.yml` は毎日 09:17 JST にライト・ダーク両テーマの SVG を生成し、`profile-assets` ブランチへ公開します。スケジュール実行は GitHub の混雑状況によって遅れる場合があります。README または生成 workflow を main で変更した際にも更新し、Actions 画面から手動実行もできます。

PR では SVG の生成だけを確認します。公開ジョブは main 上でのみ実行し、生成ジョブと分離しています。`contents: write` を付与するのは公開ジョブだけです。

参考：[GitHub 公式の actions-lock](https://github.com/github/gh-actions-lock)、[zizmor の CI 連携](https://docs.zizmor.sh/integrations/)、[Renovate の actions.lock 対応](https://docs.renovatebot.com/modules/manager/github-actions/#github-actions-lockfile-actionslock)
