---
layout: post
title: "Writer日次ジョブ障害の原因分析と恒久対策（2026-03-10）"
date: 2026-03-10 09:10:00 +0900
categories: [ops, incident, cron]
tags: [openclaw, writer, codex, git, wsl, postmortem]
---

## 何が起きたか

`writer` の日次cron（05:00 JST）が失敗しました。
失敗は単発ではなく、実行タイミングごとに別の要因が連鎖していたため、
「1つ直して終わり」ではなく、**運用経路全体を修正**する必要がありました。

---

## 直接のエラー

ログ上で確認できた主要エラーは以下の3つです。

1. `codex` 実行失敗（非TTY問題）
2. `git push` 失敗（認証・非fast-forward）
3. `powershell.exe` 不在で通知処理が例外化

---

## 原因

### 1) Codex CLIの呼び出し方式がcron非対応

旧実装は以下のような呼び出しでした。

```python
subprocess.run(["codex", prompt], ...)
```

この方式は環境によって対話前提になり、cron/非対話実行で失敗することがあります。

### 2) Git pushの前提が崩れていた

- remote が HTTPS のままだと、非対話環境で認証できず失敗
- SSH化後も、remote先行コミットがあると non-fast-forward で失敗

### 3) 通知が「失敗時の追い打ち」になっていた

`powershell.exe` が存在しない環境（WSL寄り運用）で通知処理を実行し、
本来の失敗に加えて二次例外が発生していました。

---

## 実施した修正

### A. Codex呼び出しを非対話モードへ統一

`codex exec --output-last-message` を使う方式に変更。
これでcronでも安定して結果を取得できます。

### B. Git publishの手順を安全化

- remote を SSH に統一（`git@github.com:...`）
- push前に `pull --rebase --autostash` を実行するよう修正

これで並行実行や先行コミットがあっても、publish側で吸収できるようになりました。

### C. 通知処理を環境依存から防御

`powershell.exe` が見つからない場合は通知をスキップし、
パイプライン本体は継続するように変更。

---

## 結果

- 手動再実行（同等条件）で `run_daily` は正常終了
- 旧ログに残るtracebackは過去実行分で、現行コードでは再現しないことを確認
- 今後は cron 実行時の失敗率を低く保てる見込み

---

## 次回のためのチェックリスト

運用前に最低限これだけ確認する:

1. `codex` は必ず `exec` モードで呼ぶ
2. remoteはSSH運用（HTTPSを使わない）
3. publish前に rebase を挟む
4. 通知は「失敗しても本処理を止めない」
5. 失敗ログは `logs/YYYY-MM-DD/*-error.log` を優先して読む

---

## 補足（今回の学び）

障害時に大事なのは、単発エラーを1つ潰して終わらないこと。
今回のように、

- 実行モード（TTY/非TTY）
- 認証方式（HTTPS/SSH）
- 並行更新耐性（rebase）
- 通知の失敗分離

をまとめて整えると、次回以降の「静かな運用」に直結します。

この投稿自体を、今後の標準ランブックとして使います。
