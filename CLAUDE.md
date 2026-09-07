# QuizBuzz オーケストレーター

## プロダクト概要

**クイズBuzz** = ユーザー投稿型クイズ SNS モバイルアプリ。

- **コンセプト**: 単体クイズ（1問）＋ Round（複数問セット）を、TikTok 風無限スクロールフィードで消費
- **ターゲット**: 日本・英語圏（多言語対応）
- **マネタイズ**: AdMob 広告 + サブスクリプション

現在のバージョンと開発フェーズはここに書かない（更新漏れで実態とズレるため）。
最新の状況は Notion の要件定義書ハブの「更新履歴」と、タスクボードの「対象バージョン」を参照する。

## リポジトリ構成

- `quiz-buzz-flutter/` → Flutter アプリ。配下の CLAUDE.md を必ず読むこと
- `quiz-buzz-cdk/` → AWS バックエンド。配下の CLAUDE.md を必ず読むこと

## 技術スタック

**Flutter (`quiz-buzz-flutter/`)**
- Dart 3, Material Design 3
- 状態管理: Riverpod
- FVM で Flutter バージョン固定
- 連携: AWS AppSync (GraphQL), Cognito, S3/CloudFront, Firebase Analytics

**CDK (`quiz-buzz-cdk/`)**
- TypeScript（AWS CDK v2）
- AWS AppSync（GraphQL、Pipeline Resolver 多用）
- AWS Cognito（認証、ゲストモード対応）
- Amazon DynamoDB（シングルテーブル設計 + GSI）
- AWS Lambda（Resolver、presigned URL 生成 等）
- S3 + CloudFront（画像配信。アバター / クイズ画像 / Round Cover 等）

**デザイン**
- Figma / Figma Make（AI 生成）

**バージョン同期**
- Flutter と CDK のリリースタグは同じ `vX.Y.Z` で揃える

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

## 機密データ取り扱い

- API キー、トークン、認証情報、`.env` の値などをチャットや PR 本文に出力しない
- 伏字（`****` 等）に頼らない。デフォルトは **「キー名と件数のみ報告し、値は出さない」**
  - 例: 「`AWS_ACCESS_KEY_ID` を含む 3 件を確認した」と報告。値は記載しない
- 機密ファイル (`.env`, `**/GoogleService-Info.plist`, `**/google-services.json`, `~/.ssh/**`, `~/.aws/**` 等) は `.claude/settings.json` の deny 対象。ただし **deny は Read ツールに対するもので、完全な封鎖ではない**
  - `Bash(cat *)` と `Bash(grep *)` は allow されているため、シェル経由なら到達しうる
  - sandbox の `denyRead` はプロジェクト配下 (`./**`) のみで、ホームディレクトリ配下は対象外
  - **設定に守られている前提で動かず、行動規範として自ら触れない**こと。「読めてしまったから読んでよい」ではない

## Notion MCP の活用

- タスク URL が渡されたら必ず fetch して仕様を確認
- 要件定義書（ハブ）: https://www.notion.so/30527a7e5ae9801f98a2c52b9e0f8f32
  - 配下に分割ドキュメント (00_overview / 01_features / 02_ui_ux / 03_database / 04_api / 05_infrastructure / 06_authentication / 07_monetization / 08_multilingual / 09_development_plan / 10_build_configuration / 11_admin_dashboard / 12_categories_and_tags) あり、トピックに応じて適宜 fetch
  - 要件定義書ページ末尾に **アイデアバックログ** と **QuizBuzz タスクボード** DB あり
