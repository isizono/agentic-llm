# AI Workspace - Strands Agents Setup Tasks

## 🎯 プロジェクト概要
MCPでの判断処理の困難さを解決するため、Strands Agents SDKを使用してagentic LLMを構築する。

## 📋 Claude Code TODO List

### Phase 1: 環境セットアップ
- [ ] Python環境の確認・セットアップ（venv作成）
- [ ] Strands Agents SDKのインストール
  ```bash
  pip install strands-sdk
  # or latest installation method from docs
  ```
- [ ] 必要な依存関係の確認・インストール
- [ ] 基本的な動作確認（Hello World エージェント作成）

### Phase 2: MCP統合の調査・検証
- [ ] Strands AgentsのMCP統合方法の調査
- [ ] 既存MCPツールとの連携可能性の確認
- [ ] サンプルMCPサーバーとの接続テスト
- [ ] MCP tools をStrands Agentsで使用する方法の検証

### Phase 3: プロトタイプ開発
- [ ] シンプルな判断処理エージェントの作成
- [ ] model + tools + promptの基本構成の実装
- [ ] 複数ツールを使った判断フローの実装
- [ ] エラーハンドリングと制御フローの実装

### Phase 4: 実用例の構築
- [ ] 実際のMCP判断ケースの特定
- [ ] そのケースに対応するエージェントの実装
- [ ] テスト・デバッグ・最適化
- [ ] ドキュメント化（使用方法、設定方法等）

## 🔧 技術要件
- **Python**: 3.8+ 推奨
- **モデルプロバイダー**: OpenAI/Anthropic Claude/ローカルモデル対応
- **MCP統合**: 既存MCPサーバーとの互換性
- **実行環境**: ローカル開発、将来的にはAWS等での本番運用も検討

## 📝 出力期待
- 動作するStrands Agentsのセットアップ
- MCP統合の実装例
- 判断処理の具体的な実装パターン
- 今後の開発指針とベストプラクティス

## 🚨 注意点
- MCPとの統合において、既存の判断処理ロジックをエージェントに移行
- Strands AgentsのMCPネイティブサポートを最大限活用
- シンプルさを保ちつつ、拡張可能な設計を心がける
