# ECHOES AAI Architecture

*AARC BPA-compliant approach aligned with EOSC AAI Architecture 2025*

## Executive Summary

This document describes the proposed Authentication and Authorisation Infrastructure (AAI) architecture for ECHOES in the context of the European Collaborative Cloud for Cultural Heritage.

The objective is to support secure and interoperable access to ECHOES services, sister-project services and protected APIs without creating a new project-specific identity silo. The approach is based on the [AARC Blueprint Architecture](https://aarc-community.org/architecture/) and follows the relevant technical principles of the [EOSC AAI Architecture 2025](https://zenodo.org/records/15388270): integration through AARC-compliant Infrastructure Proxies, OpenID Connect and OAuth 2.0 as the primary protocol family, secure OAuth/OIDC flows, token validation through introspection, and a harmonised claims profile for identity and authorisation information.

EGI Check-in is positioned as the AAI layer supporting the ECHOES single entry point. The single entry point represents the user-facing access path to the ECHOES / ECCCH cloud environment, while EGI Check-in provides the federated identity, authentication, group and role management, token validation and interoperability functions required to operate that access path securely.

Sister projects may integrate directly with the ECHOES AAI as services, or they may interoperate through their own AARC BPA-compliant Infrastructure Proxy and/or Community AAI. This preserves the autonomy of sister projects while enabling cross-project access and collaboration. The detailed sister-project integration options are described in [Sister-project AAI integration](sister-project-aai-integration.md). The long-term target is to support a scalable federation model based on OpenID Federation, in line with the future direction of the EOSC AAI Architecture 2025.

## 1. Purpose and scope

ECHOES aims to support a shared digital environment for Cultural Heritage Cloud services, tools, data and workflows. This requires a common approach to user authentication, service access, collaboration management and cross-project interoperability.

This document defines the proposed AAI architecture and integration approach for ECHOES. It is intended to guide technical discussions, service onboarding and future alignment with the wider EOSC AAI Federation.

The scope of the document is to describe:

- the role of EGI Check-in in the ECHOES AAI model;

- the AAI integration principles adopted by ECHOES;

- the technical requirements for ECHOES service integrations;

- the baseline claims and attribute profile;

- the security and privacy requirements;

- the proposed roadmap towards OpenID Federation-based interoperability.

The document does not centralise all authorisation decisions in ECHOES. Each service and sister project remains responsible for its own domain-specific access policies. The role of the ECHOES AAI is to provide trusted identity, harmonised claims, high-level community information and interoperable access mechanisms.

## 2. Relationship with the ECHOES Single Entry Point

ECHOES foresees a single entry point to the cloud environment. This single entry point should be understood as the user-facing access path to the ECHOES / ECCCH environment, not as the AAI system itself.

The AAI layer supports the single entry point by enabling users to authenticate with trusted credentials and by providing the identity and authorisation information required by connected services. In the ECHOES technical implementation, this AAI capability is provided through EGI Check-in.

This distinction is important:

- the single entry point is the user-facing access path to the ECHOES cloud environment;

- EGI Check-in is the AAI layer that supports authentication, identity linking, group and role management, service registration, token validation and interoperability;

- ECHOES services and sister-project services consume AAI information through standard interfaces, primarily OpenID Connect and OAuth 2.0;

- sister projects that already operate their own AAI may interoperate through AARC-compliant proxy interfaces rather than replacing their existing AAI.

This approach avoids presenting ECHOES as a central identity silo. Instead, ECHOES provides a federated AAI integration model that can support both simple service-level login and more advanced AAI-to-AAI interoperability.

## 3. Role of EGI Check-in

EGI Check-in is the proposed AAI implementation supporting the ECHOES single entry point. It provides an AARC BPA-compliant identity and access management capability for users, services and communities.

For ECHOES, EGI Check-in provides:

- federated authentication through institutional identity providers from eduGAIN, research identity providers and other supported identity sources;

- persistent user identification and identity linking;

- OpenID Connect, OAuth 2.0 and SAML 2.0 interfaces;

- service registration and lifecycle management;

- community, group and role management;

- release of harmonised identity and authorisation claims;

- token-based access to protected services and APIs;

- token introspection through its Infrastructure Proxy, including support for cross-AAI token validation in line with [AARC-G052 Proxied Token Introspection](https://aarc-community.org/guidelines/aarc-g052/);

- an integration path towards OpenID Federation.

EGI Check-in can support both direct service integration and federation with other AAI systems. This is important for sister projects, because not all sister projects will have the same technical maturity. Some may only need to connect a web application as an OpenID Connect client. Others may already operate their own AARC-compliant Community AAI and/or Infrastructure Proxy and should be able to interoperate at the AAI layer.

Guidance for services integrating directly with EGI Check-in is provided in the [EGI Check-in Service Provider integration guide](https://docs.egi.eu/providers/check-in/sp/).

## 4. AAI integration principles adopted by ECHOES

ECHOES adopts the technical integration principles of the EOSC AAI Architecture 2025. The focus is on practical interoperability through AARC BPA-compliant components and standard protocols.

### 4.1 Integration through Infrastructure Proxies

Services should not establish ad hoc trust relationships with every external identity source or sister-project AAI. Instead, services should integrate through their AARC BPA-compliant Infrastructure Proxy, which acts as the trusted interface for authentication, token validation, attribute release and access to protected services.

This keeps the model scalable and avoids the need for each service to manage separate trust relationships with multiple external AAIs.

### 4.2 OpenID Connect and OAuth 2.0 as the primary protocol family

OpenID Connect and OAuth 2.0 should be used as the primary protocols for web-based login, API access and token-based authorisation.

SAML 2.0 may be supported where required for compatibility with legacy services. However, new ECHOES and sister-project integrations should prefer OpenID Connect and OAuth 2.0.

### 4.3 Secure OAuth/OIDC flows

Services using OpenID Connect should use the Authorisation Code flow. Public clients and browser-based applications should use Proof Key for Code Exchange (PKCE). The nonce parameter should be used to protect authentication responses.

Insecure flows, in particular the OAuth 2.0 Implicit Grant, should not be used.

Redirect URIs and post-logout redirect URIs must be pre-registered as complete HTTPS URIs and must be matched exactly. Wildcard redirect URIs, partial matching, dynamically supplied redirect targets, and open redirectors must not be used.

### 4.4 Token validation through the trusted Infrastructure Proxy

Protected APIs and resource servers should validate access tokens using token introspection through their trusted Infrastructure Proxy.

From the perspective of the protected API, this remains a standard [RFC 7662 OAuth 2.0 Token introspection](https://datatracker.ietf.org/doc/html/rfc7662) interaction. The API treats the access token as opaque and calls the introspection endpoint of its trusted Infrastructure Proxy. The API should not be required to parse the token payload, validate self-contained access tokens locally, or integrate directly with every external token issuer.

Where the token was issued by an external trusted AAI, the AARC BPA-compliant Infrastructure Proxy should support [AARC-G052 Proxied Token Introspection](https://aarc-community.org/guidelines/aarc-g052/). This is a capability of the Infrastructure Proxy, not a separate protocol requirement for each protected API.

This model supports token revocation, reduces the need to embed personal information in access token payloads, and allows the Infrastructure Proxy to disclose only the token metadata needed by each protected resource.

### 4.5 Common claims and attribute profile

ECHOES and sister-project services should rely on a harmonised set of claims for identity, contact, affiliation, assurance and authorisation information.

The baseline claim set should include:

- persistent user identifiers;

- name and display information;

- email address;

- organisational affiliation;

- identity assurance information;

- community group and role information.

Group membership and role information should be expressed through the entitlements claim, following the URN syntax defined in [AARC-G069](https://aarc-community.org/guidelines/aarc-g069/), so that services can consume authorisation information in a consistent way.

### 4.6 Separation of federated identity from local authorisation

The ECHOES AAI should provide trusted identity, harmonised claims and high-level group or role information. Domain-specific authorisation decisions remain under the responsibility of the relevant service or sister project.

For example, the ECHOES AAI may assert that a user belongs to a given collaboration group or has a specific role. The service itself decides whether that role allows the user to edit a specific dataset, manage a collection, publish content, run a workflow or access restricted material.

## 5. Sister-project interoperability

Sister projects may integrate with ECHOES in different ways, depending on their technical maturity and existing AAI capabilities.

A sister-project service that does not operate its own AAI may integrate directly with the ECHOES AAI as an OpenID Connect client. A sister project that already operates its own AARC BPA-compliant Community AAI and/or Infrastructure Proxy should not be forced to replace its existing AAI. Instead, ECHOES and the sister project should establish interoperability through AARC BPA-compliant interfaces, harmonised claims, token validation mechanisms and agreed trust metadata.

The detailed sister-project integration options and cross-project access patterns are described in [Sister-project AAI integration](sister-project-aai-integration.md).

## 6. Technical requirements for ECHOES service integrations

ECHOES services and APIs that integrate with the ECHOES AAI should satisfy the following technical requirements. The same baseline also applies to sister-project integrations, as described in [Sister-project AAI integration](sister-project-aai-integration.md).

### 6.1 Protocol requirements

1. Services should support OpenID Connect and OAuth 2.0 for authentication and authorisation.

2. SAML 2.0 may be supported only for existing or legacy services that cannot support OpenID Connect or OAuth 2.0.

### 6.2 OAuth/OIDC security requirements

1. Services should use the OpenID Connect Authorisation Code flow.

2. Browser-based applications should use [PKCE](https://datatracker.ietf.org/doc/html/rfc7636); public clients in particular, are required to use PKCE.

3. The `nonce` parameter should be used to protect authentication responses.

4. The OAuth 2.0 Implicit Grant should not be used.

5. Redirect URIs and post-logout redirect URIs must be registered explicitly.

6. Wildcard redirect URIs must be avoided.

7. Services should request only the scopes and claims required for their operation.

8. Client secrets and credentials must be protected according to the client type and deployment model.

### 6.3 Token validation requirements

1. Protected APIs and resource services should validate access tokens before granting access.

2. OAuth 2.0 Token Introspection through the trusted Infrastructure Proxy should be the recommended token validation mechanism.

3. Protected APIs should treat access tokens as opaque and should not be required to parse or validate token payloads directly.

4. The Infrastructure Proxy should support AARC-G052 Proxied Token Introspection for tokens issued by external trusted AAIs.

5. From the API perspective, the interaction remains standard OAuth 2.0 Token Introspection; the AARC-G052 behaviour is handled by the Infrastructure Proxy.

6. Introspection responses should expose only the information required by the protected API. Personal data and authorisation metadata should be minimised and released only where required for the service.

7. APIs should check the token status, audience where available, expiry, scopes and relevant authorisation information returned by the trusted Infrastructure Proxy before granting access.

8. Services should not accept tokens that cannot be validated through the trusted Infrastructure Proxy.

### 6.4 Group and role requirements

1. Community membership, project membership, group membership and role information should be expressed using AARC-G069-compatible URN values.

2. Such values should be released through the entitlements claim.

3. Services should avoid relying on undocumented project-specific claim names where standard claims can be used.

4. Local services may map AARC-G069 entitlements to their internal roles and permissions.

5. Sister projects that manage their own groups and roles should remain authoritative for those groups and roles.

### 6.5 Operational requirements

1. Each integrated service should have an identified service owner.

2. Each integrated service should provide administrative, technical, support and security contacts.

3. Services should be tested in a non-production environment before production onboarding.

4. Services should provide a privacy notice describing the personal data processed and the purpose of processing.

5. Services should support incident-response handling consistent with the Sirtfi security framework.

6. Service configuration should be reviewed when redirect URIs, claim requirements, client type or ownership changes.

## 7. Claims and attribute profile

ECHOES and sister-project services should rely on a minimal and harmonised claim set. The purpose is to support interoperability while applying data minimisation.

The following baseline claims are recommended.

| Type of information | Technical claim | Purpose |
| --- | --- | --- |
| Persistent user identifier | `voperson_id` | Stable user identification across services |
| Name | `name`, `given_name`, `family_name` | Display name and user-facing workflows |
| Email address | `email` | Contact and notification |
| Organisational affiliation | `voperson_external_affiliation` | User affiliation in their home organisation |
| Home organisation | `schac_home_organization` | User’s home organisation using the domain name of the organisation |
| Assurance | `eduperson_assurance` | Identity assurance information |
| Groups and roles | `entitlements` | Community membership, roles and access rights |

Services should request only the claims they need. For example, a service that only needs login and contact may not need group information. A service that applies group/role-based access control will need `entitlements`.

`voperson_id` is the recommended claim for stable user identification. For some AAI implementations, e.g. EGI Check-in, `sub` carries the same value as `voperson_id`, in which case it may also be used for this purpose. **Email addresses must not be used as persistent user identifiers.**

The entitlements claim should be used for group and role information according to AARC-G069. This enables consistent interpretation of community roles across ECHOES and sister-project services.

## 8. Authorisation model

The ECHOES AAI should provide high-level identity and authorisation information. It should not centralise all authorisation decisions across the Cultural Heritage Cloud ecosystem.

The recommended model is:

- the AAI authenticates the user;

- the AAI releases harmonised identity claims;

- the AAI releases ECHOES or sister-project group and role information where applicable;

- APIs validate access tokens through the trusted Infrastructure Proxy;

- the service evaluates the claims or introspection response against its local access policies;

- the service remains authoritative for domain-specific permissions.

This means that ECHOES AAI can support role-based or attribute-based access control, but the final decision remains with the protected service unless a specific central decision point is introduced for a clearly defined purpose.

## 9. Security and privacy baseline

All ECHOES integrations, and sister-project integrations where applicable, should align with relevant community baselines such as the [Sirtfi](https://refeds.org/sirtfi) security framework, the [AARC-G084 Security Operational Baseline](https://aarc-community.org/guidelines/aarc-g084/) and the [REFEDS Data Protection Code of Conduct](https://wiki.refeds.org/display/CODE/Data+Protection+Code+of+Conduct+Home), or an equivalent GDPR-aligned framework.

At minimum, services should:

- use secure OAuth/OIDC flows;

- avoid insecure or deprecated flows;

- validate redirect URIs strictly;

- request only necessary claims;

- protect client credentials and tokens;

- use HTTPS for all interactions;

- validate tokens through the trusted Infrastructure Proxy before granting API access;

- minimise personal data in access token payloads;

- use introspection responses to disclose only the metadata required by the protected resource;

- apply appropriate token lifetimes aligned with AARC-G083 recommendations;

- maintain audit logs for security-relevant events;

- provide operational and security contacts;

- support security incident response;

- comply with GDPR-aligned data protection requirements;

- provide clear privacy information to users.

## 10. Onboarding process

A lightweight onboarding process should be established for ECHOES and sister-project integrations.

For services integrating directly with EGI Check-in, the EGI Check-in Service Provider integration guide should be used as the reference for registration, configuration and testing. The guide describes the service registration workflow through the EGI Federation Registry, the use of non-production environments for testing, the information required from service providers, and the protocol-specific configuration for OpenID Connect and SAML integrations.

For all integration types, the onboarding process should include the following steps.

### Step 1: Identify the integration type

The service or sister project should identify whether the integration is:

- direct service-level integration;

- Infrastructure Proxy integration;

- Community AAI integration;

- combined Community AAI and Infrastructure Proxy integration;

- API or machine-to-machine integration.

### Step 2: Collect technical metadata

The following information should be collected:

- service name;

- service description;

- service URL;

- service logo URL;

- service owner;

- contact information of the following types:

  - administrative;

  - technical;

  - support/helpdesk;

  - security

- redirect URIs where applicable;

- post-logout redirect URIs, where applicable;

- requested scopes;

- client type;

- API audience requirements, where applicable;

- token introspection requirements, where applicable;

- group or role requirements, where applicable.

### Step 3: Review security and privacy requirements

The integration should be reviewed to confirm:

- use of secure OAuth/OIDC flows;

- correct redirect URI configuration, where applicable;

- data minimisation;

- appropriate scope/claim requests;

- privacy notice availability;

- operational and security contacts;

- token introspection approach for APIs;

- group and role requirements, where applicable.

### Step 4: Configure and test in a non-production environment

The service should first be configured and tested in a non-production environment. Testing should cover:

- login;

- logout;

- claim release;

- group and role mapping, where applicable;

- token introspection, where applicable;

- error handling;

- user experience.

### Step 5: Approve production onboarding

Production onboarding should proceed once the technical checks are complete.

Approval should confirm that:

- the service is eligible for connection;

- the responsible organisation is known;

- the technical configuration is correct;

- the requested claims are justified;

- the service has appropriate contacts;

- the service has passed testing.

### Step 6: Maintain and review

Service integrations should be reviewed periodically or when significant changes occur, such as:

- change of service owner;

- change of redirect URIs;

- change of requested claims;

- change of client type;

- introduction of API access;

- introduction of group or role-based access;

- security incident;

- service decommissioning.

## 11. Roadmap towards OpenID Federation

The long-term goal is to support scalable federation using [OpenID Federation](https://openid.net/specs/openid-federation-1_0.html). This is aligned with the future direction described in the EOSC AAI Architecture 2025. OpenID Federation is expected to improve how trust metadata is published, discovered and validated between AAIs. It can reduce the need for manual and bilateral configuration, but it does not replace the underlying requirements for secure protocols, harmonised claims, group and role expression, or token validation.

Specifically, OpenID Federation can support:

- dynamic discovery of federation participants;

- publication and validation of trust metadata;

- more scalable onboarding of AAI components;

- more automated trust establishment;

- reduced need for manual bilateral configuration;

- more flexible interoperation between ECHOES and sister-project AAIs.

Until OpenID Federation is operationally mature across the AAI ecosystem, ECHOES should use existing onboarding and registration processes to establish trust between services, Infrastructure Proxies and Community AAIs. This includes registering services and AAI components, reviewing their technical metadata, agreeing the required claims and scopes, and configuring the necessary trust relationships.

As OpenID Federation becomes available, ECHOES should adopt it to automate and scale trust establishment between AAIs. The existing baseline requirements remain valid: integrations should continue to use OpenID Connect and OAuth 2.0 as the primary protocol family, secure OAuth/OIDC flows, AARC-G069-compatible group and role expression, OAuth 2.0 Token Introspection through the trusted Infrastructure Proxy for protected APIs, and AARC-G052 support at the Infrastructure Proxy for tokens issued by external trusted AAIs.

This allows ECHOES to support near-term service integration while remaining compatible with the longer-term federation model.

## 12. Summary of requirements

The following requirements summarise the proposed ECHOES AAI integration profile.

| Area | Requirement |
| --- | --- |
| Architecture | Use AARC BPA-compliant AAI components |
| Service integration | Prefer integration through an Infrastructure Proxy |
| Protocols | Use OpenID Connect and OAuth 2.0 as the primary protocol family |
| Legacy support | Support SAML 2.0 where needed for compatibility |
| OAuth/OIDC flows | Use Authorisation Code flow; use PKCE where applicable |
| Insecure flows | Do not use the Implicit Grant |
| Redirect URIs | Register and validate redirect URIs strictly |
| Claims | Use a harmonised baseline claims profile |
| Groups and roles | Express group and role information using AARC-G069-compatible URN-formatted entitlements |
| API token validation | Protected APIs should use OAuth 2.0 Token Introspection through their trusted Infrastructure Proxy |
| Cross-AAI token validation | The Infrastructure Proxy should support AARC-G052 Proxied Token Introspection for tokens issued by external trusted AAIs |
| API responsibility | APIs should treat access tokens as opaque and consume the introspection response |
| Privacy | Access token payloads should minimise personal data; required metadata should be disclosed through controlled introspection responses |
| Security | Maintain operational and security contacts and support incident response |
| Federation evolution | Prepare for OpenID Federation-based trust establishment |

## 13. Conclusion

The proposed ECHOES AAI architecture enables interoperable access to ECHOES services, sister-project services and protected APIs while avoiding the creation of a new identity silo.

EGI Check-in provides the AAI implementation supporting the ECHOES single entry point. It offers federated authentication, identity linking, group and role management, service onboarding, token introspection and interoperability with other AAI systems.

Sister projects may integrate directly with EGI Check-in as services, or they may interoperate through their own AARC BPA-compliant Infrastructure Proxy and/or Community AAI. This sister-project integration model is described in [Sister-project AAI integration](sister-project-aai-integration.md).

The model is aligned with the technical principles of the EOSC AAI Architecture 2025: integration through Infrastructure Proxies, OpenID Connect and OAuth 2.0 as the primary protocol family, secure OAuth/OIDC flows, harmonised claims, token introspection and a long-term path towards OpenID Federation.

This provides a practical and future-compatible basis for ECHOES AAI integration with sister projects and the wider Cultural Heritage Cloud ecosystem.
