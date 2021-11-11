---
description: Learn how to reset Workflow data operations from Tailer Studio.
---

# Reset Workflow data operations

## :zero: Why reset a Workflow data operation

Tailer Studio allows you to reset a Workflow data operation. The reset feature deletes all the triggered jobs so the workflow can start from scratch, as when the it was just deployed. (This feature is also available using the [Tailer API](../tailer-api/api-features.md#resetting-a-workflow).)

**Example of a case requiring a workflow reset**

We have three jobs, named JA, JB, and JC which trigger a job named JT when they are all successfully executed.

If a situation happens where JA and JB are OK, but JC is not, JT is not triggered. You fix and relaunch JC, which becomes OK, and JT is triggered. The next morning, you launch JC again to make sure it works: you get JA(0), JB(0) and JC(1). When JA and JB are automatically started a few hours later, JC is already considered as OK, which creates an unbalanced situation. A reset is necessary.

## :1234: How to proceed

To reset a Workflow data operation:

1. Log in to [Tailer Studio](http://studio.tailer.ai).
2. In the left navigation panel, in the **Data workflows** section, select **Workflow**.
3. In the right panel, access the **Status** tab.
4. Click the **Configuration id** link corresponding to the data operation of your choice.
5. In the upper right corner of the **Status** tab, click the **Reset** button.\
   The list of triggering jobs is emptied.
