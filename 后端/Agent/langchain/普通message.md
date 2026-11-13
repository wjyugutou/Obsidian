```python

@lru_cache
def get_ds_model(*, effort: Literal["none", "low", "high", "max"] = "high") -> ChatDeepSeek:
    """懒加载并缓存 DeepSeek 聊天模型实例。"""
    return ChatDeepSeek(
        model="deepseek-flash",
        api_key=AgentConfig["DEEPSEEK_API_KEY"],
        max_tokens=None,  # 不限制单次输出 token 数量
        max_retries=2,
        reasoning_effort=effort,
    )
    
get_ds_model().invoke([AIMessage])

```