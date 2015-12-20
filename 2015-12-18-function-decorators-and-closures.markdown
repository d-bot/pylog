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


