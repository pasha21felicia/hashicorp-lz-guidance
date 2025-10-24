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

This guide builds upon the foundational concepts established in [Use Terraform to build an AWS landing zone](/terraform/tutorials/validated-patterns/build-aws-lz-with-terraform) to focus specifically on organizational right-sizing and implementation patterns. While the foundational guide demonstrates enterprise-scale architecture with comprehensive Platform Landing Zones (PLZs) and Workload Landing Zones (WLZs), organizations at different stages of growth face unique opportunities in determining the right level of complexity for their current state.

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

In a multi-repo approach you are extending the structure of Single Workspace per Repository Directory to multiple repositories, by essentially moving the modules into separate repositories and possibly integrating with HCP Terraform Private Registry. Application teams have their own repositories, and module versioning is achieved through release tagging. This approach offers benefits such as easier module versioning, team self-service capability, standardization based on shared building blocks and minimize the blast radius of changes. Due to the increase in complexity for managing multiple repositories and their dependencies, this approach is particularly suitable for large organizations with a Platform-First setup.


## Small Organizations: Getting Started Right

When building your first AWS landing zone with a small team, the challenge is finding the right balance between moving fast and building something that won't break as you grow. You don't need enterprise-grade complexity, but you do need enterprise-grade foundations. AWS Control Tower provides these foundations out of the box, giving you governance guardrails that matter later without slowing you down today.

### Account Structure That Scales

Small organizations typically start with 7-10 AWS accounts: the standard Control Tower managed accounts (Management, Log Archive, and Audit), plus a consolidated shared services account for networking, security, and CI/CD infrastructure.

Give each team their own development and production accounts. This creates clear ownership boundaries while keeping operational overhead manageable and prevents the "shared development environment" conflicts that emerge as you grow.

```text
┌─────────────────────────────────────────────────────────────┐
│         AWS Control Tower Landing Zone Structure             │
│                    (Small Organization: 1-2 Teams)           │
│                                                              │
│  Root OU                                                     │
│  ├── Management Account (Control Tower home)                │
│  │                                                           │
│  ├── Security OU (Control Tower managed)                     │
│  │   ├── Log Archive Account (CT managed)                   │
│  │   └── Audit Account (CT managed)                         │
│  │                                                           │
│  ├── Infrastructure OU (Custom)                              │
│  │   └── Shared Services Account                            │
│  │       • VPC, Transit Gateway, NAT                         │
│  │       • Security services                                 │
│  │       • CI/CD, ECR, monitoring                            │
│  │                                                           │
│  └── Workloads OU (Custom)                                   │
│      ├── banking-dev (Banking team dev environment)          │
│      ├── banking-prod (Banking team production)              │
│      ├── payments-dev (Payments team dev environment)        │
│      └── payments-prod (Payments team production)            │
└─────────────────────────────────────────────────────────────┘
                              ↕
                    1:1 Workspace Mapping
                              ↕
┌─────────────────────────────────────────────────────────────┐
│              HCP Terraform Organization Structure            │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Platform-Infrastructure Project                     │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │  Workspace: shared-services                    │  │  │
│  │  │  → Manages: shared-services account (entire)   │  │  │
│  │  │  → Resources: VPC, TGW, security, CI/CD, ECR   │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Banking-Application Project                         │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │  Workspace: banking-dev                        │  │  │
│  │  │  → Manages: banking-dev account (entire)       │  │  │
│  │  │  → Resources: ECS, RDS, ALB, S3, Lambda, etc.  │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │  Workspace: banking-prod                       │  │  │
│  │  │  → Manages: banking-prod account (entire)      │  │  │
│  │  │  → Resources: All production banking resources │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Payments-Application Project                        │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │  Workspace: payments-dev                       │  │  │
│  │  │  → Manages: payments-dev account (entire)      │  │  │
│  │  │  → Resources: Lambda, DynamoDB, SQS, S3, etc.  │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │  Workspace: payments-prod                      │  │  │
│  │  │  → Manages: payments-prod account (entire)     │  │  │
│  │  │  → Resources: All production payment resources │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Organizing Teams and Projects

Use the Team-Based project pattern: each team gets their own project containing everything they need to manage their complete technology stack. This prioritizes autonomy and speed over standardization. A banking team can choose the database technology that makes sense for their use case, while a payments team optimizes for different requirements. The shared services project provides common infrastructure but doesn't constrain team-level decisions.

### Repository Organization

Start with a mono-repository approach: all infrastructure code in a single Git repository with directories organized by team and environment. Each team gets their own directory containing development and production configurations. Shared modules live in a common area where teams contribute reusable patterns as they discover them.

### HashiCorp Platform Essentials

Small organizations benefit from HCP Terraform's built-in security and governance without additional overhead.

**Dynamic Provider Credentials:** Configure OIDC authentication between HCP Terraform and AWS to eliminate static credentials. Each workspace assumes an IAM role scoped to its AWS account. This is foundational security—implement on day one.

**Advisory Sentinel Policies:** Start with warning-level policies that educate without blocking:
- Mandatory tagging (Environment, Owner, CostCenter)
- Prevent public S3 buckets
- Require encryption at rest

Policies warn but don't block deployments. Gradually increase enforcement as your team matures.

**Variable Sets:** Define common configuration once (AWS region, tags, naming conventions) and apply across workspaces. Reduces duplication and ensures consistency.

**Team Management:** Structure teams to match your organization—platform admins, application teams. Connect to your identity provider (Okta, Azure AD) for centralized user management.

**Drift Detection:** Enable health assessments to identify manual changes. Review drift weekly and either revert or import into Terraform.

### Deployment Workflows

Leverage HCP Terraform's native VCS integration with familiar Git patterns: create a feature branch, make changes, and open a pull request. HCP Terraform automatically generates a plan showing exactly what will change.

Auto-apply changes when pull requests merge to development environments for fast feedback loops. Production changes require manual approval—review the plan, approve, and HCP Terraform handles execution.

### Evolution Readiness

Establish good practices early: naming conventions, tagging, and documentation. These seem like overhead when small but become critical as you add teams. Create clear ownership boundaries and runbooks for common operations.

Resist implementing enterprise patterns before you need them. Focus on velocity and learning on solid foundations. You're ready to evolve to medium patterns when you experience deployment conflicts weekly, need compliance frameworks, or hire your first dedicated platform engineer.

## Medium Organizations: The Platform Transition

The transition to medium patterns typically happens at 4-10 teams when shared infrastructure management breaks down. Symptoms include deployment conflicts, inconsistent configurations, and knowledge concentrated in one person becoming a bottleneck. This is when organizations introduce dedicated platform capabilities.

### Recognizing the Inflection Point

The shared services account becomes a source of friction: teams step on each other's deployments, configuration drift creates issues, and simple changes require coordination across multiple teams. New requirements emerge—SOC 2 compliance, multi-region expansion, or stronger environment isolation—that simple patterns can't accommodate.

These pressures justify dedicated platform engineers who build shared capabilities that all teams leverage.

### Establishing the Platform Team

Start with your first dedicated platform engineer or transition an existing team member to full-time infrastructure focus. They shift from maintaining shared resources to building standardized patterns that teams can self-service—thinking about infrastructure as a product with internal customers.

First priority: break apart the monolithic shared services account. Networking, security, CI/CD, and identity management each get their own account with clear ownership and interfaces, enabling better isolation and granular access control.

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

Transition to multiple repositories with clear ownership boundaries. The platform team gets dedicated repositories for infrastructure and a shared module library published to HCP Terraform Private Registry. Application teams get their own repositories where they iterate quickly without affecting others.

Modules become the interface between teams. The platform team publishes tested, versioned modules that application teams consume, creating stability while allowing platform evolution.

**FinTechCorp Example:**
- **Platform repositories**: Core configurations for shared services, baselines, governance, identity, networking, policies, and log management
- **Module library**: Parameterized baseline configurations for common resources (S3, RDS, EC2) published to HCP Private Registry
- **Application repositories**: Each team (e.g., payments-app) manages their services by environment, consuming versioned modules from the registry

### Deployment Workflows

Implement hybrid workflows: platform and application teams follow different patterns optimized for their needs.

Platform teams adopt conservative practices with manual approval gates and scheduled maintenance windows for changes affecting multiple teams. Application teams move faster within their boundaries—auto-deploying to development and self-servicing through testing and production as long as they consume stable platform interfaces.

### Platform Capabilities

Develop capabilities beyond shared infrastructure: centralized security monitoring, structured cost allocation, and standardized compliance patterns. Build incrementally—start with pain points affecting multiple teams, then expand as you prove value and build trust with internal customers.

You're ready to evolve to large patterns when you have multiple business units with distinct requirements, formal compliance frameworks, or 10+ teams requiring business unit isolation.

## Large Organizations: Enterprise Scale Complexity

Large organization patterns emerge at two critical thresholds: multiple business units with distinct requirements, and regulatory demands requiring formal governance frameworks. This is about managing fundamentally different complexity—business units with conflicting requirements, regulatory frameworks demanding specific controls, and stakeholders needing different levels of visibility and governance.

### Business Unit Architecture

Multiple business units drive architecture differently than team count alone. A consumer banking division has different regulatory requirements than commercial lending, even within the same company. You need account structures providing clear business unit isolation, governance frameworks accommodating different compliance requirements, and platform services serving diverse customers without bottlenecks.

The platform team evolves into a platform organization with specialized teams: core infrastructure, security and compliance, data platforms, and developer experience. This specialization builds enterprise-grade capabilities while maintaining agility.

### Enterprise Governance and Compliance

**Hard Enforcement Policies:** Implement blocking Sentinel policies with formal exception processes:
- Security policies: No override (prevent public access, require encryption)
- Compliance policies: Exception workflow (CIS, PCI-DSS, HIPAA, GLBA)
- Operational policies: Soft enforcement (naming, tagging, cost controls)

Apply different policy sets per business unit while maintaining enterprise baselines. Security team maintains policy-as-code repository.

**Business Unit Isolation:** Use Vault namespaces to isolate secrets by business unit (/consumer-banking, /commercial-banking, /payments). Delegate namespace administration to business unit teams while platform team maintains root namespace.

**Enterprise Module Governance:** Treat modules as products with SLAs and lifecycle management:
- Certified modules: Enterprise-approved, fully supported
- Experimental modules: Beta testing, limited support
- Deprecated modules: Formal sunset process with migration guides

**Advanced Dynamic Credentials:** Implement fine-grained IAM conditions based on business unit, environment, and workspace. Add IP restrictions, time-based access, and MFA requirements for production.

**Run Tasks:** Integrate external tools into Terraform workflow:
- Security scanning (Snyk, Bridgecrew)
- Cost analysis (Infracost)
- Compliance validation (custom tools)
- Change management (ServiceNow integration)

Run tasks execute during plan phase, blocking apply if critical issues found.

**Monitoring and Reporting:** Enterprise dashboards for workspace health, drift detection, policy compliance, and cost trends. Automated compliance reports for audit readiness.

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
