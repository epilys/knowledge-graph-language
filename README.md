# Knowledge Graph Language (KGL)

Knowledge Graph Language is a query language for interacting with graphs. It accepts semantic triples (i.e. `("James", "Enjoys", "Coffee")`), indexes them, and makes them available for querying.

Here is a demo of the language in action:

https://github.com/capjamesg/knowledge-graph-language/assets/37276661/1db1683e-94a0-46bc-9292-3aa28ffc8e72

You can use this language to:

- Find all attributes associated with a node in a graph.
- Return all nodes that are connected to a node.
- Return all nodes that are connected to a node and meet a specified condition.
- Find how two nodes connect in a graph.

This README serves as the official language reference.

The following blog posts discuss the design of the KGL language and the interpreter, respectively:

- [Designing a knowledge graph query language](https://jamesg.blog/2024/03/22/kgl/)
- [Designing an interpreter for Knowledge Graph Language (KGL)](https://jamesg.blog/2024/03/25/kgl-interpreter/)

You can [try the language](https://jamesg.blog/kgl) on a knowledge graph calculated from James' Coffee Blog.

## Data Format

This project allows you to index triples of data like:

```python
("James", "Enjoys", "Coffee")
("James", "Hobbies", "Making coffee")
("James", "WorksFor", "Roboflow")
("Roboflow", "Makes", "Computer Vision")
("Roboflow", "EntityType", "Company")
```

A graph is then constructed from the triples that you can then query.

## Syntax

### Query a Single Item

You can query a single item:

```
{ James }
```

This will return all items associated with the `James` entry:

```
{'Birthday': ['March 20th, 2024'], 'WorksFor': ['Roboflow', 'PersonalWeb', 'IndieWeb'], 'Enjoys': ['Coffee'], 'Hobbies': ['Making coffee']}
```

### Sequential Queries

The Knowledge Graph Language flows from left to right. You can make a statement, then use an arrow (`->`) to query an attribute related to the result:

Consider the following query:

```
{ James -> WorksFor -> Makes }
```

This query gets the `James` item, retrieves for whom James works, then reports the `Makes` attribute for the employer.

The query returns:

```
['Computer vision software.']
```

### Filter Queries

> [!IMPORTANT]  
> This feature is not fully implemented.

You can filter queries so that the flow of data is constrained to only work with results that match a condition.

Consider this query:

```
{ Roboflow -> WorksFor ("Enjoys" = "Coffee") -> Hobbies }
```

This query gets the `Roboflow` node. Then, the query gets everyone who works at Roboflow who enjoys coffee. The query then finds who everyone works for, and returns their hobbies.

This returns:

```
['Making coffee']
```

You can filter by the number of items connected to a node in the graph, too.

Consider these triples:

```
("CLIP", "isA", "Paper")
("CLIP", "Authors", "Person 1")
("CLIP", "Authors", "Person 2")
("Person 1", "Citations", "Paper 42")
("Person 2", "Citations", "Paper 1")
("Person 2", "Citations", "Paper 2")
("Person 2", "Citations", "Paper 3")
("Person 2", "Citations", "Paper 4")
```


Suppose you want to find all authors of the CLIP paper in a research graph, but you only want to retrieve authors whose work has been cited at least three times. You can do this with the following query:

```
{ CLIP -> Authors ("Citations" > "3") }
```

This query returns:

```
['Person 2']
```

This is because only `Person 2` has greater than three citations to their works.

### Describe Relationships

Suppose you want to know how `James` and `Roboflow` relate. For this, you can use the interrelation query operator (`<->`).

Consider this query:

```
{ Roboflow <-> James }
```

This returns:

```
['Roboflow', ('James', 'WorksFor')]
```

Serialized into Knowledge Graph Language, this response is represented as:

```
Roboflow -> WorksFor
```

If we execute that query in introspection mode, we can see all information about James:

```
{ Roboflow -> WorksFor }!
```

This returns:

```
[{'James': {'Birthday': ['March 20th, 2024'], 'WorksFor': ['Roboflow', 'PersonalWeb', 'IndieWeb'], 'Enjoys': ['Coffee'], 'Hobbies': ['Coffee']}}, {'Lenny': {'WorksFor': ['MetaAI', 'Roboflow']}}]
```

### Introspection

By default, all Sequential Queries return single values. For example, this query returns the names of everyone who works at Roboflow:

```
{ Roboflow -> WorksFor }
```

The response is:

```
['James', 'Lenny']
```

We can enable introspection mode to learn more about each of these responses. To enable introspection mode, append a `!` to the end of your query:

```
{ Roboflow -> WorksFor }!
```

This returns all attributes related, within one degree, to James and Lenny, who both work at Roboflow:

```
[{'James': {'Birthday': ['March 20th, 2024'], 'WorksFor': ['Roboflow', 'PersonalWeb', 'IndieWeb'], 'Enjoys': ['Coffee'], 'Hobbies': ['Coffee']}}, {'Lenny': {'WorksFor': ['MetaAI', 'Roboflow']}}]
```

### Description Operators

By default, Knowledge Graph Language returns the value associated with your query. You can add operators to the end of your query to change the output.

You can use:

- `?` to return True if your query returns a response and False if your query returns no response.
- `#` to count the number of responses
- `!` to return an introspection response.
  
## Python API

First, install KGL:

```
pip install kgl
```

### Create a Knowledge Graph

```python
from kgl import KnowledgeGraph

kg = KnowledgeGraph()
```

### Ingest Items

You can ingest triples of strings:

```python
kg.add_node(("Roboflow", "Owned", "Lenny"))
```

You can also ingest triples whose third item is a list:

```python
kg.add_node(("Alex", "Citations", ["MetaAI", "GoogleAI", "Coffee", "Teacup", "Roboflow"]))
```

### Evaluate a Query

```python
result, query_time = kg.evaluate("{ James }")
print(result)
print(query_time)
```

Responses are valid Python objects, whose type varies depending on your query.

By default, KGL returns a list.

But:

- `!` queries return dictionaries.
- `#` queries return integers.
- `?` queries return booleans.

### Create a Node

You can create a node using the `add_node` Python function on a KGL class, or in native KGL.

To create a node in native KGL, use the following syntax:

```
{ subject, predicate, object }
```

For example, the following command would create a node called `taylor swift` and a node called `amazing`, with the `is` predicate connecting them both together:

```
{ taylor swift, is, amazing }
```

## Run the Web Interpreter

To run the KGL web interpreter, first generate a knowledge graph in a CSV file. Each line should use the format `subject, predicate, object`. Then, open `app/app.py` and replace the `all1.csv` reference with the name of your CSV file. Then, install the required dependencies:

```
cd app/
pip3 install -r requirements.txt
```

To run the web interpreter, execute the following command:

```
python3 app.py
```

The interpreter will open at `http://localhost:5000`.

## Tests

To run the project test suite, run:

```
pytest test/test.py
```

## License

This project is licensed under an [MIT license](LICENSE).
