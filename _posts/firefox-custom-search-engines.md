---
title: Firefox custom search engines
date: 2023-11-21
tags: [til, firefox, browser]
description: How to create custom search engines for Firefox? I had no idea about custom search engines until I've watched a totally unrelated finance video.
---

A few weeks ago I've watched [Dumb Money](https://www.imdb.com/title/tt13957560/) which recounts the GameStop story. I was really interested in the story and started researching on the Internet. Eventually I ended up on [Roaring Kitty's YouTube channel](https://www.youtube.com/@RoaringKitty) especially on this [video](https://www.youtube.com/watch?v=x2CBcthRVKE&ab_channel=RoaringKitty). While watching the video I saw he's created custom search engines for Chrome, which I'd never heard of before. I really liked the idea of being able to search things on specific websites with a few keystrokes so I searched whether the same was possible for Firefox.

It is possible however for Firefox you need to add the preference below to your `about:config` and set it to `true`.
```
browser.urlbar.update2.engineAliasRefresh
```

Then you can go to Settings > Search and you'll be able to see the "Add" button under the section "Search Shortcuts". You can see an example of how to add a custom search engine for YouTube, which means that you'll be able to search on YouTube simply by typing into your URL bar after you hit the shortcut "yt" while the focus is on the URL bar.

<figure>
<img src="/images/firefox-custom-search-engines/how-to-add-firefox-custom-search-engine.png" alt="How to add Firefox custom search engine?">
</figure>
