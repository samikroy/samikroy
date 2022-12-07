```
let Left_Data_Table = datatable(LeftKey:string, Continent:string)['key1','South America',    'key2','Europe',    'key2','Australia',       'key3','North America'];
let Right_Data_Table = datatable(RightKey:string, Country:string)['key2','France',  'key3','Canada',   'key3','Greenland',    'key4','India'];
//Left_Data_Table | join kind=innerunique Right_Data_Table on $left.LeftKey == $right.RightKey
//Right_Data_Table | join kind=innerunique Left_Data_Table on $left.RightKey == $right.LeftKey
//Left_Data_Table | join kind=inner  Right_Data_Table on $left.LeftKey == $right.RightKey
//Left_Data_Table | join kind=leftouter Right_Data_Table on $left.LeftKey == $right.RightKey
//Left_Data_Table | join kind=rightouter Right_Data_Table on $left.LeftKey == $right.RightKey
//Left_Data_Table | join kind=fullouter Right_Data_Table on $left.LeftKey == $right.RightKey
//Left_Data_Table | join kind=leftanti  Right_Data_Table on $left.LeftKey == $right.RightKey
//Left_Data_Table | join kind=rightanti  Right_Data_Table on $left.LeftKey == $right.RightKey
//Left_Data_Table | join kind=leftsemi   Right_Data_Table on $left.LeftKey == $right.RightKey
Left_Data_Table | join kind=rightsemi   Right_Data_Table on $left.LeftKey == $right.RightKey
```
