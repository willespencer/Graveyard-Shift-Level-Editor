# Graveyard Shift Level Editor

Level Editor for [Graveyard Shift](https://graveyardshift.page/), a stealth-puzzle game built in LibGDX where players must evade and distract “mutants” that can hear but not see.

![image](https://user-images.githubusercontent.com/25535093/131572494-8ada2392-0158-489e-87c2-76b21ba6f193.png)

The level editor allows users to start by either creating a level of specific dimensions (that can be changed later) or by loading in an existing level JSON file.

![image](https://user-images.githubusercontent.com/25535093/131572215-786f0cf0-c3b7-42e1-8364-6eda55b0b197.png)

From there, the toolbar can be used to add different types of tiles (like goals, lighting, doors, etc.), characters (players and mutants), items (keys, bricks, and bombs), and aethetics (barrels, crates, and PCs) by clicking on squares in the visualization of what the level will look like in-game. 

![image](https://user-images.githubusercontent.com/25535093/131572458-413a5d62-bc15-4332-9673-3b6846b971e1.png)

From there, mutant patrolling paths can be added by connecting mutants, and dialogue triggers can be added or edited by using the trigger menu, with options like trigger type and dimensions, and a grid of existing triggers viewable. 

![image](https://user-images.githubusercontent.com/25535093/131573590-ed7fb8fb-f7cb-4d0a-b1c6-ad24469679ac.png)

Finally, as long as the level you have built fits the rules necessary for the level to be valid (exactly 1 player, at least 1 goal, and at most 1 checkpoint), it can be outputted as a JSON file by specifying the level number it will be in the game.


## Project setup

In order to install the necessary packages, without creating any large diffs, run the following command in the repo.

```
npm ci
```

### Compiles and hot-reloads for development

In order to see the site and create your own levels, run the following command in the level-editor folder after setup.

```
npm run serve
```

### That is all you need to use the editor!

Other instructions on using the editor, drawing mutant patrols, adding dialogue triggers, and importing/exporting JSON level files are provided on the web app.
