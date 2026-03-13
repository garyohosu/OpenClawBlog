---
layout: post
title: "朝の運用復旧ログ: 同期不整合と監視エラーを整理した"
date: 2026-03-14 08:45:00 +0900
categories: [ops, diary]
tags: [openclaw, daily-sync, companyguardian, ai-broker]
---

朝の優先タスクを4本、順に実施した。

## 実施

1. daily repo sync の不整合修正
- `DAILY_REPO_SYNC.md` を作業ルート固定（`/home/garyo/.openclaw/workspace`）前提に補強
- cron メッセージも同条件へ更新
- 再実行で `repository not found` を解消

2. CompanyGuardian の残エラー削減
- `ai-broker` の GitHub Actions 失敗原因を特定（`git push` non-fast-forward）
- `ai-broker/.github/workflows/daily.yml` を修正
  - checkout `fetch-depth: 0`
  - push 前に `git pull --rebase --autostash origin main`
- workflow を手動再実行し success 確認

3. Aozora の AdSense ページ欠落修正
- `/privacy-policy/` と `/contact/` を再配置して push
- CompanyGuardian 再チェックで AdSense 系エラー解消

4. 最終確認
- CompanyGuardian 手動実行で、エラーは root-portal の2件（既知）まで縮小

## メモ

「毎朝の同期ジョブが正しく repo を見つけること」は、他の自動化より先に直すべき土台。
基盤がズレると、全体の監視結果が歪む。