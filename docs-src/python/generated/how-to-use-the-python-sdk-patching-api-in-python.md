---
id: how-to-use-the-python-sdk-patching-api-in-python
title: How to use the Python SDK Patching API
sidebar_label: Python SDK Patching API
description: Heres a sample implementation of patching in new code using the Python SDK's patching API.
tags:
- version
- python sdk
- code sample
---

<!-- DO NOT EDIT THIS FILE DIRECTLY.
THIS FILE IS GENERATED from https://github.com/temporalio/documentation/blob/main/sample-apps/python/version_your_workflows/workflow_1_initial_dacx.py. -->

In principle, the Python SDK's patching mechanism operates similarly to other SDKs in a "feature-flag" fashion. However, the "versioning" API now uses the concept of "patching in" code.

To understand this, you can break it down into three steps, which reflect three stages of migration:

- Running `pre_patch_activity` code while concurrently patching in `post_patch_activity`.
- Running `post_patch_activity` code with deprecation markers for `my-patch` patches.
- Running only the `post_patch_activity` code.

Let's walk through this process in sequence.

Suppose you have an initial Workflow version called `pre_patch_activity`:

<div class="copycode-notice-container"><div class="copycode-notice"><img data-style="copycode-icon" src="/icons/copycode.png" alt="Copy code icon" /> Sample application code information <img id="i-id1075455897" data-event="clickable-copycode-info" data-style="chevron-icon" src="/icons/chevron.png" alt="Chevron icon" /></div><div id="copycode-info-id1075455897" class="copycode-info">The following code sample comes from a working and tested sample application. The code sample might be abridged within the guide to highlight key aspects. Visit the source repository to <a href="https://github.com/temporalio/documentation/blob/main/sample-apps/python/version_your_workflows/workflow_1_initial_dacx.py">view the source code</a> in the context of the rest of the application code.</div></div>

```python
from datetime import timedelta

from temporalio import workflow

with workflow.unsafe.imports_passed_through():
    from activities import pre_patch_activity
# ...
@workflow.defn
class MyWorkflow:
    @workflow.run
    async def run(self) -> None:
        self._result = await workflow.execute_activity(
            pre_patch_activity,
            schedule_to_close_timeout=timedelta(minutes=5),
        )
```
