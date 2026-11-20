# Nodetool tasks

**tasks** - Nodetool supercommand for managing task manager tasks.

Task manager is an API-based tool for tracking long-running background operations, such as repair or compaction,
which makes them observable and controllable. Task manager operates per node.

## Supported tasks suboperations

* [abort](https://opensource.docs.scylladb.com/stable/operating-scylla/nodetool-commands/tasks/abort.md) - Aborts the task.
* [list](https://opensource.docs.scylladb.com/stable/operating-scylla/nodetool-commands/tasks/list.md) - Lists tasks in the module.
* [modules](https://opensource.docs.scylladb.com/stable/operating-scylla/nodetool-commands/tasks/modules.md) - Lists supported modules.
* [status](https://opensource.docs.scylladb.com/stable/operating-scylla/nodetool-commands/tasks/status.md) - Gets status of the task.
* [tree](https://opensource.docs.scylladb.com/stable/operating-scylla/nodetool-commands/tasks/tree.md) - Gets statuses of the task and all its descendants.
* [ttl](https://opensource.docs.scylladb.com/stable/operating-scylla/nodetool-commands/tasks/ttl.md) - Gets or sets task_ttl value.
* [wait](https://opensource.docs.scylladb.com/stable/operating-scylla/nodetool-commands/tasks/wait.md) - Waits for the task and gets its status.
