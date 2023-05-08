[SOURCE](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279)

This guide is intended to help SonarQube and SonarCloud users investigate .NET analysis performance issues. This effort can lead to:

-   self-diagnose and fix a performance issue
-   gather information to [open a well-defined topic on this community forum](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#report-a-performance-problem-34)
-   help SonarSource improve the performance of the products

Before discussing how to investigate performance issues, you should first understand the different steps of the analysis.

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#introduction-to-the-sonarsource-net-static-analysis-1)Introduction to the SonarSource .NET static analysis

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#state-of-the-art-static-analysis-2)State-of-the-art static analysis

Our .NET analysis detects [code quality 7](https://www.sonarsource.com/why-us/code-quality/) and [security 5](https://www.sonarsource.com/why-us/code-security/) issues, which analyze code at different levels:

-   syntax level - these rules are the quickest. Linters like StyleCop.Analyzers usually analyze at syntax level and are very fast.
-   semantic level - involves knowing the type of symbols - this means the rule is more accurate (less False Positives), but slower.
-   symbolic execution - this advanced analysis technique is based on the Control Flow Graph of a method and does a simulation of the program execution to detect tricky bugs such as undisposed usings or null dereference. This advanced technique can have a higher performance footprint.
-   taint analysis - it follows user input across all feasible paths inside the program (cross-file), to find injection vulnerabilities. It usually has a high resource usage.

While we are continuously making performance improvements to our products, it is important to understand that static code analysis brings a lot of benefits that come with a cost in terms of performance.

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#introduction-to-the-sonarqubesonarcloud-net-analysis-3)Introduction to the SonarQube/SonarCloud .NET analysis

Components involved during the analysis:

-   Roslyn - the .NET Compiler Framework, exposing APIs for static code analyzers. The SonarSource C# static code analyzer is a Roslyn analyzer.
-   MSBuild - the Microsoft Build Engine - invokes the compiler (with the path to Roslyn analyzers as argument).
-   S4NET - the [Scanner for .NET 31](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-msbuild/), which is used by [SonarScanner for Azure DevOps 8](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-azure-devops/). It prepares the integration with MSBuild during the **BEGIN** step (i.e. `SonarQubePrepare` / `SonarCloudPrepare` / “Prepare Analysis Configuration”) - it downloads the analyzers from SonarQube/SonarCloud and configures MSBuild to use them. What happens during the **END** step (i.e. `SonarQubeAnalyze` / `SonarCloudAnalyze` / “Run Code Analysis”) is detailed below in [Second phase](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#second-phase-the-end-step-5).

There are two phases of static analysis: one during the build and one after the build.

### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#first-phase-the-compilation-4)First phase - the compilation

The compiler (`csc.exe` or `csc.dll`) invokes the configured Roslyn analyzers during the build. Almost all [C# rules 6](https://rules.sonarsource.com/csharp) get executed in this step by the Roslyn analyzer `SonarAnalyzer.CSharp`. The results of this analysis are stored locally on disk.

In SonarQube commercial editions and SonarCloud, UCFG (Universal Control Flow Graph - an abstract representation of the program flow) files are created and stored on disk.

### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#second-phase-the-end-step-5)Second phase - the END step

During the end step (“SonarCloudAnalyze”):

-   the results of the Roslyn analysis (`First phase`) are read from disk and sent to the server.
-   code coverage reports are read and sent to the server.

In SonarQube commercial editions and SonarCloud, during the END step the [injection vulnerabilities rules 2](https://rules.sonarsource.com/csharp/tag/injection) are executed. The security taint analysis engine reads the UCFG files from disk and searches for vulnerabilities in the program.

Under the hood, the Scanner for .NET END step invokes an embedded version of the [SonarScanner 1](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/).

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#read-the-logs-6)Read the logs

Now that you have a high-level understanding of how the Sonar analysis works, you can start looking at the logs.

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#get-an-insight-of-rule-execution-times-7)Get an insight of rule execution times

To retrieve the analyzer execution times, the `reportanalyzer` property of the build has to be set to true and the output verbosity has to be diagnostic.  
If you run the build with reportanalyzer enabled, please share the results with us. Your feedback will be more than welcome and we will try to fix the performance issues.

### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#msbuild-8)msbuild

```
MsBuild.exe /t:Rebuild /p:reportanalyzer=true /v:d
```

### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#dotnet-build-9)dotnet build

```
dotnet build -p:reportanalyzer=true -v:diag
```

### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#example-of-the-logs-to-look-out-for-10)Example of the logs to look out for

```
Total analyzer execution time: 10.502 seconds.
NOTE: Elapsed time may be less than analyzer execution time because analyzers can run concurrently.
Time (s)    %   Analyzer
8.258   78   SonarAnalyzer.CSharp, Version=8.25.0.0, Culture=neutral, PublicKeyToken=c5b62af9de6d7244
2.421   23      SonarAnalyzer.Rules.SymbolicExecution.SymbolicExecutionRunner (S1944, S2053, S2259, S2583, S2589, S3329, S3655, S3900, S3966, S4158, S5773)
0.415    3      SonarAnalyzer.Rules.CSharp.InvalidCastToInterface (S1944)
0.358    3      SonarAnalyzer.Rules.CSharp.SymbolReferenceAnalyzer (S9999-symbolRef)
0.261    2      SonarAnalyzer.Rules.CSharp.TokenTypeAnalyzer (S9999-token-type)
0.242    2      SonarAnalyzer.Rules.CSharp.PrivateFieldUsedAsLocalVariable (S1450)
0.217    2      SonarAnalyzer.Rules.CSharp.MetricsAnalyzer (S9999-metrics)
0.213    2      SonarAnalyzer.Rules.CSharp.DeadStores (S1854)
0.179    1      SonarAnalyzer.Rules.CSharp.DoNotCallAssemblyLoadInvalidMethods (S3885)

...

1.534   14   SonarAnalyzer.Security, Version=9.0.0.12669, Culture=neutral, PublicKeyToken=null
1.534   14      SonarAnalyzer.Security.CSharp.UcfgGenerator (S2076, S2078, S2083, S2091, S2631, S3649, S5131, S5135, S5144, S5145, S5146, S5167, S5334, S6096)

...
```

#### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#how-to-read-the-reportanalyzer-logs-11)How to read the `reportanalyzer` logs

The first line (`SonarAnalyzer.CSharp, Version=8.25.0.0`) specifies the total time took by all the rules inside the `SonarAnalyzer.CSharp` analyzer. The percentage (in this example `78%`) specifies the share of time consumed by this analyzer out of all Roslyn analyzers which were executed during the build (`10.502 seconds`). Below each Roslyn analyzer class (identified by its fully qualified name) is listed. A class can execute one or more rules - for example the `SymbolicExecutionRunner` executes all symbolic execution rules.

In SonarQube commercial editions and SonarCloud, there is a second `SonarAnalyzer.Security` analyzer which generates UCFG files for the vulnerability analysis done in the `END` step.

Some analyzers generated metadata needed by SonarQube/SonarCloud for code highlighting and listing metrics (the `S9999-...` rules) and do not correspond to a rule (thus are not configurable in the Quality Profile).

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#evaluate-the-time-took-by-each-project-12)Evaluate the time took by each project

Sometimes a project might have a particular pattern that makes it an outlier in terms of performance. Verify in the build logs how long each project takes to analyze.

And even if you don’t have outliers - consider removing projects which do not benefit from static code analysis to improve the overall performance of the build. For example third-party code which is included in the build, sample projects used during demos etc.

You can use the following snippet of PowerShell code to find out the project for each report:

```
dotnet build ... -v:d -p:reportanalyzer=true > build_logs.txt
select-string -path “build_logs.txt ” -pattern “NOTE: Elapsed time may be less than analyzer execution time because analyzers can run concurrently.” -Context 2,30 > analyzer_times.txt
```

And in `analyzer_times.txt` you will find the project before the time summary. In the below example, you can identify that the performance summary is related to `Foo.csproj`. You may want to read the [Logging in a multi-processor environment docs 5](https://docs.microsoft.com/en-us/visualstudio/msbuild/logging-in-a-multi-processor-environment?view=vs-2019) in case your build uses multiple processors.

```
  2021-08-04T16:57:40.7829996Z Foo\Bar\Baz\LastFile.cs(65,20): warning CS0219: The variable 'fooId' is assigned but its value is never
used [c:\agent\_work\60\s\Foo\Foo.csproj]
  2021-08-04T16:57:40.7832199Z   Total analyzer execution time: 0.483 seconds.
  2021-08-04T16:57:40.7832694Z   NOTE: Elapsed time may be less than analyzer execution time because analyzers can run concurrently.
  2021-08-04T16:57:40.7833000Z   Time (s)    %   Analyzer
  2021-08-04T16:57:40.7833335Z   0.483  100   SonarAnalyzer.CSharp, Version=8.14.0.0, Culture=neutral, PublicKeyToken=c5b62af9de6d7244
```

You can go to the [A single project is taking most of the time](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#a-single-project-is-taking-most-of-the-time-33) troubleshooting section to find out how to exclude a project from analysis.

### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#verbose-logging-for-the-end-step-13)Verbose logging for the END step

If the END step (`Phase 2`) is taking too long, you should read the verbose logs to pinpoint the issue.

To run the S4NET in verbose mode, you need to set the `sonar.verbose` property.

```
SonarScanner.MSBuild.exe begin /k:“MyProject” /d:sonar.verbose=true
```

Similarly for the Azure DevOps extension (either `SonarQubePrepare` or `SonarCloudPrepare`):

```
      - task: SonarCloudPrepare@1
        displayName: 'Code Analysis - Begin'
        inputs:
          SonarCloud: 'SonarCloud'
          organization: 'FooCompany'
          scannerMode: 'MSBuild'
          projectKey: 'FooKey'
          projectName: 'FooName'
          projectVersion: '$(FOO_VERSION)'
          extraProperties: |
            sonar.verbose=true
```

The relevant debug logs will be in the END step (SonarCloudAnalyze).

Some keywords to search for in the scanner logs:

-   `Sensor CSharpSecuritySensor [security]` and `Sensor CSharpSecuritySensor [security] (done) | time=` - between these two lines you can see what takes the most time during the injection vulnerability analysis - if it’s one specific rule which eats up more time, you can consider disabling it after reporting the problem to SonarSource to investigate
-   `Sensor C# [csharp] (done) | time=` - see how much time importing the results of the Roslyn analysis (`Phase 1`) took
-   `Sensor C# Tests Coverage Report Import [csharp] (done) | time=` - see how much time parsing and importing code coverage took
-   you can also look at the time took by other sensors to see if there is an outlier
-   to verify the time taken by the injection vulnerability rules, search with the regex `rule: S\d*` and check how long each rule took

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#troubleshooting-prerequisites-14)Troubleshooting prerequisites

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#verify-the-software-version-15)Verify the software version

Verify the software version to make sure you are using a supported version of SonarQube (not required if using SonarCloud) and the latest S4NET. You need to provide the software versions when opening a topic on the community forum.

### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#sonarqube-versions-16)SonarQube versions

SonarQube version: `http://<SQ_URL>:<SQ_PORT>/api/server/version`

Please read the [Download SonarQube 1](https://www.sonarqube.org/downloads/) to verify if you are on the LTS version or the latest version. Consider updating to the latest to benefit from the latest performance improvements in the product.

### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#s4net-version-17)S4NET version

In the logs of the `BEGIN` step (`Prepare analysis`) - `SonarScanner for MSBuild X.Y` (SonarScanner for MSBuild is the historical name of SonarScanner for .NET).

If not using the Azure DevOps integration, verify if you are using the latest [SonarScanner for .NET 31](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-msbuild/).

### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#language-plugin-version-18)Language plugin version

-   On SonarCloud `https://sonarcloud.io/api/plugins/installed` and on SonarQube `http://<SQ_URL>:<SQ_PORT>/api/plugins/installed` - search the JSON by “Code Analyzer for C#” and check the “version” attribute
-   In the S4NET verbose logs, in the `BEGIN` step (`Prepare analysis`), search for the line `Processing plugin: csharp version X.Y.Z.N`

### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#msbuild-version-19)MSBuild version

The build engine version is printed in the logs when executing `dotnet build` or `MSBuild.exe`:

```
Microsoft (R) Build Engine version 16.9.0+57a23d249 for .NET
```

### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#verify-local-analysis-vs-ci-build-pipeline-analysis-20)Verify local analysis vs CI build pipeline analysis

The analysis execution time can depend on the CI (continuous integration) build pipeline setup, so to drill down and narrow the number of possible issues, please run a local analysis with SonarQube/SonarCloud and compare the execution times.

If the execution time is improved for the local analysis, the CI build pipeline configuration/build files may not be set up correctly.

Please take a look at

-   [Make sure you do not run the analysis twice](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#make-sure-you-do-not-run-the-analysis-twice-22)
-   [Slow disk access](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#slow-disk-access-27)

to make sure the pipeline is set up correctly.

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#troubleshooting-help-21)Troubleshooting help

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#make-sure-you-do-not-run-the-analysis-twice-22)Make sure you do not run the analysis twice

As mentioned in the introduction, the S4NET `BEGIN` (`Prepare analysis`) step instructs MSBuild to run the SonarSource Roslyn analyzers during compilation.

This is done by:

-   putting the [SonarQube.Integration.ImportBefore.targets](https://github.com/SonarSource/sonar-scanner-msbuild/blob/master/src/SonarScanner.MSBuild.Tasks/Targets/SonarQube.Integration.ImportBefore.targets) file in the [ImportBefore](https://docs.microsoft.com/en-us/visualstudio/msbuild/customize-your-build?view=vs-2019#msbuildextensionspath-and-msbuilduserextensionspath) MSBuild folder (usually in `%LOCALAPPDATA%\Microsoft\MSBuild\<version>\Microsoft.Common.targets\ImportBefore`) - this targets file will be imported by every MSBuild execution on the system, checking if the `.sonarqube` folder is present in the build directory
-   if the `.sonarqube` exists (the `BEGIN` step has been executed), the [SonarQube.Integration.targets](https://github.com/SonarSource/sonar-scanner-msbuild/blob/master/src/SonarScanner.MSBuild.Tasks/Targets/SonarQube.Integration.targets) is called from `.sonarqube\bin\targets` - this will instruct the compiler to run the SonarSource analysis

This means that after the `BEGIN` step has been executed, unwanted duplicate analysis could happen in the following scenarios:

### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#dotnet-build-followed-by-dotnet-test-23)dotnet build followed by dotnet test

`dotnet test` does a full build before running the tests. So if your pipeline executes `dotnet build` and then `dotnet test`, the build will be done twice, thus the analysis will be done twice.

**Solution**: run `dotnet test --no-build`, reusing the binaries from the build.

### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#multiple-builds-in-the-same-pipeline-24)Multiple builds in the same pipeline

As mentioned in the [S4NET Known issues 31](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-msbuild/), we don’t uninstall the global ImportBefore targets to support concurrent analyses on the same machine. This means that subsequent builds may trigger the analysis even if not intended when the `.sonarqube` folder is located nearby. To explicitly tell MSBuild not to import the `SonarQube.Integration.targets`, you can pass `/p:SonarQubeTargetsImported=true` to `msbuild`/`dotnet` (this means the analysis won’t run).

### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#re-analysing-the-same-project-multiple-times-25)Re-analysing the same project multiple times

You should verify the logs of MSBuild to see if the same project gets built and analyzed multiple times, and consider improving the structure of your project to avoid this if possible.

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#increase-build-machine-resources-26)Increase build machine resources

Static code analysis is resource intensive. By increasing the number of CPUs on your build machines, MSBuild will run concurrently. When analysing various open source projects with Sonar, we have seen significant improvements when increasing the CPU count.

For example, the tables below show that increasing from 4 CPUs to 8 CPUs can reduce the analysis time by almost 40%.

[![](https://europe1.discourse-cdn.com/sonarsource/uploads/sonarcommunity/optimized/3X/3/3/3328e392a25ce53b8de39e49a38e9832847d2c6c_2_690x333.png)](https://europe1.discourse-cdn.com/sonarsource/uploads/sonarcommunity/original/3X/3/3/3328e392a25ce53b8de39e49a38e9832847d2c6c.png)

  

[![image](https://europe1.discourse-cdn.com/sonarsource/uploads/sonarcommunity/optimized/3X/a/6/a6eafae263cf98ed7df766ca4707b101ea7a34bb_2_690x325.png)](https://europe1.discourse-cdn.com/sonarsource/uploads/sonarcommunity/original/3X/a/6/a6eafae263cf98ed7df766ca4707b101ea7a34bb.png "image")

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#try-enablingdisabling-concurrent-execution-of-rules-27)Try enabling/disabling concurrent execution of rules

For users of SonarCloud and SonarQube 9.1+ you now have the ability to run the analysis in concurrent execution mode. This usually has a small positive impact on your analysis time. In SonarCloud and from SonarQube 9.5 onwards this is on by default. In the rare case that this causes analysis time to increase you can turn it off.

Concurrent Execution can be set with the following environment variable in the **BEGIN** step (i.e. `SonarQubePrepare` / `SonarCloudPrepare` / “Prepare Analysis Configuration”):

`SONAR_DOTNET_ENABLE_CONCURRENT_EXECUTION: true/false`

Please note that this will have only marginal value, because by default MSBuild runs Roslyn analyzers concurrently. This setting allows for the same rule to run concurrently with itself. See the tables above - “Concurrent” in that context means refers to run rules concurrently with themselves.

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#turn-off-the-console-logger-output-28)Turn off the console logger output

The sonar analyzer outputs any issue found as a warning. This may slow down the overall build process in certain environments. To turn off the logging of warnings to the console add the [consoleLoggerParameters:ErrorsOnly 1](https://learn.microsoft.com/visualstudio/msbuild/msbuild-command-line-reference#switches-for-loggers) option to your build:

```
msbuild /clp:ErrorsOnly
```

```
dotnet build -clp:ErrorsOnly
```

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#sonar-security-analysis-hangs-or-throws-outofmemory-exception-29)Sonar-security analysis hangs or throws OutOfMemory exception

For large codebases there may not be enough memory to execute the analysis, if that is your case, consider setting the `SONAR_SCANNER_OPTS` environment variable to increase the available memory. The `-Xmx` parameter controls the Java Heap Size used by the scanner and is followed by a number and a letter (`M` or `m` for megabytes, `G` or `g` for gigabytes).

For example, to set the maximum size of the Java Heap to 8 gigabytes:

```
SONAR_SCANNER_OPTS=-Xmx8G
```

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#slow-disk-access-30)Slow disk access

One possible reason for slowness is I/O access. To diagnose this problem you can read the output of the `END` and see how long each step takes. For example, for SonarQube commercial editions and SonarCloud, verify the timestamp before and after `Reading UCFGs`. It is fine to take a few seconds and it should raise a flag if it takes several minutes.

Possible solutions:

-   you may want to look into optimizing the disk access on your build machines (either hardware or software)
-   you may want to look into your antivirus software configuration and exclude the folder with the source code / where the analysis is done

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#a-single-rule-taking-most-of-the-analysis-time-31)A single rule taking most of the analysis time

Before disabling a rule, you should carefully consider whether the benefits it brings outweigh its performance cost. Also, please [report the performance problem to SonarSource](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#report-a-performance-problem-34) for us to improve the performance of our products.

You can disable a rule at different granularities:

-   per file (if a specific file takes too long to analyze) - via .editorconfig
-   per project (if a rule behaves badly for a specific project) - via .editorconfig
-   per analysis - remove the rule from the Quality Profile

Note: Excluded files are still going to be analyzed during the compilation and the results are filtered during the `END` step according to the exclusion settings. This is why using `sonar.exclusions` does not work for solving a performance issue, and you should use `.editorconfig` files instead.

### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#disable-a-certain-rule-for-the-entire-project-32)Disable a certain rule for the entire project

_Starting in Visual Studio 2019 version 16.3, you can configure the severity of analyzer rules, or _diagnostics_, in an [EditorConfig file 1](https://docs.microsoft.com/en-us/visualstudio/code-quality/use-roslyn-analyzers?view=vs-2019#set-rule-severity-in-an-editorconfig-file) (see [MS docs 1](https://docs.microsoft.com/en-us/visualstudio/code-quality/use-roslyn-analyzers?view=vs-2019))._

Create a .editorconfig file in a folder where your project is in the file hierarchy. The [.editorconfig will apply to 1](https://docs.microsoft.com/en-us/visualstudio/ide/create-portable-custom-editor-options?view=vs-2019#file-hierarchy-and-precedence) all applicable files at that level and below.  
In the newly created .editorconfig file you should set the severity of the rule to none and that way disable its execution.

E.g.

```
[*.cs]
dotnet_diagnostic.S1186.severity = none
dotnet_diagnostic.S1144.severity = none
```

### [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#disable-a-certain-rule-for-certain-files-33)Disable a certain rule for certain file(s)

_Starting in Visual Studio 2019 version 16.3, you can configure the severity of analyzer rules, or _diagnostics_, in an [EditorConfig file 1](https://docs.microsoft.com/en-us/visualstudio/code-quality/use-roslyn-analyzers?view=vs-2019#set-rule-severity-in-an-editorconfig-file) (see [MS docs 1](https://docs.microsoft.com/en-us/visualstudio/code-quality/use-roslyn-analyzers?view=vs-2019))._

In case you have pinpointed out what files are causing the performance issue, either create a new .editorconfig file (described above) or use an existing one to disable the rules for the file causing the issue.  
This can be achieved the following way:

E.g.

```
[**/Exclude/ThisShouldNotBeAnalyzed.cs]
dotnet_diagnostic.S1186.severity = none
dotnet_diagnostic.S1144.severity = none
```

If multiple files are slowing down the execution and cannot be targeted with a single pattern then multiple patterns can be defined to cover all the files.

E.g.

```
[**/Exclude/ThisShouldNotBeAnalyzed.cs]
dotnet_diagnostic.S1186.severity = none
dotnet_diagnostic.S1144.severity = none

[**/OtherFolder/ThisShouldBeAnalyzed.cs]
dotnet_diagnostic.S1186.severity = none
dotnet_diagnostic.S1144.severity = none
```

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#a-test-project-taking-most-of-the-analysis-time-34)A test project taking most of the analysis time

TEST projects (unit tests, integration tests) are analyzed differently from MAIN projects (production code). TEST project analysis should be very fast. You can read more about [Differences between analysis of test projects and product projects 5](https://github.com/SonarSource/sonar-scanner-msbuild/wiki/Analysis-of-product-projects-vs.-test-projects).

In case the analysis of a test project is taking long, you should check the MSBuild verbose logs (run the build with `-v:diag`) and see whether the TEST project is considered to be MAIN by mistake. You should read the logs of the `SonarQubeCategoriseProject` target, which contain statements like:

```
  Sonar: (FirstProject.csproj) categorized as MAIN project (production code).

  Sonar: (FirstProjectTests.csproj) categorized as TEST project (test code).
```

To explicitly mark a project as Test for the S4NET you need to use the `SonarQubeTestProject` MSBuild property:

```
<PropertyGroup>
  <SonarQubeTestProject>true</SonarQubeTestProject>
</PropertyGroup>
```

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#generated-code-is-analyzed-35)Generated code is analyzed

By default, our analyzers do not analyze tool-generated code ([read more](https://docs.sonarqube.org/latest/analyzing-source-code/languages/csharp/)). In case you see that generated code gets analyzed, this may be a source of performance issues.

The logic to detect generated code is in [GeneratedCodeRecognizer.cs 5](https://github.com/SonarSource/sonar-dotnet/blob/master/analyzers/src/SonarAnalyzer.Common/Helpers/GeneratedCodeRecognizer.cs) - it is looking at the file name (e.g. files ending with `g.cs` are considered as generated code), comments and attributes.

When running the sonar analysis in Debug mode, during the END step the files considered as generated code are listed.

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#a-single-project-is-taking-most-of-the-time-36)A single project is taking most of the time

First, you should read how to [Get an insight of rule execution times](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#get-an-insight-of-rule-execution-times-7) and how to [Evaluate the time took by each project](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#evaluate-the-time-took-by-each-project-12). Consider if the project is essential or not.

If the project is essential, it should be analyzed for code quality and security. If it is taking most of the analysis time, you can consider to:

-   verify if there are specific rules which take most of the time
-   analyze the project separately, in parallel, in order to improve the performance of the build. Create a new Quality Profile with less rules (keep only the essential ones) and assign this Quality Profile to the project in SonarQube/SonarCloud.

You can exclude non-essential projects from analysis with the MSBuild property `SonarQubeExclude` (read more in [the docs 31](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-msbuild/)):

```
<!-- in .csproj -->
<PropertyGroup>
  <!-- Exclude the project from analysis -->
  <SonarQubeExclude>true</SonarQubeExclude>
</PropertyGroup>
```

## [](https://community.sonarsource.com/t/the-sonar-guide-for-investigating-the-performance-of-net-analysis/47279#report-a-performance-problem-37)Report a performance problem

We are always improving the performance of our products and your feedback is essential for us. When you encounter a performance problem, please open a topic in this Community Forum, under the correct category (SonarCloud / SonarQube).

Please add the tags `performance` and `dotnet` / `csharp` for us to be able to identify and respond quickly to these issues.