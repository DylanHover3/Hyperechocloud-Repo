---
title: Getting Started with Azure AI Foundry
description: Learn how to set up, develop, and scale enterprise-grade AI solutions using Azure AI Foundry — Microsoft's unified PaaS for generative AI and language applications.
date: 2025-08-05
image: "/images/placeholder value"
tags: [Azure, AI Foundry, Machine Learning, Generative AI, Azure OpenAI]
---

# Getting Started with Azure AI Foundry

Azure AI Foundry is Microsoft's next-generation platform-as-a-service (PaaS) offering for building, deploying, and scaling enterprise-grade AI applications. Designed to unify agents, models, tools, and infrastructure under a single operational framework, Azure AI Foundry simplifies the development of generative AI and language-based applications. This guide outlines how to set up and use Azure AI Foundry to create intelligent applications from prototype to production.

## Understanding Azure AI Foundry

### What Is Azure AI Foundry?

Azure AI Foundry is an integrated **Platform-as-a-Service (PaaS)** designed to streamline the development, orchestration, and deployment of large-scale artificial intelligence (AI) applications. It acts as a centralized platform that bridges the operational gap between creating machine learning models and deploying them into production environments.

By offering unified application programming interfaces (APIs), role-based access control (RBAC), and policy management, Azure AI Foundry helps enterprises maintain consistency, security, and compliance across their AI workflows. The platform supports a seamless transition from experimentation to production, reducing friction between data science teams and IT operations.

### Key Pillars

#### Unified Interface

Azure AI Foundry provides a consolidated environment that brings together model development tools, prebuilt and custom models, and deployment pipelines. This unified interface simplifies collaboration across teams and reduces context-switching, making it easier to manage the full lifecycle of AI solutions.

#### Enterprise-Grade Infrastructure

Built on Microsoft Azure’s cloud infrastructure, the platform ensures high availability, scalability, and security. This foundation supports mission-critical workloads and aligns with enterprise-grade service-level agreements (SLAs), making it suitable for organizations with complex regulatory and operational needs.

#### Multi-Model Support

One of the core strengths of Azure AI Foundry is its ability to abstract the technical differences among various large language model (LLM) providers. It achieves this by offering a consistent API contract, allowing developers to switch or combine models from different vendors without reengineering their applications. This flexibility supports innovation while minimizing vendor lock-in.

#### Responsible AI

The platform includes built-in tools for monitoring, fairness assessment, and compliance checks. These features help organizations implement **responsible AI** practices by ensuring models are transparent, auditable, and aligned with ethical and legal standards. Continuous monitoring capabilities also allow for the detection of drift and bias over time.

### Why It Matters

Azure AI Foundry represents a shift from bespoke, artisanal AI workflows to standardized, industrial-grade pipelines. This transformation enables organizations to scale their AI initiatives more efficiently and with greater control over governance and security. According to [Nasstar](https://www.nasstar.com/insights/introduction-to-azure-ai-foundry), this industrialization of AI development is a key enabler for enterprise adoption.

The platform currently supports over 70,000 customers and processes approximately 100 trillion tokens every quarter, as noted in a [YouTube overview](https://www.youtube.com/watch?v=MJSwDJgOHS8). This scale underscores its capability to handle diverse and demanding AI workloads across industries.

## Setting Up Your AI Factory

### Prerequisites

Before configuring Azure AI Foundry, ensure you have the following in place:

* An active Azure subscription with sufficient permissions to create and manage resources.
* Access to the Azure CLI or the Azure portal for provisioning and configuration tasks.
* Basic familiarity with Azure AI services such as Form Recognizer and Azure AI Language Service. This foundational knowledge will help in selecting and integrating the appropriate services into your AI Factory.

### Provisioning Azure AI Foundry

To begin using Azure AI Foundry, you need to provision a workspace. This workspace serves as the central environment for building, deploying, and managing AI pipelines.

1. Visit the [Azure AI Foundry documentation](https://learn.microsoft.com/en-us/azure/ai-foundry/what-is-azure-ai-foundry) to understand the capabilities and requirements.
2. Open the Azure portal and search for "Azure AI Foundry" in the search bar.
3. Select **Create** to initiate a new AI Foundry workspace. During setup, choose a region that aligns with your data residency and latency requirements. Also, select a pricing tier that matches your expected usage.
4. Configure foundational settings:
* **Networking**: Decide whether to expose the workspace publicly or restrict access via private endpoints.
* **Role-Based Access Control (RBAC)**: Assign roles to users and services based on the principle of least privilege.
* **Policy controls**: Define governance policies such as resource quotas, data access restrictions, and audit logging.

Once the workspace is provisioned, it becomes the operational hub for your AI workflows.

### Integrating Models and Data

After setting up the workspace, the next step is to integrate models and data sources. Azure AI Foundry supports both pre-trained large language models (LLMs) and custom models.

You can import models from Azure OpenAI, Hugging Face, or other providers. These models can be fine-tuned or used as-is depending on your application. For data, upload structured datasets (e.g., CSV files), unstructured documents (e.g., PDFs, Word files), or images directly into the workspace.

Foundry provides tools for:

* **Labeling**: Annotate data for supervised learning tasks.
* **Preprocessing**: Clean and normalize data to ensure consistency.
* **Tokenization**: Segment text into tokens suitable for model input.

These steps help prepare your data for training, evaluation, or inference.

### Example Use Case: Automating Text Extraction

A common scenario is extracting key information from documents using the Azure AI Language Service. Below is an example of how to use the Key Phrase Extraction API via a REST call:

```http
POST https://<endpoint>/language/analyze-text/jobs?api-version=2022-10-01-preview
Content-Type: application/json
Ocp-Apim-Subscription-Key: <API_KEY>

{
  "analysisInput": {
     "documents": [
       {"id": "1", "language": "en", "text": "Azure AI Foundry simplifies AI deployment."}
     ]
  },
  "tasks": [
     {"kind": "KeyPhraseExtraction"}
  ]
}

```

This request submits a document and instructs the service to extract key phrases. The response will contain terms like “Azure AI Foundry” and “AI deployment”, which can be used in downstream workflows such as classification or summarization.

For implementation details and SDK options, see the full guide on the [Key Phrase Extraction API](https://learn.microsoft.com/en-us/azure/ai-services/language-service/key-phrase-extraction/how-to/call-api).

## Building AI Applications with Foundry

### Visual & Code-Based Tools

Azure AI Foundry supports a wide range of user personas, from non-developers to experienced software engineers. For users with minimal programming experience, Foundry provides a **UI-based workflow orchestration** tool. This interface allows users to visually design AI pipelines by dragging and connecting components such as data sources, AI models, and post-processing steps. The system automatically handles dependency resolution and execution order, making it easier to construct and modify workflows without writing code.

For developers, Foundry offers **Software Development Kits (SDKs)** and extensions for .NET, Python, and JavaScript. These SDKs expose APIs for programmatically defining, training, and deploying AI models within Foundry. Developers can also integrate custom logic, extend existing workflows, or embed Foundry-powered AI into larger applications.

Foundry integrates natively with **Azure DevOps** and **GitHub**, enabling Continuous Integration and Continuous Deployment (CI/CD) workflows. This allows AI application components—such as models, data preprocessing scripts, and deployment configurations—to be versioned, tested, and deployed automatically as part of a software delivery pipeline. Developers can use pull requests, automated testing, and release pipelines to manage changes and reduce deployment risk.

### Agent Composition

Azure AI Foundry supports the creation of **composite agents** by combining multiple AI models into a single pipeline. These agents operate as modular systems where each component handles a specific task. Foundry uses **declarative schemas** to define agent behavior, specifying the sequence of operations, model inputs and outputs, and fallback logic in case of errors.

For example, consider a **document triage agent** designed to process incoming customer support documents. This agent might:

1. Use Optical Character Recognition (OCR) to extract text from scanned documents.
2. Apply a sentiment analysis model to assess customer tone.
3. Run a summarization model to generate a concise version of the document.

Each of these steps is implemented as a discrete module, and the overall behavior is defined using a high-level schema. This modularity allows developers to swap models, adjust parameters, or add new capabilities without rewriting the entire pipeline.

### Deployment & Monitoring

Once an AI application is defined, Foundry supports **one-click deployment** to Azure Kubernetes Service (AKS) or Azure App Services. This abstracts away the complexity of container orchestration or web service configuration, enabling teams to move from development to production quickly.

Foundry includes **built-in observability** features to monitor deployed agents. This includes latency tracking to measure response times, token usage monitoring for large language models, and quality metrics such as model accuracy or prediction confidence. All metrics are accessible via dashboards and can be exported to Azure Monitor or other observability platforms.

Importantly, Foundry supports **continuous fine-tuning** of models based on live production data. Developers can define feedback loops that capture user interactions or error cases, retrain models with this new data, and redeploy updated versions. This enables applications to adapt over time and maintain performance in changing environments.

## Scaling from Prototype to Production

Moving from a working prototype to a production-scale deployment requires deliberate planning, tooling, and governance. Azure AI Foundry provides a structured ecosystem to ease this transition, particularly by abstracting infrastructure concerns and offering unified APIs for model lifecycle management.

### Best Practices

Begin with a minimal proof-of-concept (PoC) that demonstrates core functionality. This reduces initial complexity and allows faster iteration. Azure AI Foundry supports this approach by enabling developers to quickly stand up experiments using pre-configured pipelines and services.

Once a PoC is validated, teams can scale confidently by leveraging Azure AI Foundry’s unified APIs. These APIs allow you to swap models—such as moving from a basic OCR to a more advanced language model—without rewriting the application logic. This abstraction separates model implementation from application integration, allowing faster upgrades and experimentation.

For secure collaboration, enable **Role-Based Access Control (RBAC)**. RBAC restricts access to resources based on user roles, ensuring that only authorized team members can modify models, access datasets, or initiate deployments. This is essential in multi-team environments where data sensitivity or compliance requirements are in play.

Policy enforcement mechanisms should also be configured early. Azure AI Foundry supports governance policies that help ensure data privacy, enforce model usage boundaries, and maintain audit trails. These controls are critical in regulated industries or where explainability and accountability are required.

### Case Study: Bulk Image Text Extraction

Mobilize.net illustrates a practical use case of scaling with Azure AI Foundry in a [blog post](https://www.mobilize.net/blog/azure-ai-foundry-what-is-it-and-why-should-you-care). In this scenario, a system was designed to process large volumes of images by extracting text in bulk.

The workflow began with users uploading multiple images to a shared storage location. Azure AI Foundry’s orchestration tools then triggered Optical Character Recognition (OCR) models in batch mode. Instead of processing each image manually or asynchronously, the system handled the entire batch in a single operation pipeline.

The extracted text was normalized and structured, then written into a database for downstream use—such as search indexing or analytics. This example demonstrates how a prototype OCR system can be scaled to handle production-level throughput using Foundry’s orchestration and model deployment capabilities.

### Long-Term Management

Transitioning to production is not the endpoint. Ongoing model reliability and relevance must be maintained. Azure AI Foundry supports scheduled automated retraining, allowing models to update periodically as new data becomes available. This reduces the risk of performance degradation over time.

Monitoring for **model drift**—where the statistical properties of input data change—helps maintain accuracy. Foundry includes tools to track performance metrics and alert teams when retraining may be necessary.

Finally, built-in dashboards allow stakeholders to visualize key performance indicators such as return on investment (ROI), processing latency, or user engagement. These metrics help justify continued investment and guide future iterations.

By combining these capabilities, Azure AI Foundry supports a robust path from experimental prototype to scalable, governed, and maintainable production systems.

## Leveraging the Azure Ecosystem

### Interoperability with Azure Services

Azure AI Foundry is designed to integrate seamlessly with core components of the Azure ecosystem, allowing developers and data scientists to build, train, and deploy AI solutions using familiar tools and services.

**Azure Cognitive Services** provide prebuilt APIs for common AI workloads such as computer vision, speech recognition, and natural language processing. These APIs are accessible through REST endpoints and SDKs, enabling quick prototyping and production-grade deployment without requiring deep machine learning expertise.

**Azure OpenAI Service** offers access to large language models like GPT-4 through a managed API. This allows teams to build generative AI applications using the same backend models as OpenAI, while benefiting from Azure’s enterprise-grade infrastructure, scalability, and compliance guarantees.

**Azure Machine Learning (Azure ML)** supports custom model development and deployment. Azure AI Foundry can connect directly to Azure ML workspaces, enabling users to orchestrate training pipelines, manage compute resources, and monitor model performance using integrated dashboards and MLOps tooling.

### Developer Ecosystem Support

Azure AI Foundry supports a wide range of development workflows and tools commonly used in modern software engineering.

**Visual Studio Code (VS Code) extensions** for Azure AI allow developers to manage resources, write and test code, and deploy models directly from the editor. These extensions include syntax highlighting, code snippets, and integrated terminals for Azure CLI commands.

The **Azure SDKs**, available in multiple languages including Python, JavaScript, and C#, provide programmatic access to AI services and infrastructure. This allows developers to automate deployments, manage datasets, and interact with models as part of larger applications.

Integration with **GitHub Copilot** and **GitHub Codespaces** further enhances productivity. Developers can use Copilot to generate AI-related code suggestions in real time, while Codespaces offers cloud-hosted development environments that are preconfigured for Azure AI workflows.

### Compliance & Security

Azure AI Foundry inherits Azure’s enterprise-grade compliance and security features by default. This includes built-in adherence to major regulatory standards such as the **General Data Protection Regulation (GDPR)**, the **Health Insurance Portability and Accountability Act (HIPAA)**, and other regional or industry-specific frameworks.

All data handled within Azure AI Foundry is protected with **end-to-end encryption**, both in transit and at rest. Additionally, **audit logging** is enabled across services, allowing organizations to track access and modifications to sensitive data and models. These features support secure collaboration and help organizations meet internal governance and external compliance requirements.

## Conclusion

Azure AI Foundry delivers a streamlined path for developing, deploying, and managing AI solutions at scale. Its unified interface and integration with the broader Azure ecosystem reduce the complexity typically associated with moving AI models from research to production. With support for a wide range of use cases—including natural language processing, computer vision, and document summarization—Azure AI Foundry equips teams with the resources to build secure, scalable, and maintainable AI systems.

By consolidating infrastructure, governance, and tooling into a cohesive platform, it allows organizations to focus on delivering business value through AI without being burdened by operational overhead. For developers and enterprises alike, Azure AI Foundry represents a practical foundation for industrializing AI workloads efficiently and responsibly.