### 1. `.iml` 文件 (IntelliJ Module Library)

- **所属工具**：JetBrains IntelliJ IDEA（专属）
- **核心作用**：存储 IDEA 对当前模块的理解，包括源码目录、输出路径、模块级依赖等。
- **产生原因**：IDEA 导入 `pom.xml` 后，会将其转换为内部的 `.iml` 格式以便高效索引和编辑。
- **关键特性**：
    - 由 IDE 自动生成和维护
    - 内容可能被 `pom.xml` 覆盖（当你点击 "Reload Maven Project" 时）
    - 不同开发者的 IDEA 版本/设置可能导致文件差异
- **类比前端**：≈ `.vscode/settings.json` 或 `node_modules/.cache` 这类编辑器本地配置
- **是否提交 Git**：❌ **通常不提交**（应加入 `.gitignore`）