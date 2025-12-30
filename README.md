# nested_lookup

Make working with JSON, YAML, and XML document responses fun again!

The `nested_lookup` package provides many Python functions for working with deeply nested documents.
A document in this case is a mixture of Python dictionary and list objects typically derived from YAML or JSON.

## Features

**nested_lookup:** Perform a key lookup on a deeply nested document. Returns a `list` of matching values.

**nested_update:** Given a document, find all occurrences of the given key and update the value. By default, returns a copy of the document. To mutate the original specify the `in_place=True` argument.

**nested_delete:** Given a document, find all occurrences of the given key and delete it. By default, returns a copy of the document. To mutate the original specify the `in_place=True` argument.

**nested_alter:** Given a document, find all occurrences of the given key and alter it with a callback function. By default, returns a copy of the document. To mutate the original specify the `in_place=True` argument.

**get_all_keys:** Fetch all keys from a deeply nested dictionary. Returns a `list` of keys.

**get_occurrence_of_key/get_occurrence_of_value:** Returns the number of occurrences of a key/value from a nested dictionary.

For examples on how to invoke these functions, please check out the tutorial sections.

## Installation

Install from PyPI using pip:

```bash
pip install nested-lookup
```

Or install from source:

```bash
git clone https://github.com/russellballestrini/nested-lookup.git
cd nested-lookup
pip install .
```

## Quick Tutorial

Before we start, let's define an example document to work on:

```python
document = [{"taco": 42}, {"salsa": [{"burrito": {"taco": 69}}]}]
```

First, we lookup a key from all layers of a document using `nested_lookup`:

```python
from nested_lookup import nested_lookup

print(nested_lookup("taco", document))
# Output: [42, 69]
```

As you can see, the function returned a list of two integers - the values from the matched key lookups.

Next, we update a key and value from all layers of a document using `nested_update`:

```python
from nested_lookup import nested_update

result = nested_update(document, key="burrito", value="test")
print(result)
# Output: [{'taco': 42}, {'salsa': [{'burrito': 'test'}]}]
```

Here you see that the key `burrito` had its value changed to the string `'test'`, as expected.

Finally, we try out a delete operation using `nested_delete`:

```python
from nested_lookup import nested_delete

result = nested_delete(document, "taco")
print(result)
# Output: [{}, {'salsa': [{'burrito': {}}]}]
```

Perfect! The returned document looks just like we expected!

## Advanced Usage

You may control the function's behavior by passing some optional arguments:

**wild** (defaults to `False`): If `wild` is `True`, treat the given `key` as a case-insensitive substring when performing lookups.

**with_keys** (defaults to `False`): If `with_keys` is `True`, return a dictionary of all matched keys and a list of values.

For example, given the following document:

```python
from nested_lookup import nested_lookup

my_document = {
    "name": "Rocko Ballestrini",
    "email_address": "test1@example.com",
    "other": {
        "secondary_email": "test2@example.com",
        "EMAIL_RECOVERY": "test3@example.com",
        "email_address": "test4@example.com",
    },
}
```

Next, we could act `wild` and find all the email addresses like this:

```python
results = nested_lookup(
    key="mail",
    document=my_document,
    wild=True
)

print(results)
# Output: ["test1@example.com", "test4@example.com", "test2@example.com", "test3@example.com"]
```

Additionally, if you also needed the matched key names, you could do this:

```python
results = nested_lookup(
    key="mail",
    document=my_document,
    wild=True,
    with_keys=True,
)

print(results)
# Output:
# {
#   "email_address": ["test1@example.com", "test4@example.com"],
#   "secondary_email": ["test2@example.com"],
#   "EMAIL_RECOVERY": ["test3@example.com"]
# }
```

### Delete and Update Examples

Let's delete and update our deeply nested key/values and see the results:

```python
from nested_lookup import nested_update, nested_delete

# Delete EMAIL_RECOVERY key
result = nested_delete(my_document, "EMAIL_RECOVERY")
print(result)
# Output: {'other': {'secondary_email': 'test2@example.com', 'email_address': 'test4@example.com'}, 'email_address': 'test1@example.com', 'name': 'Rocko Ballestrini'}

# Update 'other' value
result = nested_update(my_document, key="other", value="Test")
print(result)
# Output: {'other': 'Test', 'email_address': 'test1@example.com', 'name': 'Rocko Ballestrini'}
```

### Get All Keys

Now let's say we wanted to get a list of every nested key in a document:

```python
from nested_lookup import get_all_keys

keys = get_all_keys(my_document)
print(keys)
# Output: ['name', 'email_address', 'other', 'secondary_email', 'EMAIL_RECOVERY', 'email_address']
```

### Get Occurrence Counts

To get the number of times a key or value occurs in the document:

```python
from nested_lookup import get_occurrence_of_key, get_occurrence_of_value

# Count key occurrences
key_count = get_occurrence_of_key(my_document, key="email_address")
print(key_count)  # Output: 2

# Count value occurrences
value_count = get_occurrence_of_value(my_document, value="test2@example.com")
print(value_count)  # Output: 1
```

### Get Occurrences and Values

To get the number of occurrences and their respective values:

```python
from nested_lookup import get_occurrences_and_values

my_documents = [
    {
        "processor_name": "4",
        "processor_speed": "2.7 GHz",
        "core_details": {
            "total_numberof_cores": "4",
            "l2_cache(per_core)": "256 KB",
        }
    }
]

result = get_occurrences_and_values(my_documents, value="4")
print(result)
# Output:
# {
#   "4": {
#     "occurrences": 2,
#     "values": [
#       {
#         "processor_name": "4",
#         "processor_speed": "2.7 GHz",
#         "core_details": {
#           "total_numberof_cores": "4",
#           "l2_cache(per_core)": "256 KB"
#         }
#       },
#       {
#         "total_numberof_cores": "4",
#         "l2_cache(per_core)": "256 KB"
#       }
#     ]
#   }
# }
```

## nested_alter Tutorial

**Nested Alter**: Write a callback function which processes a scalar value. Be aware of the possible types which can be passed to the callback functions. In this example we can be sure that only int will be passed, but in production you should check the type because it could be anything.

Before we start, let's define an example document:

```python
document = [{"taco": 42}, {"salsa": [{"burrito": {"taco": 69}}]}]
```

Define a callback function:

```python
def callback(data):
    return data + 10  # add 10 to every taco price
```

The alter-version only works for scalar input (one dict). If you need to address a list of dicts, you have to manually iterate over those and pass them to nested_alter one by one:

```python
from nested_lookup import nested_alter

out = []
for elem in document:
    altered_document = nested_alter(elem, "taco", callback)
    out.append(altered_document)

print(out)
# Output: [{'taco': 52}, {'salsa': [{'burrito': {'taco': 79}}]}]
```

## Type Hints

This package includes type hints for better IDE support and type checking with tools like mypy.

## License

Public Domain

## Authors

- Russell Ballestrini
- Douglas Miranda
- Ramesh RV
- Salfiii (Florian S.)
- Matheus Lins

## Links

- https://russell.ballestrini.net
- http://douglasmiranda.com
- https://github.com/Salfiii
- https://github.com/matheuslins
