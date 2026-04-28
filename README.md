# 63Klabs Atlantis DevOps Platform Administration for Organizations

Templates and Scripts to provision developer roles, configurations, scripts, and templates within your organization on AWS.

Whether you are learning on your own, part of a small team, teaching a class, or providing platform engineering services at your small to medium-sized organization, the [63Klabs Atlantis DevOps Templates and Scripts Platform](https://github.com/63klabs/atlantis) makes it easy to improve your developer experience.

- **Basic:** Start with maintaining a [SAM Configuration repository](https://github.com/63klabs/atlantis-sam-config-scripts) to manage your developer and deployment infrastructure all backed by the templates, scripts, starter code and MCP server maintained by 63Klabs.
- **Hybrid:** Develop and host your own templates, scripts, and/or MCP server while also utilizing templates and scripts provided by 63Klabs.
- **Fully Self-Host:** Take it all on and self-host all templates, scripts, starter code, and the MCP Server.

> NOTE: The provisioning process is still undergoing development. However, we hope to provide enough context and documentation to get started. You may need to fill in some gaps on your own.

## Deployment and Management

The 63Klabs Atlantis DevOps Platform is built on top of Atlantis, from pipeline templates that deploy starter code, to the actual templates managed by the platform engineers, to the Atlantis MCP Server built using Starter #02. Once you understand the basics of using the SAM Configuration repository, the templates, and deployments, you'll understand how everything works together.

However, there is still a chicken and egg problem.

Begin by creating a repository and then deploying account-wide supporting infrastructure.

## Keep it Simple to Start

Your organization may be one AWS Account, or it may be many Organizational Accounts. Even personal accounts may consist of one or more organizational accounts.

You can manage a single, central repository in a central account that deploys to multiple accounts, or you can have a central repository in each AWS Account.

The choice is yours.

More than likely you'll want to start by setting up the SAM Configuration Repository in a single `sandbox` or `demo` account to experiment.

## Account Set Up

Once the account is established, you'll want to set up account infrastructure:

1. [SAM Config Repository for the AWS Account](./sam-config-repo/README.md)
2. [Account-Wide Set-Up: Roles, Policies, and Resources](./account-wide-set-up/README.md)
   - Account-Wide Infrastructure
   - Prefix-Based Infrastructure
3. [Application Support Infrastructure](./application-support-infrastructure/README.md)
   - Cache-Data storage for application cache
   - S3 buckets to receive logs from S3 access and CloudFront
   - If hosting Static Content fronted by CloudFront, a CloudFront cache invalidation service
4. Direct developers towards:
   - SAM Config repository docs including script and common parameter usage
   - [Atlantis tutorials](https://github.com/63klabs/atlantis-tutorials)
   - [Atlantis MCP server](https://mcp.atlantis.63klabs.net)

## Self-Hosted Platform

You can self-host as much or as little as you want. However, there are a lot of moving parts and a step learning curve. That's not because Atlantis is hard, it's because Platform Engineering and managing a FULL DevOps pipeline complete with templates, code, and deployment scripts is HARD.

Before self-hosting, it is recommended you (or someone on your team) has at least 1 year of advanced CloudFormation and CodePipeline experience (ability to code templates from scratch, troubleshoot, and has knowledge of all the ins and outs) and at least 3 months of experience using 63Klabs Atlantis DevOps Templates and Scripts, and has reviewed the templates and scripts under the hood.

The account set-up described above works with the 63klabs bucket right **Out-Of-The-Box**, but you can move to a self-hosted platform if you want.

- Set up an S3 bucket accessible to your developers (read-only) Be sure to use the `bucketname/namespace` naming as everything including the MCP server flows from that. It also helps in using a hybrid `63klabs/atlantis` and `yourorg/atlantis` and `yourorg/custom` approach.
- Use GitHub actions to package your own templates and application starters and save to the S3 bucket (GitHub workflow is already included in the template repository, you can use the CodeBuild only pipeline and modify the GitHub actions workflow to deploy via CodePipeline as well)
- Add the S3 bucket as an option (or only option) for SAM Config Scripts to access templates and starters (see the defaults/settings.json file in your SAM Config Repo)
- If you will host your own SAM Config repository template, set the settings.json to pull updates from there.
- Deploy your own Atlantis MCP server and configure to include, or only use, your resources.
