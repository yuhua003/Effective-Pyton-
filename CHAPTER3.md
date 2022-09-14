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
