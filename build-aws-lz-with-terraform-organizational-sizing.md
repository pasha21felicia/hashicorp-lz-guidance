---
id: [unique-id]
name: AWS Landing Zones with Terraform: Organizational Sizing and Implementation Patterns
short_name: AWS Landing Zone Organizational Patterns
description: Learn how to choose and implement the right AWS landing zone architecture based on your organization's size, team structure, and maturity level using HCP Terraform.
default_collection_context: validated-patterns/terraform
products_used:
  - terraform
---

**Author:** Parascovia Digori

This guide builds upon the foundational concepts established in [Use Terraform to build an AWS landing zone](/terraform/tutorials/validated-patterns/build-aws-lz-with-terraform) to focus specifically on organizational right-sizing and implementation patterns. While the foundational guide demonstrates enterprise-scale architecture with comprehensive Platform Landing Zones (PLZs) and Workload Landing Zones (WLZs), organizations at different stages of growth face unique challenges in determining the right level of complexity for their current state.

## Scope

We examine three reference implementations representing small (1-3 teams), medium (4-10 teams), and large (10+ teams) organizations, demonstrating proven patterns you can adapt to your specific context and growth trajectory.

We'll dive into practical strategies for workspace segregation, project hierarchy design, and repository layout that enable both team-level agility and enterprise-wide governance. Through practical examples using FinTechCorp—a fictional company at different growth stages—you'll discover how to implement landing zone patterns that enable team autonomy while maintaining security, compliance, and infrastructure reusability across your organization.

## Target Audience

This guide addresses companies at different scales of cloud adoption, each facing distinct challenges in implementing AWS landing zones:

**Small Companies (1-3 teams, <50 engineers)**: Startups and small businesses seeking to establish foundational cloud governance without over-engineering. These teams need simple, cost-effective patterns that provide essential security and compliance while maintaining development velocity. 

**Medium Companies (4-10 teams, 50-200 engineers)**: Growing businesses experiencing the complexity that comes with multiple teams, diverse workloads, and increasing compliance requirements. These companies need structured approaches to team autonomy, clear boundaries between environments, and standardized patterns that prevent configuration drift while supporting innovation.

**Large Enterprises (10+ teams, 200+ engineers)**: Established enterprises with complex structures, strict compliance requirements, and diverse business units. These enterprises require sophisticated governance frameworks, advanced automation, and enterprise-grade operational patterns that support scale while maintaining security and compliance across multiple business domains.

## Prerequisites

- **Foundational Knowledge**: Before proceeding, we strongly recommend completing the [AWS Landing Zone with Terraform](https://developer.hashicorp.com/validated-patterns/terraform/build-aws-lz-with-terraform) tutorial to understand the foundational architecture, platform and application team roles, and basic HCP Terraform concepts that this guide builds upon.
- **HCP Terraform Access**: Organization owner or admin rights to create projects, manage teams, and configure workspaces
- **Terraform Experience**: Familiarity with Terraform configuration, state management, and module development
- **Organizational Context**: Understanding of your current team structure, growth plans, and operational requirements

## Organization Assessment Framework

Before selecting an implementation pattern, assess your organization using these key dimensions to determine the appropriate complexity level for your AWS landing zone.

### Team Structure and Scale

- **Number of development teams**: Count distinct teams that will deploy infrastructure
- **Team autonomy level**: Degree of independence teams require in their infrastructure decisions
- **Cross-team dependencies**: Frequency and complexity of shared infrastructure needs

### Technical and Operational Maturity

- **Cloud experience**: Team familiarity with AWS services and infrastructure as code
- **Governance requirements**: Regulatory, compliance, or internal policy needs
- **Operational capabilities**: Availability of dedicated platform or DevOps teams

### Growth and Complexity Factors

- **Geographic distribution**: Single region vs. multi-region requirements
- **Regulatory environment**: Industry-specific compliance requirements
- **Resource constraints**: Budget and staffing limitations for infrastructure management

### Self-Assessment Framework

Use this framework to evaluate which reference implementation aligns with your organization. These are reference points, not rigid categories.

| Dimension            | Small (1-3 teams)                          | Medium (4-10 teams)                       | Large (10+ teams)                                    |
|----------------------|--------------------------------------------|-------------------------------------------|------------------------------------------------------|
| **Team Structure**   | Single product, limited platform expertise | Multiple products, emerging platform team | Complex portfolio, dedicated platform/security teams |
| **Primary Focus**    | Development speed, cost efficiency         | Balance autonomy with standardization     | Comprehensive governance, enterprise operations      |
| **Governance Needs** | Basic security, minimal compliance         | Growing compliance requirements           | Strong regulatory compliance, audit requirements     |
| **Geographic Scope** | Single region                              | Potential multi-region                    | Multi-region or global presence                      |
| **Key Trade-off**    | Resource sharing for simplicity            | Increased complexity for better isolation | Significant overhead for maximum control             |

**Important Note**: Team count is only one factor. A 3-team organization in a highly regulated industry may require large organization patterns due to compliance requirements. Conversely, a 15-team organization with simple requirements might operate effectively with medium patterns. Assess your specific context rather than following team count alone.

### HCP Terraform Project Organization Patterns

HCP Terraform addresses infrastructure management challenges through its project-based organizational structure:

**Organizations**: The highest level of resource hierarchy, representing your company's overall HCP account. Manages billing, user access, and contains all projects and workspaces.

**Projects**: Logical groupings within an organization that organize related workspaces and resources. Projects enable teams to separate different environments, applications, or business units while maintaining shared settings and permissions.

**Workspaces**: The actual working environments where Terraform configurations are executed and state files are stored. Each workspace manages a distinct set of infrastructure resources and maintains its own state file, variables, and run history.

```text
┌─────────────────────────────────────────────────────────────┐
│           HCP Terraform Organization (FinTechCorp)          │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐ │
│  │  Project: Platform-Infrastructure                     │ │
│  │  ┌─────────────────────────────────────────────────┐  │ │
│  │  │  Workspace: plz-networking                      │  │ │
│  │  │  • Manages: plz-networking AWS account          │  │ │
│  │  │  • State: VPCs, subnets, Transit Gateway        │  │ │
│  │  └─────────────────────────────────────────────────┘  │ │
│  │  ┌─────────────────────────────────────────────────┐  │ │
│  │  │  Workspace: plz-security                        │  │ │
│  │  │  • Manages: plz-security AWS account            │  │ │
│  │  │  • State: GuardDuty, Security Hub, Config       │  │ │
│  │  └─────────────────────────────────────────────────┘  │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Figure 1** – HCP Terraform organizational hierarchy showing the relationship between Organizations, Projects, and Workspaces

You can learn more about using HCP Terraform projects in this tutorial: [Organize workspaces with projects](https://www.hashicorp.com/en/blog/a-smoother-hcp-terraform-workspace-experience).

#### Team-Based Access Control

HCP Terraform's access model is fundamentally team-based, requiring users to belong to teams with appropriate permissions. This approach aligns with enterprise security best practices by centralizing access control and simplifying user management.

The permission model in HCP Terraform operates across three hierarchical levels: organization, project, and workspace. While granular control is possible at each level, industry experience shows that managing permissions at the project level provides the optimal balance between security and administrative efficiency, allowing for a more organized view.

By grouping related workspaces into projects and applying permissions at the project level, you can implement consistent access patterns and centralized policy enforcement. This strategy significantly reduces administrative overhead in several ways:

- Simplified Workspace Management: Streamlines onboarding and operations by assigning read, write, maintain, admin, or a customized set of permissions to a particular team which grants specific access to all existing/future workspaces and shared configurations.
- Enhanced Security Controls: Project-level permissions enable you to effectively implement the principle of least privilege as teams receive access only to specific projects and their associated resources.
- Streamlined Compliance: Enables clear audit trails and compliance reporting by providing a unified view of access patterns. For example, you can also apply policy sets at a project level, helping you consistently enforce the organization's governance standards on all workspaces in the project.

When deciding on the logical boundaries for your projects, consider these Project Boundaries Best Practices.

### Workspace-to-Account Mapping Foundation

The relationship between HCP Terraform workspaces and AWS accounts is a fundamental architectural decision that affects state management, blast radius, operational complexity, and team workflows. Organizations have two primary approaches, each with distinct trade-offs and appropriate use cases.

#### Account-Level Workspace Strategy

The [foundational guide](/terraform/tutorials/validated-patterns/build-aws-lz-with-terraform) recommends **one workspace per AWS account**, this approach treats each workspace as managing all infrastructure within a single AWS account.

```text
┌─────────────────────────────────────────────────────────────┐
│              Account-Level Workspace Strategy                │
│                                                              │
│  AWS Account                    HCP Terraform Workspace     │
│  ┌──────────────────────┐      ┌─────────────────────────┐ │
│  │ plz-networking       │◄─────┤ Workspace:              │ │
│  │ • VPC, subnets       │      │ plz-networking          │ │
│  │ • Transit Gateway    │      │ (Manages entire account)│ │
│  │ • NAT Gateways       │      └─────────────────────────┘ │
│  └──────────────────────┘                                   │
│                                                              │
│  ┌──────────────────────┐      ┌─────────────────────────┐ │
│  │ banking-prod         │◄─────┤ Workspace:              │ │
│  │ • ECS/EKS            │      │ banking-prod            │ │
│  │ • RDS databases      │      │ (Manages entire account)│ │
│  │ • S3, Lambda, etc.   │      └─────────────────────────┘ │
│  └──────────────────────┘                                   │
└─────────────────────────────────────────────────────────────┘
```

**Benefits:**
- **Clear blast radius**: Changes affect only one AWS account
- **Simplified state management**: One Terraform state file per account
- **Account-level isolation**: Natural security and compliance boundaries
- **Direct cost tracking**: One-to-one correlation between workspace and AWS billing
- **Operational simplicity**: Fewer dependencies between workspaces

#### Component-Level Workspace Strategy

Organizations may choose to break down infrastructure into smaller, focused components within separate workspaces, even when sharing AWS accounts.

```text
┌─────────────────────────────────────────────────────────────┐
│             Component-Level Workspace Strategy               │
│                                                              │
│  AWS Account: banking-prod      HCP Terraform Workspaces    │
│  ┌──────────────────────┐      ┌─────────────────────────┐ │
│  │ • ECS Services       │◄─────┤ Workspace:              │ │
│  │ • ALB, API Gateway   │      │ banking-app-prod        │ │
│  │ • Lambda Functions   │      │ (Manages compute layer) │ │
│  │                      │      └─────────────────────────┘ │
│  │ • RDS Database       │      ┌─────────────────────────┐ │
│  │ • ElastiCache        │◄─────┤ Workspace:              │ │
│  │ • S3 Buckets         │      │ banking-data-prod       │ │
│  └──────────────────────┘      │ (Manages data layer)    │ │
│                                 └─────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

**Workspace Naming Convention:** `<business-unit>-<app-name>-<layer>-<env>`

**Examples:** `banking-core-app-prod`, `payments-processing-data-dev`, `platform-network-prod`

**Key Organizational Principles for Component-Level Scoping:**

1. **Volatility Separation**: Separate components by change frequency. VPCs and databases change rarely; application configurations change frequently. This protects stable infrastructure from unnecessary risk.

2. **Stateful vs. Stateless Resources**: Isolate databases and storage systems (stateful) from compute instances and applications (stateless). This prevents accidental data loss and enables independent management.

3. **Team Responsibilities**: Different teams require different access levels. Platform teams manage core infrastructure (`platform-network-prod`), while application teams handle service-specific resources (`banking-app-prod`).

4. **Functional Separation**: Break down infrastructure by component type and environment, creating clear boundaries between networking, applications, and monitoring components.

5. **Dependency Decoupling**: Avoid tightly coupled resources across workspaces. Use variable inputs or external data sources (SSM parameters) instead of direct Terraform outputs to prevent monolithic state dependencies.

#### Diagram 1: Workspace Scoping Principle
**Type**: Visual decision tree
**Purpose**: Help readers decide how to split workspaces
**Content**:

```text
Should this be a separate workspace?
    │
    ├─ Does it change frequently? ──YES──► Separate workspace
    │                              (Volatility)
    ├─ Is it stateful (database)? ──YES──► Separate workspace
    │                              (State management)
    ├─ Different team owns it? ────YES──► Separate workspace
    │                              (Team responsibility)
    ├─ Different component type? ──YES──► Separate workspace
    │                              (Functional separation)
    └─ Loosely coupled? ───────────YES──► Separate workspace
                                   (De-coupling)
```
**Important Note**: Both strategies are valid and supported by HashiCorp documentation. The choice depends on your organization's specific context, constraints, and maturity level. Organizations should select the approach that best balances operational complexity with their governance, security, and cost requirements.

For additional guidance on workspace organization, refer to [HCP Terraform workspace best practices](https://developer.hashicorp.com/terraform/cloud-docs/workspaces/best-practices) and [workspace sizing recommendations](https://support.hashicorp.com/hc/en-us/articles/38056391267091-Best-Practices-for-HCP-Terraform-Workspace-size).

## Project Organization Approaches

Organizations typically implement one of two primary approaches when structuring HCP Terraform projects, each optimized for different organizational contexts and constraints.

### Platform-First Approach

**Structure**: Clear separation between platform infrastructure and application workloads through dedicated projects.

**How it works**: Platform teams manage foundational infrastructure (networking, security, identity) in dedicated platform projects. Application teams manage their workloads in separate application projects, consuming platform services through well-defined interfaces.

**Example Structure**:

```text
Organization: FinTechCorp
├── Platform-Infrastructure Project
│   ├── plz-networking              # Manages networking account
│   ├── plz-security                # Manages security account
│   ├── plz-shared-services         # Manages shared services account
│   └── plz-identity                # Manages identity account
├── Banking-Application Project
│   ├── banking-dev                 # Manages banking-dev account
│   ├── banking-test                # Manages banking-test account
│   └── banking-prod                # Manages banking-prod account
└── Payments-Application Project
    ├── payments-dev                # Manages payments-dev account
    ├── payments-test               # Manages payments-test account
    └── payments-prod               # Manages payments-prod account
```

**Platform Service Contracts**: Platform teams expose services via `outputs.tf`, applications consume via `variables.tf`:

```hcl
# Platform workspace: plz-networking (outputs.tf)
output "vpc_id" {
  description = "ID of the provisioned VPC"
  value       = aws_vpc.main.id
}

# Application workspace: banking-prod (variables.tf)
variable "platform_vpc_id" {
  description = "VPC ID from platform team"
  type        = string
}
```

**Key advantages**:
- **Centralized governance**: Platform team maintains standardized security controls and compliance
- **Reduced duplication**: Proven, tested infrastructure patterns shared across teams
- **Consistent standards**: Uniform security, networking, and operational practices
- **Clear separation of concerns**: Platform teams focus on infrastructure, application teams focus on workloads

**Key challenges**:
- **Complex initial setup**: Requires careful design of platform service offerings and contracts
- **Coordination overhead**: Changes to platform services affect multiple consuming teams
- **Platform team bottleneck**: Platform team capacity can limit application team velocity

### Team-Based Approach

**Structure**: Each team receives their own project containing all infrastructure required for their workloads.

**How it works**: Teams operate as independent product units, managing their complete technology stack within dedicated projects. Platform services exist as shared foundation but don't constrain team autonomy.

**Example Structure**:

```text
Organization: FinTechCorp
├── Platform-Foundation Project
│   └── shared-services             # Basic shared infrastructure
├── Banking-Team Project
│   ├── banking-dev                 # Team manages complete dev stack
│   └── banking-prod                # Team manages complete prod stack
├── Payments-Team Project
│   ├── payments-dev                # Team manages complete dev stack
│   └── payments-prod               # Team manages complete prod stack
└── Risk-Team Project
    ├── risk-dev                    # Team manages complete dev stack
    └── risk-prod                   # Team manages complete prod stack
```

**Key advantages**:
- **Team autonomy**: Teams control their complete technology stack and deployment decisions
- **Faster velocity**: No dependency on platform team for infrastructure changes
- **Technology diversity**: Teams can choose technologies best suited for their requirements
- **Simplified permissions**: Clear project boundaries align with team responsibilities

**Key challenges**:
- **Resource duplication**: Teams may implement similar solutions independently
- **Inconsistent practices**: Without central governance, standards may diverge across teams
- **Operational overhead**: Each team responsible for their complete infrastructure lifecycle
- **Security gaps**: Decentralized security implementation may create vulnerabilities

### Selecting the Right Approach

| Factor                               | Platform-First Approach              | Team-Based Approach                     |
|--------------------------------------|--------------------------------------|-----------------------------------------|
| **Organization Size**                | Medium to Large (4+ teams)           | Small to Medium (1-6 teams)             |
| **Platform Team Maturity**           | Dedicated platform teams             | Limited or part-time platform resources |
| **Standardization Needs**            | High (compliance, governance)        | Low to Medium (flexibility prioritized) |
| **Team Autonomy Requirements**       | Medium (within platform constraints) | High (full stack control)               |
| **Technology Diversity**             | Low (standardized stack)             | High (team-specific choices)            |
| **Operational Complexity Tolerance** | High (can handle coordination)       | Low (prefer simplicity)                 |
| **Regulatory Requirements**          | Strong (centralized controls needed) | Basic (team-level controls sufficient)  |

### Hybrid Approach

Most organizations often benefit from combining of all the mentioned Multi-Team patterns, considering:

- Platform-First for shared infrastructure
- Team maturity level, team with low cloud adoption maturity might lean-in to have Platform-First model, where shared services are pre-provisioned. Another team with higher maturity might choose Team-based model, which gives them more flexibility with some guardrails.
- Team-Based ownership and control for application resources
- Environment considerations using different workspaces within each project
- Application boundaries respected in resource organization

## Repository Structure Evolution

Effectively organizing your infrastructure repositories can pose significant challenges, debating between a mono-repo or a multi-repo approach. Essentially it all comes down to your teams need for granular access control and configuration changes rate.

Workspaces align closely with modern version control practices:

- **Branch Association**: Workspaces can be linked to specific branches in a version control system (VCS), enabling parallel development and testing of infrastructure changes
- **Repository Integration**: Each workspace is typically connected to a specific Git repository, allowing for seamless integration between code changes and infrastructure updates
- **CI/CD Workflows**: Workspaces facilitate continuous integration and deployment by automatically triggering runs based on repository events like pull requests or merges

Repository structure evolves with organizational complexity, supporting team workflows while promoting reusability. Considering the workspaces associations, we find the following ways to structure our repositories.

### Mono-Repo Approach

#### Single Workspace per Repository Branch

In this approach, all code and modules reside in one repository. Each workspace is aligned to a specific branch (e.g., dev, test, prod), with different variables for each workspace. Changes are promoted through branch merging. This method offers benefits such as fewer files to manage, fewer plans needed, and better access control. However, it faces the challenge of potential branch drift.

#### Single Workspace per Repository Directory

This structure uses a single repository with separate directories for each environment. Modules are kept in a separate directory within the same repo. Each workspace is configured with different Terraform working directories. This approach provides a single source of truth for each environment, simplifies infrastructure state management, and allows for simple RBAC for workspaces. The main challenge is the manual promotion process between environments.

### Multi-Repo Approach

In a multi-repo approach you are extending the structure of Single Workspace per Repository Directory to multiple repositories, by essentially moving the modules into separate repositories and possibly integrating with [HCP Terraform Private Registry](https://developer.hashicorp.com/terraform/cloud-docs/registry). Application teams have their own repositories, and module versioning is achieved through release tagging. This approach offers benefits such as easier module versioning, team self-service capability, standardization based on shared building blocks and minimize the blast radius of changes. Due to the increase in complexity for managing multiple repositories and their dependencies, this approach is particularly suitable for large organizations with a Platform-First setup.


## Small Organizations: Getting Started Right

Small organizations require architectural patterns that provide scalability without unnecessary complexity for their current operational requirements. AWS Control Tower provides essential governance primitives including automated account provisioning, centralized logging through CloudTrail and Config, and baseline security controls via Service Control Policies, while maintaining operational simplicity for development teams.

The following diagram shows how all components work together in a small organization setup, from team access and repository structure to AWS infrastructure and daily operations:

<img src="/Users/pdigori/Documents/TFC/Hashicorp/aws-landing-zones-hcp/pasha-img/small-org-schema.png" alt="Small Organization Architecture" style="width: 100%; max-width: 1000px;">

**Figure 2** – Small organization integrated architecture showing team access, repository structure, HCP Terraform organization, AWS infrastructure, and operational workflows

### Account Structure That Scales

Small organizations typically implement 7-10 AWS accounts: the standard Control Tower managed accounts (Management, Log Archive, and Audit), plus a consolidated shared services account for networking, security, and CI/CD infrastructure.

Provision dedicated development and production accounts for each team. This approach establishes clear ownership boundaries while maintaining manageable operational overhead and eliminates shared development environment conflicts that occur during organizational growth.

### Organizing Teams and Projects

Use the Team-Based project pattern: each team gets their own project containing everything they need to manage their complete technology stack. This prioritizes autonomy and speed over standardization. A banking team can choose the database technology that makes sense for their use case, while a payments team optimizes for different requirements. The shared services project provides common infrastructure but doesn't constrain team-level decisions.

### Repository Organization

Implement a mono-repository pattern with directory-based workspace mapping. This approach consolidates all infrastructure-as-code in a single Git repository while maintaining clear separation of concerns through directory structure and Terraform working directories.

**Repository Structure and Workspace Mapping:**

```text
fintech-infrastructure/
├── modules/                          # Shared Terraform modules
│   ├── vpc/                          # Reusable VPC module
│   ├── ecs-service/                  # Standardized ECS service
│   └── rds-postgres/                 # Database module
├── platform/
│   └── shared-services/              # Platform infrastructure
│       ├── main.tf                   # VPC, Transit Gateway, CI/CD
│       ├── variables.tf
│   │   ├── outputs.tf
├── banking/                          # Banking team infrastructure
│   ├── dev/
│   │   ├── main.tf                   # ECS, RDS, ALB configuration
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   └── prod/                         # Production environment
├── payments/                         # Payments team infrastructure
│   ├── dev/                          # Development environment
│   └── prod/                         # Production environment
└── docs/                             # Runbooks and architecture decisions
```

**Workspace Configuration Options:**

- **Account-Level Mapping**: Each directory maps to one HCP Terraform workspace managing an entire AWS account (`banking/dev` → `banking-dev` workspace → `banking-dev` AWS account)
- **Component-Level Mapping**: Multiple workspaces per AWS account for finer-grained state management (`banking/dev` → `banking-dev-data` and `banking-dev-api` workspaces → single `banking-dev` AWS account)

Choose account-level mapping for operational simplicity, or component-level when you need to separate components based on the Woskapce scoping criteria (above) or when different teams manage different infrastructure layers.

### HashiCorp Platform Essentials

Small organizations can leverage HCP Terraform's native security and governance capabilities to implement robust controls without introducing operational complexity.

**Dynamic Provider Credentials:** Implement foundational security by configuring OIDC authentication between HCP Terraform and AWS to eliminate static credential dependencies. Each workspace assumes an IAM role with permissions scoped to its target AWS account using HCP Terraform's OIDC provider (`app.terraform.io`). This configuration establishes workspace-level isolation and implements AWS security best practices. See the [complete OIDC setup guide](https://developer.hashicorp.com/terraform/cloud-docs/workspaces/dynamic-provider-credentials/aws-configuration) for detailed implementation steps.

**Advisory Sentinel Policies:** Implement warning-level policies that establish governance standards without blocking deployment workflows:
- Mandatory resource tagging (Environment, Owner, CostCenter)
- Prevention of public S3 buckets and unencrypted storage
- Instance type restrictions and cost controls

[Policies](https://developer.hashicorp.com/sentinel/docs/examples) generate warnings in plan output, facilitating gradual team adoption of best practices. Enforcement levels can be increased as organizational governance maturity develops.

**Variable Sets:** Maintain consistency by defining common configuration parameters (AWS region, default tags, naming conventions) once and applying them across workspaces. This approach eliminates configuration drift and reduces duplication. Implement organization-level sets for global standards and project-level sets for team-specific configuration requirements.

**Team Management:** Configure team structures that align with organizational hierarchy—platform administrators with organization-level access, application teams with project-scoped permissions. Integrate with enterprise identity providers (Okta, Azure AD, SAML) for centralized user lifecycle management. Details on [SSO integration and team management](https://developer.hashicorp.com/terraform/cloud-docs/users-teams-organizations/single-sign-on) are available in the documentation.

**Drift Detection:** Implement operational governance through [health assessments and drift detection](https://developer.hashicorp.com/terraform/cloud-docs/workspaces/health) to automatically identify manual infrastructure changes outside of Terraform workflows. Configure scheduled drift detection reports and establish remediation procedures: evaluate drift → import legitimate changes → revert unauthorized modifications.
### Deployment Workflows

Implement HCP Terraform's native VCS integration using standard Git workflows: create feature branches, implement changes, and submit pull requests. HCP Terraform automatically generates execution plans that detail proposed infrastructure modifications.

Configure automatic plan application for development environments to enable rapid iteration cycles. Production changes require manual approval workflows—review execution plans, approve changes, and allow HCP Terraform to handle infrastructure provisioning.

### Evolution Readiness

Establish foundational practices early including naming conventions, consistent tagging standards, and basic documentation. While these practices may appear as operational overhead initially, they become critical as teams scale and prevent technical debt that requires costly refactoring efforts. Define clear ownership boundaries and document common operational procedures to prevent knowledge loss or bottlenecks.

Evolution to medium organizational patterns is indicated when deployment conflicts occur frequently, compliance frameworks become mandatory, or dedicated platform engineering resources are hired.

## Medium Organizations: The Platform Transition

The transition to medium organizational patterns typically occurs at 4-10 teams when shared infrastructure management approaches reach operational limits. Key indicators include frequent deployment conflicts, configuration drift across environments, and critical knowledge dependencies on individual team members. These scaling challenges necessitate the introduction of dedicated platform engineering capabilities and more structured governance frameworks.

### Platform Engineering Transformation

Medium organizations require a fundamental shift from shared resource management to platform engineering. This transformation addresses operational friction through dedicated platform capabilities and standardized service interfaces.

**Organizational Changes:**
- Establish dedicated platform engineering roles focused on infrastructure-as-a-product
- Decompose monolithic shared services into specialized platform services
- Implement service-oriented architecture for infrastructure components

**Infrastructure Decomposition:**
Break apart the consolidated shared services account into specialized platform accounts:
- **Networking Platform**: VPC management, Transit Gateway, DNS services
- **Security Platform**: Identity management, certificate authority, security monitoring
- **CI/CD Platform**: Build systems, artifact repositories, deployment pipelines
- **Observability Platform**: Logging, monitoring, alerting infrastructure

This decomposition enables granular access control, independent service evolution, and clear ownership boundaries.

### Project Organization

Adopt platform-first patterns: separate platform infrastructure from application workloads. The platform team manages foundational infrastructure (networking, security, CI/CD, identity) in their own project, with each service getting its own workspace and AWS account.

Application teams consume platform services through well-defined interfaces rather than managing everything themselves. The platform team exposes outputs that application teams consume as inputs. This enables independent movement: platform teams upgrade infrastructure without affecting applications, while application teams iterate without worrying about underlying platform changes.

### Platform Governance and Modules

**Sentinel Policy Evolution:** Move from advisory to soft enforcement. Policies can be overridden with justification:
- Naming conventions and tagging standards
- Instance type restrictions in non-production
- Cost controls and backup requirements
- Security baselines (encryption, private subnets)

Platform team maintains policies as code. Apply policy sets at organization or project level for automatic enforcement.

**HCP Terraform Private Registry:** Publish platform modules with semantic versioning. Application teams consume tested, documented modules:

```hcl
module "vpc" {
  source  = "app.terraform.io/fintechcorp/vpc/aws"
  version = "~> 2.0"
}
```

Benefits: Version control prevents breaking changes, documentation auto-generated, usage analytics track adoption.

**Enhanced Dynamic Credentials:** Scope IAM roles to specific workspaces and run phases (plan vs apply). Platform workspaces get broader permissions; application workspaces restricted to specific services.

**HCP Vault:** Adopt when managing database credentials, API keys, or certificates across multiple applications. Vault generates short-lived credentials on-demand, eliminating long-lived secrets. Platform team configures Vault; application teams consume via Terraform provider.

**HCP Packer:** For EC2/ECS workloads, publish hardened AMIs to HCP Packer registry. Platform team maintains security-hardened images; application teams consume versioned AMIs.

### Repository Structure

Transition to multiple repositories with clear ownership boundaries. The platform team gets dedicated repositories for infrastructure and a shared module library published to [HCP Terraform Private Registry](https://developer.hashicorp.com/terraform/cloud-docs/registry). Application teams get their own repositories where they iterate quickly without affecting others.

Modules become the interface between teams. The platform team publishes tested, versioned modules that application teams consume, creating stability while allowing platform evolution.

**FinTechCorp Example:**
- **Platform repositories**: Core configurations for shared services, baselines, governance, identity, networking, policies, and log management
- **Module library**: Parameterized baseline configurations for common resources (S3, RDS, EC2) published to HCP Private Registry
- **Application repositories**: Each team (e.g., payments-app) manages their services by environment, consuming versioned modules from the registry

<img src="/Users/pdigori/Documents/TFC/Hashicorp/aws-landing-zones-hcp/pasha-img/Hashicorp-blog-schemas-Multirepo.png" alt="Small Organization Architecture" style="width: 100%; max-width: 1000px;">

Figure 2. Multi-Repo for Medium Organization

### Deployment Workflows

Implement hybrid workflows: platform and application teams follow different patterns optimized for their needs.

Platform teams adopt conservative practices with manual approval gates and scheduled maintenance windows for changes affecting multiple teams. Application teams move faster within their boundaries—auto-deploying to development and self-servicing through testing and production as long as they consume stable platform interfaces.

### Platform Capabilities

Build platform services that solve cross-team operational problems:

**Security Services:**
- Deploy GuardDuty and Security Hub across all accounts
- Implement centralized CloudTrail logging with automated alerting
- Create standardized IAM roles and policies for common use cases

**Cost Management:**
- Implement AWS Cost and Usage Reports with team-level tagging
- Set up budget alerts and spending notifications per team/project
- Create cost allocation dashboards showing team and environment spending

**Compliance Tooling:**
- Deploy AWS Config rules for security baseline enforcement
- Implement automated backup policies across all production resources
- Create compliance reporting dashboards for SOC 2 or similar frameworks

**Implementation Strategy:**
Implement platform capabilities incrementally using a value-driven approach. Begin with operational pain points that affect multiple application teams, then expand service offerings as platform adoption demonstrates measurable value to internal stakeholders. For detailed guidance on application team onboarding processes, see [A smoother HCP Terraform workspace experience](https://www.hashicorp.com/en/blog/a-smoother-hcp-terraform-workspace-experience).

**Evolution Criteria:**
Organizations should transition to large-scale patterns when operational requirements include multiple business units with distinct regulatory requirements, formal compliance frameworks mandating enterprise governance, or 10+ teams requiring business unit-level isolation and specialized platform services.

## Large Organizations: Enterprise Scale Complexity

Large organization patterns emerge at two critical thresholds: multiple business units with distinct requirements, and regulatory demands requiring formal governance frameworks. This is about managing fundamentally different complexity—business units with conflicting requirements, regulatory frameworks demanding specific controls, and stakeholders needing different levels of visibility and governance.

### Business Unit Architecture

Multiple business units drive architecture differently than team count alone. A consumer banking division has different regulatory requirements than commercial lending, even within the same company. You need account structures providing clear business unit isolation, governance frameworks accommodating different compliance requirements, and platform services serving diverse customers without bottlenecks.

The platform team evolves into a platform organization with specialized teams: core infrastructure, security and compliance, data platforms, and developer experience. This specialization builds enterprise-grade capabilities while maintaining agility.

### Enterprise Governance and Compliance

#### Hard Enforcement Policies

Implement blocking Sentinel policies with formal exception processes:
- Security policies: No override (prevent public access, require encryption)
- Compliance policies: Exception workflow (CIS, PCI-DSS, HIPAA, GLBA)
- Operational policies: Soft enforcement (naming, tagging, cost controls)

Apply different policy sets per business unit while maintaining enterprise baselines. Security team maintains policy-as-code repository.

#### Business Unit Isolation

Use Vault namespaces to isolate secrets by business unit (/consumer-banking, /commercial-banking, /payments). Delegate namespace administration to business unit teams while platform team maintains root namespace.

#### Enterprise Module Governance

Treat modules as products with SLAs and lifecycle management:
- Certified modules: Enterprise-approved, fully supported
- Experimental modules: Beta testing, limited support
- Deprecated modules: Formal sunset process with migration guides

#### Advanced Dynamic Credentials

Implement fine-grained IAM conditions based on business unit, environment, and workspace. Add IP restrictions, time-based access, and MFA requirements for production.

#### Run Tasks Integration

Integrate external tools into Terraform workflow:
- Security scanning (Snyk, Bridgecrew)
- Cost analysis (Infracost)
- Compliance validation (custom tools)
- Change management (ServiceNow integration)

Run tasks execute during plan phase, blocking apply if critical issues found.

#### Monitoring and Reporting

Enterprise dashboards for workspace health, drift detection, policy compliance, and cost trends. Automated compliance reports for audit readiness.

### Choosing Your Governance Foundation

Large organizations face a critical decision about their governance foundation that smaller organizations don't encounter: whether to build on AWS Control Tower or implement custom governance frameworks. This choice depends heavily on your existing constraints, regulatory requirements, and organizational context.

**AWS Control Tower** provides the fastest path to enterprise-grade governance for organizations starting fresh or willing to adapt their practices to AWS best practices. It offers automated account provisioning, centralized logging, and baseline security controls that work well for most enterprise scenarios. Organizations in regulated industries often find that Control Tower's built-in compliance frameworks accelerate their certification processes.

However, large organizations with existing governance frameworks, complex regulatory requirements, or significant legacy constraints may need more flexibility than Control Tower provides. Some scenarios where custom approaches make sense include:

- **Existing enterprise governance tools** that need to integrate with AWS account management
- **Complex regulatory requirements** that require specific account structures or controls not supported by Control Tower
- **Multi-cloud strategies** where AWS account governance needs to align with governance frameworks spanning other cloud providers
- **Acquisition-heavy organizations** where inherited AWS accounts need to be integrated into existing governance patterns

### Account Organization

Organize accounts around business units rather than individual teams. Each business unit gets its own organizational unit with dedicated development, testing, and production environments. This enables business unit-specific policies—a consumer banking division can implement stricter data residency requirements without affecting a payments division's global operations.

```
Root OU
├── Management Account (Governance foundation)
├── Security OU (Centralized governance)
│   ├── Log Archive Account (Centralized logging)
│   └── Audit Account (Compliance and monitoring)
├── Platform OU (Infrastructure services)
│   ├── plz-networking (Hub VPC, Transit Gateway)
│   ├── plz-security (GuardDuty, Security Hub, Config)
│   ├── plz-shared-services (CI/CD, ECR, monitoring)
│   ├── plz-identity (IAM Identity Center, SSO)
│   ├── plz-data (Data lakes, analytics)
│   └── plz-sandbox (Developer experimentation)
└── Workloads OU
    ├── Consumer-Banking OU
    │   ├── Dev OU: consumer-core-dev, consumer-mobile-dev
    │   ├── Test OU: consumer-core-test, consumer-mobile-test
    │   └── Prod OU: consumer-core-prod, consumer-mobile-prod
    ├── Commercial-Banking OU
    │   ├── Dev OU: commercial-lending-dev, commercial-treasury-dev
    │   └── Prod OU: commercial-lending-prod, commercial-treasury-prod
    ├── Payments OU
    │   ├── Dev OU: payments-processing-dev, payments-fraud-dev
    │   └── Prod OU: payments-processing-prod, payments-fraud-prod
    └── Risk-Compliance OU
        ├── Dev OU: risk-assessment-dev, regulatory-dev
        └── Prod OU: risk-assessment-prod, regulatory-prod
```

*Note: This structure works with AWS Control Tower for automated governance, or can be implemented with custom governance frameworks for AWS organizations with specific requirements.*

### Project Organization

Implement hybrid project patterns: centralized platform services with distributed business unit operations. The platform organization maintains tight control over foundational services (networking, security), while business units move independently within their domains. Specialized capabilities like module development and policy management get dedicated projects for iteration without affecting business unit operations.

### Repository Structure

Implement comprehensive repository separation with dedicated policy-as-code repositories and private module registries. Security and compliance teams define organizational policies as versioned code, automatically enforced across all deployments.

Organize business unit repositories around business domains rather than individual teams, supporting cross-team collaboration while maintaining clear ownership boundaries.

### Deployment Workflows

Implement formal change management balancing development velocity with governance requirements. Platform changes follow RFC processes with multi-stage validation (security, performance, business impact) and scheduled maintenance windows. Business unit teams get self-service capabilities with comprehensive audit trails and automated policy enforcement—moving quickly through development and testing while meeting production governance requirements.

### Platform Capabilities

Develop comprehensive capabilities: automated account vending, policy-as-code frameworks, enterprise observability, and formal risk management. Specialized teams focus on core infrastructure, security and policy enforcement, deployment automation, and data platforms.

Deliver capabilities through well-defined service catalogs with clear SLAs and support models. Treat the platform as a product with internal customers—complete with product management, customer support, and continuous improvement. This ensures platform capabilities evolve to meet changing business needs while maintaining operational excellence.


## Organizational Size Comparison

The following table provides a quick reference for how patterns scale across organization sizes:

| Aspect                   | Small (1-3 teams)             | Medium (4-10 teams)                                   | Large (10+ teams)                                |
|--------------------------|-------------------------------|-------------------------------------------------------|--------------------------------------------------|
| **AWS Accounts**         | 7-10 accounts                 | 10-15 accounts                                        | 20+ accounts                                     |
| **Platform Accounts**    | 1 (shared-services)           | 4-5 (networking, security, shared-services, identity) | 6+ (full PLZ)                                    |
| **Project Pattern**      | Team-Based                    | Platform-First                                        | Platform-First + Application-Based               |
| **Repository Structure** | Mono-repo                     | Multi-repo with shared modules                        | Enterprise multi-repo with private registry      |
| **CI/CD Complexity**     | Simple VCS-driven             | Hybrid (platform manual, app automated)               | Enterprise with formal change management         |
| **Governance**           | Basic (CT guardrails)         | Moderate (CT + custom SCPs)                           | Comprehensive (Sentinel + full policy framework) |
| **Variable Management**  | Minimal variable sets         | Structured hierarchy                                  | Advanced with policy validation                  |
| **Account Provisioning** | Control Tower Account Factory | CT Account Factory                                    | CT Account Factory + AFT                         |


## Conclusion

This guide provides opinionated recommendations for implementing AWS landing zones appropriate to your organizational context. Key principles across all sizes:

- Deploy AWS Control Tower from day one for baseline governance
- Use 1:1 workspace-to-account mapping for consistent, scalable patterns
- Implement appropriate complexity—neither over-engineering nor under-engineering
- Make architectural decisions based on specific requirements, not team count alone

The small, medium, and large patterns are illustrative examples, not rigid categories. Your organization may combine elements from multiple patterns or adapt them to unique requirements.

**Next Steps:**
1. Assess your organization using the framework provided
2. Evaluate which reference implementation aligns with your requirements
3. Understand the trade-offs for your chosen approach
4. Implement incrementally, validating against operational reality
5. Reassess periodically as context evolves
6. Reference the [foundational guide](/terraform/tutorials/validated-patterns/build-aws-lz-with-terraform) for detailed implementation guidance
