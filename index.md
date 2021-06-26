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

## Table of Contents
- [Table of Contents](#table-of-contents)
- [Patterns](#patterns)
  - [**Process Manager**](#process-manager)
  - [**Event and Document Message**](#event-and-document-message)
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
    <img src="./images/aws_mapping_process_manager.png" alt="Process Manager">
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
In ADF, the Event message construct invokes the orchestration function, and the Document message handles the internal message communication between the functions.
<br/>
<div>
    <img src="./images/adf_mapping_event_document_message.png" alt="Event Document Message">
</div>
</details>



## References

<a id="1">[Hohpe and Woolf 2004]</a>
Hohpe, G. and Woolf, B., 2004. Enterprise integration patterns: Designing, building, and deploying messaging solutions. Addison-Wesley Professional.


***
<sup id="1">1. https://aws.amazon.com/api-gateway<a href="https://aws.amazon.com/api-gateway" title="AWS API Gateway"></a></sup>