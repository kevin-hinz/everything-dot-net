### Community guides

- [[Coverage & Test Data] Generate Reports for C#, VB.net](https://community.sonarsource.com/t/coverage-test-data-generate-reports-for-c-vb-net/9871) - get up to speed on how to import code coverage for your .NET project.
- [[Coverage] Troubleshooting guide for .NET code coverage import](https://community.sonarsource.com/t/coverage-troubleshooting-guide-for-net-code-coverage-import/37151) - the most frequent problems regarding code coverage import, each with suggested solutions.

### Other possible problems

#### No coverage shown for .NET Core reports

Problem: code coverage is correctly configured for a .NET Core project, the scanner finds and processes the coverage reports, but no coverage information is shown in SonarQube/SonarCloud.

##### Possible causes:
*  [VSTest issue 800: Code coverage requires DebugType full](https://github.com/Microsoft/vstest/issues/800). Check that the `DebugType` property is set to `Full` in the project file.