# Modifier

Modifier is a major Challenge tool that can process Modifier scripts (also called modification scripts or just modifications, challenge script is an incorrect term).

Modifier can run in **Interface mode** or **Fast mode**:
- Interface mode can be seen in the Challenge Sandbox title pack when applying a script. You can see the interface as the modification flow is step-by-step.
- Fast mode can be seen in the Challenge Maker when running modifying in bulk. The interface is not visible and the modification flow is super fast (fastest it can be).

## Modification

Not to be confused with the modification script, Modification is a state of Modifier which modifies the map. The progress of the modification can be called modification flow. Modification has 3 specialized UI sections + 1 shared UI section (with Result).

### Metadata

Metadata displays information extracted from the currently-applying modification script. Currently:
- Script name
- Script author login
- Script description

(image soon)

### Script

Script currently just has a name of the currently-applying script and approximate time remaining to finish the script. The time approximation is still not elegant but it does a job.

(image soon)

### Status

Status displays the modification state, progress and current modification description

(image soon)

### Map

At the top right, there's a map name and its challenge to be made, or that is already made. This UI module is shown in both Modification and Result.

(image soon)

## Result

Result is shown after Modification (after the modification has been completed, successfully or not). It gives you a preview of the map.

There are two different types of Result:
- Successful - Shows Validate/Solve and compute shadows options and Exit button
- Failed - Shows only Exit button

Successful is shown if the script has been executed successfully or with solvable problems. Failed is shown if the script has come to an unsolvable problem.

(image soon)

## Solver

Solver is a stage of Modifier (separate from Result) which solves problems by providing a user interface. On the right, there is a list of all problems with the challenge. On the left, there is a list of possible solutions for each problem.

(image soon)

Clicking on the solution will process a code that the modification script has. The solution flow is fully customizable by the script and not defined beforehand. When the executed script ends, the user is returned back to the standard Solver interface. If the executed script ends with variable `Solver_Success = True`, the problem from the right window will disappear.

## Processor

Processor is [a library](https://github.com/challenge-project/challenge-sandbox/blob/master/Scripts/Libs/Challenge/Processor.Script.txt) that contains all of the actions (commands) Modifier can do in Modification or Solver state and many block technologies to ensure modification safety.

> ?? Modification safety basically means that the modification was executed **how the code expects**. If the script executed `RemoveBlock()` and the block wasn't removed physically even though the function returned success, then the function is not safe.

Processor processes actions that are sent from the Modification script **through** the Modifier **to the Processor**. Actions are sent via events from the Challenge API from the official functions. The action is defined usually with `Ch_[FUNCTION_NAME_FROM_API]`. In the code, each action goes through the `switch` statement and looks if the action name matches. If yes, it extracts the arguments and converts them to the correct types with help of the Conversion library, and those arguments are used to apply the actions under `CMapEditorPlugin` ManiaScript context.

### Conversion

Conversion is a library for Processor to assist with conversion through different types, especially into official API reflections. It is the major helper for converting.

> ?? Usage of struct replicas of official ManiaScript classes in the Challenge API might be controversial, but this had to be done due to the action arguments that have to be sent through JSON format. I have a potential idea that could use official classes (but honestly, sometimes their members are just not ideal for Challenge) and remove the struct subinformation change problem, which might become Challenge API 2 someday, though, rather unlikely. Most of the Challenge structs aren't exact replicas and add additional handy features. Structs are actually necessary.
