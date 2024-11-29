+++
title = "How to configure Renovate for multiple repositories"
date = "2024-10-13"
author = "Vitali Plagov"
tags = ["continuous-integration"]
draft = false
+++

I had an opportunity to integrate the Renovate bot into our project. I would like to share my experience on how to
configure Renovate for multiple repositories on Azure DevOps platform.
<!--more-->

## Why using Renovate
It's typical for a product team in the organization to be responsible for and to maintain several projects. Be it a 
microservice with Java and Spring Boot or ASP.NET Core, each projects contains not only Maven or NuGet dependencies,
but also other dependencies. Like Docker images, GitHub Actions or Azure Pipelines tasks. Keeping them all up to date
even for a single project is a time-consuming task. The one should periodically allocate time and check new versions of
used dependencies, update them manually, raise a Pull Request. The more project a team has, the more this task becomes
boring.

Renovate can save a lot of time and effort. Once added to the project and set a schedule to run, it will make all the
above automatically.

The Renovate has several options to run. I will describe how I have configured the self-hosted version to run on 
Azure Pipelines against several ASP.NET Core repositories.

## A single configuration for several repositories
All the configurations for the Renovate can be placed in the `config.js` file. Configurations defined in this file are
applied to all projects that Renovate is set to maintain. A second place Renovate settings can be extended is the 
`renovate.json` file and that file is set individually per repository, while the `config.js` is placed in a single 
instance.

Let's start now in setting it up.

First, you need to create a new repository within your project on Azure DevOps. Create the `config.js` file. Below is 
the configuration that I have used for my project.

```javascript
module.exports = {
    extends: [
        ':automergeAll',
        'group:monorepos'
    ],
    packageRules: [
        {
            groupName: "Microsoft AspNetCore packages",
            matchPackagePatterns: ["^Microsoft\.AspNetCore\."],
            groupSlug: "aspnetcore"
        }
    ],
    platform: "azure",
    endpoint: "https://dev.azure.com/acme/",
    token: process.env.TOKEN,
    hostRules: [
        {
            hostType: 'nuget',
            matchHost: 'pkgs.dev.azure.com',
            username: 'apikey',
            password: process.env.RENOVATE_TOKEN,
        }
    ],
    regexManagers: [
        {
            fileMatch: ["docker-compose.*\\.ya?ml$"],
            matchStrings: ["image: acme\\.azurecr\\.io/docker/(library/)?(?<depName>.*?):(?<currentValue>.*?)\n"],
            datasourceTemplate: "docker"
        }
    ],
    repositories: [
        'acme-project/acme-repository
    ],
    baseBranches: ["main"],
    rebaseWhen: 'behind-base-branch'
};
```

Next, we need a pipeline to run Renovate. Here goes the one that I have set up in my project.

```yaml
trigger:
  - none

schedules:
  - cron: "0 8 * * 1-5"
    displayName: Weekdays 08:00 UTC runs
    branches:
      include:
        - main
    always: true

pool:
  name: ubuntu-latest

steps:
  - checkout: self
  - checkout: git://acme-project/acme-repository-one
  - checkout: git://acme-project/acme-repository-two

  - task: npmAuthenticate@0
    inputs:
      workingFile: renovate/.npmrc

  - bash: |
      git config --global user.email 'bot@renovateapp.com'
      git config --global user.name 'Renovate Bot'
      npx --userconfig renovate/.npmrc renovate
    env:
      RENOVATE_CONFIG_FILE: renovate/config.js
      RENOVATE_PLATFORM: azure
      RENOVATE_ENDPOINT: $(System.CollectionUri)
      RENOVATE_TOKEN: $(System.AccessToken)
      GITHUB_COM_TOKEN: $(GITHUB_TOKEN)
      LOG_LEVEL: debug
```

### Explaining the `config.js` configuration
Let's go through the configuration file step by step to explain what it does.
The configuration starts with the `extends` block, which is meant to list all the built-in presets that the Renovate bot has.
For the full list of presets, please refer to the [Renovate documentation](https://docs.renovatebot.com/presets-default/)
to find presets suitable for your project. Two presets that we have found useful for different teams in our organisation are
`:automergeAll'`. This marks the pull request in Azure Repos to be automatically merged once all the required checks 
have been passed. And the second is `group:monorepos`, which merges all package updates coming from the same mono 
repository into a single PR. This will firstly limit the number of PRs raised, and secondly avoid the failed builds 
scenario when two packages from the same monorepo are dependent on each other, but are updated separately.

The next block is `packageRules`. This allows you to group packages by name. So all package updates
that match the pattern will be grouped and updated at once. Such package grouping helped us to solve the following problem.
Our projects run on .NET 8, which is an LTS version. There have been a lot of updates to `Micrososft.AspNetCore.*`,
`Microsoft.Extensions.*`, etc. packages. Some of them were compatible with .NET 8, others required .NET 9. 
The `AspNetCore` packages were the ones that required .NET 9. But all these `Microsoft.*` packages were in a single PR
making it impossible to merge them. However, we didn't want to ignore those updates that were compatible with .NET 8.
I identified packages that required .NET 9 and put them into a separate group. That way, if a new PR came out with
update, I could reject it and accept others separately.

The next block `hostRules` along with some fields are a common configuration for the bot to set the platform it will 
use to look for updates. Also, the Private Access Token (PAT) to use.

The `regexManagers` block is another customisation for the configuration of the bot. In our organisation, we have a private
docker image registry that we use to cache the docker images that we first pull from the public registries. Because our 
private registry doesn't store newly released tags, we have to fetch updates from Docker Hub and other public registries. 
So, we need to parse the docker-compose file or dockerfiles to get the docker image name and tag.


So, the `fileMatch` field sets the regex for the filename to look for.
The `matchStrings` field is a regex pattern to find within the file. A very important thing here is to put the package
name in the `depName` regex group and the current package version in the `currentValue` group. 
The `datasourceTemplate` is an instruction to Renovate telling it which package manager to use. It will take values from the
regex groups above and will look for updates against the specified package manager.

The last important block is the `repositories` list, which contains all the projects that Renovate will scan and maintain.
In Azure Repos, these should be defined in the format `project-name/repository-name`.
