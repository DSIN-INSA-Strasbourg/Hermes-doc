---
title: Workflow
weight: 1
---

hermes-client

- 1\. loads its datamodel from config file
- 2\. if it exists, loads previous datamodel from cache
- 3\. if any, notify about datamodel warnings: remote type and remote attributes present in datamodel, but not in dataschema
- 4\. if a remote schema exists in cache, load error queue from cache
- 5\. if client has not been initialized yet (no complete initSync sequence has been processed yet):
  - 5.1. process initSync sequence, if a complete initSync sequence is available on message bus
  - 5.2. restart from step `5.`
- 6\. if client has already been initialized yet (a complete initSync sequence has already been processed):
  - 6.1. if it is the first iteration of loop (step `7.` has never been reached):
    - 6.1.1. if datamodel in config differs from cached one, process the datamodel update:
      - 6.1.1.1 generate removed events for all entries of removed data types, process them, and purge those data type cache files
      - 6.1.1.2 generate a diff between cached data built upon previous datamodel, and the same data converted to new datamodel, and generate corresponding events and process them
  - 6.2. if `errorQueue_retryInterval` has passed since the last attempt, retry to process events in error queue
  - 6.3. if `trashbin_purgeInterval` has passed since the last attempt, retry to purge expired objects from trashbin
  - 6.4. loop over all events available on message bus, and process each one to call its corresponding handler when it exists in client plugin
- 7\. when at least an event was processed or if app was requested to stop:
  - 7.1. save cache files of error queue, app, data
  - 7.2. call special handler `onSave` when it exists in client plugin
  - 7.3. notify any change in error queue
- 8\. restart from step `5.` if app hasn't been requested to stop

If any exception is raised in step `6.1.1`, it will be considered as a fatal error, notified, and the client will stop.

If any exception is raised in steps `5.` to `6.`, it is notified and the client restarts from step `7.`.
