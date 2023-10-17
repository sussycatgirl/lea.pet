---
title: "Discord Experiments"
date: 2023-09-15T17:19:16+02:00
draft: false
author: Lea
author_url: https://lea.pet/@lea
---

JS snippet that allows access to Discord's Experiments tab and other developer tools. Posting this here so I don't have to dig through GitHub gists all the time.

<!--more-->

If you don't already know what this is for you should probably stay away from it :)

> Sourced from here: [gist.github.com/MPThLee/3ccb554b9d882abc6313330e38e5dfaa](https://gist.github.com/MPThLee/3ccb554b9d882abc6313330e38e5dfaa?permalink_comment_id=4404323)

```js
let wpRequire;
window.webpackChunkdiscord_app.push([[ Math.random() ], {}, (req) => { wpRequire = req; }]);
mod = Object.values(wpRequire.c).find(x => typeof x?.exports?.Z?.isDeveloper !== "undefined");
usermod = Object.values(wpRequire.c).find(x => x?.exports?.default?.getUsers)
nodes = Object.values(mod.exports.Z._dispatcher._actionHandlers._dependencyGraph.nodes)
try {
    nodes.find(x => x.name == "ExperimentStore").actionHandler["OVERLAY_INITIALIZE"]({user: {flags: 1}})
} catch (e) {}
oldGetUser = usermod.exports.default.__proto__.getCurrentUser;
usermod.exports.default.__proto__.getCurrentUser = () => ({isStaff: () => true})
nodes.find(x => x.name == "DeveloperExperimentStore").actionHandler["CONNECTION_OPEN"]()
usermod.exports.default.__proto__.getCurrentUser = oldGetUser
```
