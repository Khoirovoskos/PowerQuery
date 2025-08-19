```
= (data, ID, attributes, expected, value_column, count_column) =>
  let as_table = Table.FromRecords({data}), // Promote current Record to a Table
  reduced_data = Table.SelectColumns(as_table, List.Combine({ID, attributes})), // Reduce the data to the specified IDs and attributes
  pivotted = Table.UnpivotOtherColumns(reduced_data, ID, "Attribute", "Value"), // Reshape the data from wide format to long.
  joined = Table.NestedJoin(expected, {value_column}, pivotted, {"Value"}, "Chi Square", JoinKind.LeftOuter), // Join the observed values to the expected values.
  with_rowcounts = Table.AddColumn(joined, "squared deviations", each // Calculate discrepancies between observations and expectancies
    let count = Record.Field(_, count_column) in Number.Power(Table.RowCount([Chi Square]) - count, 2) / count)
  in List.Sum(with_rowcounts[squared deviations]) // Return the sum of the squared deviations.
```

The above will calculate a chi-square test statistic (defined as $\Sigma\frac{(o{i} - e{i}) ^ 2}{e{i}}$)

**Parameters**
|Name| Description|
|---|---|
|data|The Record of data to compare. Assumes we are looking at a wide formatted table, and applying a goodness of fit to responses across a subset of columns.|
|ID|The grouping factor or a List of factors that indicates distinct observations.|
|attributes|A List of column names across which the goodness of fit statistic should be calculated.|
|expected|A Table of response options and their expected frequencies.|
|value_column|The name of the column in `expected` that indicates response options.|
|count_column|The name of the column in `expected` that indicates expected frequencies.|

*Note*: At the moment, `expected` needs to be specified and supplied into the function separately. To do: add code to generate expected frequency table.

**Usage Notes**
This will calculate a column of Chi Square values showing how well the specified set of attribute columns match the expected frequency of responses.

```
let input_data = Table.FromRecords({
    [Session = 1,   Q1 = 2, Q2 = 2, Q3 = 6, Q4 = 5, Q5 = 7],
    [Session = 2,   Q1 = 4, Q2 = 8, Q3 = 0, Q4 = 5, Q5 = 2],
    [Session = 3,   Q1 = 7, Q2 = 4, Q3 = 2, Q4 = 7, Q5 = 1],
    [Session = 4,   Q1 = 5, Q2 = 6, Q3 = 2, Q4 = 8, Q5 = 0],
    [Session = 5,   Q1 = 8, Q2 = 6, Q3 = 0, Q4 = 1, Q5 = 0],
    [Session = 6,   Q1 = 9, Q2 = 1, Q3 = 7, Q4 = 0, Q5 = 7],
    [Session = 7,   Q1 = 8, Q2 = 9, Q3 = 8, Q4 = 6, Q5 = 6],
    [Session = 8,   Q1 = 8, Q2 = 4, Q3 = 5, Q4 = 1, Q5 = 8],
    [Session = 9,   Q1 = 5, Q2 = 5, Q3 = 6, Q4 = 6, Q5 = 0],
    [Session = 10,  Q1 = 5, Q2 = 8, Q3 = 3, Q4 = 3, Q5 = 8]
}),
sessionID = {"Session"},
questions = {"Q1", "Q2", "Q3", "Q4", "Q5"},
response_range = {0, 9},
expected_values = let
    Source = List.Generate(() => response_range{0}, each _ <= response_range{1}, each _ + 1),
    #"Converted to Table" = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Renamed Columns" = Table.RenameColumns(#"Converted to Table",{{"Column1", "Value"}}),
    #"Added Custom" = Table.AddColumn(#"Renamed Columns", "Count", each List.Count(questions) / List.Count(Source))
    in #"Added Custom",
ChiSq = (data, ID, attributes, expected, value_column, count_column) =>
  let as_table = Table.FromRecords({data}), // Promote current Record to a Table
  reduced_data = Table.SelectColumns(as_table, List.Combine({ID, attributes})), // Reduce the data to the specified IDs and attributes
  pivotted = Table.UnpivotOtherColumns(reduced_data, ID, "Attribute", "Value"), // Reshape the data from wide format to long.
  joined = Table.NestedJoin(expected, {value_column}, pivotted, {"Value"}, "Chi Square", JoinKind.LeftOuter), // Join the observed values to the expected values.
  with_rowcounts = Table.AddColumn(joined, "squared deviations", each // Calculate discrepancies between observations and expectancies
    let count = Record.Field(_, count_column) in Number.Power(Table.RowCount([Chi Square]) - count, 2) / count)
  in List.Sum(with_rowcounts[squared deviations]) // Return the sum of the squared deviations.
in Table.AddColumn(input_data, "Chi Square", each ChiSq(_, sessionID, questions, expected_values, "Value", "Count"))
```

*returns*

```
Table.FromRecords({
    [Session = 1,   Q1 = 2, Q2 = 2, Q3 = 6, Q4 = 5, Q5 = 7, "Chi Square" = 9],
    [Session = 2,   Q1 = 4, Q2 = 8, Q3 = 0, Q4 = 5, Q5 = 2, "Chi Square" = 5],
    [Session = 3,   Q1 = 7, Q2 = 4, Q3 = 2, Q4 = 7, Q5 = 1, "Chi Square" = 9],
    [Session = 4,   Q1 = 5, Q2 = 6, Q3 = 2, Q4 = 8, Q5 = 0, "Chi Square" = 5],
    [Session = 5,   Q1 = 8, Q2 = 6, Q3 = 0, Q4 = 1, Q5 = 0, "Chi Square" = 9],
    [Session = 6,   Q1 = 9, Q2 = 1, Q3 = 7, Q4 = 0, Q5 = 7, "Chi Square" = 9],
    [Session = 7,   Q1 = 8, Q2 = 9, Q3 = 8, Q4 = 6, Q5 = 6, "Chi Square" = 13],
    [Session = 8,   Q1 = 8, Q2 = 4, Q3 = 5, Q4 = 1, Q5 = 8, "Chi Square" = 9],
    [Session = 9,   Q1 = 5, Q2 = 5, Q3 = 6, Q4 = 6, Q5 = 0, "Chi Square" = 13],
    [Session = 10,  Q1 = 5, Q2 = 8, Q3 = 3, Q4 = 3, Q5 = 8, "Chi Square" = 13]
})
```
