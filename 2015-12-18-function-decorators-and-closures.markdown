---
layout: post
title: "Function Decorators and Closures"
date: 2015-12-18 09:14:24 +0000
comments: true
categories: 
---

파이썬에서 함수는 first citizen에 속한다. 이 말은 파이썬에서 함수를 다른 함수의 인자로 넘기거나 함수의 리턴 값으로 사용하는 것이 가능하며 매우 자연스러운 일이라는 뜻이다. 함수가 first citizen인 언어에서는 이를 이용한 다양한 기법들이 있으며, 이를 통해 다양하고 기발한 코딩이 가능하다. 여기서 설명하는  데코레이터도 그 중에 하나이다.

`Function decorators` 는 소스코드에서 functions 의 기능을 여러 방면에서 좀 더 향상시킬 수 있도록 마킹해 놓는다. 데코레이터를 이해할려면 클로저에 대한 이해가 좀 필요하다.

파이썬에서 가장 최근에 예약된 키워드는 Python 3.0 에서 소개된 `nonlocal` 이다. `nonlocal` 은 클로저를 이해하게 되면 그 의미가 명확해 진다. 해당 챕터에서는 아래의 주제들을 다룰 예정이다.

How Python evaluates decorator synctax
How Python decides whether a variable is local
Why closures exist and how they work
What problem is solved by `nonlocal`

With this grounding, we can tackle further decorator topics:
Implementing a well-behaved decorator
Interesting decorators in the standard library
Implementing a parameterized decorator


#### Decorator 101

기본적으로 데코레이터 함수는 함수 객체를 인자로 받아서 새로운 함수를 리턴한다. 블락인듯..

```python
>>> def function():
...     print "hello world"
...
>>> function()
hello world
```

위와 같은 `function` 이라는 함수가 있고 decorator를 이용하여 해당 함수의 수행 시간을 계산하는 계산하는 프로그램으로 변경하면 아래와 같다.

```python
def decorator(func):
  def func_wrapper():
    print "welcome to"
    func()
    print "with python"
  return func_wrapper

# 이게 중요!! 즉 decorator 는 함수를 받아서 새로운 함수를 리턴하는것이다
# @decorator 를 사용하면 사실은 func_wrapper(func) 가 호출되는 것이다!!!

@decorator
def function()
  print "hello world"

function()

#welcome to
#hello world
#with python
```

데코레이터는 다른 function 을 인자로 받는 callable 이다. (the decorated function). The decorator may perform some processing with the decorated function, and returns it or replaces it with another function or callable object.

```python

@decorate
def target():
  print('running target()')

# Has the same effect as writing this:

def target():
  print('running target()')

target = decorate(target)
# 정확히 decorate 의 뼈대 함수가 뭔지 모르겠지만 empty decorator 라면 위의 라인이 이해가 됨
```

The end of result is the same: At the end of either of these snippets, the target name does not necessarily refer to the original target function, but to whatever function is returned by `decorate(target)`.

To confirm that the decorated function is replaced, see the following console session
```python
def deco(func):
  def inner():
    print('running inner()')
  return inner  # 즉 func 를 받아서 그거에 다른 코드를 추가해서 inner 를 리턴한다!!!

@deco
def target(): # target is decorated by deco
  print('running target()')

target() # Invoking the decorated target actually runs inner
# running inner()


target # Inspection reveals that target is a now a reference to inner
# <function deco.<locals>.inner at 0x10063b598>
```

Strictly speaking, decorators are just syntactic sugar. As we just saw, you can always simply call a decorator like any regular callable, passing another function. Sometimes that is actually convenient, especially when doing metaprogramming - changing program behavior at runtime.

요약하면, 데코레이터는 decorated function 을 다른 function 으로 교체할 수 있다(replace argument function with return function) 그리고 데코레이터는 모듈이 로딩될때 즉시 실행된다 (they are executed immediately when a module is loaded).

#### When Python Executes Decorators

데코레이터 (@) 가 붙으면 바로 해당 데코레이터는 실행된다. They run right after the decorated function is defined. That is usually at import time (when a module is loaded by Python). 아래 예제는 스크립트용 (`__name__ == '__main__'` 코드라 모듈이라는 단어가 어색할 수 있지만 일반적인 장식자 사용은 모듈에 정의된다. (뒤에서 설명하겠지만 보통 장식자와 장식되는 함수는 다른 모듈에 정의된다)

```python
# registration.py

registry = []

def register(func):
    print("Added %s") % func
    registry.append(func)
    return func


@register
def func1():
    print("FUNC 1")

@decorator
def func2():
    print("FUNC 2")


def func3():
    print("FUNC 3")

def main():
    print('runnng main()')
    print('registry ->', registry)
    func1()
    func2()
    func3()

if __name__ == '__main__':
    main()  # main() is only invoked if this runs as a script

#$ python registration.py
#running register(<function f1 at 0x100631bf8>)
#running register(<function f2 at 0x100631c80>)
#running main()
#registry -> [<function f1 at 0x100631bf8>, <function f2 at 0x100631c80>]
#running f1()
#running f2()
#running f3()
```

위의 결과에서 보듯이 `register` 장식자로 장식된 함수 `func1` 과 `func2` 는 정의되면서 바로 실행되어 버렸다. 이렇게 모듈이 로딩된 후, `registry` 배열은 2개의 장식된 함수를 가르키는 레퍼런스를 가지게 된다.

만약 위의 코드가 스크립트가 아닌 모듈로 정의되고 `import` 되었다면 아래와 같은 결과만 출력한다.
```python
>>> import registration
running register(<function f1 at 0x10063b1e0>)
running register(<function f2 at 0x10063b268>)
```

이부분이 `import time` 과 `runtime` 의 차이를 보여준다. 즉, 장식자 함수는 해당 모듈이 임포트 될때 실행되지만, 장식된 함수들은 run when they're explicitly invoked

위의 예제는 일반적이지 않다. 그 이유는 다음과 같다.
1. 장식자와 장식되는 함수가 같은 모듈안에 정의되어 있다. 실제로 이렇게 사용하지 않고 각자 다른 모듈에 정의한다.
2. The `register` 장식자는 인자로 받은 함수를 리턴하는데 보통 장식자 안에서 새로운 함수를 만들어서 리턴한다.

#### Decorator-Enhanced Strategy Pattern

Example 6-6 참고. 아래 코드는 장식자를 이용한 리팩토링인데 반복되는함수를 어떻게 줄여주는지 보여준다.

```python
promos = []   1

def promotion(promo_func):   2
    promos.append(promo_func)
    return promo_func

@promotion   3
def fidelity(order):
    """5% discount for customers with 1000 or more fidelity points"""
    return order.total() * .05 if order.customer.fidelity >= 1000 else 0

@promotion
def bulk_item(order):
    """10% discount for each LineItem with 20 or more units"""
    discount = 0
    for item in order.cart:
        if item.quantity >= 20:
            discount += item.total() * .1
    return discount

@promotion
def large_order(order):
    """7% discount for orders with 10 or more distinct items"""
    distinct_items = {item.product for item in order.cart}
    if len(distinct_items) >= 10:
        return order.total() * .07
    return 0

def best_promo(order):   4
    """Select best discount available
    """
    return max(promo(order) for promo in promos)
```

`promotion` 이라는 장식자는 함수를 받아서 배열에 넣고 해당 함수를 그대로 리턴하고 best_promo 함수는 promos 배열안의 함수들을 이용해서 베스트 프로모션을 찾는다.

일반적으로 장식자는 내부적으로 함수를 정의해서 인자로 받은 함수를 바탕으로 새로운 함수를 리턴하는데 그런 `inner function` 을 이해하기 위해서는 closure 가 제대로 동작해야하고, closure 전에 파이썬에서 어떻게 scope 가 동작하는지 살펴볼 필요가 있다.


#### Variable Scope Rules


```python
b=6

def f3(a):
  global b  # if global b is not declared, it will perform a lookup for local scope.
  print(a)
  print(b)  # This will error out if the global b is not declared since no b is declared in this local scope.
  b = 9

f3(3)
#3
#6
```

#### Closures

A closure is a function with an extended scope that encompasses non-global varioables referenced in the body of the function but not defined there.  익명 함수이든 아니든 관계없이, 중요한것은 클로저는 함수밖에 정의된 non-global variable 에 접근이 가능하다는 것이다.

Consider an avg function to compute the mean of an ever-increasing series of values; for example, the average closing price of a commodity over its entire history. Every day a new price is added, and the average is computed taking into account all prices so far.
Starting with a clean slate, this is how avg could be used:

```python
class Averager():
  def __init__(self):
    self.series = []

  def __call__(self, new_value):
    self.series.append(new_value)
    total = sum(self.series)
    return total/len(self.series)


# The Averager class creates instances that are callable:

>>> avg = Averager()
>>> avg(10)
10.0
>>> avg(11)
10.5
>>> avg(12)
11.0

```

Higher-order function 식으로 작성한 코드는 아래와 같다.  make_averager 는 averager 함수 객체를 리턴한다.

```python
def make_averager():
  series = []

  def averager(new_value):
    series.append(new_value)
    total = sum(series)
    return total/len(series)

  return averager   # averager 서브 함수 객체를 리턴하는데 어떻게 series = [] 에 계속 접근이 가능하지??

>>> avg = make_averager()
>>> avg(10)
10.0
>>> avg(11)
10.5
>>> avg(12)
11.0
```

`Averager` 클래스는 `avg` 객체를 생성하여 `series` 라는 instance attribute 를 메모리에 유지시키는데, 두번째 예제에서 `avg` 는 어떻게 `series` 를 찾고 접근하는지 알아보자.

두번째 예제에서 `series` 는 `make_averager` 의 local variable 이다. 즉, `series=[]` 는 make_averager 함수 안에서 초기화가 된다. 그러나 `avg(10)` 이 실행되었을때, `make_averager` 는 이미 리턴을 한 상태이고 local scope 역시 이미 없어져버린 상태이다.

두번째 예제에서 `averager` 함수 안에서 `series` 는 `free variable` 이다. 즉 해당 로컬 스코프에 존재하지 않는다는 뜻이다. This is a technical term meaning a variable that is not bound in the local scope.

```
def make_averager():

-----------------------------------
|  series = []                    |
|                                 |
|  def averager(new_value):       |----------> Closure
|    series.append(new_value) ----|-----> series: free variable
|    total = sum(series)          |
|    return total/len(series)     |
-----------------------------------

  return averager

```

리턴된 `averager` 함수 객체를 확인해 보면, 어떻게 local variable 과 free variable 이 `__code__` attribute 안에서 표현되는지 볼 수 있다.

```python
>>> avg = make_averager()
>>> avg.__code__.co_freevars
('series',)
>>> avg.__closure__
(<cell at 0x10bd9f980: list object at 0x10bd7a908>,)
>>> avg.__closure__[0].cell_contents
[]
>>> avg(10)
10
>>> avg(11)
10
>>> avg(12)
11
>>> avg.__closure__[0].cell_contents
[10, 11, 12]
```

To summarize: A closure is a function that retains the bindings of the free variables that exist when the function is defined, so that they can be used later when the function is invoked and the defining scope is no longer available.
Note that the only situation in which a function may need to deal with external variables that are nonglobal is when it is nested in another function.

#### The nonlocal Declaration



```python
def make_averager():
    count = 0
    total = 0

    def averager(new_value):
        count += 1          # local variable
        total += new_value  # local variable
        return total / count

    return averager

#If you try to run, here is what you get:
>>> avg = make_averager()
>>> avg(10)
Traceback (most recent call last):
  ...
UnboundLocalError: local variable 'count' referenced before assignment
>>>
```

count 와 total 은 `make_averager` 에서 immutable type 즉, integer 로 정의되었다 integer 를 포함한 모든 immutable 타입의 객체에 대해서는 값 `할당(assigning)` 이 되지 않고 위의 예제에서는 count 와 total 은 `averager 안에서 그냥 local variable 로 정의된다.

이전의 예제에서는 `series` 에 아무런 값을 할당하지 않았다. 그저 `series.append` 를 호출하고 sum 과 len 을 이용하였다. So we took advantage of the fact that lists are mutable

But with immutable types like numbers, strings tuples, etc., all you can do is read. but never update.


이 문제를 임시로 해결하기 위해서 `nonlocal declaration` 기법이 python 3 에서 소개되었다. It lets you flag a variable as a free variable even when it is assigned a new value within the function. If a new value is assigned to a nonlocal variable, the binding stored in the closure is changed. A correct implementation of our newest make_averager looks as follows



```python
def make_averager():
  count = 0
  total = 0

  def averager(new_value):
    nonlocal count, total
    count +=
    total += new_value
    return total / count

  return averager
```

#### Implementing a Simple Decorator


```python
import time

def clock(func):
  def clocked(*args):
    t0 = time.perf_counter()
    result = func(*args)  # works because the closure for clocked encompasses the func free variable
    elapsed = time.perf_counter() - t0
    name = func.__name__
    arg_str = ', '.join(repr(arg) for arg in args)
    print('[%8.8fs] %s(%s) -> %r' % (elapsed, name, arg_str, result))
    return result
  return clocked  # return the inner function to replace the decorated function

```

coloc 함수를 적절히 사용함.

```python
import time
from clockdeco import clock

@clock
def snooze(seconds):
  time.sleep(seconds)

@clock
def factorial(n):
  return 1 if n < 2 else n*factorial(n-1)

if __name__ == '__main__':
  print('*' * 48, 'Calling snooze(.123)')
  snooze(.123)
  print('*' * 48, 'Calling factorial(6)')
  print('6! =', factorial(6))


#************************************************ Calling snooze(.123)
#[0.12321678s] snooze(0.123) -> None
#************************************************ Calling factorial(6)
#[0.00000261s] factorial(1) -> 1
#[0.00024416s] factorial(2) -> 2
#[0.00045513s] factorial(3) -> 6
#[0.00066262s] factorial(4) -> 24
#[0.00081329s] factorial(5) -> 120
#[0.00096144s] factorial(6) -> 720
#6! = 720
```




