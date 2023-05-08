# Overview

This page was assembled as a first draft for what we might title **The .NET User journey**. The intention is to create a new *Home* page in the SonarQube docs, specifically for .NET users; it can be their first point of entry into **Everything .NET** to locate the wide & diverse set of Sonar resources at their disposal. The text on this page can be expanded as required as the project unfolds.

# User Journey
1. User wants to setup a standard .net project
   - A manual scan
   - With one of our supported deveps platforms

1. They are not sure which version of the scanner to use and what .NET versions are supported
1. The have a more complex scenario such as a Proxy, exclusions, old versions that requires additional parameters setup
1. They want to understand if they need to do anything for PR analysis
1. The want to understand about external issue imports
1. They’d like to know if we support Unity - not yet!

1. From this there are a number of common outcomes
   - It goes smoothly now they want to add code coverage
   - The encounter one of the common errors and don’t know what to do
   - They have problems with test project detection (often no code analyzed) and don’t know how to fix it
   - The analysis take too long

# Content to integrate
The list below includes the .NET references we could find. The hyperlinks are *relative path* to the content copy/pasted into Markdown files located in this repository. As the URLs are integrated into the User Journey above, please strike-through the page names so we can track their use.

[~~User journey~~](user-journey.md)

## Docs Squad-mananaged resources

### Pages in the SonarQube Documentation

[Azure DevOps integration](azure-devops-integration.md)

[Test coverage parameters](test-coverage-parameters.md)

[SonarScanner for .NET](sonarscanner-for-dotnet.md)

[SonarQube extension for Azure DevOps](sonarqube-extension-for-azure-devops.md)

[.NET test coverage](dotnet-test-coverage.md)

## External resources

### Azure DevOPs Labs

[Driving continuous quality of your code with SonarCloud](azure-devops-labs.md)

### Community guides

[[Coverage] Troubleshooting guide for .NET code coverage import](community-guide-for-net-code-coverage-import)

[The Sonar guide for investigating the performance of .NET analysis](community-guide-for-investigating-the-performance-of-net-analysis.md)

[How to find logs about importing code coverage](community-guide-for-find-logs-about-importing-code-coverage.md)

[[Coverage & Test Data] Generate Reports for C#, VB.net](community-guide-coverage-test-data-generate-reports-for-c-vb-net.md)

[Configuration of WarningsAsErrors for .NET build](community-guide-configuration-of-warningsaserrors-for-net-build.md)

### Sonar Wikis

[Sonar Scanner for .NET wiki](msbuild-wiki.md)