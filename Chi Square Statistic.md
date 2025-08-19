```
= (data, ID, attributes, expected, value_column, count_column) =>
  let reduced_data = Table.SelectColumns(data, List.Combine({ID, attributes})),
  grouped = Table.Group(reduced_data, ID, {{"Chi Square", each
    let pivotted = Table.UnpivotOtherColumns(_, ID, "Attribute", "Value"),
    joined = Table.NestedJoin(expected, {value_column}, pivotted, {"Value"}, "Chi Square", JoinKind.LeftOuter),
    with_rowcounts = Table.AddColumn(joined, "squared deviations", each 
      let count = Record.Field(_, count_column) in Number.Power(Table.RowCount([Chi Square]) - count, 2) / count)
    in List.Sum(with_rowcounts[squared deviations])
  }})
 
  in grouped
```

The above will calculate a chi-square test statistic (defined as $\Sigma\frac{(o{i} - e{i}) ^ 2}{e{i}}$)

**Parameters**
|Name| Description|
|---|---|
|data|The Table of data to compare. Assumes we are looking at a wide formatted table, and applying a goodness of fit to responses across a subset of columns.|
|ID|The grouping factor or a List of factors that indicates distinct observations.|
|attributes|A List of column names across which the goodness of fit statistic should be calculated.|
|expected|A Table of response options and their expected frequencies.|
|value_column|The name of the column in `expected` that indicates response options.|
|count_column|The name of the column in `expected` that indicates expected frequencies.|

*Note*: At the moment, `expected` needs to be specified and supplied into the function separately. To do: add code to generate expected frequency table.
