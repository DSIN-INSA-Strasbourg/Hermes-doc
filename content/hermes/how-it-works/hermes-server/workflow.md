---
title: Workflow
weight: 1
---

hermes-server

- 1\. loads its local cache
- 2\. checks if its dataschema has changed since last run, and emits the resulting removed events (if any), and the new dataschema
- 3\. fetches all data required by its datamodel from datasource(s)
  - 3.1. enforces merge constraints
  - 3.2. merges data
  - 3.3. replaces inconsistencies and merge conflict by cached values
  - 3.4. enforce integrity constraints
- 4\. generate a diff between its cache and the fetched remote data
- 5\. loop over each diff type : added, modified, removed
  - 5.1. for each diff type, loop over each data type in their declaration order in the datamodel, except for removed diff type, for which it is the reverse declaration order
    - 5.1.1. loop over each diff item of current data type
      - 5.1.1.1. generate the corresponding event
      - 5.1.1.2. emit the event on message bus
      - 5.1.1.3. if event was successfully emitted :
        - 5.1.1.3.1. run datamodel `commit_one` action if any
        - 5.1.1.3.2. update the cache to reflect the new value of the item affected by event
- 6\. once all events have been emitted
  - 6.1. run datamodel `commit_all` action if any
  - 6.2. save cache on disk
- 7\. wait for `updateInterval` and restart from step `3.` if app hasn't been requested to stop

If any exception is raised in step `2.`, this step is restarted until it succeeds.

If any exception is raised in steps `3.` to `7.`, the cache is saved on disk, and the servers restart from step `3.`.
