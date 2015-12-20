---
layout: post
title: "First-Class Functions"
date: 2015-12-14 07:36:49 +0000
comments: true
categories:
---

파이썬에서 function 들은 first-class 객체이다. first-class 객체는 아래와 같은 성질을 가진다.

- Created at runtime
- Assigned to a variable or element in a data structure
- Passed as an argument to a function
- Returned as the result of a function

즉, 함수를 다른 함수의 인자로 넘기거나 함수의 리턴 값으로 사용하는것이 가능하며 매우 자연스러운 일이라는 뜻. 함수가 first citizen 인 언어에서는 이를 이용한 다양한 기법들이 있으며 기발한 코딩이 가능하다. (데코레이터도 그중 하나이다)

#### 함수를 객체처럼 다루기

아래 예제는 python 콘솔세션에서 실행되었고 runtime 중에 함수가 생성되었다.

```python
>>> def factorial(n):
...     '''returns n!'''
...     return 1 if n < 2 else n * factorial(n-1)
...
>>> factorial(10)
3628800
>>> factorial.__doc__  # __doc__ 는 function objects 의 attributes (속성) 중 하나이다.
'returns n!'
>>> type(factorial)   # factorial 은 `function class` 의 객체이다.
<type 'function'>
```

아래 예제는 function 객체의 `first class` nature(성질) 을 보여준다. 함수 이름을 변수에 할당(assign) 하고 다른 함수의 인자로 넘긴다(map)

```python
>>> fact = factorial
>>> fact
<function factorial at 0x7f689058e578>
>>> fact(5)
120
>>> map(factorial, range(10))
[1, 1, 2, 6, 24, 120, 720, 5040, 40320, 362880]
>>>
```

#### Higher-Order Functions

함수를 인자로 받거나 함수를 리턴하는 함수를 `higher-order function` 이라고 한다. 위의 예제에서 본 map 함수가 좋은 예제이고 key argument 를 함수로 받는 `sorted` 역시 좋은 예제이다.

```python
>>> fruits = ['strawberry', 'fig', 'apple', 'cherry', 'raspberry', 'banana']
>>> sorted(fruits, key=len) # Any one-argument function can be used as the key including user-defined functions
['fig', 'apple', 'cherry', 'banana', 'raspberry', 'strawberry']
```

map, filter 등은 여전히 존재하는 higher-order 함수들이며 해당 함수들은 list comprehension 이나 general expression 으로 대체할 수 있다. 해당 함수들은 python2 에서는 list 를 리턴했으나 python3 에서는 generator (generator expression) - (a form of iterator) 를 리턴한다.

#### Anonymous Functions

`lambda` 키워드는 익명함수를 생성한다. 그러나 lambda 함수의 body 는 assignment 나 while, try 같은 구문을 사용할 수 없다. 가장 좋은 익명 함수의 사용예는 in the context of an argument list 이다. 아래 예제를 살펴봐라

```python
# lambda 예제
>>> a = lambda x: x ** 2
>>> a(5)
25
>>> b = lambda x, y: x + y
>>> b(1,2)
3
```

```python
>>> fruits = ['strawberry', 'fig', 'apple', 'cherry', 'raspberry', 'banana']
>>> sorted(fruits, key=lambda word: word[::-1])
['banana', 'apple', 'fig', 'raspberry', 'strawberry', 'cherry']
```

함수를 인자로 넘길때 anonymous function 으로 lambda 를 이용해서 넘길때 상당히 유용하지만 그외에는 거의 사용하지 않는다. 간지네

#### The Seven Flavors of Callable Objects

The call operator `()` 는 유저가 정의한 함수들을 외에도 다른 객체들에도 적용될 수 있다. 객체가 callable 한지 판단하기 위해서는 `callable()` 빌트인 함수를 사용하면 된다. Python Data Model 문서에서는 7 가지 callable type 을 리스트하였다.

1. User-defined functions: Created with `def` statements or `lambda` expressions
2. Built-in functions: A function implemented in C (for CPython), like `len` or `time.strftime`.
3. Built-in methods: Methods implemented C, like dict.get.
4. Methods: Functions defined in the body of a class
5. Classes:
  When invoked, a class runs its `__new__` method to create an instance, then `__init__` to initialize it, and finally the instance is returned to the caller. Because there is no new oeprator in Python, calling a class is like calling a function. (Usually calling a class creates an instance of the same class, but other behaviors are possible by overriding `__new__`.
6. Class instances: If a class defined a `__call__` method, then its instances may be invoked as functions.
7. Generator functions: Functions or methods that use the `yield` keyword. When called, generator functions return a genrator obejct.


#### User-Defined Callable Types

Python function 들은 real object 일뿐만 아니라 `__call__` 만 구현하면 arbitrary Python object 도 역시 function 처럼 행동한다.

```python
import random

class BingoCage:
    def __init__(self, items):
        ''' Accepts any iterable; build a local copy prevents unexpected side effects on any list passed as an argument
        '''
        self._items = list(items)
        random.shuffle(self._items)  # shuffle is guaranteed to work because self._items is a list

    def pick(self):
        ''' main method
        '''
        try:
            return self._items.pop()
        except IndexError:
            raise LookupError('pick from empty BingoCage')  # raise exception with custom message if self._items is empty

    def __call__(self):
        ''' Shortcut to bingo.pick(): bingo()
        '''
        return self.pick()

bingo = BingoCage(range(3))

#print bingo.pick()
# >>> 0

#print bingo()
# >>> 2

#print BingoCage(range(3))()
# >>> 0

print callable(bingo)
# >>> True
```

즉 `__callable` 만 구현하면 해당 객체를 callable 하게 만들 수 있고 callable operator `()` 를 이용해 호출할 수 있다.

Decorators must be functions, but is is sometimes convenient to be able to "remember" something between calls of the decorator (e.g., for memoization - caching the results of expensive computations for later use).  functions 을 생성하는 다른 방식으로는 closure 를 사용하는 방법이 있다.


#### Function Introspection

일반적인 사용자가 정의한 클래스의 객체처럼, a function uses the `__dict__` attribute to store user attributes assigned to it. (내가 정의한 `__xxx___` 함수들이 `__dict__` 를 이용해 저장되어 있음)

```python
>>> dir(factorial)
['__annotations__', '__call__', '__class__', '__closure__', '__code__',
'__defaults__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__',
'__format__', '__ge__', '__get__', '__getattribute__', '__globals__',
'__gt__', '__hash__', '__init__', '__kwdefaults__', '__le__', '__lt__',
'__module__', '__name__', '__ne__', '__new__', '__qualname__', '__reduce__',
'__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__',
'__subclasshook__']
>>>
```

Listing attributes of functions that don't exist in plain instances (bare function - user-defined object)

```python
>>> class C: pass
>>> obj = C()
>>> def func(): pass
>>> sorted(set(dir(func)) - set(dir(obj))) # 일반적인 객체에 존자하지 않는 function attributes
['__annotations__', '__call__', '__closure__', '__code__', '__defaults__',
'__get__', '__globals__', '__kwdefaults__', '__name__', '__qualname__']
>>>
```

그럼 여기서 드는 의문은 function 이랑 user-defined object 의 다른점이 뭐고? 대체 function 이 뭐고? functional programming specific 한것인가? functional programming  은 뭔데 그럼 method 는 뭐고? (method 는 클래스 안에 정의된 함수를 지칭하는듯?)

`__annotations__` (dict)
Parameter and return annotations

`__call__` (method-wrapper)
Implementation of the () operator; a.k.a. the callable object protocol

`__closure__` (tuple)
The function closure, i.e., bindings for free variables (often is None)

`__code__` (code)
Function metadata and function body compiled into bytecode

`__defaults__` (tuple)
Default values for the formal parameters

`__get__` (method-wrapper)
Implementation of the read-only descriptor protocol (see Chapter 20)

`__globals__` (dict)
Global variables of the module where the function is defined

`__kwdefaults__` (dict)
Default values for the keyword-only formal parameters

`__name__` (str)
The function name

`__qualname__` (str)
The qualified function name, e.g., Random.choice (see PEP-3155)


#### From Positional to Keyword-Only Parameters

One of the best features of Python functions is the extremely flexible parameter handling mechanism, enhanced with keyword-only arguments in Python 3. closely related are the use of \* and \*\* to explode i
terables and mappings into separate arguments when we call a function.

파이썬3 에서는 함수 파라미터로 키워드를 사용할수 있는데 그 조건은 이전 파라미터가 반드시 튜플 `*arg` 이어야 한다.

```python
def tag(name, *content, cls=None, **attrs):
    """Generate one or more HTML tags"""
    if cls is not None:
        attrs['class'] = cls
    if attrs:
        attr_str = ''.join(' %s="%s"' % (attr, value)
                           for attr, value
                           in sorted(attrs.items()))
    else:
        attr_str = ''
    if content:
        return '\n'.join('<%s%s>%s</%s>' %
                         (name, attr_str, c, name) for c in content)
    else:
        return '<%s%s />' % (name, attr_str)

>>> tag('p', 'hello', id=33)  # id=33 은 dict 로 저장된다
'<p id="33">hello</p>'

>>> print(tag('p', 'hello', 'world', cls='sidebar')) # cls 키워드를 넘겼고 cls 는 *content 튜플에 포함되지 않는다.
<p class="sidebar">hello</p>
<p class="sidebar">world</p>

>>> tag(content='testing', name='img') # content 라는 키워드로 넘겼으나 기존의 튜플 (*content) 에 할당되지 않고 dict 로 저장되었고 name 은 기존의 name 으로 할당되었다.
'<img content="testing" />'

```

위에서 본것처럼 `cls=None` 이렇게 디폴트 값을 주지 않고 키워드를 사용하려면 `*` 를 키워드 앞에 의미없는 파라미터로 두기만 하면 된다.

```python
>>> def f(a, *, b):
...    return a, b
...
>>> f(1, b=2)
(1, 2)
```










