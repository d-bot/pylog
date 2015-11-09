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
    return self._cards[position]

deck = FrenchDeck()
print len(deck)

#>>> 52

```
