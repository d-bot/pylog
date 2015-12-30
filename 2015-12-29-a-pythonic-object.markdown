---
layout: post
title: "A Pythonic Object"
date: 2015-12-29 09:38:28 +0000
comments: true
categories:
---

파이썬에서는 `__repr__` 이나 `__str__` 외에도 object representation 을 위한 `__bytes__` 와 `__format__` 라는 2개의 object representation 함수들이 더 있다.

`__bytes__` 함수는 `__str__` 함수와 비슷하다: It's called by bytes() to get the object represented as a byte sequence.

Regarding `__format__`, both the built-in function `format()` and the str.format() 가 `__format__` 을 호출한다 to get string displays of objects using special formating codes.

```
If you’re coming from Python 2, remember that in Python 3 `__repr__`,` __str__`, and `__format__` must always return Unicode strings (type str). Only `__bytes__` is supposed to return a byte sequence (type bytes).
```

#### Vector Class Redux


```python

from array import array
import math

class Vector2d:
    typecode = 'd'  # class attribute we'll use when converting vector2d instances to/from bytes.

    def __init__(self, x, y):
        ''' float 를 이용해 잘못 들어온 인자들에 대해 발생할수 있는 에러를 미리 잡을 수 있다. '''
        self.x = float(x)
        self.y = float(y)

    def __iter__(self):
        ''' makes it iterable; this is what makes unpacking work(e.g, x,y = my_vector). We implement it simply by using a generator expression to yield the components one after the other '''
        return (i for i in (self.x, self.y))

    def __repr__(self):
        ''' builds a string by interpolating the components with {!r} to get their repr; because Vector2d is iterable, *self feeds the x and y components to format '''
        class_name = type(self).__name__
        return '{}({!r}, {!r})'.format(class_name, *self)

    def __str__(self):
        return str(tuple(self))

    def __bytes__(self):
        return (bytes([ord(self.typecode)]) +       # To generate bytes, we convert the typecode to bytes and concatenate.
                bytes(array(self.typecode, self)))  # ...bytes converted from an array built by iterating over the instance.

    def __eq__(self, other):
        ''' to quickly compare all components, build tuples out of the operands. This works for operands that are instances of Vector2d, but has issues.'''
        return tuple(self) == tuple(other)

    def __abs__(self):
        ''' The magnitude is the length of the hypotenuse of the triangle formed by the x and y components '''
        return math.hypot(self.x, self.y)

    def __bool__(self):
        """ Only 0.0 returns false """
        return bool(abs(self))


##
##
##

>>> from vector2d import *
>>> v1 = Vector2d(3, 4)
>>> print(v1.x, v1.y)   # The components of a Vector2d can be accessed directly as attributes(no getter method calls)
3.0 4.0
>>> x, y = v1   # A Vector2d can be unpacked ot a tuple of variables.
>>> x, y
(3.0, 4.0)
>>>
>>> v1
Vector2d(3.0, 4.0)
>>>
>>>
>>> v1_clone = eval(repr(v1))
>>> v1 == v1_clone
True
>>>
>>> print(v1)
(3.0, 4.0)
>>>
>>> octets = bytes(v1)
>>> octets
b'd\x00\x00\x00\x00\x00\x00\x08@\x00\x00\x00\x00\x00\x00\x10@'
>>> abs(v1)
5.0
>>> bool(v1), bool(Vector2d(0, 0))
(True, False)

```


#### An Alternative Constructor


Vector2d 를 bytes 를 통해서 바이너리로 export 할수 있으므로 Vector2d 를 바이너리 시퀀스로부터 읽을 수 있는 모듈이 필요하다. 스탠다드 라이브러리에 보면 `array.array` 이 `.frombytes` 라는 클래스메서드가 있는것을 볼 수 있다. 그럼 vector2d 에 해당 함수를 추가해보자.

```python
@classmethod  # Class method is modified by the classmethod decorator

def frombytes(cls, octets):   # 첫번째 인자로 self 가 아닌 자기자신을 나타내는 cls 가 들어갔다
  typecode = chr(octets[0])   # read the typecode from the first byte (octets[0])
  memv = memoryview(octets[1:]).cast(typecode)    # Create a memoryview resulting from the cast into the pair of arguments needed for the constructor
  return cls(*memv)

```

#### classmethod VS staticmethod

`classmethod` changes the way the method is called, so it receives the class itself as the first argument, instead of an instance, instead of an instance. (첫번째 파라미터로 self 라는 자기자신 객체를 받는게 아니라 해당 class itself 를 받는다) Its most common use is for alternative constructors, like frombytes. Note how the last line of frombytes actually uses the cls argument by invoking it to build a new instance: `cls(*memv)`. By convention, the first parameter of a class method should be named `cls` (but Python doesn't care how it's named).

In contrast, the `staticmethod` decorator changes a method so that it receives no special first argument. In essence, a static method is just like a plain function that happens to live in a class body, instead of being defined at the module level. Following example contrasts the operation of classmethod and staticmethod.


```python
>>> class Demo:
...     @classmethod
...     def klassmeth(*args):
...             return args   # klassmeth just returns all positional arguments
...     @staticmethod
...     def statmeth(*args):  # statmeth does the same
...             return args
...
>>>
>>>
>>> Demo.klassmeth()          # No matter how you invoke it, Demo.klassmeth receives the Demo class as the first argument
(<class '__main__.Demo'>,)
>>>
>>>
>>> Demo.klassmeth('spam')
(<class '__main__.Demo'>, 'spam')
>>>
>>>
>>> Demo.statmeth()           # Demo.statmeth behaves just like a plain old function
()
>>> Demo.statmeth('spam')
('spam',)
>>>
>>>

```


The `classmethod` decorator is clearly useful but I've never seen a compelling use case for `staticmethod`. If you want to define a function that does not interact with the class, just define it in the module. Maybe the function is closely related even if it never touches the class, so you want to them nearby in the code. Even so, defining the function right before or after the class in the same module is close enough for all practical purposes.



#### Formatted Displays

The `format()` built-in function and the `str.format()` method delegate the actual formatting to each type by calling their `.__format__(format_spec)` method. The format_spec is a formatting specifier, which is either:

The second argument in format(my_obj, format_spec), or

Whatever appears after the colon in a replacement field delimited with {} inside a format string used with `str.format()`

For example:

```python
>>> brl = 1/2.43  # BRL to USD currency conversion rate
>>> brl
0.4115226337448559
>>> format(brl, '0.4f')  # formatting specifier is 0.4f
'0.4115'
>>> '1 BRL = {rate:0.2f} USD'.format(rate=brl)  # formatting specifier is 0.2f. The 'rate' substring in the replacement field is called the field name. It's unrelated to the formatting specifier, but determines which argument of .format() goes into the replacement field.
'1 BRL = 0.41 USD'
```


#### A Hashable Vector2d

지금까지 정의한데로라면 Vector2d 객체는 unhashable 하므 로 set collection 안에 넣을 수 없다. Vector2d 객체를 hashable 하게 만들려면 `__hash__` (`__eq__` is also required and we already have it). We also need to make vector instances immutable as we've seen in What Is Hashable?

현재로서는 누구나 `v1.x = 7` 같은 방법으로 변수값을 변경을 할수 있다. 그렇지만 우리는 아래와 같이 x, y를 read-only 변수로 만들고 싶다.

```python
>>> v1.x, v1.y
(3.0, 4.0)
>>> v1.x = 7
Traceback (most recent call last):
  ...
  AttributeError: can't set attribute
```

We'll do that by making the x and y components read-only properties

```python
class Vector2d:
  typecode = 'd'

  def __init__(self, x, y):
    self.__x = float(x)   # Use dunder to make an attribute private
    self.__y = float(y)

  @property         # @property decorator marks the getter method of a property
  def x(self):      # the getter method is named after the public property it exposes: x
    return self.__x

  @property
  def y(self):
    return self.__y

  def __iter__(self):
    ''' Every method that just reads the x, y components can stay as they were, reading the public properties via self.x and self.y instead of the private attribute, so this listing omits the rest of the code for the class '''
    return (i for i in (self.x, self.y))

```

Now that our vectors are reasonably immutable, we can implement the `__hash__` method. It should return an int and ideally take into account(게산에 넣다, 고려하다) the hashes of the object attributes that are also used in the `__eq__` method, because objects that compare equal should have the same hash.

```python
def __hash__(self):
  return hash(self.x) ^ hash(self.y)

# With the addition of the `__hash__` method, we now have hashable vectors:
>>> v1 = Vector2d(3, 4)
>>> v2 = Vector2d(3.1, 4.2)
>>> hash(v1), hash(v2)
(7, 384307168202284039)
>>> set([v1, v2])
{Vector2d(3.1, 4.2), Vector2d(3.0, 4.0)}
>>>
>>>
>>>
```

It's not strictly necessary to implement properties or otherwise protect the instance attributes to create a hashable type. Implementing `__hash__` and `__eq__` correctly is all it takes. But the hash value of an instance is never supposed to change, so this provides an excellent opportunity to talk about read-only properties.


If you're creating a type that has a sensible scalar numeric value, you may also implement the `__int__` and `__float__` methods, invoked by the int() and float() constructors - which are used for type coercion in some contexts. There's also a `__complex__` method to support the complex() built-in constructor. Perhaps Vector2d should provide `__complex__` but I'll leave that as an exercise for you.


아래가 지금까지 만들어온 클래스의 최종본이다.

```python

from array import array
import math

class Vector2d:
    typecode = 'd'

    def __init__(self, x, y):
        self.__x = float(x)
        self.__y = float(y)

    @property
    def x(self):
        return self.__x

    @property
    def y(self):
        return self.__y

    def __iter__(self):
        return (i for i in (self.x, self.y))

    def __repr__(self):
        class_name = type(self).__name__
        return '{}({!r}, {!r})'.format(class_name, *self)

    def __str__(self):
        return str(tuple(self))

    def __bytes__(self):
        return (bytes([ord(self.typecode)]) +
                bytes(array(self.typecode, self)))

    def __eq__(self, other):
        return tuple(self) == tuple(other)

    def __hash__(self):
        return hash(self.x) ^ hash(self.y)

    def __abs__(self):
        return math.hypot(self.x, self.y)

    def __bool__(self):
        return bool(abs(self))

    def angle(self):
        return math.atan2(self.y, self.x)

    def __format__(self, fmt_spec=''):
        if fmt_spec.endswith('p'):
            fmt_spec = fmt_spec[:-1]
            coords = (abs(self), self.angle())
            outer_fmt = '<{}, {}>'
        else:
            coords = self
            outer_fmt = '({}, {})'
        components = (format(c, fmt_spec) for c in coords)
        return outer_fmt.format(*components)

    @classmethod
    def frombytes(cls, octets):
        typecode = chr(octets[0])
        memv = memoryview(octets[1:]).cast(typecode)
        return cls(*memv)
```

To recap, in this and the previous sections, we saw essential methods that you may want to implement to have a full-fledged object. Of course, it is a bad idea to implement all of these methods if your application has no real use for them. Customers don't care if your objects are Pythonic or not.



#### Private and Protected Attributes in Python

Python 에서는 private 변수를 만들수 없다. What we have in Python is a simple mechanism to prevent accidental overwriting of a private attribute in a subclass.



