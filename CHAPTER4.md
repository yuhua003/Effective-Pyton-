# CHAPTER4 第四章

我们经常需要处理列表(list)、含有键值对的字典(dict),以及集合(set)等数据结构，并且要以这种处理逻辑为基础来构建程序。Python提供了一种特殊的写法，叫作推导(comprechension),可以简洁地迭代这些结构，并根据迭代结构派生出另一套数据。这种写法能够让我们用相当清晰的代码实现许多常见任务，而且还有一些其他好处。

Python把这种理念也运用到了函数上面，这就产生了生成器(generator)，它可以让函数每次返回一系列值中的一个。凡是可以使用迭代的任务都支持生成器函数(例如循环，带星号的表达式等)。生成器可以提升性能并减少内存用量，还可以让代码更容易读懂。

## 第27条 用列表推导取代map与filter

Python里面有一种很简洁的写法，可以根据某个序列或可迭代对象派生出一份新的列表。用这种写法写成的表达式，叫做列表推导(list commprehension)。假如我们要用列表中每个元素的平方构建一份新的列表。传统的写法是，采用下面这个简单的for循环来实现。

```pyhon
a = [1,2,3,4,5,6,7,8,9,10]
squares = []

for x in a:
    squares.append(x**2)
print(squares)

>>>
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

上面那段代码可以改用列表推导来写，这样可以在迭代列表的过程中，根据表达式算出新列表中的对应元素

```python
squares = [x**2 for x in a] # List comprehension
print(squares)

>>>
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

这种功能当然也可以用内置函数map实现，它能够从多个列表中分别取出当前位置上的元素，并把它们当作参数传递给映射函数，以求出新列表在这个位置上的元素值。如果映射关系比较简单(例如只是把一个列表映射到另一个列表)，那么用列表推导来写还是比用map简单一些，因为用map的时候，必须先把映射逻辑定义成lambda函数，这看上去稍微有点繁琐。

>alt = map(lambda x: x ** 2, a)

列表推导还有一个地方比map好，就是它能够方便地过滤原列表，把某些输入值对应的计算结果从输出结果中排除。例如，假设新列表只需要纳入原列表中那些偶数的平方值，那我们可以在推导的时候，在for x in a 这一部分的右侧写一个条件表达式。

```python
even_squares = [x**2 for x in a if x%2 == 0]
print(even_squares)
[4, 16, 36, 64, 100]
```

这种功能也可以通过内置的filter与map函数来实现，但是这两个函数相结合的写法要比列表推导难懂一些。

```python
alt = map(lambda x: x**2, filter(lambda x: x ** 2, a))
assert even_squares == list(alt)
```


字典与集合也有相应的推导机制，分别叫做字典推导(dictionary comprehension) 与集合推导(set comprehension)。编写算法时，可以利用这些机制根据原字典与原集合创建新字典与新集合。

```python
even_squares_dict = {x: x**2 for x in a if x % 2 == 0}
threes_cubed_set = {x**3 for x in a if x % 3 ==0 }


print(even_squares_dict)
>>>
{2: 4, 4: 16, 6: 36, 8: 64, 10: 100}

print(threes_cubed_set)
>>>
{216, 729, 27}
```

如果改用map与filter实现，那么还必须调用相应的构造器(constructor),这会让代码变得很长，需要分成多行才能写得下。这样看起来比较混乱，不如使用推导机制的代码清晰。

```python
alt_dict = dict(map(lambda x:(x,x**2),
...:                 filter(lambda x: x % 2 == 0,a)))

alt_set = set(map(lambda x: x**3,
...:               filter(lambda x: x % 3 == 0,a)))
```

>[!IMPORTANT]
>    - 列表推导要比内置的map与filter函数清晰，因为它不用另外定义lambda表达式。
>    - 列表推导可以很容易地跳过原列表中的某些数据，假如改用map实现，那么必须搭配filter才能实现。
>    - 字典与集合可以通过推导来创建。