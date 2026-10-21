# cmpm121-code-smells
**Names**: Aaron Tan, Charles Lesser, Julian Tran

**Chosen Smells**: Shotgun Surgery, Feature Envy, Large Class

## Shotgun Surgery
Ever added something or made a change to a game, such as adding a new item or type of item, and found yourself scouring through many functions to add one precise value? We all did at one point. I can pinpoint one exact situation - when I made a clicker game based on minecraft. 14 year old me programmed it in Java in the same way I had seen when modding the game, which was with a thousand different values in different places. Let's take the example of adding a new item, coal, to the "market" I had made.
```c++
//ID reference
//...
int item1ID = 14
int coalID = 15;
int item2ID = 16
//...

//string reference
//...
char[] item1Name = "Item 1";
char[] coalName = "Coal";
char[] item2Name = "Item 2";
//...

//market reference
//...
int item1Cost = 9000;
int coalCost = 10;
int item2Cost = 30
//...

```
