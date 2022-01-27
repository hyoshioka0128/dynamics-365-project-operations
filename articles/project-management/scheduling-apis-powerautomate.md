---
title: Use Project schedule APIs with Power Automate
description: This topic provides a sample flow that uses the Project schedule application programming interfaces (APIs).
author: ruhercul
ms.date: 01/26/2022
ms.topic: article
ms.reviewer: kfend 
ms.author: ruhercul
---

# Use Project schedule APIs with Power Automate

_**Applies To:** Project Operations for resource/non-stocked based scenarios, Lite deployment - deal to proforma invoicing_

This topic describes a sample flow that shows how to create a complete project plan by using Microsoft Power Automate, how to create an Operation Set, and how to update an entity. The example demonstrates how to create a project, project team member, Operation Sets, project tasks, and resource assignments. This topic also explains how to update an entity and execute an Operation Set.

The following is a complete list of the steps that are documented in the sample flow in this topic:

1. [Create a PowerApps trigger](#1)
2. [Create a project](#2)
3. [Initialize a variable for the team member](#3)
4. [Create a generic team member](#4)
5. [Create an Operation Set](#5)
6. [Create a project bucket](#6)
7. [Initialize a variable for the link status](#7)
8. [Initialize a variable for the number of tasks](#8)
9. [Initialize a variable for the project task ID](#9)
10. [Do until](#10)
11. [Set a project task](#11)
12. [Create a project task](#12)
13. [Create a resource assignment](#13)
14. [Decrement a variable](#14)
15. [Rename a project task](#15)
16. [Run an Operation Set](#16)

## Assumptions

This topic assumes that you have a basic knowledge of the Dataverse platform, cloud flows, and the Project Schedule Application Programming Interface (API). For more information, see the [References](#references) section later in this topic.

## Create a flow

### Select an environment

You can create the Power Automate flow in your environment.

1. Go to <https://flow.microsoft.com>, and use your administrator credentials to sign in.
2. In the upper-right corner, select **Environments**.
3. In the list, select the environment where Dynamics 365 Project Operations is installed.

### Create a solution

Follow these steps to create a [solution-aware flow](/power-automate/overview-solution-flows). By creating a solution-aware flow, you can more easily export the flow to use it later.

1. In the navigation pane, select **Solutions**.
2. On the **Solutions** page, select **New solution**.
3. In the **New solution** dialog box, set the required fields, and then select **Create**.

## <a id="1"></a>Step 1: Create a PowerApps trigger

1. On the **Solutions** page, select the solution that you created, and then select **New**.
2. In the left pane, select **Cloud flows** \> **Automation** \> **Cloud flow** \> **Instant**.
3. In the **Flow name** field, enter **Schedule API Demo Flow**.
4. In the **Choose how to trigger this flow** list, select **Power Apps**. When you create a Power Apps trigger, the logic is up to you as the author. In this topic, leave the input parameters blank for testing purposes.
5. Select **Create**.

## <a id="2"></a>Step 2: Create a project

Follow these steps to create a sample project.

1. In the flow that you created, select **New step**.

    ![Adding a new step.](media/newstep.png)

2. In the **Choose an operation** dialog box, in the search field, enter **perform unbound action**. Then, on the **Actions** tab, select the operation in the list of results.

    ![Selecting an operation.](media/chooseactiontab.png)

3. In the new step, select the ellipsis (**...**), and then select **Rename**.

![Renaming a step.](media/renamestep.png)

4. Rename the step **Create Project**.
5. In the **Action Name** field, select **msdyn\_CreateProjectV1**.
6. Under the **msdyn\_subject** field, select **Add dynamic content**.
7. On the **Expression** tab, in the function field, enter **Project name - utcNow()**.
8. Select **OK**.

## <a id="3"></a>Step 3: Initialize a variable for the team member

1. In the flow, select **New step**.
2. In the **Choose an operation** dialog box, in the search field, enter **initialize variable**. Then, on the **Actions** tab, select the operation in the list of results.
3. In the new step, select the ellipsis (**...**), and then select **Rename**.
4. Rename the step **Init team member**.
5. In the **Name** field, enter **TeamMemberAction**.
6. In the **Type** field, select **String**.
7. In the **Value** field, enter **msdyn\_CreateTeamMemberV1**.

## <a id="4"></a>Step 4: Create a generic team member

1. In the flow, select **New step**.
2. In the **Choose an operation** dialog box, in the search field, enter **perform unbound action**. Then, on the **Actions** tab, select the operation in the list of results.
3. In the new step, select the ellipsis (**...**), and then select **Rename**.
4. Rename the step **Create Team Member**.
5. For the **Action Name** field, select **TeamMemberAction** in the **Dynamic content** dialog box.
6. In the **Action Parameters** field, enter the following parameter information.

    ```
    {
        "TeamMember": {
            "@@odata.type": "Microsoft.Dynamics.CRM.msdyn_projectteam",
            "msdyn_projectteamid": "@{guid()}",
            "msdyn_project@odata.bind": "/msdyn_projects(@{outputs('Create_Project')?['body/ProjectId']})",
            "msdyn_name": "ScheduleAPIDemoTM1"
        }
    } 
    ```

    Here is an explanation of the parameters:

    - **\@\@odata.type** – The entity name. For example, enter **"Microsoft.Dynamics.CRM.msdyn\_projectteam"**.
    - **msdyn\_projectteamid** – The primary key of the project team ID. The value is a globally unique identifier (GUID) expression.   The ID is generated from the expression tab.

    - **msdyn\_project\@odata.bind** – The project ID of the owning project. The value will be dynamic content that comes from the response of the "Create Project" step. Make sure that you enter the full path and add dynamic content between the parentheses. Quotation marks are required. For example, enter **"/msdyn\_projects(ADD DYNAMIC CONTENT)"**.
    - **msdyn\_name** – The name of the team member. For example, enter **"ScheduleAPIDemoTM1"**.

## <a id="5"></a>Step 5: Create an Operation Set

1. In the flow, select **New step**.
2. In the **Choose an operation** dialog box, in the search field, enter **perform unbound action**. Then, on the **Actions** tab, select the operation in the list of results.
3. In the new step, select the ellipsis (**...**), and then select **Rename**.
4. Rename the step **Create Operation Set**.
5. In the **Action Name** field, select the **msdyn\_CreateOperationSetV1** Dataverse custom action.
6. In the **Description** field, enter **ScheduleAPIDemoOperationSet**.
7. In the **Project** field, enter **/msdyn\_projects(**.
8. In the **Dynamic content** dialog box, select **msdyn\_CreateProjectV1Response ProjectId**.
9. In the **Project** field, enter **)**.

## <a id="6"></a>Step 6: Create a project bucket

1. In the flow, select **New step**.
2. In the **Choose an operation** dialog box, in the search field, enter **add new row**. Then, on the **Actions** tab, select the operation in the list of results.
3. In the new step, select the ellipsis (**...**), and then select **Rename**.
4. Rename the step **Create Bucket**.
5. In the **Table Name** field, select **Project Buckets**.
6. In the **Name** field, enter **ScheduleAPIDemoBucket1**.
7. For the **Project** field, select **msdyn\_CreateProjectV1Response ProjectId** in the **Dynamic content** dialog box.

## <a id="7"></a>Step 7: Initialize a variable for the link status

1. In the flow, select **New step**.
2. In the **Choose an operation** dialog box, in the search field, enter **initialize variable**. Then, on the **Actions** tab, select the operation in the list of results.
3. In the new step, select the ellipsis (**...**), and then select **Rename**.
4. Rename the step **Init linkstatus**.
5. In the **Name** field, enter **linkstatus**.
6. In the **Type** field, select **Integer**.
7. In the **Value** field, enter **192350000**.

## <a id="8"></a>Step 8: Initialize a variable for the number of tasks

1. In the flow, select **New step**.
2. In the **Choose an operation** dialog box, in the search field, enter **initialize variable**. Then, on the **Actions** tab, select the operation in the list of results.
3. In the new step, select the ellipsis (**...**), and then select **Rename**.
4. Rename the step **Init Number of tasks**.
5. In the **Name** field, enter **number of tasks**.
6. In the **Type** field, select **Integer**.
7. In the **Value** field, enter **5**.

## <a id="9"></a>Step 9: Initialize a variable for the project task ID

1. In the flow, select **New step**.
2. In the **Choose an operation** dialog box, in the search field, enter **initialize variable**. Then, on the **Actions** tab, select the operation in the list of results.
3. In the new step, select the ellipsis (**...**), and then select **Rename**.
4. Rename the step **Init ProjectTaskID**.
5. In the **Name** field, enter **number of tasks**.
6. In the **Type** field, select **String**.
7. For the **Value** field, enter **guid()** in the expression builder.

## <a id="10"></a>Step 10: Do until

1. In the flow, select **New step**.
2. In the **Choose an operation** dialog box, in the search field, enter **do until**. Then, on the **Actions** tab, select the operation in the list of results.
3. Set the first value in the conditional statement to the **number of tasks** variable from the **Dynamic content** dialog box.
4. Set the condition to **less than equal to**.
5. Set the second value in the conditional statement to **0**.

## <a id="11"></a>Step 11: Set a project task

1. In the flow, select **New step**.
2. In the **Choose an operation** dialog box, in the search field, enter **set variable**. Then, on the **Actions** tab, select the operation in the list of results.
3. In the new step, select the ellipsis (**...**), and then select **Rename**.
4. Rename the step **Set Project Task**.
5. In the **Name** field, select **msdyn\_projecttaskid**.
6. For the **Value** field, enter **guid()** in the expression builder.

## <a id="12"></a>Step 12: Create a project task

Follow these steps to create a project task that has a unique ID that belongs to the current project and the project bucket that you created.

1. In the flow, select **New step**.
2. In the **Choose an operation** dialog box, in the search field, enter **perform unbound action**. Then, on the **Actions** tab, select the operation in the list of results.
3. In the step, select the ellipsis (**...**), and then select **Rename**.
4. Rename the step **Create Project Task**.
5. In the **Action Name** field, select **msdyn\_PssCreateV1**.
6. In the **Entity** field, enter the following parameter information.

    ```
    {
        "@@odata.type": "Microsoft.Dynamics.CRM.msdyn_projecttask",
        "msdyn_projecttaskid": "@{variables('msdyn_projecttaskid')}",
        "msdyn_project@odata.bind": "/msdyn_projects(@{outputs('Create_Project')?['body/ProjectId']})",
        "msdyn_subject": "ScheduleAPIDemoTask1",
        "msdyn_projectbucket@odata.bind": "/msdyn_projectbuckets(@{outputs('Create_Project_Buckets')?['body/msdyn_projectbucketid']})",
        "msdyn_start": "@{addDays(utcNow(), 1)}",
        "msdyn_scheduledstart": "@{utcNow()}",
        "msdyn_scheduledend": "@{addDays(utcNow(), 5)}",
        "msdyn_LinkStatus": "192350000"
    }
    ```

    Here is an explanation of the parameters:

    - **\@\@odata.type** – The entity name. For example, enter **"Microsoft.Dynamics.CRM.msdyn\_projecttask"**.
    - **msdyn\_projecttaskid** – The unique ID of the task. The value should be set to a dynamic variable from **msdyn\_projecttaskid**.
    - **msdyn\_project\@odata.bind** – The project ID of the owning project. The value will be dynamic content that comes from the response of the "Create Project" step. Make sure that you enter the full path and add dynamic content between the parentheses. Quotation marks are required. For example, enter **"/msdyn\_projects(ADD DYNAMIC CONTENT)"**.
    - **msdyn\_subject** – Any task name.
    - **msdyn\_projectbucket\@odata.bind** – The project bucket that contains the tasks. The value will be dynamic content that comes from the response of the "Create Bucket" step. Make sure that you enter the full path and add dynamic content between the parentheses. Quotation marks are required. For example, enter **"/msdyn\_projectbuckets(ADD DYNAMIC CONTENT)"**.
    - **msdyn\_start** – Dynamic content for the start date. For example, tomorrow will be represented as **"addDays(utcNow(), 1)"**.
    - **msdyn\_scheduledstart** – The scheduled start date. For example, tomorrow will be represented as **"addDays(utcNow(), 1)"**.
    - **msdyn\_scheduleend** – The scheduled end date. Select a date in the future. For example, specify **"addDays(utcNow(), 5)"**.
    - **msdyn\_LinkStatus** – The link status. For example, enter **"192350000"**.

7. For the **OperationSetId** field, select **msdyn\_CreateOperationSetV1Response** in the **Dynamic content** dialog box.

## <a id="13"></a>Step 13: Create a resource assignment

1. In the flow, select **New step**.
2. In the **Choose an operation** dialog box, in the search field, enter **perform unbound action**. Then, on the **Actions** tab, select the operation in the list of results.
3. In the step, select the ellipsis (**...**), and then select **Rename**.
4. Rename the step **Create Assignment**.
5. In the **Action Name** field, select **msdyn\_PssCreateV1**.
6. In the **Entity** field, enter the following parameter information.

    ```
    {
        "@odata.type": "Microsoft.Dynamics.CRM.msdyn_resourceassignment",
        "msdyn_resourceassignmentid": "@{guid()}",
        "msdyn_name": "ScheduleAPIDemoAssign1",
        "msdyn_taskid@odata.bind": "/msdyn_projecttasks(@{variables('msdyn_projecttaskid')})",
        "msdyn_projectteamid@odata.bind": "/msdyn_projectteams(@{outputs('Create_Team_Member')?['body/TeamMemberId']})",
        "msdyn_projectid@odata.bind": "/msdyn_projects(@{outputs('Create_Project')?['body/ProjectId']})"
    }
    ```

7. For the **OperationSetId** field, select **msdyn\_CreateOperationSetV1Response** in the **Dynamic content** dialog box.

## <a id="14"></a>Step 14: Decrement a variable

1. In the flow, select **New step**.
2. In the **Choose an operation** dialog box, in the search field, enter **decrement variable**. Then, on the **Actions** tab, select the operation in the list of results.
3. In the **Name** field, select **number of tasks**.
4. In the **Value** field, enter **1**.

## <a id="15"></a>Step 15: Rename a project task

1. In the flow, select **New step**.
2. In the **Choose an operation** dialog box, in the search field, enter **perform unbound action**. Then, on the **Actions** tab, select the operation in the list of results.
3. In the step, select the ellipsis (**...**), and then select **Rename**.
4. Rename the step **Rename Project Task**.
5. In the **Action Name** field, select **msdyn\_PssUpdateV1**.
6. In the **Entity** field, enter the following parameter information.

    ```
    {
        "@@odata.type": "Microsoft.Dynamics.CRM.msdyn_projecttask",
        "msdyn_projecttaskid": "@{variables('msdyn_projecttaskid')}",
        "msdyn_subject": "ScheduleDemoTask1-UpdatedName"
    }
    ```

7. For the **OperationSetId** field, select **msdyn\_CreateOperationSetV1Response** in the **Dynamic content** dialog box.

## <a id="16"></a>Step 16: Run an Operation Set

1. In the flow, select **New step**.
2. In the **Choose an operation** dialog box, in the search field, enter **perform unbound action**. Then, on the **Actions** tab, select the operation in the list of results.
3. In the step, select the ellipsis (**...**), and then select **Rename**.
4. Rename the step **Execute Operation Set**.
5. In the **Action Name** field, select **msdyn\_ExecuteOperationSetV1**.
6. For the **OperationSetId** field, select **msdyn\_CreateOperationSetV1Response OperationSetId** in the **Dynamid content** dialog box.

## References

- [Overview of how to integrate flows with Dataverse - Power Automate](/power-automate/dataverse/overview?WT.mc_id=email)
- [Use Project schedule APIs to perform operations with Scheduling entities](schedule-api-preview.md)
- [Overview of the cloud flows - Power Automate](/power-automate/overview-cloud?WT.mc_id=email)
- [Overview of solution-aware flows - Power Automate](/power-automate/overview-solution-flows?WT.mc_id=email)