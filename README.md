# None-aware attribute getter

In 2015, Mark E. Haase and Steve Dower submitted PEP 505 named "None-aware operators", which has been withdrawn. Still Steve Dower has welcomed people to pick it up.

Indeed, throughout the years, some of PEP 505's ideas have kept resurfacing on a regular basis among Python users who deal with ubiquitous "optional" fields in nested data structures.

- ["Introducing a Safe Navigation Operator in Python" - discuss.python.org](https://discuss.python.org/t/introducing-a-safe-navigation-operator-in-python/35480/)
<!-- TODO: there are more of these discussions, link them -->

PEP 505 wanted to introduce four operators, one of them being the "maybe-dot" `?.` operator.
We propose a focus on this operator to facilitate working with deeply nested data in Python.

<!--
Why "None-aware attribute getter"?
- "None-aware" is a correct and descriptive term that directly references None as a special value, unlike "optional", "maybe" or "safe"; also it directly ties it to PEP 505 and the history of these ideas.
- "attribute getter" helps focus on the problem: getting attributes of an object (I'm not sold on this one)
-->

