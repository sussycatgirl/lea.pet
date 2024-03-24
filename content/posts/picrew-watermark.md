---
title: "How I hacked Picrew to remove a watermark"
date: 2024-03-24T15:58:36+01:00
draft: false
hidden: false
author: Lea
author_url: https://lea.pet/@lea
---

Don't worry, I'm not an art thief. But I think this is interesting, so I want to document how I did it.

<!--more-->

In case you don't know [Picrew](https://picrew.me) (though I'd expect most of my audience to), it's basically a platform where artists can publish avatar creators, similar to Nintendo Miis. It's a great tool to create an avatar without any artistic skills, and you'll see lots of people using them online.

With that out of the way, look at this:

{{< image "picrew-watermark/watermark.png" >}}

While I do strongly believe in crediting artists, I am not a fan of how this watermark is right in the middle of the picture. So, I started searching for a way to move it somewhere where it'll be less annoying, while still being visible for anyone interested enough to enlarge or download the avatar.

This is how the final result (and my current avatar) looks:

{{< image "picrew-watermark/pfp.png" >}}

### Reverse engineering Picrew

To get started, in this case we do get some control over the watermark positioning, but we can only move it around left and right in a very limited area (I highlighted the area with a yellow box here). We also get some tilt control.

{{< image "picrew-watermark/positioning.png" >}}

My first idea was to find out where the application state is stored and to modify the position value beyond what the UI allows. After some digging around, I found the relevant data stored in IndexedDB. \
Firefox doesn't make this obvious, but the element key is an array with the image maker ID (found in the page URL) and the ID of the respective part.

Getting the part ID is simple since it's attached as a property to the relevant Element:

{{< image "picrew-watermark/part-id.png" >}}

That means the key we're looking for is `["1787745", 2077979]`. Yes, the first part is a string and the second part is a number. No clue why.

{{< image "picrew-watermark/indexeddb.png" >}}

Searching for this, we get the following data:

```
{
    "image_maker_id": "1787745",
    "parts_id": 2077979,
    "parts_data": {
        "itmId": 6785413,
        "cId": 4546963,
        "xCnt": 0,
        "yCnt": 0,
        "spCnt": 0,
        "sNo": 0,
        "rotaCnt": 0
    }
}
```

I'm unsure what some of these keys are for, but `xCnt` and `yCnt` control the position. So, let's try editing these to move the watermark off screen!

... I'll spare you a summary of the hour I spent trying to figure out the IndexedDB API (because Firefox's IDB browser is read only), but this doesn't work. The client ensures that all values are in the allowed range whenever you change the element, reload the page or press "Done", and resets them to 0 if not. This was extremely frustrating because I had no idea if I was even editing the database properly.

Next I tried figuring out where the image maker configuration is stored - The position restrictions must be fetched from and stored somewhere, so we could try modifying the stored data somehow.

Checking the network tab, there are no requests that contain this configuration data, meaning it must either be stored in the page HTML or in one of the scripts it fetches.

An agonizing 30 minutes and several VSCode and Firefox crashes later I managed to read through the minified page source to the point where I found a hint as to where the data might be. Searching for the relevant part ID in the debugger tab does return a couple matches deep inside of minified JavaScript:

{{< image "picrew-watermark/debugger.png" >}}

Most of these reference `window.__NUXT__`, which appears to be a Vue.js library. Bingo, this seems to be where they store the global state, and printing its content in the console confirms this.

{{< image "picrew-watermark/nuxt.png" >}}

I spent some time digging through everything, and it appears that the config for the currently selected tab is stored in `window.__NUXT__.state.currentParts`. When selecting the tab with the watermark element, it looks like this:

{{< image "picrew-watermark/currentparts.png" >}}

We see three relevant keys, `lCnt`, `rotaLCnt` and `rotaRCnt`. The values of all three seem to match up with the limits of how we can move and rotate the watermark.

At this point it was as easy as modifying the `lCnt` key and moving the element off screen using the UI.

### The "solution"

We can increase the left movement limit for this element with the following command:

```js
window.__NUXT__.state.currentParts.lCnt = 10000;
```

If you're doing this with a different image maker, there might instead be a key called `rCnt`, or whatever name is used for up/down movements. If in doubt, check the properties of `window.__NUXT__.state.currentParts` and see what makes sense.

Now it's as easy as moving the watermark off screen with the available buttons!

Since the PNG is generated client side, that's all we need to do. Just click "Done" and download the generated image.

Keep in mind that since this modification was only done in memory, the next time the editor is opened the watermark will teleport back to its default position.

Finally, we can use the Inspect tool to download the watermark so we can place it in a more convenient space in an image editing program.

{{< image "picrew-watermark/watermark-dl.png" >}}
{{< image "picrew-watermark/gimp.png" >}}

Done :3

If you decide to do something like this, *please* include the watermark somewhere, or credit the artist elsewhere (e.g. in your profile description). Not crediting artists for their hard work is not cool.

For those interested, I used this image maker for my avatar: https://picrew.me/en/image_maker/1787745 <3
