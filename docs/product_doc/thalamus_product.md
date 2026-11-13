# Thalamus product specification

## Purpose

> **Status:** Draft for review. These are proposed product responsibilities, not claims of current implementation. Detailed integration contracts and enforcement responsibilities remain to be agreed.

### Why Thalamus exists

Thalamus exists to enable organizations to offer and consume large language model (LLM) inference as a service within an infrastructure and security perimeter they control. Its intended purpose is to make sovereign, self-hosted inference practical without requiring each application team to assemble and operate its own model-serving stack.

Thalamus helps organizations control model artifacts and inference deployment, including isolation of serving workloads. It automates model lifecycle, request routing, and serving capacity. This turns models and compute resources into a manageable inference service within an organization-controlled perimeter.

Thalamus does not create or manage application prompts and context. Its responsibility is to route inference requests to permitted deployments and integrate with provider and platform controls that protect data during inference.

### Intended value

For inference users, Thalamus aims to provide a stable, standards-compatible way to discover and invoke available models, starting with an OpenAI-compatible inference interface. Users and their applications should not need to understand the deployment and routing details of each provider.

For model maintainers and Thalamus service operators, Thalamus aims to provide a consistent, Kubernetes-native way to manage models and make them available as a service. This reduces the need to coordinate provider-specific deployment, readiness, discovery, and gateway configuration separately for every model.

Vendor neutrality is a central product goal: organizations should retain meaningful choice over models, accelerator hardware, and providers rather than become dependent on a single infrastructure or serving-technology vendor. This does not imply that every combination is supported or that different providers have identical capabilities.

The intended deployment scope includes datacenters, sovereign clouds, and disconnected satellite sites. Keeping sensitive assets within the chosen perimeter is an end-to-end objective shared with the surrounding platform; Thalamus alone does not establish regulatory compliance, air-gap readiness, or confidential-computing guarantees.

### Relationship to inference engines and serving platforms

Thalamus is intended to be a **provider-independent model-management and inference-service layer**, not a new inference engine. It composes serving technologies and platform services into a consistent product experience rather than reimplementing their execution, scheduling, or optimization capabilities.

The related projects occupy different, partly overlapping roles:

- **vLLM** is an inference engine and model server. It executes models and handles inference requests. Thalamus adds service-level model management and exposure around engines rather than replacing their execution logic.
- **llm-d** is a Kubernetes-native distributed inference serving stack. It focuses on serving models efficiently at scale, including inference-aware routing and distributed execution. Thalamus uses such capabilities as building blocks for its model-management and shared-service experience.
- **NVIDIA Dynamo** is a distributed inference framework that coordinates inference engines, routing, cache reuse, and scaling. It could be integrated into Thalamus through an external provider, not reproduced as a new inference engine by Thalamus.
- **KServe** is a Kubernetes model-serving platform for predictive and generative AI. **KAITO** is a Kubernetes operator suite that automates model deployment and related AI workloads. Thalamus aims to offer a common management and inference-service interface across provider choices, with sovereignty and platform integration as guiding product concerns.

These technologies can be complementary to Thalamus, although their responsibilities may overlap. Thalamus combines consistent model management and inference access with a native provider intended for independently operated, sovereign deployments across supported hardware vendors. External providers offer choice without making an externally maintained serving platform mandatory. Sovereignty remains an end-to-end property of the deployment, not a guarantee of provider selection alone.

Thalamus will provide an architecture that enables multiple providers, but the project itself will deliver and maintain only the native provider. External providers are the responsibility of their respective maintaining teams, vendors, or contributors; support for each must be established separately.

## Terminology

| Term | Meaning |
|---|---|
| **Inference engine** | Software that executes a model, such as vLLM. |
| **Provider** | The integration that connects a serving backend to Thalamus's model-management and inference-service contract, including the serving stack based on technologies such as llm-d, Dynamo, KServe, or KAITO. |
| **Native provider** | The provider delivered and maintained by the Thalamus team, together with its supported serving stack. “Native” does not imply that upstream components are developed by Thalamus. |
| **External provider** | An additional provider maintained by another team, vendor, or contributor. |

These terms describe components. **Provider maintainer** below identifies the person or team responsible for a provider, not the component itself.

## Roles

The following roles describe responsibilities and goals, not necessarily separate people or organizations. One person or team may hold multiple roles; detailed permissions and workflows will be specified separately.

### Inference user

Discovers and invokes available models through an application, AI interface, or automated client. Needs understandable model information and reliable inference access without managing serving infrastructure. Permission to consume a model does not imply permission to change its deployment or to deploy other models.

### Model maintainer

Selects model artifacts, configures deployments, and manages model lifecycle and availability. Uses externally fine-tuned models and adjusts supported deployment parameters to meet model and service requirements. Acts within the model owner's usage conditions and the service operator's policies and capacity limits.

Each Thalamus deployment may have its own model maintainers. In satellite and federated scenarios, the remote site's maintainer manages models deployed there, while the local maintainer configures the binding that exposes those remote models locally, within the service operator's connection and access policies. This binding does not grant the local maintainer authority to change remote model deployments.

### Thalamus service operator

Installs, configures, updates, and operates a Thalamus instance, including access policies, consumption limits, serving-capacity allocations, providers, and service connections to remote instances. Is responsible for service health and coordinates with model maintainers and platform operators. Connecting to a remote instance does not grant authority over its deployments.

### Platform operator

Supplies and operates the surrounding Kubernetes, compute, networking, identity, artifact storage, security, and observability services. Provides the infrastructure capacity and platform facilities on which Thalamus depends, without necessarily managing individual models or inference-service policies.

### Model owner

Controls the rights to a model and defines its permitted distribution and use, including requirements for protecting proprietary weights. May authorize model maintainers to deploy or distribute the model without operating the receiving infrastructure or Thalamus instance.

### Provider maintainer

Develops and maintains a provider, including its compatibility with Thalamus and its supported models, hardware, and configuration options. Communicates capabilities and limitations to Thalamus service operators and model maintainers. The Thalamus team holds this role for the native provider; other teams, vendors, or contributors hold it for external providers.

In federated and distributed-site scenarios, **local** and **remote** identify where a role's authority applies rather than introduce different roles. Each site's model maintainers, service operator, and platform operator retain their respective deployment, service, and infrastructure responsibilities.

### Role coordination

The roles separate concerns relevant to Thalamus's product scope. They do not prescribe an organizational structure, reporting hierarchy, or operational workflow. Deploying organizations may assign several roles to one team or distribute a role across multiple teams.

Thalamus distinguishes model management, service administration, and platform administration. Model maintainers manage model deployments; service operators configure Thalamus and its service policies; platform operators manage the underlying infrastructure and surrounding services. Model owners define permitted model use and distribution, while provider maintainers develop and maintain providers.

These distinctions identify where separate permissions or delegated authority may be needed. Holding one role must not automatically imply another role's authority—for example, inference access does not grant model-management rights, and access to remote models does not grant control over remote deployments. Detailed permission requirements will be defined per capability.

Thalamus must expose relevant state and constraints, such as deployment readiness, capacity limitations, and access restrictions, so the responsible roles can act. How organizations coordinate approvals, capacity requests, escalation, and incident response is outside the product specification.

## Boundaries

### What Thalamus provides

Thalamus provides a consistent model-management and inference-service layer: managing the lifecycle and availability of models, exposing model discovery and inference access, and connecting providers to a common service. The project provides the architecture for multiple providers and delivers and maintains the native provider; external providers belong to their respective maintaining teams, vendors, or contributors.

Thalamus will provide a packaging format that bundles models, their weights, and deployment parameters for distribution and lifecycle management in air-gapped deployments and remote satellite scenarios.

### What the surrounding platform provides

The deployment platform supplies and operates the general-purpose infrastructure and services on which Thalamus depends. The following boundaries describe responsibilities without prescribing particular vendors or products.

| Area | Surrounding platform responsibility | Thalamus responsibility |
|---|---|---|
| Kubernetes and compute | Operate clusters, networking, compute capacity, accelerators, and drivers. | Deploy and manage inference workloads on supported infrastructure through the selected provider. |
| Identity and access | Supply identities, credentials, tenant context, and identity services. | Integrate with these services and enforce the agreed model-management and inference-access policies. |
| Observability | Operate collection, storage, visualization, and alerting services for metrics, logs, and traces. | Expose relevant model-lifecycle, inference, and service-health signals for monitoring and troubleshooting. |
| Artifact storage | Operate registries and storage services for runtime images and model weights. | Integrate with controlled artifact sources and use the configured access credentials. |
| Security infrastructure | Supply certificates, secrets, keys, and infrastructure isolation facilities. | Use these facilities according to the service's security requirements. |
| Quotas and consumption limits | Supply allocated infrastructure capacity and infrastructure-level quotas. | Apply agreed inference-specific limits, such as request rate, concurrency, or token usage, and expose consumption data. |

Infrastructure quotas constrain the resources available to workloads; inference-service quotas constrain how users or tenants consume the service. Their relationship, policy ownership, accounting semantics, and enforcement mechanisms require further specification. Identity and authorization responsibilities likewise need to distinguish model management from inference access.

Externally provided does not mean outside Thalamus's integration responsibilities. The detailed specification must define required and optional dependencies, supported interfaces, and expected behavior when dependencies are unavailable. Disconnected deployments must not assume continuous access to centrally hosted platform services.

### What is outside the product boundary

Thalamus does not provide or operate a general-purpose Kubernetes platform, implement a new inference engine, or deliver a complete identity, observability, registry, or key-management platform. It does not maintain external providers enabled by its architecture.

Thalamus also does not independently guarantee regulatory compliance, sovereign operation, or confidential processing. These properties depend on the complete deployment, including its infrastructure, artifacts, policies, and operational practices.

Model training and application-level AI usage are out of scope. Thalamus does not itself perform model fine-tuning, but must enable model maintainers to use fine-tuned models and adjust deployment parameters through the selected provider. The availability of additional capabilities in a provider does not automatically make them Thalamus capabilities.

Thalamus does not provide application-level prompt construction, context management, content filtering, moderation, or response guardrails. Applications or separately integrated services remain responsible for these functions.

## Key scenarios

### 1. Standalone private-cloud inference

An organization operates one Thalamus instance within its controlled deployment perimeter. Model maintainers deploy and manage models locally, inference users discover and invoke them through the instance's API, and the local service operator manages service operation. The platform operator supplies the infrastructure and services defined in the Boundaries section.

The principal outcome is a shared inference service without requiring each consuming application or user to assemble a separate serving stack. Inference executes locally, without forwarding prompts and context to a remote inference service.

### 2. Standalone disconnected inference

An organization operates the same local inference service without depending on internet access. Installation, model deployment, updates, identity, and operational services rely on locally available dependencies and transferred artifacts, including packaged models, weights, deployment parameters, and required runtime images.

This scenario includes **air-gapped** sites with no network connection to external systems and **air-gated** sites with explicitly controlled connectivity. Their artifact-transfer and update procedures may differ, but inference operation must not require public internet access. Required platform services must be available locally or through suitable offline equivalents rather than depend on continuous access to central services.

### 3. Federated inference access

A local Thalamus instance is configured to access a remote Thalamus instance whose models are deployed and operated by a separate owner. Inference users discover and invoke locally and remotely served models through the local API endpoint, which presents a combined set of available models and forwards remote-model requests to the remote instance.

The remote site's model maintainers manage its model deployments, while its service operator manages service updates, serving-capacity allocations, and operation, coordinating with its platform operator for infrastructure capacity. Federation provides inference access; it does not grant local roles management authority over remote deployments.

Remote inference sends prompts and context beyond the local deployment perimeter. Model discovery and access policies must make the remote execution boundary explicit rather than hide it behind the combined API. Authorization, model-name conflicts, remote availability, and quota accounting require further specification.

### 4. Distributing models to independently operated sites

A model maintainer packages a model, its weights, and deployment parameters for distribution to another site under the model owner's distribution conditions. The receiving site's model maintainer imports the artifacts and deploys and manages the model locally through its own Thalamus instance, within that site's service policies, including when the site is disconnected.

The principal outcome is repeatable model distribution and lifecycle management without requiring the receiving site to fetch artifacts from public services or surrender operational ownership to the distributing site. Package versions, compatibility, verification, and update procedures require further specification.

Unlike federated inference access, this scenario transfers artifacts rather than forwarding inference requests: execution takes place at the receiving site, and prompts and context remain within that site's inference perimeter.

### Cross-cutting security variants

Protected partner models and restricted-data inference are security variants that may apply across these scenarios, not separate deployment patterns. Their isolation, confidentiality, and trust requirements must be defined with the relevant provider and surrounding platform; they are not automatic properties of a Thalamus deployment.

## High-level capabilities

This draft describes user needs and intended outcomes, not implementation designs or release commitments. References to scenarios use the numbering in Key scenarios above. Security, reliability, performance, and operability are part of the capability specification, even where fulfillment depends on a provider or surrounding platform service.

Each capability's **Responsibilities** line identifies role ownership at a high level, not a permission matrix or implementation mechanism. “Service operator” means the Thalamus service operator defined in Roles. Assignments for capabilities still under review are proposed boundaries, not confirmation of those capabilities or their detailed guarantees.

Requirement status is separate from implementation maturity:

- **Confirmed need:** The high-level need has been accepted in the preceding product discussion; the wording and details below remain subject to review.
- **Documented intention:** Project documentation describes the intention, but it has not yet been accepted as a product requirement.
- **Open question:** A candidate need or responsibility requires an explicit product decision.

### CAP-01 — Model discovery

Inference users can discover the models available through their Thalamus instance so that they can choose a model without knowing its deployment details. In federated deployments, discovery presents local and remote models together while making their execution location and data boundary distinguishable.

**Responsibilities:** Inference user discovers and selects models; model maintainer supplies model and deployment metadata and configures local or remote model bindings; service operator sets discovery and access policies.

**Status:** Confirmed need. **Scenarios:** 1–3. **Sources:** [Intended value](#intended-value), [federated inference scenario](#3-federated-inference-access), [website inference API](../../website/concepts/architecture.md#inference-api). **Open questions:** Which model metadata is needed, how are naming conflicts resolved, and how do availability and permissions affect discovery? Do we need spill over configuration capabilities or is this matter for on top solutions?

### CAP-02 — Consistent inference access

Inference users can invoke available models through a stable, standards-compatible API, starting with OpenAI compatibility, without configuring a separate endpoint for every deployment or provider. Supported interactions include streamed responses so that consuming applications can present results as they are generated.

**Responsibilities:** Inference user invokes models; service operator configures and operates the inference endpoint and its access policies; provider maintainer supplies and documents the supported serving interactions.

**Status:** Confirmed need for common inference access; streaming is a documented intention. **Scenarios:** 1–3. **Sources:** [Intended value](#intended-value), [website inference API](../../website/concepts/architecture.md#inference-api), [project feature intentions](../../README.md#features). **Open questions:** Which operations and compatibility levels are required, and how are provider-specific limitations communicated? Anthropic interface?

### CAP-03 — Model lifecycle management

Model maintainers can declare which models should be deployed, update their configuration, activate or deactivate them, and remove them through a consistent management interface. They can determine whether the requested state has been reached and understand why a model is not available, without managing every serving component individually.

**Responsibilities:** Model maintainer manages model deployments and investigates their state; service operator sets service policies and allocations; provider maintainer supplies lifecycle support through its provider.

**Status:** Confirmed need. **Scenarios:** 1–4, with deployment management remaining local to each site's model maintainers. **Sources:** [Product responsibilities](#what-thalamus-provides), [website native provider overview](../../website/concepts/backends/native/index.md). **Open questions:** What are the expected update, deactivation, and removal semantics, including requests already in progress?

### CAP-04 — Maintainer-controlled model and deployment tuning

Model maintainers can deploy externally fine-tuned models and adjust supported deployment parameters to suit their hardware, model requirements, and service objectives. Thalamus enables this configuration through the selected provider without performing model training or fine-tuning itself.

**Responsibilities:** Model maintainer selects fine-tuned artifacts and adjusts deployment parameters; service operator sets permitted configurations and capacity limits; provider maintainer documents and supports available tuning options.

**Status:** Confirmed need. **Scenarios:** 1–4. **Sources:** [Scope boundary](#what-is-outside-the-product-boundary), [provider-independent management intentions](../../website/backend-hackathon-2026/index.md). **Open questions:** Which settings are portable, which are provider-specific, and how are invalid or unsupported combinations reported?

### CAP-05 — Controlled artifact sourcing

Model maintainers can deploy models using weights and runtime images from sources under the organization's control. They can identify which artifacts a deployment uses and avoid mandatory reliance on a public model registry or internet download service.

**Responsibilities:** Model maintainer selects artifact references and configures deployment access; service operator defines permitted source and verification policies; platform operator supplies artifact storage and credential facilities; model owner defines permitted artifact use and distribution.

**Status:** Confirmed need. **Scenarios:** 1, 2, and 4; remote artifact sourcing in scenario 3 remains the remote site's responsibility under the same role division. **Sources:** [Artifact-storage boundary](#what-the-surrounding-platform-provides), [disconnected scenario](#2-standalone-disconnected-inference), [model-weight distribution motivation](../../website/ipcei-cis-workshop-2026/ocm-model-weights.md). **Open questions:** Which source types, version references, provenance information, and verification policies are required?

### CAP-06 — Model packaging and distribution

Model maintainers can package models, weights, and deployment parameters in a transferable format so another site can reproduce the intended deployment. Receiving-site model maintainers can import and manage these packages locally, including at air-gapped sites, without granting the distributing site control over their deployment.

**Responsibilities:** Distributing model maintainer prepares packages under the model owner's distribution conditions; receiving model maintainer imports and manages deployments; each site's service operator sets transfer and import policies; platform operator supplies storage and permitted transfer facilities.

**Status:** Confirmed need. **Scenarios:** 2 and 4. **Sources:** [Packaging responsibility](#what-thalamus-provides), [distribution scenario](#4-distributing-models-to-independently-operated-sites), [artifact-distribution exploration](../../website/ipcei-cis-workshop-2026/ocm-model-weights.md). **Open questions:** What must a package contain or reference, how are runtime dependencies transferred, and how are versions, compatibility, and authenticity checked?

### CAP-07 — Federated inference access

Service operators can establish service connections to independently managed remote Thalamus instances, and local model maintainers can bind their models into the local discovery and inference API within the agreed policies. Inference users can consume those models without separate endpoint configuration, while remote roles retain their deployment and operational authority.

**Responsibilities:** Local service operator configures remote service connections, trust, and access policies; local model maintainer configures remote-model bindings; remote model maintainer manages remote deployments; remote service operator controls remote service access and operation; platform operators supply required network connectivity.

**Status:** Confirmed need. **Scenario:** 3. **Source:** [Federated inference scenario](#3-federated-inference-access), agreed in this product discussion rather than established by existing implementation documentation. **Open questions:** How are remote trust, credentials, catalog changes, connectivity failures, and cross-instance usage limits handled?

### CAP-08 — Identity and access-policy integration

Service operators can integrate Thalamus with platform-provided identities and tenant context and control who may manage, discover, and invoke models. Model-management permissions and inference-use permissions are distinct so that consuming a model does not imply authority to change its deployment.

**Responsibilities:** Service operator configures service identity integration and access policies; platform operator supplies identity services, credentials, and tenant context; model owner defines model-use conditions that service policies must respect.

**Status:** Confirmed need at the responsibility level. **Scenarios:** 1–4. **Sources:** [Identity boundary](#what-the-surrounding-platform-provides), [website access integration](../../website/gateway-integrations.md#authentication). **Open questions:** Which identity interfaces and permission scopes are required, and how are local and remote permissions combined without granting unintended access?

### CAP-09 — Tenant isolation and sharing policies

Service operators can define how tenants share models and serving resources within available infrastructure isolation facilities, with explicit boundaries for data access, operational visibility, and interference between workloads. Inference users and model owners can understand the isolation guarantees of the service they consume rather than infer them from authentication alone.

**Responsibilities:** Service operator defines sharing and service-isolation policies; model maintainer configures deployments within them; platform operator supplies infrastructure isolation; provider maintainer documents and supports its provider's isolation capabilities; model owner specifies model-protection requirements.

**Status:** Open question. **Scenarios:** 1–3; receiving-site service operators in scenario 4 choose their own service policy within platform constraints. **Source:** Not an agreed commitment to particular isolation classes but some definition is needed as it's a multi tenant system. **Open questions:** Is multi-tenancy required, which shared or dedicated service classes are needed, and what isolation must Thalamus enforce versus the provider or platform?

### CAP-10 — Consumption limits and usage visibility

Service operators can apply site-wide and per-tenant inference-service quotas and view consumption by the relevant user or tenant and model so that capacity is protected and usage is understandable. Inference users can set and manage additional quotas on their own consumption. These self-imposed quotas apply alongside, and cannot override or relax, site-wide, tenant, or other operator-imposed limits. Inference users receive a clear indication when a limit prevents a request. Infrastructure quotas and inference-service quotas remain separate responsibilities.

**Responsibilities:** Service operator configures site-wide, tenant, and other operator-imposed inference-service limits and usage access and export policies; platform operator supplies infrastructure allocations, infrastructure quotas, and required storage facilities; inference user configures quotas for their own consumption without changing operator-imposed limits or other users' quotas.

**Status:** Confirmed need at the responsibility level, including user-configurable quotas. **Scenarios:** 1–3. **Sources:** [Quota boundary](#what-the-surrounding-platform-provides), [website rate limiting](../../website/gateway-integrations.md#rate-limiting) and [token counting](../../website/gateway-integrations.md#token-counting); user-configurable quotas are an explicit requirement agreed in this product discussion. **Open questions:** Which request, concurrency, and token limits are required, how strict must enforcement be, and how is federated consumption attributed? What scopes and time periods can users configure, how do changes or resets affect consumed quota, and how are applicable limits and remaining allowances presented? Billing and chargeback are not assumed requirements but billing related information needs to be persisted and either a push format or an API endpoint to fetch is needed.

### CAP-11 — Capacity management and efficient serving

Model maintainers and service operators can align serving capacity with demand and service objectives within the compute allocated by the platform. The service uses available model replicas effectively, and service operators and model maintainers can understand capacity pressure rather than discover it only through failed requests.

**Responsibilities:** Model maintainer configures model-serving capacity; service operator coordinates service-wide allocations and limits and requests additional infrastructure when needed; platform operator supplies infrastructure capacity; provider maintainer supplies and documents supported scaling and scheduling behavior.

**Status:** Documented intention. **Scenarios:** 1 and 2; each site's service operator, model maintainers, and platform operator manage their respective capacity responsibilities in scenarios 3 and 4. **Sources:** [Project scaling intentions](../../README.md#features), [website serving stack](../../website/concepts/architecture.md#kubernetes-ai-runtime), [service metrics](../../website/perses-dashboards.md). **Open questions:** Are manual scaling, automatic scaling, or scale-to-zero required, and what are the expected latency, throughput, warm-up, and overload behaviors?

### CAP-12 — Monitoring and troubleshooting

Service operators and model maintainers can observe model availability, deployment failures, inference performance, usage, and resource pressure through the platform's observability services. They can distinguish deployment, provider, gateway, and remote-connectivity problems using relevant metrics, logs, and traces rather than relying on provider-specific investigation alone.

**Responsibilities:** Service operator monitors service health and coordinates diagnosis; model maintainer investigates model-deployment issues; platform operator operates observability services and diagnoses infrastructure issues; provider maintainer supplies and documents provider-specific diagnostic signals.

**Status:** Confirmed need at the integration level. **Scenarios:** 1–4. **Sources:** [Observability boundary](#what-the-surrounding-platform-provides), [website operations](../../website/concepts/architecture.md#operations), [dashboard purposes](../../website/perses-dashboards.md). **Open questions:** Which signals and diagnostic information are mandatory, who may access them, and how are sensitive prompts, context, and credentials excluded or protected?

### CAP-13 — Installation, updates, and disconnected operation

Service operators can install and update Thalamus and its required serving components on a supported platform with identifiable versions and prerequisites. At disconnected sites, installation, model lifecycle operations, and inference continue using locally available artifacts and platform services without requiring internet access or a continuously reachable central authority.

**Responsibilities:** Service operator manages service installation and updates and coordinates offline readiness; platform operator supplies prerequisites, platform services, and any required cluster-level installation actions; model maintainer manages model artifacts and lifecycle locally; provider maintainer supplies compatible provider releases and dependency information.

**Status:** Confirmed need for standalone and disconnected operation; update procedures require review. **Scenarios:** 1–4. **Sources:** [Standalone scenarios](#key-scenarios), [website installation guide](../../website/getting-started.md), [stack-packaging intention](../../website/ipcei-cis-workshop-2026/ocm-packaging.md). **Open questions:** Which installation profiles, compatibility checks, update paths, and recovery procedures are required, and how do air-gapped and air-gated transfers differ?

### CAP-14 — Provider extensibility

Provider maintainers can develop external providers that connect their supported serving stacks to Thalamus's common model-management and inference-service architecture. This allows service operators to use a provider with required capabilities or to reuse an existing serving stack while retaining the common Thalamus interface.

**Responsibilities:** Provider maintainer develops, tests, maintains, and documents its provider; service operator selects and operates supported providers; model maintainer configures model deployments using their supported capabilities.

**Status:** Confirmed need for extensibility. **Scenarios:** 1–4. **Sources:** [Provider responsibility](#relationship-to-inference-engines-and-serving-platforms), [provider architecture intentions](../../website/backend-hackathon-2026/index.md). **Open questions:** What is the minimum provider contract, how is compatibility demonstrated, and which concrete requirements justify external providers?

### CAP-15 — Data-perimeter and security-policy control

Service operators can configure service policies for artifact access, inference execution, and operational data handling to respect the organization's chosen security perimeter and model-owner requirements. Model maintainers configure compliant deployments, supported by the platform's security facilities. Inference users can distinguish local from remote execution, and access to a remote model must not silently override restrictions on where prompts and context may be processed.

**Responsibilities:** Model owner defines model-protection and usage conditions; service operator applies service security and execution-boundary policies; model maintainer configures deployments within those policies; platform operator supplies network, key, secret, and infrastructure security controls; inference user selects permitted services appropriate to the data being submitted.

**Status:** Confirmed need at the perimeter and responsibility level. **Scenarios:** 1–4. **Sources:** [Product purpose](#why-thalamus-exists), [security boundary](#what-the-surrounding-platform-provides), [federated data boundary](#3-federated-inference-access), [project mission](../../CHARTER.md#1-mission). **Open questions:** Which placement, egress, retention, audit, and operator-access policies must the service support, and which guarantees belong to the platform?

### CAP-16 — Confidential-processing integration

Model owners and inference users with stronger confidentiality needs can use a deployment profile that protects proprietary weights and sensitive inference data within an attested processing boundary. Thalamus integrates with the necessary provider and platform facilities rather than treating confidential processing as a guarantee of every deployment.

**Responsibilities:** Model owner defines weight-protection requirements; inference user selects a service appropriate to the data's confidentiality needs; model maintainer configures the model for the selected profile; service operator enables and operates the service profile; platform operator supplies attestation, key, and confidential-infrastructure facilities; provider maintainer supports and documents the required provider capabilities.

**Status:** Documented intention. **Scenarios:** Security variant across applicable deployments. **Sources:** [Security variants](#cross-cutting-security-variants), [website confidential-computing intentions](../../website/concepts/architecture.md#confidential-computing). **Open questions:** Which threat model, hardware profiles, attestation, key-release, and verification responsibilities are required?

### CAP-17 — Reliability and failure behavior

Inference users receive understandable outcomes when a model, provider, remote instance, or required dependency is unavailable. Service operators can identify affected service functions and coordinate recovery without unintentionally changing access or data-perimeter policies. The detailed specification defines measurable availability and performance objectives rather than assuming that a running deployment is a reliable service.

**Responsibilities:** Service operator coordinates incident diagnosis and service recovery; model maintainer resolves model-deployment issues; platform operator restores failed infrastructure or surrounding services; provider maintainer addresses provider defects; inference user handles the service's reported failure outcomes.

**Status:** Open question for concrete guarantees and recovery scope. **Scenarios:** 1–4. **Sources:** [Dependency boundary](#what-the-surrounding-platform-provides), [documented service-performance signals](../../website/perses-dashboards.md). **Open questions:** What objectives, timeouts, request handling, recovery guarantees, and failure-domain expectations are needed? Automatic failover and disaster recovery are not assumed commitments.

### CAP-18 — Ecosystem catalog integration

Service operators can make Thalamus model availability and relevant deployment metadata available to external asset catalogs and operational tools so that users can discover and inspect inference resources through the organization's existing systems. Thalamus supplies model information without becoming an application-workflow or general-purpose catalog platform.

**Responsibilities:** Service operator configures catalog integration and metadata access policies; model maintainer supplies model and deployment metadata; platform operator supplies any platform-managed catalog infrastructure; inference user consumes the published information within its access permissions.

**Status:** Documented intention. **Scenarios:** Connected deployments; applicability to disconnected sites requires review. **Source:** [Website catalog-integration exploration](../../website/ipcei-cis-workshop-2026/naira-integration.md). **Open questions:** Is external catalog integration a core requirement or an optional integration, and what information, synchronization behavior, and access restrictions are needed?

### CAP-19 — Native provider

Thalamus delivers the native provider with its supported serving stack, enabling service operators to install and operate it and model maintainers to deploy models without obtaining an external provider. The Thalamus team owns the native provider's compatibility, testing, and maintenance; underlying inference engines and other upstream components remain the responsibility of their respective projects.

**Responsibilities:** Thalamus team acts as provider maintainer for the native provider and maintains its supported serving stack; service operator installs and operates it; model maintainer deploys and manages models through it; platform operator supplies the required infrastructure.

**Status:** Confirmed need. **Scenarios:** 1–4, with each site's service operator operating its provider and model maintainers managing local models. **Sources:** [Provider responsibility](#relationship-to-inference-engines-and-serving-platforms), [product responsibilities](#what-thalamus-provides), and the explicit requirement agreed in this product discussion. **Open questions:** Which models must the native provider support, and what compatibility and maintenance guarantees apply?

### CAP-20 — Hardware choice

Platform operators can supply supported accelerator hardware from multiple vendors, and model maintainers can deploy inference workloads on the configurations made available by the service operator. This reduces dependence on a single hardware supplier. Supported combinations and their limitations are explicit; hardware portability does not imply identical performance or capabilities.

**Responsibilities:** Platform operator supplies accelerator hardware and drivers; service operator makes supported serving configurations available; model maintainer selects deployment resources within the available configurations and allocations; provider maintainer validates and documents its provider's supported hardware combinations.

**Status:** Documented intention. **Scenarios:** 1–4, with hardware provisioned by each site's platform operator and deployment resources selected by its model maintainers within service allocations. **Sources:** [Vendor-neutral product purpose](#intended-value), [website multi-vendor hardware](../../website/concepts/architecture.md#multi-vendor-hardware). **Open questions:** Which hardware vendors and configurations are required, how are supported combinations validated, and which must be supported by the native provider versus external providers?
