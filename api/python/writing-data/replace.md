---
layout: api-command
language: Python
permalink: api/python/replace/
command: replace
related_commands:
    insert: insert/
    update: update/
    delete: delete/
---

# Command syntax #

{% apibody %}
table.replace(json | expr[, durability="hard", return_changes=False, non_atomic=False])
    &rarr; object
selection.replace(json | expr[, durability="hard", return_changes=False, non_atomic=False])
    &rarr; object
singleSelection.replace(json | expr[, durability="hard", return_changes=False, non_atomic=False])
    &rarr; object
{% endapibody %}

<img src="/assets/images/docs/api_illustrations/replace.png" class="api_command_illustration" />

# Description #

Replace documents in a table. Accepts a JSON document or a ReQL expression, and replaces
the original document with the new one. The new document must have the same primary key
as the original document.

The optional arguments are:

- `durability`: possible values are `hard` and `soft`. This option will override the
table or query's durability setting (set in [run](/api/python/run/)).  
In soft durability mode RethinkDB will acknowledge the write immediately after
receiving it, but before the write has been committed to disk.
- `return_changes`: if set to `True`, return a `changes` array consisting of `old_val`/`new_val` objects describing the changes made.
- `non_atomic`: if set to `True`, executes the replacement and distributes the result to replicas in a non-atomic fashion. This flag is required to perform non-deterministic updates, such as those that require reading data from another table.

Replace returns an object that contains the following attributes:

- `replaced`: the number of documents that were replaced
- `unchanged`: the number of documents that would have been modified, except that the
new value was the same as the old value
- `inserted`: the number of new documents added. You can have new documents inserted if
you do a point-replace on a key that isn't in the table or you do a replace on a
selection and one of the documents you are replacing has been deleted
- `deleted`: the number of deleted documents when doing a replace with `None`
- `errors`: the number of errors encountered while performing the replace.
- `first_error`: If errors were encountered, contains the text of the first error.
- `skipped`: 0 for a replace operation
- `changes`: if `return_changes` is set to `True`, this will be an array of objects, one for each objected affected by the `replace` operation. Each object will have two keys: `{"new_val": <new value>, "old_val": <old value>}`.

__Example:__ Replace the document with the primary key `1`.

```py
r.table("posts").get(1).replace({
    "id": 1,
    "title": "Lorem ipsum",
    "content": "Aleas jacta est",
    "status": "draft"
}).run(conn)
```

__Example:__ Remove the field `status` from all posts.

```py
r.table("posts").replace(lambda post:
    post.without("status")
).run(conn)
```

__Example:__ Remove all the fields that are not `id`, `title` or `content`.

```py
r.table("posts").replace(lambda post:
    post.pluck("id", "title", "content")
).run(conn)
```

__Example:__ Replace the document with the primary key `1` using soft durability.

```py
r.table("posts").get(1).replace({
    "id": 1,
    "title": "Lorem ipsum",
    "content": "Aleas jacta est",
    "status": "draft"
}, durability="soft").run(conn)
```

__Example:__ Replace the document with the primary key `1` and return the values of the document before
and after the replace operation.

```py
r.table("posts").get(1).replace({
    "id": 1,
    "title": "Lorem ipsum",
    "content": "Aleas jacta est",
    "status": "published"
}, return_changes=True).run(conn)
```

The result will have a `changes` field:

```py
{
    "deleted": 0,
    "errors":  0,
    "inserted": 0,
    "changes": [
        {
            "new_val": {
                "id":1,
                "title": "Lorem ipsum"
                "content": "Aleas jacta est",
                "status": "published",
            },
            "old_val": {
                "id":1,
                "title": "Lorem ipsum"
                "content": "TODO",
                "status": "draft",
                "author": "William",
            }
        }
    ],   
    "replaced": 1,
    "skipped": 0,
    "unchanged": 0
}
```
