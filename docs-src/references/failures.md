---
id: failures
title: Temporal Failures
sidebar_label: Failures
description: A Failure is Temporal's representation of various types of errors that occur in the system.
tags:
  - failure
  - explanation
  - term
---

A Failure is Temporal's representation of various types of errors that occur in the system.

There are different types of Failures, and each has a different type in the SDKs and different information in the protobuf messages (which are used to communicate with the Temporal Cluster and appear in [Event History](/workflows#event-history)).

## Temporal Failure

Most SDKs have a base class that the other Failures extend:

- TypeScript: [TemporalFailure](https://typescript.temporal.io/api/classes/common.TemporalFailure)
- Java: [TemporalFailure](https://www.javadoc.io/doc/io.temporal/temporal-sdk/latest/io/temporal/failure/TemporalFailure.html)
- Python: [FailureError](https://python.temporal.io/temporalio.exceptions.FailureError.html)

The base [Failure proto message](https://api-docs.temporal.io/#temporal.api.failure.v1.Failure) has these fields:

- `string message`
- `string stack_trace`
- `string source`: The SDK this Failure originated in (for example, `"TypeScriptSDK"`). In some SDKs, this field is used to rehydrate the stack trace into an exception object.
- `Failure cause`: The `Failure` message of the cause of this Failure (if applicable).
- `Payload encoded_attributes`: Contains the encoded `message` and `stack_trace` fields when using a [Failure Converter](/concepts/what-is-a-failure-converter).

## Application Failure

Workflow and Activity code use Application Failures to communicate application-specific failures that happen.
This is the only type of Failure created and thrown by user code.

- TypeScript: [ApplicationFailure](https://typescript.temporal.io/api/classes/common.ApplicationFailure)
- Java: [ApplicationFailure](https://www.javadoc.io/doc/io.temporal/temporal-sdk/latest/io/temporal/failure/ApplicationFailure.html)
- Go: [ApplicationError](https://pkg.go.dev/go.temporal.io/sdk/temporal#ApplicationError)
- Python: [ApplicationError](https://python.temporal.io/temporalio.exceptions.ApplicationError.html)
- Proto: [ApplicationFailureInfo](https://api-docs.temporal.io/#temporal.api.failure.v1.ApplicationFailureInfo) and [Failure](https://api-docs.temporal.io/#temporal.api.failure.v1.Failure)

### Throw from Workflows

Only Workflow errors that are Temporal Failures cause the Worklow Execution to fail; all other errors cause the Workflow Task to fail and be retried (except for Go, where any error returned from the Workflow fails the Execution, and a panic fails the Task).
Most types of Temporal Failures automatically occur, like a [Cancelled Failure](#cancelled-failure) when the Workflow is Cancelled or an [Activity Failure](#activity-failure) when an Activity Fails.
You can also explicitly fail the Workflow Execution by throwing (or returning, depending on the SDK) an Application Failure.

### Throw from Activities

In Activities, you can either throw an Application Failure or another Error to fail the Activity Task.
In the latter case, the error is converted to an Application Failure.
During conversion, the following Application Failure fields are set:

- `type` is set to the error's type name.
- `message` is set to the error message.
- `non_retryable` is set to false.
- `details` are left unset.
- `cause` is a Failure converted from the error's `cause` property.
- stack trace is copied.

When an [Activity Execution](/concepts/what-is-an-activity-execution) fails, the Application Failure from the last Activity Task is the `cause` field of the [ActivityFailure](#activity-failure) thrown in the Workflow.

### Non-retryable

When an Activity or Workflow throws an Application Failure, the Failure's `type` field is matched against a Retry Policy's list of [non-retryable errors](/retry-policies#non-retryable-errors) to determine whether to retry the Activity or Workflow.
Activities and Workflow can also avoid retrying by setting an Application Failure's `non_retryable` flag to `true`.

## Cancelled Failure

When [Cancellation](/concepts/what-is-an-activity-execution#cancellation) of a Workflow or Activity is requested, SDKs represent the cancellation to the user in language-specific ways.
For example, in TypeScript, in some cases a Cancelled Failure is thrown directly by a Workflow API function, and in other cases the Cancelled Failure is wrapped in a different Failure.
To check both types of cases, TypeScript has the [isCancellation](https://typescript.temporal.io/api/namespaces/workflow#iscancellation) helper.

<!-- TODO also link to Workflow Cancellation concept -->

When a Workflow or Activity is successfully Cancelled, a Cancelled Failure is the `cause` field of the Activity Failure or "Workflow failed" error.

- TypeScript: [CancelledFailure](https://typescript.temporal.io/api/classes/common.CancelledFailure)
- Java: [CanceledFailure](https://www.javadoc.io/doc/io.temporal/temporal-sdk/latest/io/temporal/failure/CanceledFailure.html)
- Go: [CanceledError](https://pkg.go.dev/go.temporal.io/sdk/temporal#CanceledError)
- Python: [CancelledError](https://python.temporal.io/temporalio.exceptions.CancelledError.html)
- Proto: [CanceledFailureInfo](https://api-docs.temporal.io/#temporal.api.failure.v1.CanceledFailureInfo) and [Failure](https://api-docs.temporal.io/#temporal.api.failure.v1.Failure)

## Activity Failure

An Activity Failure is delivered to the Workflow Execution when an Activity fails.
It contains information about the failure and the Activity Execution; for example, the Activity Type and Activity Id.
The reason for the failure is in the `cause` field.
For example, if an Activity Execution times out, the `cause` is a [Timeout Failure](#timeout-failure).

- TypeScript: [ActivityFailure](https://typescript.temporal.io/api/classes/common.ActivityFailure)
- Java: [ActivityFailure](https://www.javadoc.io/doc/io.temporal/temporal-sdk/latest/io/temporal/failure/ActivityFailure.html)
- Go: [ActivityError](https://pkg.go.dev/go.temporal.io/sdk/temporal#ActivityError)
- Python: [ActivityError](https://python.temporal.io/temporalio.exceptions.ActivityError.html)
- Proto: [ActivityFailureInfo](https://api-docs.temporal.io/#temporal.api.failure.v1.ActivityFailureInfo) and [Failure](https://api-docs.temporal.io/#temporal.api.failure.v1.Failure)

## Child Workflow Failure

A Child Workflow Failure is delivered to the Workflow Execution when a Child Workflow Execution fails.
It contains information about the failure and the Child Workflow Execution; for example, the Workflow Type and Workflow Id.
The reason for the failure is in the `cause` field.

- TypeScript: [ChildWorkflowFailure](https://typescript.temporal.io/api/classes/common.ChildWorkflowFailure)
- Java: [ChildWorkflowFailure](https://www.javadoc.io/doc/io.temporal/temporal-sdk/latest/io/temporal/failure/ChildWorkflowFailure.html)
- Go: [ChildWorkflowExecutionError](https://pkg.go.dev/go.temporal.io/sdk/temporal#ChildWorkflowExecutionError)
- Python: [ChildWorkflowError](https://python.temporal.io/temporalio.exceptions.ChildWorkflowError.html)
- Proto: [ChildWorkflowExecutionFailureInfo](https://api-docs.temporal.io/#temporal.api.failure.v1.ChildWorkflowExecutionFailureInfo) and [Failure](https://api-docs.temporal.io/#temporal.api.failure.v1.Failure)

## Timeout Failure

A Timeout Failure represents the timeout of an Activity or Workflow.

When an Activity times out, the last Heartbeat details it emitted is attached.

- TypeScript: [TimeoutFailure](https://typescript.temporal.io/api/classes/common.TimeoutFailure)
- Java: [TimeoutFailure](https://www.javadoc.io/doc/io.temporal/temporal-sdk/latest/io/temporal/failure/TimeoutFailure.html)
- Go: [TimeoutError](https://pkg.go.dev/go.temporal.io/sdk/temporal#TimeoutError)
- Python: [TimeoutError](https://python.temporal.io/temporalio.exceptions.TimeoutError.html)
- Proto: [TimeoutFailureInfo](https://api-docs.temporal.io/#temporal.api.failure.v1.TimeoutFailureInfo) and [Failure](https://api-docs.temporal.io/#temporal.api.failure.v1.Failure)

## Terminated Failure

A Terminated Failure is used as the `cause` of an error when a Workflow is terminated, and you receive the error in one of the following locations:

- Inside a Workflow that's waiting for the result of a Child Workflow.
- When waiting for the result of a Workflow on the Client.

In the SDKs:

- TypeScript: [TerminatedFailure](https://typescript.temporal.io/api/classes/common.TerminatedFailure)
- Java: [TerminatedFailure](https://www.javadoc.io/doc/io.temporal/temporal-sdk/latest/io/temporal/failure/TerminatedFailure.html)
- Go: [TerminatedError](https://pkg.go.dev/go.temporal.io/sdk/temporal#TerminatedError)
- Python: [TerminatedError](https://python.temporal.io/temporalio.exceptions.TerminatedError.html)
- Proto: [TerminatedFailureInfo](https://api-docs.temporal.io/#temporal.api.failure.v1.TerminatedFailureInfo) and [Failure](https://api-docs.temporal.io/#temporal.api.failure.v1.Failure)

## Server Failure

A Server Failure is used for errors that originate in the Cluster.

- TypeScript: [ServerFailure](https://typescript.temporal.io/api/classes/common.ServerFailure)
- Java: [ServerFailure](https://www.javadoc.io/doc/io.temporal/temporal-sdk/latest/io/temporal/failure/ServerFailure.html)
- Go: [ServerError](https://pkg.go.dev/go.temporal.io/sdk/temporal#ServerError)
- Python: [ServerError](https://python.temporal.io/temporalio.exceptions.ServerError.html)
- Proto: [ServerFailureInfo](https://api-docs.temporal.io/#temporal.api.failure.v1.ServerFailureInfo) and [Failure](https://api-docs.temporal.io/#temporal.api.failure.v1.Failure)
