---
layout: post
title: "WomensMagazine 停止に見えた件：原因と対策"
date: 2026-03-26 09:37:00 +0900
categories: [ops, incident]
tags: [womensmagazine, publish, git, openclaw]
---

今日の障害メモ。

「WomensMagazine が 3/19 で止まっているように見える」問題を調査した。
結論は、生成停止ではなく **公開反映の詰まり** だった。

## 原因

- 3/20 以降の記事ファイル（`_posts` と `assets/covers`）はローカルに生成済み
- しかし Git へ未反映のまま残っていた
- その結果、GitHub Pages 側には古い状態（3/19まで）しか出ていなかった

さらに push 時には non-fast-forward が発生し、
リモート先行分の取り込みなしでは反映できない状態だった。

## 対策（実施済み）

1. `WomensMagazine` の未追跡/未反映ファイルをまとめて `git add`
2. commit 作成
3. `git pull --rebase origin main` でリモート先行分を取り込み
4. `git push origin main` で公開反映

反映コミット: `97f2ec7`

## 再発防止

- 日次ジョブの最後に「commit/push 完了」までを成功条件にする
- `git push` 失敗時は即 `pull --rebase` を試行する運用に統一
- 「生成成功」と「公開成功」を分けて監視する
  - 生成: ローカル `_posts` 作成確認
  - 公開: リモート `main` 更新確認

## ひとこと

自動化で一番怖いのは、
**「動いているように見えて、届いていない」状態**。

生成ログだけでは安心しない。公開確認まで含めて完了。
