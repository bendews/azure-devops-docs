---
title: Pipeline failed test sample Power BI reports 
titleSuffix: Azure DevOps
description: Learn how to list failed tests in a Power BI report for a given pipeline in the project.
ms.subservice: azure-devops-analytics
ms.reviewer: desalg
ms.author: kaelli
ms.custom: powerbisample
author: KathrynEE
ms.topic: sample
monikerRange: '>= azure-devops-2020'  
ms.date: 10/13/2021
---

# Failed tests sample report


[!INCLUDE [version-gt-eq-2020](../../includes/version-gt-eq-2020.md)] 
 
You can create a report that lists failed tests, similar to the following image, for pipeline runs that include test tasks. For information on adding tests to a pipeline, see the [Test task resources](#test-task-resources) section later in this article. 
 

:::image type="content" source="media/pipeline-test-reports/failed-tests-table-report.png" alt-text="Screenshot of Failed Tests Table report.":::

Use the queries provided in this article to generate the following reports:

- Failed tests for build workflow
- Failed tests for release workflow
- Failed tests for a particular branch
- Failed tests for a particular test file
- Failed tests for a particular test owner 

[!INCLUDE [temp](includes/preview-note.md)]

[!INCLUDE [prerequisites-simple](../includes/analytics-prerequisites-simple.md)]

[!INCLUDE [temp](includes/sample-required-reading.md)]


## Sample queries

You can use the following queries of the `TestResultsDaily` entity set to create different but similar pipeline failed test reports. The `TestResultsDaily` entity set provides a daily snapshot aggregate of `TestResult` executions, grouped by Test.  

[!INCLUDE [temp](includes/query-filters-test-pipelines.md)]


### Failed tests for a Build workflow  

Use the following queries to view the failed tests for a **Build** workflow pipeline.

#### [Power BI query](#tab/powerbi/)

[!INCLUDE [temp](includes/sample-powerbi-query.md)]

```
let
   Source = OData.Feed ("https://analytics.dev.azure.com/{organization}/{project}/_odata/v4.0-preview/TestResultsDaily?
$apply=filter("
                &"Pipeline/PipelineName eq '{pipelineName}' "
                &"And Date/Date ge {startdate} "
        &"And Workflow eq 'Build' "
        &") "
            &"/groupby( "
                &"(TestSK, Test/TestName), "
                &"aggregate( "
            &"ResultCount with sum as TotalCount, "
                &"ResultPassCount with sum as PassedCount, "
            &"ResultFailCount with sum as FailedCount, "
        &"ResultNotExecutedCount with sum as NotExecutedCount, "
    &"ResultNotImpactedCount with sum as NotImpactedCount, "
    &"ResultFlakyCount with sum as FlakyCount)) "
    &"/filter(FailedCount gt 0) "
    &"/compute( "
    &"iif(TotalCount gt NotExecutedCount, ((PassedCount add NotImpactedCount) div cast(TotalCount sub NotExecutedCount, Edm.Decimal)) mul 100, 0) as PassRate) "
    ,null, [Implementation="2.0",OmitValues = ODataOmitValues.Nulls,ODataVersion = 4]) 
in
    Source
```

#### [OData query](#tab/odata/)

[!INCLUDE [temp](includes/sample-odata-query.md)]

```
https://analytics.dev.azure.com/{organization}/{project}/_odata/v4.0-preview/TestResultsDaily?
$apply=filter(
	Pipeline/PipelineName eq '{pipelineName}'
	And Date/Date ge {startdate}
	And Workflow eq 'Build'
	)
/groupby(
	(TestSK, Test/TestName), 
	aggregate(
	ResultCount with sum as TotalCount,
	ResultPassCount with sum as PassedCount,
	ResultFailCount with sum as FailedCount,
	ResultNotExecutedCount with sum as NotExecutedCount,
	ResultNotImpactedCount with sum as NotImpactedCount,
	ResultFlakyCount with sum as FlakyCount))
/filter(FailedCount gt 0)
/compute(
iif(TotalCount gt NotExecutedCount, ((PassedCount add NotImpactedCount) div cast(TotalCount sub NotExecutedCount, Edm.Decimal)) mul 100, 0) as PassRate)
```

***


### Failed tests for Release workflow

Use the following queries to view the failed tests for a **Release** workflow pipeline.

#### [Power BI query](#tab/powerbi/)

[!INCLUDE [temp](includes/sample-powerbi-query.md)]

```
let
   Source = OData.Feed ("https://analytics.dev.azure.com/{organization}/{project}/_odata/v4.0-preview/TestResultsDaily?
$apply=filter("
                &"Pipeline/PipelineName eq '{pipelineName}' "
                &"And Date/Date ge {startdate}) "
        &"/groupby((TestSK, Test/TestName, Workflow), "
        &"aggregate( "
            &"ResultCount with sum as TotalCount, "
                &"ResultPassCount with sum as PassedCount, "
                &"ResultFailCount with sum as FailedCount, "
            &"ResultNotExecutedCount with sum as NotExecutedCount, "
                &"ResultNotImpactedCount with sum as NotImpactedCount, "
            &"ResultFlakyCount with sum as FlakyCount)) "
        &"/filter(FailedCount gt 0) "
    &"/compute( "
    &"iif(TotalCount gt NotExecutedCount, ((PassedCount add NotImpactedCount) div cast(TotalCount sub NotExecutedCount, Edm.Decimal)) mul 100, 0) as PassRate) "
    ,null, [Implementation="2.0",OmitValues = ODataOmitValues.Nulls,ODataVersion = 4]) 
in
    Source
```

#### [OData query](#tab/odata/)

[!INCLUDE [temp](includes/sample-odata-query.md)]

```
https://analytics.dev.azure.com/{organization}/{project}/_odata/v4.0-preview/TestResultsDaily?
$apply=filter(
	Pipeline/PipelineName eq '{pipelineName}'
	And Date/Date ge {startdate})
/groupby((TestSK, Test/TestName, Workflow), 
	aggregate(
	ResultCount with sum as TotalCount,
	ResultPassCount with sum as PassedCount,
	ResultFailCount with sum as FailedCount,
	ResultNotExecutedCount with sum as NotExecutedCount,
	ResultNotImpactedCount with sum as NotImpactedCount,
	ResultFlakyCount with sum as FlakyCount))
/filter(FailedCount gt 0)
/compute(
iif(TotalCount gt NotExecutedCount, ((PassedCount add NotImpactedCount) div cast(TotalCount sub NotExecutedCount, Edm.Decimal)) mul 100, 0) as PassRate)
```

***

### Failed tests filtered by Branch

To view the failed tests of a pipeline for a particular branch, use the following queries. To create the report, carry out the following extra steps along with what is specified later in this article.

- Expand `Branch` into `Branch.BranchName`
- Select Power BI Visualization Slicer and add the field `Branch.BranchName` to the slicer's **Field**
- Select the branch name from the slicer for which you need to see the outcome summary.

To learn more about using slicers, see [Slicers in Power BI](/power-bi/visuals/power-bi-visualization-slicers).
 

#### [Power BI query](#tab/powerbi/)

[!INCLUDE [temp](includes/sample-powerbi-query.md)]

```
let
   Source = OData.Feed ("https://analytics.dev.azure.com/{organization}/{project}/_odata/v4.0-preview/TestResultsDaily?
$apply=filter("
                &"Pipeline/PipelineName eq '{pipelineName}' "
                &"And Date/Date ge {startdate} "
        &"And Workflow eq 'Build') "
        &"/groupby((TestSK, Test/TestName, Branch/BranchName), "
            &"aggregate( "
                &"ResultCount with sum as TotalCount, "
                &"ResultPassCount with sum as PassedCount, "
            &"ResultFailCount with sum as FailedCount, "
                &"ResultNotExecutedCount with sum as NotExecutedCount, "
            &"ResultNotImpactedCount with sum as NotImpactedCount, "
        &"ResultFlakyCount with sum as FlakyCount)) "
    &"/filter(FailedCount gt 0) "
    &"/compute( "
    &"iif(TotalCount gt NotExecutedCount, ((PassedCount add NotImpactedCount) div cast(TotalCount sub NotExecutedCount, Edm.Decimal)) mul 100, 0) as PassRate) "
    ,null, [Implementation="2.0",OmitValues = ODataOmitValues.Nulls,ODataVersion = 4]) 
in
    Source
```

#### [OData query](#tab/odata/)

[!INCLUDE [temp](includes/sample-odata-query.md)]

```
https://analytics.dev.azure.com/{organization}/{project}/_odata/v4.0-preview/TestResultsDaily?
$apply=filter(
	Pipeline/PipelineName eq '{pipelineName}'
	And Date/Date ge {startdate}
	And Workflow eq 'Build')
/groupby((TestSK, Test/TestName, Branch/BranchName), 
	aggregate(
	ResultCount with sum as TotalCount,
	ResultPassCount with sum as PassedCount,
	ResultFailCount with sum as FailedCount,
	ResultNotExecutedCount with sum as NotExecutedCount,
	ResultNotImpactedCount with sum as NotImpactedCount,
	ResultFlakyCount with sum as FlakyCount))
/filter(FailedCount gt 0)
/compute(
iif(TotalCount gt NotExecutedCount, ((PassedCount add NotImpactedCount) div cast(TotalCount sub NotExecutedCount, Edm.Decimal)) mul 100, 0) as PassRate)
```

***

### Failed tests filtered by test file

To view the failed tests for a pipeline and a particular test file, use the following queries. To create the report, carry out the following extra steps along with what is defined later in this article.

- Expand `Test` into `Test.ContainerName`
- Select Power BI Visualization Slicer and add the field `Test.ContainerName` to the slicer's **Field**
- Select the container name from the slicer for which you need to see the outcome summary.
  

#### [Power BI query](#tab/powerbi/)

[!INCLUDE [temp](includes/sample-powerbi-query.md)]

```
let
   Source = OData.Feed ("https://analytics.dev.azure.com/{organization}/{project}/_odata/v4.0-preview/TestResultsDaily?
$apply=filter("
                &"Pipeline/PipelineName eq '{pipelineName}' "
                &"And Date/Date ge {startdate}) "
        &"/groupby((TestSK, Test/TestName, Test/ContainerName), "
        &"aggregate( "
            &"ResultCount with sum as TotalCount, "
                &"ResultPassCount with sum as PassedCount, "
                &"ResultFailCount with sum as FailedCount, "
            &"ResultNotExecutedCount with sum as NotExecutedCount, "
                &"ResultNotImpactedCount with sum as NotImpactedCount, "
            &"ResultFlakyCount with sum as FlakyCount)) "
        &"/filter(FailedCount gt 0) "
    &"/compute( "
    &"iif(TotalCount gt NotExecutedCount, ((PassedCount add NotImpactedCount) div cast(TotalCount sub NotExecutedCount, Edm.Decimal)) mul 100, 0) as PassRate) "
    ,null, [Implementation="2.0",OmitValues = ODataOmitValues.Nulls,ODataVersion = 4]) 
in
    Source
```

#### [OData query](#tab/odata/)

[!INCLUDE [temp](includes/sample-odata-query.md)]

```
https://analytics.dev.azure.com/{organization}/{project}/_odata/v4.0-preview/TestResultsDaily?
$apply=filter(
	Pipeline/PipelineName eq '{pipelineName}'
	And Date/Date ge {startdate})
/groupby((TestSK, Test/TestName, Test/ContainerName), 
	aggregate(
	ResultCount with sum as TotalCount,
	ResultPassCount with sum as PassedCount,
	ResultFailCount with sum as FailedCount,
	ResultNotExecutedCount with sum as NotExecutedCount,
	ResultNotImpactedCount with sum as NotImpactedCount,
	ResultFlakyCount with sum as FlakyCount))
/filter(FailedCount gt 0)
/compute(
iif(TotalCount gt NotExecutedCount, ((PassedCount add NotImpactedCount) div cast(TotalCount sub NotExecutedCount, Edm.Decimal)) mul 100, 0) as PassRate)
```

***

### Failed tests filtered by test owner

To view the Failed test for a pipeline for tests owned by a particular test owner, use the following queries. To create the report, carry out the following extra steps along with what is defined later in this article.

- Expand `Test` into `Test.TestOwner`
- Select Power BI Visualization Slicer and add the field `Test.TestOwner` to the slicer's **Field**
- Select the test owner from the slicer for which you need to see the outcome summary.

#### [Power BI query](#tab/powerbi/)

[!INCLUDE [temp](includes/sample-powerbi-query.md)]

```
let
   Source = OData.Feed ("https://analytics.dev.azure.com/{organization}/{project}/_odata/v4.0-preview/TestResultsDaily?
$apply=filter("
                &"Pipeline/PipelineName eq '{pipelineName}' "
                &"And Date/Date ge {startdate}) "
        &"/groupby((TestSK, Test/TestName, Test/TestOwner), "
        &"aggregate( "
            &"ResultCount with sum as TotalCount, "
                &"ResultPassCount with sum as PassedCount, "
                &"ResultFailCount with sum as FailedCount, "
            &"ResultNotExecutedCount with sum as NotExecutedCount, "
                &"ResultNotImpactedCount with sum as NotImpactedCount, "
            &"ResultFlakyCount with sum as FlakyCount)) "
        &"/filter(FailedCount gt 0) "
    &"/compute( "
    &"iif(TotalCount gt NotExecutedCount, ((PassedCount add NotImpactedCount) div cast(TotalCount sub NotExecutedCount, Edm.Decimal)) mul 100, 0) as PassRate) "
    ,null, [Implementation="2.0",OmitValues = ODataOmitValues.Nulls,ODataVersion = 4]) 
in
    Source
```

#### [OData query](#tab/odata/)

[!INCLUDE [temp](includes/sample-odata-query.md)]

```
https://analytics.dev.azure.com/{organization}/{project}/_odata/v4.0-preview/TestResultsDaily?
$apply=filter(
	Pipeline/PipelineName eq '{pipelineName}'
	And Date/Date ge {startdate})
/groupby((TestSK, Test/TestName, Test/TestOwner), 
	aggregate(
	ResultCount with sum as TotalCount,
	ResultPassCount with sum as PassedCount,
	ResultFailCount with sum as FailedCount,
	ResultNotExecutedCount with sum as NotExecutedCount,
	ResultNotImpactedCount with sum as NotImpactedCount,
	ResultFlakyCount with sum as FlakyCount))
/filter(FailedCount gt 0)
/compute(
iif(TotalCount gt NotExecutedCount, ((PassedCount add NotImpactedCount) div cast(TotalCount sub NotExecutedCount, Edm.Decimal)) mul 100, 0) as PassRate)
```

*** 

## Substitution strings and query breakdown


[!INCLUDE [temp](includes/sample-query-substitutions.md)]
- `{organization}` - Your organization name
- `{project}` - Your team project name
- `{pipelinename}` - Your pipeline name. Example: `Fabrikam hourly build pipeline`
- `{startdate}` - The date to start your report. Format: YYYY-MM-DDZ. Example: `2021-09-01Z` represents September 1, 2021. Don't enclose in quotes or brackets and use two digits for both, month and date.

### Query breakdown

The following table describes each part of the query.

:::row:::
   :::column span="1":::
   **Query part**
   :::column-end:::
   :::column span="1":::
   **Description**
   :::column-end:::
:::row-end:::
---
:::row:::
   :::column span="1":::
   `$apply=filter(`
   :::column-end:::
   :::column span="1":::
   Start `filter()` clause.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `Pipeline/PipelineName eq '{pipelineName}'`
   :::column-end:::
   :::column span="1":::
   Return test runs for the specified pipeline.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `and CompletedOn/Date ge {startdate}`
   :::column-end:::
   :::column span="1":::
   Return test runs on or after the specified date.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `and Workflow eq 'Build'`
   :::column-end:::
   :::column span="1":::
   Return test runs for `Build` workflow pipeline.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `)`
   :::column-end:::
   :::column span="1":::
   Close `filter()` clause.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `/groupby(`
   :::column-end:::
   :::column span="1":::
   Start `groupby()` clause.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `(TestSK, Test/TestName),`
   :::column-end:::
   :::column span="1":::
   Group by the test Name
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `aggregate(`
   :::column-end:::
   :::column span="1":::
   Start `aggregate` clause to sum the test runs matching the filter criteria.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `ResultCount with sum as TotalCount,`
   :::column-end:::
   :::column span="1":::
   Count the total number of test runs as `TotalCount`.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `ResultPassCount with sum as PassedCount,`
   :::column-end:::
   :::column span="1":::
   Count the total number of passed test runs as `PassedCount`.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `ResultFailCount with sum as FailedCount,`
   :::column-end:::
   :::column span="1":::
   Count the total number of failed test runs as `FailedCount`.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `ResultNotExecutedCount with sum as NotExecutedCount`
   :::column-end:::
   :::column span="1":::
   Count the total number of not executed test runs as `NotExecutedCount`.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `ResultNotImpactedCount with sum as NotImpactedCount,`
   :::column-end:::
   :::column span="1":::
   Count the total number of not affected test runs as `NotImpactedCount`.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `ResultFlakyCount with sum as FlakyCount`
   :::column-end:::
   :::column span="1":::
   Count the total number of flaky test runs as `FlakyCount`.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `))`
   :::column-end:::
   :::column span="1":::
   Close `aggregate()` and `groupby()` clauses.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `/compute(`
   :::column-end:::
   :::column span="1":::
   Start `compute()` clause.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `iif(TotalCount gt NotExecutedCount, ((PassedCount add NotImpactedCount) div cast(TotalCount sub NotExecutedCount, Edm.Decimal)) mul 100, 0) as PassRate`
   :::column-end:::
   :::column span="1":::
   For all the tests, calculate `PassRate`.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
   `)`
   :::column-end:::
   :::column span="1":::
   Close `compute()` clause.
   :::column-end:::
:::row-end:::

 
[!INCLUDE [temp](includes/rename-query.md)]

## Expand the Test column in Power BI

Expand the `Test` column to show the expanded entity `Test.TestName`. Expanding the column flattens the record into specific fields. To learn how, see [Transform Analytics data to generate Power BI reports, Expand columns](transform-analytics-data-report-generation.md#expand-columns). 

## Change column data type

1. From the Power Query Editor, select the `TotalCount`, `PassedCount`, `FailedCount`, `NotExecutedCount`, `NotImpactedCount`, and `FlakyCount`  columns; select **Data Type** from the **Transform** menu; and then choose **Whole Number**.

1. Select the `PassRate` column; select **Data Type** from the **Transform** menu; and then choose **Decimal Number**.

To learn more about changing the data type, see  [Transform Analytics data to generate Power BI reports, Transform a column data type](transform-analytics-data-report-generation.md#transform-data-type). 

[!INCLUDE [temp](includes/close-apply.md)]

## Create the Table report
 
1. In Power BI, under **Visualizations**, choose  **Table** and drag and drop the fields onto the **Columns** area. 

	:::image type="content" source="media/pipeline-test-reports/visualizations-failed-test-table.png" alt-text="Screenshot of visualization fields selections for Failed tests table report. ":::

1. Add the following fields to the **Columns** section in the order listed.  

	- `Test.TestName`
	- `TotalCount`
	- `PassedCount`
	- `FailedCount`
	- `NotImpactedCount`
	- `NotExecutedCount`
	- `FlakyCount`
	- `PassRate`
 

Your report should look similar to the following image. 

:::image type="content" source="media/pipeline-test-reports/failed-tests-table-report.png" alt-text="Screenshot of Sample Failed Tests Table report.":::
 

[!INCLUDE [temp](includes/pipeline-test-task-resources.md)]


## Related articles

[!INCLUDE [temp](includes/sample-related-articles-pipelines.md)]

