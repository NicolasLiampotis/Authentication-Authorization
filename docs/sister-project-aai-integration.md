# Sister-project AAI Integration

*Integration patterns for ECHOES sister projects and AARC BPA-compliant AAIs*

## Executive Summary

This document describes the proposed Authentication and Authorisation Infrastructure (AAI) approach for integrating ECHOES with sister projects in the context of the European Collaborative Cloud for Cultural Heritage.

The objective is to support secure and interoperable access to ECHOES and sister-project services without creating a new project-specific identity silo. Sister projects may integrate directly with the ECHOES AAI as services, or they may interoperate through their own AARC BPA-compliant Infrastructure Proxy and/or Community AAI. This preserves the autonomy of sister projects while enabling cross-project access and collaboration.

For the general ECHOES AAI model, claims profile, token validation approach, security baseline, onboarding process and roadmap towards OpenID Federation, see [ECHOES AAI Architecture](echoes-aai-architecture.md).

## 1. Purpose and scope

This document defines the proposed AAI integration approach for ECHOES sister projects. It is intended to guide technical discussions, service onboarding and future alignment with the wider EOSC AAI Federation.

The scope of the document is to describe:

- how sister projects may connect to ECHOES using their own AARC-compliant Infrastructure Proxy and/or Community AAI;
- the main sister-project integration options;
- the ECHOES AAI integration patterns that are relevant for sister-project services and APIs;
- the baseline requirements that sister-project integrations should follow.

The document does not centralise all authorisation decisions in ECHOES. Each sister project and service remains responsible for its own domain-specific access policies. The role of the ECHOES AAI is to provide trusted identity, harmonised claims, high-level community information and interoperable access mechanisms.

## 2. Relationship with the ECHOES AAI Architecture

The ECHOES AAI layer supports the single entry point by enabling users to authenticate with trusted credentials and by providing the identity and authorisation information required by connected services.

EGI Check-in is the proposed AAI implementation supporting the ECHOES single entry point. It can support both direct service integration and federation with other AAI systems. This is important for sister projects, because not all sister projects will have the same technical maturity. Some may only need to connect a web application as an OpenID Connect client. Others may already operate their own AARC-compliant Community AAI and/or Infrastructure Proxy and should be able to interoperate at the AAI layer.

The general architecture principles are described in [ECHOES AAI Architecture](echoes-aai-architecture.md).

## 3. Sister-project integration models

Sister projects may integrate with ECHOES in different ways, depending on their technical maturity, existing AAI capabilities and the type of services they operate.

The ECHOES AAI layer is provided through EGI Check-in. Sister projects may therefore either integrate services directly with EGI Check-in, or interoperate with ECHOES through their own AARC BPA-compliant AAI components.

### 3.1 Direct service-level integration with EGI Check-in

A sister-project service that does not operate its own AAI may integrate directly with the ECHOES AAI layer, provided through EGI Check-in, as an OpenID Connect client.

This is suitable for portals, dashboards, web applications, collaborative tools, repositories or other services that need federated login and a standard set of user claims, but do not need to operate their own Infrastructure Proxy or Community AAI.

In this model:

1. the service is registered with EGI Check-in;
2. the user accesses the sister-project service;
3. the service redirects the user to EGI Check-in for authentication;
4. the user authenticates using a trusted identity provider supported by EGI Check-in;
5. EGI Check-in releases the agreed claims to the service;
6. the service uses those claims for local account creation, login, role mapping or access control.

Local service administrators remain responsible for service-specific role mapping and permissions.

This is the simplest integration model and should be the default for sister-project services that need SSO but do not need to operate their own AAI proxy.

### 3.2 Infrastructure Proxy integration

A sister project that exposes several services, APIs, data platforms or protected resources may interoperate with ECHOES through its own AARC BPA-compliant Infrastructure Proxy.

This allows the sister project to keep control of its service-facing access model while supporting interoperability with ECHOES. The sister-project services remain connected to their own Infrastructure Proxy, while the Infrastructure Proxy establishes the required trust relationship with the ECHOES AAI layer.

This model is suitable when the sister project:

- operates multiple services or protected resources;
- wants a single AAI integration layer in front of its services;
- needs to validate access tokens for APIs or machine-to-machine access;
- wants to avoid connecting each individual service directly to EGI Check-in;
- already operates an AARC BPA-compliant Infrastructure Proxy.

In this model:

1. the user accesses a sister-project service or API;
2. the service relies on the sister project’s Infrastructure Proxy for authentication, claim release and token validation;
3. the sister-project Infrastructure Proxy interoperates with the ECHOES AAI layer where cross-project access is required;
4. the service enforces its local access policies based on the claims, entitlements or introspection response provided through its trusted Infrastructure Proxy.

This model preserves the autonomy of the sister project while enabling cross-project access.

### 3.3 Community AAI integration

A sister project that manages its own user community, groups, roles or collaboration policies may interoperate with ECHOES through an AARC BPA-compliant Community AAI.

This allows community-specific membership and role information to remain under the responsibility of the sister project, while still being usable in cross-project access scenarios.

This model is suitable when the sister project:

- manages its own users, groups or roles;
- has domain-specific membership or access policies;
- needs to remain authoritative for its community information;
- already operates a Community AAI or Collaboration Management service.

In this model:

1. the sister-project Community AAI remains authoritative for its own users, groups and roles;
2. the relevant group and role information is expressed through harmonised claims, using [AARC-G069](https://aarc-community.org/guidelines/aarc-g069/)-compatible entitlements;
3. the ECHOES AAI layer and the sister-project Community AAI exchange or consume agreed identity and authorisation information according to the established trust relationship;
4. services use the resulting claims or entitlements for local access-control decisions.

This model is appropriate where the main interoperability need is community, group or role information, rather than direct service-facing access to protected resources.

### 3.4 Combined Community AAI and Infrastructure Proxy integration

More mature sister projects may operate both:

- a Community AAI for users, groups, roles and community policies;
- an Infrastructure Proxy for access to services, APIs, repositories or other protected resources.

This is the preferred model for sister projects that need both community management and protected resource access.

In this model:

1. the sister-project Community AAI manages the project’s users, groups, roles and community policies;
2. the sister-project Infrastructure Proxy provides the service-facing AAI layer for the project’s services and APIs;
3. ECHOES and the sister project establish interoperability through AARC BPA-compliant interfaces, harmonised claims, token validation mechanisms and agreed trust metadata;
4. each service remains responsible for enforcing its own domain-specific access policies.

In such cases, ECHOES should not replace the sister project’s AAI. Instead, ECHOES and the sister project should interoperate through agreed AAI interfaces and trust relationships.

This model preserves local autonomy while enabling cross-project collaboration.

### 3.5 Cross-project API access

This model applies to APIs, repositories, workflow systems or computational services that require token-based access across ECHOES and sister-project environments.

Users authenticate through a trusted AAI and obtain access tokens. Protected APIs validate these tokens by calling the OAuth 2.0 Token Introspection endpoint of their trusted Infrastructure Proxy.

Where the token is issued by an external trusted AAI, the Infrastructure Proxy uses [AARC-G052 Proxied Token Introspection](https://aarc-community.org/guidelines/aarc-g052/) to validate the token. From the protected API’s perspective, the interaction remains standard OAuth 2.0 Token Introspection.

In this model:

1. the user or client obtains an access token from a trusted AAI;
2. the client presents the access token to the protected API;
3. the protected API sends the token to the introspection endpoint of its trusted Infrastructure Proxy;
4. the Infrastructure Proxy validates the token, including by using proxied introspection where required;
5. the protected API receives the introspection response and applies its local access policies.

This allows ECHOES and sister projects to support cross-project workflows without requiring every API to integrate directly with every possible token issuer.

## 4. Baseline requirements for sister-project integrations

Sister-project integrations should follow the same baseline profile described in [ECHOES AAI Architecture](echoes-aai-architecture.md), in particular:

- use OpenID Connect and OAuth 2.0 as the primary protocol family;
- support SAML 2.0 only where needed for compatibility with existing or legacy services;
- use secure OAuth/OIDC flows, in particular the Authorisation Code flow and [PKCE](https://datatracker.ietf.org/doc/html/rfc7636) where applicable;
- avoid the OAuth 2.0 Implicit Grant;
- register redirect URIs and post-logout redirect URIs explicitly;
- request only the scopes and claims required for the service;
- express community membership, group membership and role information using [AARC-G069](https://aarc-community.org/guidelines/aarc-g069/)-compatible URN-formatted entitlements;
- validate access tokens through [RFC 7662 OAuth 2.0 Token Introspection](https://datatracker.ietf.org/doc/html/rfc7662) via the trusted Infrastructure Proxy;
- support [AARC-G052 Proxied Token Introspection](https://aarc-community.org/guidelines/aarc-g052/) at the Infrastructure Proxy for tokens issued by external trusted AAIs;
- provide administrative, technical, support and security contacts;
- support incident-response handling consistent with the Sirtfi security framework;
- provide a privacy notice describing the personal data processed and the purpose of processing.

Local services may map [AARC-G069](https://aarc-community.org/guidelines/aarc-g069/) entitlements to their internal roles and permissions. Sister projects that manage their own groups and roles should remain authoritative for those groups and roles.

## 5. Onboarding and testing

A lightweight onboarding process should be established for ECHOES and sister-project integrations.

For services integrating directly with EGI Check-in, the [EGI Check-in Service Provider integration guide](https://docs.egi.eu/providers/check-in/sp/) should be used as the reference for registration, configuration and testing. The guide describes the service registration workflow through the [EGI Federation Registry](https://aai.egi.eu/federation), the use of non-production environments for testing, the information required from service providers, and the protocol-specific configuration for OpenID Connect and SAML integrations.

For all integration types, onboarding should identify the integration type, collect technical metadata, review security and privacy requirements, configure and test the integration in a non-production environment, approve production onboarding, and maintain/review the integration over time. The complete onboarding process is described in [ECHOES AAI Architecture](echoes-aai-architecture.md#10-onboarding-process).

## 6. Roadmap towards OpenID Federation

The long-term goal is to support scalable federation using [OpenID Federation](https://openid.net/specs/openid-federation-1_0.html). OpenID Federation is expected to improve how trust metadata is published, discovered and validated between AAIs. It can reduce the need for manual and bilateral configuration, but it does not replace the underlying requirements for secure protocols, harmonised claims, group and role expression, or token validation.

Until OpenID Federation is operationally mature across the AAI ecosystem, ECHOES should use existing onboarding and registration processes to establish trust between services, Infrastructure Proxies and Community AAIs. As OpenID Federation becomes available, ECHOES should adopt it to automate and scale trust establishment between AAIs.
