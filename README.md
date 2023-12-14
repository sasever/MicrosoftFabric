 # Microsoft Fabric

 <img src="media/image/fabric.jpeg" alt="Microsoft Fabric Logo" width="10%"/>
 
 *Complementary guidance for onboarding an enterprise to [Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/microsoft-fabric-overview "MS Documentation for Microsoft Fabric")*

 Microsoft Fabric is an all-in-one analytics solution for enterprises that covers everything from data movement to data science, Real-Time Analytics, and business intelligence. It offers a comprehensive suite of services, including data lake, data engineering, and data integration, all in one place.

<img src="https://learn.microsoft.com/fabric/get-started/media/microsoft-fabric-overview/saas-foundation.png" alt="Microsoft Fabric SAAS foundation" width="90%"/>


When onboarding an Enterprise with existing Data Infrastructures to a new SAAS platform like Fabric there are important topics that needs to be discussed like,

1. How the integration with existing on premises and cloud infrastructure should look like?
2. How will be the Access Model?
3. What should be the CI/CD approach?
4. What should be the orchestration approach that combines the dependencies between all different new and existing data platforms
5. What will be the adoption process and roadmap https://learn.microsoft.com/power-bi/guidance/fabric-adoption-roadmap

In this repository we are aiming to provide some insights on how these decisions can be tackled.
the information shared in the repository bases on the state of Microsoft Fabric GA features and [publicly announced roadmap](https://learn.microsoft.com/fabric/release-plan/) as of 2023 December.

## Integration with existing on premises and cloud infrastructure 
 The evaluation for the needs for  Cloud infrastructure and on premises environment integration needs to be discussed primarily. Even though the enterprise may want to adopt Microsoft Fabric Without any existing Azure footprint, this is rarely a reality. Any existing Azure footprint that will feed and consume data to/from Microsoft Fabric should be evaluated from security(identity, networking) and governance perspective. Therefore an evaluation on [CAF (Cloud adoption framework) focusing on Enterprise Landing Zones](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/landing-zone/) should be performed as a preliminary action, and necessary policy driven governance and network configuration actions should be taken. For FSI industry and similar industries with strong regulations the [FSI Landing Zones Principles](https://github.com/microsoft/industry/blob/main/fsi/referenceImplementation/readme.md) should be followed.

 Below you can find an example Architecture positioning diagram which covers how to integrate on premises and existing Azure Landscape with Microsoft Fabric.

 ![Overall Integration Architecture Diagram](media/diagram/OverallDiagramForOnpremAzurewithRoadmap.png)
 
# Access Model

Microsoft Fabric is purchased and enabled by capacity. Within a capacity you have one unified [OneLake](https://learn.microsoft.com/fabric/onelake/onelake-overview), and one to many workspaces. [Lakehouses](https://learn.microsoft.com/fabric/data-engineering/lakehouse-overview) in respective workspaces keeps the data in this single OneLake. you can isolate access and therefore environments within the same capacity by different workspaces.

 <img src="media/diagram/WorkspaceArchitecture.png" alt="Microsoft Fabric OneLake Architecture" width="50%"/>

The access model for variaous workspaces should be designed based on below access right grant and segregation possibilities:

![Alt text](media/diagram/AccessModel.png)

* RW type WS roles are valid within their allowed activities, in every artifact covered by the specific workspace. 
   For more information refer to the [workspace roles section in documentation](https://learn.microsoft.com/fabric/get-started/roles-workspaces#-workspace-roles)

** Viewer WS role only covers Lakehouse tables and Warehouse tables/views , via SQL endpoint-TSQL interface, to have access to files the user/group needs to have either item level READ /READALL permission or a higher RW WS role.

***  users/groups mentioned in any part of the document are EntraID users/groups

**** 1. [ddm (dynamic data masking)](https://learn.microsoft.com/fabric/data-warehouse/dynamic-data-masking) applies only to the users/groups without Administrator, Member, or Contributor rights on the WS, and without elevated permissions on the Warehouse.


**** 2 . [Row level security](https://learn.microsoft.com/fabric/data-warehouse/row-level-security) only applies to Lakehouse SQL and Warehouse SQL endpoints, users/groups having elevated permissions on Lakehouse files will have access to all data. Row level security is applied with regular SQL RLS methods such as security policies and functions/predicates


**** 3 . [column-level security](https://learn.microsoft.com/fabric/data-warehouse/column-level-security) is applied  with the GRANT T-SQL statement for users/groups 

 # DevOps CI/CD approach

 Below diagram tries to address a possible CI/CD approach with how the lakehouse should be positioned within various environment types. Here it is suggested to use one git repository per environment vertical, like one repository for all dev, one repository for all test and one repository for prod. This approach bases on current available capabilities on Microsoft Fabric as of December 2023. the limitations and requirements taken into consideration are:

Requirements:
 1. The ability to have feature branches/workspaces on Dev environments
 2. The ability to have a Hotfix workspace for production
 3. Being able to back propagate hotfixes to respective prior stage environments to prevent those getting overwritten with future deployments
 4. Being able to have a backup on git for every stage.
 
 Limitations: 
 1. Fabric natively has only one tool to do promotion within environment stages called [Fabric Deployment Pipelines](https://learn.microsoft.com/fabric/cicd/deployment-pipelines/intro-to-deployment-pipelines).
 2. One workspace can only be referenced in one Fabric Deployment Pipeline
 3. One workspace can only be in sync with one git branch, there is no seemless switching between branches. When you switch you lose the artifacts that are not currently supported for git versioning.
 4. The syncing between the Workspace and and synced  git branch is not seemles and fully bidirectional. you neeed to commit the artifacts to be synced with git from workspace. if you externally commit an artifact to git, to be able to use it from workspace you need to import those via UI. There is selective commit from workspace to git but no selective import from git to workspace.

![CICD diagram for different stages of environments](media/diagram/CICD.png)