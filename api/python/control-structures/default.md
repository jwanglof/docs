---
layout: api-command
language: Python
permalink: api/python/default/
command: default
---

# Command syntax #

{% apibody %}
value.default(default_value | function) &rarr; any
sequence.default(default_value | function) &rarr; any
{% endapibody %}

# Description #

Provide a default value in case of non-existence errors. The `default` command evaluates its first argument (the value it's chained to). If that argument returns `None` or a non-existence error is thrown in evaluation, then `default` returns its second argument. The second argument is usually a default value, but it can be a function that returns a value.

__Example:__ Retrieve the titles and authors of the table `posts`.
In the case where the author field is missing or `None`, we want to retrieve the string
`Anonymous`.

```py
r.table("posts").map(lambda post:
    {
        "title": post["title"],
        "author": post["author"].default("Anonymous")
    }
).run(conn)
```

<!-- stop -->

We can rewrite the previous query with `r.branch` too.

```py
r.table("posts").map(lambda post:
    r.branch(
        post.has_fields("author"),
        {
            "title": post["title"],
            "author": post["author"]
        },
        {
            "title": post["title"],
            "author": "Anonymous" 
        }
    )
).run(conn)
```


__Example:__ The `default` command can also be used to filter documents. Retrieve all our users who are not grown-ups or whose age is unknown
(i.e., the field `age` is missing or equals `None`).

```py
r.table("users").filter(lambda user:
    (user["age"] < 18).default(True)
).run(conn)
```

One more way to write the previous query is to set the age to be `-1` when the
field is missing.

```py
r.table("users").filter(lambda user:
    user["age"].default(-1) < 18
).run(conn)
```

This can be accomplished with [has_fields](/api/python/has_fields/) rather than `default`.

```py
r.table("users").filter(lambda user:
    user.has_fields("age").not_() | (user["age"] < 18)
).run(conn)
```

The body of every [filter](/api/python/filter/) is wrapped in an implicit `.default(False)`. You can overwrite the value `False` with the `default` option.

```py
r.table("users").filter(
    lambda user: (user["age"] < 18).default(True),
    default=True
).run(conn)
```

__Example:__ The function form of `default` receives the error message as its argument.

```py
r.table("posts").map(lambda post:
    {
        "title": post["title"],
        "author": post["author"].default(lambda err: err)
    }
).run(conn)
```

This particular example simply returns the error message, so it isn't very useful. But it would be possible to change the default value based on the specific error message thrown.
