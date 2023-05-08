### Overview
If you are running multiple scans sequentially using different versions of the SonarScanner for .NET, you might see an error message like the following:

```
SonarScanner for MSBuild 5.0.4
Using the .NET Framework version of the Scanner for MSBuild
Pre-processing started.
Preparing working directories...
Cannot copy a different version of the SonarScanner for MSBuild assemblies because they are used by a running MSBuild/.Net Core process. To resolve this problem try one of the following:
- Analyze this project using the same version of SonarScanner for MSBuild
- Build your project with the '/nr:false' switch
Pre-processing failed. Exit code: 1
```

Note: in this scenario, the .NET FX version of the scanner is considered to be a different version from the .NET Core version of the scanner i.e. the assemblies files on disc are different, even if the release version number is the same ("v5.0.4" in the example above).

#### Root cause
By default, MSBuild leaves a build server process running for a certain amount of time after a build completes (based on observation, this is around twenty minutes). This is a performance optimisation, so any subsequent builds be faster. It is this process that is locking the scanner files, leading to the error message.

#### Solution
Building with `/nr:false` tells MSBuild not to use the build server for the build for the build being triggered. This will prevent the files from being locked after the build has finished. If you regularly see this error in your CI builds then you should consider passing the `/nr:false` option to the MSBuild/dotnet command when triggering the build.

### Manually shutting down a running build server
Passing `/nr:false` won't shutdown a build server that is already running. If you want to forceably close a running build server you can do so with one of the following commands:

* the `dotnet` build server can be shutdown using the command `dotnet build-server shutdown`.
* the NET FX MSBuild server can be shutdown using the command `vbcscompiler.exe -shutdown`.
