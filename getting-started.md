A brief history of the .NET story explains why there are essentially three ways to install and run the .NET SonarScanner.

[TO DO](1-2 paragraphs of text for the history of .NET)
[* **TOM**: would this be valuable? If we explained *why there are 3 versions*]

Analyzing a .NET application is a little different than when analyzing other languages; whereas the SonarScanner normally scans existing code or already built projects, the SonarScanner for .NET instead steps through the code *while the build is in process*, gathering the results and sending them to SonarQube or SonarCloud for review once the build is complete.

 To accomplish this, the SonarScanner for .NET is configured to work with the [.NET Compiler Platform (Roslyn) Analyzers](https://learn.microsoft.com/en-us/visualstudio/code-quality/roslyn-analyzers-overview?view=vs-2022) where, as the Roslyn analyzer compiles the build, the SonarScanner does it's work. Because the SonarScanner is working *during the build process*, it is expected that the build time is increased by a factor of 3-4 times; this is normal and adds little if any time to entire release process because the build and analysis happen simultaneously. 

 .NET requies a different scanner
-  the scanner looks at the code
- .NET scanner is built into the build, bcz Roslyn is the comliler that has analyzer hooks that gives the analyzer what it needs, when we need it.
    - makes it easier for function,
    - but makes it harder for the user bcz they need a different scanner.

# Moving forward
[* **TOM**: this section gets a little muddy. Can you help me remember what the goal is? Do we want to determine which path the user should take?]

How you set up your SonarScanner for .NET depends on your dev environment. You can either initiate a manual scan on your local machine or initiate a scan from your CI provider. Here we will cover the most common cases:
[* **TOM** I am not sure if we were to integrate this part, or if it's essencially the same information as in the next paragraph where we create the two branches: 1) Azure DevOps 2) everything else]

1. Initiating a manual scan without continuous integration (on your local machine). For details, please go to [this page](sonarscanner-for-dotnet.md)
1. Initiating a scan from your CI. There are methods are available according to your CI/CD plattform:

[* **TOM**: Is there unique integration menthod related between each CI & .NET, something *different* than non-dotnet projects? I am trying to understand similarities and/or differences as a way to structure this part.]
[* **TOM**: Would it make sense for us to chart this out in a flow diagram? if you *have this*, then *you go there*, etc...]

  1. Azure DevOps 
      - For SonarQube users, see the [SonarQube extension for Azure DevOps](sonarqube-extension-for-azure-devops.md)
      - If you're using SonarCloud, the [Azure DevOps integration](azure-devops-integration.md) guide will step you through the setup process.
For other CI/CD services, the [SonarScanner for .NET](sonarscanner-for-dotnet.md)
  2. Github Actions 
  3. Gitlab integration 
  4. Jenkins integration
  - [* **TOM** numbers 2,3,4 above are confusing to me. 
       - In the SQ docs, we have a section called "DevOps platform integration" which holds GH & GL  
       - and a section called **Analyzing source code** > **CI integration** which holds Jenkins & Codemagic.
       - Please explain more about points 2,3,4.]


ALT
[* THIS IS BRANCH 2, about the scanner, and how Roslyn works]
The basic steps are as follows:
1. Install Prequisites (Java)
2. Install the correct Scanner version for you .NET runtime and Install
3. Initiate the Scan
5. (optional) Setup PR decoration
6. (optional) Setup Code Coverage Import
[* **TOM** Are these steps applied to the *CI on local* or *CI in cloud* method?]


[* THE NEXT 3 SECTIONS NEED TO BE WRITTEN AS A UNIQUE BRANCH, FOR EACH OF THE 3 WAYS DESCRIBED BELOW]

## Prerequisites [* POSSIBLE PREAMBLE]

The scanner will run on Windows, Linux, and OSX. The Scanner must run in the same environment where you build your application because the SonarScanner is working during the build process. Therefore, don't be worried if your build takes a bit longer because it's now *also running an analysis*. The only additional requirement is that Java 11 or 17 be installed; this is necessary because the SonarScanner CLI needs Java to run. How you install Java depends on the environment you're running on.

- PLEASE SEE this link for this environment
- PLEASE SEE that link for that environment

[* THE REAL PREREQUISIT IS THAT YOU ANALYZS ON YOUR BUILD SERVER, 
YOUR PREREQ IS JAVA AND TO INSTALLT EH CORRECT SCANNER FOR THE ONE THAT MATCHES YOUR BUILD ENVIRONMENT
YOU BUILD IT ON YOUR BUILD ENVIRNONETMENT
- .NET FRAMEWORK: HYPERLINK
- .NET CORE: HYPERLINK
- .NET: HYPERLINK
]

[* ALT]
Prerequisites include:
- The Sonarscanner must be installed in the same environment as where you build your application. The SonarScanner is working during the build process therefore, don't be worried if your build takes a bit longer because the build is now *also running an analysis*. Knowing which .NET version you are running is important; check this [Microsoft documentation](https://learn.microsoft.com/en-us/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed) to learn which versions you have installed. 

Select your correct .NET environment below to find the correct install instructions:
    - .NET FRAMEWORK: HYPERLINK
    - .NET CORE: HYPERLINK
    - .NET: HYPERLINK

- Java 11 or 17 be installed; this is necessary because the SonarScanner CLI needs Java to run. How you install this depends on the environment you're running on.
    - PLEASE SEE this link for this environment
    - PLEASE SEE that link for that environment


## Installation

[* **TOM**: do we have a different installation proceedure for FRAMEWORK, CORE, and .NET? If yes, I think we should either have a seperate page, or integrate a collapsable (or tab) component for each version.]
[* **TOM**: 
  1) does the `dotnet tool` work only with the CORE and .NET versions? and 
  2) would it make sense, becuase FRAMEWORK is oldest, to *always list it first*, or is it better to *list FRAMEWORK last* becuase it is the less modern version?]
[* **TOM**: the sonarscanner-for-dotnet page already talks about how to install. Do you think we can integrate the installtion there and keep the getting-started page high level?]

You can use any version of the scanner that supports the .NET runtime in your environment. More details are available on the [SonarScanner for .NET](sonarscanner-for-dotnet.md) page. The simplest way to install the scanner if you are using .NET Core or later, is to run `dotnet tool install` from the command line.

With the [.NET global tool](https://learn.microsoft.com/en-us/dotnet/core/tools/global-tools-how-to-use) installed, you can _________________.
- Step 1
- Step 2
- Step 3

If you're running the not able, or don't want to install the .NET global tool, you _________________.
    - IF YOURE RUNNING THE FRAMEWORK VERSION or 
    - INSTALL THE .NET CORE OR LATER, (WITHOUT USING THE DONT WANT TO USE THE `GLOBAL COMMAND LINE TOOL`, [YOU DO THIS](/sonarscanner-for-dotnet/###.NET-Core-global-tool.md))


```
dotnet tool install --global dotnet-sonarscanner
```

If your build uses the .NET Framework, you'll need to download the .NET Framework 4.6+ version (as linked above). The are also Core versions available for download if you can't use the dotnet tool. 

## Initiating the Scan 
## ALT: Using the SonarScanner
[* THIS SHOULD HAVE THREE BRANCHES, ONE FOR EACH OF THE PATHS FROM THE 3 PREREQ PATHS ABOVE.]
[* **TOM**: Are the 3 prereqs above the FRAMEWORK, CORE, .NET versions? If yes, then we should use 3 pages/collapsable/tabs to segregate the information.]
[* **TOM**: I only see 2 methods below; is it that the `dotnet tool` works for CORE & .NET?]

Unlike other Sonar Scanners, the dotnet scan requires you to wrap your build command with a SonarScanner **Begin step** and **End step** that sets up the scan before sending the results to SonarQube or SonarCloud.
[* **TOM**: What *is the begin step*, really?? I don't know whether to format it as `code` or `parameter`, as a **UI element**, or as a *special element*; or maybe it is simply a "begin step," with no special formatting?]

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

Pull request decoration is an important tool to help deploy a [Clean as you Code](HYPERLINK) strategy. In this way, your 

In order to Clean as You Code, you will want to see all new issues raised in your PR. If you've followed [these instructions](azure-devops-integration.md) and run the build in one of our supported CI's (for example, in [GitHub](https://docs.sonarqube.org/latest/devops-platform-integration/github-integration/), [Bitbucket Server](https://docs.sonarqube.org/latest/devops-platform-integration/bitbucket-integration/bitbucket-server-integration/), [Bitbucket Cloud](https://docs.sonarqube.org/latest/devops-platform-integration/bitbucket-integration/bitbucket-cloud-integration/) [GitLab](https://docs.sonarqube.org/latest/devops-platform-integration/gitlab-integration/), or [Azure DevOps](azure-devops-integration.md)), this will happen automatically. 

## Code Coverage

To track code coverage in Sonar you must use one of the supported coverage tools during your test run before the scanner can pick up the report. For instructions and examples of how to manage code coverage, refer to the [.NET test coverage](dotnet-test-coverage.md) page.








