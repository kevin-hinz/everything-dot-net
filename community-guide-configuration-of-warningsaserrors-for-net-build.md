[SOURCE](https://community.sonarsource.com/t/configuration-of-warningsaserrors-for-net-build/32393)

Hello everyone!

Using SonarScanner for MSBuild as a part of your .NET build pipeline may change your failing build to unexpected successful build. This topic explains details about related configuration changes done by the scanner during `begin` phase.

There are two properties `TreatWarningsAsErrors` and `WarningsAsErrors` affecting behavior of warnings during build that can be configured in a project file. When set, they can fail the build with a compiler warning present in the project.

SonarScanner for MSBuild overrides these properties to prevent the build from failing.

**How it works**

Issues found by our C# and [VB.NET 1](http://vb.net/) Analyzers (e.g. S1481) are reported as warnings. Same as compiler messages (like CS0414). Using

```
<TreatWarningsAsErrors>true</TreatWarningsAsErrors>
```

in the project changes **all** warnings to errors. In that case the build would fail for every little code smell that is raised by our analyzers. And analysis results would never be reported to SonarCloud/SonarQube due to a failing build.

To prevent this, Scanner for MsBuild injects it’s own `.targets` file in the build that can be found in `.sonarqube\bin\targets\SonarQube.Integration.targets`. Among other things, this file sets

```
<TreatWarningsAsErrors>false</TreatWarningsAsErrors>
<WarningsAsErrors></WarningsAsErrors>
```

in order to

-   prevent build from failing,
-   display analysis results in SonarCloud/SonarQube.

**What should be done**

Compiler warnings should not fail the build. They should rather be imported in SonarCloud/SonarQube with a properly configured Quality Gate. There are [several options 7](https://docs.sonarcloud.io/enriching/external-analyzer-reports/) to configure when importing external issues.

**What can not be done**

`TreatWarningsAsErrors` can not be set to `true` for SonarCloud/SonarQube analysis. That would fail every build without any reported issue.

**What can be done**

`TreatWarningsAsErrors` can be kept in the project file to fail the build locally. And `WarningsAsErrors` can be overridden by additional `.targets` file to fail the pipeline build for desired warnings. All relevant CS\* compiler warnings need to be listed in that case.

Update existing `.targets` file to inject this settings or create a new one (e.g. `Directory.Build.targets` in the project root directory) with content like this:

```
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Target Name="SonarQubeRestoreWarningsAsErrors"
        Condition=" $(SonarQubeTempPath) != '' "
        AfterTargets="OverrideRoslynCodeAnalysisProperties"
        BeforeTargets="CoreCompile">
    <PropertyGroup>
      <WarningsAsErrors>CS0414</WarningsAsErrors>
    </PropertyGroup>
  </Target>
</Project>
```

More details can be found in Microsoft docs: [CSC.WarningAsErrors 51](https://docs.microsoft.com/en-us/dotnet/api/microsoft.build.tasks.csc.warningsaserrors?view=netframework-4.8).