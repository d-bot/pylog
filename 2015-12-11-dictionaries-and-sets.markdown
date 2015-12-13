---
layout: post
title: "Dictionaries and Sets"
date: 2015-12-11 10:48:53 +0000
comments: true
categories:
---

파이썬의 dictionary 와 set 은 `hashtable` 로 구현되어 있는데 hashtable 이 어떻게 동작하는지 이해하는것이 dict 와 set 을 잘 사용하는 핵심이다.

`collections.abc` 모듈은 dict 의 인터페이스들을 공식화/형식화(formalize) 하기위해 `Mapping` 과 `MutableMapping` ABC 들을 제공한다 [See](https://docs.python.org/3.3/library/collections.abc.html)

즉, `collections.abc` 모듈은 어떤 클래스가 특정 인터페이스를 제공하는지 테스트 하기 위해 abstract base classes 를 제공한다. 예를 들면 어떤 클래스가 hashable 한지 아니면 mapping 인지 확인하기 위해

#### What is hashable?

An object is hashable if it has a hash value which never changes during its lifetime(it needs a `__hash__()` method), and can be compared to other objects(it needs an `__eq__()` method). Hashable objects which compare equal must have the same hash value.

The atomic immutable types (str, bytes, numeric types) are all hashable. A frozenset is always hashable, because its elements must be hashable by definition. A tuple is hashable only if all its items are hashable.

```python
>>> a = dict(one=1, two=2, three=3)
>>> b = {'one': 1, 'two': 2, 'three': 3}
>>> c = dict(zip(['one', 'two', 'three'], [1, 2, 3]))
>>> d = dict([('two', 2), ('one', 1), ('three', 3)])
>>> e = dict({'three': 3, 'one': 1, 'two': 2})
>>> a == b == c == d == e
True
```

#### dict Comprehensions

위와같이 파이썬 문법적으로 정의하는것 외에도 dict comprehension 으로 dict 를 아래와 같이 정의할 수 있다.

```python
>>> DIAL_CODES = [
...   (86, 'China'),
...   (91, 'India'),
...   (1, 'United States'),
...   (62, 'Indonesia'),
...   (55, 'Brazil'),
...   (92, 'Pakistan'),
...   (880, 'Bangladesh'),
...   (234, 'Nigeria'),
...   (7, 'Russia'),
...   (81, 'Japan')
... ]
>>> country_code = { country: code for code, country in DIAL_CODES }
>>> country_code
{'China': 86, 'India': 91, 'Russia': 7, 'Japan': 81, 'Bangladesh': 880, 'Brazil': 55, 'United States': 1, 'Pakistan': 92, 'Nigeria': 234, 'Indonesia': 62}
>>>
>>>
>>> { code: country.upper() for country, code in country_code.items() if code < 66 }
{1: 'UNITED STATES', 55: 'BRAZIL', 62: 'INDONESIA', 7: 'RUSSIA'}

```

#### Overview of Common Mapping methods

여기서 말하는 매핑은 리스트와 배열을 sequence 라고 말하는것처럼 dictionary / hash 타입들을 통틀어서 매핑이라고 하는것 같음.

기본적인 dict 의 매핑 API 는 굉장히 다양하다. 그중 가장 유용한 2가지 variations 는 collections 모듈에 정의되어 있는 `defaultdict` 와 `OrderedDict` 이다. 그중 update 를 살펴보면

```python
#d.update(m, [**kargs])  # Update d with items from mapping or iterable of (key, value) pairs
# kargs = key arguments

>>> a = {'one': 3, 'two': 55}
>>> b = {'three': 30, 'four': 5335}
>>> a.update(b)
>>> a
{'four': 5335, 'three': 30, 'two': 55, 'one': 3}
>>> a.update([(1, 2), (3, 4)])
>>> a
{1: 2, 3: 4, 'three': 30, 'two': 55, 'four': 5335, 'one': 3}
```

`update` 가 첫번째 argument m 을 다루는 것은 `duck typing` 의 좋은 예제이다. 즉, `update` 는 m 이 `keys` 함수를 가지고 있는지 검사하고 있으면 m 을 `mapping` 으로 간주한다. 아니면, `update` 는 falls back to iterating over m. assuming its items are (key ,value) pairs.

The constructor for most Python mappings uses the logic of `update` internally, which means they can be initialized from other mappings or from any iterable object producing(key, value) pairs.

#### Handle Missing Kyes with setdefault

In line with the `fail-fast` philosopy (빠르게 실패하기 철학의 방침에 따르면), dict access with d[k] raises an error when k is not an existing key. Every Pythonista knows that d.get(k, default_value) is an alternative to d[k] whenever a default value is more convenient than handling `KeyError`.

However, when updating the value found(if it is mutable), using either `__getitem__` or `get` is awkward and inefficient. 아래의 예제는 missing key 를 `dict.get` 을 이용해서 다루는 좋지 않은 예를 보여준다.

```python
import sys
import re

WORD_RE = re.compile('\w+')

index = {}

with open(sys.argv[1], encoding='utf-8') as fp:
    for line_no, line in enumerate(fp, 1):
        for match in WORD_RE.finditer(line):
            word = match.group()
            column_no = match.start() + 1
            location = (line_no, column_no)
            # This is ugly; coded like this to make a point
            #occurrences = index.get(word, [])
            #occurrences.append(location)
            #index[word] = occurrences

            # Above 3 lines can be replaced as follows
            index.setdefault(word, []).append(location)

for word in sorted(index, key=str.upper):
    print(word, index[word])

"""
$ python3 index.py test.txt
asdf [(1, 6), (2, 1)]
qwer [(3, 1)]
test [(1, 1), (3, 6)]
"""
```

즉, setdefault 는 해당 키값을 못찾으면 자동으로 디폴트값을 입력해주어서 KeyError 를 발생시키지 않는다.

#### Mappings with Flexible Key Lookup

`defaultdict` 혹은 `dict` 를 서브클래싱하여 missing key 가 검색될때 미리 준비된 값을 리턴할수 있다.

defaultdict 는 초기화 될때 디폴트 값을 가지는 callable 을 제공하도록 한다. 즉, 해당 callable 은 `__getitem__` 이 존재하지 않는 키값으로 호출될때 제공된다. 예를 들어 `dd = defaultdict(list)` 존재하지 않는 `new-key` 를 이용하여 값을 호출하면 아래와 같은 일이 발생한다.
1. Calls `list()` to create a new list
2. Inserts the list into `dd` using 'new-key' as key
3. Returns a reference to that list.

#### The `__missing__` method

`__missing__` 은 기본적으로 dict 클래스에 정의되어 있지 않으나 dict is aware of it: `dict` 를 서브클래스하고 `__missing__` 를 정의하면 the standard `dict.__missing__` will call it whenever a key is not found, instead of raising KeyError

`__missing__` 는 `__getitem__` 의해서 호출되는데 `__getitem__` 은 d[k] 형식으로 호출되고 d.get(k) 형식이나 `__contain__` 으로는 호출되지 않는다.




