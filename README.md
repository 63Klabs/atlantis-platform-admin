# Atlantis Platform Administration for Organizations

Templates and Scripts to provision developer roles, configurations, scripts, and templates within your organization on AWS.

Whether you are learning on your own, part of a small team, teaching a class, or providing platform engineering services at your small to medium-sized organization, the [Atlantis Templates and Scripts Platform](https://github.com/63klabs/atlantis) makes it easy to improve your developer experience.

- **Basic:** Start with maintaining a [SAM Configuration repository](https://github.com/63klabs/atlantis-sam-config-scripts) to manage your developer and deployment infrastructure all backed by the templates, scripts, startercode and MCP server maintained by 63Klabs.
- **Hybrid:** Develop and host your own templates, scripts, and/or MCP server while also utilizing templates and scripts provided by 63Klabs.
- **Fully Self-Host:** Take it all on and self-host all templates, scripts, starter code, and the MCP Server.

> NOTE: The provisioning process is still undergoing development. However, we hope to provide enough context and documentation to get started.

## Deployment and Management

The Atlantis Platform is built on top of Altlantis, from pipeline templates that deploy starter code and templates managed by the platform engineers to the Atlantis MCP Server built using Starter #02, once you understand the basics of using the SAM Configuration repository, you'll understand how everything works together.

However, there is still a chicken and egg problem.

Begin by creating a repository and developer roles.

## Create a SAM Configuration Script Repository for your Organization

Your organization may be one AWS Account, or it may be many Organizational Accounts. Even personal accounts may consist of one or more organizational accounts.

You can manage a single, central repository in a central account that deploys to multiple accounts, or you can have a central repository in each AWS Account.

The choice is yours.

More than likely you'll want to start setting up the SAM Configuration Repository in a `sandbox` or `demo` account.

[Get Started Setting Up the SAM Config Repository for Your Organization](./sam-config-repo/README.md)

## Supporting Infrastructure

Once the configuration repository is established, you'll want to set up supporting infrastructure:

1. API Gateway Log Policy
2. Managed policies to attach to pipelines
3. Cache-Data storage for application cache
4. S3 buckets to receive logs from S3 access and CloudFront
5. If hosting Static Content fronted by CloudFront, a CloudFront cache invalidation service
6. Documentation for developers on accessing the SAM Config repository including `Prefix` and `Permissions Boundaries`, access to the [Atlantis tutorials](https://github.com/63klabs/atlantis-tutorials) and [Atlantis MCP server](https://mcp.atlantis.63klabs.net).

## Self-Hosted Platform

You can self-host as much or as little as you want.

- Set up an S3 bucket accessible to your developers (read-only)
- Use GitHub actions to package your own templates and application starters and save to the S3 bucket (GitHub workflow is already included in the template repository, you can use the CodeBuild only pipeline and modify the GitHub actions workflow to deploy via CodePipeline as well)
- Add the S3 bucket as an option (or only option) for SAM Config Scripts to access templates and starters (see the defaults/settings.json file in your SAM Config Repo)
- If you will host your own SAM Config repository template, set the settings.json to pull updates from there.
- Deploy your own Atlantis MCP server and configure to include, or only use, your resources.
