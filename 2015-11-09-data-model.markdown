---
layout: post
title: "Data Model"
date: 2015-11-09 07:47:38 +0000
comments: true
categories: 
---

[getitem](http://blog.weirdx.io/python-__getitem__과-slice의-이해/)

## Ruby 의 `include Enumerable` 과 비슷

```python
from collections import namedtuple
from random import choice

Card = namedtuple('Card', ['rank', 'suit'])

class FrenchDeck:
  ranks = [ str(n) for n in range(2, 11) ] + list('JQKA')
  suits = 'spades diamonds clubs hearts'.split()

  def __init__(self):
    self._cards = [ Card(rank, suit) for suit in self.suits for rank in self.ranks ]

  def __len__(self):
    return len(self._cards)

  def __getitem__(self, position):
    """
    AttributeError: FrenchDeck instance has no attribute '__getitem__'
    """
    return self._cards[position]

deck = FrenchDeck()
print len(deck)
print choice(deck)


suit_values = dict(spades=3, hearts=2, diamonds=1, clubs=0)

def spades_high(card):
  rank_index = FrenchDeck.ranks.index(card.ranks)
  return rank_index * len(suit_values) + suit_values[card.suits]

for i in sorted(deck, key=spades_high)
  print i

```

위 클래스의 핵심은 `__len__` 과 `__getitem__` 이다. 즉 `FrenchDeck` 클래스를 사용하는 사람들은 standard operation (len, choice) 같은 기능을 사용하기 위해 해당 클래스의 특정 함수이름을 기억할 필요가 없다. `__len__` 과 `__getitem__` 과 같은 함수를 이용해 쉽게 구현이 가능하기 때문이다.

`__getitem__` delegates to the [] operator of `self._cards`, our deck automatically supports slicing. Here's how we look at the top 3 cards from a brand new deck, and then pick just the aces by starting on index 12 and skipping 13 cards at a time:

```python
>>> deck[:3]
[ Card(rank='2', suit='spades'), Card(rank='3', suit='spades'), Card(rank='4', suit='spades') ]

>>> deck[12::13]
[ Card(rank='A', suit='spades'), Card(rank='A', suit='diamonds'), Card(rank='A', suit='clubs'), Card(rank='A', suit='hearts') ]

```

Just by implementing the `__getitem__` special method, our deck is also iterable:

```python

>>> for card in deck:
...   print(card)
Card(rank='2', suit='spades')
Card(rank='3', suit='spades')
Card(rank='4', suit='spades')
...

# reverse order
>>> for card in reversed(deck):
...   print(card)
Card(rank='A', suit='hearts')
Card(rank='K', suit='hearts')
Card(rank='Q', suit='hearts')

```

즉, 이런 special methods 들은 유저가 아닌 python interpreter 에 의해서 호출되는 함수들이다. 너는 `len(my_obj)` 라고 쓰고 인터프리터는 `my_obj.__len__()` 이렇게 호출한다. 이런 special method 를 직접 호출하는 경우는 메타 프로그래밍시 많이 발생한다.

## Emulating Numeric Types

```python
from math import hypot

class Vector:
  def __init__(self, x=0, y=0)
    self.x = x
    self.y = y

  def __repr__(self):
    return 'Vector(%r, %r)' % (self.x, self.y)

  def __abs__(self):
    return hypot(self.x, self.y)

  def __bool__(self):
    return bool(abs(self))

  def __add__(self, other):
    x = self.x + self.x
    y = self.y + self.y
    return Vector(x, y)

  def __mul__(self):
    return Vector(self.x * scalar, self.y * scalar)

```

Note that although we implemented 4 special methods (aprt from `__init__`), none of them is directly called within the class or in the typical usage of the class illustrated by the console listings. As mentioned before, the Python interpreter is the only frequent caller of the most special methods.

## String Representation

The `__repr__` special method is called by the repr built-in to get the string representation of the object for inspection. If we did not implement `__repr__` an instance would be shown in the console like `<Vector object at 0x10e10070>`.

The interective console and debugger call repr on the results of the expressions evaluated, as does the %r placeholder in classic format with the % operator, and the !r conversion field in the new Format String Syntax used in the str.format method

If you only implement one of these special methods, choose `__repr__`, because when no custom `__str__` is available, Python will call `__repr__` as a fallback

[differences between `__str__` and `__repr__`](http://stackoverflow.com/questions/1436703/difference-between-str-and-repr-in-python)

## Arithmetic Operators

Above example implements 2 oeprators: + and \*, to show basic usage of `__add__` and `__mul__`. Note that in both cases, the methods create and return a new instance of Vector, and do not modify either oeprand -- self or other are merely read. This is expected behavior of infix oeprators: to create new objects and not touch their operands.

## Boolean Value of a Custom Type

By default, instances of user-defined classes are considered truthy, unless either `__bool__` or `__len__` is implemented. Basically bool(x) calls `x.__bool__()` and uses the result. If `__bool__` is not implemented, Python tries to invoke `x.__len__()`, and if that return zero, bool returns False. Otherwise bool returns True.

## Summary

- By implementing special methods, your objects can behave like the built-in types, enabling the expressive coding style the community considers Pythonic.
- A basic requirement for a Python object is to provide usable string representations of itself, one used for debugging and logging, another for presentation to end users. That is why the special methods `__repr__` and `__str__` exist in the data model.
- Emulating sequences, as shown with the FrenchDeck example, is one of the most widely used applications of the special methods.
- Thanks to operator overloading, Python offers a rich selection of numeric types, from the built-ins to decimal.Decimal and fractions.Fraction, all supporting infix arithmetic operators.

