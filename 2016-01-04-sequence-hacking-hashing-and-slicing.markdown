---
layout: post
title: "Sequence Hacking Hashing and Slicing"
date: 2016-01-04 05:10:28 +0000
comments: true
categories:
---

Don't check whether it is-a duck: check whether it quacks-like-a duck, walks-like-a duc, etc, etc, depending on exactly what subset of duck-like behavior you need to play

In this chapter, we'll create a class to represent a multidimensional Vector class - a significant step up from the two-dimensional Vector2d of previous one. Vector will behave like a standard Python immutable flat sequence. Its elements will be floats, and it will support the following by the end of this chapter.

- Basic sequence protocol: `__len__` and `__getitem__`

- Safe representation of instances with many items

- Proper slicing support, producing new Vector instances

- Aggregate hashing taking into account every contained element value

- Custom formatting language extension


Will also implement dynamic attribute access with `__getattr__` as a way of replacing the read-only properties

#### Vector: A User-Defined Sequence Type

Our strategy to implement Vector will be to use composition not inheritance. We'll store the components in an array of floats and will implement the methods needed for our Vector to behave like an immutable flat sequence.

#### Vector Take #1 Vector2d Compatible

이전에 만들었던 Vector2d 와 호환되게 작성해보자. 이전에는 생성자가 2개의 인자를 받았으나 \*args 를 이용해서 여러개의 인자를 받을 수 있도록 할수 있으나 best practice for sequence constructor is to take data as an iterable argument, like built-in sequence types do.
```python
>>> Vector([3.1, 4.2])  # 리스트
Vector([3.1, 4.2])
>>> Vector((3, 4, 5))   # 튜플
Vector([3.0, 4.0, 5.0])
>>> Vector(range(10))     # 함수
Vector([0.0, 1.0, 2.0, 3.0, 4.0, ...])
```

First version


```python
from array import array
import reprlib
import math

class Vector:

    typecode = 'd'

    def __init__(self, components):
        self._components = array(self.typecode, components)

    def __iter__(self):
        return iter(self._components)

    def __repr__(self):
        components = reprlib.repr(self._components)
        components = components[components.find('['):-1]
        return 'Vector({})'.format(components)

    def __str__(self):
        return str(tuple(self))

    def __bytes__(self):
        return (bytes([ord(self.typecode)]) + bytes(self._components))

    def __eq__(self, other):
        return tuple(self) == tuple(other)

    def __abs__(self):
        return math.sqrt(sum(x * x for x in self))

    def __bool__(self):
        return bool(abs(self))

    @classmethod
    def frombytes(cls, octets):
        typecode = chr(octets[0])
        memv = memoryview(octets[1:]).cast(typecode)
        return cls(memv)
```
