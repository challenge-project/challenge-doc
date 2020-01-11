# Problem solving

Solving is a Modifier stage that is available whenever a problem has one or more possible solutions. This stage is respectively called Solver, or the entire name, Modifier Solver.

Before we get into solving, we have to address that Solver is still fairly in Beta development and doesn't work the most efficiently in many cases. But there's still a lot what can be done to it.

Our problems have no solutions, so let's add them.

> ?? This tutorial follows on from [Making your first modification](#/making-your-first-modification). You may wanna check that one first if you're too confused about this one.

```php
Void Problem_ItemCheckpoint(SChItem _Checkpoint) {
	declare SChProblem P;
	P.Name = "ITEM_CHECKPOINT";
	P.Items.add(_Checkpoint);
	P.ShortDescription = "Checkpoint is an item.";
	P.LongDescription = "This checkpoint is an item and can't be rotated.";

	P.Solutions.add(
		Solution("Replace the item manually with the rotated one."); // here
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
		Solution("Replace the block manually with the rotated one."); // here
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
***
```

> ?? If you're not sure about the used `switch` statement here, check ManiaScript syntax.

To understand, the code first needs to know which problem the map has (provided by `Solver_Problem`) and which solution the user picked (provided by `Solver_Solution`). `Solver_Problem` however isn't just a problem name - it contains problem details - so we check only `Solver_Problem.Name`. It's not a display name, but an identification name.

Also, we can't create a solution for the item rotation, as items do not provide rotation information just yet.

## ITEM_CHECKPOINT solution

The problem with items is that we can't place them automatically. We were rotating blocks by removing and **placing** the blocks. Here is where we, unfortunately, have to choose the manual work. On top of this, items provide us only with its position - no rotation, no item name. We will have to rely on the user's trust.

You can imagine it like this. You have a task to do, and the only thing that the leader cares about is the result. The leader checks whenever you've done the task, or wait until you do the task if it's not completed. The exact same flow is used in problem-solving here.

We have to allow the user to access the whole editor right at the start. We do that with `ShowEditor()`.

```php
case "ITEM_CHECKPOINT": {
	switch(Solver_Solution) {
		case 0: {
			ShowEditor();
		}
	}
}
```

Now, how do we solve this. We first need to detect if the item was removed, and THEN if the item was placed back. We can do that with two *leader* stages (two tasks the leader wants to have completed in the specific order). As we can work with positions only, we have to rely on user trust.

So we can insert a *leader* to our code that *first* checks every 50 milliseconds if the problematic item has been deleted. This can be done by using a function `Exists(SChItem _Item)`, which checks if the item is actually placed on the map (as `SChItem` can be just imaginary). Basically, if it exists, just wait before it doesn't exist.

> !? Function `Exists(SChItem _Item)` will be available **in the 31 January 2020 Update.**

> ??  There are more possible options, but this one is the most optimized currently.

We do that rather simply in a loop which checks if this task was done:

```php
case "ITEM_CHECKPOINT": {
	switch(Solver_Solution) {
		case 0: {
			ShowEditor();

			declare Checkpoint = Solver_Problem.Items[0]; // we defined the checkpoint as the first item element
			while(Exists(Checkpoint)) { // Removing stage
				sleep(50);
			}
		}
	}
}
```

The `sleep(50)` helps to optimize the performance of the script while the user is solving the problem manually.

Now the placing stage. This one is the same story in terms of using the loop. But it's a little different in a way that we *should* use `ExistsInRadius(SChItem _Item, Real _Radius)`.

> !? Function `ExistsInRadius(SChItem _Item, Real _Radius)` will be also available **in the 31 January 2020 Update.**

> ??  We don't have to use it, it's just a good practice in this case.

Items are precise things which are also proven by their position being defined as Vec3. If we would check if the new item position equals the exact expected item position, solving would be often annoying. We have to implement **tolerance**.

Again, as we can get only a position from the item, we have to rely on the user's trust there and just check if the position meets - with tolerance. Let's do that now in the code! We can use our `Checkpoint` variable as a template.

```php
case "ITEM_CHECKPOINT": {
	switch(Solver_Solution) {
		case 0: {
			ShowEditor();

			declare Checkpoint = Solver_Problem.Items[0]; // we defined the checkpoint as the first item element
			while(Exists(Checkpoint)) { // Removing stage
				sleep(50);
			}

			while(!ExistsInRadius(Checkpoint, 16.)) { // 16 units tolerance in all direction
				sleep(50);
			}
		}
	}
}
```

That should be it. To let Solver know that the solution was successful, set `Solver_Success = True;` at the end of the solution.

```php
case "ITEM_CHECKPOINT": {
	switch(Solver_Solution) {
		case 0: {
			ShowEditor();

			declare Checkpoint = Solver_Problem.Items[0]; // we defined the checkpoint as the first item element
			while(Exists(Checkpoint)) { // Removing stage
				sleep(50);
			}

			while(!ExistsInRadius(Checkpoint, 16.)) { // 16 units tolerance in all direction
				sleep(50);
			}

			Solver_Success = True;
		}
	}
}
```

Setting this variable to `True` will remove the problem from the problem list when the code reaches the end.

## CANNOT_REPLACE_BLOCK solution

More explanations coming soon.