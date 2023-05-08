[SOURCE](https://docs.sonarqube.org/latest/devops-platform-integration/azure-devops-integration/)

SonarQube's integration with Azure DevOps allows you to maintain code quality and security in your Azure DevOps repositories. It is compatible with both Azure DevOps Server and Azure DevOps Services.

With this integration, you'll be able to:

-   **Import your Azure DevOps repositories**: Import your Azure DevOps repositories into SonarQube to easily set up SonarQube projects.
-   **Analyze projects with Azure Pipelines** - Integrate analysis into your build pipeline. Starting in [Developer Edition](https://www.sonarsource.com/plans-and-pricing/developer/), the SonarQube Extension running in Azure Pipelines jobs can automatically detect branches or pull requests being built, so you don't need to specifically pass them as parameters to the scanner.
-   **Report your quality gate status to your pull requests** (starting in [Developer Edition](https://www.sonarsource.com/plans-and-pricing/developer/)): See your quality gate and code metric results right in Azure DevOps so you know if it's safe to merge your changes.

## Prerequisites

Integration with Azure DevOps Server requires Azure DevOps Server 2022, Azure DevOps Server 2020, and Azure DevOps Server 2019 (including _Express_ editions).

### Branch analysis

Community Edition doesn't support the analysis of multiple branches, so you can only analyze your main branch. Starting in [Developer Edition](https://www.sonarsource.com/plans-and-pricing/developer/), you can analyze multiple branches and pull requests.

## Importing your Azure DevOps repositories into SonarQube

Setting up the import of Azure DevOps repositories into SonarQube allows you to easily create SonarQube projects from your Azure DevOps repositories. If you're using [Developer Edition](https://www.sonarsource.com/plans-and-pricing/developer/) or above, this is also the first step in adding pull request decoration.

To set up the import of Azure DevOps repositories:

1.  Set your global DevOps platform settings
2.  Add a personal access token for importing repositories

### Setting your global settings

To import your Azure DevOps repositories into SonarQube, you need to first set your global SonarQube settings. Navigate to **Administration** > **Configuration** > **General Settings** > **DevOps Platform Integrations**, select the **Azure DevOps** tab, and click the **Create configuration** button. Specify the following settings:

-   **Configuration Name** (Enterprise and Data Center Edition only): The name used to identify your Azure DevOps configuration at the project level. Use something succinct and easily recognizable.
-   **Azure DevOps collection/organization URL**: If you are using Azure DevOps Server, provide your full Azure DevOps collection URL. For example, `https://ado.your-company.com/DefaultCollection`. If you are using Azure DevOps Services, provide your full Azure DevOps organization URL. For example, `https://dev.azure.com/your_organization`.
-   **Personal Access Token**: An Azure DevOps user account is used to decorate Pull Requests. We recommend using a dedicated Azure DevOps account with Administrator permissions. You need a [personal access token](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=tfs-2017&tabs=preview-page) from this account with the scope authorized for **Code** > **Read & Write** for the repositories that will be analyzed. Administrators can encrypt this token at **Administration** > **Configuration** > **Encryption**. See the **Settings Encryption** section of the [Security](https://docs.sonarqube.org/latest/instance-administration/security/) page for more information. This personal access token is used to report your quality gate status to your pull requests. You'll be asked for another personal access token for importing projects in the following section.

### Adding a personal access token for importing repositories

After setting your global settings, you can add a project from Azure DevOps by clicking the **Add project** button in the upper-right corner of the **Projects** homepage and selecting **Azure DevOps**.

Then, you'll be asked to provide a personal access token with `Code (Read & Write)` scope so SonarQube can access and list your Azure DevOps projects. This token will be stored in SonarQube and can be revoked at any time in Azure DevOps.

After saving your personal access token, you'll see a list of your Azure DevOps projects that can be **set up** and added to SonarQube. Setting up your projects this way also defines your project settings for pull request decoration.

For information on analyzing your projects with Azure Pipelines, see the **Analyzing projects with Azure Pipelines** section below.

## Analyzing projects with Azure Pipelines

The SonarQube extension running in Azure Pipelines jobs can automatically detect branches or pull requests being built, so you don't need to specifically pass them as parameters to the scanner.

### Installing your extension

From the Visual Studio Marketplace, install the [SonarQube extension](https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarqube) by clicking the **Get it free** button.

### Azure DevOps server - build agents

If you are using [Microsoft-hosted build agents](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops) then there is nothing else to install. The extension will work with all of the hosted agents (Windows, Linux, and macOS).

If you are self-hosting the build agents, make sure you have at least the minimum SonarQube-supported version of Java installed.

### Adding a new SonarQube service endpoint

After installing your extension, you need to declare your SonarQube server as a service endpoint in your Azure DevOps project settings:

1.  In Azure DevOps, go to **Project Settings** > **Service connections**.
2.  Select **New service connection** and then select **SonarQube** from the service connection list.
3.  Enter your SonarQube **Server URL**, an [authentication token](https://docs.sonarqube.org/latest/user-guide/user-account/generating-and-using-tokens/), and a memorable **Service connection name**. Then, select **Save** to save your connection.

### Configuring branch analysis

After adding your SonarQube service endpoint, you'll need to configure branch analysis. You'll use the following tasks in your build definitions to analyze your projects:

-   **Prepare analysis configuration**: This task configures the required settings before executing the build.
-   **Run code analysis** (Not used in Maven or Gradle projects): This task executes the analysis of source code.
-   **Publish quality gate result**: This task displays the quality gate status in the build summary letting you know if your code meets quality standards for production. This task may increase your build time as your pipeline has to wait for SonarQube to process the analysis report. It is highly recommended but optional.

Select your build technology below to expand the instructions for configuring branch analysis and to see an example `.yml` file.

### Running your pipeline

Commit and push your code to trigger the pipeline execution and SonarQube analysis. New pushes on your branches (and pull requests if you set up pull request analysis) trigger a new analysis in SonarQube.

### Maintaining pull request code quality and security

Using pull requests allows you to prevent unsafe or substandard code from being merged with your main branch. The following branch policies can help you maintain your code quality and safety by analyzing code and identifying issues in all of the pull requests on your project. These policies are optional, but they're highly recommended so you can quickly track, identify, and remediate issues in your code.

### Ensuring your pull requests are automatically analyzed

Ensure all of your pull requests get automatically analyzed by adding a [build validation branch policy](https://docs.microsoft.com/en-us/azure/devops/pipelines/repos/azure-repos-git#pr-triggers) on the target branch.

## Preventing pull request merges when the quality gate fails

Prevent the merge of pull requests with a failed quality gate by adding a `SonarQube/quality gate` [status check branch policy](https://docs.microsoft.com/en-us/azure/devops/repos/git/pr-status-policy) on the target branch.

Watch this [video](https://www.youtube.com/watch?v=be5aw9_7bBU) for a quick overview of how to prevent pull requests from being merged when they are failing the quality gate.

## Reporting your quality gate status in Azure DevOps

After you've set up SonarQube to import your Azure DevOps repositories as shown in the **Importing your Azure DevOps repositories into SonarQube** above, SonarQube can report your quality gate status and analysis metrics directly to your Azure DevOps pull requests.

To do this, add a project from Azure DevOps by clicking the **Add project** button in the upper-right corner of the **Projects** homepage and select **Azure DevOps** from the drop-down menu.

Then, follow the steps in SonarQube to analyze your project. SonarQube automatically sets the project settings required to show your quality gate in your pull requests.

If you're creating your projects manually or adding quality gate reporting to an existing project, see the following section.

### Reporting your quality gate status in manually created or existing projects

SonarQube can also report your quality gate status to Azure DevOps pull requests for existing and manually-created projects. After setting your global settings as shown in the **Importing your Azure DevOps repositories into SonarQube** section above, set the following project settings at **Project Settings** > **General Settings** > **DevOps Platform Integration**:

-   **Project name**
-   **Repository name**

### Advanced configuration

<details>
<summary> Reporting your quality gate status on pull requests in a mono repository</summary>
<p>

_Reporting quality gate statuses to pull requests in a mono repository setup is supported starting in_ [_Enterprise Edition_](https://www.sonarsource.com/plans-and-pricing/enterprise/)_._

In a mono repository setup, multiple SonarQube projects, each corresponding to a separate project within the mono repository, are all bound to the same Azure DevOps repository. You'll need to set up each SonarQube project that's part of a mono repository to report your quality gate status.

You need to set up projects that are part of a mono repository manually as shown in the **Reporting your quality gate status in manually created or existing project** section above. You also need to set the **Enable mono repository support** setting to true at **Project Settings** > **General Settings** > **DevOps Platform Integration**.

After setting your project settings, ensure the correct project is being analyzed by adjusting the Analysis Scope and pass your project names to the scanner. See the following sections for more information.

### Ensuring the correct project is analyzed

You need to adjust the analysis scope to make sure SonarQube doesn't analyze code from other projects in your mono repository. To do this set up a **Source File Inclusion** for your project at **Project Settings** > **Analysis Scope** with a pattern that will only include files from the appropriate folder. For example, adding `./MyFolderName/**/*` to your inclusions would only add code in the `MyFolderName` folder to your analysis. See [Narrowing the Focus](https://docs.sonarqube.org/latest/project-administration/narrowing-the-focus/) for more information on setting your analysis scope.

### Passing project names to the scanner

Because of the nature of a mono repository, SonarQube scanners might read all project names of your mono repository as identical. To avoid having multiple projects with the same name, you need to pass the `sonar.projectName` parameter to the scanner. For example, if you're using the Maven scanner, you would pass `mvn sonar:sonar -Dsonar.projectName=YourProjectName`.

</p>
</details>
</p>

<details>
<summary> Configuring multiple DevOps platform instances</summary>
<p>

SonarQube can report your quality gate status to multiple DevOps platform instances. To do this, you need to create a configuration for each DevOps Platform instance and assign that configuration to the appropriate projects.

-   As part of [Developer Edition](https://www.sonarsource.com/plans-and-pricing/developer/), you can create one configuration for each DevOps platform.
-   Starting in [Enterprise Edition](https://www.sonarsource.com/plans-and-pricing/enterprise/), you can create multiple configurations for each DevOps platform.

</p>
</details>
</p>

<details>
<summary> Linking issues</summary>
<p>

When adding a quality gate status to your pull requests, individual issues will be linked to their SonarQube counterparts automatically. For this to work correctly, go to **Administration** > **Configuration** > **General Settings** > **General** > **General** to set the instance's _Server base URL_. Otherwise, the links will default to `localhost`.

</p>
</details>
</p>

## FAQ

### Missing build agent capability

If you add a Windows Build Agent and install a non-oracle Java version on it, the agent will fail to detect a needed capability for the SonarQube Azure DevOps plugin. If you are sure that the `java` executable is available in the `PATH` environment variable, you can add the missing capability manually by going to **your build agent** > **capabilities** > **user capabilities** > **add capability**. Here, you can add the key, value pair java, and null which should allow the SonarQube plugin to be scheduled on that build agent. This Bug has been reported to the Microsoft Team with [azure-pipelines-agent#2046](https://github.com/microsoft/azure-pipelines-agent/issues/2046) but is currently not followed up upon.

### Interaction details between SonarQube and Azure

When you run a Sonar analysis for a pull request, each Sonar issue will be a comment on the Azure DevOps pull request. If the AzureDevOps instance is configured correctly and you set an issue in SonarQube to **resolved**, the _AzureDevOps Pull Request Comment_ will automatically be resolved. Likewise, when you fix an issue in the code and run the analysis build another time, the issue will be resolved in Sonar and deleted in AzureDevOps.