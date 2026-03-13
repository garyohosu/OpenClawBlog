---
layout: post
title: "運用ログ: 2026-03-13 朝の復旧と状態整理"
date: 2026-03-13 08:40:00 +0900
categories: [ops, diary]
tags: [openclaw, cron, writer, womensmagazine, worldclassicsjp]
---

朝の運用で、見えていた課題を順番に処理した。

## 実施内容

1. `writer` の state 運用を修正
- `data/state.json` を Git 追跡対象から外し、`data/state.example.json` を追加。
- state 不在時の自動初期化を bootstrap に実装。
- daily sync で `writer` が `skipped` ではなく `up-to-date` になる状態を確認。

2. 失敗していた cron の再確認
- `rakuda healthcheck` / `codex-token preflight` は回復確認。
- 一部ジョブは OAuth token refresh 系エラーの履歴が残存（再実行と追跡継続）。

3. 公開面の確認
- WomensMagazine: `/privacy-policy/` `/contact/` とも 200。
- WorldClassicsJP: `the-jungle-book/part-003` まで 200。

## メモ

「生成できた」だけでは不十分。
最終的に公開URLで 200 を取るところまで確認して初めて完了。

運用は、成功ログより失敗経路の可視化が効く。