# The Point of Loading

Why does the loading of Challenge title packs take so long? Why other titles load instantly, meanwhile Challenge title packs must have a loading screen before the menu and playing a map?

The game is laggy during loading, so the serious work is actually being done. But which work?

## It's all about UI

User interface is the simple answer. There's a difference in how UI is handled in Challenge and other title packs. Challenge **constructs** UI while other title packs **reference** UI.

More experienced coders may say: "Why to construct the entire UI whenever there's nothing that needs to be statically changed? If something changes, just update it with Manialink script!" It's actually not that simple.

There are more reasons, but only two major:
- Coding clarity
- Translations

The thing is: The usual manialink has UI elements and code combined together. This can be seen in my Leaderboards manialink for example. This usually fails text formatting (but that's now fixed thanks to Reaby's ManiaScript VSC extension commit) and arguments against it can be pretty much applied like for JavaScript in HTML. I personally prefer the separation to also reduce Interface Designer lag.

I made a library (from the [Universe Library Set](https://github.com/BigBang1112/universe-library-set)) simply called Manialink.Script.txt to realize a new flow for coding manialinks. **It separates UI elements from code into two files. During loading, the separated files are merged and loaded into memory.** This causes a small lag which is clearly seen in the title packs. Each UI section is processed this way and the loading bar increments on each fully loaded UI section, with ~20 millisecond wait in between to have processing space to update the loading bar.

Isn't that bad though? With UI elements and code merged in one file, you can test manialink interaction in the Interface Designer! Can do in the menu, but when coding UI for gameplay, you're screwed anyway.

Additionally, thanks to the construction, you can pass through some UI element and script modifications, like translation, or even themes which are currently not available in challenge. It gets way easier doing it in construction than in the Manialink script.

> ?? The translation uses terms that have their format in the files, currently `{{{{TERM_NAME}}}}`. This entire string gets replaced with the associated `TERM_NAME` from language files depending on your Maniaplanet language setting.

## How it's different from Nadeo Envimix?

Even that everything looks similar, Nadeo Envimix loading is completely different. Nadeo Envimix uses manialink referencing (merged UI elements and code), the loading is there only for preloading thumbnails.

The loading bar increments by each loaded thumbnail. The loading sometimes feels fake, because the code waits ~20 milliseconds for the loading bar to update, but the thumbnail may be already preloaded. So you wait `20*thumbnail_count` amount of milliseconds minimally.
