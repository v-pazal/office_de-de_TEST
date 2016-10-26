
# PageSetup.PrintTitleColumns Property (Excel)

Returns or sets the columns that contain the cells to be repeated on the left side of each page, as a string in A1-style notation in the language of the macro. Read/write  **String**.


## Syntax

 _Ausdruck_. **PrintTitleColumns**

 _Ausdruck_ A variable that represents a **PageSetup** object.


## Remarks

If you specify only part of a column or columns, Microsoft Excel expands the range to full columns.

Set this property to  **False** or to the empty string ("") to turn off title columns.

This property applies only to worksheet pages.


## Example

This example defines row three as the title row, and it defines columns one through three as the title columns.


```
Worksheets("Sheet1").Activate 
ActiveSheet.PageSetup.PrintTitleRows = ActiveSheet.Rows(3).Address 
ActiveSheet.PageSetup.PrintTitleColumns = _ 
 ActiveSheet.Columns("A:C").Address
```


## Siehe auch


#### Konzepte


[PageSetup Object](2fd22df9-5987-f723-04a9-9a3f2e84ac81.md)
#### Weitere Ressourcen


[PageSetup Object Members](http://msdn.microsoft.com/library/feabe079-cb03-f560-6032-88f5585ec8a8%28Office.15%29.aspx)