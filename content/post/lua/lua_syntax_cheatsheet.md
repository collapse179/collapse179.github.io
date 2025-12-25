---
title: LUA的语法、机制二三事
date: 2025-12-25
categories:
    - 热更新
comments: true
---
# Lua

## 一、Lua 数据类型（共 8 种）

### 1. nil
- 表示无值或空
- 所有未赋值变量默认是 nil

```lua
local a
print(a == nil)  -- true
```

### 2. boolean
- 只有两个值：`true` 和 `false`
- 只有 `false` 和 `nil` 被当作 false，其他如 `0` 和 `""` 都视为 true

```lua
if 0 then print("true") end  -- 会打印
```

### 3. number
- 默认使用双精度浮点数（Lua 5.3+ 区分 integer 和 float）

```lua
local a = 42
local b = 3.14
```

常用运算符：

| 运算 | 符号 | 示例 |
|------|------|------|
| 加法 | + | `a + b` |
| 减法 | - | `a - b` |
| 乘法 | * | `a * b` |
| 除法 | / | `a / b` |
| 除法 向下取整 | // | `a // b` |
| 取模 | % | `a % b` |
| 幂运算 | ^ | `2 ^ 3` |

### 4. string
- 字符串不可变，使用 `'`、`"`、`[[ ]]` 定义
- 使用 `..` 连接字符串

```lua
local s = "Hello" .. " World"
local longStr = [[
多行字符串
支持换行
]]
```

常用函数：

```lua
string.len(s)
string.sub(s, 2, 4)
string.find(s, "lo")
string.upper(s)
```

### 5. table
- 唯一的复合数据类型，可实现数组、字典、对象等结构

```lua
local t = {name = "Tom", age = 18}
local arr = {10, 20, 30}
```

遍历：

```lua
for i = 1, #arr do print(arr[i]) end
for k, v in pairs(t) do print(k, v) end
```

常用函数：

```lua
table.insert(t, "x")
table.remove(t, 1)
table.sort(t)
```

### 6. function
- Lua 函数是头等对象

```lua
function add(a, b)
    return a + b
end

local f = function(x) return x * x end
```

支持可变参数：

```lua
function sum(...)
    local args = {...}
    local s = 0
    for i = 1, #args do s = s + args[i] end
    return s
end
```

### 7. userdata
- 用于表示由 C/C++ 创建的数据，Lua 本身无法直接操作

### 8. thread
- 表示 Lua 协程，用于协作式多任务处理

```lua
local co = coroutine.create(function()
    print("Start")
    coroutine.yield()
    print("Resume")
end)

coroutine.resume(co)
coroutine.resume(co)
```

## 二、变量与作用域

```lua
x = 1        -- 全局变量
local y = 2  -- 局部变量
```

Lua 默认变量为全局，建议使用 `local` 限定作用域。

## 三、控制结构

### 条件语句

```lua
if a > 0 then
    print("正数")
elseif a == 0 then
    print("零")
else
    print("负数")
end
```

### 循环语句

```lua
while condition do
    -- 循环体
end

repeat
    -- 循环体
until condition

for i = 1, 10, 2 do
    print(i)
end

for k, v in pairs(tbl) do
    print(k, v)
end
```

## 四、操作符

### 比较运算符

- `==`, `~=`, `>`, `<`, `>=`, `<=`

### 逻辑运算符

- `and`, `or`, `not`

### 字符串连接符

- `..`

## 五、函数语法

```lua
function max(a, b)
    if a > b then
        return a
    else
        return b
    end
end

-- 匿名函数
local f = function(x) return x * 2 end

-- 多返回值
function ret()
    return 1, 2, 3
end

a, b = ret()  -- a = 1, b = 2
```

## 六、面向对象

```lua
Person = {}
Person.__index = Person

function Person:new(name)
    local self = setmetatable({}, Person)
    self.name = name
    return self
end

function Person:speak()
    print("Hi, I am " .. self.name)
end

local p = Person:new("Tom")
p:speak()
```

## 七、元表与元方法

```lua
local mt = {
    __add = function(a, b)
        return {value = a.value + b.value}
    end
}

local a = {value = 10}
local b = {value = 20}
setmetatable(a, mt)
setmetatable(b, mt)

local c = a + b
print(c.value)  -- 30
```

常用元方法：

- `__index`, `__newindex`
- `__add`, `__sub`, `__mul`, `__div`, `__mod`, `__pow`
- `__tostring`, `__eq`, `__lt`, `__le`
- `__call`

## 八、模块和 require

模块文件 mymod.lua：

```lua
local M = {}

function M.say()
    print("Hello from module")
end

return M
```

主文件：

```lua
local m = require("mymod")
m.say()
```

## 九、协程基础

```lua
local co = coroutine.create(function()
    print("First")
    coroutine.yield()
    print("Second")
end)

coroutine.resume(co)
coroutine.resume(co)
```

协程状态函数：

- `coroutine.status(co)`
- `coroutine.resume(co)`
- `coroutine.yield()`
- `coroutine.create(f)`
- `coroutine.wrap(f)`

## 十、标准库简述

| 模块 | 说明 |
|------|------|
| `math` | 数学运算 |
| `string` | 字符串处理 |
| `table` | 表操作 |
| `io` | 文件输入输出 |
| `os` | 系统接口 |
| `debug` | 调试工具 |
| `coroutine` | 协程支持 |

## 十一、注释

```lua
-- 单行注释

--[[
多行注释
]]
```

## 十二、LUA面向对象相关研究

- `setmetatable()` 把元表的一些元方法告诉赋值给普通表 
- `__index` 相当于一个索引。比如说查询一个普通表的时候发现没有对应键，定义了index之后就会去查询index的指向，如果指向函数，那么就执行函数，如果是一个表，则查询这个表的index,如果这个表的index指向自己那么就查询自己有没有对应键
- `__newindex` 相当于赋值操作。给普通表进行未定义的键值赋值的时候,会先查询有没有定义newindex，如果有的话则查询newindex指向，如果是函数那么就执行函数，如果指向一个表会先查询这个表有没有newindex定义,如果有则继续向上查找,如果没有则直接在这个表赋值,如果newindex指向自己会触发递归从而无法赋值

### 执行顺序相关

**元方法只在直接元表中查找**：

- 当对表进行操作时，Lua 只检查该表的**直接元表**中的元方法
- 不会递归查找元表的元表





