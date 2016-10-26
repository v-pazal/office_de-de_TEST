
# WorksheetFunction.RoundUp Method (Excel)

Rounds a number up, away from 0 (zero).


## Syntax

 _Ausdruck_. **RoundUp**( ** _Arg1_**, ** _Arg2_** )

 _Ausdruck_ A variable that represents a **WorksheetFunction** object.


### Parameters



|**Name**|**Required/Optional**|**Data Type**|**Description**|
|:-----|:-----|:-----|:-----|
| _Arg1_|Erforderlich|**Double**|Number - any real number that you want rounded up.|
| _Arg2_|Erforderlich|**Double**|Num_digits - the number of digits to which you want to round number.|

### Return Value

Double


## Remarks




- ROUNDUP behaves like ROUND, except that it always rounds a number up.
    
- If num_digits is greater than 0 (zero), then number is rounded up to the specified number of decimal places.
    
- If num_digits is 0, then number is rounded up to the nearest integer.
    
- If num_digits is less than 0, then number is rounded up to the left of the decimal point.
    

## Siehe auch


#### Konzepte


[WorksheetFunction Object](7b1d5639-363d-632c-2cf0-2232562646b6.md)
#### Weitere Ressourcen


[WorksheetFunction Object Members](http://msdn.microsoft.com/library/6811ca87-4b53-0bff-88c9-30bf7497879a%28Office.15%29.aspx)