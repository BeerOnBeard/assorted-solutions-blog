---
title: Versioned and Tagged Builds in Azure DevOps
date: 2020-05-30 12:53:15
---

Versioning a system makes it easier for a person to tie running code in production with the exact code in the repository. It is also a great way to signal when code is adding new features or introducing breaking changes. I'm using [Semantic Versioning](https://semver.org/) to version the system. In order to have the same traceability as I had with SHA codes I'm using [git tags](https://git-scm.com/book/en/v2/Git-Basics-Tagging).

To start, I'm using Azure Pipelines for my continuous integration system. The system is defined in `azure-pipelines.yml` at the root of the repository. I created two new variables that will be used to hold the major and minor version of the system.

```yaml
variables:
  major: 2
  minor: 5
```

I then used the top-level `name` to format the full semantic version. I used the [Azure-provided Rev token](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/run-number?view=azure-devops&tabs=yaml#tokens) for the patch version.

```yaml
name: $(major).$(minor).$(Rev:r)
```

Now the build will be named using the version of the system. It can also be used in the deployment pipeline by referencing the variable `$(Build.BuildNumber)` in the [release name format property](https://docs.microsoft.com/en-us/azure/devops/pipelines/release/?view=azure-devops#how-do-i-manage-the-names-for-new-releases).

Finally, in order to tag the revision in the git repository with the version that we've created, we need to use the [label sources functionality](https://docs.microsoft.com/en-us/azure/devops/pipelines/repos/azure-repos-git?view=azure-devops&tabs=yaml#label-sources). At the time of writing, this functionality is only available through the portal and cannot be defined in `azure-pipelines.yml`. Use the variable `$(build.buildNumber) as the tag format.

This is all great, but how do I make sure that versions are bumped when they should be? Having the pipeline defined as code in the repository is the real magic here. When new code comes in as a pull request, the major or minor version can be updated to reflect that a change is being made and introducing either a new feature or a breaking change. Once merged, the automated CI and CD pipelines take over.
