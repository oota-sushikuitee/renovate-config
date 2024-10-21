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

- [config:recommended](https://docs.renovatebot.com/presets-config/#configrecommended)のみ有効化

### schedule

- Timezone: Asia/Tokyo
- 毎週月曜日の1時から3時までにPRが作成される
- 毎週月曜日の3時から6時までにPRがマージされる
- 一度に作成されるPRの作成数に制限は設けない
- PRのrebaseや更新はスケジュール外では行わない

### packageRules

- major, minor, patch, pinに応じてラベルを付与する
- minor, patch, pinの場合はautomergeを有効化する
- patchはリリース後3日、minorはリリース後5日の経過を確認する
- GitHub Actionsは、`actions/`配下ではないものは、SHA256のハッシュ値を指定する

### other
