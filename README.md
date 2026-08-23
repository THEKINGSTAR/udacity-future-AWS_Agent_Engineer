# Customer Support Chatbot with Amazon Bedrock Flows

In this project you will build a customer support chatbot using Amazon Bedrock Flows. The chatbot will handle customers' questions on a fictional website, and will need to handle one of the following types of messages:
* bug reports
* platform related questions that can be answered from FAQ.

There are a number of resources that will be available to you to develop this application:

* `create_bug_report` - a tool that can create a ticket in a database
* `online_shop_faq.md` - a fictional FAQ that your application should use to respond to customer requests

You would need to create a Bedrock Flow application, and then test it in various scenarios.

## Getting Started

### Dependencies

- An AWS account with Amazon Bedrock access enabled.
- AWS CLI configured with appropriate credentials.
- Python 3.9+ with `boto3` installed.
- Access to an Amazon Bedrock model (the solution uses Amazon Nova models, but you can use any supported model).

### Project Files

| File | Description |
|------|-------------|
| `docs/tools-setup.md` | Step-by-step guide for creating the bug report tool. |
| `docs/testing.md` | Step-by-step guide for automated testing and running Bedrock Evaluations. |
| `solution/` | Reference solution with the complete flow definition, test prompts, and a diagram. |
| `cloudformation-tool.yaml` | A template for creating a tool you would need for this application. |
| `cloudformation-testing.yaml` | A template for creating resources to test your final application. |
| `create_bug_report.py` | Lambda function that implements a tool that stores bug reports in DynamoDB. |
| `generate-eval-dataset.py` | Script that runs your flow against a test suite and produces a JSONL file for Bedrock Evaluations. |
| `flow-tests-template.json` | Template for developing your test suite. |

## Project Instructions

### Step 1: Create Resources for your application

First you will deploy a tool that your application will need to create bug reports and other related resources.

When a customer reports a bug, the chatbot needs to persist it somewhere so the engineering team can follow up. In this project we use a DynamoDB table as a simple ticket store, and a Lambda function as the tool that Bedrock Agents can call to create a new ticket.

Follow the detailed walkthrough in [Tools Setup](docs/tools-setup.md), to ensure that you have everything you need for the rest of the project

### Step 2: Build the Bedrock Flow

Now having a tool set up, you can start developing Bedrock Flow application. Your application would need to handle three different types of requests:

- **Bug reports** - if a customer reports a bug on the web site. In this case, the application would need to collect additional information and create a ticket for the reported bug using the tool you've created in the previous step.
- **Platform questions** - the application should answer common questions about orders, shipping, returns, and payments using an FAQ.
- **Other requests** - in case if the question cannot be answered using FAQ and not a bug report, and application should politely redirected to a human support phone line.

The tool you've deployed accepts three parameters:

* Bug description
* Steps to reproduce
* Environment where a user has experienced a bug

Make sure that your application collects this data when creating a bug report.

Platform questions (orders, shipping, returns, payments) need to be answered from the product's FAQ. Here we will use the simplest approach and embed the document directly in the prompt — the model will see it at inference time and answers from it.

> **Note:** Embedding documents in the prompt works well for short, stable content like a FAQ. For large documents, embedding the full text in every prompt becomes expensive and hits context limits. The standard solution is **Retrieval-Augmented Generation (RAG)**, which retrieves only the relevant passages at query time using a vector index. RAG with Amazon Bedrock Knowledge Bases is outside the scope of this course.

#### Some suggestions

Here are some things to keep in mind when working on your application:

* Condition nodes in Bedrock Flows use exact string matching, so the classification output needs to be predictable.
* You can use an agent node to collect more information about a bug if initial request is unclear or incomplete. To allow an agent to ask additional questions you need to enable "User input" option in "Advanced settings".
* A single Output node can't receive connections from multiple branches. You need a separate Output node for each path.
* For platform questions, embed your FAQ directly in the prompt.
* Don't forget to deploy resources once you change them. For example you need to prepare agents if you change them.
* Try to implement and test your solution step by step.
* Use us-east-1 region, as some smaller regions might not have all Bedrock features.

## Step 3: Testing

Once you have your Bedrock Flow application you can test it manually using the chat interface in Bedrock Flows. However, this approach is too tedious and not scalable. Ideally we want to have an automated way to test your application.

To test your application you will do the following:

* Create a set of test prompts and define expected results
* Run your application programmatically on this set of prompts
* Use Bedrock Evaluations to evaluate your application's outputs

You need to follow the steps in the [Testing and Evaluation](docs/testing.md) document to run automated tests and evaluate your flow.

## Cleanup

When you are done with the project, delete the CloudFormation stacks to avoid ongoing charges:

```bash
aws cloudformation delete-stack --stack-name bug-report-tool-stack --region us-east-1
aws cloudformation delete-stack --stack-name bug-report-testing-stack --region us-east-1
```

This removes the Lambda function, DynamoDB table, IAM roles, and S3 bucket created during the project.

## Built With

* [Amazon Bedrock Flows](https://docs.aws.amazon.com/bedrock/latest/userguide/flows.html) - Orchestration of the LLM application
* [Amazon Bedrock Agents](https://docs.aws.amazon.com/bedrock/latest/userguide/agents.html) - Tool use for bug report creation
* [Amazon Bedrock Evaluations](https://docs.aws.amazon.com/bedrock/latest/userguide/evaluation.html) - LLM-as-a-judge evaluation
* [AWS Lambda](https://aws.amazon.com/lambda/) - Bug report tool runtime
* [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) - Bug report storage

## License

[License](../LICENSE.md)
