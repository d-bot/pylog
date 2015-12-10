---
layout: post
title: "Array Sequence"
date: 2015-12-08 23:22:55 +0000
comments: true
categories:
---

## Sequences
- Container sequences: list, tuple and collections.deque can hold items of different types (포함하고 있는 객체의 reference 를 가지고 있음)
- flat sequences: str, bytes, bytearray, memoryview and array.array hold items of one type. (실제 객체의 값을 할당된 메모리 안에서 hold 하고 있음, 그래서 flat sequence 가 좀 더 compact 하지만 그래서 character, bytes and numbers 같은 primitive value 만 가질 수 있다)

Mutability 로 sequence 종류를 그룹핑할수 있다.
- Mutable sequences: list, bytearray, array.array, collections.deque and memoryview
- Immutable sequences: tuple, str and bytes



## List Comprehensions and Reaadability

#### SYNTAX TIP
In Python code, line breaks are ignored inside pairs of [], {}, or (). So you can build multiline lists, listcomps, genexps, dictionaries and the like without using the ugly line continuation escape.

리스트 컴프리헨션 사용시 예전에는 표현식의 스코프가 겹쳤는데 3.x 버전부터는 표현식 자신의 local scope 가 있어서 충돌이 일어나지 않는다.

```python
## 3.4.x
>>> x = 'asdf'
>>> dummy = [ x for x in 'test' ]
>>> dummy
['t', 'e', 's', 't']
>>> x
'asdf'


##2.7.x
>>> x = 'test'
>>> dummy = [ x for x in 'ABC' ]
>>> dummy
['A', 'B', 'C']
>>> x
'C'
```

#### Generator Expressions

Tuples, arrays 등 다른 타입의 시퀀스들을 초기화하는데 listcomp 를 사용할수 있으나 genexp 가 메모리를 더 절약한다 왜냐하면 it yields items one by one using the iterator procotol instead of building a whole list just to feed another constructor.

Genexps 는 대괄호가 아니라 일반 괄호로 감싸줘야한다.

```python
>>> symbols = '$%^&'
>>> tuple( ord(symbol) for symbol in symbols )    # single argument doesn't require parentheses
(36, 194, 162, 194, 163, 194, 165, 226, 130, 172, 194, 164)
>>> import array
>>> array.array( 'I', (ord(symbol) for symbol in symbols) ) # multiple arguments require parentheses
array('I', [36L, 194L, 162L, 194L, 163L, 194L, 165L, 226L, 130L, 172L, 194L, 164L])


>>> colors = 'black red'.split()
>>> sizes = ['S', 'M', 'L']
>>> for tshirt in ('%s %s' % (c, s) for c in colors for s in sizes): # See below explanation
...     print (tshirt)
...
black S
black M
black L
red S
red M
red L
```

The generator expression yields items one by one; a list with all six T-shirt variations is never produced in this example


## Tuples

Tuples are not just immutable lists. 튜플은 immutable list 뿐만 아니라 also as records with no field names. (db?)

#### Tuples as Records

Tuples hold records: each item in the tuple holds the data for one field and the position of the item gives its meaning. When using a tuple as a collection of fields, the number of items is often ifxed and their order is always vital.

아래 예제는 튜플을 레코드로 사용하는것을 보여준다. 보시다시피 해당 튜플들을 소팅해 버리면 의미가 뒤죽박죽이 되버린다.

```python
>>> lax_coordinates = (33.9425, -118.408056)
>>> city, year, pop, chg, area = ('Tokyo', 2003, 32450, 0.66, 8014)
>>> traverler_ids = [ ('USA', '31195855'), ('BRA', 'CE452567'), ('ESP', 'XDA205856') ]
>>> for passport in sorted(traveler_ids):
...   print('%s/%s' % passport)
...
BRA/CE342567
ESP/XDA205856
USA/31195855
>>> for country, _ in traveler_ids:
...   print(country)
...
USA
BRA
ESP
```

#### Tuple Unpacking

```python
# Tuple unpacking
>>> city, year, pop, chg, area = ('Tokyo', 2003, 32450, 0.66, 8014)
>>> print('%s/%s') % ('USA', '31195855')

# swapping without temp variable
>>> b, a = a, b

# prefixing an argument w/ a start when calling a function:
>>> divmod(20,8)
(2,4)
>>> t = (20, 8)
>>> divmod(*t)
(2,4)
>>> quotient, remainder = divmod(*t)
>>> quotient, remainder
(2,4)

>>> import os
>>> _, filename = os.path.split('/home/xxx/.ssh/idrsa.pub')
>>> filename
'idrsa.pub'

# 가끔 튜플로 어사인되는 변수들중에 일부만 필요한 경우 나머지는 dummy variable 인 _ 을 사용한다. as placeholder
```

Tip: Tuple unpacking works with any iterable object. The only requirement is that the iterable yields exactly one item per variable in the receiving tuple, unless you use a start(\*) to capture excess items.

#### Using \* to grab excess items

```python
# Doesn't work with python2.7.x

>>> a, b, *rest = range(5)
>>> a,b,rest
(0, 1, [2, 3, 4])
>>> a, b, *rest = range(2)
(0, 1, [])

>>> a, *body, c, d = range(5)
>>> a, body, c, d
(0, [1, 2], 3, 4)
```

#### Named Tuples

Instances of a class that you build w/ namedtuple take exactly the same amount of memory as tuples because the field names are stored in the class. They use less memory than a regular object because they don't store attributes in a per-instance `__dict__`

namedtuple 생성시 2개의 파라미터가 필요하다. a class name and a list of field names, which can be given as an iterable of strings or as a single space-delimited string.


```python
>>> from collections import namedtuple
>>> City = namedtuple('City', 'name country population coordinates')
>>> tokyo = City('Tokyo', 'JP', 36.933, (24.689722, 139.691667))
>>> tokyo
City(name='Tokyo', country='JP', population=36.933, coordinates=(35.689722, 139.691667))
>>> tokyo.population
36.933
>>> tokyo.coordinates
(35.689722, 139.691667)
>>> tokyo[1]
'JP'

>>> City._fields  # _fields is a tuple w/ the field names of the class
('name', 'country', 'population', 'coordinates')
>>> LatLong = namedtuple('LatLong', 'lat long')
>>> delhi_data = ('Delhi NCR', 'IN', 21.935, LatLong(28.613889, 77.208889))
>>> delhi = City._make(delhi_data)  # _make() allow you to instantiate a named tuple from an iterable; City(*delhi_data) would do the same.
>>> delhi._asdict() # it returns a collections.OrderedDict built from the named tuple instance.
OrderedDict([('name', 'Delhi NCR'), ('country', 'IN'), ('population', 21.935), ('coordinates', LatLong(lat=28.613889, long=77.208889))])
>>> for key, value in delhi._asdict().items():
...     print(key + ':', value)
...
('name:', 'Delhi NCR')
('country:', 'IN')
('population:', 21.935)
('coordinates:', LatLong(lat=28.613889, long=77.208889))
```


#### Tuples as Immutable Lists

Immutable variation of list 로서 튜플은 튜플 안의 아이템들을 add/remove 하는 함수들 제외하고는 대부분 list 와 동일한 함수를 가지고 있다. (exception - tuple lacks the `__reversed__`)


## Slicing

슬라이스는 `[START : STOP : STEP]` 의 형식으로 동작하며 파이썬 내부에서는 `seq.__getitem__(slice(start, stop, step))` 을 호출한다.


```python
>>> s = 'bicycle'
>>> s[::3]
'bye'
>>> s[::-1]
'elcycib'
>>> s[::-2]
'eccb'

# From the FrenchDeck example
""" Getting all ACE cards
"""
>>> deck[12::13]
[ Card(rank='A', suit='spades'), Card(rank='A', suit='diamonds'), Card(rank='A', suit='clubs'), Card(rank='A', suit='hearts') ]

```

slice lets you assign names to slices, just like spreadsheets allow naming of cell ranges.

```python
>>> invoice = """
... 0.....6.................................40........52...55........
... 1909  Pimoroni PiBrella                     $17.50    3    $52.50
... 1489  6mm Tactile Switch x20                 $4.95    2     $9.90
... 1510  Panavise Jr. - PV-201                 $28.00    1    $28.00
... 1601  PiTFT Mini Kit 320x240                $34.95    1    $34.95
... """
>>> UNIT_PRICE = slice(40,52)
>>> DESC = slice(6,40)
>>> line_items = invoice.split('\n')[2:]
>>> for item in line_items:
...     print(item[UNIT_PRICE], item[DESC])
...
$17.50   Pimoroni PiBrella
$4.95   6mm Tactile Switch x20
$28.00   Panavise Jr. - PV-201
$34.95   PiTFT Mini Kit 320x240

```

#### Assigning to Slices

```python
>>> l = list(range(10))
>>> l
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> l[2:5] = [20, 30]
>>> l
[0, 1, 20, 30, 5, 6, 7, 8, 9]
>>> del l[5:7]
>>> l
[0, 1, 20, 30, 5, 8, 9]
>>> l[3::2] = [11, 22]
>>> l
[0, 1, 20, 11, 5, 22, 9]
```


#### += and \*=

+= 와 \*= 는 첫번째 operand 에 따라서 다르게 행동한다. += 연산자는 내부적으로 `__iadd__` 라는 스페셜 함수를 호출한다. 그러나 `__iadd__` 가 해당 객체에 정의되어 있지 않으면 Python falls back to calling `__add__`.

```python
>>> a += b
```

a 가 일반적인 mutable sequence 면 `a = a + b` 가 되겠지만 a 가 immutable object 이면 `__iadd__` 이 구현하고 있지않으므로 해당 표현은 `a = a + b` 와 같은 효과를 가지지만, `a + b` 가 먼저 evaluated first, producing a new object, which is then bound to `a`. In other words, the identity of the object bound to `a` may or may not change, depending on the availability of `__iadd__`.

```python
>>> l = [1, 2, 3]
>>> id(l)
4311953800
>>> l * 2
[1, 2, 3, 1, 2, 3]
>>> id(l)
4311953800    # same object
>>>
>>> t = (1, 2, 3)
>>> id(t)
4312681568
>>> t *= 2
>>> id(t)
4301348296    # a new tuple object was created!!
```

Repeated concatenation of immutable sequences is inefficient, because instead of just appending new items, the interpreter has to copy the whole target sequence to create a new one with the new items concatenated.


#### list.sort and sorted built-in function

list.sort 함수는 새로운 객체를 생성하지 않고 그냥 해당 list 를 소팅해주고 `None` 을 리턴한다. `None` 을 리턴하는 의미는 해당 오퍼레이션이 새로운 객체를 생성하지 않고 target object itself 를 변경했다는 의미이다. 파이썬 컨벤션에서 function 이나 method 가 새로운 객체를 생성하지 않고 target object itself 가 변경될때 `None` 을 리턴해준다. `random.shuffle` 도 동일하게 `None` 을 리턴한다. 이 컨벤션의 단점은 cascade call 을 할수 없다는것? x.sort.shuffle.call 뭐 이런 표현


반대로, `sorted` 라는 built-in 함수는 새로운 list 를 생성하고 리턴한다. `sorted` accepts any iterable object as an argument, including immutable sequences and generators. Regardless of the type of iterable given to `sorted`, it always returns a newly created list

```python
>>> fruits = ['grape', 'raspberry', 'apple', 'banana']
>>> sorted(fruits)
['apple', 'banana', 'grape', 'raspberry']
>>>
>>> fruits
['grape', 'raspberry', 'apple', 'banana']
>>>
>>>
>>> sorted(fruits, reverse=True)
['raspberry', 'grape', 'banana', 'apple']
>>> sorted(fruits, key=len)
['grape', 'apple', 'banana', 'raspberry']
>>> sorted(fruits, key=len, reverse=True)
['raspberry', 'banana', 'grape', 'apple']
>>> fruits
['grape', 'raspberry', 'apple', 'banana']
>>>
>>>
>>> fruits.sort()
>>> fruits
['apple', 'banana', 'grape', 'raspberry']
```

Once the sequences are sorted, they can be very efficeintly searched.


#### Managing Ordered Sequences with bisect

bisect 모듈은 binary search 알고리즘을 사용하는 두가지 함수를 제공한다. `bisect` 와 `insort` 는 빠르게 item 을 찾고 추가할수 있다. 단 해당 sequence 는 이미 정렬이 되어 있어야 한다.

아래에서 `bisect` 는 신규로 추가될 아이템 위치의 인덱스 넘버를 리턴한다.

```python
#!/usr/bin/env python3.4

import bisect
import sys

HAYSTACK = [1, 4, 5, 6, 8, 12, 15, 20, 21, 23, 23, 26, 29, 30]
NEEDLES = [0, 1, 2, 5, 8, 10, 22, 23, 29, 30, 31]

ROW_FMT = '{0:2d} @ {1:2d}    {2}{0:<2d}'

def demo(bisect_fn):
    for needle in reversed(NEEDLES):
        position = bisect_fn(HAYSTACK, needle)
        offset = position * '  |'
        print(ROW_FMT.format(needle, position, offset))

if __name__ == '__main__':

    if sys.argv[-1] == 'left':
        bisect_fn = bisect.bisect_left
    else:
        bisect_fn = bisect.bisect

    print('DEMO:', bisect_fn.__name__)
    print('haystack ->', ' '.join('%2d' % n for n in HAYSTACK))
    demo(bisect_fn)

"""
$ ./test.py
DEMO: bisect
haystack ->  1  4  5  6  8 12 15 20 21 23 23 26 29 30
31 @ 14      |  |  |  |  |  |  |  |  |  |  |  |  |  |31
30 @ 14      |  |  |  |  |  |  |  |  |  |  |  |  |  |30
29 @ 13      |  |  |  |  |  |  |  |  |  |  |  |  |29
23 @ 11      |  |  |  |  |  |  |  |  |  |  |23
22 @  9      |  |  |  |  |  |  |  |  |22
10 @  5      |  |  |  |  |10
 8 @  5      |  |  |  |  |8
 5 @  3      |  |  |5
 2 @  1      |2
 1 @  1      |1
 0 @  0    0
"""
```

From Python Documentation

`bisect(a, x, lo=0, hi=len(a))` 여기서 `lo` 와 `hi` 는 옵션이고 defualt value 는 lo=0, hi=len(a) 이다. They allow narrowing the region in the sequence to be searched when inserting.

`bisec` is actually an alias for `bisect_right`, and there is a sister function called `bisect_left`. Their difference is apparent only when the needle compares equal to an item in the list: `bisect_right` returns an insertion point after the existing item, and `bisect_left` returns the position of the existing item, so insertion would occur before it. With simple types like `int` this makes no difference, but if the sequence contains objects that are distinct yet compare equal, then it may be relevant. For example, 1 and 1.0 are distinct, but 1 == 1.0 is True.

```python
$ ./test.py left
DEMO: bisect_left
haystack ->  1  4  5  6  8 12 15 20 21 23 23 26 29 30
31 @ 14      |  |  |  |  |  |  |  |  |  |  |  |  |  |31
30 @ 13      |  |  |  |  |  |  |  |  |  |  |  |  |30
29 @ 12      |  |  |  |  |  |  |  |  |  |  |  |29
23 @  9      |  |  |  |  |  |  |  |  |23
22 @  9      |  |  |  |  |  |  |  |  |22
10 @  5      |  |  |  |  |10
 8 @  4      |  |  |  |8
 5 @  2      |  |5
 2 @  1      |2
 1 @  0    1
 0 @  0    0
```

#### Converting test score to grade

```python
>>> import bisect
>>>
>>> def grade(score, breakpoints=[60, 70, 80, 90], grades='FDCBA'):
...     i = bisect.bisect(breakpoints, score)
...     return grades[i]
...
>>> [ grade(score) for score in [33, 99, 77, 70, 89, 90, 100] ]
['F', 'A', 'C', 'C', 'B', 'A', 'A']
>>>
```

#### Inserting with bisect.insort

소팅 오프레이션은 비용이 많이 드는 작업이므로 일단 소팅이 되면 해당 상태로 sequence 를 보존하는것이 앞으로의 sequence 오퍼레이션관련 리소스를 절약할 수 있다.

`insort(seq, item)` inserts item into seq so as to keep seq in ascending order.

```python
mport bisect
import random

SIZE = 7

random.seed(1729)

my_list = []
for i in range(SIZE):
    new_item = random.randrange(SIZE*2)
    bisect.insort(my_list, new_item)
    print('%2d ->' % new_item, my_list)

"""
$ python3 insort.py
10 -> [10]
 0 -> [0, 10]
 6 -> [0, 6, 10]
 8 -> [0, 6, 8, 10]
 7 -> [0, 6, 7, 8, 10]
 2 -> [0, 2, 6, 7, 8, 10]
10 -> [0, 2, 6, 7, 8, 10, 10]
"""
```

`insort` 역시 `lo` 와 `hi` 라는 2개의 옵션 인자를 가지고 있다 to limit the search to a sub-sequence. There is also an `insort_left` variation that uses `bisect_left` to find insertion points.



#### When a List is Not the Answer

가끔 list 자료구조로 보다 더 나은 선택이 있을 수 있다. 예를 들면 십만개의 소수점 값들을 저장해야한다면 `array` 가 훨씬 효율적인 자료구조형이 될것이다 왜냐하면 `array` 는 실제로 full-fledged (완전히 성장한) float objects 를 hold 하고 있기 때문이다. but only the packed bytes representing their machine values - just like an array in the C language. on the other hand, if you are constantly adding and removing items from the ends of a list as a FIFO or LIFO data structure, a deque (double-ended queue) works faster.

#### Array

If the list will only contain numbers, an `array.array` is more efficient than a lisT: it supports all mutable sequence operations (including .pop, .insert and .extend) and additional methods for fast loading and saving such as `.frombytes` and `.tofile`

파이썬의 `array` 는 C 만큼이나 lean 하다. `array` 를 생성할때, typecode 를 지정해야한다. 예를 들면, `b` is the typecode for signed char. 즉, `array('b')` 를 생성하면, each item will be stored in a single byte and interpreted as an integer from -128 to 127. For large sequences of numbers, this saves a lot of memory and Python will not let you put any number that doesn't match the type for the array.


#### Deques and Other Queues

일반적인 리스트의 item 변경 작업은 코스트가 많이 드는 작업이다 왜냐하면 리스트 전체의 shift 가 발생해하므로. 그러나 The class `collections.deque` is a thread-safe double-ended queue designed for fast inserting and removing from both ends. It is also the way to go if you need to keep a list of "last seen items" or something like that, because a `deque` can be bounded (i.e., created with a maximum length) and then, when it is full, it discards items from the opposite end when you append new ones.

```python
>>> from collections import deque
>>> dq = deque(range(10), maxlen=10)
>>> dq
deque([0, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)
>>> dq.rotate(3)
>>> dq
deque([7, 8, 9, 0, 1, 2, 3, 4, 5, 6], maxlen=10)
>>> dq.rotate(-4)
>>> dq
deque([1, 2, 3, 4, 5, 6, 7, 8, 9, 0], maxlen=10)
>>> dq.appendleft(-1)
>>> dq
deque([-1, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)
>>> dq.extend([11, 22, 33])
>>> dq
deque([3, 4, 5, 6, 7, 8, 9, 11, 22, 33], maxlen=10)
>>> dq.extendleft([10, 20, 30, 40])
>>> dq
deque([40, 30, 20, 10, 3, 4, 5, 6, 7, 8], maxlen=10)

# also look at popleft
```

## Chapter Summary

Mastering the standard library sequence types is a prerequisite for writing consise, effective, and idiomatic Python code.

Python sequences are often categorized as mutable or immutable but it is also useful to consider a different axis: flat sequences and container sequences. The former are more compact, faster and easier to use, but are limited to storing atomic data such as numbers, characters, and bytes. Container sequences are more flexible, but may surprise you when they hold mutable objects, so you need to be careful to use them correctly with nested data structures.

List comprehensions and generator expressions are powerful notations to build and initialize sequences. If you are not yet comfortable with them, take the time to master their basic usage. It is not hard, and soon you will be hooked.

Tuples in Python play two roles: as records with unnamed fields and as immutable lists. When a tuple is used as a record, tuple unpacking is the safest, most readable way of getting at the fields. The new \* syntax makes tuple unpacking even better by making it easier to ignore some fields and to deal with optional fields. Named tuples are not so new, but deserve more attention: like tuples, they have very little overhead per instance, yet provide convenient access to the fields by name and a handy `._asdict()` to export the record as an OrderedDict.

Sequence slicing is a favorite Python syntax feature, and it is even more powerful than many realize. Multidimensional slicing and ellipsis (...) notation, as used in NumPy, may also be supported by user-defined sequences. Assigning to slices is a very expressive way of editing mutable sequences.

Repeated concatenation as in seq \* n is convenient and, with care, can be used to initialize lists of lists containing immutable items. Augmented assignment with += and \*= behaves differently for mutable and immutable sequences. In the latter case, these operators necessarily build new sequences. But if the target sequence is mutable, it is usually changed in place - but not always, depending on how the sequence is implemented.

The `sort` method and the `sorted` built-in function are easy to use and flexible, thanks to the key optional argument they accept, with a function to calculate the ordering criterion. By the way, key can also be used with the min and max built-in functions. To keep a sorted sequence in order, always insert items into it using bisect.insort; to search it efficiently, use bisect.bisect.

















