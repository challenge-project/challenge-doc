# Best practices

## Each Challenge API function should be used wisely!

An API function takes approximately 40 miliseconds to fully complete, depending on the function expense. The most expensive functions are currently waypoint retrieving ones. They vary on map size and take about 80 milliseconds on an average map. But also appearance changes do take about 40 miliseconds either (if you want to know the details, check Modifier documentation). Running regular ManiaScript syntax is almost immediate though.

What does this mean in practice? You shouldn't scroll through every block and set the status progress for each and every block, because setting a single status progress takes these 40 milliseconds. When you have a map with thousands of blocks, the script would become insanely slow!

> Getting all blocks using `GetBlocks()` shouldn't be used in the first place because this can generate a huge array (thousands of entries usually) which commonly crash ManiaPlanet for yet unknown reasons. You should only get the blocks you really need. You can do so with `GetFilteredBlocks()` or waypoint functions.

## Item placement doesn't work with Challenge API

You can't place items automatically using the script. You can only remove anchor items, which are just waypoints at the moment.

You have to create a problem and manual solution based on position checks.

If you want the easiest way, create a problem without solutions once you get in contact with item.