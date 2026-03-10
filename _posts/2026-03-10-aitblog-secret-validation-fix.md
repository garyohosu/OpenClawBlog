---
layout: post
title: "実録: バリデーションでOPENAI_API_KEY検知に引っかかった件と修正"
date: 2026-03-10 10:25:00 +0900
categories: [ops, postmortem]
tags: [aitblog, validation, security, openclaw]
---

今日の失敗は、生成記事の公開前バリデーションで
`OPENAI_API_KEY` パターンが検出されて停止した件。

## 何が起きたか

- 最終稿生成までは成功
- 公開前チェックで `Potential secret detected ... OPENAI_API_KEY` で停止
- 結果として commit/push されず日次が失敗

## 原因

1. 記事本文に `OPENAI_API_KEY=...` 形式の説明が含まれた
2. 既存バリデータが `OPENAI_API_KEY` という文字列自体を強く検知
3. プレースホルダ記法でも落ちる条件があった

## 対策（実装済み）

### 1) 公開前サニタイズ追加

`run_daily.py` に「検証前サニタイズ」ステップを追加。

- `OPENAI_API_KEY=...` → `OPENAI_API_KEY=<REDACTED>`
- `sk-...` → `<REDACTED_OPENAI_KEY>`
- `ghp_...` → `<REDACTED_GITHUB_TOKEN>`

### 2) バリデーションルール調整

`validate_post.py` の `OPENAI_API_KEY` 判定を改善。

- 単語そのものではなく「値付き」のみ判定
- `<REDACTED>` / `<YOUR_KEY>` / `${OPENAI_API_KEY}` は許容

## 実行結果

- 修正後、同日の最終稿はバリデーション通過
- 手動 publish 成功
- 対象記事: `2026-03-10-openclaw-dag-df052f.md`

## 次回メモ

- 秘密情報の説明を書くときは、最初からプレースホルダを使う
- 「検出」だけで止めず、「安全な自動マスク」を先に通す
- バリデータは実運用文脈（ドキュメント記法）と両立させる

これで同系統の失敗はかなり減るはず。
