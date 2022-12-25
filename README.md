
<!-- markdownlint-disable -->
# github-actions-workflows-docker-ecr-spacelift

 [![Latest Release](https://img.shields.io/github/release/cloudposse/github-actions-workflows-docker-ecr-spacelift.svg)](https://github.com/cloudposse/github-actions-workflows-docker-ecr-spacelift/releases/latest) [![Slack Community](https://slack.cloudposse.com/badge.svg)](https://slack.cloudposse.com)
<!-- markdownlint-restore -->

[![README Header][readme_header_img]][readme_header_link]

[![Cloud Posse][logo]](https://cpco.io/homepage)

<!--




  ** DO NOT EDIT THIS FILE
  **
  ** This file was automatically generated by the `build-harness`.
  ** 1) Make all changes to `README.yaml`
  ** 2) Run `make init` (you only need to do this once)
  ** 3) Run`make readme` to rebuild this file.
  **
  ** (We maintain HUNDREDS of open source projects. This is how we maintain our sanity.)
  **





-->

Release workflow for Dockerized application using ECR and deploying with Spacelift.

---

This project is part of our comprehensive ["SweetOps"](https://cpco.io/sweetops) approach towards DevOps.
[<img align="right" title="Share via Email" src="https://docs.cloudposse.com/images/ionicons/ios-email-outline-2.0.1-16x16-999999.svg"/>][share_email]
[<img align="right" title="Share on Google+" src="https://docs.cloudposse.com/images/ionicons/social-googleplus-outline-2.0.1-16x16-999999.svg" />][share_googleplus]
[<img align="right" title="Share on Facebook" src="https://docs.cloudposse.com/images/ionicons/social-facebook-outline-2.0.1-16x16-999999.svg" />][share_facebook]
[<img align="right" title="Share on Reddit" src="https://docs.cloudposse.com/images/ionicons/social-reddit-outline-2.0.1-16x16-999999.svg" />][share_reddit]
[<img align="right" title="Share on LinkedIn" src="https://docs.cloudposse.com/images/ionicons/social-linkedin-outline-2.0.1-16x16-999999.svg" />][share_linkedin]
[<img align="right" title="Share on Twitter" src="https://docs.cloudposse.com/images/ionicons/social-twitter-outline-2.0.1-16x16-999999.svg" />][share_twitter]




It's 100% Open Source and licensed under the [APACHE2](LICENSE).












## Introduction

Use provided [GitHub Actions reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows) 
to implement consistent release workflow for any kind of application that use 
* [Docker](https://docs.docker.com/engine/reference/builder/) for developing, shipping, and running,
* [ECR](https://aws.amazon.com/ecr/) to store the Docker images
* [Spacelift](https://spacelift.io/) as CD platform 

## Preconditions

Application repository reference this reusable workflows should follows this structure

```
  .
  ├── Dockerfile
  └── test/
      ├── docker-compose.yml
      └── test.sh
```











<!-- markdownlint-disable -->
## Workflows

| Name | Description |
|------|-------------|
| [Features branch (Pull request) workflow ](features-branch-pull-request-workflow) | Build, test Docker image, deploy it with Spacelift to QA environments depends of the PR labels   |
| [Hotfix branch (Pull request into release/\* branches) workflow ](hotfix-branch-pull-request-into-release-branches-workflow) | Build, test Docker image, deploy it with Spacelift to `hotfix` environment depends of PR labels   |
| [Hotfix workflow enable](hotfix-workflow-enable) | For each new release create `release/{version}` branch that is key puzzle for `hotfix` workflow. |
| [Hotfix release workflow](hotfix-release-workflow) | Build, test Docker image, deploy it with Spacelift on `production` environment and reintegrate `hotfix` into the main branch   |
| [Main branch workflow](main-branch-workflow) | Build, test Docker image, deploy it with Spacelift on `dev` environment and draft new release   |
| [Release workflow ](release-workflow) | Promote existing Docker image to release version, deploy it with Spacelift to `staging` and then `production` environments.   |




## Features branch (Pull request) workflow 

Build, test Docker image, deploy it with Spacelift to QA environments depends of the PR labels  

### Usage 

Create in your repo  __`.github/workflows/feature.yaml`__

```yaml
  name: Feature Branch
  on:
    pull_request:
      branches: [ 'master' ]
      types: [opened, synchronize, reopened, closed, labeled, unlabeled]
  
  permissions:
    pull-requests: write
    deployments: write
    id-token: write
    contents: read
  
  jobs:
    do:
      uses: cloudposse/github-actions-workflows-docker-ecr-spacelift/.github/workflows/feature-branch.yml@main
      with:
        organization: "${{ github.event.repository.owner.login }}"
        repository: "${{ github.event.repository.name }}"
        open: ${{ github.event.pull_request.state == 'open' }}
        labels: ${{ toJSON(github.event.pull_request.labels.*.name) }}
        ref: ${{ github.event.pull_request.head.ref  }}
        spacelift-organization: "${{ github.event.repository.owner.login }}"
      secrets:
        github-private-actions-pat: "${{ secrets.PUBLIC_REPO_ACCESS_TOKEN }}"
        registry: "${{ secrets.ECR_REGISTRY }}"
        secret-outputs-passphrase: "${{ secrets.GHA_SECRET_OUTPUT_PASSPHRASE }}"
        ecr-region: "${{ secrets.ECR_REGION }}"
        ecr-iam-role: "${{ secrets.ECR_IAM_ROLE }}"
        spacelift-api-key-id: ${{ secrets.SPACELIFT_API_KEY_ID }}
        spacelift-api-key-secret: ${{ secrets.SPACELIFT_API_KEY_SECRET }}
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| labels | Pull Request labels | string | ${{ toJSON(github.event.pull\_request.labels.\*.name) }} | false |
| open | Pull Request open/close state. Set true if opened | boolean | ${{ github.event.pull\_request.state == 'open' }} | false |
| organization | Repository owner organization (ex. acme for repo acme/example) | string | ${{ github.event.repository.owner.login }} | false |
| ref | The fully-formed ref of the branch or tag that triggered the workflow run | string | ${{ github.event.pull\_request.head.ref }} | false |
| repository | Repository name (ex. example for repo acme/example) | string | ${{ github.event.repository.name }} | false |
| spacelift-organization | Spacelift organization name | string | N/A | true |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| registry | ECR ARN | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |
| spacelift-api-key-id | Spacelift API Key ID | true |
| spacelift-api-key-secret | Spacelift API Key Secret | true |






## Hotfix branch (Pull request into release/* branches) workflow 

Build, test Docker image, deploy it with Spacelift to `hotfix` environment depends of PR labels  

### Usage 

Create in your repo  __`.github/workflows/hotfix-branch.yaml`__

```yaml
  name: Hotfix Branch
  on:
    pull_request:
      branches: [ 'release/**' ]
      types: [opened, synchronize, reopened, closed, labeled, unlabeled]
  
  permissions:
    pull-requests: write
    deployments: write
    id-token: write
    contents: read
  
  jobs:
    do:
      uses: cloudposse/github-actions-workflows-docker-ecr-spacelift/.github/workflows/hotfix-branch.yml@main
      with:
        organization: "${{ github.event.repository.owner.login }}"
        repository: "${{ github.event.repository.name }}"
        open: ${{ github.event.pull_request.state == 'open' }}
        labels: ${{ toJSON(github.event.pull_request.labels.*.name) }}
        ref: ${{ github.event.pull_request.head.ref  }}
        spacelift-organization: "${{ github.event.repository.owner.login }}"
      secrets:
        github-private-actions-pat: "${{ secrets.PUBLIC_REPO_ACCESS_TOKEN }}"
        registry: "${{ secrets.ECR_REGISTRY }}"
        secret-outputs-passphrase: "${{ secrets.GHA_SECRET_OUTPUT_PASSPHRASE }}"
        ecr-region: "${{ secrets.ECR_REGION }}"
        ecr-iam-role: "${{ secrets.ECR_IAM_ROLE }}"
        spacelift-api-key-id: ${{ secrets.SPACELIFT_API_KEY_ID }}
        spacelift-api-key-secret: ${{ secrets.SPACELIFT_API_KEY_SECRET }}
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| labels | Pull Request labels | string | ${{ toJSON(github.event.pull\_request.labels.\*.name) }} | false |
| open | Pull Request open/close state. Set true if opened | boolean | ${{ github.event.pull\_request.state == 'open' }} | false |
| organization | Repository owner organization (ex. acme for repo acme/example) | string | ${{ github.event.repository.owner.login }} | false |
| ref | The fully-formed ref of the branch or tag that triggered the workflow run | string | ${{ github.event.pull\_request.head.ref }} | false |
| repository | Repository name (ex. example for repo acme/example) | string | ${{ github.event.repository.name }} | false |
| spacelift-organization | Spacelift organization name | string | N/A | true |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| registry | ECR ARN | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |
| spacelift-api-key-id | Spacelift API Key ID | true |
| spacelift-api-key-secret | Spacelift API Key Secret | true |






## Hotfix workflow enable

For each new release create `release/{version}` branch that is key puzzle for `hotfix` workflow.

### Usage 

Create in your repo  __`.github/workflows/hotfix-enabled.yaml`__

```yaml
  name: Release branch
  on:
    release:
      types: [published]

  permissions:
    id-token: write
    contents: write

  jobs:
    hotfix:
      name: release / branch
      uses: cloudposse/github-actions-workflows-docker-ecr-eks-helmfile/.github/workflows/hotfix-mixin.yml@main
      with:
        version: ${{ github.event.release.tag_name }}
```

or add `hotfix` job to existing __`.github/workflows/release.yaml`__

```
  jobs:
    hotfix:
      name: release / branch
      uses: cloudposse/github-actions-workflows-docker-ecr-eks-helmfile/.github/workflows/hotfix-mixin.yml@main
      with:
        version: ${{ github.event.release.tag_name }}  
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| version | Release version tag | string | N/A | true |








## Hotfix release workflow

Build, test Docker image, deploy it with Spacelift on `production` environment and reintegrate `hotfix` into the main branch  

### Usage 

Create in your repo  __`.github/workflows/hotfix-release.yaml`__

```yaml
  name: Hotfix Release
  on:
    push:
      branches: [ 'release/**' ]
  
  permissions:
    contents: write
    id-token: write
  
  jobs:
    do:
      uses: cloudposse/github-actions-workflows-docker-ecr-eks-helmfile/.github/workflows/hotfix-release.yml@main
      with:
        organization: "${{ github.event.repository.owner.login }}"
        repository: "${{ github.event.repository.name }}"
        spacelift-organization: "${{ github.event.repository.owner.login }}"
      secrets:
        github-private-actions-pat: "${{ secrets.PUBLIC_REPO_ACCESS_TOKEN }}"
        registry: "${{ secrets.ECR_REGISTRY }}"
        secret-outputs-passphrase: "${{ secrets.GHA_SECRET_OUTPUT_PASSPHRASE }}"
        ecr-region: "${{ secrets.ECR_REGION }}"
        ecr-iam-role: "${{ secrets.ECR_IAM_ROLE }}"
        spacelift-api-key-id: ${{ secrets.SPACELIFT_API_KEY_ID }}
        spacelift-api-key-secret: ${{ secrets.SPACELIFT_API_KEY_SECRET }}
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| default\_branch | Default branch for this repo | string | main | true |
| organization | Repository owner organization (ex. acme for repo acme/example) | string | N/A | true |
| repository | Repository name (ex. example for repo acme/example) | string | N/A | true |
| spacelift-organization | Spacelift organization name | string | N/A | true |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| registry | ECR ARN | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |
| spacelift-api-key-id | Spacelift API Key ID | true |
| spacelift-api-key-secret | Spacelift API Key Secret | true |






## Main branch workflow

Build, test Docker image, deploy it with Spacelift on `dev` environment and draft new release  

### Usage 

Create in your repo  __`.github/workflows/main.yaml`__

```yaml
  name: Main Branch
  on:
    push:
      branches: [ master ]
  
  permissions:
    contents: write
    id-token: write
  
  jobs:
    do:
      uses: cloudposse/github-actions-workflows-docker-ecr-spacelift/.github/workflows/main-branch.yml@main
      with:
        organization: "${{ github.event.repository.owner.login }}"
        repository: "${{ github.event.repository.name }}"
        spacelift-organization: "${{ github.event.repository.owner.login }}"
      secrets:
        github-private-actions-pat: "${{ secrets.PUBLIC_REPO_ACCESS_TOKEN }}"
        registry: "${{ secrets.ECR_REGISTRY }}"
        secret-outputs-passphrase: "${{ secrets.GHA_SECRET_OUTPUT_PASSPHRASE }}"
        ecr-region: "${{ secrets.ECR_REGION }}"
        ecr-iam-role: "${{ secrets.ECR_IAM_ROLE }}"
        spacelift-api-key-id: ${{ secrets.SPACELIFT_API_KEY_ID }}
        spacelift-api-key-secret: ${{ secrets.SPACELIFT_API_KEY_SECRET }}

```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| organization | Repository owner organization (ex. acme for repo acme/example) | string | ${{ github.event.repository.owner.login }} | false |
| repository | Repository name (ex. example for repo acme/example) | string | ${{ github.event.repository.name }} | false |
| spacelift-organization | Spacelift organization name | string | N/A | true |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| registry | ECR ARN | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |
| spacelift-api-key-id | Spacelift API Key ID | true |
| spacelift-api-key-secret | Spacelift API Key Secret | true |






## Release workflow 

Promote existing Docker image to release version, deploy it with Spacelift to `staging` and then `production` environments.  

### Usage 

Create in your repo  __`.github/workflows/release.yaml`__

```yaml
  name: Release
  on:
    release:
      types: [published]
  
  permissions:
    id-token: write
    contents: write
  
  jobs:
    perform:
      uses: cloudposse/github-actions-workflows-docker-ecr-spacelift/.github/workflows/release.yml@main
      with:
        organization: "${{ github.event.repository.owner.login }}"
        repository: "${{ github.event.repository.name }}"
        version: ${{ github.event.release.tag_name }}
        spacelift-organization: "${{ github.event.repository.owner.login }}"
      secrets:
        github-private-actions-pat: "${{ secrets.PUBLIC_REPO_ACCESS_TOKEN }}"
        registry: "${{ secrets.ECR_REGISTRY }}"
        secret-outputs-passphrase: "${{ secrets.GHA_SECRET_OUTPUT_PASSPHRASE }}"
        ecr-region: "${{ secrets.ECR_REGION }}"
        ecr-iam-role: "${{ secrets.ECR_IAM_ROLE }}"
        spacelift-api-key-id: ${{ secrets.SPACELIFT_API_KEY_ID }}
        spacelift-api-key-secret: ${{ secrets.SPACELIFT_API_KEY_SECRET }}
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| organization | Repository owner organization (ex. acme for repo acme/example) | string | ${{ github.event.repository.owner.login }} | false |
| repository | Repository name (ex. example for repo acme/example) | string | ${{ github.event.repository.name }} | false |
| spacelift-organization | Spacelift organization name | string | N/A | true |
| version | Release version tag | string | ${{ github.event.release.tag\_name }} | false |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| registry | ECR ARN | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |
| spacelift-api-key-id | Spacelift API Key ID | true |
| spacelift-api-key-secret | Spacelift API Key Secret | true |





<!-- markdownlint-restore -->



## Share the Love

Like this project? Please give it a ★ on [our GitHub](https://github.com/cloudposse/github-actions-workflows-docker-ecr-spacelift)! (it helps us **a lot**)

Are you using this project or any of our other projects? Consider [leaving a testimonial][testimonial]. =)



## Related Projects

Check out these related projects.

- [github-actions-workflows](https://github.com/cloudposse/github-actions-workflows) - Reusable workflows for different types of projects


## References

For additional context, refer to some of these links.

- [example-app-on-ecs](https://github.com/cloudposse/example-app-on-ecs) - Example application for CI/CD demonstrations of GitHub Actions deal with Docker, ECR, ECS and Spacelift


## Help

**Got a question?** We got answers.

File a GitHub [issue](https://github.com/cloudposse/github-actions-workflows-docker-ecr-spacelift/issues), send us an [email][email] or join our [Slack Community][slack].

[![README Commercial Support][readme_commercial_support_img]][readme_commercial_support_link]

## DevOps Accelerator for Startups


We are a [**DevOps Accelerator**][commercial_support]. We'll help you build your cloud infrastructure from the ground up so you can own it. Then we'll show you how to operate it and stick around for as long as you need us.

[![Learn More](https://img.shields.io/badge/learn%20more-success.svg?style=for-the-badge)][commercial_support]

Work directly with our team of DevOps experts via email, slack, and video conferencing.

We deliver 10x the value for a fraction of the cost of a full-time engineer. Our track record is not even funny. If you want things done right and you need it done FAST, then we're your best bet.

- **Reference Architecture.** You'll get everything you need from the ground up built using 100% infrastructure as code.
- **Release Engineering.** You'll have end-to-end CI/CD with unlimited staging environments.
- **Site Reliability Engineering.** You'll have total visibility into your apps and microservices.
- **Security Baseline.** You'll have built-in governance with accountability and audit logs for all changes.
- **GitOps.** You'll be able to operate your infrastructure via Pull Requests.
- **Training.** You'll receive hands-on training so your team can operate what we build.
- **Questions.** You'll have a direct line of communication between our teams via a Shared Slack channel.
- **Troubleshooting.** You'll get help to triage when things aren't working.
- **Code Reviews.** You'll receive constructive feedback on Pull Requests.
- **Bug Fixes.** We'll rapidly work with you to fix any bugs in our projects.

## Slack Community

Join our [Open Source Community][slack] on Slack. It's **FREE** for everyone! Our "SweetOps" community is where you get to talk with others who share a similar vision for how to rollout and manage infrastructure. This is the best place to talk shop, ask questions, solicit feedback, and work together as a community to build totally *sweet* infrastructure.

## Discourse Forums

Participate in our [Discourse Forums][discourse]. Here you'll find answers to commonly asked questions. Most questions will be related to the enormous number of projects we support on our GitHub. Come here to collaborate on answers, find solutions, and get ideas about the products and services we value. It only takes a minute to get started! Just sign in with SSO using your GitHub account.

## Newsletter

Sign up for [our newsletter][newsletter] that covers everything on our technology radar.  Receive updates on what we're up to on GitHub as well as awesome new projects we discover.

## Office Hours

[Join us every Wednesday via Zoom][office_hours] for our weekly "Lunch & Learn" sessions. It's **FREE** for everyone!

[![zoom](https://img.cloudposse.com/fit-in/200x200/https://cloudposse.com/wp-content/uploads/2019/08/Powered-by-Zoom.png")][office_hours]

## Contributing

### Bug Reports & Feature Requests

Please use the [issue tracker](https://github.com/cloudposse/github-actions-workflows-docker-ecr-spacelift/issues) to report any bugs or file feature requests.

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

Copyright © 2017-2022 [Cloud Posse, LLC](https://cpco.io/copyright)



## License

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

See [LICENSE](LICENSE) for full details.

```text
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
```









## Trademarks

All other trademarks referenced herein are the property of their respective owners.

## About

This project is maintained and funded by [Cloud Posse, LLC][website]. Like it? Please let us know by [leaving a testimonial][testimonial]!

[![Cloud Posse][logo]][website]

We're a [DevOps Professional Services][hire] company based in Los Angeles, CA. We ❤️  [Open Source Software][we_love_open_source].

We offer [paid support][commercial_support] on all of our projects.

Check out [our other projects][github], [follow us on twitter][twitter], [apply for a job][jobs], or [hire us][hire] to help with your cloud strategy and implementation.



### Contributors

<!-- markdownlint-disable -->
|  [![Igor Rodionov][goruha_avatar]][goruha_homepage]<br/>[Igor Rodionov][goruha_homepage] |
|---|
<!-- markdownlint-restore -->

  [goruha_homepage]: https://github.com/goruha
  [goruha_avatar]: https://img.cloudposse.com/150x150/https://github.com/goruha.png

[![README Footer][readme_footer_img]][readme_footer_link]
[![Beacon][beacon]][website]
<!-- markdownlint-disable -->
  [logo]: https://cloudposse.com/logo-300x69.svg
  [docs]: https://cpco.io/docs?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=docs
  [website]: https://cpco.io/homepage?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=website
  [github]: https://cpco.io/github?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=github
  [jobs]: https://cpco.io/jobs?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=jobs
  [hire]: https://cpco.io/hire?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=hire
  [slack]: https://cpco.io/slack?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=slack
  [linkedin]: https://cpco.io/linkedin?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=linkedin
  [twitter]: https://cpco.io/twitter?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=twitter
  [testimonial]: https://cpco.io/leave-testimonial?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=testimonial
  [office_hours]: https://cloudposse.com/office-hours?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=office_hours
  [newsletter]: https://cpco.io/newsletter?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=newsletter
  [discourse]: https://ask.sweetops.com/?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=discourse
  [email]: https://cpco.io/email?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=email
  [commercial_support]: https://cpco.io/commercial-support?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=commercial_support
  [we_love_open_source]: https://cpco.io/we-love-open-source?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=we_love_open_source
  [terraform_modules]: https://cpco.io/terraform-modules?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=terraform_modules
  [readme_header_img]: https://cloudposse.com/readme/header/img
  [readme_header_link]: https://cloudposse.com/readme/header/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=readme_header_link
  [readme_footer_img]: https://cloudposse.com/readme/footer/img
  [readme_footer_link]: https://cloudposse.com/readme/footer/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=readme_footer_link
  [readme_commercial_support_img]: https://cloudposse.com/readme/commercial-support/img
  [readme_commercial_support_link]: https://cloudposse.com/readme/commercial-support/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-actions-workflows-docker-ecr-spacelift&utm_content=readme_commercial_support_link
  [share_twitter]: https://twitter.com/intent/tweet/?text=github-actions-workflows-docker-ecr-spacelift&url=https://github.com/cloudposse/github-actions-workflows-docker-ecr-spacelift
  [share_linkedin]: https://www.linkedin.com/shareArticle?mini=true&title=github-actions-workflows-docker-ecr-spacelift&url=https://github.com/cloudposse/github-actions-workflows-docker-ecr-spacelift
  [share_reddit]: https://reddit.com/submit/?url=https://github.com/cloudposse/github-actions-workflows-docker-ecr-spacelift
  [share_facebook]: https://facebook.com/sharer/sharer.php?u=https://github.com/cloudposse/github-actions-workflows-docker-ecr-spacelift
  [share_googleplus]: https://plus.google.com/share?url=https://github.com/cloudposse/github-actions-workflows-docker-ecr-spacelift
  [share_email]: mailto:?subject=github-actions-workflows-docker-ecr-spacelift&body=https://github.com/cloudposse/github-actions-workflows-docker-ecr-spacelift
  [beacon]: https://ga-beacon.cloudposse.com/UA-76589703-4/cloudposse/github-actions-workflows-docker-ecr-spacelift?pixel&cs=github&cm=readme&an=github-actions-workflows-docker-ecr-spacelift
<!-- markdownlint-restore -->
