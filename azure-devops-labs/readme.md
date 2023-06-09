---
title: Driving continuous quality of your code with SonarCloud
layout: page
sidebar: vsts2
permalink: /labs/vstsextend/sonarcloud/
folder: /labs/vstsextend/sonarcloud/
---

<div class="rw-ui-container"></div>

## Overview

[SonarCloud](https://www.sonarsource.com/products/sonarcloud/){:target="\_blank"} is the Clean Code solution for your cloud-based development workflow featuring:

- 26 languages, including Java, JS, C#, C/C++, Objective-C, TypeScript, Python, ABAP, PLSQL, T-SQL, and more.
- Thousands of rules to track down hard-to-find bugs and quality issues thanks to powerful static code analyzers.
- Cloud CI Integrations, with Travis, Azure DevOps, BitBucket, AppVeyor and more.
- Deep code analysis, to explore all source files, whether in branches or pull requests, to reach a green Quality Gate and promote the build.
- Fast, automatic analysis of most languages - no configuration required.

<div class="bg-slap"><img src="./images/mslearn.png" class="img-icon-cloud" alt="MS teams" style="width: 48px; height: 48px;">Want additional learning? Check out the <a href="https://docs.microsoft.com/en-us/learn/modules/scan-for-vulnerabilities/" target="_blank"><b><u> Scan code for vulnerabilities in Azure Pipelines</u></b></a> module on Microsoft Learn.</div>

### What's covered in this lab

In this lab, you will learn how to integrate Azure DevOps Services with SonarCloud:

- Setup an Azure DevOps project and CI build to integrate with SonarCloud.
- Analyze SonarCloud reports.
- Integrate static analysis into the Azure DevOps pull request process.

### Prerequisites for the lab

1. You will need a Microsoft account.

1. Using this account, sign in to [**Azure DevOps Services**](https://azure.microsoft.com/en-us/products/devops/){:target="\_blank"}.

1. Create a new Azure DevOps project for this lab:
  
   Every project in Azure DevOps belongs to an organization. You will be placed into an automatically created default organization on sign in, the name of which is based on your user name (in our example, the user Claudia Sonarova has been given the organization **claudiasonarova**).
   
   Inside this organization, create a project called **SonarExamples**:
   
   > Unless you intend to sign up for a paid plan with SonarCloud (see below), make sure that you set your Azure DevOps project to be public. If you *do* intend to sign up for a paid plan, then you can use a private or public project. More details about [Pricing](https://docs.sonarcloud.io/managing-your-subscription/pricing/){:target="_blank"} is explained in the docs.

   ![Create project](images/azure-create-project.png)

   Import the **Sonar Scanning Examples repository** from GitHub at https://github.com/SonarSource/sonar-scanning-examples.git or use the HTTPS address of your own repo. The scanning examples repository contains sample projects for a number of build systems and languages including C# with MSBuild, Maven, and Gradle with Java.
   
   Go to **Your project** > **Repos**:

   ![Go to repos](images/azure-project-repos.png)

   Then, select **Import**:

   ![Import repo](images/azure_import_repo.png)

   See [this Microsoft documentation](https://docs.microsoft.com/en-us/azure/devops/repos/git/import-git-repository?view=azure-devops) for detailed instructions on importing a repository.

1. Install the SonarCloud Azure DevOps extension in your Azure DevOps account. The SonarCloud extension contains build tasks, build templates and a custom dashboard widget to help with the construction of your pipeline.

   Find the [SonarCloud extension](https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarcloud) on the Visual Studio Marketplace and click **Get it free** to install it, then select **Proceed to organization** after the installation has finished.

   ![Marketplace](images/sonar-vs-marketplace.png)

   ![Marketplace installed](images/sonar-extension-installed.png)

   > If you do not have the appropriate permissions to install an extension from the marketplace, a request will be sent to the account administrator to ask them to approve the installation.

1. Using the same account as you used for Azure DevOps, sign into SonarCloud: [https://sonarcloud.io/login](https://sonarcloud.io/login)

   ![SonarCloud Login](images/sonarcloud-login.png)

1. In SonarCloud, create an organization and, within that, a new project. The organization and project you set up in SonarCloud will mirror the organization and project that you set up in Azure DevOps.

   Once you sign in, select **Import an organization from Azure** on the Welcome to SonarCloud page.

   ![SonarCloud Welcome](images/import-organization-from-azure.png)

   Follow the SonarCloud in-product tutorial to create an organization. First, add your **Azure DevOps organization name** (dev.azure.com/{YOUR-ORG}) in SonarCloud. Next in Azure, go to **Azure DevOps** > **User settings** > **Security** > **Personal access tokens** to create a new Personal Access Token; the SonarCloud in-product tutorial provides a link to Azure's User settings so if you are doing this in parallel, creating a PAT will be easy. See the Microsoft documentation to [Create a PAT](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page) if more details are needed.

   ![PAT](images/azure-create-personal-access-token.png)

   In Azure, define the token's **Name** and select the correct **Organization** (if you have multiple) from the list. Then, navigate to **Scopes** >  **Code** and select **Read & Write** before you **Create** the new token.

   <img src="images/azure-scope-of-personal-access-token.png"  width="300" alt="PAT">
   
   Copy your PAT from Azure and paste it into SonarCloud before continuing with the SonarCloud in-product tutorial.

    ![SonarCloud organization access token](images/create-organization.png)
   
   Define an **Organization Key**. A key will be automatically formatted for you if you define the **Name** above. Note that the name/key must be unique within the SonarCloud system or you cannot continue. You have an opportunity to add additional info if desired.

   Select **Continue** to move to the final step.

   ![SonarCloud org creation](images/create-organization2.png)

   Lastly, choose your plan. For this example, we selected a free plan (that is, one for public repos only), but you can choose a paid plan if you intend to have private repos:

   ![Choose SonarCloud plan](images/create-organization3.png)

   You have now created the SonarCloud organization that will mirror your Azure DevOps organization.

   Now, from within your new organization, create a SonarCloud project that will mirror the Azure DevOps project *SonarExamples* we imported earlier. Select **Analyze a new project** to continue.

   ![Create SonarCloud project](images/analyze-new-project.png)

   Choose your Azure DevOps project(s) to add and select **Set Up** to create a project on SonarCloud.

   ![Choose SonarCloud project](images/sonar-choose-project.png)

   Let's follow the SonarCloud in-product tutorial to set up the scanning in Azure Pipelines. Select **With Azure DevOps Pipelines** to get started: 

   ![Scan using Azure Pipelines](images/choose-analysis-method.png)

   - You can skip **Install our extension** if done previously. 
   - Next, **Add a new SonarCloud Service Endpoint** to your Azure project. Go to **Project settings** > **Service connections** and use the token provided by the in-product tutorial.
   - With your Service Endpoint verified, move on to **Configure Azure Pipeline** and select the option that best describes your build: 
   
      ![Analyze Azure project tutorial](images/analyze-project-tutorial.png)
         
      We will use a **.NET** project for Exercise 1 to configure the Azure DevOps Pipeline.

## Exercise 1: Set up a pipeline that integrates with SonarCloud

We will set up a new build pipeline that integrates with SonarCloud to analyze the **SonarExamples** project. 

Azure DevOps offers two ways to create pipeline: using a .YAML file or with the classic editor. Both methods begin the same way, by creating a new pipeline.

In your new Azure DevOps project, go to the **Pipelines** > **Pipelines** tab and select **Create Pipeline**:

   ![Create pipeline](images/1/azure-create-pipeline.png)

Here you have two options: you can configure the pipeline with either the **YAML** editor or choose to use the **Classic editor** to create a pipeline without YAML.

With the classic editor, you can take advantage of the pre-defined templates that were installed as part of the SonarCloud Extension above. With the YAML editor, you can use a separately provided YAML template file. We will cover both methods in this exercise, starting with the **YAML** editor.

### YAML Editor

1. On the **Where is your code?** page:: Because we have already cloned our repo to the SonarExamples project in Azure, we will select **Azure Repos Git**. You may have a different configuration in your project.

   ![Select Azure Repos Git](images/1/azure-yaml-select-azure-repos-git.png)

1. On the next screen select the git repository that you imported earlier, our repo is called **SonarExamples**:

   ![Select SonarExamples](images/1/azure-yaml-select-your-repository.png)

1. Now select a YAML file template. We will be building and analyzing the .NET code in our example imported repository, so we will start by choosing the **.NET Desktop** YAML template:

   ![Choose .NET Desktop template](images/1/azure-yaml-configure-your-pipeline-dot-net.png)

1. The YAML editor will open with the template YAML file. In order to configure it correctly, you will need to adjust it (or replace it) so that it looks like the following example file:  [net-desktop-sonarcloud.yml](https://github.com/SonarSource/sonar-scanner-vsts/blob/master/yaml-pipeline-templates/net-desktop-sonarcloud.yml)

   We must customize it to fit our needs:

   1. **Modify the Build task**: change the pointer to the solution. Note that we: 
      - added a `solution` variable in this example to improve the template, 
      - we updated the `NuGetToolInstaller`, and 
      - we used wildcards because our `.sln` file is not located in the root of the repo:

      ![](images/1/yaml-template-update.png)

   1. **Modify the `SonarcloudPrepare` task with your information**: Follow in-product tutorial from Sonarcloud to add the required steps in your pipeline. The steps will be different depending on your build option and because we are using .NET for this tutorial, we have these items to configure:

      ![SonarCloud Tutorial](images/1/sonar-in-product-tutorial.png)
   
      1. **Set your Fetch Depth**: See SonarCloud for the code sample to copy/paste.
      
      1. **Prepare Analysis Configuration**: To continue, type `sonarcloud` in the Azure task search bar and select **Prepare Analysis Configuration**. Then, using the values provided by the SonarCloud in-product tutorial, complete the following steps:
      
         1. Select the SonarCloud endpoint you created a few minutes ago.
         2. Copy/paste your **Organization**.
         3. Depending on your choosen build option, SonarCloud will recommend the correct way to run the analysis.
         4. Copy/paste the **Project Key**.
         5. Copy/paste the **Project Name**.

         Note in the following image how Azure DevOps will format the task parameters for you.

         ![Prepare the analysis configuration](images/1/azure-prepare-analysis-configuration.png)


   When you are done making the changes to the YAML file, edit the commit message and select **Save and run**:

   ![Save and run YAML](images/1/azure-review-your-pipeline.png)



### Classic Editor (skip this method if you chose the YAML option)

Use of the classic editor is still supported by Azure DevOps and therefore still supported by SonarCloud. 

To continue with this alternate part of **Exercise 1**, you should have already started the setup process in SonarCloud so that your service endpoint is created, and installed the [SonarCloud extension](https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarcloud) in Azure DevOps.

1. To configure the pipeline using the classic editor, select **Use the classic editor** on the **Where is your code?** page:

   ![Use classic editor](images/1/azure-classic-select-classic-editor.png)

1. Select your **source**, **Repository** and **Default branch** for builds. The selection needed for the steps followed in this tutorial are highlighted. Then click **Continue**.

   ![Select your source](images/1/azure-classic-select-source.png)

   > The [SonarCloud extension](https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarcloud) installed earlier provides SonarCloud-enabled custom build templates for Maven, Gradle, .NET Core and .NET Desktop applications. The templates are based on the standard Azure DevOps templates and have additional analysis-specific tasks as well as some pre-configured settings to make the process easierto configure.

1. Select the **.NET Desktop with SonarCloud** template and click **Apply**.

   ![.NET Desktop with SonarCloud template](images/1/azure-classic-select-template.png)

   The template contains all of the necessary tasks and most of the required settings. However, You will need to provide a few additional settings.

1. The pipeline editor will open and you must now define your tasks. First, select your **Agent pool** and **Agent Specification**; we used _Azure Pipelines_ for the Agent pool and, for our project, _windows-latest_ as the Agent Specification.

   ![Agent pool](images/1/azure-classic-agent-specs.png)

1. Go to **Get sources** in Azure and disable your **Fetch Depth**. The SonarCloud tutorial recommends to set the Fetch depth to `0`, but this is not an option when using the Classic editor. When the fetch depth is ≠ 0, some features, such as automatic assignment of issues, may be missing or broken. Disabling shallow clone is recommended for improving the relevancy of reports, but it is not required to run an analysis.

   ![Set fetch depth](images/1/azure-fetch-depth.png)

1. Next, we must add the analysis configuration values given by the SonarCloud in-product tutorial to the **Prepare analysis on SonarCloud** step in the pipeline.
   
   1. If you haven't already defined your **SonarCloud Service Endpoint**, do it now; it was the last step we took in SonarCloud before starting [Exercise 1](#exercise-1-set-up-a-pipeline-that-integrates-with-sonarcloud). To create a new service endpoint, click the **New** button then add your **SonarCloud Token** and **Verify** the connection. If verification works, give your token a name then select **Verify and save** to define it in the task.

      ![Create new Service Endpoint](images/1/azure-classic-create-endpoint.png) 

      The SonarCloud in-product tutorial will generate an endpoint token for you during project setup. This token identifies your account on that system and allows other services, in this case, Azure DevOps, to connect to that account. You can create and revoke tokens at any time by going to **My Account** > **Security** in [SonarCloud](https://sonarcloud.io/account/security). For this tutorial, we used the token provided by the in-product tutorial.

      ![Token for service endpoint](images/1/analyze-project-tutorial-2.png)

   1. If the token was correctly copied, Azure DevOps should connect with your SonarCloud account and you should be able to click on the **Organization** drop-down to select the organization you created earlier; choose it from the dropdown (in our case `claudiasonarova-azdo-org`).

   1. SonarCloud's in-product tutorial gives you the next values you must define: **Choose the way to run the analysis**, copy/paste your **Project Key**, and copy/paste your **Project Name** (items 3 & 4 in the image).

      ![New service connection](images/1/azure-classic-analysis-configuration.png)

   1. With all of the project settings defined in Azure, select **Save and queue**, add a commit message, and select **Save and run** to test your pipeline.

      <img src="images/1/azure-classic-save-and-run.png"  width="300" alt="Save and run">

   If you set up everything correctly, the pipeline will run and your SonarCloud in-product tutorial page will refresh and present your project's first analysis results!

   ![First analysis](images/1/azure-classic-first-analysis.png)

1. [Optional] Enable the **Publish Quality Gate Result** step in your pipeline.

   This step is added by default via the **.NET Desktop with SonarCloud** template, and is recommended to use SonarSource's Clean as You Code best practices. Enabling this feature will delay the completion of the build until the processing on SonarCloud has finished. However, it can be removed if desired.

   ![Publish your Quality Gate](images/1/azure-classic-publish-quality-gate.png)

1. **Save and queue** the build to update the pipeline after making changes. Once the build has started, you should see something like this:

   ![Recently run pipeline](images/1/azure-manually-run.png)

1. If you did not change the default **Enabled** value in the _Publish Quality Gate Result_ step of your pipeline, the **Build Summary** > **Extensions** tab in Azure will contain a summary of the analysis report. See step 11 about setting up your Sonar Quality Gate.

    ![SonarCloud analysis report](images/1/azure-classic-quality-gate-status.png)

1. Either select the **Detailed SonarCloud report >** link in the Azure build summary, or browse to SonarCloud and open the project to view the analysis results in SonarCloud.


1. To be able to see the Sonar Quality Gate result in your Azure Build summary, you must:
   - Run your first analysis to connect your Azure DevOps pipeline to your SonarCloud project. 
   - Define your [New Code Period](https://docs.sonarcloud.io/improving/new-code-definition/), and
   - Issue your first PR to check the new code against your [Quality Gate](https://docs.sonarcloud.io/improving/quality-gates/).

We have now created a new organization on SonarCloud and configured our Azure DevOps build to perform an analysis and push the results of the build to SonarCloud.

![SonarCloud report](images/1/sonar-full-analysis.png)

In Exercise 2, we will look at what to do with those reports in SonarCloud.

## Exercise 2: Analyze SonarCloud Reports

Open the **SonarExamples** project in the SonarCloud Dashboard. Under **_Bugs and Vulnerabilities_**, we can see a bug has been found.

![Bug found](images/sc_bug_found.png)

The page has other metrics such as **_Code Smells_**, **_Coverage_**, **_Duplications_** and **_Size_**. The following table briefly explains each of these terms.

| Terms | Description |
| --- | --- |
| **Bugs** | An issue that represents something wrong in the code. If this has not broken yet, it will, and probably at the worst possible moment. This needs to be fixed |
| **Vulnerabilities** | A security-related issue which represents a potential backdoor for attackers |
| **Code Smells** | A maintainability-related issue in the code. Leaving it as-is means that at best maintainers will have a harder time than they should make changes to the code. At worst, they'll be so confused by the state of the code that they'll introduce additional errors as they make changes |
| **Coverage** | To determine what proportion of your project's code is actually being tested by tests such as unit tests, code coverage is used. To guard effectively against bugs, these tests should exercise or 'cover' a large proportion of your code |
| **Duplications** | The duplications decoration shows which parts of the source code are duplicated |
| **Size** | Provides the count of lines of code within the project including the number of statements, Functions, Classes, Files and Directories |

{% include important.html content= "Notice that a **C** is displayed alongside the bug count. This is the **Reliability Rating**. **C** indicates that there is **at least 1 major bug** in this code. For more information on Reliability Rating, click [here](https://docs.sonarqube.org/display/SONAR/Metric+Definitions#MetricDefinitions-Reliability). For more information on rule types see [here](https://docs.sonarqube.org/latest/user-guide/rules/) and for more information on severities, see [here](https://docs.sonarqube.org/latest/user-guide/issues/)." %}

1. Click on the **Bugs** count to see the details of the bug. The Issues page will appear:

   ![Click on bugs](images/sc_issues.png)

2. Click on the bug to navigate to the code. You will see the error in line number 9 of **Program.cs** file as **Change this condition so that it does not always evaluate to 'true'; some subsequent code is never executed.**:

   ![Bug details](images/sc_bug_in_code.png)

3. We can also see which lines of code are not covered by tests:

   ![Test coverage](images/sc_bug_in_code_coverage.png)

   Our sample project is very small and has no historical data. However, there are thousands of [public projects on SonarCloud](https://sonarcloud.io/explore/projects){:target="\_blank"} that have more interesting and realistic results.



## Exercise 3: Set up pull request integration

Configuring SonarCloud analysis to run when a pull request is created involves two steps:

- A SonarCloud project needs to be provided with an access token so it can add PR comments to Azure DevOps, and
- Branch Policy needs to be configured in Azure DevOps to trigger the PR build

1. Create a **Personal Access Token** in Azure DevOps.

   - Follow the instructions in this [article](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops) to create a token with **Code (read and write)** scope.

     > SonarCloud will post comments to the pull request as if it is a user who owns the personal access token. The recommended practice is to create a separate "bot" Azure DevOps user for this so that it is clear which comments are from real developers and which are from SonarCloud.

   Make sure to set the scope under **Code** to **Read & write**:

   ![Personal access token permissions](images/azdo_create_pat.png)

   Click **Create** and copy the generated token:

   ![Copy personal access token](images/azdo_copy_pat.png)

   > You should treat Personal Access Tokens like passwords. It is recommended that you save them somewhere safe so that you can re-use them for future requests.

2. Configure SonarCloud to analyze pull requests

   - Browse to the **Sonar Examples** dashboard in SonarCloud
   - Click on **Administration** > **General Settings**

     ![Click general settings](images/sc_click_general_settings1.png)

   - Select the **Pull Requests** tab
   - Set the **Provider** drop-down to **Azure DevOps Services** and click **Save**
   - Paste in the **Personal access token** you copied earlier and click **Save**

     ![Pull request configuration](images/sc_pull_request_config.png)

3. Configure the branch policy for the project in Azure DevOps

   - Navigate to the **SonarExamples** project in Azure DevOps
   - Click on **Repos**, **Branches** to view the list of branches
   - Click on the settings link ("**...**") for **master** and select **Branch policies**

     ![Click branch policies](images/azdo_click_branch_policies.png)

   - Click the **+** beside **Build Validation** to add a new build policy:

     ![Click add build policy](images/azdo_click_add_build_policy.png)

   - Select the build definition we created earlier from the **Build definition** drop-down
   - Set the **Display name** to **SonarCloud analysis**
   - Click **Save**

     ![Add build policy](images/azdo_add_build_policy.png)

   Azure DevOps is now configured to trigger a SonarCloud analysis when any pull request targeting the **master** branch is created.

4. Create a new pull request

   Now we will make a change to a file and create a new request so that we can check that the pull request triggers an analysis.

   - Navigate to the code file **Program.cs** at **sonarqube-scanner-msbuild/CSharpProject/SomeConsoleApplication/Program.cs** and click **Edit**
   - Add an empty method to the code as shown in the following screen shot, then click **Commit...**

   ``public void Unused(){}``

      ![Edit program](images/azdo_add_unused.png)

   In the dialog that appears,

   - Change the branch name from **master** to **branch1**
   - Check the **Create a pull request** checkbox
   - Click **Commit**, then click **Create** on the next screen to submit the pull request

      ![Commit changes](images/azdo_commit_pr.png)

      ![Create pull request](images/azdo_create_pr.png)

      If the pull request integration is correctly configured the UI will show that an analysis build is in progress.

     ![Analysis in progress](images/azdo_pr_analysis_in_progress.png)

5. Review the results of the Pull Request analysis

   The results show that the analysis builds completed successfully, but that the new code in the PR failed the Code Quality check.
   Comment has been posted to the PR for the new issue that was discovered.

   ![Check failed](images/check_failed.png)

   ![Check failed](images/azdo_pr_comment.png)

   Note that the only issues in code that was changed or added in the pull request are reported - pre-existing issues in **Program.cs** and other files are ignored.

6. Block pull requests if the Code Quality check failed

   At this point, it is still possible to complete the pull request and commit the changes even though the Code Quality check has failed.
   However, it is simple to configure Azure DevOps to block the commit unless the Code Quality check passes:

   - Go to the **Branch Policy** page of the **master** branch (since the master branch is the one you will want your pull requests to merge into, this is where you have to adjust the policy).
   - Click **Add status policy**

      ![Return to branch policies](images/azdo_add_status_check.png)

   - Select **SonarCloud/quality gate** from the **Status to check** drop-down
   - Set the **Policy requirement** to **Required**
   - Click **Save**

     ![vsts_status_policy_add](images/azdo_add_status_policy.png)

   Users will now be unable to merge the pull request until the Code Quality check is successful (the Code Quality check succeeds when all issues have been either fixed or marked as **confirmed** or **resolved** in SonarCloud).

## Exercise 4: Check the SonarCloud Quality Gate status in a Continuous Deployment scenario (In Preview)

**Disclaimer: This feature is in preview, and may not reflect its final version. Please look at the notes at the end of this exercise for more information.**

Starting from version 1.8.0 of the SonarCloud extension for Azure DevOps, a pre-deployment gate is available for your release pipeline. It allows you to check the status of the SonarCloud Quality Gate for the artifact you want to deploy and block the deployment if the Quality Gate failed.

Prerequisites :

- Enable the **Publish Quality Gate Result** in your build pipeline
- Have an artifact built and published in this pipeline to feed the release pipeline

Setup :

1. Click on **Pipelines** on the left pane, then click on **Releases**
1. Click on **New Pipeline**

   ![new_release_pipeline](images/ex4/new_release_pipeline.png)

1. On the template selection, choose the template you want. We will choose **Empty job** for this exercise.

   ![empty_job](images/ex4/empty_job.png)

1. Close for now the Stage properties
1. Click on the pre-deployment conditions on Stage 1

   ![pre_deploy_conditions](images/ex4/predeploy_conditions.PNG)

1. Click on **Enabled** beside **Gates**
1. Click on **+ Add**, then select **SonarCloud Quality Gate status check**

   ![pre_deploy_conditions_settings](images/ex4/predeploy_conditions_settings.PNG)

1. In order to have the fastest result possible for this exercise, we recommend you to set the evaluations options as per this screenshot (See [how Gate evaluation flow works](https://docs.microsoft.com/en-us/azure/devops/pipelines/release/approvals/gates?view=azure-devops), as per Microsoft Documentation)

   ![gate_setting](images/ex4/gate_setting.PNG)

1.  That's it. You can close this panel.
1. Click now on **Add an artifact** on the left.
1. Currently, only build artifacts are supported. Choose the project and the source (build pipeline) of your artifact, its alias should match the name of the artifact published in the build pipeline.
1. Click on **Add**

    ![artifact_setting](images/ex4/artifact_settings.PNG)

1. Setup the CD trigger.

   ![artifact_setting](images/cd.PNG)

1. You can now save your pipeline.
1. Go back to the build pipeline section, trigger a build of the pipeline that creates the artifact.
1. Once the build is completed and succeeded, it will trigger the CD automatically.

   
1. Go to the  **Releases** page.
1. After few minutes (as set up on the point 8 of this exercise), your Quality Gate check should have been performed (at least twice to get a 'go/nogo' for the stage), and if it's green, it should look like this:

   ![qg_green](images/ex4/qg_green.PNG)

Otherwise, if it's failed, then read important notes below to find out how it happened and how to get a green Quality Gate.

**Important notes about this feature**

- The **Publish Quality Gate Result** task in your build pipeline has to be enabled in order for the release gate to work.
- If the Quality Gate is in the failed state, it will not be possible to get the pre-deployment gate passing as this status will remain in its initial state. You will have to execute another build with either the current issues corrected in SonarCloud or with another commit for fixing them.
- Please note also that current behavior of the pre-deployment gates in Release Pipelines is to check the status every 5 minutes, for a duration of 1 day by default. However, if a Quality Gate for a build has failed it will remain failed so there is no point in re-checking the status. Knowing this, you can set the timeout after which gates fail to a maximum of 6 minutes so the gate will be evaluated only twice as described above, or just cancel the release itself.
- Only the primary build artifact related Quality Gate of the release will be checked.
- During a build, if multiple analyses are performed, all of the related Quality Gates are checked. If one of them has the status either WARN, ERROR or NONE, then the Quality Gate status on the Release Pipeline will be failed.

## Summary

With the **SonarCloud** extension for **Azure DevOps Services**, you can embed automated testing in your CI/CD pipeline to automate the measurement of your technical debt including code semantics, testing coverage, vulnerabilities. etc. You can also integrate the analysis into the Azure DevOps pull request process so that issues are discovered before they are merged.
