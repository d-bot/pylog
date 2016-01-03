---
layout: post
title: "Code Snippets"
date: 2016-01-03 22:07:09 +0000
comments: true
categories: 
---

#### 점수별 학점 계산


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

#### Dict Comprehension

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
