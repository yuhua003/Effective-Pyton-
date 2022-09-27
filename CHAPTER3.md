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

现在，调用方拿到函数的返回值之后，不用先判断操作是否成功了。因为这次可以假设，只要能拿到返回值，就说明函数肯定顺利执行完了，所以只需要用try把函数包起来并在else块里处理运算结果就好(这种结构的详细用法，参见第63条)。

```python
x,y=5,2
try:
    result = careful_divide(x,y)
except ValueError:
    print('Invalid inputs')
else:
    print('Result is %.1f' % result)

Result is 2.5
```

这个办法也可以扩展到那些使用类型注解的代码中(参见第90条)，我们可以把函数的返回值指定为float类型，这样他就不可能返回None了。然而，Python采用的是动态类型与静态类型相搭配的gradual类型系统，我们不能在函数的接口上指定函数可能抛出哪些异常(有的编程语言支持这样的受异常检查(checked exception),调用方必须应对这些异常)。所以，我们只好把有可能抛出的异常写在文档里面，并希望调用方能够根据这份文档适当的捕获相关的异常(参见第84条)。

下面我们给刚才那个函数加类型注解，并为它编写docstring

```py
def careful_divide(a: float,b: float) -> float:
    """Divides a by b.
    
    Raises:
        ValueError: when the inputs cannot be divided.
    """
    try:
        return a/b
    except ZeroDivisionError as e:
        raise ValueError('Invalid inputs')
```

这样写，输入，输出与异常都显得很清晰，所以调用方出错的概率就变得很小了。

>[!IMPORTANT]
>   - 用返回值None表示特殊情况是很容易出错的，因为这样的值在表达式里面，没办法与0和空白字符串之类的值区别，这些值相当于False。
>   - 用异常表示特殊的情况，而不要返回None。让调用的这个函数的程序根据文档里写的异常情况做出处理。
>   - 通过类型注解可以明确禁止函数返回None，即便在特殊情况下，它也不能返回这个值。


## 第21条 了解如何在闭包里使用外围作用域中的变量

有时，我们要给列表中的元素排序，而且要优先把某个群组之中的元素放在其他元素的前面。例如，渲染用户界面时，可能就需要这样做，因为关键的消息和特殊的事件应该优先显示在其他信息之前。

实现这种做法的一种常见方案，是把辅助函数通过key参数传给列表中的sort方法(参见第14条)，让这个方法根据辅助函数所返回的值来决定元素在列表中的先后顺序。辅助函数先判断当前元素是否储在重要群组里，如果在，就把返回值的第一项写成0，让它能够排在不属于这个组的那些元素之前。

```py
In [1]: def sort_priority(values,group):
   ...:     def helper(x):
   ...:         if x in group:
   ...:             return (0,x)
   ...:         return (1,x)
   ...:     values,sort(key=helper)
```

这个函数可以处理比较简单的输入数据。

```python
numbers = [8,3,1,2,5,4,7,6]
group = {2,3,5,7}
sort_priority(numbers,group)
print(numbers)

>>>
[2, 3, 5, 7, 1, 4, 6, 8]
```

>[!NOTE]它为什么能够实现这个功能呢？这要分三个原因来讲：
>   - Python支持闭包(closure),这让定义在大函数里面的小函数也能引用大函数之中的变量。具体到这个例子，sort_priority函数里面的那个helper函数也能够引用前者的group参数
>   - 函数在Python里是头等对象(first-class object),所以你可以像操作其他对象那样，直接引用它们，把它们赋给变量、将它们当成参数传给其他函数，或是在in表达式与if语句里面对它比较，等等。闭包函数也是函数，所以，同样可以传给sort方法的key参数。
>   - Python在判断两个序列(包括元组)的大小时，有自己的一套规则。它首先比较0号位置的那两个参数，如果相等，那就比较1号位置的那两个元素；如果还相等，那就比较2号位置的那两个元素；依次类推，直到得出结论为止。所以，我们还可以利用那套规则让helper这个闭包函数返回一个元组，并把关键址表写为元组的首个元素以表示当前排序的值是否属于重要群组(0表示属于，1表示布属于)。


如果这个sort_priority函数还能告诉我们，列表里面没有位于重要群组之中的元素，那就更好了，因为这样可以让用户界面开发者更方便地做出相应处理。添加这样一个功能似乎相当简单，因为闭包函数本身就需要判断当前值是否处在重要群组之中，既然这样，那么不妨让它在发现这种值时，顺便把标志变量翻转过来。最后，让闭包外的函数地大函数(即sort_priority函数)返回这个标志变量，如果闭包函数当时遇到过这样的值，那么这个标志肯定是True

下面，试着用最直接的写法来实现

```python
def sort_priority2(numbers,group):
    found = False
    def helper(x):
        if x  in group:
            found = True #Seems simple
            return (0,x)
        return (1,x)
    numbers.sort(key=helper)
    return found
```

我们还是用刚才的输入数据来运行这个新函数

```python
found = sort_priority2(numbers,group)
print('Found:',found)

>>>
Found: False

print(numbers)
>>>
[2, 3, 5, 7, 1, 4, 6, 8]

```

排序结果没有我呢提，可以看到：在排过序的numbers里面，重要群组group里的那些元素(2,3,5,7)，确实出现在了其他元素前面。既然这样，那表示函数返回值的found变量就应该是True，但我们看到的确实False，这是为什么？

在表达式中引用某个变量时，Python解释器会按照下面的顺序，在各个作用域(scope)里面查找这个变量，以解析(resolve)这次引用。

1. 当前函数的作用域
2. 外围作用域(例如包含当前函数的其他函数所对应的作用域)
3. 包含当前代码的那个模块所对应的作用域(也叫做全局作用域，global scope)
4. 内置作用域(built-inscope,也就是包含len与str等函数的那个作用域)

如果这些作用域中都没有定义名称相符的变量，那么程序就抛出NameError异常。

```python
In [21]: foo = does_not_exist * 5

>>>
---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
<ipython-input-21-527dc1900e9d> in <module>
----> 1 foo = does_not_exist * 5

NameError: name 'does_not_exist' is not defined
```

刚才讲的是怎么引用变量，现在我们来讲怎么给变量赋值，这要分两种情况处理。如果变量已经定义在当前作用域中，那么直接把新值交给它就行了。如果当前作用域中不存在这个变量，那么即便外围作用域里面有同名的变量，Python也还是会把这次的赋值操作当作变量的定义来处理，这回产生一个重要的效果，也就是说，Python会把包含赋值操作的这个函数当成新定义的这个变量的作用域。

这可以解释刚才那种写法错在何处。sort_priority2函数里面的helper闭包函数是把True赋给了found变量。当前作用域里没有这样一个叫作found的变量，所以就算外围的sort。priority2函数里面有found变量，系统还是会把这次赋值当作定义，也就是会在helper里面定义一个新的found变量，而不是把它当成给sort_priority2已有的那个found变量赋值。

```python
def sort_priority2(numbers,group):
    found = False   #Scope: 'sort_priority2'
    def helper(x):
        if x in group:
            found = True # Scope: 'helper' --Bad
        return (1,x)
    numbers.sort(key=helper)
    return found
```

这种问题有时也称作用域bug(scoping bug)，Python新手可能认为这样的赋值规则很奇怪，但实际上Python是故意这么设计的。因为这样可以防止函数中的局部变量污染外部模块。假如不这样做，那么函数里的每条赋值语句都有可能影响全局作用域的变量，这样不仅混乱，而且会让全局变量之间彼此交互影响，从而导致很多难以探查的bug。

Python有一种特殊的写法，可以把闭包里面的数据赋给闭包外面的变量。用nonlocal语句描述变量，就可以让系统在处理针对这个变量的赋值操作时，去外围作用域查找。然而，nolocal有个限制，就是不能侵入模块级别的作用域(以防污染全局作用域)。

```python
def sort_priority3(numbers,group):
    found = False
    def helper(x):
        nonlocal found # Added
        if x in group:
            found = True
            return (0,x)
        return (1,x)
    numbers,sort(key=helper)
    return found
```

nolocal语句清楚的表明，我们要把数据赋给闭包之外的变量。有一种跟它互补的语句，叫作global，用这种语句描述某个变量后，在给这个变量赋值时，系统会直接把它放到模块作用域(或者说全局作用域)中。

我们都知道全局变量不应该滥用，其实nonlocal也这样。除比较简单的函数外，大家尽量不要用这个语句，因为它造成的副作用有时很难发现。尤其是在那种比较长的函数里。nonlocal语句与其关联变量的赋值操作之间可能隔得很远。

如果nolocal得用法比较复杂，那最好是改用辅助类来封装状态。下面就定义了这样一个类，用来实现与刚才那种写法相同的效果。这样虽然稍微长一点，但看起来更清晰易读(__call__这个特殊方法，请参见第38条)。

```python
class Sorter:
    def __init__(self,group):
        self.group = group
        self.found = False
    def __call__(self,x):
        if x in self.group:
            self.found = True
            return (0,x)
        return (1,x)

sorter = Sorter(group)
numbers.sort(key=sorter)
assert sorter.found is True
```

> [!IMPORTANT]
>   - 闭包函数可以引用定义它们的那个外围作用域之中的变量。
>   - 按照默认的写法，在闭包里面给变量赋值并不会改变外围作用域中的同名变量。
>   - 先用nonlocal语句说明，然后赋值，可以修改外围作用域中的变量。
>   - 除特别简单的函数外，尽量少用nonlocal语句。

## 第22条 用数量可变的位置参数给函数设计清晰的参数列表

让函数接受数量可变的位置参数(positional argument)，可以把函数设计得更加清晰(这些位置参数通常简称varargs)，或者叫作star args，因为我们习惯用*args指代)。例如，假设我们要记录调试信息。如果采用参数数量固定的方案来设计，那么函数应该接收一个表示信息的message参数和一个values列表(这个列表用于存放需要填充到信息里的那些值)。

```python
def log(message,values):
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{message}: {values_str}')


In [2]: log('My numbers are', [1,2])
>>>
My numbers are: 1, 2

In [3]: log('Hi there',[])
>>>
Hi there
```

即便没有值需要填充到信息里面，也必须专门传一个空白的列表进去，这样显得多余，而且让代码看起来比较乱。最好是能允许调用者把第二个参数留空。在Python里，可以给最后一个位置参数加前缀*，这样调用就只需要提供不带星号的那些参数，然后可以不再指其他参数，也可以继续指定任意数量的位置参数。函数的主题代码不用改，只修改调用代码即可。

```python
def log(message,*values): # The only difference
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{message}: {values_str}')


In [5]: log('My numbers are',1,2)
>>>
My numbers are: 1, 2

In [6]: log('Hi there') # Much better
>>>
Hi there
```

>[!NOTE]
>    - `.join()` 方法用于将序列中的元素以指定的字符连接生成一个新的字符串。这是是用','来做分割的。
>    - if not 的作用与 if 语句相反。它测试一个条件是否为False，然后执行一些语句。在python中 None, False, 空字符串"", 0, 空列表[], 空字典{}, 空元组()都相当于False ，即：
>        >not None == not False == not '' == not 0 == not [] == not {} == not ()
>    - `if not x` 等价于 `if x is None` 等价于 `if not x is None`

这种写法与拆解数据时用在赋值语句左边带星号的unpacking操作非常类似(参见第13条)。

如果想把已有的序列(例如某列表)里面的元素当成参数传给像log这样的参数个数可变的函数(variadic function)，那么可以在传递序列的时采用*操作符。这会让Python把序列中的元素都当成位置参数传给这个函数。

```python
favorites = [7,33,99]
log('Favorite colors', *favorites)

>>>
Favorite colors: 7, 33, 99
```

令函数接受数量可变的位置参数，可能导致两个问题。
第一个问题是，程序总是必须先把这些参数转化成一个元组，然后才能把它们当成可选的位置参数传给函数。这意味着，如果调用函数时，把带*操作符的生成器传了过去，那么程序必须先把这个生成器里的所有元素迭代完成(以便形成元组)，然后才能继续往下执行(相关知识，参见第30条)。这个元组包含生成器所给出的每个值，这可耗费大量内存，甚至会让程序奔溃。

```python
def my_generator():
    for i in range(10):
        yield i

 def my_func(*args):
     print(args)

it = my_generator()
>>>

my_func(*it)
>>>
(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
```

接受*args参数的函数，适合处理输入值不太多，而且数量可以提前预估的情况。在调用这种函数时，传给*args这一部分的应该时许多个字面值或变量名才对。这种机制，主要是为了让代码写起来更方便、读起来更清晰。

第二个问题是，如果用了*args之后，又要给函数添加新的位置参数，那么原有的调用操作就需要全部更新。例如给参数列表开头添加新的位置参数sequence，那么没有据此更新的那些调用代码就会出错。

```python
def log(sequence,message,*values):
    if not values:
        print(f'{sequence} - {message}')
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{sequence} - {message}: {values_str}')

log(1,'Favorites',7,33) # New with *args Ok
>>>
1 - Favorites: 7, 33

log(1,'Hi there') # New message only OK
>>>
1 - Hi there

log('Favorite numbers',7,33) # Old usage breaks
>>>>
Favorite numbers - 7: 33
```

问题在于： 第三次调用log函数的那个地方并没有根据新的参数列表传入sequence参数，所以`F啊v哦rite numbers`就成了sequence参数，7就成了message参数。这样的bug很难排查，因为程序不会抛出异常，指挥采用错误的数据继续运行下去。为了彻底避免这种漏洞，在给这种*arg函数添加参数时，应该使用只能通过关键字来指定的参数(keyword-only argument,参见第25条)。只要想做的更稳妥一些，可以考虑添加类型注解(参见第90条)。

>[!IMPORTANT]
>    - 用def定义函数时，可以通过*args的写法让函数接受数量可变的位置参数。
>    - 调用函数时，可以在序列左边加上\*操作符，把其中的元素当成位置参数传给*args所表示的这一部分。
>    - 如果*操作符加在生成器前，那么传递参数时，程序有可能因为耗尽内存而奔溃。
>    - 给接受*args的函数添加新位置参数，可能导致难以排查的bug。

## 第22条 用数量可变的位置参数给函数设计清晰的参数列表

</p>

让函数接受数量可变的位置参数(positional argument),可以把函数设计得更加清晰(这些位置参数通常简称varargs，或者叫作star args，因为我们习惯用*args指代)。例如，假设我们要记录调试信息。如果采用参数数量固定的方案来设计，那么函数应该接受一个表示信息的message参数和一个values列表(这个别表用于存放需要填充到信息里的那些值)。

```python
def log(message,values):
    if not values:
        print(message)
    else:
        values_str = ','.join(str(x) for x in values)
        print(f'{message}: {values_str}')

log('My numbers are',[1,2])
>>>
My numbers are: 1,2

log('Hi there',[])
>>>
Hi there
```

即便没有值需要填充到信息里面，也必须专门传一个空白的列表进去，这样显得多余，而且让代码看起来比较乱。最好是能允许调用者把两个参数留空。在Python里，可以给最后一个位置参数加前缀*,这样调用者就只需要提供不带星号的那些参数，然后可以不再指其他参数，也可以继续指定任意数量的位置参数。函数的主题代码不用改，只修改调用代码即可。

</p>

```python
def log(message,*values): # The only difference
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{message}: {values_str}')

print('My numbers are',1,2)
My numbers are 1 2

log('Hi there') # Much better
Hi there
```

这种写法与拆解数据时用在赋值语句左边带星号的unpacking操作非常类似(参见第13条)。

如果想把已有序列(例如某列表)里面的元素当成参数传给像log这样的参数个数可变的函数(variadic function)，那么可以在传递序列的时采用*操作符。这会让Python把序列种的元素都当成位置参数传给这个函数。

</p>

```python
favorites = [7,33,99]
log('Favorite colors',*favorites)

>>>
Favorite colors: 7, 33, 99
```

</p>
令函数接受数量可变的位置参数，可能导致两个问题。
第一个问题是，程序总是必须先把这些参数转化为一个元组，然后才能把它们当成可选的位置参数传给函数。这意味着，如果调用函数时，把带*操作符的生成器传了过去，那么程序必须先把这个生成器里的所有元素迭代完(以便形成元组)，然后才能继续往下执行(相关知识，参见第30条)。这个元组包含生成器所给出的每个值，这可能耗费大量内存，甚至会让程序崩溃。
</p>

```python
def my_generator():
    for i in range(10):
        yield i
def my_func(*args):
    print(args)
it = my_generator()
my_func(*it)

>>>
(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
```

接受*\args参数的函数，适合处理输入值不太多，而且数量可以提前预估的情况。在调用这种函数时，传给*args这一部分的应该时许多个字面值或变量名才对。这种机制，主要是为了让代码写起来更方便、读起来更清晰。

第二个问题是，如果用了*args之后，又要给函数添加新的位置参数，那么原有的调用操作就需要全部更新。例如给参数列表开头添加新的位置参数sequence，那么没有据此更新的那些调用代码就会出错。

</p>

```python
def log(sequence,message,*values):
    if not values:
        print(f'{sequence} - {message}')
    else:
        values_str = ','.join(str(x) for x in values)
        print(f'{sequence} - {message}: {values_str}')

log (1,'Favorites',7,33) # New with *args Ok

>>>
1 - Favorites: 7,33

log(1,'Hi there') # New  message only ok

>>>
1 - Hi there

log('Favorite numbers',7,33) # Old usage breaks

>>>
Favorite numbers - 7: 33
```

问题在于：第三次调用log函数的那个地方并没有根据新的参数列表传入sequence参数，所以`Favorite numbers`就变成了sequence参数，7就变成了message参数。这样的bug很难排查，因为程序不会抛出异常，指挥采用错误的数据继续运行下去。为了彻底避免这种漏洞，在给这种*arg函数添加参数时，应该使用只能通过关键字来指定的参数(keyword-only argument,参见第25条)。要是想做得更稳妥一些，可以考虑添加类型注解(参见第90条)

>[!IMPORTANT]
>   - 用def定义函数时，可以通过*args的写法让函数接受数量可变的位置参数。
>   - 调用函数时，可以在序列左边加上*/操作符，把其中的元素当成位置参数传给*args所表示的这一部分。
>   - 如果*操作符加在生成器前，那么传给参数时，程序有可能因为耗尽内存而崩溃。
>   - 给接受*args的函数添加新位置参数，可能导致难以排查的bug。

## 第23条 用关键字参数来表示可选的行为

与大多数其他编程语言一样，Python允许在调用函数时，按照位置传递参数。

```python
def remainder(number,divisor):
    return number % divisor

assert remainder(20,7) == 6
```

Python函数里面的所有普通参数，除了按位置传递外，还可以按关键字传递。调用函数时，在调用括号内可以把关键字的名字写在 = 左边，把参数卸载等号右边。这种写法不在乎参数的顺序，只要把必须指定的所有位置参数全部传过去即可。另外，关键字形式与位置形式也可以混用。下面这四种写法的效果相同:

> remainder(20,7)
> remainder(20,divisor=7)
> remainder(number=20,divisor=7)
> remainder(divisor=7,number=20)

如果混用，那么位置参数必须出现在关键字参数之前，否则就会出错。

```python
remainder(number=20,7)

>>>
  Cell In [3], line 1
    remainder(number=20,7)
                         ^
SyntaxError: positional argument follows keyword argument
```

每个参数只能指定一次，不能即通过位置形式指定，又通过关键字形式指定。

```python
remainder(20,number=7)

>>>
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
Cell In [4], line 1
----> 1 remainder(20,number=7)

TypeError: remainder() got multiple values for argument 'number'
```

如果有一份字典，而且字典里面的内容能够用来调用remainder这样的函数，那么可以把**运算符加在字典前面，这会让Python把字典里面的键值以关键字参数的形式传递给函数。

```python
my_kwargs = {
    'number': 20,
    'divisor': 7,
}
assert remainder(**my_kwargs) == 6
```

调用函数时，带**操作符的参数可以和位置参数或关键字参数混用，只要不重复指定就行。

```python
my_kwargs = {
    'divisor': 7,
}
assert remainder(number=20,**my_kwargs) == 6
```

也可以对多个字典分别施加**操作，只要这些字典所提供的参数不重叠就好。

```python
my_kwargs = {
    'number': 20,
}
other_kwargs = {
    'divisor': 7,
}
assert remainder(**my_kwargs,**other_kwargs) == 6
```

定义函数时，如果想让这个函数接受任意数量的关键字参数，那么可以在参数列表里写上万能形参**kwargs,它会把调用者传进来的参数收集合到一个字典里面稍后处理(第26条讲了一种特别合适这么做的情况)。

```python
def print_parameters(**kwargs):
    for key,value in kwargs.items():
        print(f'{key} = {value}')


print_parameters(alpha=1.5,beta=9,gamma=4)

>>> 
alpha = 1.5
beta = 9
gamma = 4
```

### 关键字参数的灵活用法可以带来三个好处

#### 第一个好处是

用关键字参数调用函数可以让初次阅读代码的人更容易看懂。例如，读到`remainder(20,7)`这样的调用代码，就不太容易看出谁是被除数number，谁是初始divisor，只有去查看remainder的具体实现方法才能明白。但如果改用关键字形式来调用，例如remainder(number=20，divisor=7)，那么每个参数的含义就相当明了。

#### 的第二个好处是 
它可以带有默认值，该值是在定义函数时指定的。在大多数情况下，调用者只需要沿用这个值就好了，但有时也可以明确指定自己想要传的值。这样能够减少重复代码，让程序看上去干净些。

例如，我们要计算液体流入容器的速率。如果这个容器带刻度，那么可以取前后两个时间点的刻度差(weight_diff)，并把它跟这两个时间点的时间差(time_diff)相除，就可以算出流速了。

```python
def flow_rate(weight_diff,time_diff):
    return weight_diff / time_diff

weight_diff = 0.5
time_diff = 3
flow = flow_rate(weight_diff,time_diff)
print(f'{flow:.3} kg per second')

>>>
0.167 kg per second
```

一般来说，我们会用美妙的千克数表示流速。但有时候，我们还想估算更长的时间段(例如几小时或几天)内的流速结果。只需给同一个函数加一个period参数来表示那个时间段相当于多少秒即可。

```python
def flow_rate(weight_diff,time_diff,period):
    return (weight_diff / time_diff) * period
```

这样写有个问题，就是每次调用函数时，都得明确指定period参数。即便我们想计算每秒钟的流速，也还得明确指定period为1。

> flow_per_second = flow_rate(weight_diff,time_diff,1)

为了简化这种写法，我们可以给period参数设定默认值。

    def flow_rate(weight_diff,time_diff,period=1):
        return (weight_diff / time_diff) * period

这样的话，period就变成了可选参数。

    flow_per_second = flow_rate(weight_diff,time_diff)
    flow_per_hour = flow(weight_diff,time_diff,period=3600)

这个办法适用与默认值比较简单的情况。如果默认值本身要根据比较复杂的逻辑来确定(可参考第24条)，那就得仔细考虑一下了。

#### 关键字参数的第三个好处

我们可以很灵活地扩充函数的参数，而不用担心会影响原有的函数调用代码。这样的话，我们就可以通过这些新参数的函数里面实现许多新的功能，同时又无须修改早前写好的调用代码，这也让程序不容易因此出现bug。

例如，我们想继续扩充上述flow_rate函数的功能，让它可以用千克之外的其他重量单位来计算流速。那只需要在添加一个可选参数，用来表示1千克相当于多少个那样的单位即可。

```python
def flow_rate(weight_diff,time_diff,
             period=1,units_per_kg=1):
    return((weight_diff * units_per_kg) / time_diff) * period
```

新参数`units_per_key`的默认值为1，这表示默认情况下，依然以千克为重量单位来计算。于是，以前写好的那些调用代码就不用修改了。以后调用flow_rate时，可以通过关键字形式给这个参数指定值，以表示他们想用的的那种单位(例如磅，1千克约等于2.2磅)。

```python
pounds_per_hour = flow_rate(weight_diff,time_diff,
                            period=3600,units_per_kg=2.2)
```

可选的关键字参数有助于维护向后兼容(backward compatibility)。这是个相当重要的问题，对于接受带*args参数的函数，也要注意向后兼容(参见第22条)。

像period和units_per_kg这样可选的关键字参数，只有一个缺点，就是调用者然然能够按照位置来指定。

> pounds_per_hour = flow_rate(weight_diff,time_diff,3600,2.2)

通过位置来指定可选参数，可能会让读代码的人有点儿糊涂，因为他不太清楚3600和2.2这两个值分别指哪个量的缩放系数。所以，最好是能以关键字的形式给这些参数传值，而不要按位置去传。从设计函数的角度来说，还可以考虑用更加明确的方案以降低出错概率(参见第25条)。

>[!IMPORTANT]
>   - 函数的参数可以按位置指定，也可以用关键字的形式指定。
>   - 关键字可以让每个参数的作用更加明了，因为在调用函数时只按位置指定参数，可能导致这些参数的含义不够明确。
>   - 应该通过带默认值的关键字参数来扩展函数的行为，因为这不会影响原有的函数调用代码。
>   - 可选的关键字参数总是应该通过参数名来传递，而不应按位置传递。

