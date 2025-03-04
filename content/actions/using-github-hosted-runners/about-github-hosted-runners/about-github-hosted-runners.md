---
title: About GitHub-hosted runners
shortTitle: About GitHub-hosted runners
intro: '{% data variables.product.prodname_dotcom %} offers hosted virtual machines to run workflows. The virtual machine contains an environment of tools, packages, and settings available for {% data variables.product.prodname_actions %} to use.'
redirect_from:
  - /articles/virtual-environments-for-github-actions
  - /github/automating-your-workflow-with-github-actions/virtual-environments-for-github-actions
  - /github/automating-your-workflow-with-github-actions/virtual-environments-for-github-hosted-runners
  - /actions/automating-your-workflow-with-github-actions/virtual-environments-for-github-hosted-runners
  - /actions/reference/virtual-environments-for-github-hosted-runners
  - /actions/reference/software-installed-on-github-hosted-runners
  - /actions/reference/specifications-for-github-hosted-runners
versions:
  fpt: '*'
  ghes: '*'
  ghec: '*'
---

{% data reusables.actions.enterprise-github-hosted-runners %}

## Overview of {% data variables.product.prodname_dotcom %}-hosted runners

Runners are the machines that execute jobs in a {% data variables.product.prodname_actions %} workflow. For example, a runner can clone your repository locally, install testing software, and then run commands that evaluate your code.

{% data variables.product.prodname_dotcom %} provides runners that you can use to run your jobs, or you can [host your own runners](/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners). Each {% data variables.product.prodname_dotcom %}-hosted runner is a new virtual machine (VM) hosted by {% data variables.product.prodname_dotcom %} with the runner application and other tools preinstalled, and is available with Ubuntu Linux, Windows, or macOS operating systems. When you use a {% data variables.product.prodname_dotcom %}-hosted runner, machine maintenance and upgrades are taken care of for you.

{% ifversion github-hosted-runners-emus-entitlements %}

{% note %}

**Note:** {% data reusables.actions.entitlement-minutes-emus %} For more information, see "[AUTOTITLE](/admin/identity-and-access-management/using-enterprise-managed-users-for-iam/about-enterprise-managed-users)."

{% endnote %}

{% endif %}

{% ifversion not ghes %}

## Using a {% data variables.product.prodname_dotcom %}-hosted runner

To use a {% data variables.product.prodname_dotcom %}-hosted runner, create a job and use `runs-on` to specify the type of runner that will process the job, such as `ubuntu-latest`, `windows-latest`, or `macos-latest`. For the full list of runner types, see "[AUTOTITLE](/actions/using-github-hosted-runners/about-github-hosted-runners/about-github-hosted-runners#supported-runners-and-hardware-resources)."{% ifversion repository-actions-runners %} If you have `repo: write` access to a repository, you can view a list of the runners available to use in workflows in the repository. For more information, see "[Viewing available runners for a repository](#viewing-available-runners-for-a-repository)."{% endif %}

When the job begins, {% data variables.product.prodname_dotcom %} automatically provisions a new VM for that job. All steps in the job execute on the VM, allowing the steps in that job to share information using the runner's filesystem. You can run workflows directly on the VM or in a Docker container. When the job has finished, the VM is automatically decommissioned.

The following diagram demonstrates how two jobs in a workflow are executed on two different {% data variables.product.prodname_dotcom %}-hosted runners.

![Diagram of a workflow that consists of two jobs. One job runs on Ubuntu and the other runs on Windows.](/assets/images/help/actions/overview-github-hosted-runner.png)

The following example workflow has two jobs, named `Run-npm-on-Ubuntu` and `Run-PSScriptAnalyzer-on-Windows`. When this workflow is triggered, {% data variables.product.prodname_dotcom %} provisions a new virtual machine for each job.

- The job named `Run-npm-on-Ubuntu` is executed on a Linux VM, because the job's `runs-on:` specifies `ubuntu-latest`.
- The job named `Run-PSScriptAnalyzer-on-Windows` is executed on a Windows VM, because the job's `runs-on:` specifies `windows-latest`.

```yaml copy
name: Run commands on different operating systems
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  Run-npm-on-Ubuntu:
    name: Run npm on Ubuntu
    runs-on: ubuntu-latest
    steps:
      - uses: {% data reusables.actions.action-checkout %}
      - uses: {% data reusables.actions.action-setup-node %}
        with:
          node-version: '14'
      - run: npm help

  Run-PSScriptAnalyzer-on-Windows:
    name: Run PSScriptAnalyzer on Windows
    runs-on: windows-latest
    steps:
      - uses: {% data reusables.actions.action-checkout %}
      - name: Install PSScriptAnalyzer module
        shell: pwsh
        run: |
          Set-PSRepository PSGallery -InstallationPolicy Trusted
          Install-Module PSScriptAnalyzer -ErrorAction Stop
      - name: Get list of rules
        shell: pwsh
        run: |
          Get-ScriptAnalyzerRule
```

While the job runs, the logs and output can be viewed in the {% data variables.product.prodname_dotcom %} UI:

![Screenshot of a workflow run. The steps for the "Run PSScriptAnalyzer on Windows" job are displayed.](/assets/images/help/repository/actions-runner-output.png)

{% data reusables.actions.runner-app-open-source %}

{% ifversion repository-actions-runners %}

## Viewing available runners for a repository

{% data reusables.actions.about-viewing-runner-list %}

{% data reusables.repositories.navigate-to-repo %}
{% data reusables.repositories.actions-tab %}
{% data reusables.repositories.repository-runners %}
1. Review the list of available GitHub-hosted runners for the repository.
{% data reusables.actions.copy-runner-label %}

{% data reusables.actions.actions-tab-new-runners-note %}

{% endif %}

## Supported runners and hardware resources

{% ifversion actions-hosted-runners %}

{% note %}

**Note**: {% data variables.product.prodname_dotcom %} also offers {% data variables.actions.hosted_runner %}s, which are available in larger configurations for Linux, Windows, and macOS virtual machines. Autoscaling is enabled by default and optional dedicated IP addresses are available for Linux and Windows. For more information, see "[AUTOTITLE](/actions/using-github-hosted-runners/using-larger-runners#machine-specs-for-larger-runners)."

{% endnote %}
{% endif %}

{% data reusables.actions.supported-github-runners %}

Workflow logs list the runner used to run a job. For more information, see "[AUTOTITLE](/actions/monitoring-and-troubleshooting-workflows/viewing-workflow-run-history)."

### {% data variables.actions.hosted_runner_caps %}s

{% data reusables.actions.about-larger-runners %}

For more information, see "[AUTOTITLE](/actions/using-github-hosted-runners/about-larger-runners)."

## Supported software

The software tools included in {% data variables.product.prodname_dotcom %}-hosted runners are updated weekly. The update process takes several days, and the list of preinstalled software on the `main` branch is updated after the whole deployment ends.

### Preinstalled software

Workflow logs include a link to the preinstalled tools on the exact runner. To find this information in the workflow log, expand the `Set up job` section. Under that section, expand the `Runner Image` section. The link following `Included Software` will describe the preinstalled tools on the runner that ran the workflow.

For more information, see "[AUTOTITLE](/actions/monitoring-and-troubleshooting-workflows/viewing-workflow-run-history)." For the overall list of included tools for each runner operating system, see the [Available Images](https://github.com/actions/runner-images#available-images) documentation the runner images repository.

{% data variables.product.prodname_dotcom %}-hosted runners include the operating system's default built-in tools, in addition to the packages listed in the above references. For example, Ubuntu and macOS runners include `grep`, `find`, and `which`, among other default tools.

{% ifversion actions-sbom %}

You can also view a software bill of materials (SBOM) for each build of the Windows and Ubuntu runner images. For more information, see "[AUTOTITLE](/actions/security-guides/security-hardening-for-github-actions#reviewing-the-supply-chain-for-github-hosted-runners)."

{% endif %}

### Using preinstalled software

We recommend using actions to interact with the software installed on runners. This approach has several benefits:
- Usually, actions provide more flexible functionality like versions selection, ability to pass arguments, and parameters
- It ensures the tool versions used in your workflow will remain the same regardless of software updates

If there is a tool that you'd like to request, please open an issue at [actions/runner-images](https://github.com/actions/runner-images). This repository also contains announcements about all major software updates on runners.

### Installing additional software

You can install additional software on {% data variables.product.prodname_dotcom %}-hosted runners. For more information, see "[AUTOTITLE](/actions/using-github-hosted-runners/customizing-github-hosted-runners)".

## Cloud hosts used by {% data variables.product.prodname_dotcom %}-hosted runners

{% data variables.product.prodname_dotcom %} hosts Linux and Windows runners on `Standard_DS2_v2` virtual machines in Microsoft Azure with the {% data variables.product.prodname_actions %} runner application installed. The {% data variables.product.prodname_dotcom %}-hosted runner application is a fork of the Azure Pipelines Agent. Inbound ICMP packets are blocked for all Azure virtual machines, so ping or traceroute commands might not work. For more information about the `Standard_DS2_v2` resources, see "[Dv2 and DSv2-series](https://docs.microsoft.com/azure/virtual-machines/dv2-dsv2-series#dsv2-series)" in the Microsoft Azure documentation. {% data variables.product.prodname_dotcom %} hosts macOS runners in Azure data centers.

## Workflow continuity

{% data reusables.actions.runner-workflow-continuity %}

In addition, if the workflow run has been successfully queued, but has not been processed by a {% data variables.product.prodname_dotcom %}-hosted runner within 45 minutes, then the queued workflow run is discarded.

## Administrative privileges

The Linux and macOS virtual machines both run using passwordless `sudo`. When you need to execute commands or install tools that require more privileges than the current user, you can use `sudo` without needing to provide a password. For more information, see the "[Sudo Manual](https://www.sudo.ws/man/1.8.27/sudo.man.html)."

Windows virtual machines are configured to run as administrators with User Account Control (UAC) disabled. For more information, see "[How User Account Control works](https://docs.microsoft.com/windows/security/identity-protection/user-account-control/how-user-account-control-works)" in the Windows documentation.

## IP addresses

To get a list of IP address ranges that {% data variables.product.prodname_actions %} uses for {% data variables.product.prodname_dotcom %}-hosted runners, you can use the {% data variables.product.prodname_dotcom %} REST API. For more information, see the `actions` key in the response of the "[AUTOTITLE](/rest/meta#get-github-meta-information)" endpoint.

Windows and Ubuntu runners are hosted in Azure and subsequently have the same IP address ranges as the Azure datacenters. macOS runners are hosted in {% data variables.product.prodname_dotcom %}'s own macOS cloud.

Since there are so many IP address ranges for {% data variables.product.prodname_dotcom %}-hosted runners, we do not recommend that you use these as allowlists for your internal resources. Instead, we recommend you use {% data variables.actions.hosted_runner %}s with a static IP address range, or self-hosted runners. For more information, see "[AUTOTITLE](/actions/using-github-hosted-runners/about-larger-runners)" or "[AUTOTITLE](/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)."

The list of {% data variables.product.prodname_actions %} IP addresses returned by the API is updated once a week.

## The `etc/hosts` file

{% data reusables.actions.runners-etc-hosts-file %}

## File systems

{% data variables.product.prodname_dotcom %} executes actions and shell commands in specific directories on the virtual machine. The file paths on virtual machines are not static. Use the environment variables {% data variables.product.prodname_dotcom %} provides to construct file paths for the `home`, `workspace`, and `workflow` directories.

| Directory | Environment variable | Description |
|-----------|----------------------|-------------|
| `home` | `HOME` | Contains user-related data. For example, this directory could contain credentials from a login attempt. |
| `workspace` | `GITHUB_WORKSPACE` | Actions and shell commands execute in this directory. An action can modify the contents of this directory, which subsequent actions can access. |
| `workflow/event.json` | `GITHUB_EVENT_PATH` | The `POST` payload of the webhook event that triggered the workflow. {% data variables.product.prodname_dotcom %} rewrites this each time an action executes to isolate file content between actions.

For a list of the environment variables {% data variables.product.prodname_dotcom %} creates for each workflow, see "[AUTOTITLE](/actions/learn-github-actions/variables#default-environment-variables)."

### Docker container filesystem

Actions that run in Docker containers have static directories under the `/github` path. However, we strongly recommend using the default environment variables to construct file paths in Docker containers.

{% data variables.product.prodname_dotcom %} reserves the `/github` path prefix and creates three directories for actions.

- `/github/home`
- `/github/workspace` - {% data reusables.repositories.action-root-user-required %}
- `/github/workflow`

## Further reading

- "[AUTOTITLE](/billing/managing-billing-for-github-actions)"
- You can use a matrix strategy to run your jobs on multiple images. For more information, see "[AUTOTITLE](/actions/using-jobs/using-a-matrix-for-your-jobs)."

{% endif %}
