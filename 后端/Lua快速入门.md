# Lua 快速入门

> Lua 是一门轻量、可嵌入的脚本语言，常用于游戏逻辑（如 OpenResty/Nginx、Redis Lua、游戏引擎）、配置与扩展。语法极简、表（table）是唯一复合结构，适合作为宿主语言的「胶水层」。

## 简介与用途

| 点 | 说明 |
|---|---|
| **定位** | 嵌入式脚本 / 配置与业务扩展，不是通用应用主语言 |
| **特点** | 体积小、运行快、C API 友好、语法少 |
| **典型场景** | OpenResty / Nginx 动态逻辑、Redis `EVAL`、游戏（Unity/Unreal 插件、Roblox）、Wireshark、Neovim 配置 |
| **版本** | 常见 5.1（LuaJIT / OpenResty）、5.3/5.4（官方）；注意整数与位运算在 5.3+ 才原生支持 |

## 安装与运行

```bash
# Windows（scoop / chocolatey 任选）
scoop install lua
# 或
choco install lua

# 验证
lua -v

# 交互 REPL
lua

# 运行脚本
lua hello.lua
```

最小文件 `hello.lua`：

```lua
print("hello")
```

> 若走 OpenResty / Redis，用的是宿主自带的 Lua 运行时，不必单独装；API 以宿主文档为准。

## 基础语法

### 变量与类型

- 全局默认；`local` 限当前作用域（**优先用 local**）
- 动态类型；`nil` 表示「无值」
- 八种基本类型：`nil` / `boolean` / `number` / `string` / `table` / `function` / `userdata` / `thread`

```lua
local a = 1           -- number（5.3+ 分 integer / float）
local b = "hi"        -- string（不可变）
local c = true        -- boolean；只有 false 和 nil 为假
local d               -- 等价于 nil
print(type(a), type(b), type(d))  -- number  string  nil
```

字符串拼接用 `..`，长度用 `#`（对字符串、数组式表）：

```lua
print("a" .. "b")     -- ab
print(#"hello")       -- 5
```

### 表（table）——唯一复合结构

表既可当数组（下标从 **1** 开始），也可当字典 / 对象：

```lua
-- 数组部分（连续整数键从 1 起）
local arr = {10, 20, 30}
print(arr[1], #arr)   -- 10  3

-- 字典 / 对象
local user = {
  name = "wj",
  age = 20,
}
print(user.name, user["age"])

-- 混合
local t = { "a", "b", tag = "x" }
```

常用遍历：

```lua
-- 数组式：ipairs（遇 nil 停止）
for i, v in ipairs(arr) do
  print(i, v)
end

-- 全部键：pairs（顺序不保证）
for k, v in pairs(user) do
  print(k, v)
end
```

### 函数

```lua
local function add(x, y)
  return x + y
end

-- 多返回值
local function divmod(a, b)
  return a // b, a % b   -- // 为 5.3+ 整除；5.1 用 math.floor
end
local q, r = divmod(10, 3)

-- 可变参数
local function sum(...)
  local s = 0
  for _, v in ipairs({...}) do
    s = s + v
  end
  return s
end
```

函数是一等公民，可作表字段（方法）：

```lua
local obj = {
  n = 0,
  inc = function(self)
    self.n = self.n + 1
  end,
}
obj:inc()   -- 语法糖：obj.inc(obj)
```

### 条件与循环

```lua
local x = 5
if x > 10 then
  print("big")
elseif x > 0 then
  print("mid")
else
  print("small")
end

-- while
local i = 1
while i <= 3 do
  print(i)
  i = i + 1
end

-- 数值 for（含两端）
for i = 1, 5, 2 do   -- 1, 3, 5
  print(i)
end

-- repeat-until（至少执行一次；条件为真时结束）
repeat
  i = i - 1
until i <= 0
```

> 没有 `continue`；用 `goto`（5.2+）或把循环体包进函数/`do ... end` 规避。`and` / `or` 做短路，常写 `x = x or default`。

## 模块与 require

约定：一个模块返回一个 table（或函数）。

`mylib.lua`：

```lua
local M = {}

function M.greet(name)
  return "hi, " .. tostring(name)
end

return M
```

使用：

```lua
local mylib = require("mylib")   -- 按 package.path 找 mylib.lua
print(mylib.greet("lua"))
```

要点：

- `require` 有缓存（`package.loaded`），同名只加载一次
- 模块内用 `local`，只暴露返回值上的接口
- 路径：`package.path`（Lua）、`package.cpath`（C 扩展）
- OpenResty 等环境常用 `require("resty.xxx")`，路径由宿主配置

## 常见坑

| 坑 | 说明 |
|---|---|
| **下标从 1 开始** | `t[0]` 合法但不是「数组首元素」；`#t` 只看连续整数键 |
| **全局污染** | 忘写 `local` 会进 `_G`，多文件互相踩脚；开发可用 `luacheck` |
| **只有 false / nil 为假** | `0`、`""` 都是真 |
| **`#` 对稀疏表不可靠** | `{[1]=1,[3]=3}` 的 `#` 结果未定义；长度靠自己维护或 `table.maxn`（已弃用） |
| **`==` 与引用** | table / function 比的是引用，不是深相等 |
| **字符串不可变** | 大量 `..` 拼接用 `table.concat` |
| **版本差异** | 5.1 无 `//`、无位运算、`unpack` 在全局；5.3+ 在 `table.unpack`，有整数类型 |
| **`pairs` 顺序** | 非数组键遍历顺序不保证 |
| **`nil` 打断数组** | `ipairs` / `#` 在中间 `nil` 处停止或变短 |

## 最小示例

读标准输入两数求和（命令行脚本）：

```lua
-- sum.lua
local function main()
  local a = tonumber(arg[1])
  local b = tonumber(arg[2])
  if not a or not b then
    io.stderr:write("usage: lua sum.lua <a> <b>\n")
    os.exit(1)
  end
  print(a + b)
end

main()
```

```bash
lua sum.lua 3 5   # → 8
```

表 + 模块风格的小工具：

```lua
-- util.lua
local util = {}

function util.map(list, fn)
  local out = {}
  for i, v in ipairs(list) do
    out[i] = fn(v)
  end
  return out
end

return util
```

```lua
-- main.lua
local util = require("util")
local nums = {1, 2, 3}
local doubled = util.map(nums, function(x) return x * 2 end)
print(table.concat(doubled, ","))  -- 2,4,6
```

## 速查

| 需求 | 写法 |
|---|---|
| 局部变量 | `local x = 1` |
| 拼接 | `"a" .. "b"` |
| 长度 | `#s` / `#arr` |
| 默认值 | `x = x or 0` |
| 多返回 | `local a, b = f()` |
| 方法调用 | `obj:method(args)` |
| 加载模块 | `require("name")` |
| 类型 | `type(x)` |

---
上手顺序：装好 `lua` → 写 `local` + table → 用 `require` 拆模块 → 再对宿主 API（OpenResty / Redis / 游戏引擎）查文档。