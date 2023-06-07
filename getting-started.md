A brief history of the .NET story explains why there are essentially three ways to install and run the .NET SonarScanner

[...](1-2 paragraphs of text for the history of .NET)

Scanning a .NET application is a little different than when analyzing other languages where the SonarScanner steps through the code to find issues before any build process is initiated; the SonarScanner for .NET instead steps through the code *while the build is in process*, gathering the results and sending them to SonarQube or SonarCloud for review once the build is complete.

 To accomplish this, the SonarScanner for .NET is configured to work with the [.NET Compiler Platform (Roslyn) Analyzers](https://learn.microsoft.com/en-us/visualstudio/code-quality/roslyn-analyzers-overview?view=vs-2022) where, as the Roslyn analyzer compiles the build, the SonarScanner does it's work. Because the SonarScanner is working *during the build process*, it is expected that the build time is increased by a factor of 3-4 times; this is normal and adds little if any time to entire release process because the build and analysis happen simultaneously. 

# Moving forward

 How you set up your SonarScanner for .NET depends on your dev environment. Here we will cover the most common cases:
[*TOM* I am not sure if we were to integrate this part, or if it's essencially the same information as in the next paragraph where we create the two branches: 1) Azure DevOps 2) everything else]
1. Initiating a manual scan without CI integration (on your local machine)
1. Initiating a scan from your CI
    1. Azure Devops - If this is the case follow the [Azure DevOps integration](azure-devops-integration.md) guide.
    1. Jenkins
    1. Github Actions
    1. Gitlab


:NET requies a different scanner
-  the scanner looks at the code
- .NET scanner is built into teh build, bcz Roslyn is the comliler that has analyzer hooks that gives the analyzer what it needs, when we need it.
  - makes it easier for function,
  - but makes it harder for the user bcz they need a different scanner.


[* GO TO THE AZURE DEVOPS GUIDE -OR- A TAB TO SPLIT]

[* THIS IS BRANCH 2, about the scanner, and how Roslyn works]
The basic steps are as follows:
1. Install Prequisites (Java)
2. Install the correct Scanner version for you .NET runtime and Install
3. Initiate the Scan
5. (optional) Setup PR decoration
6. (optional) Setup Code Coverage Import


[* THE NEXT 3 SECTIONS NEED TO BE WRITTEN AS A UNIQUE BRANCH, FOR EACH OF THE 3 WAYS DESCRIBED BELOW]

## Prequisites [* POSSIBLE PREAMBLE]

The Scanner must run in the same environment as where you build your application because the analysis happens during the build. [* THIS IS CONFUSING BCZ YOU ARE RUNNNING WHILE YOU ARE BUILDING] Therefore, don't be worried if your build takes a bit longer because it's now *also running an analysis*. The scanner will run on Windows, Linux or even OSX. The only additional prerequisite may be that Java 11 or 17 is required [* IS AN ALIEN CONSEPT THAT YOU NEED JAVA INSTALLED, BCZ THE SCANNER CLI NEEDS JAVA]. How you install this depends on the environment you're running on.

[* THE REAL PREREQUISIT IS THAT YOU ANALYZS ON YOUR BUILD SERVER, 
YOUR PREREQ IS JAVA AND TO INSTALLT EH CORRECT SCANNER FOR THE ONE THAT MATCHES YOUR BUILD ENVIRONMENT
YOU BUILD IT ON YOUR BUILD ENVIRNONETMENT
- .NET FRAMEWORK: HYPERLINKE
- .NET CORE: HYPERLINK
- .NET: HYPERLINK
]

## Installation

You can use any version of the scanner that supports the .NET runtime in your environment. More details can be found on the [SonarScanner for .NET](sonarscanner-for-dotnet.md) page. The simplest way to install the scanner if you are using .NET Core or later, is to use the dotnet global tool:

  [*
  - IF YOU HAVE THE donet `GLOBAL COMMAND LINE TOOL` INSTALLED, YOU DO THIS, OR, USE ONE OF THE OTHER METHODS (EASIEST, if you *can install the dotnet command line tool*, you can do this)
  - IF NOT: 
    - IF YOURE RUNNING THE FRAMEWORK VERSION or 
    - INSTALL THE .NET CORE OR LATER, (WITHOUT USING THE DONT WANT TO USE THE `GLOBAL COMMAND LINE TOOL`, [YOU DO THIS](/sonarscanner-for-dotnet/###.NET-Core-global-tool.md))

  ]

```
dotnet tool install --global dotnet-sonarscanner
```

If your build uses the .NET Framework, you'll need to download the .NET Framework 4.6+ version (as linked above). The are also Core versions available for download if you can't use the dotnet tool. 

## Initiating the Scan 
[* THIS SHOULD HAVE THREE BRANCHES, ONE FOR EACH OF THE PATHS FROM THE 3 PREREQ PATHS ABOVE.]

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








