# None-aware attribute getter

In 2015, Mark E. Haase and Steve Dower submitted PEP 505 named "None-aware operators", which has been withdrawn. Still Steve Dower has welcomed people to pick it up.

Indeed, throughout the years, some of PEP 505's ideas have kept resurfacing on a regular basis among Python users who deal with ubiquitous "optional" fields in nested data structures.

- ["Introducing a Safe Navigation Operator in Python" - discuss.python.org](https://discuss.python.org/t/introducing-a-safe-navigation-operator-in-python/35480/)
<!-- TODO: there are more of these discussions, link them -->

PEP 505 wanted to introduce four operators, one of them being the "maybe-dot" `?.` operator.
We propose a focus on this operator to facilitate working with deeply nested data in Python.

<!--
Why "None-aware attribute getter"?
- "None-aware" is a correct and descriptive term that directly references None as a special value, unlike "optional", "maybe" or "safe"; also it directly ties it to PEP 505 and the history of these ideas.
- "attribute getter" helps focus on the problem: getting attributes of an object (I'm not sold on this one)
-->

# Motivation

In many common programming scenarios, including but not restricted to application development, Python users often have to work with deeply nested data structures, extracting values from them.

<!-- FIXME: find a simpler, better example -->
For example, given an API with a role-based permission system where only the fields authorized for the authenticated user are put in a response object, every field at every level of the object would be typed as optional in the API schema.

For instance, consider the following schema of the response's payload of an endpoint of an HTTP JSON API:

```json
{
  "type": "object",
  "properties": {
    "employee": {
      "type": "object",
      "properties": {
        "name": {
          "type": "string"
        },
        "payroll": {
          "type": "object",
          "properties": {
            "frequency": {
              "type": "string"
            },
            "salary": {
              "type": "object",
              "properties": {
                "base": {
                  "type": "object",
                  "properties": {
                    "amount": {
                      "type": "number"
                    },
                    "currency": {
                      "type": "string"
                    }
                  }
                }
              }
            }
          }
        }
      },
      "required": [
        "name"
      ]
    }
  },
  "required": [
    "employee"
  ]
}
```

Examples of valid JSON response's payloads would be:

```json
{"employee": {"name": "Will Smith"}}
{"employee": {"name": "Will Smith", "payroll":{"frequency": "monthly"}}}
{"employee": {"name": "Will Smith", "payroll":{"frequency": "monthly", "salary": {"base": {"amount": 100000, "currency": "USD"}}}}}
```

Let's first assume that the JSON has been deserialized as nested `dict`s.
This is a widespread manner of handling such data, as the `json` module from the CPython standard library provides the `json.loads` function that deserializes JSON objects to `dict`.
In order to get, for instance, the amount of the employee's base salary, a common pattern is to chain `dict.get` and setting intermediate calls' default value to `{}`.

```python
amount: int | None = data.get("employee", {}).get("payroll", {}).get("salary", {}).get("base", {}).get("amount")
```

However, for various reasons, a common pattern is to parse the JSON payload into a Python object.

For instance, using `pydantic`, one may define the following models.

```python
from pydantic import BaseModel


class Employee(BaseModel):
  name: str
  payroll: Payroll | None


class Payroll(BaseModel):
  frequency: str
  salary: Salary | None


class Salary(BaseModel):
  base: Base | None


class Base(BaseModel):
  amount: int | None
  currency: str | None
```

Once the payload is parsed into such a model, it is no longer possible to use the same mechanism that we had with `dict.get`.

Instead, code will look like something similar to:

```python
employee: Employee = ...
amount = (
    employee.payroll.salary.base.amount
    if (
      employee.payroll is not None
      and employee.payroll.salary is not None
      and employee.payroll.salary.base is not None
    )
    else None
)
```

This results in verbose boilerplate code that arguably does not convey the intent.

If Python were to have a None-aware attribute getter as we propose, the same result could be achieved with the code:

```python
employee: Employee = ...
amount = employee?.payroll?.salary?.base
```

