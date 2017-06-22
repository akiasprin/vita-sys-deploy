# Erlang

[TOC]

## 顺序编程

### 1. 模式匹配

    Lhs = Rhs

### 2. Atoms

    小写字母变量均为Erlang的原子数据类型，一个原子的值就是原子自身

### 3. Tuples, List, Map

1. 元组：{a, b}，使用模式匹配获取值

2. 列表：[A, B, C]，提供 **|** 语法，赋值左边分割，赋值右边合并

### 4. Module and Function

1. 模块

``` erlang
-module(name). %文件名为name.erl，导出name模块
-export([fac/1, mult/2]). %导出外部可见方法列表 [方法名/传入变量个数]
-import(lists, [map/2, sum/1]). %导入模块中的方法列表
```

2. 普通函数

``` erlang
sum(L) -> sum(L, 0).
sum([], N)    -> N;
sum([H|T], N) -> sum(T, H+N).

sum([])    -> 0;
sum([H|T]) -> H + sum(T).

fac(1) -> 1;
fac(N) -> N * fac(N - 1).

map(_, [])    -> [];
map(F, [H|T]) -> [F(H)|map(F, T)].
```

3. 匿名函数

``` erlang
Double = fun(X) -> 2 * X end.
Even   = fun(X) -> (X rem 2) =!= 0 end.

List   = [1, 2, 3, 4].
lists:map(Double, List). %[2,4,6,8]
lists:filter(Even, List). %[2, 4]
```

4. 自定义抽象流程控制

``` erlang
for(Max, Max, F) -> [F(Max)];
for(I, Max, F)   -> [F(I)|for(I+1, Max, F)].

for(1, 10, fun(I) -> I end).
```

5. 列表解析

``` erlang
[F(X) || X <- L]. %对于列表L中的每一个X计算F(X)值，构造一个新的列表
```

更常见的形式：

X可以为任意一个表达式，每个限定词(Qualifier)可以是一个生成器或者是一个过滤器

生成器通常写为**Pattern <- ListExpr**，其中ListExpr必须是一个对列表项求值的表达式

过滤器可以是一个谓词，也可以一个布尔表达式

``` erlang
[X || Quanlifier1, Qualfier2, ...].
```

6. 快速排序

[X] ++ [Y] 是列表连接符

``` erlang
qsort([]) -> [];
qsort([Pivot|T]) ->
    qsort([X || X <- T, X < Pivot])
    ++ [Pivot] ++
    qsort([X || X <- T, X >= Pivot]).
```

7. 变位词（全排列）

  X--Y是列表的分离操作符，它能从列表X中分离出元素Y

``` erlang
perms([]) -> [[]];
perms(L)  -> [[H|T] || H <- L, T <- perms(L--[H])].
```

### 4. 表达式与断言

1. 算术表达式

|操作|描述|参数类型|优先级|
|---|---|---|---|
|+X|+X|Number|1|
|-X|-X|Number|1|
|X*Y|X*Y|Number|2|
|X/Y|X/Y(浮点数相除)|Number|2|
|bnot X|对X按位取反|Integer|2|
|X div Y|X//Y(整除)|Integer|2|
|X rem Y|X%Y|Integer|2|
|X band Y|对X和Y按位取与|Integer|2|
|X+Y|X+Y|Number|3|
|X-Y|X-Y|Number|3|
|X bor Y|对X和Y按位取或|Integer|3|
|X bxor Y|对X和Y按位取异或|Integer|3|
|X bsl Y|X<<Y|Integer|3|
|X bsr Y|X>>Y|Integer|3|

2. 部分特殊比较表达式

|操作|描述|
|---|---|
|X==Y|X等于Y|
|X/=Y|X不等于Y|
|X=:=Y|X全等于Y|
|X=/=Y|X不全等于Y|

3. 断言

断言基本使用方式如下，以when/if引导的断言表达式

```erlang
max(X, Y) when X > Y -> X;
max(X, Y) -> Y.

F(A, B, C) when is_tuple(A), size(B) =:= C -> true;
F(A, B, C) -> false.

if
    Guard -> Expressions;
    Guard -> Expressions;
    Guard -> Expressions;

    true  -> Expressions;
end

```

### 记录

1. 记录模型构造：

``` erlang
-record(Name, { %保存为records.hrl
    key1 = value1,
    key2 = value2,
}).

rr("records.hrl"). %在shell里面读取对应文件的record模型
```

2. 创建记录：

``` erlang
X1 = #Name{}.
X2 = #Name{key1 = "KEVIN", key2 = "CATHLEEN"}.
```

3. 更新记录：

``` erlang
X3 = X2#Name{key2 = "JANE"}.
```

4. 提取记录：

``` erlang
#Name{key1 = K1, key2 = K2} = X2.
K1.
K2.

X2#Name.key1. %点语法
```

5. 在函数中对记录进行模式匹配并修改：

``` erlang
clear_status(#Name{key1=K1, key2=K2} = R) ->
    R#Name{key1=K2}.
```

### case/if

1. case表达式

``` erlang
case Expression of
    Pattern1 [when Guard1] -> Expr_seq1;
    Pattern2 [when Guard2] -> Expr_seq2;
end
```

比如说filter的实现

``` erlang
filter(P, [H|T]) ->
    case P(H) of
        true  -> [H|filter(P, T)];
        false -> filter(P, T)
    end;
filter(P, []) ->
    [].
```

2. if表达式

``` erlang
if
    Guard1 ->
        Expr_seq1;
    Guard2 ->
        Expr_seq2;
end
```

### BIF

内置函数就不展开讨论了，用的时候查文档即可

1. apply(Mod, Func, [Args..]).

2. -AtomTag(...)

3. -SomeTag(...)

4. 块表达式

``` erlang
begin
    Expr1,
    ...,
    ExprN
end
```

5. 宏

### 异常

1. try .. catch ..

``` erlang
try FuncOrExpressionSequence of
    Pattern1 [when Guard1] -> Expressions1;
    Pattern2 [when Guard2] -> Expressions2;
catch
    ExceptionType: ExPattern1 [when ExGuard1] -> ExExpressions1;
    ExceptionType: ExPattern2 [when ExGuard2] -> ExExpressions2;
after
    AfterExpressions
end
```

其编程惯例：

``` erlang
generate_exception(1) -> a;
generate_exception(2) -> throw(a);
generate_exception(3) -> exit(a);
generate_exception(4) -> {'EXIT', a};
generate_exception(5) -> erlang:error(a).


demo() ->
    [catcher(I) || I <- [1, 2, 3, 4, 5]].

catcher(N) ->
    try generate_exception(N) of
        Val -> {N, normal, Val} %此处Val为try求解的值
    catch
        throw:X -> {N, caught, thrown, X};
        exit:X  -> {N, caught, exited, X};
        error:X -> {N, caught, error, X};
    end.
```

2. catch

``` erlang
demo() -> 
    [{I, (catch generate_exception(I))} || I <- [1, 2, 3, 4, 5]].
```

## 并发编程