---
layout: api-command
language: JavaScript
permalink: api/javascript/table_wait/
command: tableWait
io:
    -   - r
        - object
---
# Command syntax #

{% apibody %}
r.tableWait('tablename'[, 'tablename' ...]) &rarr; object
{% endapibody %}

# Description #

Wait for a table (or tables) to be ready. A table may be temporarily unavailable after creation, rebalancing or reconfiguring.

The return value is an array of one or more objects providing information about the table's shards, replicas and replica readiness states, one object for each table. Each object has the following fields:

* `db`: database name.
* `name`: table name.
* `id`: table UUID.
* `shards`: an array of objects, one for each shard, with the following keys per object:
    * `primary_replica`: name of the shard's primary server.
    * `replicas`: an array of objects showing the status of each replica, with the following keys:
        * `server`: name of the replica server.
        * `state`: one of `ready` or `transitioning`.
* `status`: an object with the following boolean keys:
    * `all_replicas_ready`
    * `ready_for_outdated_reads`
    * `ready_for_reads`
    * `ready_for_writes

__Example:__ Get a table's status.

```js
r.table('superheroes').tableWait().run(conn, callback);
// Result passed to callback
[
  {
    "db": "database",
    "id": "5cb35225-81b2-4cec-9eef-bfad15481265",
    "name": "superheroes",
    "shards": [
      {
        "primary_replica": null,
        "replicas": [
          {
            "server": "jeeves",
            "state": "ready"
          }
        ]
      },
      {
        "primary_replica": null,
        "replicas": [
          {
            "server": "jeeves",
            "state": "ready"
          }
        ]
      }
    ],
    "status": {
      "all_replicas_ready": true,
      "ready_for_outdated_reads": true,
      "ready_for_reads": true,
      "ready_for_writes": true
    }
  }
]
```