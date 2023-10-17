---
title: "Patching your favourite Reddit client to keep working after the API pricing change"
date: 2023-07-01T19:47:06+02:00
draft: false
author: Lea
author_url: https://lea.pet/@lea
---

[ReVanced](https://github.com/revanced) is mainly used to patch the YouTube app to disable ads and some of the obnoxious "features", but we can also use it to make our favourite Android Reddit clients work again.
<!--more-->
For this, we will first need to get our own client ID. To get this, head to the [Reddit preferences](https://old.reddit.com/prefs/apps/), scroll to the bottom and create an application.

Give it a descriptive name (you can set it to whatever you want, really) and set the **type** to "**installed app**".

For the **redirect url** field, use the following depending on what client you use:

| Application | Redirect URL                  |
| :---------- | :---------------------------- |
| Infinity    | `infinity://localhost`        |
| Boost       | `http://rubenmayayo.com`      |
| Sync        | `http://redditsync/auth`      |
| BaconReader | `http://baconreader.com/auth` |
| rif is fun  | `redditisfun://auth`          |
| Relay       | `dbrady://relay`              |

Finally, complete the captcha. The screen should now look like this:

{{< image "reddit-custom-client-id/create-new-application.png" >}}

Once you click "create app", Reddit will switch to this screen. Under the application name at the top, there is a string of random characters - This is your **client ID**. Take note of it, you will need it later.

{{< image "reddit-custom-client-id/application-details.png" >}}

# Patching our app

<!-- ## On device -->

The easiest way to do this is to use ReVanced Manager to patch the app directly on device. This can be done both with the app already installed or with an apk file.

[Download ReVanced Manager from here](https://github.com/revanced/revanced-manager/releases/latest), install and open it. If it asks for permission to access your files, **grant it**. If not, go to `Settings > Apps > Special app access > All files access` and manually give it permission - Patching will fail otherwise.

Before patching, we need to tell ReVanced our client ID. This is done by writing it to a file called `reddit_client_id_revanced.txt` in the root of the user data directory (This is why ReVanced manager needs access to your files).

If you have ADB set up, you can do so quickly with this command:
```bash
adb shell 'echo -n "your-client-id-here" > /storage/emulated/0/reddit_client_id_revanced.txt'
```

Or with Termux:
```bash
termux-setup-storage # if you haven't already
echo -n "your-client-id-here" | tee /storage/emulated/0/reddit_client_id_revanced.txt
```

Otherwise, you can use any text editor app for this. Just make sure that there are no trailing newlines or spaces.

> For reference: The file must be in the user's "home" directory (`/storage/emulated/0`). \
> This is the place where you find the `Download`, `DCIM` and `Android` directories, among others.

Once you're done, open ReVanced Manager again, switch to the "**Patcher**" tab, click "Select an application" and choose your Reddit client. \
If you have it installed already, it should show up in the list. Otherwise, use the "**Storage**" button and select the APK file.

{{< image "reddit-custom-client-id/revanced-manager.png" >}}

Tap on "Selected patches" and enable `Change Oauth Client Id`. If you're interested in any of the other patches, you can pick those as well.

Once that's done, click on "**Patch!**"

Check the logs, and make sure it says `Applied change-oauth-client-id`. If all went right, you should have the option to install as root or non-root. If you pick the non-root option, you'll have to **uninstall your current Infinity version first** or Android will most likely throw an error. You can also export the patched apk file from the menu in the top right.

> The root install method tends to cause issues when updating the app, so I recommend you use the non-root method instead, even if your device is rooted.

### Did it work?

Open the installed app and verify the patch was applied correctly: Add a new account and enter your username and password. It should now display the name of the application you created in the first step.

{{< image "reddit-custom-client-id/infinity-oauth-screen.png" >}}

Hurray! You should now be able to continue using Reddit for the forseeable future, albeit probably [without access to NSFW tagged content](https://www.reddit.com/r/redditsync/comments/12qwwjh/an_update_regarding_reddits_api_changes_to_how/).

After updating the app, you can simply run ReVanced Manager again to re-patch the new version. It should remember your selected patches, so this shouldn't take longer than a minute!

<!--
## Using ReVanced CLI

If ReVanced Manager doesn't work, we can also use the ReVanced CLI to patch an APK file on our computer. This is more advanced and will be tedious to do every time you update the app, so only do this if you need to.

Download [the JAR file from here](https://github.com/revanced/revanced-cli/releases/tag/v2.21.5) and verify you can run it:

```bash
$ java -jar revanced-cli-2.21.5-all.jar
```

This should print an error message along with usage instructions.

You will also need the ReVanced Patches - Download the [JAR file from ReVanced's GitHub releases](https://github.com/revanced/revanced-patches/releases/latest).

Finally, create a file called `options.json` with your client ID:

```json
[
    {
        "patchName" : "change-oauth-client-id",
        "options" : [
            {
                "key" : "client-id",
                "value" : "put-your-client-id-here"
            }
        ]
    }
]
```

You can now patch the APK file! Run `java -jar revanced-cli-*.jar` again with the following flags:

| Flag | Value |
| :--- | :------ |
| `-a` | The unpatched input apk file |
| `-o` | The name of the patched output file |
| `-i` | List of patches we want to apply, in this case only `change-oauth-client-id` |
| `-b` | The patch bundle (`revanced-patches-vx.xx.x.jar`) |
| `--options` | Tell ReVanced to use `options.json`, where we specified the client ID |

The full command should look something like this:

```bash
java -jar revanced-cli.jar \
    -a Infinity-v6.0.2.apk \
    -o infinity-patched.apk \
    -i change-oauth-client-id \
    -b revanced-patches-*.jar \
    --options options.json
```

No errors? Great! Finally, you can install the patched apk and [make sure it was patched correctly](#did-it-work).
-->
