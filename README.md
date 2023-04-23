## PyBT

## Installation 

```python
pip install python-property-based-testing 
```

## Introduction 

You can find a primer on property-based testing [here](https://vrindisbacher.github.io/pybt.pdf). 

`PyBT` is a library for property based testing in python. The main idea, is to state and randomly test properties about functions at large scale. The main functionality `PyBT` provides is a decorator named `pybt`. 

The parameter takes the following arguments:

* **n**: the number of tests to run (defaults to 1000).

* **generators**: user provided generators to be used to generate function arguments (defaults to None).

* **hypotheses**: hypotheses that generated arguments are constrained by. 

* **max_basic_arg_size**: maximum size of string, ints, floats etc. 

* **max_complex_arg_size**: maximum size to use for complex structures like list and dict. Also applies to the depth of random types generated by any. 

## Types Supported 

`PyBT` supports basic types, union types, and nested types. It supports `list`, `dict`, `int`, `str`, `float`, `bool`. It also supports `any`, which will generate a random type, and arguments of that type. The types generated by `any` are constrained to `list`, `dict`, `int`, `str`, `float`, and `bool`. 

There are plans to add random generation of classes and callables in the future. It is always possible to test functions with types not supported by PyBT. To do this, one needs only use custom generators. You can see an example below, in the section `Generators`. 

## Use and Example 

`PyBT` is used by decorating test cases with the `pybt` function. The key difference to other kinds of testing, is that the test defined only states a **property** that should hold for a function or program. `PyBT` will then test this property `n` times by generating random arguments matching the type annotations of your test. Let's run through an example. 


```python
from unittest import TestCase
from pybt.core import pybt


def rev(l):
    return l[::-1]


class TestRevSimple(TestCase):
    pybt_small = pybt(max_complex_arg_size=5, max_basic_arg_size=1000)

    @pybt_small
    def test_rev(self, l: list):
        assert rev(rev(l)) == l
```

Here we have a function named `rev`, which reverses a list. We want to test this function using the `unittest` framework. We declare everything as we would with a normal unit test, with a few caveates. The most important distinction is that our test is a property over an abstracted list `l`. We are not specifying the actual test case by defining what `l` might be. 

First we import `pybt` from PyBT. We also import the `TestCase` class from the unittest framework. Then, we declare our test class, which inherits from `TestCase`. This is common practice in python unit testing. 

Then, we declare a modified decorator `pybt_small`, which sets the maximum basic type size to `1000`, and the maximum complex arg size to `5`. Finally, we declare the property we'd like to test in the method `test_rev`. We decorate this function with `pybt_small`. Notice that the type of `l` is a generic `list`. This is equivalent to annotating `l` as `list[any]`. 

We can now run `test_rev` through the `unittest` framework. When we do this, `pybt_small` will generate `n = 1000` random cases of `list[any]`. If we want to see what the test cases are, we can simply `print(l)` within `test_rev`. 

```python 
[201.71629512275953, True, True, False]
[True, -342.3693684184344]
[True, True, False]
[-467.5006903969188]
[558.8977953526747, 150, 263.0229074872361, -1.0862251449434799]
[67.66371838907641, -20.983597734832696, -55, False, -224.6313237684315]
[True, 618.2506920564614]
[False, -168.82956631670407, True, True]
[266.1946254289064, -139.94823413505858, 28, -260.6052834651554, False]
[False]
[True, 96, -500.6663721540626, -458.2161805936233]
[22.251442861132976, 2.7328417702831436, -239.33983760608413, -249, False]
...
```

Running this again, we see that both the generated lists and their types are different. 

```python 
[{'OqbXQRTbZjesxinNdgxwjkrNqCDSVbhXloFYgvNHjlDmjHglDFihmmMPgrnEzYgmMjKbvuNayXkIMlGxQWrEHquLvtiYpuJRriPmFzKuGgsZBUfSCPXTHNVQWdsqXFdfNkbdVZnVEvhYTnEanGOfONjGRssGpHAesEkPdblYTqZebfbbEsjtAYhBczhJLvEEvOfamgcjsyEsnNagDxCrmiEDKUlxgQcCourCnxxkqNDdfQdyWMtXjeUgUVIYAiTipPBVdrBrxnSBtBhrOwSeyZbmoovrqkMhRiZebi': 'LqcgYSTdxAVEokqYnJMOBKiZAxIKdJBGQnIPtnJWrnIQshzjobsDikLBqJiIDvGoUCOKqbgTxqBhGcbwRanVNXBc', 'VlxxfsXmfRtkDLvEkMUeetRWNFmUhZTmXxNmsrFmKMJSWYKtgyyrFFxFVhZOPskXludESQCfbDGjiaItGpsbbMCCXhRrSYcKvdozuoKcnqcovPPawnJDWqZKxaNrfrLqypyAyNQUiKgHYDvqcDtFUMFFaRZMheGACyuACWJiCZUzjFlSIYhPvatsArIpAPgOoESeGkMBMdUwhsdLfGCYShNXapuuvmItPkQLjxkafwOeMECJcmaIgCxvPXTLHbdhQLSyaoeVbpfjHnfVFvQdaCPofARZLDlLYLQVzlhcivPiEXuOsRiLdmfvjpxBioccscYhnqASfsVYLQzkuKwDQeMNDcSBVeGBJAmiTHazSFqKtTlzYizVMJRfBWUGIrJYpIrsxsGcwaXUkdneUSvHgwiJEWqZFEhCUJpBTaakjrnwiJPmScrOTsOIZUIDAumnWyAkVMDRBSZeQAocgbvNyipFJMDAeJrRdRSFYPDRmwcbEBDlTKykexOGMtiOlLqlXWGlxPyaIOvyuuQrptxxxzwrhfPlfAIPscohiXvIapzhBQrwHCZPGtddIKpRTPunHuuJEVuRRHJbFYQYzfhGWxizJtAAEgYjPQFrSMuchvhifpURmizpFpUBQKolSakHUamKBIuzvNWrZqWgliJhoHwrQRYDTtsMVFEhUZJBWqqrrEqpH': 'WJEVtzSUTSmRlHGbnJkuAzLDhizFnVRoeZeofrLZckypBJmMWQuRbWAXyEzSboXwPnkoPBLqILGuBwkYVQBOEiVvfxylpGaiPuFYpxnsPblCNnQqusHuEklSltNivYQUZEXnVZUvmkDcbZLWgqFlzziwJPWBVGovmQGAcGEYUUtsNSrtjIHQHIIbUNnAkigFedoynnLHZpVwoTSdkvDeDBzaQxyLtyzRqchxEiEqvVOttDrzfRXJsHDUVDKYtionmXXihsLeFpEpZYpwVEdivqCcMtRAZtQsWNQtcFBgeXteacnMgeHtuEnRJCWhlBwIgHZdRXDcajycfeuuZjvAxvRUmdPHyqVsJkDzyVUgogKqYhcUORwcjiKsAXxJxfYsdVhePqOnwtpFIYCnSCTNJVfQHzpxDYAglKsepxsKqsufscDWpOkrAmHXikCFtpoHeSwbkEEzrlvPAyNxnntcfEWQQxJxgvYkRxLDtCWNvjUXvBZRqOlQIWGPLgPApaIxtNjsqWnuzsymHXtxYJnOxkJzgIyZnpkzdGrflJmncnPEeOhlgpUKnkASSuRnocvHNOMCnyJGhkWybKZisfMXcfiCysUKyxJEuqTOEfkzgWSHdyp', 'UUjaAnpquicDjbeqKWghUjJqvXQlWgeSwDZfvJwefygaRfouOCMzbBysRwSAjNKsWjhYjTdtsXjYEROVaTypRgtKblnLhdYBVMtvQAnMgEApoHnqZFRqCzRPOAMetgWoGimhWHhxSYVbCWrDkWxiJPtUEyC': -338.5739091864339}]
[176, {101: -245}, -899, {496: [{}, [-763, False, 51.61292833180636, -704.6588436548865]], 657: -220, -36: 496, -870: [{'mHeJJRaBYYIgzivUkaGhCciYICrHuCDUCLvwmGGhHetxwhzdzpHskZNXwuqJwEiuhJfjuFJNCdNclseiVUNteLvvUKsIBhQqgwySkkiDebsGfmBEiZZdXWmYMqRMXHqeHtOeMjyAgjOeLOBIklpYTjMoWWWWBE': 694.309135177688}, True, False, [515, -132.03329321479674], True], 129: 672}, {287: 'pztxwzqtxGwmXaydOBfScIczLFbTSXxHEYMCQTLIiIXeLbBzXllgHJqzOSJRbPqPXVZxwhbwQYdjxPqUmvwaRxfSWqlBCJQPBaLSJoWdAfVkJyyyXQLaeOZAQRhjMairmGiuzvAlzJxCqfFREOsgguowxtsmldbdYBQZbumdXkzLLfnCgvdqaFvEchoKiIIrJqZaByTNRHvMMyzNupHYjDdEocdUcBikQycuurOUGfXFRNjPPjEzTuYTkoCdWjxkjgMJPyLfgwNddOyeJglITnZmsZJzzEVPzmSrDmHAzGSkjYoNCQStyespkUtwPPDzFrCziQIxQLFhvtxjpYZUyIOHzWKcTGvEPaeyHWtoisieNrGgYinewuAYMzsCPbRjBydVLoZtkIZEgdtvHzibbKrCKxeBSBjSSMDQgzgMLLwVmpWIqESlgiqnToqIROFQvgZRHBemwdggcSSVABBqFuViQynQGuxMsrJLqokLRxCGtLKoXYuMFzFqumzjhNAeYwJLSZgGmoezGsxabNfAuknkKNqfIU', 77: [[-13.487063433086522, -85.48259578583449]]}]
...
```

As we can see, our second run generated much more complex lists than our first. Both the scale and complexity of the arguments generated give us strong guarantees that the property we enforced for our function `rev` holds. 

## Advanced Use 

### Hypotheses 

If you need to constrain the arguments that are passed to your function, you can constrain them using `hypotheses`. For example, if we wanted every single integer that appeared in `l` to be less than `2` for `rev`, we would specify that in the following way: 

```python 

def constrain_l(l : list):
    for el in l: 
        if type(el) == int:
            if el > 2:
                return False 
    return True 


hypotheses = {
    "l" : lambda l : constrain_l(l)
}


pybt_small = pybt(max_complex_arg_size=5, max_basic_arg_size=1000, hypotheses=hypotheses)
```

### Generators 

If you have arguments that should be generated in a very specific way, then you can provide your own generators, through the `generators` argument. For example, let's see we only wanted to our rev function on lists of ints that are coerced to strings. We could do the following. 

```python
import random 


def gen_l():
    l = []
    for _ in range(5):
        el = random.randint(1000)
        l.append(str(el))
    return l


generators = {
    "l" : gen_l 
}


pybt_small = pybt(max_complex_arg_size=5, max_basic_arg_size=1000, generators=generators)
```
