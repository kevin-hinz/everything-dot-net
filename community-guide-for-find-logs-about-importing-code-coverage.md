[SOURCE](https://community.sonarsource.com/t/how-to-find-logs-about-importing-code-coverage/73317)

When coverage data isn’t appearing as expected in SonarQube / SonarCloud, the first place you should look is the **scanner logs**.

## [](https://community.sonarsource.com/t/how-to-find-logs-about-importing-code-coverage/73317#where-do-i-find-the-scanner-logs-1)Where do I find the scanner logs?

![:warning:](https://emoji.discourse-cdn.com/twitter/warning.png?v=12 ":warning:") This will likely be broken out into its own guide soon

The Scanner logs are the output of executing one of the [scanners 8](https://docs.sonarqube.org/latest/analysis/overview/), either directly or wrapped by one of many integrations, such as the [SonarScanner for Jenkins 3](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-jenkins/), [SonarScanner for Azure DevOps 13](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-azure-devops/), [SonarQube](https://github.com/SonarSource/sonarqube-scan-action)/[SonarCloud 3](https://github.com/marketplace/actions/sonarcloud-scan) GitHub Actions or [SonarQube](https://bitbucket.org/sonarsource/sonarqube-scan/src/master/) / [SonarCloud 1](https://bitbucket.org/sonarsource/sonarcloud-scan/src/master/) Bitbucket Pipes.

![:thinking:](https://emoji.discourse-cdn.com/twitter/thinking.png?v=12 ":thinking:") The answer to the question "**how was Sonar analysis enabled for this repository"** should usually answer the question, **“where do I find the scanner logs”**?

The following list should help you find the scanner logs if you’re really not sure where to look – as well as how to add a `DEBUG` flag for more verbose log information.

| Scanner |	Command	|  Debug Flag | Notes |
| :-------- | :----- | :----- | :----- |
| SonarScanner for Gradle	| gradle sonarqube |	--debug	| Almost no logs are included without --info or --debug
| SonarScanner for .NET (.NET Global Tool) |	dotnet sonarscanner end	| /d:sonar.verbose=true	|  |
| SonarScanner for .NET (.NET Core) |	dotnet <path to SonarScanner.MSBuild.dll> end |	/d:sonar.verbose=true	|  |
| SonarScanner for .NET (.NET Framework) |	SonarScanner.MSBuild.exe end |	/d:sonar.verbose=true	|  |	
| SonarScanner for Maven |	mvn sonar:sonar |	-X	|  |	
| SonarScanner for Ant |	ant sonar |	-v	|  |	
| SonarScanner |	sonar-scanner |	-X	|  |


## [](https://community.sonarsource.com/t/how-to-find-logs-about-importing-code-coverage/73317#how-do-i-find-information-about-coverage-import-2)How do I find information about coverage import?

A **Scan** is made up of various **Sensors** with different duties such as analyzing code, and also the import of coverage data. In the logs, you’ll find information about what each sensor has done, including warnings/errors.

Here’s an example of the `JavaScript/TypeScript Coverage [javascript]` sensor

```
INFO: Sensor JavaScript/TypeScript Coverage [javascript]
INFO: No LCOV files were found using /home/ec2-user/agent/_work/1/s/**/coverage/lcov.info
WARN: No coverage information will be saved because all LCOV files cannot be found.
INFO: Sensor JavaScript/TypeScript Coverage [javascript] (done) | time=703ms
```

The following are the sensors you should be on the lookout for when diagnosing/reporting coverage import issues. You can directly search the text in the “Sensor” column in your scanner logs.

| Language |	Sensor	|
| :----- | :----- |
| Generic Test Data |	Generic Coverage Report |
| Apex |	Test coverage Sensor for Apex [apex] |
| C/C++/Objective-C (gcov) |	gcov [cpp] |
| C/C++/Objective-C (llvm-cov) |	llvm-cov [cpp] |
| C/C++/Objective-C (Visual Studio Coverage) |	VisualStudioCoverage [cpp] |
| C/C++/Objective-C (Bullseye) |	bullseye [cpp] |
| C# |	C# Tests Coverage Report Import [csharp] |
| Flex |	Flex Cobertura [flex] |
| Go |	Go Cover sensor for Go coverage [go] |
| Java/JVM-based language (Scala/Kotlin) |	JaCoCo XML Report Importer (Jacoco) |
| JavaScript/TypeScript |	JavaScript/TypeScript Coverage [javascript] |
| PHP |	Sensor PHP sensor [php] |
| Python |	Cobertura Sensor for Python coverage [python] |
| Ruby |	SimpleCov Sensor for Ruby coverage [ruby] |
| Scala |	Scoverage sensor for Scala coverage [scala] |
| Swift |	Swift Code Quality and Security [swift] |
| VB.NET |	VB.NET Tests Coverage Report Import [vbnet] |



It’s these logs (between the start of the sensor and when it’s `(done)` that are most useful in understanding what happened during the import of coverage data.