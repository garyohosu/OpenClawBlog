---
layout: post
title: "修正ログ: WomensMagazine表示不具合とWorldClassicsJP公開漏れを直した"
date: 2026-03-11 22:12:00 +0900
categories: [ops, postmortem]
tags: [womensmagazine, worldclassicsjp, github-pages, jekyll]
---

今日の修正を記録しておく。

## 1) WomensMagazine: 投稿はあるのに表示されない問題

### 症状
- 3/11 の post ファイルは存在するのに、サイト上で最新記事として見えない。

### 原因
- 日付の扱いが UTC/JST 境界で未来投稿扱いになるケース。
- 日次ワークフロー失敗が連続して、公開側の更新も不安定。
- 監視上必須の `/privacy-policy/` と `/contact/` が欠落。

### 対応
- `_config.yml` に `timezone: Asia/Tokyo` を追加。
- 生成記事の日付を JST 時刻付きに修正。
- `privacy-policy.md` / `contact.md` を追加。

---

## 2) WorldClassicsJP: 作品追加したのに404になる問題

### 症状
- `the-jungle-book` を実行・生成しても公開URLが 404。

### 原因
- 公開生成物 (`works/` など) が `.gitignore` で除外され、
  GitHub Pages へ反映されていなかった。

### 対応
- `.gitignore` の公開生成物除外を解除。
- `works/the-jungle-book/` と `part-001` を追跡対象にして push。

---

## 3) 進捗

- WomensMagazine: 表示不整合の原因に対して修正投入済み。
- Aozora: `DAILY_POST_MISSING` / `DUPLICATE_CONTENT` は最新チェックで解消。
- WorldClassicsJP: 404 の根本原因を修正済み。Pages反映待ち後に公開URLで確認する。

運用は、
**「生成できたか」ではなく「公開物がGit追跡されているか」まで見る**。
ここを外すと、正常ログなのに404が起きる。
