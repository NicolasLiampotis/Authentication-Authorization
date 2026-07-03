# Frequently Asked Questions about EGI Check-in Integration

## Context

This document collects frequently asked questions about the integration of EGI Check-in as the federated Authentication and Authorisation Infrastructure (AAI) for ECHOES.

The questions cover the main topics relevant to the ECHOES Single Entry Point and related services, including Virtual Organisation (VO) and group management, entitlements, token handling, identity propagation, logging, non-web access, onboarding and migration from existing identity systems.

The document is intended as supporting implementation material for the [ECHOES Authentication and Authorisation documentation](../README.md).

## 1. VO and Group Management

### Question

What is the best strategy for managing multiple Virtual Organisations (VOs) under a single project umbrella, with different access policies and services per VO?

### Clarification

The ECHOES project involves a diverse consortium of cultural heritage institutions, each potentially operating its own workflows, datasets, and applications. Some may require full isolation, others semi-shared environments.

### Questions to clarify

- **Q:** Do we create one VO for ECHOES, or multiple VOs per domain, country, or work package?

**Answer:** Start with one VO, and use groups/subgroups to represent different domains, communities, or work packages for easier coordination and access control.

- **Q:** VO request has already been made, what’s the status?

**Answer:** Created, more generally you can submit a support ticket via EGI Helpdesk.

- **Q:** How do we enforce group-level access within a VO to specific services or datasets?

**Answer:** In VO, define groups/subgroups to manage specific services or datasets. Groups might be associated with some specific roles and granular access control.

- **Q:** Can users belong to multiple VOs or groups simultaneously, and how are entitlements handled in such cases?

**Answer:** Yes, users can belong to multiple VOs and groups; Check-in supports attribute aggregation, and entitlements are combined and scoped by VO/group.

- **Q:** How scalable is this VO management if dozens of partners and cascading grants are onboarded over time?

**Answer:** VOs support a hierarchical group structure, making it scalable to manage many partners, roles, and nested group permissions.

- **Q:** Is there a self-service or delegated admin model to let partners manage their own groups/users?

**Answer:** The VO manager is responsible for the operation of the VO, and could not find any delegated administration capabilities. In short, the admin of VO is responsible for managing (admin or group, or admins). Delegation is possible at the group level; group managers can manage their own members.

## 2. Custom Entitlements

### Question

Can we define and enforce custom entitlements or scopes within EGI Check-in, and are these reliably passed to service providers across all identity providers?

### Clarification

The EGI Check-in service supports the inclusion of entitlements—structured attribute strings embedded in the OIDC UserInfo response and the OAuth 2.0 Token Introspection responses—to represent group membership, roles, or access levels.

### Questions to clarify

- **Q:** Are entitlements customizable per VO or per user group within a VO?

**Answer:** Both. In Vo, defining groups and specific roles. Each role is associated with an entitlement

**Example 1:**

A user with the role "administrator" in the VO "vo.echoes.eu" receives the entitlement urn:mace:egi.eu:group:vo.echoes.eu:role=administrator#aai.egi.eu.

**Example 2:**

A user with the role "member" in the group "domain-x" of the VO "vo.echoes.eu" receives the entitlement urn:mace:egi.eu:group:vo.echoes.eu:domain-x:role=member#aai.egi.eu

- **Q:** Are there limits to the number of entitlements or scopes?

**Answer:** There are no limits. All roles defined within a group are automatically mapped to entitlements

- **Q:** What tools or APIs does EGI offer to programmatically assign and manage these entitlements?

**Answer:** Keycloak Group Management API and Perunweb tools

## 3. Identity Propagation Across Microservices

### Question

What is the recommended best practice to integrate EGI Check-in with distributed microservice-based systems, especially for secure identity propagation and token introspection?

### Clarification

ECHOES adopts a microservice and message-driven architecture, where services must interact seamlessly while respecting user identity, roles, and access constraints. When a user authenticates via EGI Check-in and receives a token (e.g., OIDC JWT), that identity must be securely propagated across internal services, often without user interaction.

### Questions to clarify

- **Q:** Can downstream services independently verify tokens without contacting the Check-in service?

**Answer:** rather standard approach with JWT validation, so it seems to be independent.

- **Q:** What is the recommended method for passing the token between services (in HTTP headers, message queues)?

**Answer:** Bearer Token

- **Q:** How should M2M communication authenticate (SA, user credentials)?

**Answer:** Client Credentials Grant flow see https://docs.egi.eu/providers/check-in/sp/#client-credentials

## 4. Token Lifecycle

### Question

What are the token expiration and revocation policies for EGI Check-in (access tokens, refresh tokens, ID tokens)? Is manual revocation or automatic session expiry supported?

The lifetimes of access tokens, refresh tokens and ID tokens are configurable per OIDC/OAuth client:

ID Token lifetime: Default: 10 min

Access Token: Default: 1 hour, Max 25 hours

Refresh Token: Default: 8 hours, Max 13 months

### Clarification

ECHOES integrates both interactive and long-running cloud services (e.g., data pipelines), where reliable and secure identity propagation is essential

### Questions to clarify

**Answer:** Most of the answer can be found in the OIDC integration documentation

- **Q:** Can token lifetimes be customized per client or service provider?

**Answer:** Yes, token lifetimes can be customized per client (i.e., per integrated service).

- **Q:** Are refresh tokens available to clients?

**Answer:** Yes, refresh tokens are supported in the OIDC authorization code flow (requires offline_access scope).

- **Q:** Are there rate limits or constraints on token refresh requests?

Answer There are no strict public rate limits, but behavior may be made dependent on specific configuration and policies.

- **Q:** Is a revocation endpoint supported as part of EGI Check-in?

**Answer:** Yes, EGI Check-in supports the OIDC token revocation endpoint, as per standard specifications.

- **Q:** Can users, VO admins, or service admins manually revoke active tokens?

**Answer:** Users can revoke their own sessions via the Check-in interface. VO admins cannot directly revoke tokens, but can disable users/groups, indirectly revoking access. Service admins can use OIDC introspection or session policies. Do users revoke themselves?

Are tokens revoked automatically if:

A user is removed from a VO? Yes, the next token refresh or re-authentication will reflect the removal; access is revoked accordingly.

An account is suspended or deleted? Yes, access is lost, and tokens become invalid on the next check.

Group membership or entitlement changes? Yes, new entitlements are reflected in newly issued tokens; existing tokens are not forcibly revoked but will expire or be refreshed.

**Answer:**

- **Q:** Does EGI Check-in support front-channel or back-channel logout?

**Answer:** Yes, EGI Check-in supports both front-channel and back-channel logout, following the OIDC session management and logout specifications.

- **Q:** Can logout be cascaded across all services where a token was used?

**Answer:** Partially: logout can be cascaded if the client services implement back-channel logout and are registered accordingly. Full propagation depends on service integration.

- **Q:** Is session expiry managed at the IdP or Check-in level?

**Answer:** Session expiry is managed at the Check-in level (via Keycloak), which acts as the central IdP. Upstream IdPs may have their own session timeouts, but Check-in controls session lifetime for federated access.

## 5. External IAM Integration

### Question

Is it possible to integrate external authorization frameworks (INDIGO IAM?) with EGI Check-in for enhanced policy management and group control?

### Clarification

ECHOES project operates within a complex, federated research environment where fine-grained access control and reusable policy models are crucial. While EGI Check-in provides a robust Identity Provider and federated login capabilities, the management of authorization policies (roles, group hierarchies, access rules) may need to be outsourced or extended through external Identity and Access Management (IAM) solutions.

Questions to Clarify:

- **Q:** Can an external IAM act as a Client or a Service Provider (SP) to EGI Check-in?

**Answer:** Yes, Indigo may become a middleware service between EGI and some internal apps

- **Q:** Can EGI Check-in be configured to consume entitlements and group memberships asserted by an external IAM?

**Answer:** Yes

- **Q:** Are external IAMs allowed to introspect or validate EGI tokens securely?

**Answer:** OAuth2 Token Introspection, or public key endpoints

- **Q:** Are there existing patterns for VO-wide entitlement delegation using an external IAM?

**Answer:** Yes. Check-in is connected to external IAM/VO group management system which can express VO/group information according to AARC/EOSC interoperability guideline AARC-G069 and can forward these entitlements to connected clients (pass-through model) or can map external entitlements to internal ones (either internal VO/group entitlements or resource capabilities according to AARC-G027)

- **Q:** Can ECHOES delegate VO/group administration to external IAMs while retaining identity federation through EGI Check-in?

**Answer:** Yes. Check-in can be connected to external IAM systems managing VO/group information. These external IAM systems MUST support expression of VO/group information according to AARC/EOSC interoperability guideline AARC-G069

## 6. Persistent User Identification

### Question

How does EGI Check-in manage persistent user identifiers across different IdPs? Can we establish a stable internal ID to track users even if their external ID changes?

### Clarification

Users may authenticate using a variety of institutional or community identity providers, such as universities, research institutions, or social accounts. These IdPs can differ in how they expose user identifiers, and some may not guarantee long-term stability (email addresses).

Questions to Clarify:

- **Q:** Is it globally unique and guaranteed stable over time, regardless of IdP?

**Answer:** EGI check-in generates a persistent, globally unique, and stable identifier for each user, regardless of the external IdP

- **Q:** Can users link multiple identities (e.g., from different IdPs) to a single EGI Check-in profile?

**Answer:** yes, social media, ORCID, etc

- **Q:** Is this linking done manually by the user or administratively by operators?

**Answer:** Both

- **Q:** What happens if a user’s upstream identifier changes (e.g., username, email)?

**Answer:** If the upstream identifier changes (e.g. SAML subject-id/eduPersonPrincipalName/Persistent NameID or OIDC sub), a new EGI persistent ID is generated. Email changes have no effect on the Check-in identifier.

- **Q:** Can authorization policies be preserved or automatically re-associated?

**Answer:** Yes, via Identity Linking

- **Q:** Which identifier should relying services (e.g., SEP, microservices) use for authorization decisions and persistent storage?

**Answer:** sub (subject) claim in the OIDC ID token and Access Token

- **Q:** Are there any known exceptions where IdP behavior breaks identifier continuity?

**Answer:**

When users change ldPs or lose access and create new unlinked EGI Check-in identity

When the IdPs change the user’s identifier

When the IdPs change their own entity identifier

- **Q:** Is the persistent user ID consistently available in logs and monitoring tools?

**Answer:** Yes, the unique user identifier available through the sub claim is also logged in access events for auditing

- **Q:** How does EGI Check-in support traceability and compliance with GDPR or institutional auditing policies?

**Answer:** Traceability: Check-in complies with the REFEDS Sirtfi Security framework

## 7. Monitoring and Logging

### Question

What monitoring, logging, and auditing features are provided by EGI Check-in for tracking login events, token use, and user access?

### Clarification

In a federated, multi-tenant infrastructure like ECHOES, effective monitoring and auditing mechanisms are essential to ensure security, track user behavior, and comply with institutional and regulatory requirements (institutional audits, EU project reporting).

This points directly to Task T6.6 of ECHOES, which focuses on infrastructure monitoring, traceability

### Questions to clarify

- **Q:** Does EGI Check-in log successful and failed login attempts?

Yes, both successful and failed login attempts are logged. These events are kept for 18 months.

- **Q:** Are logs timestamped and include source IdP, IP addresses, and user identifiers?

Yes, logs are timestamped and include IdP, IP address, and user identifiers (e.g., eduPersonPrincipalName, sub). These logs are kept for 18 months

- **Q:** Can we monitor token issuance and usage?

Token issuance is logged; token usage monitoring depends on the integrated service and its validation method.

- **Q:** Are token introspection or validation events recorded?

Token introspection events are logged by the introspection endpoint, but actual usage tracking must be implemented by services.

- **Q:** Are changes to group memberships, VO assignments, or custom entitlements logged?

Changes in VO/group memberships are logged

- **Q:** Can we track which admin or system triggered these changes?

Admin actions are logged with user identifiers, enabling traceability of who made changes.

- **Q:** Are logs accessible via a dashboard or API?

Logs are not accessible but the events are accessible to Check-in operators through Keycloak’s admin dashboard/API

- **Q:** How long are logs retained, and are they exportable (to ELK)?

Logs are retained for 18 months according to Check-in’s Privacy Policy

- **Q:** Can logs be filtered per VO or tenant for project-specific auditing?

No

- **Q:** Are logs anonymized or pseudonymized where appropriate?

Logs include pseudonymized identifiers (e.g., persistent IDs); access to identifiable data is restricted per GDPR.

Logs are removed after the retention period. The access related metrics are anonymised.

- **Q:** How does EGI Check-in support compliance with GDPR and other data protection requirements?

EGI Check-in adheres to GDPR, with data minimization, purpose limitation, and access controls.

- **Q:** Does EGI Check-in support real-time event feeds or alerts?

EGI Check-in supports only email alerts for VO/group membership changes sent to the VO/group administrators, with no native real-time event feeds or other alerts.

- **Q:** Can we integrate with external observability platforms (Prometheus)?

EGI Check-in, built on Keycloak, does not natively integrate with Prometheus, but extensions like keycloak-metrics-spi could enable it. Foreseen ELK integration will enhance auditing capabilities.

## 8. Non-Web Service Access

### Question

Can EGI Check-in be used to authenticate users for non-web services (e.g., SSH)? What tools or protocols are supported?

### Clarification

While many AAI solutions are tailored primarily for browser-based web applications, ECHOES requires federated authentication to extend to non-web use cases.

Questions to Clarify:

- **Q:** Can access/refresh tokens issued by Check-in be stored and reused securely in non-browser contexts (e.g., CLI tools)?

**Answer:** Yes, with tools like oidc-agent

- **Q:** Is offline access (long-lived refresh tokens) supported for scheduled/batch workloads?

**Answer:** Yes, Check-in supports offline_access for long-lived refresh tokens via OpenID Connect, suitable for user-authorised scheduled/batch workloads. Service accounts can use client credentials to obtain access tokens without refresh tokens for non-interactive jobs.

Are there known integrations or documented case studies where EGI Check-in has been used for:

SSH login to remote VMs or portals

- **Q:** Kubernetes-based platforms?

**Answer:**

For SSH: https://github.com/EOSC-synergy/ssh-oidc

- **Q:** Kubernetes: ???

- **Q:** Can user tokens carry the necessary VO/group context and entitlements for service-level authorization?

**Answer:**

Check-in enables service-level authorisation by providing VO/group entitlements (e.g., urn:mace:egi.eu:group:vo.example.org:role=member#aai.egi.eu) through its Token Introspection endpoint, following AARC interoperability guidelines and best practices for controlled attribute exposure, support for access token revocation, and reduced token size

- **Q:** Can the Check-in token endpoint be accessed via CLI tools (e.g., curl, oidc-agent) for programmatic login flows?

**Answer:** Yes, EGI Check-in’s token endpoint (/auth/realms/egi//protocol/openid-connect/token) supports CLI tools like curl and oidc-agent for programmatic login flows. curl can request tokens using flows like Client Credentials, while oidc-agent handles user-driven flows like Authorization Code or Device Authorization

- **Q:** What best practices are recommended to secure token storage, refresh handling, and revocation in CLI contexts?

**Answer:**

Token Storage: Store tokens in encrypted storage (e.g. oidc-agent)

Refresh Handling: Enable Refresh Token Rotation to issue new refresh tokens per request (note: tokens are automatically revoked if the same refresh token is reused). Use short-lived access tokens

Revocation: Use OAuth2 Token Introspection (/protocol/openid-connect/token/introspect) to validate access tokens (note: Check-in supports proxied introspection (AARC-G052) which supports validation of tokens issued by external trusted AAIs/token issuers). CLI tools can call the revocation endpoint (/protocol/openid-connect/revoke).

Audience Restriction: Consider audience-restricted tokens using the resource parameter (RFC 8707) to limit tokens to specific services

- **Q:** Are token scopes and audience restrictions enforced for specific services?

**Answer:** Check-in issues tokens with scopes (e.g. standard OIDC/OAuth2 scopes like openid, email, profile, entitlements or custom scopes), with allowed scopes configured per client via the EGI Federation Registry (https://aai.egi.eu/federation). It supports audience restrictions via OAuth2 Resource Indicators (RFC 8707), allowing clients to specify a specific audience. Services enforce these scopes and audience claims via JWT parsing or, preferably, via OAuth2 Token Introspection (RFC 7662), ensuring only authorized tokens are accepted.

## 9. Onboarding and Federation Testing

### Question

- **Q:** Does EGI provide a sandbox to simulate the onboarding of new partners and identity providers before production integration? Can onboarding be automated (for instance via APIs)?

### Clarification

The ECHOES project involves a dynamic, multi-partner environment, where institutions may join at different times, with varied identity management systems and internal governance. To ensure operational scalability and minimize onboarding friction, it is essential to test and validate integrations in a controlled environment before promoting them to production.

Questions to Clarify:

- **Q:** Is there a dedicated sandbox instance of the EGI Check-in service that mimics production behavior?

**Answer:** The EGI Check-in Demo environment (aai-demo.egi.eu) mirrors the production configuration, serving as a dedicated sandbox for testing service/client integrations. Integration requires a lightweight technical review, unlike production (aai.egi.eu), which mandates policy compliance, including the Privacy Policy and Acceptable Use Policy documents. A separate development environment (aai-dev.egi.eu) is available for testing experimental features of Check-in.

- **Q:** Can we onboard and test institutional IdPs (e.g., SAML-based academic providers, etc) within the sandbox before registering them in production?

**Answer:** Non-eduGAIN IdPs (SAML or OIDC) can be onboarded with a lightweight technical review in the demo environment (aai-demo.egi.eu). IdPs already in eduGAIN should be supported by Check-in, as it is registered as a Service Provider in eduGAIN, and typically require only attribute release verification. Check supported IdPs at:

Production: https://aai.egi.eu/auth/realms/id/account/#/personal-info

Demo: https://aai-demo.egi.eu/auth/realms/id/account/#/personal-info

Development: https://aai-dev.egi.eu/auth/realms/id/account/#/personal-info

- **Q:** Is metadata ingestion automatic, or does EGI need to manually validate/test each IdP integration?

**Answer:** EGI Check-in automatically ingests metadata for approx. 6.000 eduGAIN SAML IdPs via the eduGAIN metadata aggregate, refreshed every 4 hours. Non-eduGAIN SAML IdPs require manual submission of the SAML metadata URL and entityID, or can be ingested through SAML metadata aggregates, with refresh periods configurable from 1 hour to daily. OIDC IdPs are ingested via their .well-known/openid-configuration endpoint with configurable automatic refresh, and support for OpenID Federation is in progress for federated OIDC connections. All IdPs require testing in the Demo environment (aai-demo.egi.eu) with a lightweight technical review to validate attribute release and authentication.

- **Q:** Are there validation tools provided by EGI for IdP compliance (e.g., eduGAIN metadata or attribute mapping checks)?

**Answer:** EGI Check-in provides no dedicated validation tools for SAML IdP metadata compliance. The Check-in Operations team manually verifies metadata compliance when configuring the IdP metadata source. Verification of attribute release requires a test user with an account at the IdP.

- **Q:** Can Service Providers and Virtual Organisations be registered and configured within the sandbox for testing role/entitlement propagation?

**Answer:** Yes, it is recommended to register and configure Service Providers (SAML or OIDC) and Virtual Organisations in the EGI Check-in demo environment (aai-demo.egi.eu) for testing role/entitlement propagation. SPs are onboarded via the EGI Federation Registry (https://aai.egi.eu/federation) with a lightweight technical review. VOs are configured with the required roles entitlements, which can then be tested for propagation through the entitlements claim.

- **Q:** Can entitlements be mocked or simulated for test users?

**Answer:** No, entitlements cannot be mocked or simulated for test users. Test users must enroll with a Virtual Organisation in the EGI Check-in sandbox (aai-demo.egi.eu) using a supported institutional or social IdP to receive VO-assigned entitlements for testing.

- **Q:** Are there example configurations to speed up integration?

**Answer:**

For SAML-based services, refer to https://docs.egi.eu/providers/check-in/sp/#example-saml-service-provider-configurations

For OIDC-based services, refer to https://docs.egi.eu/providers/check-in/sp/#example-oidc-client-configurations

- **Q:** Does EGI provide a technical readiness checklist or guidelines for what a new partner must prepare before onboarding?

**Answer:** Refer to the Service Provider Integration documentation at https://docs.egi.eu/providers/check-in/sp/#service-provider-integration-workflow

## 10. Migration Paths from Existing Identity Systems

### Question

What approaches and tools does EGI Check-in provide to support migration from existing local identity systems to a federated model? Are there recommended strategies for maintaining service continuity during transition periods?

### Clarification

ECHOES includes partners with varying levels of identity infrastructure maturity. Some institutions already operate established authentication systems (e.g., local Keycloak, LDAP, AD, etc) with existing user bases, permissions, and integrated applications.

Questions to clarify

- **Q:** Are there best practices for transitioning services one-by-one rather than all at once?

Answer : EGI Check-in supports a hybrid setup where it brokers authentication via OIDC/SAML from local identity system (e.g., an existing Keycloak, or LDAP/AD configured as a SAML/OIDC IdP) and federated IdPs (e.g. institutional IdPs from eduGAIN and social IdPs). Services can be migrated incrementally as follows:

Connect the local IdP to EGI Check-in using OIDC/SAML (e.g. configure an existing Keycloak as an OIDC IdP, or LDAP/AD via SAML/OIDC if supported; a bridge, such as a local Keycloak, is only needed if the local identity system cannot act as an OIDC/SAML IdP).

Onboard one SP (SAML or OIDC) to EGI Check-in via the EGI Federation Registry (https://aai.egi.eu/federation), configuring it to trust EGI Check-in for authentication.

Map identifiers: EGI Check-in includes a stable user identifier attribute from the local IdP in tokens/assertions, allowing the SP to match EGI Check-in’s new user identifier (available through the sub claim) to the local IdP’s user identifier.

Test the SP in the demo environment (aai-demo.egi.eu) to verify user login via local or federated IdPs and ensure identifier mapping maintains user continuity.

Keep other SPs on the local IdP, which remains operational for non-migrated services. Repeat for each SP, ensuring gradual migration with no disruption. The local IdP can optionally be phased out if all SPs and users transition to Check-in’s federated IdPs.

- **Q:** What mechanisms exist to link pre-existing local accounts with new federated identities?

Answer : EGI Check-in supports account linking, allowing users to connect their local account (e.g. from LDAP/AD via SAML/OIDC, or Keycloak) to a federated identity (e.g. eduGAIN or social IdP) via the user profile management page (e.g. https://aai.egi.eu/auth/realms/id/account/#/security/linked-accounts for production). After authenticating via OIDC/SAML, users can link identities, creating a unified profile with a single user identifier (released to services through the sub claim).

- **Q:** Can user attributes and entitlements be bulk-imported or mapped from existing systems?

**Answer:** Users should first register with Check-in. Then Check-in's VO management API supports enrolling users to VOs/groups managed by Check-in. There is also a user attributes API to allow trusted clients to add attributes. Using an export of the local LDAP/AD system these APIs can be used to add/update attributes in bulk and assign group memberships and roles.

- **Q:** How are conflicts or duplicate identities handled during synchronization?

**Answer:** Conflicts are resolved via account linking and by using persistent unique identifiers (e.g., eduPersonUniqueId, sub) to unify identities.

- **Q:** Are there recommended proxy configurations or gateway patterns (e.g., using Keycloak as a broker) for incremental migration?

**Answer:** See response to first question about incremental migration

- **Q:** Do tools exist for testing identity continuity before and after migration?

**Answer:** The EGI Check-in demo environment (aai-demo.egi.eu) can be used to test authentication flows, local IdP integration, and attribute propagation. Test users authenticate via local IdP or federated IdPs (eduGAIN, social) using OIDC/SAML, verifying identity continuity by checking user attributes.

- **Q:** What rollback strategies are recommended if migration encounters issues?

**Answer:** Maintain dual authentication paths during migration, allowing SPs to authenticate via either EGI Check-in or the local identity system. Back up SP configurations and local IdP data before onboarding to EGI Check-in. If issues arise, reconfigure the SP to use the local IdP directly, restoring access via local identifiers.

- **Q:** Can identity federation be temporarily disabled for specific services if problems arise?

**Answer:** Yes, individual SPs (SAML or OIDC) can disable EGI Check-in authentication by updating their configuration to revert to the local IdP. This change is isolated to the specific SP, leaving other services unaffected.

- **Q:** What documentation or resources does EGI provide to help communicate changes to end users?

**Answer:** EGI provides official documentation, onboarding guides, and helpdesk templates at docs.egi.eu.
