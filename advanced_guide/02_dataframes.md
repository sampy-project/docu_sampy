# Dataframes in Sampy

Most objects in Sampy have a dataframe as attribute, storing various informations (about vertices in the case of a graph, about individual agents in the case of a population, etc...). In Sampy, we use a custom kind of dataframe called DataFrameXS.

## What is a DataFrameXS

A DataFrameXS is essentially a container for 1D numpy arrays, with a few quality of life features like the ability to shuffle its rows. It is design to feel like pandas dataframe, with a few notable difference.

1. Columns of a dataframe `df` are created and accessed using square brackets:
    ```python
    df['name_column'] = some_1d_array # create a new column
    ```

## Why not Pandas?

