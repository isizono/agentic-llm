# Strands Agents 技術詳細・実装パターン

## 基本的なエージェント作成

```python
from strands import Agent, tool

# ツール定義
@tool
def word_count(text: str) -> int:
    """Count words in text. 
    This docstring is used by the LLM to understand the tool's purpose.
    """
    return len(text.split())

@tool
def search_web(query: str) -> str:
    """Search the web for information."""
    # 実装例
    return f"Search results for: {query}"

# エージェント作成
agent = Agent(
    model="anthropic:claude-sonnet-4-20250514",
    tools=[word_count, search_web],
    prompt="You are a helpful assistant that can count words and search the web."
)

# 実行
result = agent.run("How many words are in 'Hello world' and search for Python tutorials")
```

## MCP統合パターン

```python
from strands import Agent
from mcp_integration import MCPToolProvider

# MCPツールの読み込み
mcp_tools = MCPToolProvider.from_server("localhost:8080")

agent = Agent(
    model="openai:gpt-4",
    tools=mcp_tools.get_all_tools(),
    prompt="You have access to MCP tools. Use them to help users."
)
```

## マルチエージェント構成

```python
# 専門特化エージェント
research_agent = Agent(
    model="claude-4-opus",
    tools=[web_search, document_reader],
    prompt="You are a research specialist. Gather information comprehensively."
)

analysis_agent = Agent(
    model="gpt-4",
    tools=[data_analyzer, chart_generator],
    prompt="You analyze data and create insights."
)

# エージェント連携
def multi_agent_workflow(query):
    research_result = research_agent.run(f"Research: {query}")
    analysis_result = analysis_agent.run(f"Analyze: {research_result}")
    return analysis_result
```

## 設定・環境変数

```bash
# モデル設定
export STRANDS_MODEL_PROVIDER=anthropic
export STRANDS_MODEL_ID=claude-sonnet-4-20250514
export STRANDS_MAX_TOKENS=32000
export STRANDS_BUDGET_TOKENS=1024

# API Keys
export ANTHROPIC_API_KEY=your_api_key
export OPENAI_API_KEY=your_api_key
export AWS_ACCESS_KEY_ID=your_aws_key
export AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## モデル設定例

```python
# Anthropic Claude設定
agent = Agent(
    model="anthropic:claude-sonnet-4",
    model_config={
        "max_tokens": 32767,
        "temperature": 0.1
    }
)

# AWS Bedrock設定
agent = Agent(
    model="bedrock:anthropic.claude-v2",
    model_config={
        "region": "us-east-1",
        "max_tokens": 4000
    }
)

# ローカルモデル（Ollama）設定
agent = Agent(
    model="ollama:llama2",
    model_config={
        "host": "localhost:11434"
    }
)
```

## エラーハンドリング・リトライ

```python
from strands import Agent, RetryPolicy

agent = Agent(
    model="openai:gpt-4",
    tools=tools,
    prompt=prompt,
    retry_policy=RetryPolicy(
        max_attempts=3,
        backoff_factor=2.0,
        retry_on_timeout=True
    )
)
```

## 観測性・ロギング

```python
import logging
from strands.observability import trace, metrics

# OpenTelemetry統合
@trace
def my_agent_workflow(input_data):
    with metrics.timer("agent_execution"):
        result = agent.run(input_data)
        metrics.counter("agent_calls").increment()
        return result
```

## ベストプラクティス

### プロンプト設計
- 明確なロール定義
- 期待する出力形式の指定
- エラー時の行動指針
- ツール使用の制約・ガイドライン

### ツール設計
- 単一責任の原則
- 明確なdocstring（LLMが理解できる説明）
- 適切な型アノテーション
- エラーハンドリング

### パフォーマンス最適化
- 適切なモデルサイズの選択
- ツールの実行時間制限
- キャッシュ戦略の活用
- 並列処理の考慮
