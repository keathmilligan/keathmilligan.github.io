---
layout: post
title: "Obsidian plugin development: cross-platform mobile/desktop testing"
date: 2022-03-09 18:13:00 -0600
categories: [dev]
tags: [obsidian, javascript, typescript, android, ios, chrome]
image: /assets/images/obsidian.png
---

![Debugging Obsidian](/assets/images/obsidian-debug.png)

Testing and debugging an Obsidian plugin on multiple platforms.

<!--more-->

> If you're new to Obsidian plugin development, checkout the [Obsidian Plugin Developer Docs](https://marcus.se.net/obsidian-plugin-docs/getting-started/create-your-first-plugin). This guide assumes you are familiar with the basics.

When developing a plugin for [Obsidian](https://obsidian.md/), it is important to make sure it behaves correctly on all of the platform types that it is going to run on. While most plugins that work on desktop will *probably* work OK in other environments, there's no way of knowing for sure without testing first. And it isn't uncommon to discover issues that are unique to mobile or a specific platform.

# Setup your environment

Prepare your plugin development environment. For testing on multiple platforms, you'll need some sort of synchronization method available. I use [Obsidian Sync](https://obsidian.md/sync), but other methods will also work.

## Create and configure a test vault

> It's important not to do plugin development and testing in your regular notes vault. Plugins can modify and delete notes, folders and even the entire vault, so always work in a test vault.
{: .danger}

On a desktop system (Mac, Linux or Windows), create a new vault for testing your plugin, e.g. "PluginDev".

Open the test vault in Obsidian and go to Settings > Community plugins and disable safe mode to allow third party plugins. This is also a good time to change any other settings you need to adjust and to install other plugins you need.

## Setup vault synchronization

In order to propagate changes to your plugin to other devices, you will need to use a synchronization method.

> **If you are using Obsidian Sync:**
> - Go to **Settings** and enable the **Sync** plugin.
> - Go to **Sync settings > Pick remote vault** and click the **Choose** button.
> - Create a new remote vault.
> - Connect to the vault and click **Start Synching**.
> - Connect to the vault and click **Start Synching**.
> - Go to **Sync settings** again and enable:
>     - Active core plugin list
>     - Core plugin settings
>     - Active community plugin list
>     - Installed community plugins

# Desktop testing

Now we're ready to test our plugin on desktop.

## Checkout local workspace

Clone or create your plugin project workspace. `cd` into this directory and run `yarn` and `yarn build` (or `npm i` and `npm run build`).

### An alternative approach to storing your project in the `.obsidian/plugins` dir

> The plugin development guide suggests cloning your plugin repository into `<your vault>/.obsidian/plugins` directly. This is a simple approach and works fine, but I prefer to manage my Git repositories elsewhere and sym-link to them from the plugins directory. You don't have to do it this way, but there are some advantages especially when synchronizing plugin projects that have a large number of dependencies.
{: .tip}

Instead of linking the entire directory, we will create a folder in the `.obsidian/plugins` directory for our plugin and create links to the `main.js`, `manifest.json` and `styles.css` files individually. By doing this, we avoid syncing the entire project across multiple devices - for small projects, this is not too much of a problem, but for more complex plugins with a lot of dependencies, the unnecessary sync of the `node_modules` directory can take quite a while and consume a lot of space in the remote vault.

Open a terminal window (Mac/Linux) or a PowerShell window (Windows) and `cd` to the `.obsidian/plugins` directory of your test vault. If this directory does not exist, create it.

Next create a directory for your plugin project and `cd` into this directory.

#### Mac/Linux

Create the sym-links on Mac/Linux with:

```bash
ln -s /path/to/workspace/main.js main.js
ln -s /path/to/workspace/manifest.json manifest.json
ln -s /path/to/workspace/main.js main.js
```

Where `/path/to/workspace` is where your plugin source code is stored.

#### Windows (PowerShell)

> To create symbolic links with PowerShell, you may need to enable Windows [Developer Mode](https://docs.microsoft.com/en-us/windows/apps/get-started/enable-your-device-for-development). TLDR: Go to **Settings > Update & Security > For developers** and turn on "Developer Mode".

```powershell
New-Item -ItemType SymbolicLink -Path "$PWD\main.js" -Target "path\to\workspace\main.js"
New-Item -ItemType SymbolicLink -Path "$PWD\manifest.json" -Target "path\to\workspace\manifest.json"
New-Item -ItemType SymbolicLink -Path "$PWD\styles.css" -Target "path\to\workspace\styles.css"
```

## Run/debug

Now you are ready to run and debug your plugin. Start `yarn dev` or `npm run dev` in the project directory. In Obsidian, navigate to **Settings > Community Plugins** and click the reload button. Make sure your plugin is listed and is enabled.

Now you can press Shift+Ctrl+I (Windows/Linux) or Alt+⌘+I (Mac) to open Developer Tools to browse the DOM, see console output, etc. When running in dev/watch mode, you should see your plugin's `main.ts` listed in the Sources tab:

![Obsidian Source Map](/assets/images/obsidian-source-map.png)

If your plugin project has additional sources (for example in a `src` directory), they should also be visible.

Use the source maps to set breakpoints, step through your code, etc:

![Debugging Obsidian](/assets/images/obsidian-debug.png)

> Source-level debugging will only work on the desktop system that is running the `yarn dev` or `npm run dev` command. If you synchronize your plugin to the remote vault and open it on another desktop system, it will work and you can use the developer tools to view the DOM and see the console, but the sources (`main.ts`, etc.) will not be able available. To debug on another desktop system, you will need to check out your plugin source there and setup the environment as described above. 
{: .warning }

### Hot Reload

When you make changes to your plugin's source code, the dev/watch process will automatically rebuild `main.js` when you save, but these changes will not be reflected in Obsidian until you enable/disable the plugin in settings or restart Obsidian. Fortunately, the [Hot Reload](https://github.com/pjeby/hot-reload) plugin will automatically reload your plugin as you make changes.

Follow the [installation instructions](https://github.com/pjeby/hot-reload) to add Hot Reload to your Obsidian test vault.

#### Enable hot reload

Normally, Hot Reload will automatically reload a plugin with a `.git` sub-directory in its installation directory. If you are using the sym-linking approach described above, the `.git` sub-directory will not be present. An alternative way to enable hot reload is to create a `.hotreload` file in the plugin directory:

Mac/Linux:
```bash
touch /path/to/vault/.obsidian/plugins/your-plugin-name/.hotreload
```

Windows (PowerShell):
```powershell
New-Item \path\to\vault\.obsidian\plugins\your-plugin-name\.hotreload
```

# Mobile testing

## Setup your test vault

On your mobile device, create a new test vault, e.g. "PluginDev". As mentioned above, *don't* use your regular vault for plugin development testing.

Next, go to **Settings > Community plugins** and make sure "Safe Mode" is off.

> **If you are using Obsidian Sync:**
> - Go to **Settings** and enable the **Sync** plugin.
> - Go to **Sync settings > Pick remote vault** and click the **Choose** button.
> - Create a new remote vault.
> - Connect to the vault and click **Start Synching**.
> - Connect to the vault and click **Start Synching**.
> - Go to **Sync settings** again and enable:
>     - Active core plugin list
>     - Core plugin settings
>     - Active community plugin list
>     - Installed community plugins

### Check your plugin

> I have found that at this point, it is necessary to restart Obsidian or close and reopen the vault in order for the plugins to be enabled.
{: .tip }

If everything has synchronized correctly, your plugin should now be active. Go to **Settings > Community plugins** to check.

## Mobile Debugging

### Android devices

### IOS devices

While it is possible to use Safari's web inspector to inspect and debug Capacitor apps in development, this won't work for debugging Obsidian plugins because IOS does not allow inspection of release-mode apps (bummer). Until/unless the Obsidian dev team comes up with an alternative, the best bet for debugging on IOS is to use one of the following:

- [Obsidian Dev Tools](https://github.com/KjellConnelly/obsidian-dev-tools) - a plugin that displays a console log within Obsidian. Unfortunately, it looks like this plugin is no longer maintained.
- Add [this script](https://gist.github.com/liamcain/3f21f1ee820cb30f18050d2f3ad85f3f) to your plugin. It will create a console log file in your plugin's directory.
