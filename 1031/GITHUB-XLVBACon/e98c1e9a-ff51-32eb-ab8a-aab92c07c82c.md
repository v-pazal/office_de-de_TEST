
# DataTable.HasBorderOutline Property (Excel)

 **True** if the chart data table has outline borders. Read/write **Boolean**.


## Syntax

 _Ausdruck_. **HasBorderOutline**

 _Ausdruck_ A variable that represents a **DataTable** object.


## Example

This example causes the embedded chart data table to be displayed with an outline border and no cell borders.


```
With Worksheets(1).ChartObjects(1).Chart 
 .HasDataTable = True 
 With .DataTable 
 .HasBorderHorizontal = False 
 .HasBorderVertical = False 
 .HasBorderOutline = True 
 End With 
End With
```


## Siehe auch


#### Konzepte


[DataTable Object](aca0850b-2e72-cde9-b751-633876e1df99.md)
#### Weitere Ressourcen


[DataTable Object Members](http://msdn.microsoft.com/library/5a46944b-e7e6-ac7c-6b95-736975a0a3eb%28Office.15%29.aspx)