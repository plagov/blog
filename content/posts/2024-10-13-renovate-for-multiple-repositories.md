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

```js
here comes the configuration
```

Next, we need a pipeline to run Renovate. Here goes the one that I have set up in my project.

```yaml
azure-pipelines
```

### Explaining the `config.js` configuration
Let's go through the configuration file step by step to explain what it does.
