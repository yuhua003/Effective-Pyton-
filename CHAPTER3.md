# 第三章 函数

Python程序员首先应考虑的代码整理工具就是函数(function)。与其他编写语言一样，Python的函数也可以把大段程序分解为多个小块儿，而且用直观的名字表示每块儿代码的用途。这样可以让代码更好理解，也更容易复用与重构。

Python函数提供了许多能够简化编程工作的特性，其中有些和其他编程语言类似，还有一些是Python独有的。这些特性能够更明确地体现出函数地目标。利用这些特性来消除干扰，并突出调用者的相法，可以使程序里面不再出现那么多难以查找的bug

## 第19条 不要把函数返回的多个数值拆分到三个以上的变量中


unpacking机制允许Python函数返回一个以上的值(参见第6条)。假设现在要统计一群鳄鱼的各项指标，把每条鳄鱼的体长都保存在列表里。接着要编写函数，查出列表中最短和最长的鳄鱼。这可以用下面的这种写法实现，它可以同时把这两个值都返回。

```python
def get_stats(numbers):
    minimum = min(numbers)
    maximum = max(numbers)
    return minimum,maximum

lengths=[63,73,72,60,67,66,71,61,72,70]
minimum,maximum = get_stats(lengths)
print(f'Min: {minimum}, Max: {maximum}')
>>>
Min: 60, Max: 73
```

函数返回的其实是个元组(tuple)，同时返回的那两个值就是元组中的两个元素。程序通过把函数返回的元组赋给minimum与mazimum这两个变量来拆解元组，相当于用这两个变量分别接收元组中的两个元素。下面举一个更简单的例子，来演示unpacking语句和返回多个值的函数是怎么使用的。

```py
first,second = 1,2
assert first == 1
assert second == 2
def my_function():
    return 1,2
first,second = my_function()
assert first == 1
assert second == 2
```

在返回多个值的时候，可以用带星号的表达式接收那些没有被普通变量捕获的值(参见13条)。例如，我们还需要写一个函数，用来计算每条鳄鱼的长度与这些鳄鱼的平均长度之比。该函数会把比值放在列表里访问，但我们在接收的时候，可以只接收最长与最短的那两条鳄鱼所对应的比值，而把中间那些鳄鱼的比值用带星号的写法总括。

```py
lengths = [63,73,72,60,67,66,71,61,72,70]
def get_avg_ratio(numbers):
    average = sum(numbers) / len(numbers)
    scaled = [x/average for x in numbers]
    scaled.sort(reverse=True)
    return scaled
lengths = [63,73,72,60,67,66,71,61,72,70]
longest,*middle,shortest = get_avg_ratio(lengths)

>>>
print(f'Longest:{longest:>4.0%}')
Longest:108%
>>>
print(f'Shortest: {shortest:>4.0%}')
Shortest:  89%
```


> [!NOTE]
>    - 此处的 `=[x/average for x in numbers]` 应用了**python推导式**中的*列表推导式*，迭代将numbers(也就是lengths中的每个参数依次进行)的值赋值给x，使得运行x/average，并最终将结果赋值给`longest,*middle,shortest`
>    - `reverse=True`,指定可迭代对象中的一个元素来进行排序，模式为升序(severse=False),可指定降序(reverse=True)。

假设现在需求又变了,我们这次还想知道平均长度、中位长度(长度的中位)以及样本的总数。我们可以扩展原有的get_stats函数，让它把这些指标也计算出来，然后一并通过元组返回给调用方，让调用方自己去拆分

```py
def get_stats(numbers):
    minimum = min(numbers)
    maximum = max(numbers)
    count = len(numbers)
    average = sum(numbers) / count
    sorted_numbers = sorted(numbers)
    middle = count // 2
    if count % 2 == 0:
        lower = sorted_numbers[middle - 1]
        upper = sorted_numbers[middle]
        median = (lower + upper) / 2
    else:
         median = sorted_numbers[middle]
    return minimum,maximum,average,median,count


minimum,maximum,average,median,count = get_stats(lengths)

>>>
print(f'Min: {minimum},Max: {maximum}')
Min: 60,Max: 73
>>>
print(f'Average: {average},Median: {median},Count: {count}')
Average: 67.5,Median: 68.5,Count: 10
```

这样写有两个问题。首先，函数返回的五个值都是数字，所以很容易就会搞错顺序(例如，把平均数average和中位数media弄颠倒了)，这可能让程序出现难以查找的bug。调用方同时受到这么一大堆返回值，也特别容易出错。

```py
#Correct
minimum,maximum,average,median,count = get_stats(lengths)

#Oops! Median and average swapped!
minimum,maximum,median,average,count = get_stats(lengths)
```

第二个问题是，调用函数并拆分返回值的那行代码会写得比较长，所以按照PEP8风格指南，可能需要拆行(参见第2条)，这让代码看起来很别扭。

```python
minimum,maximum,average,median,count = get_stats(
    lengths)

minimum,maximum,average,median,count = \
    get_stats(lengths)

(minimum,maximum,average,
median,count) = get_stats(lengths)

(minimum,maximum,average,median,count
    ) = get_stats(lengths)
```

为避免这些问题，我们不应该把函数返回的多个值拆分到三个以上的变量里。一个元组最多只拆成三个普通变量，或两个普通变量与一个万能变量(带星号的变量)。当用于接收的变量个数也可以比这更少。假如要拆分的值确实很多，那最好还是定义一个轻量的类或namedtuple(参见第37条)，并让函数返回这样的实例。

> [!IMPORTANT]
>    - 函数可以把多个值喊起来通过一个元组返回给调用者，以便利用Python的unpacking机制去拆分。
>    - 对于函数返回的多个值，可以把普通变量没有捕获到的那些值全部捕获到一个带星的变量里。
>    - 把返回的值拆分到四个或四个以上的变量是很容易出错的，所以最好不要那么写，那是应该通过小类或namedtuple实例完成。


## 第20条 遇到意外状况时应该抛出异常，不要返回None

编写工具函数(utility function)时，许多Python程序员都爱用None这个返回值来表示特殊情况。对于某些函数来说，这或许有几分道理。例如，我们要偏写一个辅助函数计算两数相除的结果。在除数是0的情况下，返回None似乎相当合理，因为这种除法的结果是没有意义的。

```python
def careful_divide(a,b):
    try:
        return a/b
    except ZeroDivisionError:
        return None
```
调用这个函数时，可以按自己的方式处理这样的返回值

```py
x,y=1,0
result = careful_divide(x,y)
if result is None:
    print('Invalid inputs')

>>>
Invalid inputs
```

如果传给careful_fivide函数的被除数为0，会怎么样呢？在这种情况下，只要除数不为0，函数返回的结果就应该是0。问题是，这个函数的返回值有时可能会用在if条件语句里面，那时可能会根据值本身是否相当于False来做判断，而不像刚才那样明确判断这个值是否为None(第5条叶列举了类似的例子)。

```python
x,y=0,5
result=careful_divide(x,y)
if not result:
    print('Invalid inputs') #This runs! But shouldn't

>>>
Invalid inputs
```

上面这种if语句，会把函数返回0时的情况，也当成函数返回None时那样来处理，这种写法经常出现在Python代码里，因此像careful_divide这样，用None来表示特殊状况的函数很容易出错的。这两种办法可以减少这样的错误。

第一种办法是，利用二元组把计算结果分成两部分返回(与这种写法有关的基础知识，参见第19条)。元组的首个元素表示操作是否成功，第二个元素表示计算的实际值。

```python
def careful_divide(a,b):
    try:
        return True,a/b
    except ZeroDivisionError:
        return False,None
```

这样写，会促使调用函数者去拆分返回值，他可以先看看这次运算是否成功，然后再决定怎么处理运算结果。

```py
success,result = careful_divide(x,y)
if not success:
    print('Invalid imputs')
```

但问题是，有些调用方总喜欢忽略返回元组的第一部分(在Python代码里，习惯用下划线表示用不到的变量)。这样写成的代码，乍一看似乎没问题，但实际上还是无法区分返回0与返回None这两种情况。

```python
_,result = careful_divide(x,y)
if not result:
    print('Invalid inputs')

>>>
Invalid inputs
```

第二种方法比刚才那种更好，那就是不采用None表示特例，而是向调用方抛出异常(exception)，让他自己去处理。下面我们把执行除法时发生的ZeroDivisionError转化成ValueError告诉调用方输入的值不对(什么时候应该使用Exception类的子类，可参见第83条)。

```python
def careful_divide(a,b):
    try:
        return a/b
    except ZeroDivisionError:
        raise ValueError('Invalid inputs')
```

现在，调用方拿到函数的返回值之后，不用先判断