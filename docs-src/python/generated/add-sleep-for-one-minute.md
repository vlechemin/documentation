---
id: add-sleep-for-one-minute
title: Add a call to sleep
sidebar_label: Add sleep call
description: Add a call to sleep for one minute to the beginning of the Workflow.
tags:
- timer
- sleep
- logger
---

<!-- DO NOT EDIT THIS FILE DIRECTLY.
THIS FILE IS GENERATED from https://github.com/temporalio/documentation/blob/main/sample-apps/python/backgroundcheck_replay/backgroundcheck_dacx.py. -->

In the following sample, we add a couple of logging statements and a Timer to the Workflow code to see how this affects the Event History.

Use the `asyncio.sleep()` API to cause the Workflow to sleep for a minute before the call to execute the Activity.
The Temporal Python SDK offers deterministic implementations to the following API calls:

- [workflow.now()](https://python.temporal.io/temporalio.workflow.html#now)
- [workflow.random()](https://python.temporal.io/temporalio.workflow.html#random)
- [workflow.time_ns()](https://python.temporal.io/temporalio.workflow.html#time_ns)
- [workflow.time()](https://python.temporal.io/temporalio.workflow.html#time)
- [workflow.uuid4()](https://python.temporal.io/temporalio.workflow.html#uuid4)

Use the `workflow.logger` API to log from Workflows to avoid seeing repeated logs from the Replay of the Workflow code.

<div class="copycode-notice-container"><div class="copycode-notice"><img data-style="copycode-icon" src="/icons/copycode.png" alt="Copy code icon" /> Sample application code information <img id="i-id402417216" data-event="clickable-copycode-info" data-style="chevron-icon" src="/icons/chevron.png" alt="Chevron icon" /></div><div id="copycode-info-id402417216" class="copycode-info">The following code sample comes from a working and tested sample application. The code sample might be abridged within the guide to highlight key aspects. Visit the source repository to <a href="https://github.com/temporalio/documentation/blob/main/sample-apps/python/backgroundcheck_replay/backgroundcheck_dacx.py">view the source code</a> in the context of the rest of the application code.</div></div>

```python
import asyncio
from datetime import timedelta

from temporalio import workflow

with workflow.unsafe.imports_passed_through():
    from ssntraceactivity import ssn_trace_activity

@workflow.defn()
class BackgroundCheck:
    @workflow.run
    async def run(self, ssn: str) -> str:
        random_number = workflow.random().randint(1, 100)
        if random_number < 50:
            await asyncio.sleep(60)
            workflow.logger.info("Sleeping for 60 seconds")
        return await workflow.execute_activity(
            ssn_trace_activity,
            ssn,
            schedule_to_close_timeout=timedelta(seconds=5),
        )
```
