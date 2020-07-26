# Calculating With Functions 


code:
```python
def _process(data, base):
    num = data[0]
    oper = data[1]
    if oper == "*":
        return base * num

    if oper == "/":
        return base // num

    if oper == "+":
        return base + num

    if oper == "-":
        return base - num


def zero(data=None):
    if isinstance(data, tuple):
        return _process(data, 0)

    return 0


def one(data=None):
    if isinstance(data, tuple):
        return _process(data, 1)

    return 1


def two(data=None):
    if isinstance(data, tuple):
        return _process(data, 2)

    return 2


def three(data=None):
    if isinstance(data, tuple):
        return _process(data, 3)

    return 3


def four(data=None):
    if isinstance(data, tuple):
        return _process(data, 4)

    return 4


def five(data=None):
    if isinstance(data, tuple):
        return _process(data, 5)

    return 5


def six(data=None):
    if isinstance(data, tuple):
        return _process(data, 6)

    return 6


def seven(data=None):
    if isinstance(data, tuple):
        return _process(data, 7)

    return 7


def eight(data=None):
    if isinstance(data, tuple):
        return _process(data, 8)

    return 8


def nine(data=None):
    if isinstance(data, tuple):
        return _process(data, 9)

    return 9


def plus(num):
    return (num, "+")


def minus(num):
    return (num, "-")


def times(num):
    return (num, "*")


def divided_by(num):
    return (num, "/")

```