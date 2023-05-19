Scanning a .NET application is a little different than when analyzing other languages and requires a dedicated scanner that works with MSBuild. How you set this up depends on your environment. Here we will cover the most common cases:
1. Initiating a manual scan without CI integration
1. Initiating a scan from your CI
    1. Azure Devops - If this is the case follow the [Azure DevOps integration](azure-devops-integration.md) guide.
    1. Jenkins
    1. Github Actions
    1. Gitlab

The basic steps are as follows:
1. Install Prequisites (Java)
2. Install the correct Scanner version for you .NET runtime and Install
3. Initiate the Scan
5. (optional) Setup PR decoration
6. (optional) Setup Code Coverage Import

## Prequisites 

The Scanner must run in the same environment as where you build your application because the analysis happens during the build. Therefore, don't be worried if your build takes a bit longer because it's now *also running an analysis*. The scanner will run on Windows, Linux or even OSX. The only additional prerequisite may be that Java 11 or 17 is required. How you install this depends on the environment you're running on.

## Installation

You can use any version of the scanner that supports the .NET runtime in your environment. More details can be found on the [SonarScanner for .NET](sonarscanner-for-dotnet.md) page. The simplest way to install the scanner if you are using .NET Core or later, is to use the dotnet global tool:

```
dotnet tool install --global dotnet-sonarscanner
```

If your build uses the .NET Framework, you'll need to download the .NET Framework 4.6+ version (as linked above). The are also Core versions available for download if you can't use the dotnet tool. 

## Initiating the Scan

Unlike other Sonar Scanners, the dotnet scan requires you to wrap your build command with a SonarScanner Begin and End step that sets up the scan before sending the results to SonarQube/Cloud.

For example if you are using the dotnet tool version, your build command could look like this:
```
dotnet tool install --global dotnet-sonarscanner
dotnet sonarscanner begin /k:"project-key" /d:sonar.token="<token>"
dotnet build <path to project file or .sln file>
dotnet sonarscanner end /d:sonar.token="<token>"
```
And if you are using the .NET Framework version, your build command might look like this:
```
SonarScanner.MSBuild.exe begin /k:"project-key" /d:sonar.token="myAuthenticationToken"
MSBuild.exe <path to project file or .sln file> /t:Rebuild
SonarScanner.MSBuild.exe end /d:sonar.token="myAuthenticationToken"
```
With everything set up correctly, this is all you need to analyse your project.

## PR Decoration

In order to Clean as You Code, you will want to see all new issues raised in your PR. If you've followed [these instructions](azure-devops-integration.md) and run the build in one of our supported CI's (for example, in [GitHub](https://docs.sonarqube.org/latest/devops-platform-integration/github-integration/), [Bitbucket Server](https://docs.sonarqube.org/latest/devops-platform-integration/bitbucket-integration/bitbucket-server-integration/), [Bitbucket Cloud](https://docs.sonarqube.org/latest/devops-platform-integration/bitbucket-integration/bitbucket-cloud-integration/) [GitLab](https://docs.sonarqube.org/latest/devops-platform-integration/gitlab-integration/), or [Azure DevOps](azure-devops-integration.md)), this will happen automatically. 

## Code Coverage

To track code coverage in Sonar you must use one of the supported coverage tools during your test run before the scanner can pick up the report. For instructions and examples of how to manage code coverage, refer to the [.NET test coverage](dotnet-test-coverage.md) page.








