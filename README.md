# aether-models

[Aether](https://github.com/gaoyuyun/Aether) 网关的预设模型目录。网关启动时和之后每隔一段时间会拉取本仓库的 `models.json`，用于那些上游不提供模型列表接口的渠道类型；拉取失败时回退到编译进二进制的内嵌副本。

## 文件

- `models.json`：唯一的数据文件，被网关直接消费。

## 格式

```json
{
  "schema_version": 1,
  "updated_at": "2026-09-19",
  "providers": {
    "<provider_type>": [
      {
        "id": "模型 ID，作为请求里的 model 使用",
        "object": "model",
        "owned_by": "模型所属厂商",
        "display_name": "管理端展示名",
        "api_formats": ["该模型接受的 API 格式，例如 openai:chat / claude:messages / gemini:generate_content / openai:image"]
      }
    ]
  }
}
```

网关只接受下面这些 `provider_type`，其它键会被忽略并记录警告：

| provider_type | 说明 |
| --- | --- |
| `claude_code` | Claude Code OAuth 渠道，纯预设，不查上游 |
| `grok` | Grok 网页渠道，纯预设，不查上游 |
| `gemini_cli` | Gemini CLI 渠道，模型来自预设，上游只提供套餐元数据 |
| `kiro` | Kiro 渠道，正常实时查上游，这里只是 Key 没有端点时的兜底 |

校验规则：`id` 与 `api_formats` 必填且非空，同一渠道内 `id` 不能重复，任何一处不合法整份文件都会被拒绝并保留当前生效的目录。

## 更新流程

1. 修改 `models.json`，更新 `updated_at`。
2. 用 `python3 -m json.tool models.json > /dev/null` 确认 JSON 合法。
3. 提交到 `main`。所有网关实例会在下一次刷新周期（默认 3 小时）内自动生效，也可以在管理端手动刷新对应渠道的模型。

## 网关侧配置

| 环境变量 | 默认值 | 说明 |
| --- | --- | --- |
| `PRESET_MODEL_CATALOG_URLS` | raw.githubusercontent 与 jsDelivr 两个地址 | 逗号分隔，按顺序尝试，首个成功即停 |
| `PRESET_MODEL_CATALOG_REFRESH_MINUTES` | `180` | 刷新间隔，范围 30 到 10080 |
| `PRESET_MODEL_CATALOG_REFRESH_ENABLED` | `true` | 设为 `false` 只使用内嵌副本 |
