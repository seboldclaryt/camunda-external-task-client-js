# `Client`

```js
const { Client } = require("camunda-external-task-handler-js");

const client = new Client({ baseUrl: "http://localhost:8080/engine-rest" });

client.subscribe("foo", async function({ task, taskService}) {
  // Put your business logic
});
```

Client is the core class of the external task client.
It is mainly used to start/stop the external task client and subscribe to a certain topic.

## `new Client(options)`

Options are **mandatory** when creating a _Client_ instance.

Here"s a list of the available options:

| Option | Description | Type | Required | Default |
|:--------------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|------------------------|:--------:|:----------------:|
| baseUrl | Path to the engine api | string | ✓ |  |
| workerId | The id of the worker on which behalf tasks are fetched. The returned tasks are locked for that worker and can only be completed when providing the same worker id. | string |  | "some-random-id" |
| maxTasks | The maximum number of tasks to fetch | number |  | 10 |
| maxParallelExecutions | The maximum number of tasks to be worked on simultaneously | number |  |  |
| interval | Interval of time to wait before making a new poll. | number |  | 300 |
| lockDuration | The default duration to lock the external tasks for in milliseconds. | number |  | 50000 |
| autoPoll | If true, then polling start automatically as soon as a Client instance is created. | boolean |  | true |
| asyncResponseTimeout | The Long Polling timeout in milliseconds. | number |  |  |
| interceptors | Function(s) that will be called before a request is sent. Interceptors receive the configuration of the request and return a new configuration. | function or [function] |  |  |
| use | Function(s) that have access to the client instance as soon as it is created and before any polling happens.  Check out [logger](/lib/logger.js) for a better understanding of the usage of middlewares. | function or [function] |  |  |

### About interceptors

* Interceptors receive the configuration of the request and return a new configuration.
* In the case of multiple interceptors, they are piped in the order they are provided.
* Check out [BasicAuthInterceptor](/lib/BasicAuthInterceptor.js) for a better understanding of the usage of interceptors.

## `client.start()`

Triggers polling.

### About Polling

* Polling tasks from the engine works by performing a fetch & lock operation of tasks that have subscriptions. It then calls the handler registered to each task.
* Polling is done periodically based on the _interval_ configuration.
* [Long Polling](https://docs.camunda.org/manual/latest/user-guide/process-engine/external-tasks/#long-polling-to-fetch-and-lock-external-tasks) is enabled by configuring the option _asyncResponseTimeout_.

## `client.subscribe(topic, options, handler)`

Subscribes a handler to a specific topic and returns a _topic subscription_.
Here"s a list of the available parameters:

| Parameter | Description                                           | Type     | Required |
| --------- | ----------------------------------------------------- | -------- | -------- |
| topic     | topic name for which external tasks should be fetched | string   | ✓        |
| options   | options about subscription                            | object   |          |
| handler   | function to handle fetched task. Checkout [this](/docs/handler.md) for more information               | function | ✓        |

The currently supported options are:

| Option       | Description                                                                         | Type   | Required | Default                                               |
|--------------|-------------------------------------------------------------------------------------|--------|----------|-------------------------------------------------------|
| lockDuration | specifies the lock duration for this specific handler.                              | number |          | global lockDuration configured in the client instance |
| variables    | defines a subset of variables available in the handler.                             | array  |          | global lockDuration configured in the client instance |
| businessKey  | A value which allows to filter tasks based on process instance business key         | string |          |                                                       |
| processDefinitionId  | A value which allows to filter tasks based on process definition id         | string |          |                                                       |
| processDefinitionIdIn  | A value which allows to filter tasks based on process definition ids         | string |          |                                                       |
| processDefinitionKey  | A value which allows to filter tasks based on process definition key         | string |          |                                                       |
| processDefinitionKeyIn  | A value which allows to filter tasks based on process definition keys         | string |          |                                                       |
| processDefinitionVersionTag  | A value which allows to filter tasks based on process definition Version Tag         | string |          |                                                       |
| withoutTenantId  | A value which allows to filter tasks based on tenant ids         | string |          |                                                       |
| tenantIdIn   | A value which allows to filter tasks without tenant id                              | boolean |         |                                                       |


### About topic subscriptions

A topic subscription, which is returned by the **subscribe()** method, is a an object that provides the following:

* **handler:** a function that is executed whenever a task is fetched & locked for the topic subscribed to.
* **unsubscribe():** a function to unsubscribe from a topic.
* **lockDuration:** the configured lockDuration for this specific topic subscription.
* **variables:** the selected subset of variables.

```js
const { Client } = require("camunda-external-task-client-js");

const client = new Client({ baseUrl: "http://localhost:8080/engine-rest" });

const topicSubscription = client.subscribe("foo", async function({ task, taskService}) {
  // Put your business logic
});

// unsubscribe from a topic
topicSubscription.unsubscribe();
```

## `client.stop()`

Stops polling.

## Client Events

> Check out [logger](/lib/logger.js) for a better understanding of the usage of events.
> Here"s a list of available client events:

* `client.on("subscribe", function(topic, topicSubscription) {})`
* `client.on("unsubscribe", function(topic, topicSubscription) {})`
* `client.on("poll:start", function() {})`
* `client.on("poll:stop", function() {})`
* `client.on("poll:success", function(tasks) {})`
* `client.on("poll:error", function(error) {})`
* `client.on("complete:success", function(task) {})`
* `client.on("complete:error", function(task, error) {})`
* `client.on("handleFailure:success", function(task) {})`
* `client.on("handleFailure:error", function(task, error) {})`
* `client.on("handleBpmnError:success", function(task) {})`
* `client.on("handleBpmnError:error", function(task, error) {})`
* `client.on("extendLock:success", function(task) {})`
* `client.on("extendLock:error", function(task, error) {})`
* `client.on("unlock:success", function(task) {})`
* `client.on("unlock:error", function(task, error) {})`
