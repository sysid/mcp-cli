# CHUK Tool Processor

An async-native toolkit to process, execute, and manage complex tool calls integrated with language models. It provides a robust and extensible system for:

- Parsing tool calls from text or structured input (LLM outputs)
- Executing tools with multiple strategies (in-process, subprocess, remote MCP)
- Wrapping executions with caching, rate limiting, and retry mechanisms
- Integrating with MCP (Multi-Channel Protocol) servers via stdio or SSE transports
- Registering and discovering tools dynamically, including LangChain integration
- Structured logging with contextual awareness and metrics collection
- Supporting streaming execution and results from tools

---

## Features

- **Extensible Parsing:** Multiple parser plugins handle various tool call formats (JSON, OpenAI function calls, XML tags, etc.)  
- **Execution Strategies:** Choose from in-process async execution or subprocess isolation  
- **Caching:** Transparent caching layer with TTL and cache invalidation support  
- **Rate Limiting:** Global and per-tool rate limiters with async-safe sliding window algorithms  
- **Retry Logic:** Automatic retries with exponential backoff and customizable policies  
- **MCP Integration:** Easy setup for MCP remote tools over stdio or SSE, with local wrappers  
- **Tool Registry:** Async-native registry supporting namespaces, metadata, and dynamic discovery  
- **Streaming Support:** Tools can expose streaming interfaces for incremental results  
- **Structured Logging:** JSON-formatted logs with async context tracking and metrics logging

---

## Installation

```bash
pip install chuk_tool_processor
```

*(Installation instructions for dependencies and MCP environment may vary)*

---

## Basic Usage

Create a `ToolProcessor` instance or use the default global processor to parse and execute tool calls:

```python
from chuk_tool_processor.core.processor import ToolProcessor

async def main():
    processor = ToolProcessor()
    text = "Please call the translate tool with args {\"text\": \"Hello\", \"target\": \"es\"}"
    results = await processor.process_text(text)
    for result in results:
        print(result)
```

Or process structured calls directly:

```python
calls = [
    {"tool": "weather", "arguments": {"location": "New York"}}
]
results = await processor.process(calls)
```

---

## MCP Integration

Set up a full MCP environment with StreamManager and remote tool registration:

```python
from chuk_tool_processor.mcp.setup_mcp_stdio import setup_mcp_stdio

processor, stream_manager = await setup_mcp_stdio(
    config_file="mcp_config.yaml",
    servers=["my_mcp_server"],
)
```

---

## Tool Registration

Register tools via decorators or programmatically:

```python
from chuk_tool_processor.registry.decorators import register_tool

@register_tool(name="add")
class AddTool:
    async def execute(self, x: int, y: int) -> int:
        return x + y
```

Or register plain Python functions asynchronously:

```python
from chuk_tool_processor.registry.auto_register import register_fn_tool

async def multiply(x: int, y: int) -> int:
    return x * y

await register_fn_tool(multiply, name="multiply")
```

---

## Caching, Rate Limiting, and Retries

Decorate tools with metadata to enable wrappers:

```python
from chuk_tool_processor.execution.wrappers.caching import cacheable
from chuk_tool_processor.execution.wrappers.rate_limiting import rate_limited
from chuk_tool_processor.execution.wrappers.retry import retryable

@cacheable(ttl=600)
@rate_limited(limit=10, period=60)
@retryable(max_retries=3)
class MyTool:
    async def execute(self, x: int) -> int:
        return x
```

---

## Logging and Metrics

CHUK uses structured JSON logging and captures rich contextual data for each operation. Logs and metrics can be integrated with your monitoring infrastructure.

---

## Security Considerations

When using CHUK Tool Processor, consider the following security aspects:

### 1. **Untrusted Input Execution**

- The system executes tools based on parsed calls from input strings or structures, which might originate from untrusted sources (e.g., user inputs or LLM responses).  
- **Risk:** Maliciously crafted inputs could trigger unintended tool executions or injection attacks if tools perform unsafe operations.  
- **Mitigation:**  
  - Validate and sanitize all user inputs before processing.  
  - Restrict accessible tools to a safe subset.  
  - Use argument validation (`pydantic` models) as enforced by the processor.  
  - Prefer isolated execution strategies (e.g., subprocesses) for higher security boundaries.

### 2. **Code Injection / Reflection**

- Tools may be dynamically imported or instantiated from class names or modules.  
- **Risk:** If dynamically loading tools or plugins from untrusted locations, attackers could inject malicious code.  
- **Mitigation:**  
  - Only load plugins from trusted packages or explicit registrations.  
  - Avoid loading tools from arbitrary user-provided module paths.  
  - The plugin discovery restricts untrusted imports, but review your environment carefully.

### 3. **Network Security**

- MCP transports use stdio or SSE over HTTP connections.  
- **Risk:** If MCP servers are untrusted or communications are unencrypted, intercepted data or unauthorized tool executions could occur.  
- **Mitigation:**  
  - Use secure channels (e.g., HTTPS for SSE transport).  
  - Use authentication and authorization controls on MCP servers.  
  - Validate and limit MCP tool registrations and calls.

### 4. **Resource Exhaustion**

- The system supports concurrency, caching, and retries which could be exploited by excessive calls.  
- **Risk:** Attackers could cause denial-of-service by flooding tool executions or cache entries.  
- **Mitigation:**  
  - Use the built-in rate limiting layers.  
  - Use concurrency limits to cap the number of running tasks.  
  - Monitor cache size and prune aggressively.

### 5. **Timeouts and Cancellation**

- The processor enforces timeouts and cancellation for long-running calls.  
- **Risk:** Misconfigured timeouts or unhandled cancellation logic could leave tasks hanging or cause resource leaks.  
- **Mitigation:**  
  - Configure sensible default timeouts.  
  - Use the provided shutdown procedures to gracefully cancel tasks.

### 6. **Third-Party Dependencies**

- This project uses various dependencies (e.g., `pydantic`, `httpx`, `asyncio`).  
- **Risk:** Vulnerabilities in dependencies might affect security.  
- **Mitigation:**  
  - Keep dependencies up to date with security patches.  
  - Use dependency scanning tools.

---

## Conclusion

CHUK Tool Processor offers a powerful asynchronous framework for tool invocation and execution in complex LLM integrations.

However, since it executes code and processes LLM-generated instructions, **careful security considerations are essential**, especially when inputs or plugins come from untrusted sources. Proper input validation, controlled plugin/tool registration, secure communications, and resource management are key to safe operation.

---

## License

This project is (specify your license here).

---

## Contributing

Please open issues or pull requests on the repository.

---

## Contact

For support or questions, contact (your contact info).

---

# End of README.md
