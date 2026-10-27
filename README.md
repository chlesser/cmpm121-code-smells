# cmpm121-code-smells
**Names**: Aaron Tan, Charles Lesser, Julian Tran

**Chosen Smells**: Shotgun Surgery, Feature Envy, Primitive Obsession

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

//random drops
public static int getRandomIDByArea() {
  std::random_device rd;
  std::mt19937 gen(rd());
  std::uniform_int_distribution<int> distrib(1, 100);
  int random_num = distrib(gen);

  //...
  if(0 < num && num <= 10) return item1ID
  else if(10 < num && num <= 20) return coalID
  else if(20 < num && num <= 30) return item2ID
  //...
}
```

There are issues here beyond just shotgun surgery, but for the purposes of this example, one can see that to change anything about the coal drop rate, cost, or item names, I would hunt down particular values in particular functions in particular files. A cleaner example here is to merge the "shotgun" surgery into a much more precise operation each time.

First, we would create a class for an item with _Combine Functions Into Class (144)_
```c++
class Item {
    public:
        int ID;
        char[] name;
        int cost;
        int dropChance;
};
```

While I won't write out the constructor, now one can create an item with an easy initialization in the same place.
```c++
Item coal = new Item(14, "Coal", 10, 10)
```
At the end of this, we have taken these scattered value and combined them into a single place where they can be easily edited. This takes away the risk of forgetting details here and there in code.

## Feature Envy
Feature envy is when a class uses methods and properties from another class more than its own. This problem arises when you manipulate the data of a class within the methods of another class. You would do this because the class lacks a method of its own to manipulate or get its own data, so you relegate that duty to another class when you want them to interact with each other. Feature envy is dangerous because intertwines the two classes. Because of this, any changes to one class will affect the other. It may even lead to the affected class to stop working as intended or produce errors.

Feature envy can be solved by moving and extracting functions. Accessing data from another class can be replaced with access and manipulation functions from that class. Or, if the method only uses data from this other class, just move that method to the other class. To prevent feature envy, avoid directly calling properties from another class. Instead, make sure that they are only called by its own class, and if you wish to access or manipulate these properties, create a method within that class and call that instead.

Many examples of feature envy can be found in my GameMaker project. In this particular example, the function for when a player heals tries to update the UI's health sprites. This function is called by the player, yet it accesses data from the UI object by directly calling them.
```gml
function HealPlayer(Regen, Type)
{
	if (Type == BODY_DMG)
	{
		for (var i = 0; i < array_length(obj_UI.body_HEALTH); i++)
		{
			with (obj_UI.body_HEALTH[i])
			{
				resolve(global.player_BODY_HEALTH, min(global.player_BODY_HEALTH + Regen, global.player_BODY_HEALTH_MAX));
			}
		}
		global.player_BODY_HEALTH = min(global.player_BODY_HEALTH + Regen, global.player_BODY_HEALTH_MAX);
	}
}
```
The problem with this implementation is that, if the UI object does not exist, this code breaks and produces an error. Additionally, if there are any issues with the UI object at all, then the scope of where issues may lie is extended to this function, rather than just to the functions of the UI object.

One way this can be improved is by making this a method of the UI object.
```gml
update_HEALTH = function(newHealth)
{
    for (var i = 0; i < array_length(body_HEALTH); i++)
	{
		with (body_HEALTH[i])
		{
			resolve(global.player_BODY_HEALTH, newHealth);
		}
	}
};
```
Then, you can update the HealPlayer function.
```gml
function HealPlayer(Regen, Type)
{
	if (Type == BODY_DMG)
	{
		with (obj_UI)
		{
			update_HEALTH(min(global.player_BODY_HEALTH + Regen, global.player_BODY_HEALTH_MAX));
		}
		global.player_BODY_HEALTH = min(global.player_BODY_HEALTH + Regen, global.player_BODY_HEALTH_MAX);
	}
}
```
This way, the UI object's variables remain outside of the scope of the player.

## Primitive Obsession
Primitive Obsession is a code smell that occurs when programmers rely too heavily on basic data types such as strings, integers, and booleans to represent more complicated concepts. Although these primitive types are useful for storing simple values, using them to represent something with its own rules or behavior can make code harder to maintain.

This smell often happens because using primitive values is quicker and easier than creating separate classes. However, as a program grows, developers might find themselves repeatedly checking and manipulating the same values throughout their code. This can lead to duplicated logic and make it easier for mistakes to occur.

One way to fix Primitive Obsession is through Fowler's _Replace Primitive with Object (174)_ refactoring. Instead of using primitive values to represent something with its own properties or rules, we can create a dedicated class to handle that information. This makes the code easier to organize and allows related logic to be kept in one place.

For example, imagine we're creating a pet simulator where pets have different rarities that determine their money multipliers and display colors. A simple implementation in TypeScript might look like this:
```typescript
function getMultiplier(rarity: string): number {
    if (rarity === "Common") return 1;
    if (rarity === "Rare") return 2;
    if (rarity === "Legendary") return 5;
    return 1;
}

function getColor(rarity: string): string {
    if (rarity === "Common") return "gray";
    if (rarity === "Rare") return "blue";
    if (rarity === "Legendary") return "gold";
    return "gray";
}

let petRarity = "Legendary";

console.log(getMultiplier(petRarity)); // 5
console.log(getColor(petRarity)); // gold
```

While this implementation works, both functions rely on strings to determine the correct result. If we accidentally misspell a rarity, such as "Legendery", the functions won't recognize it and will instead return their default values. Additionally, if we wanted to add more rarities, we would have to update both functions.

To improve this, we can create a PetRarity class that stores the properties associated with each rarity.
```typescript
class PetRarity {
    constructor(
        public name: string,
        public multiplier: number,
        public color: string
    ) {}
}

const common = new PetRarity("Common", 1, "gray");
const rare = new PetRarity("Rare", 2, "blue");
const legendary = new PetRarity("Legendary", 5, "gold");

function getMultiplier(rarity: PetRarity): number {
    return rarity.multiplier;
}

function getColor(rarity: PetRarity): string {
    return rarity.color;
}

let petRarity = legendary;

console.log(getMultiplier(petRarity)); // 5
console.log(getColor(petRarity)); // gold
```

With this refactoring, the rarity information is stored in one class instead of being represented by arbitrary strings. The functions no longer need to check multiple string values because they can directly access the properties of the PetRarity object.

This also makes adding new rarities easier since we can define them in the class without modifying the existing functions. TypeScript can also catch errors when a function expecting a PetRarity receives an ordinary string.