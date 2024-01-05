---
title: DuckDB to HTML  (plain .md to compare with .qmd)
subtitle: Directly convert DuckDB result set to HTML table string
description: |
  Useful to write HTML tables without pandas
categories:
  - Databases
  - HTML
author: LiqC
comments:
  giscus:
    input-position: top
    mapping: pathname
    repo: liquidcarbon/blog
    theme: dark
date: "2023-09-16T00:00"
date-format: "ddd, MMM D YYYY HH:mm" 
image: "DuckDB-to-HTML.png"
image-alt: ""
toc: true
jupyter: python3
---

# Write HTML tables from DuckDB directly

Feature request: https://github.com/duckdb/duckdb/discussions/8961 

## Monkey Patching `DuckDBPyConnection`

```{python}
from duckdb import DuckDBPyConnection

def to_html(
    self,
    classes: str | None = None,
    table_id: str | None = None,
    indent: int = 2,
):
    _indent = " "*indent
    if classes:
        _classes = f' class="{classes}"'
    else:
        _classes = ""

    if table_id:
        _table_id = f' id="{table_id}"'
    else:
        _table_id = ""

    html_table = f"<table{_classes}{_table_id}>\n"
    
    html_table += f"{_indent}<thead>\n"
    html_table += f"{_indent}<tr>\n"
    for col_name in self.description:
        html_table += f"{_indent}{_indent}<th>{col_name[0]}</th>\n"
    html_table += f"{_indent}</tr>\n"
    html_table += f"{_indent}</thead>\n"
    
    for row in self.fetchall():
        html_table += f"{_indent}<tr>\n"
        for value in row:
            html_table += f"{_indent}{_indent}<td>{str(value)}</td>\n"
        html_table += f"{_indent}</tr>\n"
    
    html_table += "</table>"
    return html_table

setattr(DuckDBPyConnection, "to_html", to_html)

import duckdb
con = duckdb.connect()
con.execute("CREATE TABLE items(item VARCHAR, value DECIMAL(10, 2), count INTEGER)")
con.execute("INSERT INTO items VALUES ('jeans', 20.0, 1), ('hammer', 42.2, 2)")

con.execute("SELECT * FROM items")
html = con.to_html(classes="table table-hover", table_id="ducktable")
print(html[:104])
```

```{python}
from IPython.display import HTML
display(HTML(html))
```
