# Special blocks

Special block is a Challenge API technology that defines multiple standard blocks as one block. Special block is basically made with 1-4 macroblocks, depending on the number of block variants you want to have.

Special blocks were invented to make the missing opposite waypoints possible, but its usage can be used in many other ways. Using them optimizes modifier code and improves clarity.

## How to create a special block

Macroblock name is defined by `[SpecialBlockName]-[Variant].Macroblock.Gbx`.

Full file names of the set of macroblocks would be:
- `Blocks/[Environment]/Path/To/Macroblock-Air.Macroblock.Gbx`
- `Blocks/[Environment]/Path/To/Macroblock-Ground.Macroblock.Gbx`

For Valley, Processor additionally supports two more variants:
- `Blocks/[Environment]/Path/To/Macroblock-Forest.Macroblock.Gbx`
- `Blocks/[Environment]/Path/To/Macroblock-Field.Macroblock.Gbx`

That should be it. If you have those files provided in your Challenge title, Challenge API functions should now recognize your new special block.

## Inspiration

- You can combine two kinds of waypoints in one place.
- You can create missing waypoints variants.