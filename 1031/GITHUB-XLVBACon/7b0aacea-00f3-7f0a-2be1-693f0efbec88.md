
# Worksheet.ProtectScenarios Property (Excel)

 **True** if the worksheet scenarios are protected. Read-only **Boolean**.


## Syntax

 _Ausdruck_. **ProtectScenarios**

 _Ausdruck_ A variable that represents a **Worksheet** object.


## Example

This example displays a message box if scenarios are protected on Sheet1.


```
If Worksheets("Sheet1").ProtectScenarios Then _ 
 MsgBox "Scenarios are protected on this worksheet."
```


## Siehe auch


#### Konzepte


[Worksheet Object](182b705e-854a-81cc-a4b0-59b942de55ae.md)
#### Weitere Ressourcen


[Worksheet Object Members](http://msdn.microsoft.com/library/f8c1afea-1a1c-f5e4-37e3-52c434c8c157%28Office.15%29.aspx)