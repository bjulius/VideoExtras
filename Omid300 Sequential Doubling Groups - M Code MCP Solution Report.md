# **Sequential Doubling Groups - Solution Report** #

## Problem Statement ##

  Given a table of daily sales data, group consecutive rows such that:
  1. Groups are formed sequentially from top to bottom
  2. Each group minimizes the number of rows (closes as soon as threshold is met)
  3. Each subsequent group's total sales must be ≥ 2× the previous group's total sales

  Input Data: 17 rows of sales data ranging from 1 to 30

  Target Output: 5 groups with totals: 14, 28, 57, 122, 146

  Results Comparison

| Group | Target Total | Actual Total | Status  |
| ----- | ------------ | ------------ | ------- |
| 1     | 14           | 14           | ✅ MATCH |
| 2     | 28           | 28           | ✅ MATCH |
| 3     | 57           | 57           | ✅ MATCH |
| 4     | 122          | 122          | ✅ MATCH |
| 5     | 146          | 146          | ✅ MATCH |

  Result: 5/5 groups match perfectly (100%)

 Final M Code

  

```
let
      Source = Excel.CurrentWorkbook(){[Name="InputData"]}[Content],

  // Convert to list for processing
  SalesList = Table.ToRecords(Source),

  // Accumulate with grouping logic
  Grouped = List.Accumulate(
      SalesList,
      [Group = 1, CurrentSum = 0, PreviousSum = 0, Results = {}],
      (state, current) =>
          let
              NewSum = state[CurrentSum] + current[Sales],
              RequiredSum = if state[Group] = 1 then 0 else state[PreviousSum] * 2,
              MeetsThreshold = NewSum >= RequiredSum,

              // Add current row to current group
              NewRecord = [Sales = current[Sales], Group = state[Group]],
              NewResults = state[Results] & {NewRecord},

              // If threshold met, increment group for next iteration
              NextGroup = if MeetsThreshold then state[Group] + 1 else state[Group],
              NextPreviousSum = if MeetsThreshold then NewSum else state[PreviousSum],
              NextCurrentSum = if MeetsThreshold then 0 else NewSum
          in
              [
                  Group = NextGroup,
                  CurrentSum = NextCurrentSum,
                  PreviousSum = NextPreviousSum,
                  Results = NewResults
              ]
  ),

  // Convert back to table
  ResultsTable = Table.FromRecords(Grouped[Results]),

  // Group by Group number and sum Sales
  FinalGrouped = Table.Group(
      ResultsTable,
      {"Group"},
      {{"Total Sales", each List.Sum([Sales]), type number}}
  )

  in

###     FinalGrouped
```

##  Iteration Summary ##

  Iteration 1: ✅ SUCCESS

  Approach: Implemented a state-machine algorithm using List.Accumulate with the following logic:
  - State tracking: Current group number, running sum for current group, and previous group's total
  - Threshold logic: For group 1, accept any value (threshold = 0). For subsequent groups, require sum ≥ 2× previous group total
  - Group closure: When a row causes the threshold to be met, assign it to the current group, then increment the group counter and reset the running       
    sum for the next row
  - Aggregation: After assigning groups to all rows, use Table.Group to sum sales by group

  Result: All 5 groups matched target values on first attempt with 100% accuracy.

  Key insight: The problem required minimizing rows per group, which meant closing each group immediately when the doubling threshold was reached. This  was achieved by checking the threshold after adding each row, not before.

## PROMPT USED TO GENERATE REPORT (CLAUDE SONNET 4.5) ##

```
Create a plan to solve the problem framed in @Omid300.png.  

Using the data in the left-hand side table as input, write M code that will Group the rows (minimizing the number of rows per group) sequentially from the top so that the first group contains one row, and the total sales in each subsequent group are greater than or equal to twice the total sales of the previous group.

Then total the sales in each group, and test your answer row by row against the target values in the righthand-side table of @Omid300.png using the quick_test tool

Keep iterating until the values for all rows match.

When finished provide a markdown report summarizing the problem, show the results of the M code vs the target values comparing line by line, provide the final M code and summarize briefly what you did in each iterative loop
```

Brian Julius, 1 Oct 2025