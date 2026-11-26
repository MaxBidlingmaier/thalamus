# CAP-07 — Federated inference access

> **Status:** Draft capability definition for review. The high-level need is confirmed; this document specifies intended product behavior, not current implementation or a release commitment.

Questions: Keep "remote" or just thalamus instance?

## Purpose and intended outcome

Thalamus enables inference users to discover and invoke models served by independently operated remote Thalamus instances through a common local discovery and inference interface. Users select a distinguishable model identifier without configuring a separate endpoint or remote service credential for each instance.

This capability aggregates access to inference services; it does not execute models or manage remote serving workloads. Remote model maintainers retain authority over their deployments, and remote service operators retain authority over access, capacity, and service operation.

Federation must make remote execution explicit. A common API must not obscure that prompts, context, and other inference inputs leave the local deployment perimeter when a remote model is invoked.

## Scope and product boundaries

- Federation connects to **remote Thalamus instances**, not arbitrary third-party inference services or directly to inference engines.
- The federated access component exposes OpenAI-compatible and Anthropic-compatible inference interfaces. A request is forwarded only when the selected remote model supports the requested API format and operation.
- OpenAI-to-Anthropic or Anthropic-to-OpenAI translation is not included. Exposing both interfaces does not imply that every model is invocable through both.
- The capability is an independently operable, standalone component, not a mandatory part of the inference runtime. Local model serving must not require federation to be enabled. Its relationship to the common discovery and inference experience is a product integration requirement; its internal structure is outside this specification.
- In a federated Thalamus deployment, users must be able to discover local and permitted remote models together and invoke them through the common interface, with their execution boundaries distinguishable.
- Federation does not transfer model artifacts, including model weights, runtime images, or deployment parameters. It provides access to remotely served models without deploying or managing their serving workloads, changing remote deployment parameters, or granting remote administrative rights.
- Load balancing between instances serving the same model, smart model routing, automatic spillover, and automatic failover to another instance are out of scope.
- This capability does not cache inference requests or responses. Any inference caching, if provided, is the responsibility of the inference runtime. Retaining model-catalog information for discovery does not constitute inference caching.

## Roles and responsibilities

| Role | Responsibility in this capability |
|---|---|
| **Inference user** | Discovers permitted models, selects a model and supported API format, and submits inference inputs appropriate to the advertised remote execution boundary. |
| **Local service operator** | Configures remote service connections, names and prefixes, trust, credentials, publication modes, refresh intervals, and discovery and inference access policies; operates and troubleshoots the federation service. |
| **Local model maintainer** | Selects and manages remote-model bindings for connections using maintainer-controlled publication, within operator policies. This does not confer authority over remote deployments. |
| **Remote model maintainer** | Manages remote model deployments and their model information, supported interactions, and availability. |
| **Remote service operator** | Controls the remote service's catalog and inference access, credentials, policies, capacity, consumption limits, and operation. |
| **Platform operator** | Supplies permitted network connectivity, identity facilities, certificates, trust material, secret management, and observability services for the respective site. |

Holding inference-use permissions must not grant connection-management, model-publication, or remote administrative permissions. Local and remote roles remain independently accountable for their respective service boundaries.

## Product requirements

### 1. Remote service connections

The local service operator must be able to establish, inspect, update, disable, and remove connections to remote Thalamus instances.

Each connection must identify:

- The remote endpoint URL.
- The remote API type: OpenAI-compatible or Anthropic-compatible.
- A connection name and a unique endpoint prefix used to distinguish its models in the local catalog.
- Credentials for remote catalog retrieval.
- The inference authentication mode and, where applicable, configured inference credentials.
- The publication mode: automatic or maintainer-controlled.
- A configurable catalog refresh interval.
- Applicable local trust, discovery, inference access, and data-perimeter policies.

Configuration errors and unsuccessful connection establishment must be visible to the operator. Merely configuring a connection must not be presented as proof of successful authentication, a current catalog, or working inference access.

Disabling or removing a connection must prevent new inference requests from being forwarded through it. Removing it must remove its models from the currently available federated offering. The treatment of requests already in progress requires a separate lifecycle decision.

### 2. Trust and authentication

Remote connections must use HTTPS with trust established through platform-provided certificate and security facilities. Federation must not silently bypass certificate validation or weaken the configured execution-boundary policies to restore connectivity.

Catalog retrieval uses operator-managed credentials dedicated to that purpose. It must not depend on an inference user's credentials being present. The catalog retrieved this way is an input to local publication and access decisions, not an entitlement for every local user to discover or invoke every listed model.

The operator must be able to select either of these inference authentication modes per connection:

| Mode | Intended behavior |
|---|---|
| **Passthrough** | Forward the caller's applicable authentication credentials to the selected remote instance. The remote instance decides whether those credentials authorize the request. |
| **Configured API key** | Authenticate the forwarded request with an operator-managed API key. The operator may reuse the catalog-retrieval credentials or configure separate inference credentials. Users do not need to obtain or see this remote key. |

Local authorization applies in both modes. In particular, using a shared remote API key must not allow users to bypass local model-access policies. The remote instance independently enforces its own authorization and service policies.

Credentials must be protected using platform secret-management facilities and must not be exposed in model catalogs, user-facing errors, or routine diagnostic output. Passthrough credentials must be sent only to the selected, permitted remote instance.

Future authentication extensions may include workload identities, short-lived service tokens, OAuth-based service authorization, and mutual TLS. These are extension possibilities, not requirements for the initial capability; their behavior is not specified here.

### 3. Catalog acquisition and refresh

The component must retrieve the catalog of each enabled remote connection and generate a dynamic federated catalog from the resulting model information.

Catalog retrieval must occur:

- When an enabled connection is first configured or relevant connection configuration changes.
- At the interval configured for that connection.
- When the local service operator explicitly requests a refresh.

“On change” refers to local connection configuration changes. Remote catalog change notifications are not required; remote changes are discovered through subsequent successful refreshes.

The service operator must be able to distinguish the last successful retrieval, the outcome of the latest refresh attempt, and the freshness of retained catalog information. A failed retrieval must not be interpreted as a successfully retrieved empty catalog.

### 4. Model publication and naming

Each connection must support an operator-selected publication mode:

| Mode | Intended behavior |
|---|---|
| **Automatic publication** | Newly discovered remote models become available in the local offering automatically, subject to the configured discovery and inference policies. No separate maintainer binding is required. |
| **Maintainer-controlled publication** | Remote models are discovered for authorized maintainers, but become available in the local offering only after a local model maintainer selects and publishes a binding. |

In either mode, inference users must see only the model information permitted by local discovery policies. Catalog visibility and permission to invoke a model remain distinct permissions.

Federated model identifiers must incorporate the configured endpoint prefix and the remote model identifier. Identical model identifiers from different connections must remain distinguishable and individually selectable. Federated identifiers must not collide with identifiers in the combined local offering.

A published identifier must select its associated connection and remote model unambiguously. The component must not silently substitute another instance offering a model with the same underlying name. The catalog must identify the remote connection and execution boundary separately from the model's identity or provider information.

Maintainers must be able to inspect their bindings and determine whether the corresponding remote model remains present and usable. A successful refresh confirming that a remote model has been removed must remove it from the currently available offering and make affected bindings' loss of availability visible. A removed model must not remain advertised as available merely because an earlier catalog contained it.

The exact prefix syntax and model-identifier representation belong to the later integration contract. Compatibility expectations when prefixes, remote identifiers, or publication modes change remain to be decided.

### 5. Model information

Because both peers are Thalamus instances, federation must use an agreed Thalamus catalog contract with a minimum viable information set and a predictable format. This capability defines the information needs, but does not yet designate individual metadata fields as mandatory or optional.

| Information category | Information the catalog must be able to convey |
|---|---|
| **Identity** | Model identifier, version, and provider information. |
| **Capabilities** | Supported interactions such as text generation, embeddings, vision, and tool calling. Listing a capability does not itself commit CAP-07 to support its invocation. |
| **Limits** | Context window and maximum output-token limits. |
| **Invocation** | Invocation information, supported API formats and operations, and streaming support. Users invoke models through the common local interface rather than configuring the remote endpoint themselves. |
| **Commercial and service details** | Pricing where applicable, availability information, and advertised rate limits. |
| **Federation context** | Prefixed model identifier, source connection, remote execution boundary, and catalog freshness or relevant connection status. |

The common catalog schema, required and optional fields, compatibility rules, and handling of missing or invalid metadata must be defined in the upcoming integration and technical architecture work. Unknown facts must not be represented as verified capabilities, unlimited capacity, or zero pricing.

### 6. Inference forwarding

An authorized request using a federated model identifier must be forwarded to the corresponding remote Thalamus connection and model. The user must not need to configure that remote endpoint separately.

Forwarding must preserve the meaning of the request and the response within the agreed API compatibility scope. Federation must not silently discard unsupported features or claim to provide capabilities absent from the selected remote model. Unsupported formats or operations must produce understandable failures rather than implicit cross-format conversion.

Streaming responses must be supported where the requested API interaction and selected remote model support streaming. Exact OpenAI and Anthropic operation coverage and compatibility levels are defined with CAP-02; the presence of an API-type selector is not a claim of complete compatibility with either API.

Local access and data-perimeter restrictions must be respected before forwarding. Remote execution must not become a means of bypassing restrictions that would otherwise prevent the request.

### 7. Availability and failure behavior

When a catalog refresh fails, the last successfully retrieved catalog must be retained with a clear stale indication and the latest known connection status. Authorized users and operators must be able to distinguish retained metadata from verified current availability.

Catalog retrieval failure alone must not be treated as proof that inference is unavailable: catalog access and inference access can fail independently. Conversely, retained catalog entries do not guarantee that invocation will succeed. Applicable policies continue to govern whether requests may be forwarded.

An error must explain what failed and which federated connection or selected model is affected, using information appropriate to the recipient's permissions. Users must be able to distinguish relevant failure categories, including:

- An unknown, removed, or unavailable selected model.
- An unsupported API format or operation.
- Local access denial versus remote authentication or authorization failure.
- Connection, trust-validation, or remote-service failure.
- A locally or remotely imposed consumption limit.
- Failure before a response begins versus interruption of an already-started streamed response.

Error messages must not reveal secrets or sensitive remote service internals. Operators must receive sufficient protected diagnostic information to identify the responsible service boundary through the platform's observability facilities.

Failure affecting one remote connection must not by itself withdraw catalog information or inference access for unrelated connections or local models. The service must not silently reroute a failed request to another instance, relax security policies, or conceal an interrupted response as a successful completion.

Timeouts, retry behavior, in-flight request handling, recovery objectives, and measurable availability or performance guarantees require further specification with CAP-17.

### 8. Policies, consumption, and operational visibility

Federation must respect applicable local and independently enforced remote service policies. A local connection or binding does not override remote model-owner conditions, authorization decisions, or consumption limits.

CAP-07 must expose relevant connection, catalog-refresh, model-publication, and forwarding failure information for monitoring and troubleshooting, without making prompts, context, or credentials part of routine diagnostic output.

Detailed federated usage attribution, quota accounting, billing-related information, and the relationship between local and remote consumption limits are deferred to CAP-10. This document does not require synchronized cross-instance quota balances or assume that remote usage information is complete.

## Observable acceptance outcomes

- An operator can configure two remote Thalamus connections exposing the same model identifier, and users can distinguish and explicitly invoke each published offering through its prefixed identifier.
- Automatic publication exposes newly discovered models within operator policies; maintainer-controlled publication does not expose them to inference users before a binding is published.
- Users can identify that a selected model executes remotely and which connection supplies it, without seeing remote credentials.
- A supported request reaches the selected remote model using the configured authentication mode. Local access denial prevents forwarding even when the configured remote API key would permit access.
- A request using an unsupported API format or operation fails clearly without cross-format translation or silent feature loss.
- A failed catalog refresh retains the last known information with visible freshness and status. A later successful refresh reflects confirmed model additions and removals.
- A failure at one remote connection is reported against that connection and does not automatically change the target, permissions, or availability of unrelated offerings.
- Federation can be operated as a standalone component, and ordinary local model serving does not depend on enabling it.

## Related capabilities and remaining decisions

| Capability | Relationship or decision still required |
|---|---|
| **CAP-01 — Model discovery** | Common catalog presentation, mandatory and optional metadata, availability and freshness representation, and local/federated naming compatibility. |
| **CAP-02 — Consistent inference access** | Supported OpenAI and Anthropic operations, compatibility levels, streaming semantics, and reporting of unsupported interactions. |
| **CAP-08 — Identity and access-policy integration** | Detailed connection and binding permissions, credential integration, and interaction of local and remote authorization. |
| **CAP-10 — Consumption limits and usage visibility** | Federated usage attribution, accounting, applicable limits, and billing-related information. |
| **CAP-12 — Monitoring and troubleshooting** | Required signals, diagnostic access, and protection of sensitive operational information. |
| **CAP-15 — Data-perimeter and security-policy control** | Detailed execution-boundary, egress, trust, retention, and audit requirements. |
| **CAP-17 — Reliability and failure behavior** | Timeouts, retries, in-flight behavior on connection changes or removal, and measurable service objectives. |

Additional product decisions remain open for renaming and publication-mode transitions, binding retention and reactivation after remote model removal, and catalog-contract version compatibility. None of these unresolved details should be inferred as a guarantee from this draft.

## Basis

- [Thalamus product specification](thalamus_product.md), particularly Roles, Boundaries, scenario 3, and CAP-01, CAP-02, CAP-07, CAP-08, CAP-10, CAP-12, CAP-15, and CAP-17.
- [CAP-07 preparation notes](CAP07_pre.md).
- Product clarifications agreed during preparation: Thalamus-only remote peers; configurable publication modes; matching API formats without translation; refresh on local configuration changes, periodically, and on demand; a shared Thalamus catalog contract with required fields still undecided; retained stale catalogs; operator-managed discovery credentials and selectable inference authentication; and detailed accounting deferred to CAP-10.
