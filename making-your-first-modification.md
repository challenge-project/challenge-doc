# Making your first modification

We can start making a modification that will rotate all checkpoints by 180°. Let's call it *Czechspin iksde* because why not. As always, we begin with the default structure.

```php
***Metadata***
***
Script.Name = "Czechspin iksde";
Script.AuthorLogin = "bigbang1112"; // put your Maniaplanet login there
Script.Description = "Rotates all checkpoints by 180°.";
Script.CompatibleCollections = "Canyon,Stadium,Valley,Lagoon";
***

***Main***
***

***
```

Let's write the stuff in the Main.

As we focus only on checkpoints, we should only extract checkpoint information from the map. But first, we have to know what kinds of checkpoints can exist. Yes, items! Currently, it's not possible to place items with Challenge API (even the actual ManiaScript API doesn't support it). And so we can't make item checkpoints to rotate automatically. We'll define this as a problem later in the tutorial. We'll extract both blocks and items for now.

```php
***Main***
***
declare Checkpoints = GetCheckpoints();
***
```

We get an array of checkpoints that we can now use to scroll through and rotate. To do so, we use `foreach`.

```php
***Main***
***
declare Checkpoints = GetCheckpoints();
foreach(Checkpoint in Checkpoints) {

}
***
```

The thing to remember: the type of `Checkpoint` is `SChCheckpoint`, which defines:

- `IsBlock`
- `Block`
- `IsItem`
- `Item`

Checkpoint still can be block or item and we don't know it straight away. Because we can't place items using Challenge API, right now, we only check if the checkpoint is a block.

```php
***Main***
***
declare Checkpoints = GetCheckpoints();
foreach(Checkpoint in Checkpoints) {
	if(Checkpoint.IsBlock) {

	}
}
***
```

Now we can get each checkpoint block and spin it. This is doable with a handy function called `ReplaceBlock()` as it contains a special parameter `Boolean _OppositeDirection`.

> !! DON'T change the subinformation of each checkpoint to a different direction! Even though it may sound way easier, nicer and cleaner, modifying subinformation of structs does not modify maps. Only function calls can do that. You can imagine the checkpoint as a printed paper. Changing stuff on the paper doesn't change the actual document you have on your drive.

The function returns a little more advanced replacement result, known as `SReplacementResult`: containing if the block was removed and placed.

```php
***Main***
***
declare Checkpoints = GetCheckpoints();
foreach(Checkpoint in Checkpoints) {
	if(Checkpoint.IsBlock) {
		declare Replacement = ReplaceBlock(Checkpoint.Block, Checkpoint.Block.Name, True);
	}
}
***
```

Ok, so what is going on here...

`ReplaceBlock(Checkpoint.Block, Checkpoint.Block.Name)` is a technique that removes and places the exact same block. This function alone is pretty useless. But the opposite direction parameter can make it actually useful.

The script as of now would work and is usable in many cases. You can test it now using the Test button.

But it isn't done yet. First thing after you tested the script, you've noticed that the modification status is not moving at all. We are going to fix that.

## Making user interface to work properly

The only thing you can change about the user interface when modification happens is to send commands to Modifier Status. Progress is the most complex out of all these so I'm going to cover it first.

### Progress

We have to decide and code in how our percentage is going to progress until 100%. It always depends on specific scripts, but in our case, we just want to divide all of the checkpoints into each progress section, so it goes as smoothly as it can go.

We need to have some kind of counter of each checkpoint progressively. There are two ways for this. Using `for` loop instead of `foreach`, which counts as you loop through, or have another variable before `foreach` and add `+1` on each loop. The first way can become a mess with more complex scripts so we are going to stick with the second way.

```php
***Main***
***
declare Checkpoints = GetCheckpoints();

declare Counter = 0;
foreach(Checkpoint in Checkpoints) {
	if(Checkpoint.IsBlock) {
		declare Replacement = ReplaceBlock(Checkpoint.Block, Checkpoint.Block.Name, True);
	}

	Counter += 1;
	SetStatusProgress(Counter/Checkpoints.count);
}
***
```

Simple enough right. Well, this won't work for one reason which you shouldn't forget next time.

`Counter` and `Checkpoints.count` are of a type `Integer`. Operations with integers will return an integer as well. As we are working with range 0-1, dividing can only return 0 or 1. We need a decimal point number for the progress. To solve this problem, you have to convert either `Counter` or `Checkpoints.count` to `Real`.

Real is defined by a number and **one decimal point**, so the fix is rather easy. Change

```php
declare Counter = 0;
```

to

```php
declare Counter = 0.;
```

Now, after you test the script, the Modifier Status should nicely show exactly what's going on.

### Stage

The current modification standard is:

- You first retrieve map information (that includes blocks), which is called **analyzing**.
- Then do actual modifications on a map, called **modifying**.

So, the code would be:

```php
***Main***
***
SetStatusStage("{{{{ANALYZING}}}}..."); // {{{{}}}} is a translation definer
declare Checkpoints = GetCheckpoints();

SetStatusStage("{{{{MODIFYING}}}}...");
declare Counter = 0.;
foreach(Checkpoint in Checkpoints) {
	if(Checkpoint.IsBlock) {
		declare Replacement = ReplaceBlock(Checkpoint.Block, Checkpoint.Block.Name, True);
	}

	Counter += 1;
	SetStatusProgress(Counter/Checkpoints.count);
}
***
```

The status DONE is displayed automatically after the script reaches the end.

### Message

Works pretty much the same way, just by using `SetStatusMessage()`.

```php
SetStatusStage("{{{{MODIFYING}}}}...");
declare Counter = 0.;
foreach(Checkpoint in Checkpoints) {
	if(Checkpoint.IsBlock) {
		SetStatusMessage("Rotating block " ^ Checkpoint.Block.Name ^ " by 180°...");
		declare Replacement = ReplaceBlock(Checkpoint.Block, Checkpoint.Block.Name, True);
	}

	Counter += 1;
	SetStatusProgress(Counter/Checkpoints.count);
}
***
```

When modification ends, the message is set automatically to how the script has been executed.

## Now about handling problems

You can see that on some maps you try, the results aren't so accurate. They even may not ever be accurate, but the script shouldn't ever end up as successful. In this first tutorial, we won't be creating solutions for them.

### "If the checkpoint is item" problem

Let's define the problem standardly below the Main section:

```php
	SetStatusProgress(Counter/Checkpoints.count);
}
***

Void Problem_ItemCheckpoint(SChItem _Checkpoint) {
	declare SChProblem P;
	P.Name = "ITEM_CHECKPOINT";
	P.Items.add(_Checkpoint);
	P.ShortDescription = "Checkpoint is an item.";
	P.LongDescription = "This checkpoint is an item and can't be rotated.";
	Problem(P);
}
```

Once this function is called, a fatal problem happens as there is no solution defined. Fatal problems end the script instantly and the map can't be saved nor played.

So we check if a checkpoint is an item, then call the problem out. Very simple as of now:

```php
SetStatusStage("{{{{MODIFYING}}}}...");
declare Counter = 0.;
foreach(Checkpoint in Checkpoints) {
	if(Checkpoint.IsBlock) {
		SetStatusMessage("Rotating block " ^ Checkpoint.Block.Name ^ " by 180°...");
		declare Replacement = ReplaceBlock(Checkpoint.Block, Checkpoint.Block.Name, True);
	}
	else if(Checkpoint.IsItem) {
		Problem_ItemCheckpoint(Checkpoint.Item);
	}

	Counter += 1;
	SetStatusProgress(Counter/Checkpoints.count);
}
***
```

Even though this normally shouldn't happen, if the found checkpoint is not a block or even item, we should have a statement for this case as well. We'll just add an "unknown" problem defined by the Challenge API already.

```php
SetStatusStage("{{{{MODIFYING}}}}...");
declare Counter = 0.;
foreach(Checkpoint in Checkpoints) {
	if(Checkpoint.IsBlock) {
		SetStatusMessage("Rotating block " ^ Checkpoint.Block.Name ^ " by 180°...");
		declare Replacement = ReplaceBlock(Checkpoint.Block, Checkpoint.Block.Name, True);
	}
	else if(Checkpoint.IsItem) {
		Problem_ItemCheckpoint(Checkpoint.Item);
	}
	else {
		Problem();
	}

	Counter += 1;
	SetStatusProgress(Counter/Checkpoints.count);
}
***
```

### "Block not properly replaced" problem

Blocks can sometimes be of a ghost (and relatively often on custom maps!). Challenge API can't work with ghost blocks, but can efficiently detect them. Let's first add the new check. We do that around `Replacement` variable. It's good to check both removed and placed at once to save some lines of code.

```php
SetStatusStage("{{{{MODIFYING}}}}...");
declare Counter = 0.;
foreach(Checkpoint in Checkpoints) {
	if(Checkpoint.IsBlock) {
		SetStatusMessage("Rotating block " ^ Checkpoint.Block.Name ^ " by 180°...");
		declare Replacement = ReplaceBlock(Checkpoint.Block, Checkpoint.Block.Name, True);
		if(!Replacement.Removed || !Replacement.Placed) {
			Problem_CannotReplaceBlock(Replacement.Block);
		}
	}
	else if(Checkpoint.IsItem) {
		Problem_ItemCheckpoint(Checkpoint.Item);
	}
	else {
		Problem();
	}

	Counter += 1;
	SetStatusProgress(Counter/Checkpoints.count);
}
***
```

Guess what, we forgot to add the function.

```php
	Problem(P);
}

Void Problem_CannotReplaceBlock(SChBlock _Block) {
	declare SChProblem P;
	P.Name = "CANNOT_REPLACE_BLOCK";
	P.Blocks.add(_Block);
	P.ShortDescription = "Cannot replace " ^ _Block.Name;
	P.LongDescription = "This block cannot be replaced. The block to replace is probably ghost block.";
	Problem(P);
}
```

That should be enough of the possible problems the script can have. If one of these problems will happen on a certain map, the challenge will become invalid (therefore unplayable and not possible to save).

These problems have potential solutions though! The solving is a bit more advanced topic which I cover in Problem solving tutorial.

## The full code!

```php
***Metadata***
***
Script.Name = "Czechspin iksde";
Script.AuthorLogin = "bigbang1112"; // put your Maniaplanet login there
Script.Description = "Rotates all checkpoints by 180°.";
Script.CompatibleCollections = "Canyon,Stadium,Valley,Lagoon";
***

***Main***
***
SetStatusStage("{{{{ANALYZING}}}}...");
declare Checkpoints = GetCheckpoints();

SetStatusStage("{{{{MODIFYING}}}}...");
declare Counter = 0.;
foreach(Checkpoint in Checkpoints) {
	if(Checkpoint.IsBlock) {
		SetStatusMessage("Rotating block " ^ Checkpoint.Block.Name ^ " by 180°...");
		declare Replacement = ReplaceBlock(Checkpoint.Block, Checkpoint.Block.Name, True);
		if(!Replacement.Removed || !Replacement.Placed) {
			Problem_CannotReplaceBlock(Checkpoint.Block);
		}
	}
	else if(Checkpoint.IsItem) {
		Problem_ItemCheckpoint(Checkpoint.Item);
	}
	else {
		Problem();
	}

	Counter += 1;
	SetStatusProgress(Counter/Checkpoints.count);
}
***

Void Problem_ItemCheckpoint(SChItem _Checkpoint) {
	declare SChProblem P;
	P.Name = "ITEM_CHECKPOINT";
	P.Items.add(_Checkpoint);
	P.ShortDescription = "Checkpoint is an item.";
	P.LongDescription = "This checkpoint is an item and can't be rotated.";
	Problem(P);
}

Void Problem_CannotReplaceBlock(SChBlock _Block) {
	declare SChProblem P;
	P.Name = "CANNOT_REPLACE_BLOCK";
	P.Blocks.add(_Block);
	P.ShortDescription = "Cannot replace " ^ _Block.Name;
	P.LongDescription = "This block cannot be replaced. The block to replace is probably ghost block.";
	Problem(P);
}
```

## Why is this script problematic

Challenge scripts may not sometimes be as easy as we expect. I didn't expect the outcome of this script even myself.

When you rotate blocks which have a very fixed angle to be placeable, like tilted checkpoints or 90° steep checkpoints, it just isn't rotated 180° the right way.

Currently, there's no other technology for this case other than making block relations. It's basically a huge array of each block with their relation block and additionally with relative position and direction.

You can learn about block relations [here](#/working-with-relations).

I consider scripts that use relations for all possible environments as professional and high quality, as they would work on every Nadeo map out there and on maps without ghost blocks.

Though, it's a thing you shouldn't bother when you want to experiment with what could be fun as a challenge. Feel free to contact BigBang1112 on Discord if you need any help with your script.