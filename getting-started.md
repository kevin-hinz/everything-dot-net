A brief history of the .NET story explains why there are essentially three ways to install and run the .NET SonarScanner.

[TO DO](A few sentences of text for the history of .NET)
    [* **COMMENT**: Let's figure out who the intended audiance is, first. Could be a combination of the statement: for these applications, go down this path...for those kind of applications, go down that path.]

# SonarScanner
Analyzing a .NET application is a little different than when analyzing other languages; whereas the SonarScanner normally scans existing code or already built projects, the SonarScanner for .NET analyzes the code [*while the build is in process*](https://learn.microsoft.com/en-us/visualstudio/code-quality/roslyn-analyzers-overview?view=vs-2022). Once the build is complete and the code is analyzed, results are gathered and sent to SonarQube or SonarCloud for review. Because the analysis processes are integrated with the build, it is expected that the build time will increased by a factor of 3-4 times; this is normal. 

## Installing the scanner

You can use any version of the SonarScanner that supports the .NET runtime in your environment. More details are available on the [SonarScanner for .NET](sonarscanner-for-dotnet.md) page. The simplest way to install the scanner if you are using .NET Core or later, is to run `dotnet tool install` from the command line (NEEDS HYPERLINK TO EXISTING DOCS).

If your build uses the .NET Framework, you'll need to download the .NET Framework 4.6+ version (NEEDS HYPERLINK). The are also Core versions available for download if you can't use the dotnet tool. (NEEDS HYPERLINK TO EXISTING DOCS)

# Prerequisites
   [* **TOM**: I think it made sense to keep, at least, the global prereqs here. And I have no strong opinion if you disagree. Also, I know this repeats some things from the #SonarScanner header above, but having it in #Prerequisites might help avoid TLDR syndrome.]

Prerequisites include that the Sonarscanner be installed in the same environment as where you build your application. The SonarScanner is working during the build process therefore, don't be worried if your build takes a bit longer because as mentioned above, the build is now *also running an analysis*. Knowing which .NET version you are running is important; check this [Microsoft documentation](https://learn.microsoft.com/en-us/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed) to learn which versions you have installed. 

## Your environment
Select your correct .NET environment below to find the correct install instructions:
    - .NET FRAMEWORK: HYPERLINK
    - .NET CORE: HYPERLINK
    - .NET: HYPERLINK

## Java

Either Java 11 or 17 must be installed; this is necessary because the SonarScanner CLI needs Java to run. The Java version you choose depends on the environment you're running on:
    - PLEASE SEE THIS LINK FOR THIS ENVIRONMENT
    - PLEASE SEE THAT LINK FOR THAT ENVIRONMENT
    - PLEASE SEE THAT LINK FOR THE OTHER ENVIRONMENT

# CI Environments

How you set up your SonarScanner for .NET depends on your dev environment. Here we will cover the most common cases:

## Azure DevOps

Azure Pipelines/Sonar extension(?)
- OVERVIEW SENTENCE
    - WHAT IS PECULIAR TO GITHUB
- For SonarQube users, see the [SonarQube extension for Azure DevOps](sonarqube-extension-for-azure-devops.md)
    - & CHECK THAT .NET IS THERE
- If you're using SonarCloud, the [Azure DevOps integration](azure-devops-integration.md) guide will step you through the setup process.
    - & CHECK THAT .NET IS THERE
- HIGH-LEVEL BASIC STEPS(?)

## GitHub
    
GitHub Actions
- OVERVIEW SENTENCE
    - WHAT IS PECULIAR TO GitHub 
- LINK TO EXISTING & CHECK THAT .NET IS THERE
- HIGH-LEVEL BASIC STEPS(?)

## GitLab

GitLab integration
- OVERVIEW SENTENCE
    - WHAT IS PECULIAR TO GitLab 
- LINK TO EXISTING & CHECK THAT .NET IS THERE
- HIGH-LEVEL BASIC STEPS(?)

## Jenkins

Jenkins integration
- OVERVIEW SENTENCE
    - WHAT IS PECULIAR TO GitHub 
- LINK TO EXISTING & CHECK THAT .NET IS THERE
- HIGH-LEVEL BASIC STEPS(?)


## SAMPLE BASIC STEPS
[* **TOM**: We can remove this if it's determined that we *don't want to include basic steps* in the CI sections above.]
THE BASIC STEPS ARE AS FOLLOWS
1. Install Prequisites (Java) (ALL)
2. Install the correct Scanner version for you .NET runtime and Install (CI)
3. Initiate the Scan (ALL)
5. ~~(optional) Setup PR decoration (ALL)~~
6. (optional) Setup Code Coverage Import

## Code Coverage

To track code coverage in Sonar you must use one of the supported coverage tools during your test run before the scanner can pick up the report. For instructions and examples of how to manage code coverage, refer to the [.NET test coverage](dotnet-test-coverage.md) page.