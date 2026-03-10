---
layout: post
title: "OpenClaw をローカルで動かすまで"
date: 2026-03-10
categories: [チュートリアル]
tags: [install, rust, quickstart]
---

OpenClaw をローカル環境で動かすまでの手順をまとめます。
Rust の toolchain さえあれば、外部サービスなしで動きます。

## 前提条件

- Rust 1.75 以上（`rustup` 推奨）
- Git

## インストール

```bash
git clone https://github.com/zeroclaw-labs/zeroclaw
cd zeroclaw
cargo build --release
```

ビルド後は `target/release/zeroclaw` のシングルバイナリが生成されます。

## 設定ファイル

`config.toml` を作成して LLM プロバイダを指定します。

```toml
[provider]
type = "openai"
api_key = "sk-..."
model = "gpt-4o"

[memory]
path = "./memory.db"

[channel]
type = "cli"
```

OpenAI 互換エンドポイントであれば `base_url` を変えるだけでローカル LLM（Ollama など）にも対応できます。

```toml
[provider]
type = "openai"
base_url = "http://localhost:11434/v1"
api_key = "ollama"
model = "llama3"
```

## 起動

```bash
./target/release/zeroclaw --config config.toml
```

CLI チャネルが起動し、対話セッションが始まります。

```
OpenClaw 🦀 ready.
> hello
Hello! I'm OpenClaw, your autonomous AI assistant. How can I help you?
```

## リソース消費の確認

```bash
# 起動直後のメモリ使用量を確認
ps aux | grep zeroclaw
```

RAM 5MB 以下というのは誇張ではなく、実際に計測可能です。
次回は Telegram チャネルへのデプロイを扱います。
