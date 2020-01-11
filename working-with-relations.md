# Working with block relations

> ?? Relation system is still not completed in terms of creating relations, but the base will possibly stay as I still haven't found anything better after several months.

Block relation is currently the most effective Modifier technology for defining block similarities and sharing information between blocks.

If we take off the exception of its complexity, by using relations, we can create almost 100% accurate challenges compatible with every environment. Again though, if the map contains ghost blocks, there's nothing that the script can do with this. Scripts using the relation system are considered as high tier. Reverse, Checkpointless, Comeback, and NoCP currently use the relation system.

For each block name, you can define one or more connection partners. To align the partners properly The partner can have small differences, so you can adjust its relative location so it works just like the first block. Each relation is defined in a set of relations for each environment. Each environment can have its own relation set. Each block relation from the relation set has its specific blocks with adjusts, like the coordination offset or clockwise rotation offset.

## Relations in code

So, the relation variable type is `SChRelationBlock[][][Text]`. XD Now yes, this looks really complex at first, but we can make it look way simpler by expressing this in JSON.

```json
{
	"Canyon": [
		[
			{
				"Name": "RoadRaceCheckpointMirror",
				"Coord": [0,0,0]
			},
			{
				"Name": "RoadRaceStraight",
				"Coord": [0,0,0],
				"Direction": 1
			}
		],
		[
			{
				"Name": "RoadRaceCheckpoint"
			},
			{
				"Name": "RoadRaceStraight",
				"Direction": 3
			}
		]
	],
	"Stadium": [
		[
			{
				"Name": "StadiumRoadMainCheckpoint"
			},
			{
				"Name": "StadiumRoadMain"
			}
		],
		[
			{
				"Name": "StadiumGrassCheckpoint"
			},
			{}
		]
	]
}
```

If you know JSON, you surely see here the pattern.

This is a relation set between checkpoints and roads. First, the separation between environments is happening. For each environment, you get `SChRelationBlock[][]`. This is, again, a set of relations of one environment. The relation is then defined by `SChRelationBlock[]`. Here, usually, this contains only a pair of blocks, but you can add even 3 blocks or more. Although, you shouldn't really have one element in a relation set.

If you have relation between a block and **no block**, you shouldn't forget adding an empty element as the second one. As you can see in the example relation set, this is pretty much defined like this in JSON:

```json
[
	{
		"Name": "StadiumGrassCheckpoint"
	},
	{} // this
]
```

Each block can but doesn't have to contain offset values.

- `Coord` - 3-dimentional integer defining location offset
- `Direction` - Clockwise rotation offset

If you need to have any general relations (like relation between start and finish), I've defined several relation sets in the official API which you can use as one-liners:

- `GetRelation_StartFinish()`
- `GetRelation_StartMultilap()`
- `GetRelation_FinishCheckpoint()`
- `GetRelation_MultilapStartTwoway()`
- `GetRelation_MultilapFinishTwoway()`
- `GetRelation_StartTwowayFinishTwoway()` - doesn't internally work yet
- `GetRelation_CheckpointRoad()`

## Reading through the relation set

Reading is actually the easiest part of using relations.

But first, we have to extract the environment of the current track so that the modification is optimized. We do that by extracting the map info with `GetMapInfo()` or `GetOriginalMapInfo()`. The first or second one doesn't really matter in this case.

```php
***Main***
***
declare MapInfo = GetMapInfo();
***
```

You can get environment info from `MapInfo.Environment` To start reading the actual relation set, definitely, the best way of doing this is with `foreach`:

```php
***Main***
***
declare MapInfo = GetMapInfo();

foreach(Relation, GetRelation_StartFinish()[MapInfo.Environment]) {

}
***
```

With this, inside the `foreach` loop, you have access to every start and finish relation.

To make it more readable, we assign the relation blocks into better-named variables:

```php
***Main***
***
declare MapInfo = GetMapInfo();

foreach(Relation, GetRelation_StartFinish()[MapInfo.Environment]) {
	declare StartBlock = Relation[0];
	declare FinishBlock = Relation[1];
}
***
```

> !? Don't forget! If the relation contains various amount of blocks, you may end up with script crashes by calling `Relation[0]` or `Relation[1]` straight away when the block isn't there. In these cases (therefore you don't have to do it always), check the `Relation.count` value.

Another reminder, the relation block is **not** a normal block of type `SChBlock`, but `SChRelationBlock`! It's a similar type to `SChApproxBlock` although instead of exact position and direction, the offsets of these are presented instead.

So if we want to replace all starts with all finishes (just a theory example lol), we would do:

```php
***Main***
***
declare MapInfo = GetMapInfo();

foreach(Relation, GetRelation_StartFinish()[MapInfo.Environment]) {
	declare StartBlock = Relation[0];
	declare FinishBlock = Relation[1];

	ReplaceAllBlocks(StartBlock.Name, FinishBlock.Name);
}
***
```

To use the power of offset, you add up to exact coordination and direction. Doing that for coordination is rather easy, for directions, you can use additional functions.

```php
declare Direction = ClockwiseDirection(SomeBlock.Direction, StartBlock.Direction);
declare Coord = SomeBlock.Coord + StartBlock.Coord;
PlaceBlock(SomeBlock.Name, Coord, Direction);
```

I don't have an example of this one just yet. Learn more about directions [here](#/direction-functions).

## Writing own relation sets

There are more ways of doing this, the first one is by doing is the "variable" way.

```php
declare SChRelationBlock[][][Text] RelationSet;
declare SChRelationBlock[][] Relations;
declare SChRelationBlock[] Relation;
declare SChRelationBlock Block1;
declare SChRelationBlock Block2;

Block1.Name = "OneBlock";
Block2.Name = "SimilarBlock";

Relation.add(Block1);
Relation.add(Block2);
Relations.add(Relation);
RelationSet["Canyon"] = Relations;
```

I don't know personally how to do the cleanest system of this one.

Another way is by parsing the JSON file into entire `SChRelationBlock[][][Text]`:

```php
declare SChRelationBlock[][][Text] RelationSet;
declare Success = RelationSet.fromjson("""{
	"Canyon": [
		[
			{
				"Name": "RoadRaceCheckpointMirror",
				"Coord": [0,0,0]
			},
			{
				"Name": "RoadRaceStraight",
				"Coord": [0,0,0],
				"Direction": 1
			}
		],
		[
			{
				"Name": "RoadRaceCheckpoint"
			},
			{
				"Name": "RoadRaceStraight",
				"Direction": 3
			}
		]
	],
	"Stadium": [
		[
			{
				"Name": "StadiumRoadMainCheckpoint"
			},
			{
				"Name": "StadiumRoadMain"
			}
		],
		[
			{
				"Name": "StadiumGrassCheckpoint"
			},
			{}
		]
	]
}""");
```

> ?? Formatting may be a little wrong on this one, apologies.

Or you can make the most compact relation set definer ever:

```php
declare SChRelationBlock[][][Text] RelationSet;
declare Success = RelationSet.fromjson("""{"Canyon":[[{"Name":"RoadRaceCheckpointMirror","Coord":[0,0,0]},{"Name":"RoadRaceStraight","Coord":[0,0,0],"Direction":1}],[{"Name":"RoadRaceCheckpoint"},{"Name":"RoadRaceStraight","Direction":3}]],"Stadium":[[{"Name": "StadiumRoadMainCheckpoint"},{"Name":"StadiumRoadMain"}],[{"Name":"StadiumGrassCheckpoint"},{}]]}""");
```

You can always contact BigBang1112 on Discord if you need help with relations.