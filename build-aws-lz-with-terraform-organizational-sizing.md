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

In this guide we explore patterns for architecting HCP Terraform deployments that support team autonomy while maintaining operational consistency. We'll dive into practical strategies for workspace segregation, project hierarchy design, and repository layout that enable both team-level agility and enterprise-wide governance. Through practical examples using FinTechCorp—a fictional company at different growth stages—you'll discover how to implement landing zone patterns that enable team autonomy while maintaining security, compliance, and infrastructure reusability across your organization.

## Scope

We examine three reference implementations representing small (1-3 teams), medium (4-10 teams), and large (10+ teams) organizations, demonstrating proven patterns you can adapt to your specific context and growth trajectory.

Each implementation reveals how AWS account structures map to HCP Terraform workspaces, how project boundaries align with team responsibilities, and how repository organization supports both collaboration and autonomy. Understanding these patterns enables you to select the approach that fits your current scale while establishing practices that support your future growth.

## Target Audience

This guide addresses companies at different scales of cloud adoption, each facing distinct challenges in implementing AWS landing zones:

**Small Companies (1-3 teams, <50 engineers)**: Startups and small businesses seeking to establish foundational cloud governance without over-engineering. These teams need simple, cost-effective patterns that provide essential security and compliance while maintaining development velocity. The focus is on establishing good practices early that can scale as the company grows.

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

#### Understanding HCP Terraform Hierarchy

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

**When to Use:**
- Medium to large organizations (4+ teams)
- Organizations with sufficient AWS accounts (8+ accounts)
- Regulated industries requiring account-level isolation
- Teams with dedicated platform/DevOps capabilities
- When account provisioning is automated (Control Tower + AFT)

#### Component-Level Workspace Strategy

Organizations may choose to break down infrastructure into smaller, focused components within separate workspaces, even when sharing AWS accounts. This approach follows HashiCorp's [workspace scoping recommendations](https://developer.hashicorp.com/terraform/cloud-docs/workspaces/best-practices).

```
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
- `<business-unit>`: Team or business unit owning the workspace
- `<app-name>`: Application or service name
- `<layer>`: Infrastructure layer (app, data, network)
- `<env>`: Environment (dev, test, prod)

**Examples:** `banking-core-app-prod`, `payments-processing-data-dev`, `platform-network-prod`

**Key Organizational Principles for Component-Level Scoping:**

1. **Volatility Separation**: Separate components by change frequency. VPCs and databases change rarely; application configurations change frequently. This protects stable infrastructure from unnecessary risk.

2. **Stateful vs. Stateless Resources**: Isolate databases and storage systems (stateful) from compute instances and applications (stateless). This prevents accidental data loss and enables independent management.

3. **Team Responsibilities**: Different teams require different access levels. Platform teams manage core infrastructure (`platform-network-prod`), while application teams handle service-specific resources (`banking-app-prod`).

4. **Functional Separation**: Break down infrastructure by component type and environment, creating clear boundaries between networking, applications, and monitoring components.

5. **Dependency Decoupling**: Avoid tightly coupled resources across workspaces. Use variable inputs or external data sources (SSM parameters) instead of direct Terraform outputs to prevent monolithic state dependencies.

**When to Use:**
- Small organizations (1-3 teams) with limited AWS accounts
- Cost-constrained environments where account overhead is prohibitive
- High-trust teams with significant collaboration requirements
- Organizations with complex, stable accounts requiring fine-grained management

### Diagram 1: Workspace Scoping Principles
**Type**: Visual decision tree
**Purpose**: Help readers decide how to split workspaces
**Content**:
```
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

### Project Organization Approaches

Organizations typically implement one of two primary approaches when structuring HCP Terraform projects, each optimized for different organizational contexts and constraints.

#### Platform-First Approach

**Structure**: Clear separation between platform infrastructure and application workloads through dedicated projects.

**How it works**: Platform teams manage foundational infrastructure (networking, security, identity) in dedicated platform projects. Application teams manage their workloads in separate application projects, consuming platform services through well-defined interfaces.

**Example Structure**:
```
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

#### Team-Based Approach

**Structure**: Each team receives their own project containing all infrastructure required for their workloads.

**How it works**: Teams operate as independent product units, managing their complete technology stack within dedicated projects. Platform services exist as shared foundation but don't constrain team autonomy.

**Example Structure**:
```
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

#### Selecting the Right Approach

| Factor                               | Platform-First Approach              | Team-Based Approach                     |
|--------------------------------------|--------------------------------------|-----------------------------------------|
| **Organization Size**                | Medium to Large (4+ teams)           | Small to Medium (1-6 teams)             |
| **Platform Team Maturity**           | Dedicated platform teams             | Limited or part-time platform resources |
| **Standardization Needs**            | High (compliance, governance)        | Low to Medium (flexibility prioritized) |
| **Team Autonomy Requirements**       | Medium (within platform constraints) | High (full stack control)               |
| **Technology Diversity**             | Low (standardized stack)             | High (team-specific choices)            |
| **Operational Complexity Tolerance** | High (can handle coordination)       | Low (prefer simplicity)                 |
| **Regulatory Requirements**          | Strong (centralized controls needed) | Basic (team-level controls sufficient)  |

#### The Best of all Patterns

Most organizations often benefit from combining of all the mentioned Multi-Team patterns, considering:

* Platform-First for shared infrastructure
* Team maturity level, team with low cloud adoption maturity might lean-in to have Platform-First model, where shared services are pre-provisioned. Another team with higher maturity might choose Team-based model, which gives them more flexibility with some guardrails.
* Team-Based ownership and control for application resources
* Environment considerations using different workspaces within each project
* Application boundaries respected in resource organization 

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

When you're building your first AWS landing zone with a small team, the challenge isn't technical complexity—it's finding the right balance between moving fast and building something that won't break as you grow. Most small organizations start with 1-3 product teams, each wearing multiple hats and sharing infrastructure responsibilities.

The key insight for small organizations is that you don't need enterprise-grade complexity, but you do need enterprise-grade foundations. AWS Control Tower provides these foundations out of the box, giving you the governance guardrails that will matter later without slowing you down today.

### The Small Organization Reality

Small teams face a unique set of constraints. Everyone is focused on building the product, and infrastructure often becomes a shared responsibility that rotates between team members. You might have one person who's "good with AWS" handling most infrastructure decisions, but they're also writing application code and attending product meetings.

This reality shapes your architecture decisions. You need patterns that are simple enough for any team member to understand and maintain, but robust enough to support real production workloads. The goal is to establish good practices early that will scale naturally as your organization grows.

### Account Structure That Scales

Small organizations typically start with 7-10 AWS accounts, following the foundational patterns but with simplified implementation. You'll have the standard Control Tower managed accounts (Management, Log Archive, and Audit), plus a consolidated shared services account that handles networking, security, and CI/CD infrastructure.

The key decision is how to organize your workload accounts. Most successful small organizations give each team their own development and production accounts, creating clear ownership boundaries while keeping operational overhead manageable. This might seem like overkill when you're small, but it prevents the "shared development environment" problems that plague growing teams.

```
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

The architecture follows the same 1:1 workspace-to-account mapping recommended in the foundational guide, but with fewer total accounts to manage. Each AWS account maps to exactly one HCP Terraform workspace, creating predictable boundaries and simplified state management.

### Organizing Teams and Projects

Small organizations benefit from the Team-Based project pattern in HCP Terraform. Rather than separating platform and application concerns, each team gets their own project containing everything they need to manage their complete technology stack.

This approach recognizes that small teams need autonomy and speed more than they need standardization. A banking team can choose the database technology that makes sense for their use case, while a payments team can optimize for different requirements. The shared services project provides common infrastructure like networking and CI/CD, but doesn't constrain team-level decisions.

The project structure naturally evolves with your organization. When you eventually need more standardization or governance, you can migrate to platform-first patterns without rebuilding everything from scratch.

### Keeping Code Simple

Small organizations typically start with a mono-repository approach, keeping all infrastructure code in a single Git repository with directories organized by team and environment. This might seem counterintuitive if you're used to microservices architectures, but it solves real problems for small teams.

When everyone is wearing multiple hats, having all infrastructure code in one place means anyone can understand and modify any part of the system. Cross-team collaboration becomes natural when you can see how other teams solve similar problems. Code reviews become learning opportunities rather than gatekeeping exercises.

The repository structure mirrors your team organization, with each team getting their own directory containing development and production configurations. Shared modules live in a common area where teams can contribute reusable patterns as they discover them. This organic approach to standardization works better than top-down mandates when you're moving fast.

Figure 1. Mono-repo structure with directory VCS

### Deployment Workflows That Work

Small teams need deployment workflows that are simple enough for anyone to understand but robust enough for production use. The key is leveraging HCP Terraform's native VCS integration to create predictable, reviewable infrastructure changes.

The workflow follows familiar Git patterns that most developers already understand. When someone needs to change infrastructure, they create a feature branch, make their changes, and open a pull request. HCP Terraform automatically generates a plan showing exactly what will change, and the team can review it just like any code change.

For development environments, you can safely auto-apply changes when pull requests merge, giving teams fast feedback loops. Production changes require manual approval, but the process is the same—review the plan, approve the change, and HCP Terraform handles the execution.

This approach scales naturally as your team grows. The same patterns that work for infrastructure changes also work for application deployments, creating consistency across your entire development workflow.

### Building for Tomorrow

The most important consideration for small organizations is building patterns that will scale as you grow. This doesn't mean over-engineering your current solution, but it does mean making thoughtful choices about foundations.

Start with good practices around naming, tagging, and documentation. These seem like overhead when you're small, but they become critical as you add teams and complexity. Establish clear ownership boundaries between teams, even if those boundaries feel artificial at first. Create runbooks for common operations, even if everyone already knows how to do them.

The goal is to establish muscle memory around good practices while you're still small enough to change quickly. When you eventually need more sophisticated governance or standardization, you'll have the foundations in place to evolve naturally rather than rebuilding from scratch.

Most importantly, resist the temptation to implement enterprise patterns before you need them. The complexity that makes sense for a 50-team organization will slow down a 3-team organization. Focus on velocity and learning, but do it on solid foundations that will support your future growth.

## Medium Organizations: The Platform Transition

Growing from a small to medium organization brings a specific set of challenges that force architectural evolution. You'll typically hit this transition point when you have 4-10 teams and start feeling the pain of shared infrastructure management. The symptoms are familiar: deployment conflicts, inconsistent configurations, and that one person who knows how everything works becoming a bottleneck.

This is when successful organizations make the transition to dedicated platform capabilities. It's not just about adding more people—it's about fundamentally changing how you think about infrastructure ownership and standardization.

### When Shared Infrastructure Breaks Down

The inflection point usually comes when your shared services account becomes a source of friction rather than efficiency. Teams start stepping on each other's deployments, configuration drift creates mysterious issues, and what used to be simple changes now require coordination across multiple teams.

At the same time, you're likely facing new requirements that your simple patterns can't easily accommodate. Maybe you need SOC 2 compliance, or you're expanding to multiple regions, or you've hired security-conscious engineers who want better isolation between environments.

These pressures create the business case for platform investment. Instead of every team solving infrastructure problems independently, you can justify dedicated platform engineers who focus on building shared capabilities that all teams can leverage.

### The Platform Team Emergence

The transition to medium organization patterns typically starts with hiring your first dedicated platform engineer or having an existing team member focus full-time on infrastructure. This person becomes responsible for the shared services that were previously managed collaboratively.

But more importantly, they start thinking about infrastructure as a product with internal customers. Instead of just maintaining shared resources, they begin building standardized patterns that teams can self-service. This shift from "shared infrastructure" to "platform services" is what enables the next phase of growth.

The platform team's first priority is usually breaking apart that monolithic shared services account. Networking, security, CI/CD, and identity management each get their own account with clear ownership and interfaces. This separation allows for better isolation, clearer ownership, and more granular access control.

### Rethinking Project Organization

The move to platform-first patterns requires rethinking how you organize HCP Terraform projects. Instead of each team managing their complete stack, you create clear separation between platform infrastructure and application workloads.

The platform team gets their own project containing all the foundational infrastructure that other teams depend on. This includes networking, security services, CI/CD infrastructure, and identity management. Each of these gets its own workspace and AWS account, allowing for specialized management and clear ownership.

Application teams still get their own projects, but now they consume platform services rather than managing everything themselves. This creates natural interfaces between platform and application concerns, with the platform team exposing well-defined outputs that application teams can consume as inputs.

This separation enables both teams to move independently within their domains. The platform team can upgrade networking infrastructure without affecting application deployments, while application teams can iterate on their services without worrying about underlying platform changes.

### Breaking Apart the Mono-Repo

As your organization grows, the mono-repo that worked well for small teams starts showing strain. Different teams want to move at different speeds, and changes to shared modules affect everyone. The solution is transitioning to multiple repositories with clear ownership boundaries.

The platform team gets dedicated repositories for their infrastructure and a shared module library where they publish standardized patterns. Application teams get their own repositories where they can iterate quickly without affecting other teams. This separation allows for different release cycles and reduces the coordination overhead that was slowing everyone down.

The key insight is that modules become the interface between platform and application teams. Instead of sharing code directly, the platform team publishes tested, versioned modules that application teams can consume. This creates stability for application teams while giving the platform team freedom to evolve their implementations.

Taking as example the FinTechCorp organization, you can see the following setup and separation of responsibilities:

- **Platform team maintains core configurations** in dedicated repositories such as Shared-Services, Platform-Baselines, consequently tied to a Project. In these workspaces, Platform team defines governance and management infrastructure for the whole organization such as identity, shared network, policies, log management etc.
- **Platform team defines baseline configuration** for common infrastructure resources such as S3, RDS, EC2, each located in a shared repository terraform-modules under multiple directories or organized as one dedicated repository per one module such as terraform-modules-compute, terraform-aws-db. The modules are parametrized so that Application teams can leverage and customize the configuration. Platform Team can share terraform-modules via HCP Terraform Private Registry.
- **Each Application team receives their own dedicated repository** (e.g., payments-app), where they can build and manage their specific services by environment. They reference the versioned terraform-modules from the HCP Terraform Private repository when building their configurations.

Figure 2: Multi-repo structure for Medium-Large organizaiton

### Evolving Deployment Patterns

The transition to medium organization patterns requires more sophisticated deployment workflows. Platform changes now affect multiple teams, so they need different approval processes than application changes. You'll typically implement hybrid workflows where platform and application teams follow different patterns optimized for their specific needs.

Platform teams adopt more conservative deployment practices with manual approval gates and scheduled maintenance windows. When a networking change affects all application teams, you need coordination and communication that wasn't necessary when everything was self-contained.

Application teams, meanwhile, can move faster within their boundaries. They can auto-deploy to development environments and self-service their way through testing and production deployments, as long as they're consuming stable platform interfaces.

### Building Platform Capabilities

Medium organizations start developing real platform capabilities beyond just shared infrastructure. This includes centralized security monitoring, structured cost allocation, and standardized compliance patterns. The platform team becomes responsible for capabilities that would be too expensive for each application team to implement independently.

The key is building these capabilities incrementally rather than trying to implement everything at once. Start with the pain points that are affecting multiple teams, and gradually expand the platform's scope as you prove value and build trust with your internal customers.

## Large Organizations: Enterprise Scale Complexity

The transition to large organization patterns typically happens when you cross two critical thresholds: multiple business units with distinct requirements, and regulatory or compliance demands that require formal governance frameworks. This isn't just about having more teams—it's about managing fundamentally different types of complexity.

Large organizations face challenges that medium organizations don't encounter. Different business units may have conflicting requirements, regulatory frameworks that demand specific controls, and stakeholders who need different levels of visibility and governance. The platform capabilities that worked for a single product organization need to evolve into enterprise-grade services that can support diverse business needs.

### When Business Units Drive Architecture

The defining characteristic of large organizations is the emergence of multiple business units, each with their own product portfolios, compliance requirements, and operational constraints. A consumer banking division has different regulatory requirements than a commercial lending division, even within the same company.

This business unit complexity drives architectural decisions in ways that pure team count doesn't. You need account structures that provide clear isolation between business units, governance frameworks that can accommodate different compliance requirements, and platform services that can serve diverse internal customers without creating bottlenecks.

The platform team evolves into a platform organization with specialized capabilities. Instead of generalists who handle all infrastructure concerns, you develop teams focused on specific domains: core infrastructure, security and compliance, data platforms, and developer experience. This specialization allows you to build enterprise-grade capabilities while maintaining the agility that made your platform successful.

### Choosing Your Governance Foundation

Large organizations face a critical decision about their governance foundation that smaller organizations don't encounter: whether to build on AWS Control Tower or implement custom governance frameworks. This choice depends heavily on your existing constraints, regulatory requirements, and organizational context.

**AWS Control Tower** provides the fastest path to enterprise-grade governance for organizations starting fresh or willing to adapt their practices to AWS best practices. It offers automated account provisioning, centralized logging, and baseline security controls that work well for most enterprise scenarios. Organizations in regulated industries often find that Control Tower's built-in compliance frameworks accelerate their certification processes.

However, large organizations with existing governance frameworks, complex regulatory requirements, or significant legacy constraints may need more flexibility than Control Tower provides. Some scenarios where custom approaches make sense include:

- **Existing enterprise governance tools** that need to integrate with AWS account management
- **Complex regulatory requirements** that require specific account structures or controls not supported by Control Tower
- **Multi-cloud strategies** where AWS account governance needs to align with governance frameworks spanning other cloud providers
- **Acquisition-heavy organizations** where inherited AWS accounts need to be integrated into existing governance patterns

### Account Organization Patterns

Regardless of your governance foundation, large organizations typically implement comprehensive account structures organized around business units rather than individual teams. Each business unit gets its own organizational unit with dedicated development, testing, and production environments.

This structure provides the isolation and governance boundaries that enterprise stakeholders require while enabling business unit-specific policies and controls. A consumer banking division can implement stricter data residency requirements without affecting a payments division's global operations, and each business unit can evolve their practices within the broader enterprise framework.

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

### Project Organization at Scale

Large organizations require hybrid project patterns that can accommodate both centralized platform services and distributed business unit operations. The platform organization maintains centralized projects for foundational services, while each business unit gets dedicated projects that align with their operational boundaries.

This hybrid approach recognizes that different parts of the organization have different needs for autonomy and standardization. The platform organization can maintain tight control over foundational services like networking and security, while business units can move independently within their domains.

The project structure also needs to accommodate specialized platform capabilities like module development and policy management. These get their own dedicated projects where the platform organization can iterate on shared services without affecting business unit operations.

### Enterprise Repository Patterns

Large organizations typically implement comprehensive repository separation with dedicated policy-as-code repositories and private module registries. This goes beyond the multi-repo patterns used by medium organizations to include formal governance and compliance frameworks.

The key addition is policy-as-code repositories where security and compliance teams can define and maintain organizational policies as versioned code. These policies get automatically enforced across all infrastructure deployments, providing the governance frameworks that enterprise stakeholders require.

Business unit repositories are organized around business domains rather than individual teams, reflecting the reality that large organizations need to coordinate across multiple teams within each business unit. This structure supports the cross-team collaboration that's necessary for complex product development while maintaining clear ownership boundaries.

### Enterprise Deployment Workflows

Large organizations require formal change management processes that can accommodate both the velocity needs of development teams and the governance requirements of enterprise stakeholders. This typically means implementing comprehensive CI/CD pipelines with automated security scanning, policy validation, and compliance checks.

Platform changes follow formal RFC processes with multi-stage validation including security, performance, and business impact assessments. These changes often require coordination across multiple business units and may need scheduled maintenance windows to minimize business disruption.

Business unit teams get sophisticated self-service capabilities within their domains, but with comprehensive audit trails and automated policy enforcement. They can move quickly through development and testing environments while still meeting the governance requirements for production deployments.

### Building Enterprise Platform Capabilities

Large organizations develop comprehensive platform capabilities that go far beyond shared infrastructure. This includes automated account vending, policy-as-code frameworks, enterprise observability platforms, and formal risk management processes.

The platform organization typically includes specialized teams focused on different aspects of the enterprise platform: core infrastructure teams that manage foundational services, security teams that develop and enforce policies, DevOps teams that build deployment automation, and data platform teams that provide analytics and machine learning capabilities.

These capabilities are delivered through well-defined service catalogs with clear SLAs and support models. Business units consume platform services through standardized interfaces, allowing them to focus on their business logic while leveraging enterprise-grade infrastructure capabilities.

The key to success at this scale is treating the platform as a product with internal customers, complete with product management, customer support, and continuous improvement processes. This product mindset ensures that platform capabilities evolve to meet changing business needs while maintaining the operational excellence that enterprise stakeholders require.


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
