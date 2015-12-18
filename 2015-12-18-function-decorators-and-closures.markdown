---
layout: post
title: "Function Decorators and Closures"
date: 2015-12-18 09:14:24 +0000
comments: true
categories: 
---

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

데코레이터는 다른 function 을 인자로 받는 callable 이다. (the decorated function). The decorator may perform some processing with the decorated function, and returns it or replaces it with another function or callable object.

```python

@decorate
def target():
  print('running target()')

# Has the same effect as writing this:

def target():
  print('running target()')

target = decorate(target)
```

The end of result is the same: At the end of either of these snippets, the target name does not necessarily refer to the original target function, but to whatever function is returned by `decorate(target)`.

To confirm that the decorated function is replaced, see the following console session
```python
def deco(func):
  def inner():
    print('running inner()')
  return inner  # deco returns its inner function object

@deco
def target(): # target is decorated by deco
  print('running target()')

target() # Invoking the decorated target actually runs inner
# running inner()


target # Inspection reveals that target is a now a reference to inner
# <function deco.<locals>.inner at 0x10063b598>
```

Strictly speaking, decorators are just syntactic sugar. As we just saw, you can always simply call a decorator like any regular callable, passing another function. Sometimes that is actually convenient, especially when doing metaprogramming - changing program behavior at runtime.

요약하면, 데코레이터는 decorated function 을 다른 function 으로 교체할 수 있다(replace) 그리고 데코레이터는 모듈이 로딩될때 즉시 실행된다 (they are executed immediately when a module is loaded).


