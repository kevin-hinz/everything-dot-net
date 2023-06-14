[SOURCE](https://docs.sonarqube.org/latest/analyzing-source-code/scanners/sonarscanner-for-dotnet)

5.13

2023-04-05

Support for sonar.token parameter and improved error messages

The SonarScanner for .NET is the recommended way to launch an analysis for projects built using `MSBuild` or `dotnet`. It is the result of a [collaboration between SonarSource and Microsoft](https://www.sonarsource.com/blog/easy-analysis-of-visual-studio-solutions-with-the-sonarqube-scanner-for-msbuild/).

SonarScanner for .NET is distributed as a standalone command line executable, as an extension for [Azure DevOps Server](https://docs.sonarqube.org/latest/analyzing-source-code/scanners/sonarqube-extension-for-azure-devops/), and as a plugin for [Jenkins](https://docs.sonarqube.org/latest/analyzing-source-code/scanners/jenkins-extension-sonarqube/).

It supports .NET Core on every platform (Windows, macOS, Linux).

## Prerequisites

[* TOM: Would it be possible to fit these prerequisites into a matrix, not unlike this one in the [.NET wikipedia](https://en.wikipedia.org/wiki/.NET_Framework_version_history) page?]

-   At least the minimal version of Java supported by your SonarQube server
-   The SDK corresponding to your build system:
    -   If you are using the .NET Framework version of the scanner you will need [.NET Framework v4.6 or above](https://dotnet.microsoft.com/en-us/download/dotnet-framework). For commercial versions of SonarQube to benefit from security analysis you will need [.NET Framework v4.7.2 or above](https://dotnet.microsoft.com/en-us/download/dotnet-framework)
    -   If you are using the .NET version of the scanner or the [.NET Core Global Tool](https://www.nuget.org/packages/dotnet-sonarscanner) you will need [.NET Core SDK 2.0 or above](https://dotnet.microsoft.com/en-us/download/dotnet)

## Installation

### Standalone executable

-   Expand the downloaded file into the directory of your choice. We'll refer to it as `<INSTALL_DIRECTORY>` in the next steps.
    -   On Windows, you might need to unblock the ZIP file first (right-click **file > Properties > Unblock**).
    -   On Linux/OSX you may need to set execute permissions on the files in `<INSTALL_DIRECTORY>/sonar-scanner-(version)/bin`.
-   Uncomment, and update the global settings to point to your SonarQube server by editing `<INSTALL_DIRECTORY>/SonarQube.Analysis.xml`. Values set in this file will be applied to all analyses of all projects unless overwritten locally.  
    Consider setting file system permissions to restrict access to this file.

```
<SonarQubeAnalysisProperties  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://www.sonarsource.com/msbuild/integration/2015/1">
  <Property Name="sonar.host.url">http://localhost:9000</Property>
  <Property Name="sonar.token">[my-user-token]</Property>
</SonarQubeAnalysisProperties>
```

-   Add `<INSTALL_DIRECTORY>` to your `PATH` environment variable.

### .NET Core global tool

```
dotnet tool install --global dotnet-sonarscanner --version x.x.x
```
[* WE SHOULD MOVE THIS BIT ABOUT THE VERSION SOMEHWERE ELSE... HERE IS A VESION PROPERTY, BUT YOU DON'T NEED IT]
The `--version` argument is optional. If it is omitted the latest version will be installed. The full list of releases is available on the [NuGet page](https://www.nuget.org/packages/dotnet-sonarscanner).

.NET Core Global Tool is available from .NET Core 2.1+.

### On Linux/OSX, if your SonarQube server is secured

1.  Copy the server's CA certs to `/usr/local/share/ca-certificates`
2.  Run `sudo update-ca-certificates`

## Use

There are two versions of the SonarScanner for .NET. In the following commands, you need to pass an [authentication token](https://docs.sonarqube.org/latest/user-guide/user-account/generating-and-using-tokens/) using the `sonar.token` property. Any project file accepted by MSBuild.exe or dotnet can be used, for example `.sln`, `.proj`, `.csproj`, or `.vbproj`.

### "Classic" .NET framework invocation

The first version is based on the "classic" .NET Framework. To use it, execute the following commands from the root folder of your project:

```
SonarScanner.MSBuild.exe begin /k:"project-key" /d:sonar.token="myAuthenticationToken"
MSBuild.exe <path to project file or .sln file> /t:Rebuild
SonarScanner.MSBuild.exe end /d:sonar.token="myAuthenticationToken"
```

Note: On macOS or Linux, you can also use `mono <path to SonarScanner.MSBuild.exe>`.

### .NET Core and .NET Core global tool invocation

The second version is based on .NET Core which has a very similar usage:

```
dotnet <path to SonarScanner.MSBuild.dll> begin /k:"project-key" /d:sonar.token="<token>"
dotnet build <path to project file or .sln file>
dotnet <path to SonarScanner.MSBuild.dll> end /d:sonar.token="<token>" 
```

The .NET Core version can also be used as a .NET Core Global Tool. After installing the Scanner as a global tool as described above it can be invoked as follows:

```
dotnet tool install --global dotnet-sonarscanner
dotnet sonarscanner begin /k:"project-key" /d:sonar.token="<token>"
dotnet build <path to project file or .sln file>
dotnet sonarscanner end /d:sonar.token="<token>"
```

<table><tbody><tr><td><strong>Scanner Flavor</strong></td><td><strong>Invocation</strong></td></tr><tr><td>.NET 5+</td><td><code>dotnet &lt;path to SonarScanner.MSBuild.dll&gt;</code>&nbsp;etc.</td></tr><tr><td>.NET Core Global Tool</td><td><code>dotnet sonarscanner begin</code>&nbsp;etc.</td></tr><tr><td>.NET Core 2.0+</td><td><code>dotnet &lt;path to SonarScanner.MSBuild.dll&gt;</code>&nbsp;etc.</td></tr><tr><td>.NET Framework 4.6+</td><td><code>SonarScanner.MSBuild.exe begin</code>&nbsp;etc.</td></tr></tbody></table>

Notes:

-   The .NET Core version of the scanner does not support TFS XAML builds and automatic finding/conversion of Code Coverage files. Apart from that, all versions of the Scanner have the same capabilities and command line arguments.

## Analysis steps

### Begin

The begin step is executed when you add the `begin` command line argument. It hooks into the build pipeline, downloads SonarQube quality profiles and settings, and prepares your project for analysis.

Command Line Parameters:

<table><tbody><tr><td><strong>Parameter</strong></td><td><strong>Description</strong></td></tr><tr><td><code>/k:&lt;project-key&gt;</code></td><td>[required] Specifies the key of the analyzed project in SonarQube</td></tr><tr><td><code>/n:&lt;project name&gt;</code></td><td>[optional] Specifies the name of the analyzed project in SonarQube. Adding this argument will overwrite the project name in SonarQube if it already exists.</td></tr><tr><td><code>/v:&lt;version&gt;</code></td><td>[recommended] Specifies the version of your project.</td></tr><tr><td><code>/d:sonar.token=&lt;token&gt;</code></td><td>[recommended] Requires version 5.13+. Use <code>sonar.login</code> for earlier versions.<p>Specifies the&nbsp;<a href="https://docs.sonarqube.org/9.8/user-guide/user-account/generating-and-using-tokens/">authentication token</a>&nbsp;used to authenticate with to SonarQube. If this argument is added to the begin step, it must also be added to the end step.</p></td></tr><tr><td><code>/d:sonar.clientcert.path=&lt;ClientCertificatePath&gt;</code></td><td>[optional] Specifies the path to a client certificate used to access SonarQube. The certificate must be password protected.</td></tr><tr><td><code>/d:sonar.clientcert.password=&lt;ClientCertificatePassword&gt;</code></td><td>[optional] Specifies the password for the client certificate used to access SonarQube. Required if a client certificate is used. If this argument is added to the begin step, it must also be added to the end step.</td></tr><tr><td><code>/d:sonar.verbose=true</code></td><td>[optional] Sets the logging verbosity to detailed. Add this argument before sending logs for troubleshooting.</td></tr><tr><td><code>/d:sonar.dotnet.excludeTestProjects=true</code></td><td>[optional] Excludes Test Projects from analysis. Add this argument to improve build performance when issues should not be detected in Test Projects.</td></tr><tr><td><code>/d:&lt;analysis-parameter&gt;=&lt;value&gt;</code></td><td>[optional] Specifies an additional SonarQube&nbsp;<a href="https://docs.sonarqube.org/latest/analyzing-source-code/analysis-parameters/">analysis parameter</a>, you can add this argument multiple times.</td></tr><tr><td><code>/s:&lt;custom.analysis.xml&gt;</code></td><td>[optional] Overrides the&nbsp;<code>$install_directory/SonarQube.Analysis.xml</code>. You need to give the absolute path to the file.</td></tr></tbody></table>

For detailed information about all available parameters, see [analysis parameters](https://docs.sonarqube.org/9.8/analyzing-source-code/analysis-parameters/).

### Build

Between the `begin` and `end` steps, you need to build your project, execute tests and generate code coverage data. This part is specific to your needs and it is not detailed here. See [.NET test coverage](https://docs.sonarqube.org/latest/analyzing-source-code/test-coverage/dotnet-test-coverage/) for details.

### End

The end step is executed when you add the "end" command line argument. It cleans the MSBuild/dotnet build hooks, collects the analysis data generated by the build, the test results, the code coverage and then uploads everything to SonarQube

There are only two additional arguments that are allowed for the end step:

<table><tbody><tr><td><strong>Parameter</strong></td><td><strong>Description</strong></td></tr><tr><td><code>/d:sonar.token=&lt;token&gt;</code></td><td>This argument is required if it was added to the begin step.</td></tr><tr><td><code>/d:sonar.clientcert.password=&lt;ClientCertificatePassword&gt;</code></td><td>This argument is required if it was added to the begin step. Specifies the password for the client certificate used to access SonarQube.</td></tr></tbody></table>

### Known limitations

-   MSBuild versions older than 14 are not supported.
-   Web Application projects are supported. Legacy Web Site projects are not.
-   Projects targeting multiple frameworks and using preprocessor directives could have slightly inaccurate metrics (lines of code, complexity, etc.) because the metrics are calculated only from the first of the built targets.

## Code coverage

See [.NET test coverage](https://docs.sonarqube.org/9.8/analyzing-source-code/test-coverage/dotnet-test-coverage/) for details.

## Excluding projects from analysis

Some project types, such as [Microsoft Fakes](https://msdn.microsoft.com/en-us/library/hh549175.aspx), are automatically excluded from analysis. To manually exclude a different type of project from the analysis, place the following in its .xxproj file.

```
<!-- in .csproj –->
<PropertyGroup>
  <!-- Exclude the project from analysis -->
  <SonarQubeExclude>true</SonarQubeExclude>
</PropertyGroup>
```

## Advanced topics

**Analyzing MSBuild 12 projects with MSBuild 14**  
The Sonar Scanner for .NET requires your project to be built with MSBuild 14.0. We recommend installing Visual Studio 2015 update 3 or later on the analysis machine in order to benefit from the integration and features provided with the Visual Studio ecosystem (VSTest, MSTest unit tests, etc.).

Projects targeting older versions of the .NET Framework can be built using MSBuild 14.0 by setting the "TargetFrameworkVersion" MSBuild property as documented by Microsoft:

-   [How to: Target a Version of the .NET Framework](https://msdn.microsoft.com/en-us/library/bb398202.aspx)
-   [MSBuild Target Framework and Target Platform](https://msdn.microsoft.com/en-us/library/hh264221.aspx)

For example, if you want to build a .NET 3.5 project, but you are using a newer MSBuild version:

```
MSBuild.exe /t:Rebuild /p:TargetFramework=net35
```

If you do not want to switch your production build to MSBuild 14.0, you can set up a separate build dedicated to the SonarQube analysis.

**Detection of test projects**

You can read a full description of that subject on our wiki [here](https://github.com/SonarSource/sonar-scanner-msbuild/wiki/Analysis-of-product-projects-vs.-test-projects).

**Per-project analysis parameters** Some analysis parameters can be set for a single MSBuild project by adding them to its .csproj file.

```
<!-- in .csproj -->
<ItemGroup>
  <SonarQubeSetting Include="sonar.stylecop.projectFilePath">
    <Value>$(MSBuildProjectFullPath)</Value>
  </SonarQubeSetting>
</ItemGroup>
```

**Analyzing languages other than C# and VB**

For newer SDK-style projects (used by .NET Core, .NET 5, and later), the SonarScanner for .NET will analyze all file types that are supported by the available language plugins unless explicitly excluded.

For older-style projects, the scanner will only analyze files that are listed in the `.csproj` or `.vbproj` project file. Normally this means that only C# and VB files will be analyzed. To enable the analysis of other types of files, include them in the project file.

More specifically, any files included by an element of one of the `ItemTypes` in [this list](https://github.com/SonarSource/sonar-scanner-msbuild/blob/master/src/SonarScanner.MSBuild.Tasks/Targets/SonarQube.Integration.targets#L112) will be analyzed automatically. For example, the following line in your `.csproj` or `.vbproj` file

```
<Content Include="foo\bar\*.js" />
```

will enable the analysis of all JS files in the directory `foo\bar` because `Content` is one of the `ItemTypes` whose includes are automatically analyzed.

You can also add `ItemTypes` to the default list by following [these directions](https://github.com/SonarSource/sonar-scanner-msbuild/blob/master/src/SonarScanner.MSBuild.Tasks/Targets/SonarQube.Integration.targets#L75).

You can check which files the scanner will analyze by looking in the file .sonarqube\\out\\sonar-project.properties after MSBuild has finished.

**Using SonarScanner for .NET with a proxy**  
On build machines that connect to the Internet through a proxy server you might experience difficulties connecting to SonarQube. To instruct the Java VM to use the system proxy settings, you need to set the following environment variable before running the SonarScanner for .NET:

```
SONAR_SCANNER_OPTS = "-Djava.net.useSystemProxies=true"
```

To instruct the Java VM to use specific proxy settings or when there is no system-wide configuration use the following value:

```
SONAR_SCANNER_OPTS = "-Dhttp.proxyHost=yourProxyHost -Dhttp.proxyPort=yourProxyPort"
```

Where _yourProxyHost_ and _yourProxyPort_ are the hostname and the port of your proxy server. There are additional proxy settings for HTTPS, authentication and exclusions that could be passed to the Java VM. For more information see the following article: [https://docs.oracle.com/javase/8/docs/technotes/guides/net/proxies.html](https://docs.oracle.com/javase/8/docs/technotes/guides/net/proxies.html).

You also need to set the appropriate proxy environment variables used by .NET. `HTTP_PROXY`, `HTTPS_PROXY`, `ALL_PROXY`, and `NO_PROXY` are all supported. You can find more details [here](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpclient.defaultproxy?view=net-5.0).

## Known issues

**I have multiple builds in the same pipeline, each of them getting analyzed even if the Run Code Analysis has already been executed**

We don't uninstall the global `ImportBefore` targets to support concurrent analyses on the same machine. The main effect is that if you build a solution where a .sonarqube folder is located nearby, then the sonar-dotnet analyzer will be executed along your build task.

To avoid that, you can disable the targets file by adding a build parameter:

```
msbuild /p:SonarQubeTargetsImported=true
dotnet build -p:SonarQubeTargetsImported=true
```