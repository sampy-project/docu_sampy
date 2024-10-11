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
4. For various reasons, DataFrameXS allow empty columns. When retrieving an empty column (which would generally happen if the dataframe is empty), one get a `None`. This may cause errors when trying to do array-like operation (such as `array > 0`). 

## How DataFrameXS works with Numba?

Numba is a powerful library that allows to write compiled functions manipulating numpy arrays in pure python. Thanks to numba, we can write   
