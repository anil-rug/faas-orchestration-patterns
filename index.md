<style>
img
{
    display:block;
    float:none;
    margin-left:auto;
    margin-right:auto;
    width:60%;
}

embed
{
    width="100%";
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
<embed type="text/html" src="aws_step_functions\process_manager.html">
</details>

<details>
<summary><b>Zeebe</b></summary>
<embed type="text/html" src="aws_step_functions\process_manager.html" width="100%" style="background: whitesmoke">
</details>

<details>
<summary><b>Azure Durable Functions</b></summary>
<embed type="text/html" src="aws_step_functions\process_manager.html" width="100%" style="background: whitesmoke">
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


## References

<a id="1">[Hohpe and Woolf 2004]</a>
Hohpe, G. and Woolf, B., 2004. Enterprise integration patterns: Designing, building, and deploying messaging solutions. Addison-Wesley Professional.
