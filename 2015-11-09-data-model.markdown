---
layout: post
title: "Data Model"
date: 2015-11-09 07:47:38 +0000
comments: true
categories: 
---

[namedtuple](https://www.reddit.com/r/Python/comments/3qw7m4/improving_your_code_readability_with_namedtuples/)

[getitem](http://blog.weirdx.io/python-__getitem__과-slice의-이해/)

```python
import collections

Card = collections.namedtuple('Card', ['rank', 'suit'])

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

#>>> 52

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








