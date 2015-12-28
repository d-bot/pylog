---
layout: post
title: "Object References"
date: 2015-12-27 21:45:08 +0000
comments: true
categories:
---

#### Variables Are Not Boxes


she would say: “Variable s is assigned to the seesaw,” but never “The seesaw is assigned to variable s.” With reference variables, it makes much more sense to say that the variable is assigned to an object, and not the other way around. After all, the object is created before the assignment. Below example proves that the righthand side of an assignment happens first.

```python
>>> class Gizmo:
...    def __init__(self):
...         print('Gizmo id: %d' % id(self))
...
>>> x = Gizmo()
Gizmo id: 4301489152  1
>>> y = Gizmo() * 10  2
Gizmo id: 4301489432  3
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  TypeError: unsupported operand type(s) for *: 'Gizmo' and 'int'
  >>>
  >>> dir()  4
  ['Gizmo', '__builtins__', '__doc__', '__loader__', '__name__',
  '__package__', '__spec__', 'x']
```

1. The output Gizmo id: ... is a side effect of creating a Gizmo instance.
2. Multiplying a Gizmo instance will raise an exception.
3. Here is proof that a second Gizmo was actually instantiated before the multiplication was attempted.
4. But variable y was never created, because the exception happened while the right-hand side of the assignment was being evaluated.
  TIP

파이썬에서 Assignment 를 이해하기위해서는 항상 오른쪽 표현식을 먼저 읽어라: That's where the object is created or retrieved. After that, the variable on the left is bound to the object, like a label stuck to it. Because variables are more labels, nothing prevents an object from having several labels assigned to it.


#### Identity, Equality and Aliases

```python
>>> charles = {'name': 'Charles L. Dodgson', 'born': 1832}
>>> lewis = charles
>>> lewis is charles
True
>>> id(charles), id(lewis)
(140476663588872, 140476663588872)
>>> lewis['balance'] = 950
>>> charles
{'born': 1832, 'balance': 950, 'name': 'Charles L. Dodgson'}

# 동일한 값을 이요애서 alex 라는 변수를 새로 할당해보고 비교해보자.
>>> alex = {'name': 'Charles L. Dodgson', 'born': 1832, 'balance': 950}
>>> charles == alex
True
>>> charles is alex
False
>>> id(charles), id(alex)
(140476663588872, 140476663106952)
```

위에서 보다시피 lewis 는 charles 의 `alias` 이다. `==` 오퍼레이터는 객체의 값을 비교하는 연산자이지만 `is` 는 객체의 `identity`를 비교한다. If you are comparing a variable to a singleton, then it makes sense to use `is`. By far, the most common case is checking whether a variable is bound to `None`. This is recommended way to do it

```python
x is None

# 반대는

x is not None
```

`is` 연산자는 `==` 연산자보다 빠르다 왜냐하면 `is` 는 overloading 이 안되기 때문이다 즉, Python does not have to find and invoke special methods to evaluate it, and computing is as simple as comparing two integer IDs. 반대로 `a == b`  는 syntactic sugar for `a.__eq__(b)`. The `__eq__` method inherited from `object` compares object IDs, so it produces the same result as `is`. But most built-in types override `__eq__` with more meaningful implementations that actually take into account the values of the object attributes. Equality may involve a lot of processing - for example, when comparing large collections or deeply nested structures.

#### The Relative Immutability of Tuples

튜플도 역시 다른 컬렉션들과 같이 객체에 대한 레퍼런스를 가지고 있다. 만약 레퍼런스된 아이템들이 mutable 하면, 튜플안에 존재하더라도 해당 아이템들은 변경될수 있다 (If the referenced items are mutable, they may change even if the tuple itself does not. In other words, the immutability of tuples really refers to the physical contents of the tuple data structure (i.e., the references it holds), and does not extend to the referenced objects.

Below example illustrates the situation in which the value of a tuple changes as result of changes to a mutable object referenced in it. What can never change in a tuple is the identity of the items it contains.

```python
>>> t1 = (1, 2, [30, 40])
>>> t2 = (1, 2, [30, 40])
>>> t1 == t2
True
>>> id(t1[-1])  # t1[-1] 의 object id 를 확인
140476663098120
>>> t1[-1].append(99)
>>> t1          # 튜플의 값이 변경됐다!!
(1, 2, [30, 40, 99])
>>> id(t1[-1])  # object id 가 변하지 않았다!! 튜플의 immutability 는 이것을 말하는건가
140476663098120
>>> t1 == t2    # 당연히 값은 다르다
False
>>>
```

This relative(상대적인) immutability of tuples is behind the riddle. It's also the reason why some tuples are unhashable, as we've seen in What Is Hashable

The distinction between equality and identity has further implications when you need to copy an object. A copy is an equal object with a different ID. But if an object contains other objects, should the copy also duplicate the inner objects, or is it OK to share them?


#### Copies Are Shallow by Default

가장 간단한 리스트 복사 방법은 아래와 같다.

```python
>>> l1 = [3, [55, 44], (7, 8, 9)]
>>> l2 = list(l1)
>>> l2
[3, [55, 44], (7, 8, 9)]
>>> l1 == l2
True
>>> l2 is l1
False
>>>

# 혹은

>>> l3 = l1[:]
>>> l3
[3, [55, 44], (7, 8, 9)]
```

위와 같이 constructor 나 [:] 를 사용하면 `shallow copy` (겉의 컨테이너는 새로 생성되나 내부의 아이템들은 오리지날 객체가 가지고 있는 아이템들을 레퍼런스하고 있는 형식). This saves memory and causes no problems if all the items are immutable. 그러나 mutable 아이템이 있으면 골치아프게 된다. 다음 예제에서 `shallow copy` 를 이용한 문제점을 살펴보자.

```python
>>> l1 = [3, [66, 55, 44], (7,8,9)]
>>> l2 = list(l1)
>>> l1.append(100)
>>> l1[1].remove(55)
>>>
>>> print('l1:', l1)
l1: [3, [66, 44], (7, 8, 9), 100]
>>>
>>> print('l2:', l2)
l2: [3, [66, 44], (7, 8, 9)]
>>>
>>>
>>>
>>> l2[1] += [33, 22]   # mutable object can be changed
>>> l2[2] += (10, 11)   # += creates a new tuple and rebinds the variable l2[2] here. This is the same as doing l2[2] = l2[2] + (10, 11) 즉, 새로운 튜플 객체가 l2[2] 에 할당된다.. 이제 l1 과 l2 의 튜플은 더이상 같은 object 가 아니다.
>>>
>>> print('l1:', l1)
l1: [3, [66, 44, 33, 22], (7, 8, 9), 100]
>>>
>>> print('l2:', l2)
l2: [3, [66, 44, 33, 22], (7, 8, 9, 10, 11)]
>>>
```

#### Deep and Shallow Copies of Arbitrary Objects

가끔은 shallow copy 와는 반대로 객체를 복제하되 그안의 아이템들을 공유하지 않고 아이템도 새로 복제할 필요가 있다. `copy` 모듈은 `deepcopy` and `copy` 함수를 제공하여 해다아 기능을 가능하게 해준다.




















