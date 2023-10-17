---
title: "Patching an App (YouTube) with ReVanced"
date: 2023-10-12T18:02:56.071Z
draft: false
author: Sneexy
author_url: https://lea.pet/@sneexy
---

This is a guide made *not* by Lea, but rather - yours truly - [Sneexy](https://lea.pet/@sneexy)!
This is a (yet another) guide which focuses on patching apps using [ReVanced](https://revanced.app/). I'll mostly be focusing on YouTube for this one, but this procedure should mostly apply for other apps you may feel like patching with ReVanced.

<!--more-->

# Getting Started

The first thing you'd want to do is go ahead and grab the ReVanced Manager from their [lovely download page](https://revanced.app/download).

If you're going to be patching YouTube, you may want to also grab [Vanced's patched microG](https://github.com/TeamVanced/VancedMicroG/releases) if you plan on logging in with your Google account. This isn't required if you don't plan on logging in to your patched YouTube.

This entire procedure is done on Android/mobile, you won't need any extra or special devices to do this.

# Checking (and probably even patch) an app

Easiest way to check if you can patch an app is by opening the ReVanced Patcher and heading into the `Patcher` category. Tap on `Select an application` and apps that you both have installed and are patchable will show up first in the list with all of the metadata complete:

{{< image "revanced-guide/select-an-application.png" >}}

> ℹ️ Note: Depending on some default patch selections, you may have to take a detour and enable `Enable changing selecton` in ReVanced's settings as some apps have no patches selected by default and you won't be able to change the selection without enabling that setting.

In most cases of other apps, you can actually just go ahead and now patch the app you wish to fix up if the version of app you use either *exactly* matches up the `Suggested` version, or if the `Suggested` version is `All versions`. If this is the route you take, go ahead and skip to [Patching an app](#patching-an-app).

However, in the case of other apps, or most popularly, YouTube, you will need a specific version of the app. If this is the case, then you'll need to:

# Grabbing your app's APK

Go ahead and open a new tab in your browser, and search up `<name of app> <specific version> apk`. For example, in the case of YouTube (as the time of writing this), `youtube 18.38.44 apk`. You can also directly search in a app repository, which personally I would recommend using [APKMirror](https://www.apkmirror.com/) as I haven't had any issues with them. When you get to the page of your *specific version* (remember!) of the app and head to the `Avaliable Downloads` section, do ***NOT*** download the APK that has the **green badge that says "Bundle"**! It will **NOT** work, and it will just be a waste of time. Instead, download the one usually below that one as its the correct APK we're looking for.

{{< image "revanced-guide/apkmirror-download-section.png" >}}

Wait for the next page to load, scroll down then hit the big red `Download APK` button.

{{< image "revanced-guide/apkmirror-download-apk.png" >}}

> ⚠️ Warning: There may or may not be a bunch of Ads, especially those fake download buttons. I don't know, I use an adblocker (you should too, cmon, it's 2023) and can't confirm this.

# Patching an app

> ℹ️ Note: If you came from [Checking (and probably even patch) an app](#checking-and-probably-even-patch-an-app), you can skip this APK part.

Now that we have the APK we want, we can go ahead and chuck that into ReVanced Manager to patch it! Note that if you're doing this in order to patch a version of an app that you have installed that is newer than the suggested version, **you will have to uninstall your copy of the app ***first*** before attempting to patch and install!**

Open up ReVanced and head into the `Patcher` section, then tap on `Select an application`, tap the `Storage` button that's floating on the bottom right of the screen. It will open up a file picker from where you can choose the downloaded APK file, which is most likely in your Downloads folder unless you've moved it. ReVanced will automatically detect what the app is and automatically choose a preset of patches to use. In the case of YouTube, the default selection works perfectly and you can just go ahead and tap the `Patch` button on the bottom right of the screen.

It will now start patching your app! At this point you just need to wait for it to finish, and once it's done you'll see an `Install` button on the bottom right of the screen which once you tap, it does exactly as it's labeled as and tada! You have now successfully patched and installed an app using ReVanced Manager!