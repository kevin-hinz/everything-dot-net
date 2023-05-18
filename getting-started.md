Scanning .NET application is a little different to other languages and requires a dedicated scanner that works with MSBuild. How you set this up depends on your environment. Here we will cover the most common cases:
1. Initiating a manual scan without CI integration
2. Initiating a scan from your CI
  3. Azure Devops
  4. Jenkins
  5. Github Actions
  6. Gitlab

The basic steps are as follows:
1. Install Prequisites (Java)
2. Select the correct Scanner version for you .NET runtime and Install
3. Set the correct parameters
4. (optional) Setup PR decoration
5. (optional) Setup Code Coverage Import



