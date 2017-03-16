---
title: "Numba Functions"
teaching: 30
exercises: 20
questions:
objectives:
keypoints:
---
## Calling other functions

Numba functions can call other Numba functions. Of course, both functions must have the `@jit` decorator, otherwise the code will be much slower.

~~~
from numba import jit
@jit("void(f4[:])",nopython=True)
def bubblesort(X):
    N = len(X)
    for end in range(N, 1, -1):
        for i in range(end - 1):
            cur = X[i]
            if cur > X[i + 1]:
                tmp = X[i]
                X[i] = X[i + 1]
                X[i + 1] = tmp
               
@jit
def do_sort():
    sorted[:] = shuffled[:]
    bubblesort(sorted)
~~~
{: .python}


The slowest run took 129.15 times longer than the fastest. This could mean that an intermediate result is being cached 
1000 loops, best of 3: 717 µs per loop

Time how long it takes to run the do_sort() function.

### NumPy universal functions

Numba’s @vectorize decorator allows Python functions taking scalar input arguments to be used as NumPy ufuncs. Creating a traditional NumPy 
ufunc is not the most straightforward process and involves writing some C code. Numba makes this easy. Using the @vectorize decorator, Numba 
can compile a pure Python function into a ufunc that operates over NumPy arrays as fast as traditional ufuncs written in C.

The @vectorize decorator has two modes of operation:
* Eager, or decoration-time, compilation. If you pass one or more type signatures to the decorator, you will be building a Numpy universal function 
  (ufunc). We're just going to consider eager compilation here.
* Lazy, or call-time, compilation. When not given any signatures, the decorator will give you a Numba dynamic universal function (DUFunc) 
  that dynamically compiles a new kernel when called with a previously unsupported input type.

Using @vectorize, you write your function as operating over input scalars, rather than arrays. Numba will generate the surrounding loop 
(or kernel) allowing efficient iteration over the actual inputs. The following code defines a function that takes two integer arrays 
and returns an integer array.

~~~
from numba import vectorize, int64
​
@vectorize([int64(int64, int64)])
def vec_add(x, y):
    return x + y

a = np.arange(6, dtype=np.int64)
print(vec_add(a, a))
b = np.linspace(0, 10, 6, dtype=np.int64)
print(vec_add(b, b))
[ 0  2  4  6  8 10]
[ 0  2  4  6  8 10]
[ 0  4  8 12 16 20]
~~~
{: .python}

This works because NumPy array elements are int64. If the elements are a different type, and the arguments cannot be safely coerced, 
then the function will raise an exception:

~~~
c = a.astype(float)
print(c)
print(vec_add(c, c))
[ 0.  1.  2.  3.  4.  5.]
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-74-9f06063afeeb> in <module>()
      1 c = a.astype(float)
      2 print c
----> 3 print vec_add(c, c)

TypeError: ufunc 'vec_add' not supported for the input types, and the inputs could not be safely coerced to any supported types according to the casting rule ''safe''
~~~
{: .python}

## Challenge
> Redefine the vec_add() function so that it takes float64 as arguments and produces the correct results.
>
> from nose.tools import assert_equal
> c = np.linspace(0, 1, 6)
> assert_equal((c * 2 == vec_add(c, c)).all(), True)
> print("Correct!")
{: .challenge}