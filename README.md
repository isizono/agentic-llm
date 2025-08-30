# Agentic LLM Framework Exploration

MCPでの判断処理の困難さを解決するため、agentic LLMフレームワークを探索するプロジェクトです。

## 検討フレームワーク

### Strands Agents (AWS, 2025年5月発表)
- **モデル駆動アプローチ**: model + tools + prompt の3要素
- **MCPネイティブサポート**: 既存MCPツールを直接活用
- **シンプル**: 数行のコードでエージェント作成
- **プロバイダー自由**: AWS Bedrock、OpenAI、Anthropic、ローカルモデル対応

### LangChain/LangGraph (2025年最新版)
- **グラフベースオーケストレーション**: ノードとエッジによる複雑なフロー制御
- **細かい制御**: 低レベルでの柔軟なカスタマイズ
- **豊富なエコシステム**: LangGraph Platform、LangSmith、Studio等
- **実績**: Klarna、Uber、LinkedIn等での実用例多数
- **MCP対応**: エージェントが自動でMCPエンドポイント公開

## プロジェクト目標

MCPでの複雑な判断処理を、LLMの推論能力を活用したagenticアプローチで解決する。

## ステータス

🚀 **開始**: 2025-08-30
📋 **フェーズ**: フレームワーク調査・検証
