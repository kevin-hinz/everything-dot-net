[SOURCE](https://community.sonarsource.com/t/coverage-test-data-generate-reports-for-c-vb-net/9871)

To include test coverage in your analysis, you’ll need to:

-   provide the relevant analysis parameter to the [SonarScanner for MSBuild 1.9k](https://redirect.sonarsource.com/doc/install-configure-scanner-msbuild.html) `begin` step
-   execute your unit tests just after `build` and before the [SonarScanner for MSBuild 1.9k](https://redirect.sonarsource.com/doc/install-configure-scanner-msbuild.html)’s `end` step.

We support:

-   Import of coverage reports from VSTest, dotCover, OpenCover, Coverlet, Altcover and NCover 3.
-   Import of unit test results from VSTest, NUnit and xUnit.

We’ve documented here for each engine:

-   what is the relevant analysis parameter to pass during the `begin` step.
-   how to generate the report.

In case you encounter problems along the way, check the [Troubleshooting guide for .NET code coverage import 815](https://community.sonarsource.com/t/coverage-troubleshooting-guide-for-net-code-coverage-import/37151).

## [](https://community.sonarsource.com/t/coverage-test-data-generate-reports-for-c-vb-net/9871#import-test-coverage-reports-1)Import test coverage reports

To import a test coverage report, during the _Begin_ step you need to pass a parameter that points to the file that will be generated:

-   for C#
    -   `sonar.cs.vscoveragexml.reportsPaths` for Visual Studio Code Coverage
    -   `sonar.cs.dotcover.reportsPaths` for dotCover
    -   `sonar.cs.opencover.reportsPaths` for OpenCover, Coverlet or Altcover
    -   `sonar.cs.ncover3.reportsPaths` for NCover3 **(deprecated)**
-   for VB .NET
    -   `sonar.vbnet.vscoveragexml.reportsPaths` for Visual Studio Code Coverage
    -   `sonar.vbnet.dotcover.reportsPaths` for dotCover
    -   `sonar.vbnet.opencover.reportsPaths` for OpenCover, Coverlet or Altcover
    -   `sonar.vbnet.ncover3.reportsPaths` for NCover3 **(deprecated)**

You can find more details about this in [Test Coverage & Execution 84](https://docs.sonarqube.org/latest/analysis/test-coverage/dotnet-test-coverage/).

### [](https://community.sonarsource.com/t/coverage-test-data-generate-reports-for-c-vb-net/9871#visual-studio-code-coverage-2)Visual Studio Code Coverage

**Run Unit Tests To Collect Code Coverage**

`"%VSINSTALLDIR%\Common7\IDE\CommonExtensions\Microsoft\TestWindow\vstest.console.exe" /EnableCodeCoverage "UnitTestProject1\bin\Debug\UnitTestProject1.dll"`

**Convert the Code Coverage Report from Binary into XML**

`"%VSINSTALLDIR%\Team Tools\Dynamic Code Coverage Tools\CodeCoverage.exe" analyze /output: "%CD%\VisualStudio.coveragexml" "%CD%\VisualStudio.coverage"`

Note: If you use **dotnet** instead of **vstest**, the commands are slightly different, but the output is the same (`.coverage` and `.xml` files)  
`dotnet test ... --collect "DotnetCodeCoverage"`  
`CodeCoverage.exe analyze /output: "%CD%\DotnetCoverage.coveragexml" "%CD%\DotnetCoverage.coverage"`

**External documentation**

-   [VSTest.Console.exe 271](https://docs.microsoft.com/en-us/visualstudio/test/vstest-console-options?view=vs-2019)
-   [dotnet test 617](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-test)

### [](https://community.sonarsource.com/t/coverage-test-data-generate-reports-for-c-vb-net/9871#dotcover-3)dotCover

**Run Unit Tests To Collect Code Coverage**

`"%LOCALAPPDATA%\JetBrains\Installations\dotCover02\dotCover.exe" analyse /ReportType=HTML /Output="%CD%\dotCover.html" "/TargetExecutable=%VSINSTALLDIR%\Common7\IDE\CommonExtensions\Microsoft\TestWindow\vstest.console.exe" /TargetWorkingDir=. "/TargetArguments=UnitTestProject1\bin\Debug\UnitTestProject1.dll"`

### [](https://community.sonarsource.com/t/coverage-test-data-generate-reports-for-c-vb-net/9871#opencover-4)OpenCover

**Run Unit Tests To Collect Code Coverage**

`"%LOCALAPPDATA%\Apps\OpenCover\OpenCover.Console.exe" -output:"%CD%\opencover.xml" -register:user -target:"%VSINSTALLDIR%\Common7\IDE\CommonExtensions\Microsoft\TestWindow\vstest.console.exe" -targetargs:"UnitTestProject1\bin\Debug\UnitTestProject1.dll"`

### [](https://community.sonarsource.com/t/coverage-test-data-generate-reports-for-c-vb-net/9871#coverlet-5)Coverlet

**Run Unit Tests To Collect Code Coverage in OpenCover format**

`coverlet UnitTestProject1\bin\Debug\UnitTestProject1.dll --target "%VSINSTALLDIR%\Common7\IDE\CommonExtensions\Microsoft\TestWindow\vstest.console.exe" --targetargs "UnitTestProject1\bin\Debug\UnitTestProject1.dll" --format opencover`

### [](https://community.sonarsource.com/t/coverage-test-data-generate-reports-for-c-vb-net/9871#altcover-6)Altcover

Altcover by default is using the OpenCover format, that is already supported.

**Add the package to your test project**  
`dotnet add package AltCover`

**Run tests with coverage**  
`dotnet test /p:AltCover=true`

### [](https://community.sonarsource.com/t/coverage-test-data-generate-reports-for-c-vb-net/9871#ncover-3-deprecated-7)**NCover 3 (deprecated)**

**Run Unit Tests To Collect Code Coverage**

`"%ProgramFiles(x86)%\NCover\NCover.Console.exe" "%VSINSTALLDIR%\Common7\IDE\CommonExtensions\Microsoft\TestWindow\vstest.console.exe" "UnitTestProject1\bin\Debug\UnitTestProject1.dll" //x "%CD%\coverage.nccov"`

## [](https://community.sonarsource.com/t/coverage-test-data-generate-reports-for-c-vb-net/9871#import-unit-test-results-8)Import unit test results

To import a test execution report, during the _Begin_ step you need to pass a parameter that points to the file that will be generated:

-   for C#:
    -   `sonar.cs.vstest.reportsPaths` for VSTest
    -   `sonar.cs.nunit.reportsPaths` for NUnit
    -   `sonar.cs.xunit.reportsPaths` for xUnit
-   for VB .NET
    -   `sonar.vbnet.vstest.reportsPaths` for VSTest
    -   `sonar.vbnet.nunit.reportsPaths` for NUnit
    -   `sonar.vbnet.xunit.reportsPaths` for xUnit

You can find more details about this in [Test Coverage & Execution 1.7k](https://docs.sonarqube.org/latest/analysis/coverage/).

### [](https://community.sonarsource.com/t/coverage-test-data-generate-reports-for-c-vb-net/9871#vstest-9)VSTest

**Run Unit Tests and Save Results in the “TestResults” folder using a generated \*.trx filename**

`"%VSINSTALLDIR%\Common7\IDE\CommonExtensions\Microsoft\TestWindow\vstest.console.exe" /Logger:trx "UnitTestProject1\bin\Debug\UnitTestProject1.dll"`

### [](https://community.sonarsource.com/t/coverage-test-data-generate-reports-for-c-vb-net/9871#nunit-10)NUnit

**Run Unit Tests and Save Results in file “NUnitResults.xml”**

`packages\NUnit.ConsoleRunner.3.7.0\tools\nunit3-console.exe --result=NUnitResults.xml "NUnitTestProject1\bin\Debug\NUnitTestProject1.dll"`

or, for older NUnit 2

`"%ProgramFiles(x86)%\NUnit 2.6.4\bin\nunit-console.exe /result=NUnitResults.xml "NUnitTestProject1\bin\Debug\NUnitTestProject1.dll"`

### [](https://community.sonarsource.com/t/coverage-test-data-generate-reports-for-c-vb-net/9871#xunit-11)xUnit

**Run Unit Tests and Save Results in file “XUnitResults.xml”**

`packages\xunit.runner.console.2.1.0\tools\xunit.console.exe XUnitProject1\bin\Debug\XUnitProject1.dll -xml XUnitResults.xml`