---
title: Unify multiple Azure Monitor Application Insights resources  | Microsoft Docs
description: This article provides details on how to use a function in Azure Monitor Logs to query multiple Application Insights resources and visualize that data. 
author: bwren
ms.author: bwren
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.reviewer: yossi-y
ms.date: 03/23/2022

---

# Unify multiple Azure Monitor Application Insights resources 
This article describes how to query and view all your Application Insights log data in one place, even when they are in different Azure subscriptions, as a replacement for the deprecation of the Application Insights Connector. The number of Application Insights resources that you can include in a single query is limited to 100.

## Recommended approach to query multiple Application Insights resources 
Listing multiple Application Insights resources in a query can be cumbersome and difficult to maintain. Instead, you can leverage function to separate the query logic from the applications scoping.  

This example demonstrates how you can monitor multiple Application Insights resources and visualize the count of failed requests by application name.

Create a function using union operator with the list of applications, then save the query in your workspace as function with the alias *applicationsScoping*. 

You can modify the listed applications at any time in the portal by navigating to Query explorer in your workspace and selecting the function for editing and then saving, or using the `SavedSearch` PowerShell cmdlet. 

>[!NOTE]
>This method can’t be used with log alerts because the access validation of the alert rule resources, including workspaces and applications, is performed at alert creation time. Adding new resources to the function after the alert creation isn’t supported. If you prefer to use function for resource scoping in log alerts, you need to edit the alert rule in the portal or with a Resource Manager template to update the scoped resources. Alternatively, you can include the list of resources in the log alert query.

The `withsource= SourceApp` command adds a column to the results that designates the application that sent the log. The parse operator is optional in this example and used to extract the application name from SourceApp property. 

```
union withsource=SourceApp 
app('Contoso-app1').requests,  
app('Contoso-app2').requests, 
app('Contoso-app3').requests, 
app('Contoso-app4').requests, 
app('Contoso-app5').requests 
| parse SourceApp with * "('" applicationName "')" *  
```

You are now ready to use applicationsScoping function in the cross-resource query:  

```
applicationsScoping 
| where timestamp > ago(12h)
| where success == 'False'
| parse SourceApp with * '(' applicationName ')' * 
| summarize count() by applicationName, bin(timestamp, 1h) 
| render timechart
```

The query uses Application Insights schema, although the query is executed in the workspace since the applicationsScoping function returns the Application Insights data structure. The function alias returns the union of the requests from all the defined applications. The query then filters for failed requests and visualizes the trends by application.

![Cross-query results example](media/unify-app-resource-data/app-insights-query-results.png)

>[!NOTE]
>[Cross-resource queries](../logs/cross-workspace-query.md) in log alerts are only supported in the current [scheduledQueryRules API](/rest/api/monitor/scheduledqueryrule-2018-04-16/scheduled-query-rules). If you're using the legacy Log Analytics Alerts API, you'll need to [switch to the current API](../alerts/alerts-log-api-switch.md). [See example templates](../alerts/alerts-log-create-templates.md).

## Application Insights and Log Analytics workspace schema differences
The following table shows the schema differences between Log Analytics and Application Insights.  

| Log Analytics workspace properties| Application Insights resource properties|
|------------|------------| 
| AnonUserId | user_id|
| ApplicationId | appId|
| ApplicationName | appName|
| ApplicationTypeVersion | application_Version |
| AvailabilityCount | itemCount |
| AvailabilityDuration | duration |
| AvailabilityMessage | message |
| AvailabilityRunLocation | location |
| AvailabilityTestId | id |
| AvailabilityTestName | name |
| AvailabilityTimestamp | timestamp |
| Browser | client_browser |
| City | client_city |
| ClientIP | client_IP |
| Computer | cloud_RoleInstance | 
| Country | client_CountryOrRegion | 
| CustomEventCount | itemCount | 
| CustomEventDimensions | customDimensions |
| CustomEventName | name | 
| DeviceModel | client_Model | 
| DeviceType | client_Type | 
| ExceptionCount | itemCount | 
| ExceptionHandledAt | handledAt |
| ExceptionMessage | message | 
| ExceptionType | type |
| OperationID | operation_id |
| OperationName | operation_Name | 
| OS | client_OS | 
| PageViewCount | itemCount |
| PageViewDuration | duration | 
| PageViewName | name | 
| ParentOperationID | operation_Id | 
| RequestCount | itemCount | 
| RequestDuration | duration | 
| RequestID | id | 
| RequestName | name | 
| RequestSuccess | success | 
| ResponseCode | resultCode | 
| Role | cloud_RoleName |
| RoleInstance | cloud_RoleInstance |
| SessionId | session_Id | 
| SourceSystem | operation_SyntheticSource |
| TelemetryTYpe | type |
| URL | url |
| UserAccountId | user_AccountId |

## Next steps

Use [Log Search](../logs/log-query-overview.md) to view detailed information for your Application Insights apps.

