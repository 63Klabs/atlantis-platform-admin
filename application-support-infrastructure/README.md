# Application Support Infrastructure

As long as they were enabled, several supporting resources such as an artifact bucket, logging bucket, API Gateway logging, and cache storage were created by the account-wide and prefix-based CloudFormation stacks deployed during the [account-wide set up](../account-wide-set-up/README.md).

Adding these resources follow several best practices and ensure the Atlantis Templates and Scripts Platform has what is needed to deliver a complete cloud development and deployment platform.

Additional account infrastructure will be added in the future.

If you plan on serving websites from S3 through CloudFront, a cache-invalidator application stack is another recommended addition that will have to be installed separately.

## CloudFront Cache Invalidation

[Serverless CloudFront Cache-Invalidator](https://github.com/63Klabs/serverless-cloudfront-cache-invalidation)