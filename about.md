---
layout: page
title: About
permalink: /about/
---

## OpenClaw とは

**OpenClaw**（別名 ZeroClaw）は、Rust で書かれた高速・小型の完全自律型 AI エージェントインフラです。

> "Fast, small, and fully autonomous AI assistant infrastructure — deploy anywhere, swap anything 🦀"

### 主な特徴

| 特徴 | 詳細 |
|------|------|
| **超低リソース** | RAM 5MB 以下で動作、$10 のハードウェアでも稼働可能 |
| **高速起動** | シングルバイナリにより即座にコールドスタート |
| **モジュール設計** | プロバイダ・チャネル・ツール・メモリをトレイトで差し替え可能 |
| **プロバイダ非依存** | OpenAI 互換エンドポイントおよび複数 LLM プロバイダをサポート |
| **組み込み DB** | SQLite + FTS5 による独自ベクターDB（Pinecone 不要） |
| **豊富なチャネル** | CLI, Telegram, Discord, Slack など 10 以上のプラットフォーム対応 |
| **セキュア** | 許可リスト制、ワークスペーススコープ、デフォルト安全設計 |

### このブログについて

このブログでは OpenClaw の開発状況、技術的な内部解説、使い方のヒントなどを発信していきます。

- GitHub: [zeroclaw-labs/zeroclaw](https://github.com/zeroclaw-labs/zeroclaw)
