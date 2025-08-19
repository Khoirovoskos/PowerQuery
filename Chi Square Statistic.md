```
= (data, ID, attributes, expected, count_column, value_column) =>
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
