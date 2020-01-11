# Understanding modification scripts

Challenge is a creative project. Every modification is based on a relatively simple code that users can write themselves.

The selected coding language is ManiaScript. It is the core language of every ManiaPlanet creation, so it was an easy choice for the Challenge project. It's that kind of a language that is not hard to pick up thanks to its similarity to other languages. If you want to understand the syntax of ManiaScript, you can check the ManiaScript syntax tutorial [here](https://doc.maniaplanet.com/maniascript/syntax-basics).

## Base construction

Modifier script has a rather nice, straightforward, ManiaScriptish construction.

The script is divided into so-called labels, which are basically code sections.

```php
***SomeLabel***
***
// some related code
***

***AnotherLabel***
***
// another related code
***
```

They are defined with a name on the top and code between. These sections are called via [Modifier](#/modifier).
> ?? Labels can be called with a syntax `+++SomeLabel+++`, though this practice shouldn't be used in the modification script environment.

Modifier currently recognizes these labels:
- Metadata
- Main
- Solver

**Metadata** section should only set information about the script, as this section runs in both Browser and Modifier. In Modifier, the label runs just before the Main label.
Information is definable by using the Script global (which uses struct `SChMetadata`).

```php
***Metadata***
***
Script.Name = "My First Modification";
Script.AuthorLogin = "bigbang1112";
Script.Description = "Some brief description.";
Script.CompatibleCollections = "Canyon,Stadium,Valley,Lagoon";
***
```

Let's briefly summarize these simple statements:

- `Script.Name`: Displayed name of the script which can contain spaces.
- `Script.AuthorLogin`: Login of the author. When publishing, it should be YOURS otherwise you won't be able to publish your modification.
- `Script.Description`: Rather short description of the script which is shown to the user and when modifying.
- `Script.CompatibleCollections`: In which environments you can use the script. Define with a comma without spaces in between.

Next up is **Main** section which is where you will spend most of the time in most of the cases. It defines the actual actions Modifier should do. Reaching the end of this code section will result in completing the modification - it redirects Modifier to the completion sequence (usually by saying "DONE").

Then we have **Solver**, a section that is used only in Modifier Solver. If the script ends with `Solver_Success = False`, then the solution was unsuccessful. Very simplified: if the script ends with `Solver_Success = True`, then the solution was successful. This part will be covered more in Defining problems and solutions.

## Challenge API

API is basically an interface of many functions, structs, and globals that provide extended functionality for scripting. **Challenge API** adds almost a hundred of functions defining not only base of Map Editor Plugin (see Modifier for further information about MEP), dozens of structs and a few useful globals.

> !! **VERY IMPORTANT**: Functions of Challenge API return **ONLY INFORMATIVE VALUES**. This is especially crucial with structs. Some functions return a struct that contains subinformation. Changing a value in subinformation is allowed with regular ManiaScript, but **doesn't have an effect** in the real map modification! That means whatever you change in the structs (that aren't defined in Globals), you won't see any actual map changes. **Map does only modify with function calls**.

To change parameters of the block, there's no other way than removing the block and placing it back with new parameters (unless you use one of the more advanced functions like `ReplaceBlock()`, but the functions still work on removing and placing back).

The API is available in all sections of the modification script (Metadata, Main, Solver). In Metadata, however, you should only use the `Script` global.

See the interactive reference [here](#/reference). 

## Defining problems and solutions

Here is where things may get tricky (but fun at the same time!). Even a simple removal of all checkpoints can be problematic on some maps. Users shouldn't end up with a "successful" modification even though one checkpoint wasn't removed for some reason. To do this, we define **problems**.

Problem should have **solutions** if a challenge has a potential to be even possible to do on the map. Sometimes though, solutions are simply none or impossible, so it's completely fine to have a problem with no solution. That way, the challenge is marked as invalid. We define **solutions** for a single **problem**.

Solutions are **solvable** in **Solver**. Solver is a submodule of Modifier with a user interface made to solve problems. Solver reads problems - which contain possible solutions.

Now let's apply some code to this theory.

Problems are definable with function `Problem(SChProblem)`. The parameter here is a struct `SChProblem` which contains some of the deeper stuff:

- `Name`: Identification name of the problem. Usual format is `PROBLEM_[UPPERCASE_PROBLEM_NAME]`
- `Coords`: Coordinations that Solver will use to navigate you throughout the problem.
- `Waypoints`: Waypoints that Solver will use to navigate you throughout the problem.
- `Blocks`: Blocks that Solver will use to navigate you throughout the problem.
- `Items`: Items that Solver will use to navigate you throughout the problem.
- `ApproxBlocks`: Approximate blocks that Solver will use to navigate you throughout the problem.
- `ShortDescription`: Short description of the problem displayed on the right side.
- `LongDescription`: Long description of the problem displayed in the Solver user interface.
- `Solutions`: Solutions that Solver will use to navigate you throughout the problem.

You see, there can be a lot defined with problems. Each subinformation of `SChProblem` will be available in the Solver label.

Solution can be defined by either `SChSolution` struct or as a one-liner with `Solution()` function which returns the struct.

- `Value`: Brief description of the solution.
- `AdditionalInfo`: Additional information about the solution.
- `IsInteractive`: If you can click and perform the automated solution.

The function is basically defined by `Solution(Text _Value, Text _AdditionalInfo, Boolean _IsInteractive)` which creates this struct in one line.

Adding a solution to a problem can be as simple as:

```php
declare SChProblem Problem;
Problem.Solutions.add(
	Solution("Remove the checkpoint manually.", "", True)
);
```

### Defining problems most efficiently

This practice can be seen in the official modification scripts. Because problems are often reusable and signing all of the needed information can't be clearly done in one line, in the modification script at the top of the code, we write down a function that creates the struct and assigns it all of the details. We can optionally add parameters if the problem is slightly more dynamic. I call these functions **problem functions**.

```php
Void Problem_CannotRemoveCheckpoint(SChWaypoint _Checkpoint) {
	declare SChProblem P;
	P.Name = "CANNOT_REMOVE_CHECKPOINT";
	P.Waypoints.add(_Checkpoint);
	P.ShortDescription = "Cannot remove checkpoints.";
	P.LongDescription = "This checkpoint can't be removed for some reason.";
	P.Solutions.add(
		Solution("Remove the checkpoint manually.", "", True)
	);
	Problem(P);
}
```

> ?? This is just a good practice which isn't required but is heavily recommended.

As a developer which writes this documentation, I want to find a way to have pre-written problem functions that can be easily extended by the community, but currently, you have to live with this method.

Many of the Challenge API functions can notice if they executed improperly, especially the block related ones like `PlaceBlock()` or `RemoveSpecificBlock()`. Use as many returning values as possible to define all of the problem details.

## Interact with Modifier Status

You can let the user know what is currently happening in the scene by sending commands to the Modifier Status.

- `SetStatusProgress(Real _Progress)` - Percentage of the entire modification (you can set it to like 0.4).
- `SetStatusMessage(Text _Message)` - Exact info about modifier actions (Checkpoint found!)
- `SetStatusStage(Text _Stage)` - Header of the status (MODIFYING...)
- `SetStatusColor(Vec3 _Color)`- Color of the progress bar in RGB. If problem occurs, the color you set with this gets overriden.

That's pretty much all Challenge API has to offer for Modifier Status.

Defining progress also generates approximate remaining time of completion. In the next chapter, you will be able to learn how to use these functions properly.

## Advanced, perhaps handy information

Scripting environment runs under class context called `CMlScript`. This class contains some hidden unofficial members and functions which might be handy for you to use.
> ?? The script actually runs under two different subcontexts: `CMapEditorPluginLayer` and `CManiaAppTitleLayer`. `CMapEditorPluginLayer` runs when the modification is happening, `CManiaAppTitleLayer` runs when you're browsing scripts in Browser. Because one context doesn't have members of the other context, you shouldn't use any of the specialized members or functions from them, as that would crash scripts in one or another environment.