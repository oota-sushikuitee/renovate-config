# Common Renovate configuration for oota-sushikuitee

## About

GitHub Organization全体で有効化したいRenovateの設定をまとめたものです。

## Usage

```json
{
  "extends": [
    "github>oota-sushikuitee/renovate-config"
  ]
}
```

## Configuration

### extends

- [config:recommended](https://docs.renovatebot.com/presets-config/#configrecommended)
- [:configMigration](https://docs.renovatebot.com/presets-default/#configmigration) — 非推奨設定を自動で移行PRにする
- [:semanticCommitTypeAll(chore)](https://docs.renovatebot.com/presets-default/#semanticcommittypeallarg0) — commit prefixを`chore(deps)`に統一
- [group:allNonMajor](https://docs.renovatebot.com/presets-group/#groupallnonmajor) — minor/patchを1つのPRにまとめる
- [group:allDigest](https://docs.renovatebot.com/presets-group/#groupalldigest) — digest更新(SHA refresh)を1つのPRにまとめる

### schedule / merge

- Timezone: Asia/Tokyo
- 毎週月曜日の1時から3時までにPRが作成される
- automergeは固定時間枠を設けず、CIが緑かつ経過日数条件を満たし次第マージする（無料プランで repo の "Allow auto-merge" が使えないため `platformAutomerge: false`、Renovate自身が直接マージする）
- 一度に作成されるPRの作成数に制限は設けない
- `rebaseWhen: conflicted` — コンフリクト時のみrebaseし、base更新ごとの不要なrebase/CI再実行を避ける

### packageRules

- major, minor, patch, pinに応じてラベルを付与する
- minor, patch, pin, digestの場合はautomergeを有効化する
- patchはリリース後3日、minor/majorはリリース後5日の経過を確認する
- GitHub Actionsは、`actions/`配下ではないものは、SHA256のハッシュ値を指定する

### other

- dependencyDashboardの有効化
