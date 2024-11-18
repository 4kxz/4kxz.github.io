+++
title = 'A Fast Way to Save Pandas DataFrames to PostgreSQL'
date = 2022-07-13T15:01:01+02:00
draft = false
tags = ["Python", "SQL"]
+++
I found Pandas' built-in `DataFrame.to_sql()` too slow for my use case, so here's a faster way to load a DataFrame to a PostgreSQL database using the `COPY` command. 


```python
from io import StringIO

from pandas import DataFrame

from config import postgresql_engine


def save_dataframe_to_postgres(dataframe: DataFrame, table_name: str):
    """Saves DataFrame to Postgres, overwriting the existing table."""
    dataframe = dataframe.reset_index()  # This turns the index into a normal column
    # Create empty table (drops it before if it exists)
    dataframe[0:0].to_sql(table_name, con=postgresql_engine, if_exists='replace')
    # Save the CSV to a string
    f = StringIO()
    dataframe.to_csv(f, header=False, index=False)
    f.seek(0)
    # This next step uses psycopg's copy function, won't work with other engines
    fmt = """COPY "{}" ({}) FROM STDIN WITH (FORMAT CSV, HEADER FALSE)"""
    fields = ', '.join('"{}"'.format(x) for x in dataframe.columns)
    query = fmt.format(table_name, fields)
    connection = postgresql_engine.raw_connection()
    with connection.cursor() as cursor:
        cursor.copy_expert(query, f)
    connection.commit()
```

There are some tradeoffs. The DF has to fit in memory and it relies on psycopg copy, it won't work with other connectors or DBs. It might be optimized further by splitting the data in chunks and saving in parallell, but this was good enough for my needs.