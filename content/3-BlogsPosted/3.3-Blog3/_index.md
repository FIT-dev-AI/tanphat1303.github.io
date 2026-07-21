---
title: "Modernizing applications with Strands Agents and Amazon Bedrock AgentCore"
date: 2026-07-10
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---


### AUTOMATING APPLICATION MODERNIZATION WITH STRANDS AGENTS, AWS TRANSFORM CUSTOM, AND AMAZON BEDROCK AGENTCORE

![Agentic AI Architecture](/images/3-BlogsPosted/3.3-Blog3/blog3.png)

Hello everyone. In this article, I will share a solution featured on the AWS DevOps & Developer Productivity Blog: large-scale application modernization using Generative AI Agents, Strands Agents, AWS Transform Custom, and Amazon Bedrock AgentCore.

This article is suitable for those interested in Generative AI, Agentic AI, DevOps, Cloud Migration, and Application Modernization.

## 1. Real-world problem

During cloud transformation, organizations often need to modernize many legacy applications. Common tasks include:

- Upgrading application runtime versions.
- Migrating old SDKs to newer ones.
- Refactoring frameworks.
- Standardizing codebases across multiple repositories.
- Checking dependencies and identifying what needs to change.

If done manually, each repository must be analyzed, modified, tested, and deployed individually. When the number of applications reaches hundreds of repositories, the process can take months or even years and often leads to inconsistent results across teams.

That is why the key challenge is how to automate analysis, create transformations, and execute changes across many repositories in a consistent, controlled, and scalable way.

## 2. Solution overview and workflow

The solution uses an agentic AI architecture where multiple AI agents work together to automate application modernization.

Users can interact with the system through a React UI or an API. From there, they can submit either a single repository or a list of repositories using a CSV file. The request is processed asynchronously through an API layer and forwarded to an orchestrator agent running on Amazon Bedrock AgentCore.

The workflow can be summarized as follows:

1. The user submits a repository or a CSV file.
2. The API receives the request.
3. The Orchestrator Agent processes it on Amazon Bedrock AgentCore.
4. The agent analyzes the codebase and determines the appropriate transformation.
5. If no transformation exists, it creates a new one with AI.
6. The Execution Agent runs the transformation using AWS Batch.
7. Results are stored and displayed in the UI for tracking.

In more detail, the system starts by analyzing the repository to identify the programming language, dependencies, and upgrade needs such as runtime version changes or SDK migration. If a suitable transformation already exists, the system reuses it. If not, the creation agent automatically generates a new transformation from natural-language requirements and adds it to the registry for future reuse.

After the transformation is identified or created, the execution agent runs the conversion at scale using AWS Batch jobs. Each job executes the AWS Transform Custom CLI, enabling parallel processing across many repositories rather than sequentially handling one application at a time.

## 3. Main services and tools

- AWS Transform Custom: used to create and run reusable transformations for runtime, SDK, or framework upgrades.
- Strands Agents: a framework for building multi-agent systems and coordinating complex workflows.
- Amazon Bedrock AgentCore: provides managed runtime, memory, and observability for running AI agents reliably.
- Amazon Bedrock: provides the Generative AI foundation to support analysis and transformation generation.
- AWS Batch: runs transformation jobs in parallel across many repositories.
- Amazon S3: stores the frontend, input data, or processing results.
- Amazon CloudFront: distributes the web UI to users.
- AWS Lambda: handles asynchronous processing and connects the API with the AgentCore runtime.
- Amazon DynamoDB: stores job state and processing information.
- AWS CDK and AWS SAM: used to deploy the infrastructure and the agent runtime.

## 4. Highlights of the solution

The biggest strength of this solution is the use of multiple AI agents to automate application modernization.

Instead of developers manually reading each repository, identifying what needs to be upgraded, and writing transformation logic themselves, the system can automatically perform important tasks such as:

- Analyzing repositories.
- Identifying the language, dependencies, and frameworks in use.
- Determining the right transformation.
- Creating new transformations when needed.
- Running transformations across many repositories.
- Tracking status and results.

Another useful point is that the system can learn and expand over time. When the creation agent produces a new transformation, it can be published to a registry and reused for other repositories later.

## 5. Main flows in the application

### Flow 1: Create a custom transformation from natural language

The user opens the Create Custom tab and enters a request in natural English, such as "Upgrade Spring Boot 2 applications to Spring Boot 3". The creation agent then analyzes a reference repository, creates the transformation definition, and allows the user to review it before publishing it to the registry.

### Flow 2: Run transformations across many repositories using CSV

The user uploads a CSV file containing repository URLs and target transformations. After submission, each line in the CSV becomes a separate AWS Batch job. These jobs can run in parallel, making it much faster to process large numbers of repositories.

### Flow 3: Monitor job status

Users can track processing progress through the Jobs tab. The interface shows the status of each job, the transformation results, and the output for review.

## 6. Benefits of the solution

This solution brings many benefits to both businesses and engineering teams:

- Reduces manual effort in application modernization.
- Automatically analyzes repositories and identifies change needs.
- Can create new transformations using Generative AI.
- Reuses transformations across different repositories.
- Processes many repositories in parallel with AWS Batch.
- Improves consistency across teams during modernization.
- Supports workflow tracking, management, and result control.
- Can scale to other code analysis and automation use cases.

## 7. Some deployment considerations

When deploying this solution in practice, a few points are worth noting:

- IAM permissions should be tightly controlled for Lambda, AWS Batch, S3, DynamoDB, and Bedrock AgentCore.
- Transformations should be reviewed carefully before being applied to real repositories.
- A review step should exist before publishing transformations to the registry.
- Costs should be monitored when running many AWS Batch jobs or using models on Amazon Bedrock.
- Repository information must be kept secure, especially if the codebase contains sensitive data.
- Automated testing should be integrated to confirm that the transformed code still works correctly.

## 8. Expansion opportunities

This architecture is not limited to application modernization. It can also be extended to many other workflows, such as:

- Large-scale codebase analysis.
- Automatic framework upgrades.
- SDK migration across versions.
- Standardizing coding practices across repositories.
- Integrating into CI/CD pipelines for continuous modernization.
- Supporting cloud migration by automatically handling source code changes.

## 9. Conclusion

Through this article, I see that Generative AI is not only useful for building chatbots or answering questions, but can also directly support complex technical workflows such as application modernization.

The combination of Strands Agents, AWS Transform Custom, and Amazon Bedrock AgentCore makes it possible to build an agentic AI system that can analyze repositories, identify necessary changes, create new transformations, and execute large-scale modernization workflows.

This is a practical example of how AWS is applying Generative AI to DevOps and Application Modernization, helping organizations reduce upgrade time, increase consistency, and accelerate cloud transformation.

### Reference

- [AWS Blog - Use generative AI agents for application modernization at scale with Strands, Amazon Transform Custom, and Amazon Bedrock AgentCore](https://aws.amazon.com/vi/blogs/devops/use-generative-ai-agents-for-application-modernization-at-scale-with-strands-amazon-transform-custom-and-amazon-bedrock-agentcore/)
- [Modernizing Applications with Strands Agents, AWS Transform Custom and Amazon Bedrock AgentCore - Facebook Discussion](https://www.facebook.com/groups/awsstudygroupfcj/posts/2210139623084331)
