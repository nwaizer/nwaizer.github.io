---
layout: post
title: Adding rsync to goland flatpak
tags: [go, redfish]
---
![flatpak](https://blogs.gnome.org/mclasen/files/2018/02/flatpak.png)

<iframe width="560" height="315" src="https://www.youtube.com/embed/GkgPIJp8_30" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

So ever since I got this work laptop, with Red Hat Enterprise Linux pre-installed, I was introduced to flatpaks.
This makes perfect sense to me as I find myself using containers these days in testing much new stuff I get online.

One drawback with the Goland flatpak was it was missing rsync, to allow me to develop code remotely as I often do.

![reddit](/assets/posts/goland_rsync/reddit_linuxquestions.png)
So I asked on [Reddit](https://www.reddit.com/r/linuxquestions/comments/vbiwsh/why_does_this_flatpak_allow_access_to_some_shared/).
Then opened an [issue](https://github.com/flathub/com.jetbrains.GoLand/issues/38) for the flatpak maintainer.
Then learned about how to write my own flatpak, and then submitted a change request.

![merged](/assets/posts/goland_rsync/goland_rsync_merged.png)
I am happy to report this effort has ended well, as this PR was [merged](https://github.com/flathub/com.jetbrains.GoLand/pull/42) .
So now anyone can use rsync in their Goland flatpak.
I wish, I could also add [xxhash](https://github.com/flathub/com.jetbrains.GoLand/pull/42/commits/362255050b17870ce0aba404a54ff9e66f69b9d3#diff-9a487e6de407e9111ca188e4945c497cfea10356c043732b7124597cbb23a751R8) support for rsync, but hey, this is a different issue, right?