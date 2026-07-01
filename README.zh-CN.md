# Awesome LLMs

[English](./README.md) | [贡献指南](./CONTRIBUTING.zh-CN.md) | [共享文档](./docs/curation-policy.zh-CN.md)

这是一个按树形结构组织的 LLM 相关总仓库，在同一个 Git 边界下维护多个主题子目录。

## 结构说明

本仓库采用分层入口：

- 根 README：负责导航和共享规则
- 主题 README：负责说明主题范围、阅读入口和维护边界
- 主题 `docs/`：负责主题专属说明、primer、地图和深入文档

## 主题目录

- [DLLM](./DLLM/README.zh-CN.md) - Diffusion LLM 资源、技术说明和主题阅读路径
- [mcp](./mcp/README.zh-CN.md) - MCP 生态资源、入门说明和发布检查清单
- [skills](./skills/README.zh-CN.md) - LLM / Agent skill 资源、生态地图和编写参考

## 共享文档

以下文档由总仓库统一维护，主题目录不再各自保留重复副本：

- [docs/curation-policy.md](./docs/curation-policy.md)
- [docs/curation-policy.zh-CN.md](./docs/curation-policy.zh-CN.md)
- [docs/link-check.md](./docs/link-check.md)
- [docs/link-check.zh-CN.md](./docs/link-check.zh-CN.md)
- [docs/resource-template.md](./docs/resource-template.md)
- [docs/resource-template.zh-CN.md](./docs/resource-template.zh-CN.md)

## 阅读顺序

1. 先从本 README 选择要进入的主题。
2. 再进入对应主题 README 了解范围和主入口。
3. 最后进入该主题自己的 `docs/` 阅读更细的主题文档。

## 说明

- `CONTRIBUTING.md` 和 `CONTRIBUTING.zh-CN.md` 统一放在仓库根目录。
- 各主题目录只保留主题内容和双语 README 入口。
- GitHub 的 Markdown 公式渲染有时不稳定。遇到公式较多的文档在线显示异常时，建议下载或 clone 仓库后在本地查看 Markdown。
- 所有 Git 操作都应在 `awesome-LLMs/` 总仓库边界理解和执行。
