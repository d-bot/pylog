---
layout: post
title: "Object References Mutability and Recycling"
date: 2015-12-29 09:36:29 +0000
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

가끔은 shallow copy 와는 반대로 객체를 복제하되 그안의 아이템들을 공유하지 않고 아이템도 새로 복제할 필요가 있다. `copy` 모듈은 `deepcopy` and `copy` 함수를 제공하여 해당 기능을 가능하게 해준다.

아래 예제는 손님들을 태우고 내리는 버스를 나타낸다.

```python

class Bus:
    def __init__(self, passengers=None):
        if passengers is None:
            self.passengers = []
        else:
            self.passengers = list(passengers)

    def pick(self, name):
        self.passengers.append(name)

    def drop(self, name):
        self.passengers.remove(name)

##
##

>>> from bus import *
>>> import copy
>>> bus1 = Bus(['Alice', 'Bill', 'Claire', 'David'])
>>> bus2 = copy.copy(bus1)
>>> bus3 = copy.deepcopy(bus1)
>>> id(bus1), id(bus2), id(bus3)
(140058071605544, 140058037798672, 140058037799008) # created 3 distinct Bus instances
>>> bus1.drop('Bill')
>>> bus2.passengers
['Alice', 'Claire', 'David']
>>> id(bus1.passengers), id(bus2.passengers), id(bus3.passengers)
(140058071603656, 140058071603656, 140058037695944) # Inspection of the passengers attributes show that bus 1 and bus 2 share the same list object, because bus2 is a shallow copy of bus1
>>> bus3.passengers
['Alice', 'Bill', 'Claire', 'David']

```

#### Function Parameters as References

파이썬에서 parameter passing 모드는 오직 `call by sharing` 뿐이다. 루비, 스몰톡, 자바 등의 OO 언어에서 대부분 사용하는 모드이다. `Call by sharing` means that each formal parameter of the function gets a copy of each reference in the arguments. 즉, 함수안의 파라미터는 실제 파라미터의 alias 가 된다.

The result of this scheme is that a function may change any mutable object passed as a parameter, but it cannot change the identity of those objects.

```python
>>> def f(a, b):
...     a += b
...     return a
...
>>> x = 1
>>> y = 2
>>> f(x, y)
3
>>> x, y    # INTEGER x has NOT changed!!! because integer is immutable
(1, 2)
>>> a = [1, 2]
>>> b = [3, 4]
>>> f(a, b)
[1, 2, 3, 4]
>>> a, b    # LIST a has changed!!! because list is mutable
([1, 2, 3, 4], [3, 4])
>>>
>>> t = (10, 20)
>>> u = (30, 40)
>>> f(t, u)
(10, 20, 30, 40)
>>>
>>> t, u    # TUPLE t has NOT changed!!! becuase tuple is immutable
((10, 20), (30, 40))
```

mutable 오브젝트를 파라미터로 사용하는 또다른 문제는 다음 예제에서 또 나타난다.


#### Mutable Types as Parameter Defaults: Bad Idea

Optional parameter with default values are a great feature of Python function definitions, allowing our APIs to evolve while remaining backward-campatible. However, you should avoid mutable objects as default values for parameters.

To illustrate this point, following example, we take the `Bus` calss from the previous example and change its `__init__` method to create `HauntedBus`. Here we tried to be clever and instead of having a default value of `passengers=None`, we have `passengers=[]`, thus avoiding the if in the previous `__init__`. This "cleverness" gets us into trouble.

```python

class HauntedBus:
    def __init__(self, passengers=[]):
        self.passengers = passengers    # 여기서 self.passengers 는 passengers 라는 파라미터의 alias 가 된다.

    # 아래의 append 나 remove 가 사용될때마다 mutable object 인 passengers 라는 객체의 값을 변경하게 된다.
    def pick(self, name):
        self.passengers.append(name)

    def drop(self, name):
        self.passengers.remove(name)

##

>>> bus1 = HauntedBus(['Alice', 'Bill'])
>>> bus1.passengers
['Alice', 'Bill']
>>> bus1.pick('Charlie')
>>> bus1.drop('Alice')
>>>
>>> bus1.passengers
['Bill', 'Charlie']
>>>
>>>
>>>
>>> bus2 = HauntedBus()
>>> bus2.pick('Carrie')
>>>
>>> bus2.passengers
['Carrie']
>>>
>>>
>>>
>>> bus3 = HauntedBus()
>>> bus2.passengers
['Carrie']
>>> bus3.pick('Dave')
>>>
>>>
>>> bus2.passengers
['Carrie', 'Dave']      # bus3 에 Dave 를 넣었는데 bus2 에도 들어가있음.
>>> bus2.passengers is bus3.passengers
True                    # bus2 와 bus3 는 동일한 객체를 가르킨다.
>>> bus1.passengers
['Bill', 'Charlie']     # bus1 에는 변화가 없다.
>>>
>>>
>>>
```

여기서 문제는 Bus 인스턴스가 파라미터 없이 초기화 될경우 기존에 존재하는 Bus 인스턴스들과 같은 defualt parameter 를 공유한다는 것이다. (The problem is that `Bus` instances that don't get an initial passenger list end up sharing same passenger list among themselves)


객체 초기화시 파라미터 값을 넘기면 괜찮지만 아무 파라미터 없이 초기화하여 default parameter 를 사용하게 되면 골치아파진다 왜냐하면 `self.passengers` 가 default value of the passenger parameter 의 alias 가 되기때문이다.

진짜 문제는 default value 값은 함수가 정의될때 evaluated 되기때문이다. (Usually when the module is loaded - and the default values become attributes of the function object. So if a default value is a mutable object, and you change it, the change will affect every future call of the function.

After running the lines in above example, you can inspect the `HauntedBus.__init__` object and see the ghost students haunting its `__defaults__` attribute:


```python
>>> dir(HauntedBus.__init__)  # doctest: +ELLIPSIS
['__annotations__', '__call__', ..., '__defaults__', ...]
>>> HauntedBus.__init__.__defaults__
(['Carrie', 'Dave'],)

# Finally we can verify that bus2.passengers is an alias bound to the first element of the HauntedBus.__init__.__defaults__ attribute

>>> HauntedBus.__init__.__defaults__[0] is bus2.passengers
True

```

위의 예제에서 보다시피 `None` 이 왜 optional parameter 들의 디폴트 인자로(default parameter) 선호되는지 알수 있다. 다음 예제에서는 `__init__` 가 passengers argument 가 None 인지 아닌지 그리고 빈 리스트를 self.passengers 에 할당한다. passengers 가 None 이 아니면 a copy of it to self.passengers 를 할당한다.


#### Defensive Programming with Mutable Parameter

mutable parameter 를 받는 함수를 정의할때, 반드시 caller 는 해당 파라미터의 값이 변경될건지 고민해 봐야한다. 예를 들면, 어떤 함수가 dict 를 파라미터로 받고 함수를 실행하는 도중 해당 dict 의 값이 변경되어야 하는 경우 해당 변경사항이 함수 실행 종료후 어떻게 리턴될지 어떤 사이드 이펙트가 있는지 잘 확인해야한다.


```python
class TwilightBus:
    """A bus model that makes passengers vanish"""

    def __init__(self, passengers=None):
        if passengers is None:
            self.passengers = []
        else:
            self.passengers = passengers

    def pick(self, name):
        self.passengers.append(name)

    def drop(self, name):
        self.passengers.remove(name)

#

>>> basketball_team = ['Sue', 'Tina', 'Maya', 'Diana', 'Pat']  1
>>> bus = TwilightBus(basketball_team)  2
>>> bus.drop('Tina')  3
>>> bus.drop('Pat')
>>> basketball_team  4
['Sue', 'Maya', 'Diana']

```

위에서 보다시피 `basketball_team` 을 파라미터로 넘기면 `self.passengers` 는 `basketball_team` 의 alias 가 되어 `basketball_team` 이라는 오리지날 변수의 값을 변경시켜버린다. 해당 픽스는 아래와 같이 간단하다. `list` 를 사용해서 파라미터의 복제본을 만드는 것이다.

```python
def __init__(self, passengers=None):
  if passengers is None:
    self.passengers = []
  else:
    self.passengers = list(passengers)  # make a copy of the passengers list, or convert it to a list if it's not one.
```

As a bonus, this solution is more flexible: now the argument passed to the passengers parameter may be a tuple or may other iterable, like a set or even database results, because the list constructor accepts any iterable. As we create our own list to manage, we ensure that it supports the necessary remove() and append() operations we use in the pick() methods.


#### del and Garbage Collection


`del` statement 는 이름을 지우지 객체를 지우지 않는다. An object may be garbage collected as result of a `del` command, but only if the variable deleted holds the last reference to the object, or if the object becomes unreachable. Rebindiong a variable may also cause the number of references to an object to reach zero, causing its destruction.


There is a `__del__` special method, but it doese not cause the disposal of the instance, and should not be called by your codee. `__del__` is invoked by the Python interpreter when the instance is about to be destroyed to give it a chance to release external resources. You will seldom need to implement `__del__` in your own code, yet some Python beginners spend time coding it for no good reason. The proper use of `__del__` is rather tricky.

CPython 에서는 모든 객체에는 얼마나 많이 레퍼런스 되는지 refcount 값을 추적하는데 이게 0이 되면 해당 객체는 바로 파괴된다. 이때 `__del__` 메소드가 호출이 되고 해당 객체가 차지하고 있는 메모리를 해제시킨다. 다른 Python Implementation 에서는 다른 방식으로 garbage collecting 을 한다.




#### Summary

Every Python object has an identity, a type and a value. Only the value of an object changes over time.

두개의 변수가 같은 값을 가지고 있는 immutable object 를 reference 하고 있고,  if they refer to copies or are aliases referring to the same object 면 거의 문제가 되지 않는다 왜냐하면 an immutable object 의 값은 변하지 않기 때문이다. 한가지 예외상황을 제외하고는. 그 예외상황은 tuple 이나 frozensets 같이 immutable collections 인 경우이다: If an immutable collection holds references to mutable items, then its value may actually change when the value of a mutable item changes. In practice, this scenario is not so common. What never changes in an immutable colection are the identities of the objects within. (그냥 그렇다고 뭐)


The fact that variables hold references has many practical consequences in Python programming

1. 간단한 assignment 는 복제본을 생성하지 않는다 (Simple assignment does not create copies)

2. Augmented assignment with += or \*= creates new obejcts if the lefthand variable is bound to an immutable object, but may modify a mutable object in place.

3. 이미 존재하는 객체에 값을 할당해도 이전의 객체를 변경하지 않는다. 이런 경우를 rebinding 이라고 하는데 해당 assignment 는 새로운 객체에 bound 된다. 해당 assignment 로 인해서 이미 존재하던 객체의 refcount 가 0 이 되면 garbage collecting 된다.

4. Function parameters are passed as aliases, which means the function may change any mutable object received as an argument. There is no way to prevent this, except making local copies or using immutable objects(e.g., passing a tuple instead of a list)

5. Using mutable objects as default values for function parameters is dangerous because if the parameters are changed in place, then the default is changed, affecting every future call that relies on the default.

In CPython, objects are discarded as soon as the number of references to them reaches zero. They may also be discarded if they form groups with cyclic references but no outside references. In some situations, it may be useful to hold a reference to an object that will not - by itself - keep an object alive. One example is a class that wants to keep track of all its current instances. This can be done with weak references, a low-level mechanism underlying the more useful collections WeakValueDictionary, WeakKeyDictionary, WeakSet, and the finalize function from the weakref module.

