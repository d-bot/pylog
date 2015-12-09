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
In Python code, line breaks are ignored inside pairs of [], {}, or (). So you can build multiline lists, listcomps, genexps, dictionaries and the like without using the ugly \ line continuation escape.

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
>>> symbols = '$¢£¥€¤'
>>> tuple( ord(symbol) for symbol in symbols )    # single argument doesn't require parentheses
(36, 194, 162, 194, 163, 194, 165, 226, 130, 172, 194, 164)
>>> import array
>>> array.array( 'I', (ord(symbol) for symbol in symbols) ) # multiple arguments require parentheses
array('I', [36L, 194L, 162L, 194L, 163L, 194L, 165L, 226L, 130L, 172L, 194L, 164L])
```

```python
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



















