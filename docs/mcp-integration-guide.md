# Strands Agents × MCP統合ガイド

## MCPとStrands Agentsの関係

### なぜMCP統合が重要か
- **既存資産活用**: 開発済みMCPツールをそのまま使用可能
- **判断処理の移譲**: 複雑な判断ロジックをエージェントに委譲
- **ツールエコシステム**: MCPツールライブラリを即座に活用
- **標準化**: Model Context Protocolによる統一インターフェース

### Strands AgentsのMCP優位性
- **ネイティブサポート**: フレームワークレベルでMCP統合
- **自動発見**: MCPサーバーとツールの自動検出・登録
- **透過的利用**: MCPツールを通常ツールと同様に使用可能

## MCP統合の実装パターン

### パターン1: MCPサーバー直接接続
```python
from strands import Agent
from strands.mcp import MCPConnection

# MCPサーバーへの接続
mcp_conn = MCPConnection("ws://localhost:8080/mcp")
tools = mcp_conn.discover_tools()

agent = Agent(
    model="anthropic:claude-sonnet-4",
    tools=tools,
    prompt="Use available MCP tools to help users."
)
```

### パターン2: 複数MCPサーバー統合
```python
# 複数のMCPサーバーからツールを集約
file_tools = MCPConnection("ws://localhost:8080/files").tools
db_tools = MCPConnection("ws://localhost:8081/database").tools
api_tools = MCPConnection("ws://localhost:8082/apis").tools

agent = Agent(
    model="openai:gpt-4",
    tools=file_tools + db_tools + api_tools,
    prompt="You have access to file operations, database queries, and API calls."
)
```

### パターン3: 選択的ツール利用
```python
# 特定の判断処理に必要なツールのみ選択
mcp_conn = MCPConnection("ws://localhost:8080/mcp")
selected_tools = mcp_conn.get_tools_by_category("data-processing")

agent = Agent(
    model="anthropic:claude-4-opus", 
    tools=selected_tools,
    prompt="Focus on data processing tasks using available tools."
)
```

## 判断処理の移譲戦略

### Before (従来のMCP)
```python
# 複雑な判断ロジックをコードで実装
def complex_decision(data):
    if condition1:
        result = tool1(data)
        if condition2:
            return tool2(result)
        else:
            return tool3(result)
    elif condition3:
        return tool4(data)
    # ... 複雑な分岐処理
```

### After (Strands Agents)
```python
# LLMによる判断に委譲
agent = Agent(
    model="claude-sonnet-4",
    tools=[tool1, tool2, tool3, tool4],
    prompt="""
    You are a decision-making agent. Based on the data provided:
    - Analyze the situation carefully
    - Use the appropriate tools in the right sequence
    - Make informed decisions based on intermediate results
    - Provide clear reasoning for your choices
    """
)

result = agent.run(data)  # LLMが自動的に適切なツールを選択・実行
```

## 実用例: ファイル処理エージェント

```python
from strands import Agent
from strands.mcp import MCPConnection

# ファイルMCPサーバーに接続
file_mcp = MCPConnection("ws://localhost:8080/files")
tools = file_mcp.get_tools([
    "read_file", "write_file", "list_directory", 
    "search_files", "get_file_info"
])

file_agent = Agent(
    model="anthropic:claude-sonnet-4",
    tools=tools,
    prompt="""
    You are a file management assistant. You can:
    - Read and analyze file contents
    - Search for files by name or content
    - Organize and manage directories
    - Make decisions about file operations based on content analysis
    
    Always explain your reasoning before taking actions.
    """
)

# 使用例
result = file_agent.run("""
Find all Python files in the project that contain 'TODO' comments, 
read them, and create a summary report of pending tasks.
""")
```

## エラーハンドリングと回復

```python
from strands import Agent, ToolError
from strands.mcp import MCPConnectionError

class ResilientMCPAgent:
    def __init__(self, primary_mcp, fallback_tools=None):
        self.primary_mcp = primary_mcp
        self.fallback_tools = fallback_tools or []
        self.agent = None
        self._initialize_agent()
    
    def _initialize_agent(self):
        try:
            tools = self.primary_mcp.discover_tools()
            self.agent = Agent(
                model="anthropic:claude-sonnet-4",
                tools=tools + self.fallback_tools,
                prompt="Use MCP tools when available, fallback to alternatives if needed."
            )
        except MCPConnectionError:
            # MCP接続失敗時はfallbackツールのみで動作
            self.agent = Agent(
                model="anthropic:claude-sonnet-4", 
                tools=self.fallback_tools,
                prompt="MCP tools unavailable. Use available fallback tools."
            )
    
    def run(self, query):
        try:
            return self.agent.run(query)
        except ToolError as e:
            # ツールエラー時の回復処理
            return self.agent.run(f"Previous tool failed: {e}. Try alternative approach: {query}")
```

## デバッグ・監視

```python
from strands.observability import MCPTracer

# MCP通信の詳細ログ
tracer = MCPTracer(
    log_requests=True,
    log_responses=True, 
    log_errors=True
)

agent = Agent(
    model="openai:gpt-4",
    tools=mcp_tools,
    prompt=prompt,
    tracer=tracer
)

# 実行時にMCP通信が詳細ログ出力される
result = agent.run("Process the data file")
```

## パフォーマンス最適化

### MCP接続プーリング
```python
from strands.mcp import MCPConnectionPool

# 接続プールでMCPサーバーへの接続を効率化
pool = MCPConnectionPool(
    "ws://localhost:8080/mcp",
    pool_size=5,
    keepalive=True
)

agent = Agent(
    model="anthropic:claude-sonnet-4",
    tools=pool.get_tools(),
    prompt=prompt
)
```

### ツールキャッシュ
```python
from strands import Agent
from strands.cache import ToolCache

# よく使うMCPツールの結果をキャッシュ
cache = ToolCache(ttl=300)  # 5分間キャッシュ

agent = Agent(
    model="openai:gpt-4",
    tools=mcp_tools,
    tool_cache=cache,
    prompt=prompt
)
```

## トラブルシューティング

### よくある問題と解決法

1. **MCPサーバー接続失敗**
   - サーバーが起動しているか確認
   - ポート・プロトコル（ws/wss）の確認
   - ファイアウォール設定

2. **ツール実行エラー**
   - MCP tool definitionの確認
   - パラメータ型の整合性
   - 権限・認証の問題

3. **パフォーマンス問題**
   - MCP接続のレイテンシ
   - ツール実行時間の制限
   - 並列実行の検討

### ログレベル設定
```python
import logging

# MCP関連の詳細ログ
logging.getLogger('strands.mcp').setLevel(logging.DEBUG)
logging.getLogger('strands.tools').setLevel(logging.INFO)
```
