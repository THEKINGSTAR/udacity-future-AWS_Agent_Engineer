# Udacity Future AWS Agent Engineer: Customer Support Chatbot

This repository contains the implementation of a generative AI-powered customer support chatbot using **Amazon Bedrock AgentCore**. The project demonstrates advanced prompt engineering, state management, tool integration (AWS Lambda/DynamoDB), and LLM-as-a-judge evaluation frameworks.

## 🏗️ Architecture Overview

The system uses a single Amazon Bedrock AgentCore harness governed by a strict system prompt. The prompt acts as the orchestrator, classifier, and state machine, routing user intents into one of three distinct paths:

1. **Bug Reports (Tool Use):** The agent sequentially collects missing parameters (`description`, `stepsToReproduce`, `environment`). Once the gate condition is met, it executes a Bedrock Gateway tool call to an AWS Lambda function, persisting the ticket in DynamoDB.
2. **Platform Questions (RAG/Embedded context):** The agent answers operational queries strictly using an embedded FAQ document. 
3. **Other Requests (Hand-off):** Unrelated queries are immediately routed to a polite human hand-off response.

## 📂 Repository Structure

### Documentation & Rubric
* `README.md`: The primary entry point and project overview.
* `AI_ASSISTANCE.md`: Documentation of Generative AI tools utilized during the development of this project.
* `LICENSE.md`: Repository licensing information.
* `PROJECT_RUBRIC/`: Contains the official Udacity project requirements, evaluation criteria (`PROJECT_RUBRIC.md`), and supporting figures.

### Submission Evidence
* `AWS_CLI_screen_shots/`: Contains all required visual evidence for the Udacity evaluation, including:
  * DynamoDB table records verifying tool execution (`bug-report-tool-stack-bug-reports.jpeg`).
  * CloudFormation stack deployments (`bug-report-testing-stack.jpeg`).
  * Bedrock evaluation job metric results (`support-chatbot-eval-run-1.jpeg`).
  * Written analysis of the LLM-as-a-judge evaluation (`observations.txt`).

### Core Application (`/project/starter/`)
* `system_prompt.txt`: The core architectural controller that defines the three categories, state management rules, and embeds the FAQ.
* `create_harness.py`: Boto3 script to dynamically provision the Bedrock AgentCore harness.
* `chat.py`: CLI interface for interacting with the Bedrock agent.
* `create_bug_report.py`: The AWS Lambda function handler deployed behind the AgentCore Gateway.
* `online_shop_faq.md`: The knowledge base embedded into the system prompt for platform questions.
* `setup_gateway.py` & `cleanup_agentcore.py`: Scripts for provisioning and tearing down Bedrock dependencies.

### Testing & Evaluation (`/project/starter/`)
* `harness-tests.json`: The test suite defining inputs and expected behaviors for all three routing paths.
* `generate-eval-dataset.py`: Automation script that invokes the harness for each test case in isolated sessions.
* `output_eval_dataset.jsonl`: The generated dataset used for the evaluation job.
* `cloudformation-testing.yaml` & `cloudformation-tool.yaml`: IaC templates for provisioning the evaluation S3 bucket, IAM roles, and Lambda/DynamoDB infrastructure.

## ✅ Udacity Rubric Mapping

| Rubric Requirement | Implementation / Evidence Location |
| :--- | :--- |
| **Classification and Routing** | Enforced via `system_prompt.txt`. Evidence: `AWS_CLI_screen_shots/first_chat_with_bot.png`. |
| **Bug Report Path (Tool Use)** | Executes `[tool call] bugreports___create_bug_report` only when all 3 fields are collected. Evidence: `AWS_CLI_screen_shots/bug-report-tool-stack-bug-reports.jpeg`. |
| **Platform Question Path** | Handled via `{{FAQ}}` injection in the prompt. Fallback logic explicitly defined for uncovered questions. Evidence captured in chat transcripts. |
| **Testing and Evaluation** | Evaluated using Amazon Bedrock Evaluations via `amazon.nova-pro-v1:0`. Evidence: `output_eval_dataset.jsonl`, `observations.txt`, and `AWS_CLI_screen_shots/support-chatbot-eval-run-1.jpeg`. |

## 🚀 Execution Steps

**1. Provision the Harness**
```bash
cd project/starter
python create_harness.py
```

**2. Manual Testing**
Start a multi-turn conversation to verify state retention and tool execution:
```bash
python chat.py
```

**3. Automated Evaluation Pipeline**
Generate the dataset against the active harness:
```bash
python generate-eval-dataset.py --tests-json harness-tests.json
```