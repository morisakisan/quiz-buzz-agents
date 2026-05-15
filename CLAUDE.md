# QuizBuzz オーケストレーター

## リポジトリ構成

- `quiz-buzz-flutter/` → Flutter アプリ。配下の CLAUDE.md を必ず読むこと
- `quiz-buzz-cdk/` → AWS バックエンド。配下の CLAUDE.md を必ず読むこと

## 並列実行ルール

- Flutter タスクは `quiz-buzz-flutter/` 配下のサブエージェントに委譲
- CDK タスクは `quiz-buzz-cdk/` 配下のサブエージェントに委譲
- 各サブエージェントは配下の CLAUDE.md を必ず読むこと
- CDK → Flutter 依存があるタスクは CDK 完了後に Flutter を開始

## タスクの受け取り方

Notion URL が渡されたら MCP で fetch して、Flutter か CDK かを判断してから適切なサブエージェントに委譲する。

## Git / PR ルール

- main から feature ブランチを切って開発（main への直 push 禁止）
- コミットメッセージは conventional commits 形式
- 実装完了後、bot 名義で PR 作成:

```bash
git config credential.helper '!~/.config/quizbuzz/git-credential-github-app.sh'
GH_TOKEN=$TOKEN gh pr create --title "feat: xxx" --body "..."
git config --unset credential.helper
```

- PR 作成後、squash merge を人間に委ねる前に必ず以下の公式 Skill を実行する:
  - `/security-review` — 機密漏洩、認証バイパス、OWASP top 10 等のチェック
  - `/review` — 一般的なコードレビュー
- 両 Skill の結果を確認し、Critical / High の指摘があれば修正コミットを当該ブランチに push してから人間にマージを委ねる
- PR マージは絶対にしない（人間のみ）

## Notion MCP の活用

- タスク URL が渡されたら必ず fetch して仕様を確認
- 要件定義書 URL: https://www.notion.so/30527a7e5ae9801f98a2c52b9e0f8f32
