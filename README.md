<!-- This file was automatically generated by the `build-harness`. Make all changes to `README.yaml` and run `make readme` to rebuild this file. -->
[![README Header][readme_header_img]][readme_header_link]

[![Cloud Posse][logo]](https://cpco.io/homepage)

# terraform-aws-ecs-codepipeline [![Build Status](https://travis-ci.org/cloudposse/terraform-aws-ecs-codepipeline.svg?branch=master)](https://travis-ci.org/cloudposse/terraform-aws-ecs-codepipeline) [![Latest Release](https://img.shields.io/github/release/cloudposse/terraform-aws-ecs-codepipeline.svg)](https://github.com/cloudposse/terraform-aws-ecs-codepipeline/releases/latest) [![Slack Community](https://slack.cloudposse.com/badge.svg)](https://slack.cloudposse.com)


Terraform Module for CI/CD with AWS Code Pipeline using GitHub webhook triggers and Code Build for ECS.


---

This project is part of our comprehensive ["SweetOps"](https://cpco.io/sweetops) approach towards DevOps.
[<img align="right" title="Share via Email" src="https://docs.cloudposse.com/images/ionicons/ios-email-outline-2.0.1-16x16-999999.svg"/>][share_email]
[<img align="right" title="Share on Google+" src="https://docs.cloudposse.com/images/ionicons/social-googleplus-outline-2.0.1-16x16-999999.svg" />][share_googleplus]
[<img align="right" title="Share on Facebook" src="https://docs.cloudposse.com/images/ionicons/social-facebook-outline-2.0.1-16x16-999999.svg" />][share_facebook]
[<img align="right" title="Share on Reddit" src="https://docs.cloudposse.com/images/ionicons/social-reddit-outline-2.0.1-16x16-999999.svg" />][share_reddit]
[<img align="right" title="Share on LinkedIn" src="https://docs.cloudposse.com/images/ionicons/social-linkedin-outline-2.0.1-16x16-999999.svg" />][share_linkedin]
[<img align="right" title="Share on Twitter" src="https://docs.cloudposse.com/images/ionicons/social-twitter-outline-2.0.1-16x16-999999.svg" />][share_twitter]


[![Terraform Open Source Modules](https://docs.cloudposse.com/images/terraform-open-source-modules.svg)][terraform_modules]



It's 100% Open Source and licensed under the [APACHE2](LICENSE).







We literally have [*hundreds of terraform modules*][terraform_modules] that are Open Source and well-maintained. Check them out!







## Usage


**IMPORTANT:** The `master` branch is used in `source` just as an example. In your code, do not pin to `master` because there may be breaking changes between releases.
Instead pin to the release tag (e.g. `?ref=tags/x.y.z`) of one of our [latest releases](https://github.com/cloudposse/terraform-aws-ecs-codepipeline/releases).



### Trigger on GitHub Push

In this example, we'll trigger the pipeline anytime the `master` branch is updated.
```hcl
module "ecs_push_pipeline" {
  source             = "git::https://github.com/cloudposse/terraform-aws-ecs-codepipeline.git?ref=master"
  name               = "app"
  namespace          = "eg"
  stage              = "staging"
  github_oauth_token = "xxxxxxxxxxxxxx"
  repo_owner         = "cloudposse"
  repo_name          = "example"
  branch             = "master"
  service_name       = "example"
  ecs_cluster_name   = "example-ecs-cluster"
  privileged_mode    = "true"
}
```

### Trigger on GitHub Releases

In this example, we'll trigger anytime a new GitHub release is cut by setting the even type to `release` and using the `json_path` to *exactly* match an `action` of `published`.

```hcl
module "ecs_release_pipeline" {
  source                      = "git::https://github.com/cloudposse/terraform-aws-ecs-codepipeline.git?ref=master"
  name                        = "app"
  namespace                   = "eg"
  stage                       = "staging"
  github_oauth_token          = "xxxxxxxxxxxxxx"
  repo_owner                  = "cloudposse"
  repo_name                   = "example"
  branch                      = "master"
  service_name                = "example"
  ecs_cluster_name            = "example-ecs-cluster"
  privileged_mode             = "true"
  github_webhook_events       = ["release"]
  webhook_filter_json_path    = "$.action"
  webhook_filter_match_equals = "published"
}
```
(Thanks to [Stack Overflow](https://stackoverflow.com/questions/52516087/trigger-aws-codepipeline-by-github-release-webhook#comment91997146_52524711))




## Examples

Complete usage can be seen in the [terraform-aws-ecs-web-app](https://github.com/cloudposse/terraform-aws-ecs-web-app/blob/master/main.tf) module.

## Example Buildspec

Here's an example `buildspec.yaml`. Stick this in the root of your project repository.

```yaml
version: 0.2
phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws --version
      - eval $(aws ecr get-login --region $AWS_DEFAULT_REGION --no-include-email)
      - REPOSITORY_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME
      - IMAGE_TAG=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...
      - REPO_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME
      - docker pull $REPO_URI:latest || true
      - docker build --cache-from $REPO_URI:latest --tag $REPO_URI:latest --tag $REPO_URI:$IMAGE_TAG .
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker images...
      - REPO_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME
      - docker push $REPO_URI:latest
      - docker push $REPO_URI:$IMAGE_TAG
      - echo Writing image definitions file...
      - printf '[{"name":"%s","imageUri":"%s"}]' "$CONTAINER_NAME" "$REPO_URI:$IMAGE_TAG" | tee imagedefinitions.json
artifacts:
  files: imagedefinitions.json
```



## Makefile Targets
```
Available targets:

  help                                Help screen
  help/all                            Display help for all targets
  help/short                          This help short screen

```
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| attributes | Additional attributes (e.g. `policy` or `role`) | list | `<list>` | no |
| aws_account_id | AWS Account ID. Used as CodeBuild ENV variable when building Docker images. [For more info](http://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html) | string | `` | no |
| aws_region | AWS Region, e.g. us-east-1. Used as CodeBuild ENV variable when building Docker images. [For more info](http://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html) | string | `` | no |
| badge_enabled | Generates a publicly-accessible URL for the projects build badge. Available as badge_url attribute when enabled. | string | `false` | no |
| branch | Branch of the GitHub repository, _e.g._ `master` | string | - | yes |
| build_compute_type | `CodeBuild` instance size. Possible values are: `BUILD_GENERAL1_SMALL` `BUILD_GENERAL1_MEDIUM` `BUILD_GENERAL1_LARGE` | string | `BUILD_GENERAL1_SMALL` | no |
| build_image | Docker image for build environment, _e.g._ `aws/codebuild/docker:docker:17.09.0` | string | `aws/codebuild/docker:17.09.0` | no |
| build_timeout | How long in minutes, from 5 to 480 (8 hours), for AWS CodeBuild to wait until timing out any related build that does not get marked as completed. | string | `60` | no |
| buildspec | Declaration to use for building the project. [For more info](http://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html) | string | `` | no |
| delimiter | Delimiter to be used between `name`, `namespace`, `stage`, etc. | string | `-` | no |
| ecs_cluster_name | ECS Cluster Name | string | - | yes |
| enabled | Enable `CodePipeline` creation | string | `true` | no |
| pipeline_bucket_lifecycle_enabled | Enable bucket lifecycle rules. | string | `false` | no |
| pipeline_bucket_lifecycle_expiration_days | The amount of days before expiring a bucket object | string | `60` | no |
| environment_variables | A list of maps, that contain both the key 'name' and the key 'value' to be used as additional environment variables for the build. | list | `<list>` | no |
| github_oauth_token | GitHub OAuth Token with permissions to access private repositories | string | - | yes |
| github_webhook_events | A list of events which should trigger the webhook. See a list of [available events](https://developer.github.com/v3/activity/events/types/) | list | `<list>` | no |
| github_webhooks_token | GitHub OAuth Token with permissions to create webhooks. If not provided, can be sourced from the `GITHUB_TOKEN` environment variable | string | `` | no |
| image_repo_name | ECR repository name to store the Docker image built by this module. Used as CodeBuild ENV variable when building Docker images. [For more info](http://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html) | string | `UNSET` | no |
| image_tag | Docker image tag in the ECR repository, e.g. 'latest'. Used as CodeBuild ENV variable when building Docker images. [For more info](http://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html) | string | `latest` | no |
| name | Solution name, e.g. 'app' or 'jenkins' | string | `app` | no |
| namespace | Namespace, which could be your organization name, e.g. 'cp' or 'cloudposse' | string | `global` | no |
| poll_source_changes | Periodically check the location of your source content and run the pipeline if changes are detected | string | `false` | no |
| privileged_mode | If set to true, enables running the Docker daemon inside a Docker container on the CodeBuild instance. Used when building Docker images | string | `false` | no |
| repo_name | GitHub repository name of the application to be built and deployed to ECS. | string | - | yes |
| repo_owner | GitHub Organization or Username. | string | - | yes |
| s3_bucket_force_destroy | A boolean that indicates all objects should be deleted from the CodePipeline artifact store S3 bucket so that the bucket can be destroyed without error | string | `false` | no |
| service_name | ECS Service Name | string | - | yes |
| stage | Stage, e.g. 'prod', 'staging', 'dev', or 'test' | string | `default` | no |
| tags | Additional tags (e.g. `map('BusinessUnit', 'XYZ')` | map | `<map>` | no |
| webhook_authentication | The type of authentication to use. One of IP, GITHUB_HMAC, or UNAUTHENTICATED. | string | `GITHUB_HMAC` | no |
| webhook_enabled | Set to false to prevent the module from creating any webhook resources | string | `true` | no |
| webhook_filter_json_path | The JSON path to filter on. | string | `$.ref` | no |
| webhook_filter_match_equals | The value to match on (e.g. refs/heads/{Branch}) | string | `refs/heads/{Branch}` | no |
| webhook_target_action | The name of the action in a pipeline you want to connect to the webhook. The action must be from the source (first) stage of the pipeline. | string | `Source` | no |
| code_deploy_application_name | Code Deploy application name. | string | `` | no |
| code_deploy_deployment_group_name | Code Deploy deployment group name. | string | `` | no |
| code_deploy_sns_topic_arn | The SNS topic to send notification messages. | string | `` | no |
| code_deploy_lambda_hook_arns | The lambda arns this code depoloy app should be permitted to access. | string | `` | no |

## Outputs

| Name | Description |
|------|-------------|
| badge_url | The URL of the build badge when badge_enabled is enabled |
| webhook_id | The CodePipeline webhook's ARN. |
| webhook_url | The CodePipeline webhook's URL. POST events to this endpoint to trigger the target |
| default_role_arn | The CodePipeline role arn |




## Share the Love

Like this project? Please give it a ★ on [our GitHub](https://github.com/cloudposse/terraform-aws-ecs-codepipeline)! (it helps us **a lot**)

Are you using this project or any of our other projects? Consider [leaving a testimonial][testimonial]. =)


## Related Projects

Check out these related projects.

- [terraform-aws-alb](https://github.com/cloudposse/terraform-aws-alb) - Terraform module to provision a standard ALB for HTTP/HTTP traffic
- [terraform-aws-alb-ingress](https://github.com/cloudposse/terraform-aws-alb-ingress) - Terraform module to provision an HTTP style ingress rule based on hostname and path for an ALB
- [terraform-aws-codebuild](https://github.com/cloudposse/terraform-aws-codebuild) - Terraform Module to easily leverage AWS CodeBuild for Continuous Integration
- [terraform-aws-ecr](https://github.com/cloudposse/terraform-aws-ecr) - Terraform Module to manage Docker Container Registries on AWS ECR
- [terraform-aws-ecs-alb-service-task](https://github.com/cloudposse/terraform-aws-ecs-alb-service-task) - Terraform module which implements an ECS service which exposes a web service via ALB.
- [terraform-aws-ecs-container-definition](https://github.com/cloudposse/terraform-aws-ecs-container-definition) - Terraform module to generate well-formed JSON documents that are passed to the aws_ecs_task_definition Terraform resource
- [terraform-aws-lb-s3-bucket](https://github.com/cloudposse/terraform-aws-lb-s3-bucket) - Terraform module to provision an S3 bucket with built in IAM policy to allow AWS Load Balancers to ship access logs.




## References

For additional context, refer to some of these links.

- [aws_codepipeline_webhook](https://www.terraform.io/docs/providers/aws/r/codepipeline_webhook.html) - Provides a CodePipeline Webhook


## Help

**Got a question?**

File a GitHub [issue](https://github.com/cloudposse/terraform-aws-ecs-codepipeline/issues), send us an [email][email] or join our [Slack Community][slack].

[![README Commercial Support][readme_commercial_support_img]][readme_commercial_support_link]

## Commercial Support

Work directly with our team of DevOps experts via email, slack, and video conferencing.

We provide [*commercial support*][commercial_support] for all of our [Open Source][github] projects. As a *Dedicated Support* customer, you have access to our team of subject matter experts at a fraction of the cost of a full-time engineer.

[![E-Mail](https://img.shields.io/badge/email-hello@cloudposse.com-blue.svg)][email]

- **Questions.** We'll use a Shared Slack channel between your team and ours.
- **Troubleshooting.** We'll help you triage why things aren't working.
- **Code Reviews.** We'll review your Pull Requests and provide constructive feedback.
- **Bug Fixes.** We'll rapidly work to fix any bugs in our projects.
- **Build New Terraform Modules.** We'll [develop original modules][module_development] to provision infrastructure.
- **Cloud Architecture.** We'll assist with your cloud strategy and design.
- **Implementation.** We'll provide hands-on support to implement our reference architectures.



## Terraform Module Development

Are you interested in custom Terraform module development? Submit your inquiry using [our form][module_development] today and we'll get back to you ASAP.


## Slack Community

Join our [Open Source Community][slack] on Slack. It's **FREE** for everyone! Our "SweetOps" community is where you get to talk with others who share a similar vision for how to rollout and manage infrastructure. This is the best place to talk shop, ask questions, solicit feedback, and work together as a community to build totally *sweet* infrastructure.

## Newsletter

Signup for [our newsletter][newsletter] that covers everything on our technology radar.  Receive updates on what we're up to on GitHub as well as awesome new projects we discover.

## Contributing

### Bug Reports & Feature Requests

Please use the [issue tracker](https://github.com/cloudposse/terraform-aws-ecs-codepipeline/issues) to report any bugs or file feature requests.

### Developing

If you are interested in being a contributor and want to get involved in developing this project or [help out](https://cpco.io/help-out) with our other projects, we would love to hear from you! Shoot us an [email][email].

In general, PRs are welcome. We follow the typical "fork-and-pull" Git workflow.

 1. **Fork** the repo on GitHub
 2. **Clone** the project to your own machine
 3. **Commit** changes to your own branch
 4. **Push** your work back up to your fork
 5. Submit a **Pull Request** so that we can review your changes

**NOTE:** Be sure to merge the latest changes from "upstream" before making a pull request!


## Copyright

Copyright © 2017-2019 [Cloud Posse, LLC](https://cpco.io/copyright)



## License

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

See [LICENSE](LICENSE) for full details.

    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

      https://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.









## Trademarks

All other trademarks referenced herein are the property of their respective owners.

## About

This project is maintained and funded by [Cloud Posse, LLC][website]. Like it? Please let us know by [leaving a testimonial][testimonial]!

[![Cloud Posse][logo]][website]

We're a [DevOps Professional Services][hire] company based in Los Angeles, CA. We ❤️  [Open Source Software][we_love_open_source].

We offer [paid support][commercial_support] on all of our projects.

Check out [our other projects][github], [follow us on twitter][twitter], [apply for a job][jobs], or [hire us][hire] to help with your cloud strategy and implementation.



### Contributors

|  [![Erik Osterman][osterman_avatar]][osterman_homepage]<br/>[Erik Osterman][osterman_homepage] | [![Igor Rodionov][goruha_avatar]][goruha_homepage]<br/>[Igor Rodionov][goruha_homepage] | [![Andriy Knysh][aknysh_avatar]][aknysh_homepage]<br/>[Andriy Knysh][aknysh_homepage] | [![Sarkis Varozian][sarkis_avatar]][sarkis_homepage]<br/>[Sarkis Varozian][sarkis_homepage] |
|---|---|---|---|

  [osterman_homepage]: https://github.com/osterman
  [osterman_avatar]: https://github.com/osterman.png?size=150
  [goruha_homepage]: https://github.com/goruha
  [goruha_avatar]: https://github.com/goruha.png?size=150
  [aknysh_homepage]: https://github.com/aknysh
  [aknysh_avatar]: https://github.com/aknysh.png?size=150
  [sarkis_homepage]: https://github.com/sarkis
  [sarkis_avatar]: https://github.com/sarkis.png?size=150



[![README Footer][readme_footer_img]][readme_footer_link]
[![Beacon][beacon]][website]

  [logo]: https://cloudposse.com/logo-300x69.svg
  [docs]: https://cpco.io/docs
  [website]: https://cpco.io/homepage
  [github]: https://cpco.io/github
  [jobs]: https://cpco.io/jobs
  [hire]: https://cpco.io/hire
  [slack]: https://cpco.io/slack
  [linkedin]: https://cpco.io/linkedin
  [twitter]: https://cpco.io/twitter
  [testimonial]: https://cpco.io/leave-testimonial
  [newsletter]: https://cpco.io/newsletter
  [email]: https://cpco.io/email
  [commercial_support]: https://cpco.io/commercial-support
  [we_love_open_source]: https://cpco.io/we-love-open-source
  [module_development]: https://cpco.io/module-development
  [terraform_modules]: https://cpco.io/terraform-modules
  [readme_header_img]: https://cloudposse.com/readme/header/img?repo=cloudposse/terraform-aws-ecs-codepipeline
  [readme_header_link]: https://cloudposse.com/readme/header/link?repo=cloudposse/terraform-aws-ecs-codepipeline
  [readme_footer_img]: https://cloudposse.com/readme/footer/img?repo=cloudposse/terraform-aws-ecs-codepipeline
  [readme_footer_link]: https://cloudposse.com/readme/footer/link?repo=cloudposse/terraform-aws-ecs-codepipeline
  [readme_commercial_support_img]: https://cloudposse.com/readme/commercial-support/img?repo=cloudposse/terraform-aws-ecs-codepipeline
  [readme_commercial_support_link]: https://cloudposse.com/readme/commercial-support/link?repo=cloudposse/terraform-aws-ecs-codepipeline
  [share_twitter]: https://twitter.com/intent/tweet/?text=terraform-aws-ecs-codepipeline&url=https://github.com/cloudposse/terraform-aws-ecs-codepipeline
  [share_linkedin]: https://www.linkedin.com/shareArticle?mini=true&title=terraform-aws-ecs-codepipeline&url=https://github.com/cloudposse/terraform-aws-ecs-codepipeline
  [share_reddit]: https://reddit.com/submit/?url=https://github.com/cloudposse/terraform-aws-ecs-codepipeline
  [share_facebook]: https://facebook.com/sharer/sharer.php?u=https://github.com/cloudposse/terraform-aws-ecs-codepipeline
  [share_googleplus]: https://plus.google.com/share?url=https://github.com/cloudposse/terraform-aws-ecs-codepipeline
  [share_email]: mailto:?subject=terraform-aws-ecs-codepipeline&body=https://github.com/cloudposse/terraform-aws-ecs-codepipeline
  [beacon]: https://ga-beacon.cloudposse.com/UA-76589703-4/cloudposse/terraform-aws-ecs-codepipeline?pixel&cs=github&cm=readme&an=terraform-aws-ecs-codepipeline
