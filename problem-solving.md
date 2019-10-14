# Problem solving

Solving is a Modifier stage which is available whenever a problem has one or more possible solutions. This stage is respectively called Solver, or the entire name, Modifier Solver.

Before we get into solving, we have to address that Solver is still fairly in Beta development and doesn't work the most efficiently in many cases. But there's still a lot what can be done to it.

Our problems have no solutions, so let's add them.

> ?? This tutorial follows on from Making your first modification. You may wanna check that one first if you're too confused from this one.

```php
Void Problem_ItemCheckpoint(SChItem _Checkpoint) {
	declare SChProblem P;
	P.Name = "ITEM_CHECKPOINT";
	P.Items.add(_Checkpoint);
	P.ShortDescription = "Checkpoint is an item.";
	P.LongDescription = "This checkpoint is an item and can't be rotated.";

	P.Solutions.add(
		Solution("Remove the item manually."); // here
	)

	Problem(P);
}

Void Problem_CannotReplaceBlock(SChBlock _Block) {
	declare SChProblem P;
	P.Name = "CANNOT_REPLACE_BLOCK";
	P.Blocks.add(_Block);
	P.ShortDescription = "Cannot replace " ^ _Block.Name;
	P.LongDescription = "This block cannot be replaced. The block to replace is probably ghost block.";

	P.Solutions.add(
		Solution("Remove the item manually."); // here
	)

	Problem(P);
}
```

Just by adding this, the Solver gets entirely unlocked! You are able to scroll through the blocks and items you have defined in your problem.

Each solution you add gains its ID by 1, starting from 0. Our problems don't have more than 1 solution so we can prepare this structure:

```php
***Solver***
***
switch(Solver_Problem.Name) {
	case "ITEM_CHECKPOINT": {
		switch(Solver_Solution) {
			case 0: {
				
			}
		}
	}
	case "CANNOT_REPLACE_BLOCK": {
		switch(Solver_Solution) {
			case 0: {
				
			}
		}
	}
}
```

More explanations coming soon.