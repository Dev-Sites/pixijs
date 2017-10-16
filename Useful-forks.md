### Explanation 

Sometimes just making a plugin is not enough to add new things in PIXI. Here's list of forks that add special features that are too heavy to be made into separate js, because they change a lot in pixi core itself.

It takes more time to keep forks up-to-date, but if you need particular feature and trust the author, that can be optimal pick.

### 4.0.0-alpha 3d fork

Adds 3d transforms, all the features will be reproduced in pixi-projection later: https://github.com/gameofbombs/pixi.js

Few examples: https://gameofbombs.github.io/pixi-bin/index.html?s=flip&f=cards.js&title=Cards

### Node-js fork

Allows to use pixi with nodejs: https://github.com/saschagehlich/pixi.js/tree/feature/node-js

### Finscn fork

Has many useful algorithms, including pixi-lights reproduced in v4 and better mesh update: https://github.com/finscn/pixi.js . Ask the author about examples.

### Typescript fork for v5

Work in progress: no filters, no renderTexture, but already supports both WebGL1&2 and has advanced stage tree: https://github.com/gameofbombs/gobi