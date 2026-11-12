# Thalamus product specification

## Purpose

> **Status:** Draft for review. These are proposed product responsibilities, not claims of current implementation. Detailed integration contracts and enforcement responsibilities remain to be agreed.

### Why Thalamus exists

Thalamus exists to enable organizations to offer and consume large language model (LLM) inference as a service within an infrastructure and security perimeter they control. Its intended purpose is to make sovereign, self-hosted inference practical without requiring each application team to assemble and operate its own model-serving stack.

Thalamus helps organizations control model artifacts and inference deployment, including isolation of serving workloads. It automates model lifecycle, request routing, and serving capacity. This turns models and compute resources into a manageable inference service within an organization-controlled perimeter.

Thalamus does not create or manage application prompts and context. Its responsibility is to route inference requests to permitted deployments and integrate with the backend and platform controls that protect data during inference.

### Intended value

For inference users, Thalamus aims to provide a stable, standards-compatible way to discover and invoke available models, starting with an OpenAI-compatible inference interface. Users and their applications should not need to understand the deployment and routing details of each serving backend.

For model maintainers and platform operators, Thalamus aims to provide a consistent, Kubernetes-native way to manage models and make them available as a service. This reduces the need to coordinate backend-specific deployment, readiness, discovery, and gateway configuration separately for every model.

Vendor neutrality is a central product goal: organizations should retain meaningful choice over models, accelerator hardware, and serving backends rather than become dependent on a single infrastructure or inference provider. This does not imply that every combination is supported or that different backends have identical capabilities.

The intended deployment scope includes datacenters, sovereign clouds, and disconnected satellite sites. Keeping sensitive assets within the chosen perimeter is an end-to-end objective shared with the surrounding platform; Thalamus alone does not establish regulatory compliance, air-gap readiness, or confidential-computing guarantees.

### Relationship to inference engines and serving platforms

Thalamus is intended to be a **backend-independent model-management and inference-service layer**, not a new inference engine. It composes serving technologies and platform services into a consistent product experience rather than reimplementing their execution, scheduling, or optimization capabilities.

The related projects occupy different, partly overlapping roles:

- **vLLM** is an inference engine and model server. It executes models and handles inference requests. Thalamus adds service-level model management and exposure around engines rather than replacing their execution logic.
- **llm-d** is a Kubernetes-native distributed inference serving stack. It focuses on serving models efficiently at scale, including inference-aware routing and distributed execution. Thalamus uses such capabilities as building blocks for its model-management and shared-service experience.
- **NVIDIA Dynamo** is a distributed inference framework that coordinates inference engines, routing, cache reuse, and scaling. Its intended relationship to Thalamus is as a possible serving backend, not an engine that Thalamus must reproduce.
- **KServe** is a Kubernetes model-serving platform for predictive and generative AI. **KAITO** is a Kubernetes operator suite that automates model deployment and related AI workloads. Thalamus aims to offer a common management and inference-service interface across backend choices, with sovereignty and platform integration as guiding product concerns.

These technologies can be complementary to Thalamus, although their responsibilities may overlap. Thalamus combines consistent model management and inference access with an included, project-maintained serving backend intended for independently operated, sovereign deployments across supported hardware vendors. Additional backend integrations offer choice without making an externally maintained serving platform mandatory. Sovereignty remains an end-to-end property of the deployment, not a guarantee of backend selection alone.

Thalamus will provide an architecture that enables multiple serving backends, but the project itself will deliver and maintain only one included serving backend implementation. Additional backend implementations are the responsibility of their respective providers or contributors; support for each must be established separately.

## Roles

The following roles describe responsibilities and goals, not necessarily separate people or organizations. One person or team may hold multiple roles; detailed permissions and workflows will be specified separately.

### Inference user

Discovers and invokes available models through an application, AI interface, or automated client. Needs understandable model information and reliable inference access. The inference API should be standard and client-agnostic.

### Model maintainer

Selects model artifacts, configures deployments, and manages model lifecycle and availability. Uses externally fine-tuned models and adjusts supported deployment parameters to meet model and service requirements. Acts within the model owner's usage conditions and the service operator's policies and capacity limits.

Each Thalamus deployment may have its own model maintainers. In satellite and federated scenarios, the remote site's maintainer manages models deployed there, while the local maintainer configures the binding that exposes those remote models locally, within the service operator's connection and access policies. This binding does not grant the local maintainer authority to change remote model deployments.

### Thalamus service operator

Configures and operates a Thalamus instance, including access policies, consumption limits, backend integrations, and physical connections to remote instances. Is responsible for service health and coordinates with model maintainers and platform operators. Connecting to a remote instance does not grant authority over its deployments.

### Platform operator

Supplies and operates the surrounding Kubernetes, compute, networking, identity, artifact storage, security, and observability services. Provides the infrastructure capacity and platform facilities on which Thalamus depends, without necessarily managing individual models or inference-service policies.

### Model owner

Controls the rights to a model and defines its permitted distribution and use, including requirements for protecting proprietary weights. May authorize model maintainers to deploy or distribute the model without operating the receiving infrastructure or Thalamus instance.

### Backend provider

Develops and maintains a serving-backend integration, including its compatibility with Thalamus and its supported models, hardware, and configuration options. Communicates capabilities and limitations to operators and maintainers. The Thalamus team holds this role for the included backend; other providers or contributors hold it for additional integrations.

In federated and distributed-site scenarios, **local** and **remote** identify where a role's authority applies rather than introduce different roles. Each site's owners and operators retain responsibility for their own deployments.

## Boundaries

### What Thalamus provides

Thalamus provides a consistent model-management and inference-service layer: managing the lifecycle and availability of models, exposing model discovery and inference access, and integrating serving backends into a common service. The project provides the architecture for multiple backends and delivers and maintains one included serving backend implementation; additional implementations belong to their respective providers or contributors.

Thalamus will provide a packaging format that bundles models, their weights, and deployment parameters for distribution and lifecycle management in air-gapped deployments and remote satellite scenarios.

### What the surrounding platform provides

The deployment platform supplies and operates the general-purpose infrastructure and services on which Thalamus depends. The following boundaries describe responsibilities without prescribing particular vendors or products.

| Area | Surrounding platform responsibility | Thalamus responsibility |
|---|---|---|
| Kubernetes and compute | Operate clusters, networking, compute capacity, accelerators, and drivers. | Deploy and manage inference workloads on supported infrastructure through the selected backend. |
| Identity and access | Supply identities, credentials, tenant context, and identity services. | Integrate with these services and enforce the agreed model-management and inference-access policies. |
| Observability | Operate collection, storage, visualization, and alerting services for metrics, logs, and traces. | Expose relevant model-lifecycle, inference, and service-health signals for monitoring and troubleshooting. |
| Artifact storage | Operate registries and storage services for runtime images and model weights. | Integrate with controlled artifact sources and use the configured access credentials. |
| Security infrastructure | Supply certificates, secrets, keys, and infrastructure isolation facilities. | Use these facilities according to the service's security requirements. |
| Quotas and consumption limits | Supply allocated infrastructure capacity and infrastructure-level quotas. | Apply agreed inference-specific limits, such as request rate, concurrency, or token usage, and expose consumption data. |

Infrastructure quotas constrain the resources available to workloads; inference-service quotas constrain how users or tenants consume the service. Their relationship, policy ownership, accounting semantics, and enforcement mechanisms require further specification. Identity and authorization responsibilities likewise need to distinguish model management from inference access.

Externally provided does not mean outside Thalamus's integration responsibilities. The detailed specification must define required and optional dependencies, supported interfaces, and expected behavior when dependencies are unavailable. Disconnected deployments must not assume continuous access to centrally hosted platform services.

### What is outside the product boundary

Thalamus does not provide or operate a general-purpose Kubernetes platform, implement a new inference engine, or deliver a complete identity, observability, registry, or key-management platform. It does not maintain every backend implementation enabled by its architecture.

Thalamus also does not independently guarantee regulatory compliance, sovereign operation, or confidential processing. These properties depend on the complete deployment, including its infrastructure, artifacts, policies, and operational practices.

Model training and application-level AI usage are out of scope. Thalamus does not itself perform model fine-tuning, but must enable model maintainers to use fine-tuned models and adjust deployment parameters through the selected backend. The availability of additional capabilities in an integrated backend does not automatically make them Thalamus capabilities.

Thalamus does not provide application-level prompt construction, context management, content filtering, moderation, or response guardrails. Applications or separately integrated services remain responsible for these functions.

## Key scenarios

### 1. Standalone private-cloud inference

An organization operates one Thalamus instance within its controlled deployment perimeter. Model maintainers deploy models locally, and inference users discover and invoke them through the instance's API. The local owner manages model deployments and service operation, with the surrounding platform supplying the infrastructure and services defined in the Boundaries section.

The principal outcome is a shared inference service without requiring each consuming application or user to assemble a separate serving stack. Inference executes locally, without forwarding prompts and context to a remote inference service.

### 2. Standalone disconnected inference

An organization operates the same local inference service without depending on internet access. Installation, model deployment, updates, identity, and operational services rely on locally available dependencies and transferred artifacts, including packaged models, weights, deployment parameters, and required runtime images.

This scenario includes **air-gapped** sites with no network connection to external systems and **air-gated** sites with explicitly controlled connectivity. Their artifact-transfer and update procedures may differ, but inference operation must not require public internet access. Required platform services must be available locally or through suitable offline equivalents rather than depend on continuous access to central services.

### 3. Federated inference access

A local Thalamus instance is configured to access a remote Thalamus instance whose models are deployed and operated by a separate owner. Inference users discover and invoke locally and remotely served models through the local API endpoint, which presents a combined set of available models and forwards remote-model requests to the remote instance.

The remote owner retains full responsibility for the remote instance, including model deployment, updates, capacity, and operation. Federation provides inference access; it does not grant the local owner management authority over remote deployments.

Remote inference sends prompts and context beyond the local deployment perimeter. Model discovery and access policies must make the remote execution boundary explicit rather than hide it behind the combined API. Authorization, model-name conflicts, remote availability, and quota accounting require further specification.

### 4. Distributing models to independently operated sites

A model maintainer packages a model, its weights, and deployment parameters for distribution to another site. The receiving site's owner imports the artifacts and deploys and manages the model locally through its own Thalamus instance, including when the site is disconnected.

The principal outcome is repeatable model distribution and lifecycle management without requiring the receiving site to fetch artifacts from public services or surrender operational ownership to the distributing site. Package versions, compatibility, verification, and update procedures require further specification.

Unlike federated inference access, this scenario transfers artifacts rather than forwarding inference requests: execution takes place at the receiving site, and prompts and context remain within that site's inference perimeter.

### Cross-cutting security variants

Protected partner models and restricted-data inference are security variants that may apply across these scenarios, not separate deployment patterns. Their isolation, confidentiality, and trust requirements must be defined with the relevant backend and surrounding platform; they are not automatic properties of a Thalamus deployment.

## High-level capabilities

This draft describes user needs and intended outcomes, not implementation designs or release commitments. References to scenarios use the numbering in Key scenarios above. Security, reliability, performance, and operability are part of the capability specification, even where fulfillment depends on a backend or surrounding platform service.

Requirement status is separate from implementation maturity:

- **Confirmed need:** The high-level need has been accepted in the preceding product discussion; the wording and details below remain subject to review.
- **Documented intention:** Project documentation describes the intention, but it has not yet been accepted as a product requirement.
- **Open question:** A candidate need or responsibility requires an explicit product decision.

### CAP-01 — Model discovery

Inference users can discover the models available through their Thalamus instance so that they can choose a model without knowing its deployment details. In federated deployments, discovery presents local and remote models together while making their execution location and data boundary distinguishable.

**Status:** Confirmed need. **Scenarios:** 1–3. **Sources:** [Intended value](#intended-value), [federated inference scenario](#3-federated-inference-access), [website inference API](../../website/concepts/architecture.md#inference-api). **Open questions:** Which model metadata is needed, how are naming conflicts resolved, and how do availability and permissions affect discovery? Do we need spill over configuration capabilities or is this matter for on top solutions?

### CAP-02 — Consistent inference access

Inference users can invoke available models through a stable, standards-compatible API, starting with OpenAI compatibility, without configuring a separate endpoint for every deployment or backend. Supported interactions include streamed responses so that consuming applications can present results as they are generated.

**Status:** Confirmed need for common inference access; streaming is a documented intention. **Scenarios:** 1–3. **Sources:** [Intended value](#intended-value), [website inference API](../../website/concepts/architecture.md#inference-api), [project feature intentions](../../README.md#features). **Open questions:** Which operations and compatibility levels are required, and how are backend-specific limitations communicated? Anthropic interface?

### CAP-03 — Model lifecycle management

Model maintainers can declare which models should be deployed, update their configuration, activate or deactivate them, and remove them through a consistent management interface. They can determine whether the requested state has been reached and understand why a model is not available, without managing every serving component individually.

**Status:** Confirmed need. **Scenarios:** 1–4, with deployment management remaining local to each owner. **Sources:** [Product responsibilities](#what-thalamus-provides), [website native backend overview](../../website/concepts/backends/native/index.md). **Open questions:** What are the expected update, deactivation, and removal semantics, including requests already in progress?

### CAP-04 — Maintainer-controlled model and deployment tuning

Model maintainers can deploy externally fine-tuned models and adjust supported deployment parameters to suit their hardware, model requirements, and service objectives. Thalamus enables this configuration through the selected backend without performing model training or fine-tuning itself.

**Status:** Confirmed need. **Scenarios:** 1–4. **Sources:** [Scope boundary](#what-is-outside-the-product-boundary), [backend-independent management intentions](../../website/backend-hackathon-2026/index.md). **Open questions:** Which settings are portable, which are backend-specific, and how are invalid or unsupported combinations reported?

### CAP-05 — Controlled artifact sourcing

Model maintainers and platform operators can deploy models using weights and runtime images from sources under the organization's control. They can identify which artifacts a deployment uses and avoid mandatory reliance on a public model registry or internet download service.

**Status:** Confirmed need. **Scenarios:** 1, 2, and 4; remote artifacts in scenario 3 remain the remote owner's responsibility. **Sources:** [Artifact-storage boundary](#what-the-surrounding-platform-provides), [disconnected scenario](#2-standalone-disconnected-inference), [model-weight distribution motivation](../../website/ipcei-cis-workshop-2026/ocm-model-weights.md). **Open questions:** Which source types, version references, provenance information, and verification policies are required?

### CAP-06 — Model packaging and distribution

Model maintainers can package models, weights, and deployment parameters in a transferable format so another site can reproduce the intended deployment. Receiving owners can import and manage these packages locally, including at air-gapped sites, without granting the distributing owner control over their deployment.

**Status:** Confirmed need. **Scenarios:** 2 and 4. **Sources:** [Packaging responsibility](#what-thalamus-provides), [distribution scenario](#4-distributing-models-to-independently-operated-sites), [artifact-distribution exploration](../../website/ipcei-cis-workshop-2026/ocm-model-weights.md). **Open questions:** What must a package contain or reference, how are runtime dependencies transferred, and how are versions, compatibility, and authenticity checked?

### CAP-07 — Federated inference access

Platform operators can configure access to independently managed remote Thalamus instances and expose their models through the local discovery and inference API. Inference users can consume those models without separate endpoint configuration, while the remote owner retains all deployment and operational authority.

**Status:** Confirmed need. **Scenario:** 3. **Source:** [Federated inference scenario](#3-federated-inference-access), agreed in this product discussion rather than established by existing implementation documentation. **Open questions:** How are remote trust, credentials, catalog changes, connectivity failures, and cross-instance usage limits handled?

### CAP-08 — Identity and access-policy integration

Platform operators can integrate the service with the platform's identities and tenant context and control who may manage, discover, and invoke models. Model-management permissions and inference-use permissions are distinct so that consuming a model does not imply authority to change its deployment.

**Status:** Confirmed need at the responsibility level. **Scenarios:** 1–4. **Sources:** [Identity boundary](#what-the-surrounding-platform-provides), [website access integration](../../website/gateway-integrations.md#authentication). **Open questions:** Which identity interfaces and permission scopes are required, and how are local and remote permissions combined without granting unintended access?

### CAP-09 — Tenant isolation and sharing policies

Platform operators can define how tenants share models and serving resources, with explicit boundaries for data access, operational visibility, and interference between workloads. Inference users and model owners can understand the isolation guarantees of the service they consume rather than infer them from authentication alone.

**Status:** Open question. **Scenarios:** 1–3; receiving sites in scenario 4 choose their own policy. **Source:** Not an agreed commitment to particular isolation classes but some definition is needed as it's a multi tenant system. **Open questions:** Is multi-tenancy required, which shared or dedicated service classes are needed, and what isolation must Thalamus enforce versus the backend or platform?

### CAP-10 — Consumption limits and usage visibility

Platform operators can apply agreed inference-service limits and view consumption by the relevant user or tenant and model so that capacity is protected and usage is understandable. Inference users receive a clear indication when a limit prevents a request. Infrastructure quotas and inference-service quotas remain separate responsibilities.

**Status:** Confirmed need at the responsibility level. **Scenarios:** 1–3. **Sources:** [Quota boundary](#what-the-surrounding-platform-provides), [website rate limiting](../../website/gateway-integrations.md#rate-limiting) and [token counting](../../website/gateway-integrations.md#token-counting). **Open questions:** Which request, concurrency, and token limits are required, how strict must enforcement be, and how is federated consumption attributed? Billing and chargeback are not assumed requirements but billing related information needs to be persisted and either a push format or an API endpoint to fetch is needed.

### CAP-11 — Capacity management and efficient serving

Model maintainers and platform operators can align serving capacity with demand and service objectives within the compute allocated by the platform. The service uses available model replicas effectively, and operators can understand capacity pressure rather than discover it only through failed requests.

**Status:** Documented intention. **Scenarios:** 1 and 2; each owner manages its own capacity in scenarios 3 and 4. **Sources:** [Project scaling intentions](../../README.md#features), [website serving stack](../../website/concepts/architecture.md#kubernetes-ai-runtime), [service metrics](../../website/perses-dashboards.md). **Open questions:** Are manual scaling, automatic scaling, or scale-to-zero required, and what are the expected latency, throughput, warm-up, and overload behaviors?

### CAP-12 — Monitoring and troubleshooting

Platform operators and model maintainers can observe model availability, deployment failures, inference performance, usage, and resource pressure through the platform's observability services. They can distinguish deployment, backend, gateway, and remote-connectivity problems using relevant metrics, logs, and traces rather than relying on backend-specific investigation alone.

**Status:** Confirmed need at the integration level. **Scenarios:** 1–4. **Sources:** [Observability boundary](#what-the-surrounding-platform-provides), [website operations](../../website/concepts/architecture.md#operations), [dashboard purposes](../../website/perses-dashboards.md). **Open questions:** Which signals and diagnostic information are mandatory, who may access them, and how are sensitive prompts, context, and credentials excluded or protected?

### CAP-13 — Installation, updates, and disconnected operation

Platform operators can install and update Thalamus and its required serving components on a supported platform with identifiable versions and prerequisites. At disconnected sites, installation, model lifecycle operations, and inference continue using locally available artifacts and platform services without requiring internet access or a continuously reachable central authority.

**Status:** Confirmed need for standalone and disconnected operation; update procedures require review. **Scenarios:** 1–4. **Sources:** [Standalone scenarios](#key-scenarios), [website installation guide](../../website/getting-started.md), [stack-packaging intention](../../website/ipcei-cis-workshop-2026/ocm-packaging.md). **Open questions:** Which installation profiles, compatibility checks, update paths, and recovery procedures are required, and how do air-gapped and air-gated transfers differ?

### CAP-14 — Backend extensibility

Backend providers can integrate independently maintained serving implementations into Thalamus's common model-management and inference-service architecture. This allows operators to use a backend with required capabilities or an existing backend investment while retaining the common Thalamus interface.

**Status:** Confirmed need for extensibility. **Scenarios:** 1–4. **Sources:** [Backend responsibility](#relationship-to-inference-engines-and-serving-platforms), [provider architecture intentions](../../website/backend-hackathon-2026/index.md). **Open questions:** What is the minimum provider contract, how is compatibility demonstrated, and which concrete requirements justify additional backend integrations?

### CAP-15 — Data-perimeter and security-policy control

Platform operators and model owners can configure artifact access, inference execution, and operational data handling to respect the organization's chosen security perimeter. Inference users can distinguish local from remote execution, and access to a remote model must not silently override restrictions on where prompts and context may be processed.

**Status:** Confirmed need at the perimeter and responsibility level. **Scenarios:** 1–4. **Sources:** [Product purpose](#why-thalamus-exists), [security boundary](#what-the-surrounding-platform-provides), [federated data boundary](#3-federated-inference-access), [project mission](../../CHARTER.md#1-mission). **Open questions:** Which placement, egress, retention, audit, and operator-access policies must the service support, and which guarantees belong to the platform?

### CAP-16 — Confidential-processing integration

Model owners and inference users with stronger confidentiality needs can use a deployment profile that protects proprietary weights and sensitive inference data within an attested processing boundary. Thalamus integrates with the necessary backend and platform facilities rather than treating confidential processing as a guarantee of every deployment.

**Status:** Documented intention. **Scenarios:** Security variant across applicable deployments. **Sources:** [Security variants](#cross-cutting-security-variants), [website confidential-computing intentions](../../website/concepts/architecture.md#confidential-computing). **Open questions:** Which threat model, hardware profiles, attestation, key-release, and verification responsibilities are required?

### CAP-17 — Reliability and failure behavior

Inference users receive understandable outcomes when a model, backend, remote instance, or required dependency is unavailable. Platform operators can identify affected service functions and restore operation without unintentionally changing access or data-perimeter policies. The detailed specification defines measurable availability and performance objectives rather than assuming that a running deployment is a reliable service.

**Status:** Open question for concrete guarantees and recovery scope. **Scenarios:** 1–4. **Sources:** [Dependency boundary](#what-the-surrounding-platform-provides), [documented service-performance signals](../../website/perses-dashboards.md). **Open questions:** What objectives, timeouts, request handling, recovery guarantees, and failure-domain expectations are needed? Automatic failover and disaster recovery are not assumed commitments.

### CAP-18 — Ecosystem catalog integration

Platform operators can make Thalamus model availability and relevant deployment metadata available to external asset catalogs and operational tools so that users can discover and inspect inference resources through the organization's existing systems. Thalamus supplies model information without becoming an application-workflow or general-purpose catalog platform.

**Status:** Documented intention. **Scenarios:** Connected deployments; applicability to disconnected sites requires review. **Source:** [Website catalog-integration exploration](../../website/ipcei-cis-workshop-2026/naira-integration.md). **Open questions:** Is external catalog integration a core requirement or an optional integration, and what information, synchronization behavior, and access restrictions are needed?

### CAP-19 — Included, project-maintained serving backend

Thalamus includes a serving backend delivered and maintained by the Thalamus team, enabling operators to deploy and serve models without obtaining a separately maintained backend integration. The team owns its integration, compatibility, testing, and maintenance; underlying inference engines and other upstream components remain the responsibility of their respective projects.

**Status:** Confirmed need. **Scenarios:** 1–4, with local serving managed by each deployment owner. **Sources:** [Backend responsibility](#relationship-to-inference-engines-and-serving-platforms), [product responsibilities](#what-thalamus-provides), and the explicit requirement agreed in this product discussion. **Open questions:** Which models must the included backend support, and what compatibility and maintenance guarantees apply?

### CAP-20 — Hardware choice

Platform operators and model maintainers can deploy inference workloads on supported accelerator hardware from multiple vendors, reducing dependence on a single hardware supplier. Supported combinations and their limitations are explicit; hardware portability does not imply identical performance or capabilities.

**Status:** Documented intention. **Scenarios:** 1–4, with hardware selected by each deployment owner. **Sources:** [Vendor-neutral product purpose](#intended-value), [website multi-vendor hardware](../../website/concepts/architecture.md#multi-vendor-hardware). **Open questions:** Which hardware vendors and configurations are required, how are supported combinations validated, and which must be supported by the included backend versus additional integrations?
