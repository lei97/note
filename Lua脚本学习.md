

##   Lua学习记录

[toc]

###  Hello world

> Lua 是类 C 语言，对大小写字符敏感。

```Lua
print("hello world")
```

> 类似于 Python，可以在进入 Lua 命令后进入 Shell 中执行语句。

```Lua
> print("hello world")
hello world
```

> 执行 Lua 文件
```shell
> lua file.lua
```

用 shell 执行 Lua 脚本

```shell 
chmod +x hello.lua
./hello.lua
```

***

###  语法

####  注释

```lua
-- 行注释

--[[
	多行注释
  --]]
```

####  变量

#####  实数

> Lua 的数字类型只有 **double** 类型，以下方式表示实数。

```lua
data = 100
data = 10.1
data = 3.321412
data = 323.1412e-2
data = 0.31416E1
data = 0xff
data = 0x52
```

#####  字符串

> 字符串可以使用单引号，双引号，支持 C 类型的转义，比如：'\a' （响铃），'\b' （退格），'\f' （表单）， ‘\n’ （换行）， ‘\r’ （回车）， ‘\t’ （横向制表）， ‘\v’ （纵向制表）， ‘\\’ （反斜杠）， ‘\”‘ （双引号）， 以及 ‘\” （单引号) 

```lua
-- 下面定义完全相同字符串
a = 'alo\n123"'
a = "alo\n123\""
a = '\97lo\10\04923"'
a = [[alo
123"]]
```

#####  布尔类型

> 在 Lua 语言中 **nil** 为 **NULL** ,其中没有申明过的变量就是 **nil** 。

##### 局部变量

> Lua 中没有特殊说明，全是全局变量，局部变量前面必须加 `local` 关键字。

```lua
globalVar = "global var"     -- 全局变量
local data = "local data"    -- 局部变量
```

#####  全局变量

> Lua 中的变量，如果没有 local 关键字，全都是全局变量，Lua 也是用 Table 来管理全局变量的，Lua 把这些全局变量放在了一个叫 “_G” 的 Table 里 。

```lua
_G.globalVar 
_G["globalVar"]
```

####  控制语句

#####  循环语句

```lua
-- 数据累加 while 循环
sum = 0
num = 1
while num <= 100 do
    sum = sum + num
    num = num + 1
end
print("sum =", sum)
```

```lua
-- for 循环
-- 0 - 100 累加
sum = 0
for i = 1, 100 do
    sum = sum + i
end

-- 0 -100 奇数和
sum = 0
for i = 1, 100, 2 do
    sum = sum + i
end

-- 100 - 1 偶数和
for i = 100, 1, -2 do
    sum  = sum + i
end
```

```lua
-- until 循环
sum = 2
repeat 
    sum  = sum ^ 2    -- 幂操作
    print(sum)
until sum > 1000
```

#####  判断语句

```lua
if a == 40 and b == 40 then 
    print("a and b is 40")
elseif a ~= b then    -- "~=" 是不等于
    print("a ~= b")
else
    print("a <= b")
    print("a is "..a)    -- 字符串拼接操作符..
```

####  函数

```lua
-- 递归函数
function fib(n)
  if n < 2 then 
      return 1 
  end
  return fib(n - 2) + fib(n - 1)
end
```

``` lua
-- 匿名函数 事例一
function newCounter()
    local i = 0
    return function()
        i = i + 1
        return i
    end
end

c1 = newCounter() 
print(c1())  -- 1
print(c1())  -- 2
```

```lua
-- 事例二
function myPower(x)
    return function(y)
        y ^ x
    end
end

power2 = myPower(2)
power3 = myPower(3)

print(power2(4)) -- 4 ^ 2
```

#####  函数返回值

> 在一条语句上可以赋多个值，如下：

```lua
name, age, bGay = "haoel", 37, false
-- 丢弃第二个数据
name = "tim", 34
```

> 函数同时可以返回多个数据，如下：

```lua
function getData(id)
	print(id)
    return "haoel", 37, "haoel@hotmail.com"
end

name, age, email = getData()
```

#####  局部函数

> 函数前面加上 **local** 就是局部函数

#### Table

> **Table** 是一个 Key Value 数据结构，类似于 Map，JavaScript 中的 Object

```lua
haoel = { name = "ChenHao", age = 37, handsome = True }
t = { [20] = 100, ['name'] = "chenHao", [3.14] = "PI"}
-- 进行访问
t[20]
```

```lua
-- 具体操作
haoel.web = "https://coolshell.cn/"
local age = haoel.age
haoel.name = nil
```

```lua
arr = {10, 20, 30, 40}
arr = {[1] = 10, [2] = 20, [3] = 30, [4] = 40}
arr = {"string", 100, "haoel", function() print("coolshell.cn" end)}
arr[4]() --进行函数调用
```

> Lua 的下标不是从 0 开始的，是从 **1** 开始的。

```lua
for i = 1, #arr do    -- #arr 表示 arr 长度
	print(arr[i])
end

-- 遍历Table
for k, v in pairs(t) do
    print(k,v)
end
```

####  MeteTable 和 MetaMethod

> MetaTable 类似于 C++ 重载操作符

```lua
fraction_a = {numerator = 2, denominator = 3}
fraction_b = {numerator = 4, denominator = 7}
```

```lua
-- 实现分数的相加
fraction_op = {}
function fraction_op.__add(f1, f2)
    ret = {}
    ret.numerator = f1.numerator * f2.denominator + f2.numerator * f1.denominator
	ret.denominator = f1.denominator * f2.denominator
    return ret
end

-- 为之前定义的两个 table 设置 MetaTable
setmetatable(fraction_a, fraction_op)
setmetatable(fraction_b, fraction_op)

-- 实现分数相加
fraction_s = fraction_a + fraction_b
```

> Lua 下 MetaMethod 约定：

```lua
__add(a, b)                     对应表达式 a + b
__sub(a, b)                     对应表达式 a - b
__mul(a, b)                     对应表达式 a * b
__div(a, b)                     对应表达式 a / b
__mod(a, b)                     对应表达式 a % b
__pow(a, b)                     对应表达式 a ^ b
__unm(a)                        对应表达式 -a
__concat(a, b)                  对应表达式 a .. b
__len(a)                        对应表达式 #a
__eq(a, b)                      对应表达式 a == b
__lt(a, b)                      对应表达式 a < b
__le(a, b)                      对应表达式 a <= b
__index(a, b)                   对应表达式 a.b
__newindex(a, b, c)             对应表达式 a.b = c
__call(a, ...)                  对应表达式 a(...)
```

####  面向对象

> __index 这个重载 （find key）。类似于 JavaScript 的 prototype。

> 两个对象 a 和 b，b 作为 a 的 prototype 需要设置 ` setmetatable(a, {__index = b}) `

```lua
-- 具体实例
window_Prototype = {x = 0, y = 0, width = 100, height = 100}
MyWin = {title = 'hello'}
setmetatable(MyWin, {__index = window_Prototype})
-- 可以在MyWin 访问x, y, width, height
```

Lua 面向对象

```lua
Person={}

function Person:new(p)
    local obj = p
    if (obj == nil) then
        obj = {name = "ChenHao", age = 37, handsome = true}
    end
    self.__index = self
    return setmetatable(obj, self)
end
 
function Person:toString()
    return self.name .." : ".. self.age .." : ".. (self.handsome and "handsome" or "ugly")
end
```

* self 就是 Person，Person:new(p)，相当于 Person.new (self , p)。
* new 方法的 self.__index = self 的意图是怕 self  被扩展后改写，所以，让其保持原样 。
* setmetatable 返回的是第一个参数的值。

```lua
-- 调用方法
me = Person:new()
print(me:tostring())

kf = Person:new{name="King's fucking", age=70, handsome=false}
print(kf:toString())
```

#####  继承

```lua
Student = Person:new()
 
function Student:new()
    newObj = {year = 2013}
    self.__index = self
    return setmetatable(newObj, self)
end
 
function Student:toString()
    return "Student : ".. self.year.." : " .. self.name
end
```

####  模块

> 使用 `require("model_name")` 导入 Lua 文件

* require函数，载入同样的 Lua 文件，只有第一次去执行。
* `dofile("hello")` 多次执行。
* `loadfile()` 函数为等你需要时才执行。

```lua
-- hello.lua
print("hello, world")

require("hello")  -- 输出为：hello, world
```

```lua
local hello = loadfile("hello")

-- 其他程序
hello()
```

```lua
-- MYMOD.Lua
local HaosModel = {}

local function getname()
    return "hao chen"
end

function HaoModel.Greeting()
    print("hello, my name is"..getname())
end

local hao_model = require("MYMOD")
hao_model.Greeting()

local hao_model = ( function ()
        -- MYMOD.Lua 文件内容
    end)()
```

