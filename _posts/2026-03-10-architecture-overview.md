---
layout: post
title: "OpenClaw のアーキテクチャ概観"
date: 2026-03-10
categories: [技術解説]
tags: [rust, architecture, trait]
---

OpenClaw の設計の核心は **トレイト駆動のモジュラーアーキテクチャ**です。
コアランタイムを変えずに、あらゆるコンポーネントを差し替えられます。

## 4つのコアトレイト

```
┌─────────────────────────────────────────────┐
│               OpenClaw Runtime              │
│                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │ Provider │  │ Channel  │  │  Tools   │  │
│  │(LLM API) │  │(CLI/TG/..)│  │(shell/..)│  │
│  └──────────┘  └──────────┘  └──────────┘  │
│                ┌──────────┐                 │
│                │  Memory  │                 │
│                │(SQLite)  │                 │
│                └──────────┘                 │
└─────────────────────────────────────────────┘
```

### Provider（LLM プロバイダ）

OpenAI 互換エンドポイントを実装したトレイトです。
OpenAI、Anthropic、ローカル LLM（Ollama など）を統一インターフェースで扱えます。

```rust
trait Provider: Send + Sync {
    async fn complete(&self, messages: &[Message]) -> Result<String>;
}
```

### Channel（入出力チャネル）

エージェントと人間（またはシステム）の接点です。
CLI・Telegram・Discord・Slack など、同一のエージェントロジックを複数チャネルで動かせます。

### Tools（ツール）

エージェントが実行できる操作を定義します。

- `shell` — コマンド実行（許可リスト制）
- `file` — ファイル読み書き
- `git` — git 操作
- `browser` — ブラウザ自動化
- `schedule` — タスクスケジューリング

### Memory（メモリ）

独自の SQLite + FTS5 ベクターDBです。Pinecone などの外部サービス不要で、
会話履歴・知識ベースをローカルに保持します。

## なぜ Rust か

- **ゼロコスト抽象**: トレイトの差し替えは実行時コストなし
- **メモリ安全**: バッファオーバーフロー・UAF を型システムで排除
- **シングルバイナリ**: `cargo build --release` で依存なし実行可能ファイルが生成される

次回はそれぞれのコンポーネントをもう少し深掘りしていきます。
