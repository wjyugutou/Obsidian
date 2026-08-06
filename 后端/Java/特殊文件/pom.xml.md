### 1. `pom.xml` (Project Object Model)

- **所属工具**：Apache Maven（Java 生态最主流的构建/依赖管理工具）
- **核心作用**：定义项目“是什么”以及“需要什么”。它是跨 IDE 通用的标准配置文件。
- **主要内容**：
    - **项目坐标**：`groupId`, `artifactId`, `version`（GAV 三元组，唯一标识一个项目）
    - **依赖管理**：`<dependencies>` 声明第三方库（类似前端的 `package.json`）
    - **构建插件**：编译、测试、打包等生命周期配置
    - **多模块管理**：父工程与子工程的继承关系
- **类比前端**：≈ `package.json` + `vite.config.ts` / `webpack.config.js` 的结合体
- **是否提交 Git**：✅ **必须提交**，这是团队协作的基础
