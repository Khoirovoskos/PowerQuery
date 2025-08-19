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
