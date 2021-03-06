# Z3 API in Python

原文地址：http://www.cs.tau.ac.il/~msagiv/courses/asv/z3py/guide-examples.htm

Z3是微软研究院开发的一个高效的定理证明器。Z3可以被用于很多场景，例如：软件/硬件测试、约束求解、混合系统分析、安全、生物（药物设计）和几何问题。

这个教程展示了Z3Py的主要功能：python里的Z3 API。阅读此教程不需要Python基础。但学习Python很有用，同时也有很多的Python免费教程供我们学习（https://docs.python.org/3/tutorial/）

Z3同样包含了C、.net和OCaml的API。Z3Py的源代码可以在Z3发行版中看到，你可以按照你的需求进行修改。源代码也展示了如何使用Z3 4.0的新功能。其他酷炫的Z3前端有ScalaZ3和SBV。



###入门

让我们从下面这个例子开始：

```python
x = Int('x')
y = Int('y')
solve(x > 2, y < 10, x + 2*y == 7)
```

函数 **Int('x')** 定义了一个叫x的Z3整型变量， **solve** 函数求解这一约束系统。上面的例子用到了两个变量 **x** 和 **y** ，还有三个约束。Z3Py和Python一样使用 **=** 赋值。运算符 **<,<=,>,>=,==,!=** 用于比较。在上述例子中，表达式 **x+2*y==7** 是一个Z3约束。Z3可以求解并处理该等式。

下一个例子展示了如何用Z3公式/表达式 **simplifier**

```python
x = Int('x')
y = Int('y')
print simplify(x + y + 2*x + 3)
print simplify(x < y + x + 2)
print simplify(And(x + 1 >= 3, x**2 + x**2 + y**2 + 2 >= 5))
```

默认情况下，Z3Py（Web版）使用数学记号显示表达式。通常∧是逻辑与，∨是逻辑或。指令

```python
set_option(html_mode=False)
```
使所有公式和表达式以Z3Py的记号显示。这也是Z3发行版的默认模式。

```python
x = Int('x')
y = Int('y')
print x**2 + y**2 >= 1
set_option(html_mode=False)
print x**2 + y**2 >= 1py
```

Z3提供拆分表达式的函数

```python
x = Int('x')
y = Int('y')
n = x + y >= 3
print "num args: ", n.num_args()
print "children: ", n.children()
print "1st child:", n.arg(0)
print "2nd child:", n.arg(1)
print "operator: ", n.decl()
print "op name:  ", n.decl().name()
```

Z3支持所有基础数学运算。Z3Py使用与Python相同的运算优先级。如Python中，**是乘幂。Z3可以求解非线性多项式约束。

```python
x = Real('x')
y = Real('y')
solve(x**2 + y**2 > 3, x**3 + y < 5)
```

**Real('x')** 建立一个实变量x。Z3Py可以表示任意大的整数，有理数（如上面例子所示），代数数。代数数是指任何整系数多项式的复根。Z3内部精确的表示了每个数。无理数被显示为十进制使我们较容易读出结果。

```python
x = Real('x')
y = Real('y')
solve(x**2 + y**2 == 3, x**3 == 2)

set_option(precision=30)
print "Solving, and displaying result with 30 decimal places"
solve(x**2 + y**2 == 3, x**3 == 2)
```

**set_option** 指令用于设置Z3环境。它被用于设置全局属性，如结果如何显示等。指令

```python
set_option(precision=30)
```
设置显示结果时显示的十进制数有几位。数值 **1.2599210498?** 中的 **?** 标记表示输出是被截断的。

下面的例子展示了一个常见的错误。表达式 **3/2** 是个Python数值但不是Z3中的有理数。这个例子也展示了创建有理数的不同方法。函数 **Q(num,den)** 创建一个Z3有理数，其中num是分子，den是分母。**RealVal(1)** 创建一个表示数值1的Z3实数。

```python
print 1/3
print RealVal(1)/3
print Q(1,3)

x = Real('x')
print x + 1/3
print x + Q(1,3)
print x + "1/3"
print x + 0.25
```

有理数也可以以十进制显示

```python
x = Real('x')
solve(3*x == 1)

set_option(rational_to_decimal=True)
solve(3*x == 1)

set_option(precision=30)
solve(3*x == 1)
```

一个约束系统不一定有解，在这种情况下，我们称系统无法满足（unsatisfiable）

```python
x = Real('x')
solve(x > 4, x < 0)
```

跟Python一样，单行注释使用 **#** ，但Z3Py不支持多行注释

```python
# This is a comment
x = Real('x') # comment: creating x
print x**2 + 2*x + 2  # comment: printing polynomial
```

### 逻辑运算

Z3支持的逻辑运算符：And,Or,Not,Implies(蕴含),If(if-then-else)。双蕴含用 **==** 表示。下面的例子展示了如何求解一个布尔约束。

```python
p = Bool('p')
q = Bool('q')
r = Bool('r')
solve(Implies(p, q), r == Not(q), Or(Not(p), r))
```

Python中的布尔常量 **True** 和 **False** 可以用于Z3表达式。

```python
p = Bool('p')
q = Bool('q')
print And(p, q, True)
print simplify(And(p, q, True))
print simplify(And(p, False))
```

下面的例子同时使用了多项式和布尔表达式约束。

```python
p = Bool('p')
x = Real('x')
solve(Or(x < 5, x > 10), Or(p, x**2 == 2), Not(p))
```

### 求解器

Z3提供了不同的求解器。上述例子中使用的命令 **solve** 调用了Z3求解器的API。这些调用可以在Z3目录下的 **z3.py** 中找到。

```python
x = Int('x')
y = Int('y')

s = Solver()
print s

s.add(x > 10, y == x + 2)
print s
print "Solving constraints in the solver s ..."
print s.check()

print "Create a new scope..."
s.push()
s.add(y < 11)
print s
print "Solving updated set of constraints..."
print s.check()
	
print "Restoring state..."
s.pop()
print s
print "Solving restored set of constraints..."
print s.check()
```

**Solver()** 命令定义了一个通用的求解器。所有约束式可以通过 **add** 方法添加。我们称约束式在求解器中 **被声明**。**check()** 方法求解所有已添加进求解器的约束式。如果可以求得解，该方法返回 **sat** 。如果不能求出解则返回 **unsat** 。也有可能出现声明的约束式不成立的情况，这时求解器返回 **unknown** 。

在一些应用下，我们希望求解一些相似的问题，这些问题有部分同样的约束式。我们可以使用 **push** 和 **pop** 命令。在Z3中，每个求解器维护一个保存声明的栈， **push** 命令保存当前栈空间大小，并创建一个空间，压入新的声明。 **pop** 命令删除上一个 **push** 命令压入的所有声明。**check** 方法计算求解器栈内的所有约束式。

下面的展示了一个Z3不能求解的例子。这种情况下求解器返回**unknown** 。注意Z3可以求解非线性多项式约束，但 2**x 不是多项式。

```python
x = Real('x')
s = Solver()
s.add(2**x == 3)
print s.check()
```

下面的例子展示了如何将声明的约束式传入求解器，以及如何通过**check** 方法获取求解器状态。

```python
x = Real('x')
y = Real('y')
s = Solver()
s.add(x > 1, y > 1, Or(x + y > 3, x - y < 2))
print "asserted constraints..."
for c in s.assertions():
    print c

print s.check()
print "statistics for the last check method..."
print s.statistics()
# Traversing statistics
for k, v in s.statistics():
    print "%s : %s" % (k, v)
```

当求解器找到解时， **check** 方法返回 **sat** 。我们称Z3求出满足约束式的解。我们称这个解为所声明的约束式的一个模型。一个模型是指一种解释，能使所有约束式满足。下面的例子展示了查看模型的基本方法。

```python
x, y, z = Reals('x y z')
s = Solver()
s.add(x > 1, y > 1, x + y > 3, z - x < 10)
print s.check()

m = s.model()
print "x = %s" % m[x]

print "traversing model..."
for d in m.decls():
    print "%s = %s" % (d.name(), m[d])
```

在上面的例子中，函数 **Reals('x y z')** 创建了变量 x，y和z。这是下述写法的简写：

```python
x = Real('x')
y = Real('y')
z = Real('z')
```

表达式 **m[x]** 模型m中的返回x的值。表达式 **"%s = %s" % (d.name(), m[d])** 返回一个字符串，其中第一个%s被d对象的name属性替换，第二个%s被**m[d]** 的值替换（注：Python语法）。Z3Py会自动地将Z3对象的值全部转换为文本。

### 计算

Z3支持实型和整型变量。在单个问题中两种变量可以混合使用。与大多数编程语言一样，当我们需要时，Z3Py会自动地将整型表达式强制转换为实型。下面的例子展示了各种声明整型和实型变量的方法。

```python
x = Real('x')
y = Int('y')
a, b, c = Reals('a b c')
s, r = Ints('s r')
print x + y + 1 + (a + s)
print ToReal(y) + c
```

函数**ToReal** 将一个整型表达式转换为实型。

Z3Py支持所有的基本的运算符。

```python
a, b, c = Ints('a b c')
d, e = Reals('d e')
solve(a > b + 2,
      a == 2*c + 10,
      c + b <= 1000,
      d >= e)
```

函数**simplify** 提供了一些方法，可以对Z3表达式进行简单变换。

```python
x, y = Reals('x y')
# Put expression in sum-of-monomials form
t = simplify((x + y)**3, som=True)
print t
# Use power operator
t = simplify(t, mul_to_power=True)
print t
```

**help_simplify** 函数打印**simplify** 函数所有可用的参数。Z3Py允许用户以两种方式写入这些参数。Z3内部的参数名以**:** 开头并以**-** 隔开各个单词。这些参数可以被Z3Py使用。Z3Py也提供了一些符合Python变量命名规则的名字，在这里**:** 被移除，**-** 被**_** 取代。下面的例子展示了怎样使用以这两种方式调用函数。

```python
x, y = Reals('x y')
# Using Z3 native option names
print simplify(x == y + 2, ':arith-lhs', True)
# Using Z3Py option names
print simplify(x == y + 2, arith_lhs=True)

print "\nAll available options:"
help_simplify()
```

Z3Py可以支持任意大的数字。下面的例子展示了使用大数，有理数和无理数进行基本运算。无理数中，Z3Py只支持代数数。代数数已经足够表示多项式约束的解了。Z3Py总是以十进制显示无理数，因为这方便阅读。可以使用 **sexpr()** 方法提取Z3内部的表达式，它能以S-表达式（s-expression）的形式显示Z3内部的数学表达式。

```python
x, y = Reals('x y')
solve(x + 10000000000000000000000 == y, y > 20000000000000000)

print Sqrt(2) + Sqrt(3)
print simplify(Sqrt(2) + Sqrt(3))
print simplify(Sqrt(2) + Sqrt(3)).sexpr()
# The sexpr() method is available for any Z3 expression
print (x + Sqrt(y) * 2).sexpr()
```

### 硬件相关的运算

（注：此处为意译）

现代CPU和主流的编程语言使用基于固定大小的位向量的进行运算（注：如32位CPU采用基于位宽为32的数据进行运算）。在Z3Py中可以以位向量的方式进行这样的运算。位向量使我们可以看到有符号与无符号数的精确的补码表示。

下面的例子展示了如何创建位向量变量和常数。函数 **BitVec('x',16)** 在Z3中创建了一个16位，名字为 **x** 的变量。为了方便使用，整数常量可以用于初始化位向量。函数 **BitVecVal(10,32)** 创建了一个32位的，值为10 的位向量。

```python
x = BitVec('x', 16)
y = BitVec('y', 16)
print x + 2
# Internal representation
print (x + 2).sexpr()

# -1 is equal to 65535 for 16-bit integers 
print simplify(x + y - 1)

# Creating bit-vector constants
a = BitVecVal(-1, 16)
b = BitVecVal(65535, 16)
print simplify(a == b)

a = BitVecVal(-1, 32)
b = BitVecVal(65535, 32)
# -1 is not equal to 65535 for 32-bit integers 
print simplify(a == b)
```

其他编程语言，如C，C++，C#，Java的有符号与无符号数的各种运算对应的实际操作没有什么区别。但在Z3中，有符号数与无符号数使用不同的运算操作。 **<,<=,>,>=,/,%,>>** 运算对应有符号数，相对应的无符号数运算符为 **ULT,ULE,UGT,UGE,UDiv,URem,LShR** 

```python
# Create to bit-vectors of size 32
x, y = BitVecs('x y', 32)

solve(x + y == 2, x > 0, y > 0)

# Bit-wise operators
# & bit-wise and
# | bit-wise or
# ~ bit-wise not
solve(x & y == ~y)

solve(x < 0)

# using unsigned version of < 
solve(ULT(x, 0))
```

运算符 **>>** 是算术右移， **<<** 是算术左移。逻辑右移是 **LShR** 

```python
# Create to bit-vectors of size 32
x, y = BitVecs('x y', 32)

solve(x >> 2 == 3)

solve(x << 2 == 3)

solve(x << 2 == 24)
```

### 函数

不像其他编程语言，函数可以抛出异常或者无法返回。Z3的函数是完全定义的，即，函数是被所有输入的值所定义的，其中输入的值可以是一个函数，如划分函数。Z3是基于一阶逻辑的。

当我们给出约束式，如 **x+y>3** 时，我们称 **x** 和 **y** 是变量。在很多书中，**x** 和 **y** 称为未定义常量。即，它们可以被赋值为任何满足约束式 **x+y>3** 的值。

准确地说，一阶逻辑中的函数和符号常量是未定义的。这与很多其他理论中的函数相反，如算法中的函数 **+** 拥有标准的定义（将两数相加）。未定义函数和常量可以提供最大限度的灵活性：只要符合约束式，允许对函数和常量做出任何定义。（注：此段感觉翻译有误）

为了解释未定义的函数和常量的含义，我们定义了两个整数常量x和y。然后声明一个未进行定义的函数，这个函数有一个整型参数且返回值为一个整型。这个例子展示了我们可以使嵌套调用两次的f(x)等于x，即f(f(x)) == x

```python
x = Int('x')
y = Int('y')
f = Function('f', IntSort(), IntSort())
solve(f(f(x)) == x, f(x) == y, x != y)
```

返回的结果为：f函数的定义是：f(0)=1 f(1)=0 f(a)=1 当a≠0,1

（注：这段因为涉及到一阶逻辑的知识翻了好久还是一头雾水，不过看了下代码，这段大概的意思应该就是，Z3的输入可以是一个未定义具体内容的函数，它可以通过与该函数相关的一些约束式来求出符合要求的函数模型）

在Z3中，我们也可以计算在一系列约束式下的一些表达式的值。下面的例子说明了如何使用 **evaluate** 方法

```python
x = Int('x')
y = Int('y')
f = Function('f', IntSort(), IntSort())
s = Solver()
s.add(f(f(x)) == x, f(x) == y, x != y)
print s.check()
m = s.model()
print "f(f(x)) =", m.evaluate(f(f(x)))
print "f(x)    =", m.evaluate(f(x))
```

###可满足性和有效性

当一个约束式/公式 F的值在任何有效取值下始终为真，称该公式为普遍有效公式（即恒真公式）。当约束式在一些取值下为真则称公式为可满足的。验证有效性时，即需要找出命题为永真的证明。验证可满足性则是求出一个满足式子的集合。考虑一个包含 **a** 和 **b** 的公式 **F** 。**F** 是否普遍有效即是否在 **a** 和 **b** 的所有取值下**F** 都为真。如果**F** 是有效的，则**Not(F)** 为永假的，即**Not(F)** 在任何取值下都不为真，即，**Not(F)** 是不可满足的。所以公式**F** 在**Not(F)** 不可满足时为永真的。 同样，当且仅当 **Not(F)** 不为永真时，**F** 是可满足的。下面的例子证明了**德摩根定理** 。

下面的例子定义了一个Z3Py函数，它接收了一个公式作为参数。这个函数创建一个Z3求解器，并添加了传入参数的公式的**非** ，检查这个公式的非是否是不可满足的。下述的函数是Z3Py的命令**prove** 的简化版。

```python
p, q = Bools('p q')
demorgan = And(p, q) == Not(Or(Not(p), Not(q)))
print demorgan

def prove(f):
    s = Solver()
    s.add(Not(f))
    if s.check() == unsat:
        print "proved"
    else:
        print "failed to prove"

print "Proving demorgan..."
prove(demorgan)
```

### 列表解析式

Python支持列表解析式。列表解析式为创建列表提供了一个简便的方法。它们可以用于创建Z3表达式。下面的例子展示了如何在Z3Py中使用过列表解析式。

```python
 Create list [1, ..., 5] 
print [ x + 1 for x in range(5) ]

# Create two lists containg 5 integer variables
X = [ Int('x%s' % i) for i in range(5) ]
Y = [ Int('y%s' % i) for i in range(5) ]
print X

# Create a list containing X[i]+Y[i]
X_plus_Y = [ X[i] + Y[i] for i in range(5) ]
print X_plus_Y

# Create a list containing X[i] > Y[i]
X_gt_Y = [ X[i] > Y[i] for i in range(5) ]
print X_gt_Y

print And(X_gt_Y)

# Create a 3x3 "matrix" (list of lists) of integer variables
X = [ [ Int("x_%s_%s" % (i+1, j+1)) for j in range(3) ] 
      for i in range(3) ]
pp(X)
```

上面的例子中，表达式 **"x%s" %i** 返回一个由i的值替换%s的字符串。

命令**pp** 与**print** 类似，但它使用Z3用于格式化元组和列表的格式化字符串，而不是Python的。

Z3Py也提供创建布尔、整型和实型向量的函数，这些函数可以在列表解析式中被执行。

```Python
X = IntVector('x', 5)
Y = RealVector('y', 5)
P = BoolVector('p', 5)
print X
print Y
print P
print [ y**2 for y in Y ]
print Sum([ y**2 for y in Y ])
```



（注：下面内容为Z3在一些领域应用的示例）

###运动方程

在高中中，我们学习过运动方程。这些等式描述了位移、时间、加速度、初速度和末速度的关系。在Z3Py中可以表示为：

```python
 d == v_i * t + (a*t**2)/2
 v_f == v_i + a*t
```

#### 例题1

Ima Hurryin以30m/s的速度靠近红绿灯。此时灯变黄，Ima踩下刹车。如果Ima的加速度为-8.00m/s^2，请计算车辆在减速过程中的位移。

```python
d, a, t, v_i, v_f = Reals('d a t v__i v__f')

equations = [
   d == v_i * t + (a*t**2)/2,
   v_f == v_i + a*t,
]
print "Kinematic equations:"
print equations

# Given v_i, v_f and a, find d
problem = [
    v_i == 30,
    v_f == 0,
    a   == -8
]
print "Problem:"
print problem 

print "Solution:"
solve(equations + problem)
```

#### 例题2

Ben Rushin在等红绿灯。当灯变绿时，Ben以6.00m/s^2的加速度加速4.10秒。请计算这段时间Ben的位移。

```python
d, a, t, v_i, v_f = Reals('d a t v__i v__f')

equations = [
   d == v_i * t + (a*t**2)/2,
   v_f == v_i + a*t,
]

# Given v_i, t and a, find d
problem = [
    v_i == 0,
    t   == 4.10,
    a   == 6
]

solve(equations + problem)

# Display rationals in decimal notation
set_option(rational_to_decimal=True)

solve(equations + problem)
```

###位运算技巧

一些底层的位运算技巧在C程序员中很流行。我们试着用Z3实现其中一些技巧。

#### 2的幂

这个技巧常用于C程序（包括Z3源码）中测试一个整数是否是2的幂。我们可以用Z3证明这个算法的正确性。此处即证明 **x != 0 && !(x & (x - 1))** 为真，当且仅当**x** 为2的幂。

```python
x      = BitVec('x', 32)
powers = [ 2**i for i in range(32) ]
fast   = And(x != 0, x & (x - 1) == 0)
slow   = Or([ x == p for p in powers ])
print fast
prove(fast == slow)

print "trying to prove buggy version..."
fast   = x & (x - 1) == 0
prove(fast == slow)
```

####相反数

下面的技巧用于测试两个数是否互为相反数。

```python
x      = BitVec('x', 32)
y      = BitVec('y', 32)

# Claim: (x ^ y) < 0 iff x and y have opposite signs
trick  = (x ^ y) < 0

# Naive way to check if x and y have opposite signs
opposite = Or(And(x < 0, y >= 0),
              And(x >= 0, y < 0))

prove(trick == opposite)
```

### 数学问题

#### 猫狗和老鼠

我们花费100美元要购买100只动物。每只狗15美元，猫1美元，老鼠25美分。且必须每种至少买一只，求购买方案。

```python
# Create 3 integer variables
dog, cat, mouse = Ints('dog cat mouse')
solve(dog >= 1,   # at least one dog
      cat >= 1,   # at least one cat
      mouse >= 1, # at least one mouse
      # we want to buy 100 animals
      dog + cat + mouse == 100,  
      # We have 100 dollars (10000 cents):
      #   dogs cost 15 dollars (1500 cents), 
      #   cats cost 1 dollar (100 cents), and 
      #   mice cost 25 cents 
      1500 * dog + 100 * cat + 25 * mouse == 10000)
```

#### 数独

下面的例子展示了如何用Z3求解数独问题。其中**instance** 定义了数独矩阵。这个例子大量使用了Python中的列表解析式。

```
# 9x9 matrix of integer variables
X = [ [ Int("x_%s_%s" % (i+1, j+1)) for j in range(9) ] 
      for i in range(9) ]

# each cell contains a value in {1, ..., 9}
cells_c  = [ And(1 <= X[i][j], X[i][j] <= 9) 
             for i in range(9) for j in range(9) ]

# each row contains a digit at most once
rows_c   = [ Distinct(X[i]) for i in range(9) ]

# each column contains a digit at most once
cols_c   = [ Distinct([ X[i][j] for i in range(9) ]) 
             for j in range(9) ]

# each 3x3 square contains a digit at most once
sq_c     = [ Distinct([ X[3*i0 + i][3*j0 + j] 
                        for i in range(3) for j in range(3) ]) 
             for i0 in range(3) for j0 in range(3) ]

sudoku_c = cells_c + rows_c + cols_c + sq_c

# sudoku instance, we use '0' for empty cells
instance = ((0,0,0,0,9,4,0,3,0),
            (0,0,0,5,1,0,0,0,7),
            (0,8,9,0,0,0,0,4,0),
            (0,0,0,0,0,0,2,0,8),
            (0,6,0,2,0,1,0,5,0),
            (1,0,2,0,0,0,0,0,0),
            (0,7,0,0,0,0,5,2,0),
            (9,0,0,0,6,5,0,0,0),
            (0,4,0,9,7,0,0,0,0))

instance_c = [ If(instance[i][j] == 0, 
                  True, 
                  X[i][j] == instance[i][j]) 
               for i in range(9) for j in range(9) ]

s = Solver()
s.add(sudoku_c + instance_c)
if s.check() == sat:
    m = s.model()
    r = [ [ m.evaluate(X[i][j]) for j in range(9) ] 
          for i in range(9) ]
    print_matrix(r)
else:
    print "failed to solve"
```

#### 八皇后问题

八皇后问题：在8×8格的国际象棋上摆放八个皇后，使其不能互相攻击。即任意两个皇后都不能处于同一行、同一列或同一斜线上的摆法。

```python
# We know each queen must be in a different row.
# So, we represent each queen by a single integer: the column position
Q = [ Int('Q_%i' % (i + 1)) for i in range(8) ]

# Each queen is in a column {1, ... 8 }
val_c = [ And(1 <= Q[i], Q[i] <= 8) for i in range(8) ]

# At most one queen per column
col_c = [ Distinct(Q) ]

# Diagonal constraint
diag_c = [ If(i == j, 
              True, 
              And(Q[i] - Q[j] != i - j, Q[i] - Q[j] != j - i)) 
           for i in range(8) for j in range(i) ]

solve(val_c + col_c + diag_c)
```

### 应用：解决安装问题

常常用软件的安装问题通常是因为无法在系统上安装一组新包。Z3的这个应用适用于使用包管理器的系统。很多包都有其依赖。每个软件的发行版都包含一个**meta-data**文件用于指明该软件的依赖。**meta-data** 文件内包含依赖包的名字、版本等。更重要的是，它指明了软件的依赖包和有冲突的包。系统中不能安装那些有冲突的包。

Z3可以方便地解决安装问题。思路就是给每个包定义一个布尔变量。如果这个包必须被安装，则值为True。如果a包依赖于b，c和z，我们可以写为：

```python
DependsOn(a, [b, c, z])
```

**DependsOn** 用于获取依赖包的条目：

```python
def DependsOn(pack, deps):
   return And([ Implies(pack, dep) for dep in deps ])
```

因此，DependsOn(a, [b, c, z])返回下式的值

```python
And(Implies(a, b), Implies(a, c), Implies(a, z))
```

即，如果用户安装a包，则他们应该安装b，c和z。

如果d包与e包冲突，我们可以写为**Conflict(d, e)**

其中

```python
def Conflict(p1, p2):
    return Or(Not(p1), Not(p2))
```

函数返回表达式 **Or(Not(d), Not(e))** 。

通过这两个函数，我们可以把该问题描述为：

```python
def DependsOn(pack, deps):
    return And([ Implies(pack, dep) for dep in deps ])

def Conflict(p1, p2):
    return Or(Not(p1), Not(p2))

a, b, c, d, e, f, g, z = Bools('a b c d e f g z')

solve(DependsOn(a, [b, c, z]),
      DependsOn(b, [d]),
      DependsOn(c, [Or(d, e), Or(f, g)]),
      Conflict(d, e),
      a, z)
```

注意这个例子隐含了约束

```python
DependsOn(c, [Or(d, e), Or(f, g)])
```

意思就是要安装c，必须安装d或e，和f或g。

现在，我们优化一下上面的代码。首先我们定义了**DependsOn** ，可以通过**DependsOn(b, d)** 调用。我们还定义了 **install_check** ，用于返回必须在系统上安装的包的列表，即计算的结果。函数**Conflict** 用于检测冲突，可以接收多个参数。

```python
def DependsOn(pack, deps):
    if is_expr(deps):
        return Implies(pack, deps)
    else:
        return And([ Implies(pack, dep) for dep in deps ])

def Conflict(*packs):
    return Or([ Not(pack) for pack in packs ])

a, b, c, d, e, f, g, z = Bools('a b c d e f g z')

def install_check(*problem):
    s = Solver()
    s.add(*problem)
    if s.check() == sat:
        m = s.model()
        r = []
        for x in m:
            if is_true(m[x]):
                # x is a Z3 declaration
                # x() returns the Z3 expression
                # x.name() returns a string
                r.append(x())
        print r
    else:
        print "invalid installation profile"

print "Check 1"
install_check(DependsOn(a, [b, c, z]),
              DependsOn(b, d),
              DependsOn(c, [Or(d, e), Or(f, g)]),
              Conflict(d, e),
              Conflict(d, g),
              a, z)

print "Check 2"
install_check(DependsOn(a, [b, c, z]),
              DependsOn(b, d),
              DependsOn(c, [Or(d, e), Or(f, g)]),
              Conflict(d, e),
              Conflict(d, g),
              a, z, g)
```



### 在本地使用Z3Py

Z3Py是Z3发行版的一部分。在z3文件夹中**python** 子目录。为了在本地使用，必须在Python脚本中声明

from Z3 import *

z3 python的目录必须在环境变量**PYTHONPATH** 中。Z3Py会自动地搜索Z3运行库（z3.dll（WIN），libz3.so（Linux）或libz3.dylib（OSX））。你也可以使用下述命令手动初始化Z3Py：

```python
init("z3.dll")
```





ref:

原文：http://www.cs.tau.ac.il/~msagiv/courses/asv/z3py/guide-examples.htm

ScalaZ3：http://lara.epfl.ch/~psuter/ScalaZ3/

SBV：http://hackage.haskell.org/package/sbv