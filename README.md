# mizuochi-cmd LP画像置き場

> このリポジトリは元々カーブス案件用に作られたためrepo名が `cvs-lp-images` ですが、
> 現在は複数案件の画像が混在する運用も可能です。案件ごとに別repoを作ってもOKです。

## 🟢 重要：SquadBeyondにおける画像URLの仕様（2026-05-13 検証確定）

**SBは「ドメイン許可リスト」を持っていない。代わりに `article_uid` ごとに「最初の入稿時の画像URL状態」を記憶する仕様。**

### 鉄則

新規SB記事 (article) を作ったら、**最初の `uploadVersionHtml` で必ず最終形態の絶対URL HTML** を送ること。

- ✅ `<img src="https://mizuochi-cmd.github.io/<repo>/images/<案件>/<file>.png">` で最初から入稿
- ❌ 相対パス `<img src="images/<file>.png">` を1度でも入稿すると、その記事は履歴ロックされてGitHub Pages運用ができなくなる
- ❌ `upload_external_photos=true` も履歴汚染するので使わない

履歴ロックされた記事は復活不能。`createAbTestArticle` で**新規記事を作成し、最初から絶対URL HTMLでupload**するのが唯一の対処法。

詳細: `.claude/knowledge/article-lp-deployment-playbook.md` の section 3-1-CRITICAL

## ディレクトリ構造の方針

### 方針A: 案件ごとに別repoを作る（クリーン）
```
cvs-lp-images/     # カーブス専用
dio-lp-images/     # DIOクリニック専用
sponge-lp-images/  # スポンジ案件専用
...
```

### 方針B: 共通repoに混在（このrepoの現状）
```
cvs-lp-images/
└── images/
    ├── lp-A-v3/ ... lp-J/    # カーブス（フラット階層・既存）
    ├── dio-c/                # DIOクリニック 記事C（暫定）
    └── <案件slug>/           # 新規はこの階層から
        └── <バリエーション>/
```

どちらでもSB上の挙動は同じ。**ドメインじゃなく「記事の入稿履歴」が問題なので、repo構成は管理しやすい方を選んでOK**。

## このrepoの既存案件一覧

| 案件 | パス | 用途 |
|---|---|---|
| カーブス | `images/lp-A-v3/` 〜 `images/lp-J/` | フラット階層・既存。動いているので動かさない |
| DIOクリニック（記事C） | `images/dio-c/` | フラット階層・初回案件のため暫定 |

## 反映時間

`git push` 後、GitHub Pages のビルドキューを経て **1〜2分で反映**。

content-length が一致するまで待つコマンド:
```bash
LOCAL_SIZE=$(stat -f %z images/<案件>/<file>.png)
until [ "$(curl -sI "https://mizuochi-cmd.github.io/cvs-lp-images/images/<案件>/<file>.png?cb=$(date +%s)" | grep -i 'content-length' | awk '{print $2}' | tr -d '\r\n')" = "$LOCAL_SIZE" ]; do sleep 15; done
```

## 入稿時の事前検証

```bash
# 1. GitHub Pages 側で200か確認
curl -sI "https://mizuochi-cmd.github.io/<repo>/images/<案件>/<file>.png" | head -1

# 2. uploadVersionHtml 後に getArticleHtml で data-src を確認
# data-src="https://..." → 成功
# data-src="images/..." → 履歴ロック発生（新規記事作り直し）
```
