---
layout: post
title: "実録: WomensMagazine日次ジョブの無出力ハングを追跡した"
date: 2026-03-10 12:25:00 +0900
categories: [ops, postmortem]
tags: [womensmagazine, timeout, local-llm, codex, debugging]
---

今回止まったのは `WomensMagazine` の日次生成ジョブ。
見た目は「無出力で止まる」だったけど、実体は違った。

## 症状

- `python3 scripts/generate_daily.py` 実行中に進捗が出ない
- ログファイルがほぼ空
- commit/push まで到達しない

## 何をしたか

まず、`generate_daily.py` に段階ログを追加した。

- topics
- article
- image
- fact_check
- review
- revise
- publish

それぞれに時刻と経過秒を出すようにした。

次に `local_llm_client.py` 側で、

- attempt番号
- target（どのOllama先か）
- timeout秒
- 成功/失敗

を出すようにした。

## 原因

切り分け結果:

- 停止に見えていた箇所は `STEP article: generate_article`
- その中の local LLM 呼び出し（`qwen2.5:7b`）で `Read timed out` が連続
- さらに Codex polish も待ち時間が長くなると全体が詰まりやすい

つまり「Pythonが固まっていた」のではなく、
**外部推論待ちが長すぎて無音時間が発生していた**のが本体だった。

## 対策（反映済み）

1. 段階ログを常設
2. local LLM 呼び出しログを常設
3. timeoutの上限を短縮
4. Codex呼び出しを `codex exec --output-last-message` に統一
5. polish失敗時はドラフトで継続（全停止回避）

## 学び

「無出力ハング」は、コード停止とは限らない。
今回みたいに、

- どの段階で待っているか
- どの外部依存が詰まっているか

を先に可視化すると、修正は早い。

運用では、まずログ。
感覚より、観測値。
