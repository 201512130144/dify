# Dify Core 开发者快速参考

## 快速导航

### 核心模块位置

| 模块 | 路径 | 核心文件 | 说明 |
|------|------|----------|------|
| Model Runtime | `/api/core/model_runtime/` | `model_provider_factory.py` | 模型调用统一接口 |
| Workflow Engine | `/api/core/workflow/` | `workflow_entry.py`, `graph_engine.py` | 工作流执行引擎 |
| Agent System | `/api/core/agent/` | `base_agent_runner.py`, `fc_agent_runner.py` | 智能体系统 |
| RAG System | `/api/core/rag/` | 各子目录 | 检索增强生成 |
| Tools | `/api/core/tools/` | `tool_manager.py` | 工具系统 |
| App Framework | `/api/core/app/` | `base_app_runner.py` | 应用框架 |
| Plugin | `/api/core/plugin/` | 插件相关 | 插件系统 |
| MCP | `/api/core/mcp/` | `mcp_client.py` | MCP 协议 |

## 常用类和接口

### 1. 模型调用

```python
from core.model_manager import ModelInstance
from core.provider_manager import ProviderManager

# 获取模型实例
provider_manager = ProviderManager()
configurations = provider_manager.get_configurations(tenant_id)
model_instance = provider_manager.get_model_instance(
    tenant_id=tenant_id,
    model_type=ModelType.LLM,
    provider="openai",
    model="gpt-4"
)

# 调用模型
result = model_instance.invoke(
    prompt_messages=[...],
    model_parameters={...},
    stream=False
)
```

### 2. 工作流执行

```python
from core.workflow.workflow_entry import WorkflowEntry
from core.workflow.entities.variable_pool import VariablePool

# 初始化工作流
workflow_entry = WorkflowEntry(
    tenant_id=tenant_id,
    app_id=app_id,
    workflow_id=workflow_id,
    workflow_type=WorkflowType.WORKFLOW,
    graph_config=graph_config,
    graph=graph,
    user_id=user_id,
    user_from=user_from,
    invoke_from=invoke_from,
    call_depth=0,
    variable_pool=variable_pool
)

# 执行工作流
for event in workflow_entry.run():
    # 处理事件
    pass
```

### 3. RAG 检索

```python
from core.rag.retrieval.retrieval import Retrieval

# 执行检索
retrieval = Retrieval()
documents = retrieval.retrieve(
    query="用户问题",
    dataset_ids=[...],
    top_k=5
)
```

### 4. 工具调用

```python
from core.tools.tool_manager import ToolManager

# 获取工具
tool_manager = ToolManager()
tool = tool_manager.get_tool(
    provider_type="builtin",
    provider_name="google",
    tool_name="search"
)

# 调用工具
result = tool.invoke(
    user_id=user_id,
    tool_parameters={"query": "search term"}
)
```

### 5. Agent 执行

```python
from core.agent.fc_agent_runner import FunctionCallAgentRunner

# 初始化 Agent
agent_runner = FunctionCallAgentRunner(
    tenant_id=tenant_id,
    app_id=app_id,
    ...
)

# 运行 Agent
for event in agent_runner.run():
    # 处理事件
    pass
```

## 关键数据结构

### ProviderModelBundle
```python
from core.entities.provider_configuration import ProviderModelBundle

# 包含提供商配置和模型实例
bundle = ProviderModelBundle(
    configuration=provider_configuration,
    model_type_instance=model_type_instance
)
```

### VariablePool
```python
from core.workflow.entities.variable_pool import VariablePool

# 工作流变量池
variable_pool = VariablePool(
    system_variables={...},
    user_inputs={...}
)

# 获取变量
value = variable_pool.get(["node_id", "output_key"])

# 添加变量
variable_pool.add(["node_id", "output_key"], value)
```

### ModelConfigWithCredentialsEntity
```python
from core.app.entities.app_invoke_entities import ModelConfigWithCredentialsEntity

# 带凭证的模型配置
model_config = ModelConfigWithCredentialsEntity(
    provider="openai",
    model="gpt-4",
    credentials={...},
    parameters={...}
)
```

## 事件系统

### Workflow 事件

```python
from core.workflow.entities.node_entities import NodeRunResult

# 节点运行结果
class NodeRunResult:
    status: WorkflowNodeExecutionStatus
    outputs: dict
    metadata: dict
    error: Optional[str]
```

### Queue 事件

```python
from core.app.entities.queue_entities import (
    QueueLLMChunkEvent,
    QueueMessageEndEvent,
    QueueAgentMessageEvent
)

# LLM 流式输出事件
chunk_event = QueueLLMChunkEvent(
    chunk=chunk_content,
    delta_text=delta_text
)

# 消息结束事件
end_event = QueueMessageEndEvent(
    llm_result=llm_result
)
```

## 错误处理

### 常见异常

```python
# 模型运行时错误
from core.model_runtime.errors.invoke import (
    InvokeAuthorizationError,
    InvokeConnectionError,
    InvokeRateLimitError,
    InvokeBadRequestError
)

# 工作流错误
from core.workflow.errors import (
    WorkflowNodeRunFailedError,
    WorkflowRunFailedError
)

# 提供商错误
from core.errors.error import (
    ProviderTokenNotInitError,
    ModelNotExistError
)

# 工具错误
from core.tools.errors import (
    ToolProviderNotFoundError,
    ToolNotFoundError
)
```

### 错误处理示例

```python
try:
    result = model_instance.invoke(...)
except InvokeAuthorizationError as e:
    # 处理认证错误
    logger.error(f"Authorization failed: {e}")
except InvokeRateLimitError as e:
    # 处理限流错误
    logger.error(f"Rate limit exceeded: {e}")
except Exception as e:
    # 处理其他错误
    logger.error(f"Unexpected error: {e}")
```

## 配置管理

### 模型提供商配置

YAML 配置文件位置：
- `/api/core/model_runtime/model_providers/{provider_name}/{model_name}.yaml`

配置示例：
```yaml
model: gpt-4
label:
  en_US: GPT-4
model_type: llm
features:
  - agent-thought
  - vision
model_properties:
  mode: chat
  context_size: 8192
parameter_rules:
  - name: temperature
    use_template: temperature
    label:
      en_US: Temperature
    type: float
    default: 1.0
    min: 0.0
    max: 2.0
```

### 工具配置

YAML 配置文件位置：
- `/api/core/tools/builtin_tool/providers/{provider_name}/tools/{tool_name}.yaml`

配置示例：
```yaml
identity:
  name: google_search
  author: Dify
  label:
    en_US: Google Search
description:
  en_US: Search Google for information
parameters:
  - name: query
    type: string
    required: true
    label:
      en_US: Query
    human_description:
      en_US: The search query
```

## 常用工具函数

### 提示词处理

```python
from core.prompt.simple_prompt_transform import SimplePromptTransform
from core.prompt.advanced_prompt_transform import AdvancedPromptTransform

# 简单提示词转换
simple_transform = SimplePromptTransform()
messages = simple_transform.get_prompt(
    prompt_template=prompt_template,
    inputs=inputs
)

# 高级提示词转换
advanced_transform = AdvancedPromptTransform()
messages = advanced_transform.get_prompt(
    prompt_template=advanced_prompt_template,
    inputs=inputs,
    query=query
)
```

### 文件处理

```python
from core.file.models import File
from factories import file_factory

# 创建文件
file = file_factory.build_from_local_file(
    filename=filename,
    tenant_id=tenant_id,
    user_id=user_id
)

# 上传文件
upload_file = file_factory.build_from_mapping(
    mapping=file_mapping,
    tenant_id=tenant_id,
    user_id=user_id
)
```

### 加密解密

```python
from core.helper import encrypter

# 加密
encrypted = encrypter.encrypt_token(tenant_id, token)

# 解密
decrypted = encrypter.decrypt_token(tenant_id, encrypted)
```

## 测试

### 单元测试

```python
import pytest
from unittest.mock import Mock, patch

def test_model_invoke():
    # 模拟依赖
    with patch('core.model_manager.ModelInstance') as mock_model:
        mock_model.return_value.invoke.return_value = Mock(
            content="response",
            usage={"total_tokens": 100}
        )
        
        # 执行测试
        result = model_instance.invoke(...)
        
        # 断言
        assert result.content == "response"
        assert result.usage["total_tokens"] == 100
```

### 集成测试

```python
from tests.integration_tests.model_providers import ModelProviderTestSuite

class TestOpenAIProvider(ModelProviderTestSuite):
    def test_invoke(self):
        # 测试实际调用
        result = self.provider.invoke(...)
        assert result is not None
```

## 性能优化建议

### 1. 使用缓存

```python
from extensions.ext_redis import redis_client

# 缓存提供商凭证
cache_key = f"provider_credentials:{tenant_id}:{provider}"
credentials = redis_client.get(cache_key)
if not credentials:
    credentials = fetch_credentials()
    redis_client.setex(cache_key, 3600, credentials)
```

### 2. 批量操作

```python
# 批量获取模型实例
model_instances = [
    model_manager.get_model_instance(...)
    for model in models
]

# 并发调用
from concurrent.futures import ThreadPoolExecutor
with ThreadPoolExecutor(max_workers=5) as executor:
    results = list(executor.map(invoke_model, model_instances))
```

### 3. 流式处理

```python
# 使用生成器减少内存占用
def stream_workflow_events(workflow_entry):
    for event in workflow_entry.run():
        yield event
        
# 流式输出给客户端
for event in stream_workflow_events(workflow_entry):
    send_to_client(event)
```

## 调试技巧

### 1. 日志记录

```python
import logging
logger = logging.getLogger(__name__)

# 调试日志
logger.debug(f"Processing node: {node_id}")

# 信息日志
logger.info(f"Workflow completed: {workflow_id}")

# 错误日志
logger.error(f"Failed to invoke model: {error}", exc_info=True)
```

### 2. 断点调试

```python
# 使用 pdb
import pdb; pdb.set_trace()

# 或使用 ipdb (更友好)
import ipdb; ipdb.set_trace()
```

### 3. 性能分析

```python
import time

start_time = time.perf_counter()
# 执行操作
result = expensive_operation()
elapsed = time.perf_counter() - start_time

logger.info(f"Operation took {elapsed:.2f} seconds")
```

## 扩展开发

### 添加新的模型提供商

1. 创建提供商目录：
   ```bash
   mkdir -p /api/core/model_runtime/model_providers/my_provider
   ```

2. 创建提供商类：
   ```python
   # my_provider.py
   from core.model_runtime.model_providers.__base.model_provider import ModelProvider
   
   class MyProvider(ModelProvider):
       def validate_provider_credentials(self, credentials: dict) -> None:
           # 验证提供商凭证
           pass
   ```

3. 创建模型类：
   ```python
   # llm/llm.py
   from core.model_runtime.model_providers.__base.large_language_model import LargeLanguageModel
   
   class MyLLM(LargeLanguageModel):
       def _invoke(self, ...):
           # 实现模型调用
           pass
   ```

4. 创建配置文件：
   ```yaml
   # my_provider.yaml
   provider: my_provider
   label:
     en_US: My Provider
   supported_model_types:
     - llm
   configurate_methods:
     - predefined-model
   ```

### 添加新的工作流节点

1. 创建节点目录：
   ```bash
   mkdir -p /api/core/workflow/nodes/my_node
   ```

2. 创建节点类：
   ```python
   # node.py
   from core.workflow.nodes.base import BaseNode
   
   class MyNode(BaseNode):
       @classmethod
       def get_node_type(cls) -> NodeType:
           return NodeType.MY_NODE
       
       def _run(self):
           # 实现节点逻辑
           pass
   ```

3. 注册节点：
   ```python
   # node_mapping.py
   NODE_TYPE_CLASSES_MAPPING = {
       NodeType.MY_NODE: MyNode,
       # ...
   }
   ```

### 添加新的工具

1. 创建工具目录：
   ```bash
   mkdir -p /api/core/tools/builtin_tool/providers/my_tool/tools
   ```

2. 创建工具类：
   ```python
   # my_tool.py
   from core.tools.builtin_tool.tool import BuiltinTool
   
   class MyTool(BuiltinTool):
       def _invoke(self, user_id: str, tool_parameters: dict):
           # 实现工具逻辑
           pass
   ```

3. 创建配置文件：
   ```yaml
   # my_tool.yaml
   identity:
     name: my_tool
     author: Your Name
   parameters:
     - name: param1
       type: string
       required: true
   ```

## 常见问题

### Q: 如何切换模型提供商？

A: 修改应用的模型配置，指定新的 provider 和 model：
```python
model_config = {
    "provider": "anthropic",
    "model": "claude-3-opus",
    "credentials": {...}
}
```

### Q: 如何调整工作流节点超时时间？

A: 在节点配置中设置 timeout 参数：
```python
node_config = {
    "timeout": 300  # 秒
}
```

### Q: 如何实现自定义工具？

A: 使用 API 工具功能，通过 OpenAPI schema 定义：
```python
from core.tools.custom_tool.provider import ApiToolProviderController

provider = ApiToolProviderController.from_openapi_schema(
    schema=openapi_schema,
    credentials=credentials
)
```

### Q: 如何监控模型调用？

A: 配置追踪系统（如 LangFuse）：
```python
# 在 .env 中配置
LANGFUSE_PUBLIC_KEY=your_key
LANGFUSE_SECRET_KEY=your_secret
LANGFUSE_HOST=https://cloud.langfuse.com

# 自动追踪所有模型调用
```

## 参考资源

- [Model Runtime 文档](./api/core/model_runtime/README.md)
- [Dify 架构分析](./DIFY_CORE_ANALYSIS.md)
- [架构图](./DIFY_CORE_ARCHITECTURE_DIAGRAMS.md)
- [贡献指南](./CONTRIBUTING.md)
- [API 文档](./api/README.md)

## 开发环境设置

```bash
# 1. 安装 UV
curl -LsSf https://astral.sh/uv/install.sh | sh

# 2. 创建虚拟环境
cd api
uv venv

# 3. 安装依赖
uv sync

# 4. 运行开发服务器
./dev/start-api

# 5. 运行 Celery Worker
./dev/start-worker

# 6. 运行测试
uv run --project api pytest

# 7. 代码格式化
./dev/reformat

# 8. 类型检查
uv run --directory api basedpyright
```

## 命令行快捷方式

```bash
# 后端开发
alias dify-api="cd /path/to/dify/api && uv run --project api"
alias dify-test="dify-api pytest"
alias dify-lint="cd /path/to/dify && ./dev/reformat"

# 前端开发
alias dify-web="cd /path/to/dify/web && pnpm"
alias dify-web-dev="dify-web dev"
alias dify-web-test="dify-web test"
```
