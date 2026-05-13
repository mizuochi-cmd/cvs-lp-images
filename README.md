# mizuochi-cmd LP画像置き場（汎用）

> **このリポジトリは「カーブス専用」ではなく、SquadBeyond配信用の汎用LP画像置き場として運用しています。**
> 由来はカーブス案件のために最初に作られたため `cvs-lp-images` という名前ですが、現在は複数案件の画像が共存しています。

## なぜこのrepoを使うのか

SquadBeyond（SB）には外部画像ドメインに関する事実上の許可制があり、過去に使った既存ドメインの絶対URLは `data-src` に保持されますが、新規ドメイン（新規GitHub Pages repo）は SBが相対パスに強制書き換えして配信できなくなります。

そのため、新規案件ごとに repo を作るのではなく、**実績のあるこの repo を全案件で使い回す**運用にしています。

## ディレクトリ構造

```
cvs-lp-images/
└── images/
    ├── lp-A-v3/           # カーブス LP-A v3
    ├── lp-B-v2/           # カーブス LP-B v2
    ├── lp-C/              # カーブス LP-C
    ├── lp-D-v2/           # カーブス LP-D v2
    ├── lp-E/ ... lp-J/    # カーブス LP-E〜J
    ├── dio-c/             # ⚠️ 暫定flat: DIOクリニック 記事C（初回案件）
    │
    └── <案件slug>/        # 🆕 新規案件は階層構造で配置
        └── <バリエーション>/
            └── *.png
```

### 新規案件の追加ルール（2026-05-13〜）

新規案件は以下の階層構造で配置すること:

```
images/
└── <案件slug>/             # 例: dio-clinic, sponge-x, ...
    ├── article-a/           # 記事LP A
    ├── article-b/           # 記事LP B
    └── article-c/           # 記事LP C
        ├── cover.png
        ├── home.png
        └── ...
```

#### HTML側の参照URL

```html
<img src="https://mizuochi-cmd.github.io/cvs-lp-images/images/dio-clinic/article-a/cover.png">
```

## 既存案件のディレクトリ一覧

| 案件 | パス | 用途 |
|---|---|---|
| カーブス | `images/lp-A-v3/` 〜 `images/lp-J/` | フラット階層・既存。動いてるので動かさない |
| DIOクリニック（記事C） | `images/dio-c/` | フラット階層・初回案件のため暫定 |

## 反映時間

`git push` 後、GitHub Pages のビルドキューを経て **1〜2分で反映** されます。Pages のステータスは `https://github.com/mizuochi-cmd/cvs-lp-images/deployments` で確認可能。

content-length が一致するまで待つコマンド:
```bash
LOCAL_SIZE=$(stat -f %z images/<案件>/<file>.png)
until [ "$(curl -sI "https://mizuochi-cmd.github.io/cvs-lp-images/images/<案件>/<file>.png?cb=$(date +%s)" | grep -i 'content-length' | awk '{print $2}' | tr -d '\r\n')" = "$LOCAL_SIZE" ]; do sleep 15; done
```

## SBが新規ドメインを弾く件の参考リンク

- 詳細手順: `.claude/knowledge/article-lp-deployment-playbook.md` の section 3-1-CRITICAL
- 2026-05-13 DIO案件で発覚・対処済み
