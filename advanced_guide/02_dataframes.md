# Dataframes in Sampy

Most objects in Sampy have a dataframe as attribute, storing various informations (about vertices in the case of a graph, about individual agents in the case of a population, etc...). In Sampy, we use a custom kind of dataframe called DataFrameXS.

## What is a DataFrameXS

A DataFrameXS is essentially a container for 1D numpy arrays, with a few quality of life features like the ability to shuffle its rows. It is design to feel like pandas dataframe, with a few notable difference.

1. Similarly to pandas dataframe, columns of a DataFrameXS are created and accessed using square brackets.
    ```python
    df['name_column'] = some_1d_array # create a new column
    x = df['name_column'] # get a REFERENCE to an already created column
    ```
2. When a column is created, the dataframe stores it as a 1D numpy array. This never changes, i.e. contrary to what happens in Pandas, memory layout always remains the same within the dataframe.
3. When retrieving a column, one get a reference to the underlying 1D array. This allows functions and methods to directly modify the values stored in columns, which is extremely useful for optimization. 
4. For various reasons, DataFrameXS allow empty columns. When retrieving an empty column (which would generally happen if the dataframe is empty), one get a `None`. This may cause errors when trying to do array-like operation (such as `array > 0`). The fact that requesting en empty column returns `None` will likely change in the future, since it causes unecessary complications in ABMs' scripts.

## How DataFrameXS works with Numba?

### What is Numba?
Numba is a powerful library that allows to write compiled functions manipulating numpy arrays in pure python. Thanks to numba, we can write computationaly efficient functions while keeping a hand on memory allocation. In order to properly describe what we mean with this last point, let us focus on a toy example. Let say that we have a one dimensional array of floats `X`, and that we want to modify the values in `X` in place as follows : if `X[i] < 0`, then we replace `X[i]` by `-X[i]`, else we replace it with `exp(X[i])`. With numba, such a function can be written as follows.

```python
@nb.njit
def toy_func(X):
    for i in range(X.shape[0]):
        if X[i] >= 0:
            X[i] = np.exp(X[i])
        else:
            X[i] = -X[i]
```

Note that, in this simple case, it is possible to write with numpy alone a function that will do this modification in place. However, it is in general more efficient and easier to read with Numba. 

### Numba and DataFrameXS

The core reason why the syntax `df['name_column']` returns a reference to the underlying numpy array (and not a copy or another type of object) is to allow them to be directly passed as parameters to numba-compiled functions, and allow them to be modified in place by such functions. This allows for efficient computation, since (properly written) functions decorated with numba are compiled, and since it gives a lot of control over memory allocation.
