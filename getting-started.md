Scanning .NET application is a little different to other languages and requires a dedicated scanner that works with MSBuild. How you set this up depends on your environment. Here we will cover the most common cases:
1. Initiating a manual scan without CI integration
2. Initiating a scan from your CI
  3. Azure Devops - If this is the case follow [this](https://docs.sonarqube.org/latest/devops-platform-integration/azure-devops-integration/) guide
  4. Jenkins
  5. Github Actions
  6. Gitlab

The basic steps are as follows:
1. Install Prequisites (Java)
2. Install the correct Scanner version for you .NET runtime and Install
3. Initiate the Scan
5. (optional) Setup PR decoration
6. (optional) Setup Code Coverage Import

## Prequisites 

The Scanner must run in the same environment as where you build your application because the analysis happens during the build (so don't be worried if your build takes a bit longer because its now also running an analysis), this can be on Windows, Linux or even OSX. The only additional prerequisite you may require is Java 11 or 17. How you install this depends on the envinment your running on.

## Installation

You can use any version of the scanner that supports the .NET runtime in the environment your running it in. 
The simplest way to install the scanner if you are using .NET Core or later is using dotnet global tool

```
dotnet tool install --global dotnet-sonarscanner
```

If your building using the .NET Framework you'll need to download the .NET Framework 4.6+ version linked above. The are also the Core versions available for download if you can't use the dotnet tool. 

## Initiating the Scan

Unlike other Sonar Scanners the scan requires you to wrap your build command with SonarScanner Begin and End step that sets up the scan and then sends the results to SonarQube/Cloud

For example if you are using the dotnet tool version your build command could look like this:
```
dotnet tool install --global dotnet-sonarscanner
dotnet sonarscanner begin /k:"project-key" /d:sonar.token="<token>"
dotnet build <path to project file or .sln file>
dotnet sonarscanner end /d:sonar.token="<token>"
```
And if you are using the .NET Framework version:
```
SonarScanner.MSBuild.exe begin /k:"project-key" /d:sonar.token="myAuthenticationToken"
MSBuild.exe <path to project file or .sln file> /t:Rebuild
SonarScanner.MSBuild.exe end /d:sonar.token="myAuthenticationToken"
```
All being well you will, this is all you need to see your analysed project

## PR Decoration

In order to Clean as You Code you'll want to see any new issues raised in your PR. If you've followed these instructions and run in one of our supported CI's this will happen automatically 

## Code Coverage

To track code coverage in Sonar you need to use one of the supported coverage tools during your test run and then the scanner will pick up the report. For instructions and examples of how to do this refer to [.NET Test Coverage](https://docs.sonarqube.org/latest/analyzing-source-code/test-coverage/dotnet-test-coverage/)








