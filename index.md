<style>
img
{
    display:block;
    float:none;
    margin-left:auto;
    margin-right:auto;
    width:40%;
}

iframe
{
    width:100%;
    height:100%;
    background: whitesmoke;
}
</style>
<link href="prism.css" rel="stylesheet" />
<script src="prism.js"></script>

## Table of Contents
- [Table of Contents](#table-of-contents)
- [Patterns](#patterns)
  - [**Process Manager**](#process-manager)
  - [**Event and Document Message**](#event-and-document-message)
  - [**Message Endpoint**](#message-endpoint)
  - [**Pipes and Filters**](#pipes-and-filters)
  - [**Multicast**](#multicast)
  - [**Content-based Router**](#content-based-router)
  - [**Loop**](#loop)
  - [**Delay**](#delay)
  - [**Gateway**](#gateway)
  - [**Content Filter**](#content-filter)
  - [**Content Enricher**](#content-enricher)
  - [**Claim Check**](#claim-check)
  - [**Normalizer**](#normalizer)
  - [**Message History**](#message-history)
  - [**Splitter and Aggregator**](#splitter-and-aggregator)
  - [**Implicit termination**](#implicit-termination)
  - [**Nested Workflows**](#nested-workflows)
  - [**Callback**](#callback)
  - [**Error Handling**](#error-handling)
  - [**Workflow Data**](#workflow-data)
- [References](#references)

## Patterns

### **Process Manager**

![Design Decision - Process Manager](images/Design_decisions_process_manager.png)

**Problem**: How does the serverless workflow determine the path in which the message needs to flow if it consists of multiple functions and conditions?

**Decision**: The _Process Manager_ acts as a central processing component for the system. As workflows are influenced by each step's output message, execution states need to be maintained, and based on the result; the succeeding component is invoked.

**Source**: [[Hohpe and Woolf 2004]](#1)

**Pattern**: Enterprise Integration Pattern

**Type**: Construct

**Synonyms**: -

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
<!-- <iframe src="aws_step_functions\process_manager.html"></iframe> -->
States can be orchestrated using ASF State Machine.
<br/>
<div>
    <img src="./images/aws_mapping_process_manager.png" alt="Process Manager" style="
    height: 300px;
    width: 200px;
">
</div>
<br/>
<b>ASF snippet</b>:
<br/>
<pre>
  <code>
    {
    "Comment": "ASF Template",
    "StartAt": "Function",
    "States": {
        "Function": {
        "Type": "Pass",
        "End": true
        }
    }
    }
  </code>
</pre>
</details>

<details>
<summary><b>Zeebe</b></summary>
The "Process Manager" pattern for Zeebe is the broker coordinating the various tasks in the workflow. Here the various tasks are associated with their corresponding hosted function.
<br/>
<div>
    <img src="./images/zeebe_mapping_process_manager.png" alt="Process Manager">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
Here the message routing "Process Manager" pattern for ADF is presented. Here the various functions are orchestrated using the primary Orchestration Function.
<br/>
<div>
    <img src="./images/adf_mapping_process_manager.png" alt="Process Manager">
</div>
</details>

<br />

----

<br />

### **Event and Document Message**

![Design Decision - Event and Document Message](images/Design_decisions_event_document_message.png)

**Problem**: How can the serverless workflow and its involved functions be executed/triggered?

**Decision**: External services or clients can invoke the serverless data processing workflow by an _Event Message_. Furthermore, _Event Messages_ can be used to invoke other workflows or services. As functions are considered a black box, the _Document Message_ containing the data structure message is the most optimum choice when communicating between internal states/functions.

**Source**: [[Hohpe and Woolf 2004]](#1)

**Pattern**: Enterprise Integration Pattern

**Type**: Construct

**Synonyms**: -

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
In ADF, the Event message construct invokes the orchestration function, and the Document message handles the internal message communication between the functions, as depicted by the figure.
<br/>
<div>
    <img src="./images/adf_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<br />

----

<br />

### **Message Endpoint**

![Design Decision - Message Endpoint](images/Design_decisions_message_endpoint.png)

**Problem**: How are various functions in a serverless workflow connected?

**Decision**: With the _Message Endpoint_ construct, the various functions do not need to be aware of the message formats, channel, or other functions present in the serverless workflow. The functions only need to be mindful that they will receive requests, and it just needs to process and send the acknowledgment/response back to the system

**Source**: [[Hohpe and Woolf 2004]](#1)

**Pattern**: Enterprise Integration Pattern

**Type**: Construct

**Synonyms**: -

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
In ASF, Message Endpoint are linked to states with the "Task" type. The Task state has the following required fields:<br/>
- Resource: ARN that uniquely identifies the specific AWS Lambda to execute.
<br/>
<pre>
  <code class="language-json">
    "State": {
    "Type": "Task",
    "Resource": "arn:aws:states:::lambda:invoke",
    "Parameters": {
        "FunctionName": "arn:aws:lambda:REGION:ACCOUNT_ID:function:FUNCTION_NAME",
        "Payload": {
        "Input.$": "$"
        }
    },
    "Next": "NEXT_STATE"
    }
  </code>
</pre>
</details>

<details>
<summary><b>Zeebe</b></summary>
The Message Endpoint construct, which accepts the messages and processes the message, is mapped to the "Service Task" with Type = "lambda" (Based on the FaaS vendor provider). The below figure illustrates how this construct can be used in the BPMN 2.0 Zeebe modeler.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
<br/>
<pre>
  <code class="language-xml">
    &lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot;?&gt;
    &lt;bpmn:definitions xmlns:bpmn=&quot;http://www.omg.org/spec/BPMN/20100524/MODEL&quot; xmlns:bpmndi=&quot;http://www.omg.org/spec/BPMN/20100524/DI&quot; xmlns:dc=&quot;http://www.omg.org/spec/DD/20100524/DC&quot; xmlns:zeebe=&quot;http://camunda.org/schema/zeebe/1.0&quot; id=&quot;Definitions_0dmi4p0&quot; targetNamespace=&quot;http://bpmn.io/schema/bpmn&quot; exporter=&quot;Zeebe Modeler&quot; exporterVersion=&quot;0.11.0&quot;&gt;
    &lt;bpmn:process id=&quot;Zeebe_Process&quot; name=&quot;Zeebe Model&quot; isExecutable=&quot;true&quot;&gt;
        &lt;bpmn:serviceTask id=&quot;ServiceTask_Lambda&quot; name=&quot;Service Task&quot;&gt;
        &lt;bpmn:extensionElements&gt;
            &lt;zeebe:taskDefinition type=&quot;lambda&quot; /&gt;
        &lt;/bpmn:extensionElements&gt;
        &lt;/bpmn:serviceTask&gt;
    &lt;/bpmn:process&gt;
    &lt;bpmndi:BPMNDiagram id=&quot;BPMNDiagram_1&quot;&gt;
        &lt;bpmndi:BPMNPlane id=&quot;BPMNPlane_1&quot; bpmnElement=&quot;Zeebe_Process&quot;&gt;
        &lt;bpmndi:BPMNShape id=&quot;Activity_079frpn_di&quot; bpmnElement=&quot;ServiceTask_Lambda&quot;&gt;
            &lt;dc:Bounds x=&quot;160&quot; y=&quot;80&quot; width=&quot;100&quot; height=&quot;80&quot; /&gt;
        &lt;/bpmndi:BPMNShape&gt;
        &lt;/bpmndi:BPMNPlane&gt;
    &lt;/bpmndi:BPMNDiagram&gt;
    &lt;/bpmn:definitions&gt;
  </code>
</pre>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
The Message Endpoint construct, which receives the messages and processes the message, is realized by the Activity Function. The functions must be idempotent as it follows the at-least-once execution strategy. The below code snippet illustrates how this construct can be used in ADF.
<br/>
<pre>
  <code class="language-javascript">
    const function = yield context.df.callActivity("Activity Function", "Payload")
  </code>
</pre>
</details>

<br />

----

<br />

### **Pipes and Filters**

![Design Decision - Pipes and Filters](images/Design_decisions_pipes_and_filters.png)

**Problem**: How to decompose a task that performs complex processing into a series of separate elements that can be reused?

**Decision**: _Pipes and Filters_ help in implementing complex processing in a granular, independent, resilient and sequential manner. Moreover, the fundamental building blocks of serverless workflows are functions, and each function in the pipeline is generally responsible for small transactions making this pattern style optimum.

**Source**: [[Hohpe and Woolf 2004]](#1)

**Pattern**: Enterprise Integration Pattern

**Type**: Construct

**Synonyms**: Sequence, Sequential routing, Serial Routing

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
"Pipes and Filters" play an essential aspect in standardizing a workflow execution and this is referred to as Function Chaining pattern<sup><a href="#20" id="20">20</a></sup> in ADF. The below code snippet shows how each Activity Function (filter) performs only one distinct operation and the pipes that are the JSON message that coordinate the various functions.
<br/>
<pre>
  <code class="language-javascript">
    import * as df from "durable-functions"

    module.exports = df.orchestrator(function* (context) {
        try {
            const function1Result = yield context.df.callActivity("function1", context.df.getInput())
            const function2Result = yield context.df.callActivity("function2", function1Result)
            const function3Result = yield context.df.callActivity("function3", function2Result)
            return function3Result;
        }
        catch (error) {
            console.error(error)
        }
    });
  </code>
</pre>
</details>

<br />

----

<br />

### **Multicast**

![Design Decision - Multicast](images/Design_decisions_multicast.png)

**Problem**: How will the serverless workflow route the same message to several endpoints and process them differently?

**Decision**: A _Multicast_ pattern is used to model the execution of parallel flows/concurrency by sending a copy of the same message to multiple recipients without checking any conditions. Here all outgoing flows are executed at the same time.

**Source**: [[Ibsen and Anstey 2010]](#2)

**Pattern**: Enterprise Integration Pattern

**Type**: Control Flow

**Synonyms**: Parallel Split, AND-Split, Parallel Routing, Fork

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
The "Multicast" pattern is implemented in ADF by following the below code snippet. In this implementation, the same data is sent to multiple Activity Functions and executed simultaneously.
<br/>
<pre>
  <code class="language-javascript">
    const df = require("durable-functions");

    module.exports = df.orchestrator(function* (context) {
        const parallelTasks = [];

        // Get input
        const data = context.df.getInput()

        // Perform parallel processing
        parallelTasks.push(context.df.callActivity("function1", data));
        parallelTasks.push(context.df.callActivity("function2", data));

        const arrayParallelTasksResult = yield context.df.Task.all(parallelTasks);

        return arrayParallelTasksResult
    });
  </code>
</pre>
</details>

<br />

----

<br />

### **Content-based Router**

![Design Decision - Content-based Router](images/Design_decisions_content_based_router.png)

**Problem**: Functions must be orchestrated to adhere to a process flow to generate an error-free/desired output. How can the messages be routed to the correct workflow execution path within the workflow based on the message content?

**Decision**:  A _Content-based Router_ helps in controlling the workflow based on the message content. Each outgoing flow connected from the router corresponds to a condition, and the flow with the satisfied condition is traversed. Based on the condition, one or many flows can be traversed. In this pattern, the router examines the message content using numerous criteria like fields, values, and conditions before routing to the appropriate path.

**Source**: [[Hohpe and Woolf 2004]](#1)

**Pattern**: Enterprise Integration Pattern

**Type**: Control Flow

**Synonyms**: Exclusive Choice, XOR-Split, Conditional Routing, Switch, Decision, Selection and OR-Split

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
With the below code snippet, a Content-based Router is realized in ADF by using conditionals to control the orchestration flow.
<br/>
<pre>
  <code class="language-javascript">
    const df = require("durable-functions");

    module.exports = df.orchestrator(function* (context) {
        var result
        // Get input
        const data = context.df.getInput()

        // Perform parallel processing
        if (data.isFunction1) {
            result = yield context.df.callActivity("function1", data)
        } else {
            result = yield context.df.callActivity("function2", data)
        }

        return result
    });
  </code>
</pre>
</details>

<br />

----

<br />

### **Loop**

![Design Decision - Loop](images/Design_decisions_loop.png)

**Problem**: In a serverless workflow, certain functions have to be executed multiple times to produce the desired outcome. How can the workflow orchestrate a function to be reused when it needs to be triggered recursively?

**Decision**:  The _Loop_ pattern is used to loop through the function multiple times

**Source**: [[Ibsen and Anstey 2010]](#2)

**Pattern**: Enterprise Integration Pattern

**Type**: Control Flow

**Synonyms**: Arbitrary Cycles, Iteration, Cycle

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
In ADF, looping of the functions can be implemented using entry/exit controlled loops. The below code snippet shows how Loop pattern is implemented using a <i>While</i> loop. 
<br/>
<pre>
  <code class="language-javascript">
    const df = require("durable-functions");

    module.exports = df.orchestrator(function* (context) {
        var result
        // Get input
        const data = context.df.getInput()

        // Loop till condition is false
        while (data.loopCondition) {
            result = yield context.df.callActivity("function1", data)
        }

        return result
    });
  </code>
</pre>
</details>

<br />

----

<br />

### **Delay**

![Design Decision - Delay](images/Design_decisions_event_delay.png)

**Problem**: There are situations during a workflow execution when it needs to be paused or delayed to wait for a response/acknowledgment from an external system. How can the workflow incorporate a delay or wait?

**Decision**:  The _Delay_ pattern helps in waiting or delaying a function from executing. The delay/wait can be configured by setting a time/period.

**Source**: [[Ibsen and Anstey 2010]](#2)

**Pattern**: Enterprise Integration Pattern

**Type**: Control Flow

**Synonyms**: -

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
ADF provides durable timers<sup><a href="#3" id="3">3</a></sup> for orchestrator functions to implement delays or set up timeouts on async actions. The below code snippet depicts how the "Delay" pattern is used in ADF.
<br/>
<pre>
  <code class="language-javascript">
    const df = require("durable-functions");
    const moment = require("moment");

    module.exports = df.orchestrator(function* (context) {

        const function1Result = yield context.df.callActivity("function1", context.df.getInput())

        // Perform delay operation
        const delay = moment.utc(context.df.currentUtcDateTime).add(30, "s");
        yield context.df.createTimer(delay.toDate())

        const function2Result = yield context.df.callActivity("function2", function1Result)

        return function2Result
    });
  </code>
</pre>
</details>

<br />

----

<br />

### **Gateway**

![Design Decision - Gateway](images/Design_decisions_gateway.png)

**Problem**: Business and operational/implementation logic must be as decoupled as possible to allow core business logic to remain simple?

**Decision**:  The ingestion and output logic need to be encapsulated in separate functions with the help of _Message Gateway_ pattern and this pattern also helps in dividing messaging-specific implementation from the business logic code.

**Source**: [[Hohpe and Woolf 2004]](#1)

**Pattern**: Enterprise Integration Pattern

**Type**: Function Specific

**Synonyms**: -

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
 The Gateway pattern is function-specific, and the pattern mapping is equivalent to Section~\ref{subsubsection:aws_gateway}.
</details>

<br />

----

<br />

### **Content Filter**

![Design Decision - Content Filter](images/Design_decisions_content_filter.png)

**Problem**: How can the workflow simplify dealing with large messages and transmit only the essential data to the required functions?

**Decision**:  The _Content Filter_ pattern simplifies the structure of the messages by removing irrelevant data.

**Source**: [[Hohpe and Woolf 2004]](#1)

**Pattern**: Enterprise Integration Pattern

**Type**: Function Specific

**Synonyms**: -

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
The Content Filter pattern is function-specific, and the pattern mapping is equivalent to code snippet presented in Section~\ref{subsubsection:aws_content_filter}. This pattern can also be realized in the Orchestration function using the below code snippet by filtering data before sending it as a payload to the subsequent function call.
<br/>
<pre>
  <code class="language-javascript">
    const df = require("durable-functions");
    const utils = require("../utility/utils.js");

    module.exports = df.orchestrator(function* (context) {
        var function1Result = yield context.df.callActivity("function1", context.df.getInput())
        // Start : Filter result
        function1Result = utils.removeField(function1Result,'parameter1')
        // End : Filter result
        const function2Result = yield context.df.callActivity("function2", function1Result)
        return function2Result
    });
  </code>
</pre>
</details>

<br />

----

<br />

### **Content Enricher**

![Design Decision - Content Enricher](images/Design_decisions_content_enricher.png)

**Problem**: How can the workflow fetch additional data required by the functions to process the message?

**Decision**:  The _Content Enricher_ pattern accesses external data source and augments the original message with the missing information.

**Source**: [[Hohpe and Woolf 2004]](#1)

**Pattern**: Enterprise Integration Pattern

**Type**: Function Specific

**Synonyms**: -

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
 The Content Enricher pattern is function-specific, and the pattern mapping is equivalent to Section~\ref{subsubsection:aws_content_filter}. The pattern can also be performed in the orchestration function by following the below template. The below code snippet shows how a function's result enriches another function and then is used as a payload to another activity function.
<br/>
<pre>
  <code class="language-javascript">
    const df = require("durable-functions");
    const utils = require("../utility/utils.js");

    module.exports = df.orchestrator(function* (context) {
        var function1Result = yield context.df.callActivity("function1", context.df.getInput())
        var function2Result = yield context.df.callActivity("function2", context.df.getInput())
        // Start : Enrich result
        function1Result = utils.addNewField(function1Result, function2Result)
        // End : Enrich result
        const function3Result = yield context.df.callActivity("function3", function1Result)
        return function3Result
    });
  </code>
</pre>
</details>

<br />

----

<br />

### **Claim Check**

![Design Decision - Claim Check](images/Design_decisions_claim_check.png)

**Problem**: Functions that pass large payloads of data within the workflow can be terminated due to size limitations. How will the communication between functions be handled when large messages need to be passed within the workflow?

**Decision**:  Large fields are temporarily filtered in the source function and enriched in the destination function using the _Claim Check_ pattern. The payload is stored in a persistent store, and a _Claim Check_ is passed to the target component. Internally, _Claim Check_ uses the _Content Filter_ and _Content Enricher_ pattern. The _Content Filter_ pattern removes insignificant data from an output message leaving only essential information, thus simplifying its structure. The target function then uses the _Content Enricher_ pattern to augment the received message with the missing information, usually with the help of an external data source.

**Source**: [[Hohpe and Woolf 2004]](#1)

**Pattern**: Enterprise Integration Pattern

**Type**: Function Specific

**Synonyms**: -

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
 The Claim Check pattern internally uses the Content Filter and Enricher pattern. Hence, the mapping is similar to AWS Step Functions.
</details>

<br />

----

<br />

### **Normalizer**

![Design Decision - Normalizer](images/Design_decisions_normalizer.png)

**Problem**: How can the output from each terminal function in the workflow branches be normalized, which otherwise would require having an additional normalization function?

**Decision**:  The _Normalizer_ pattern helps solve this problem by ensuring that the messages produced from any branch confirm with a standard format that is understandable by the recipient component. In this pattern, each message is passed through a custom message translator so that the resulting messages match a standard format. Hence this pattern helps in preventing the creation and invoking of additional functions to handle this scenario.

**Source**: [[Hohpe and Woolf 2004]](#1)

**Pattern**: Enterprise Integration Pattern

**Type**: Function Specific

**Synonyms**: -

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>

</details>

<br />

----

<br />

### **Message History**

![Design Decision - Message History](images/Design_decisions_message_history.png)

**Problem**: How can we effectively analyze and debug the flow of messages in a loosely coupled and granular system?

**Decision**:  The primary purpose of employing a serverless paradigm is to build loosely coupled and granular systems. However, building such systems induces the complexity of debugging and traceability as it is not intuitively possible to comprehend the flow of the message. This problem can be solved using the _Message History_ pattern, in which the system maintains the history of the message. Thus when a message fails to be processed in the system, the developer can trace back the steps and provide instant feedback and solution.

**Source**: [[Hohpe and Woolf 2004]](#1)

**Pattern**: Enterprise Integration Pattern

**Type**: Function Specific

**Synonyms**: -

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
The Message History of the Azure Durable Orchestration function is maintained using the execution history table<sup><a href="#4" id="4">4</a></sup> as shown in the below figure. When <i>yield</i> is invoked the Activity function result is stored in the History Table. In a Azure Durable Orchestration function execution, the Activity Functions have an at-least-once policy making the History Table crucial to check if the function has been executed or not. The History Table provides the input and result for each function.
<br/>
<div>
    <img src="./images/adf_mapping_message_history.png" alt="Message History">
</div>
</details>

<br />

----

<br />

### **Splitter and Aggregator**

![Design Decision - Splitter and Aggregator](images/Design_decisions_splitter_and_aggregator.png)

**Problem**: How can the serverless workflow process multiple homogeneous records concurrently that are part of a single payload?

**Decision**:  A _Splitter_ pattern helps split a single message into a sequence of sub-messages that can be processed individually. Likewise, the _Aggregator_ pattern performs the contrary by collecting a complete set of related messages. Combining the two patterns simulates the MapReduce<sup><a href="#2" id="2">2</a></sup> implementation, which can be used to split the array payload into smaller chunks that be processed in a parallel fashion and, more importantly, avoid payload limit issues.

**Source**: [[Hohpe and Woolf 2004]](#1)

**Pattern**: Enterprise Integration Pattern

**Type**: Function Specific

**Synonyms**: Fan-out, Fan-in

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
The presented code snippet shows how the Splitter and Aggregator pattern is implemented using ADF and is also referred to as Fan-out/Fan-in pattern<sup><a href="#5" id="5">5</a></sup>. Similar to Multicast, in this pattern, the data is processed using a parallel construct. In ADF, the Splitter and Aggregator pattern is implemented by first splitting the data into batches, and then each batch is processed using the same function parallelly. The result of each branch is aggregated using another Activity Function.
<br/>
<pre>
  <code class="language-javascript">
    const df = require("durable-functions");

    module.exports = df.orchestrator(function* (context) {
        const mapTasks = [];

        // Get a list of batches to process in parallel
        const batch = yield context.df.callActivity("function1");

        // Perform parallel processing of the batches (Map)
        for (let i = 0; i < batch.length; i++) {
            mapTasks.push(context.df.callActivity("function2", batch[i]));
        }
        const arrayParallelTasksResult = yield context.df.Task.all(mapTasks);

        // Aggregate the results (Reduce)
        yield context.df.callActivity("function3", arrayParallelTasksResult);
    });
  </code>
</pre>
</details>

<br />

----

<br />

### **Implicit termination**

![Design Decision - Implicit termination](images/Design_decisions_implicit_termination.png)

**Problem**: How to terminate the workflow when no execution steps are remaining?

**Decision**:  The _Implicit Termination_ pattern states that if there is no task to be performed, stop the workflow

**Source**: [[Russell et al. 2006a]](#3), [[van der Aalst et al. 2003]](#4)

**Pattern**: Workflow Control-Flow Pattern

**Type**: Control Flow

**Synonyms**: -

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
Implicit termination pattern in ADF occurs when the Orchestration function reaches the last execution statement or when a "return" statement is reached. The below code snippet depicts when the return statement is reached, the ADF orchestration function is terminated.
<br/>
<pre>
  <code class="language-javascript">
    const df = require("durable-functions");

    module.exports = df.orchestrator(function* (context) {
        const function1Result = yield context.df.callActivity("function1", context.df.getInput())
        return function1Result
    });
  </code>
</pre>
</details>

<br />

----

<br />

### **Nested Workflows**

![Design Decision - Nested Workflows](images/Design_decisions_nested_workflows.png)

**Problem**: If some tasks are alike, how do we abstract and represent them as a hierarchical and reusable model?

**Decision**:  _Nested Workflows_ patterns help facilitate reusable workflows, abstracting complex logic, effective communication, and hierarchical and modular modeling.

**Source**: [[Russell et al. 2006a]](#3), [[van der Aalst et al. 2003]](#4)

**Pattern**: Workflow Control-Flow Pattern

**Type**: Control Flow

**Synonyms**: Sub-workflow

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
In ADF, a nested workflow pattern is constructed when another durable orchestration function is invoked from the parent orchestration function. The below code snippet shows how the sub orchestration function can be triggered using the <i>callSubOrchestrator</i><sup><a href="#6" id="6">6</a></sup> function call.
<br/>
<pre>
  <code class="language-javascript">
    const df = require("durable-functions");

    module.exports = df.orchestrator(function* (context) {
        const result = context.df.callSubOrchestrator("subOrchestration", context.df.getInput())
        return result
    });
  </code>
</pre>
</details>

<br />

----

<br />

### **Callback**

![Design Decision - Callback](images/Design_decisions_callback.png)

**Problem**: How can the serverless workflow handle external invocations from a service or a human-performed activity?

**Decision**:  In the _Callback_ pattern, the workflow pauses execution and waits until an appropriate response is received to proceed with the execution. These tasks can be human, service, or some response from an external process.

**Source**: [[Russell et al. 2006a]](#3), [[van der Aalst et al. 2003]](#4)

**Pattern**: Workflow Control-Flow Pattern

**Type**: Control Flow / Function Specific

**Synonyms**: -

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
The callback pattern can be implemented in ADF using <i>waitForExternalEvent</i><sup><a href="#7" id="7">7</a></sup>, allowing an orchestrator to wait and listen for an external event asynchronously. The below code snippet presents how the pattern is implemented using ADF.
<br/>
<pre>
  <code class="language-javascript">
    const df = require("durable-functions");

    module.exports = df.orchestrator(function* (context) {
        const token = yield context.df.waitForExternalEvent("externalFunction");
        if (token) {
            // token received from external and continue processing
        } else {
            // token failed
        }
    });
  </code>
</pre>
</details>

<br />

----

<br />

### **Error Handling**

![Design Decision - Error Handling](images/Design_decisions_error_handling.png)

**Problem**: How can the system handle error exceptions that might occur in the workflow and manage them gracefully?

**Decision**:  The _Error Handling_ pattern helps handle exceptions due to abnormal input or conditions and can retry the processing when needed.

**Source**: [[Russell et al. 2006a]](#3)

**Pattern**: Workflow Control-Flow Pattern

**Type**: Control Flow / Function Specific

**Synonyms**: -

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
Error handling<sup><a href="#8" id="8">8</a></sup> in ADF is implemented using the programming language's built-in error-handling features (try-catch), as shown in the code snippet. Exceptions thrown in an Activity Function are directed back to the orchestrator function and thrown as a <i>FunctionFailedException</i>.
<br/>
<pre>
  <code class="language-javascript">
    const df = require("durable-functions");

    module.exports = df.orchestrator(function* (context) {
        try {
            const function1 = yield context.df.callActivity("function1", context.df.getInput())
            return function1;
        }
        catch (error) {
            console.error(error)
        }
    });
  </code>
</pre>
</details>

<br />

----

<br />

### **Workflow Data**

![Design Decision - Workflow Data](images/Design_decisions_workflow_data.png)

**Problem**:

- Sharing external and internal dependencies so that code duplication can be kept to the bare minimum and prevent maintainability issues.
- Reduce the size of your deployment package.
- Ensure the usage of common versions of dependencies/data between various components.

**Decision**:  The _Workflow Data_ pattern states that the data required for the whole workflow will be available to all functions. In this pattern, the shared libraries and packages are placed under the appropriate directory or vendor-specific offerings.

**Source**: [[Russell et al. 2005]](#5)

**Pattern**: Workflow Data Pattern

**Type**: Function Specific

**Synonyms**: -

**Mapping**:

<details>
<summary><b>AWS Step Functions</b></summary>
ASF can be triggered using an event message via the API Gateway<sup><a href="#1" id="1">1</a></sup>. The various states in ASF are traversed using a document message that is a JSON structured message.
<br/>
<div>
    <img src="./images/aws_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Zeebe</b></summary>
In Zeebe, the Event and Document message constructs invoke the workflow and handle the internal communication between elements, respectively. A client can invoke the intermediatory Zeebe client, which in turn invokes the BPMN 2.0 Zeebe workflow via gRPC. Internally, the workflow uses variables and JSON messages to interact with the states.
<br/>
<div>
    <img src="./images/zeebe_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
The Workflow Data pattern is function-specific, and sharing utilities, libraries, and helper code can be done by placing all these compiled files in a folder at the root level of the functions.
<br/>
<div>
    <img src="./images/azure_mapping_workflow_data.png" alt="Workflow Data">
</div>
</details>

<br />

----

<br />

## References

<a id="1">[Hohpe and Woolf 2004]</a>
Hohpe, G. and Woolf, B., 2004. Enterprise integration patterns: Designing, building, and deploying messaging solutions. Addison-Wesley Professional.

<a id="2">[Ibsen and Anstey 2010]</a>
Ibsen, C. and Anstey, J., 2018. Camel in action. Simon and Schuster.

<a id="3">[Russell et al. 2006a]</a>
Russell, N., Ter Hofstede, A.H., Van Der Aalst, W.M. and Mulyar, N., 2006. Workflow control-flow patterns: A revised view. BPM Center Report BPM-06-22, BPMcenter. org, pp.06-22.

<a id="4">[van der Aalst et al. 2003]</a>
van Der Aalst, W.M., Ter Hofstede, A.H., Kiepuszewski, B. and Barros, A.P., 2003. Workflow patterns. Distributed and parallel databases, 14(1), pp.5-51.

<a id="5">[Russell et al. 2005]</a>
Russell, N., Ter Hofstede, A.H., Edmond, D. and Van der Aalst, W.M., 2005, October. Workflow data patterns: Identification, representation and tool support. In International Conference on Conceptual Modeling (pp. 353-368). Springer, Berlin, Heidelberg.


***
<sup id="1"><a href="https://aws.amazon.com/api-gateway" title="AWS API Gateway">1. https://aws.amazon.com/api-gateway</a></sup>

<sup id="2"><a href="https://en.wikipedia.org/wiki/MapReduce" title="MapReduce">2. https://en.wikipedia.org/wiki/MapReduce</a></sup>

<sup id="20"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-sequence?tabs=javascript" title="Timers">20. https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-sequence?tabs=javascript</a></sup>

<sup id="3"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-timers?tabs=javascript" title="Timers">3. https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-timers?tabs=javascript</a></sup>

<sup id="4"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-orchestrations?tabs=javascript\#orchestration-history" title="MessageHistory">4. https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-orchestrations?tabs=javascript\#orchestration-history</a></sup>

<sup id="5"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=javascript\#fan-in-out" title="FanInOut">5. https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=javascript\#fan-in-out</a></sup>

<sup id="6"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-sub-orchestrations?tabs=javascript" title="NestedWorkflow">6. https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-sub-orchestrations?tabs=javascript</a></sup>

<sup id="7"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-external-events?tabs=javascript" title="CallBack">7. https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-external-events?tabs=javascript</a></sup>

<sup id="8"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-error-handling?tabs=javascript" title="ErrorHandling">8. https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-error-handling?tabs=javascript</a></sup>
