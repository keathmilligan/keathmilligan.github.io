---
layout: post
title: 'Cmder tip: enable forward slash completion'
date: 2016-10-07 23:01:31 -0500
categories: [dev]
tags: [cmder, tools]
featured-img: cmder_icon.png
---

<img src="/assets/images/cmder_icon.png" align="right" style="max-width:150px;">[Cmder](http://cmder.net/) includes better command-line tab completion – normally, paths will be completed using the default Windows path separator, the backslash (“\”). But Windows has recognized the forward slash (“/”) as an optional path separator for a while now and there are instances where this might be desirable, especially if you switch between Windows and Unix-like environments (macOS, Linux) often as I do.
<!--more-->

Internally, cmder uses a utility called [clink](https://mridgers.github.io/clink/) to enable bash-style command-line completion (and other things). You can enable forward-slash as a path separator, but adding some additional clink configuration to your cmder setup. If it does not exist, create cmder.lua in the config directory where you installed cmder. Add the following code:

```lua
local function force_fwd_slashes(text, first, last)
    clink.slash_translation(1)
    return false
end
clink.register_match_generator(force_fwd_slashes, -1)
```

Save the file and restart cmder. Paths will now be completed using forward slashes instead of backslashes.

This issue was originally raised in the [clink issues database](https://github.com/mridgers/clink/issues/395).
