Wish you a Merry Christmas.<br>
Here is the KQL to build Christmas !!!<br>
Feel free to try your own<br>
Thank you<br>

```

//Christmas KQL
let tree_height = 10;
let invisible_space = '\u00AD';
range i from 0 to tree_height*2  step 2   
 | extend side_width = tree_height + 1 - i /2
| extend side_space = strcat_array
(repeat(strcat(invisible_space,''),  side_width), " ")
| project Happy_Holidays = 
case(i != 0, strcat(side_space, "🎄", 
strcat_array(repeat("-", i-1), ""), @"🎄", side_space),
strcat(side_space, " 🎄", side_space))

```
