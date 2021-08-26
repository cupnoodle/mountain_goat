---
layout: post
title: How to truncate long text and show read more / less button
date: 2021-08-26 18:43 +0800
---

If you are working on an app that has user generated content (like blog posts, comments etc), there might be scenario where it is too long to display and it would make sense to truncate them and put a "read more" button below it.

This post will solely focus on the front end, which the web browser will detect if the text needs to be truncated, and only show read more / read less as needed. (dont show "read more" if the text is short enough)

## Truncate text using CSS

Referenced from [TailwindCSS/line-clamp plugin](https://github.com/tailwindlabs/tailwindcss-line-clamp)
Using the combination of these CSS properties, we can truncate the text (eg: `<p>...</p>`) to the number of lines we want :
```css
.line-clamp-4 {
  overflow: hidden;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  /* truncate to 4 lines */
  -webkit-line-clamp: 4;
}

```

If the text is longer than 4 lines, it will be truncated and will have ending of "**...**". 
If the text is shorter than 4 lines, no changes is made.

![truncate demo](https://rubyyagi.s3.amazonaws.com/25-how-to-truncate-long-text-and-show-read-more-less-button/truncate_demo.png)

Now that we managed to truncate the text, the next step is to check whether a text is truncated or not, as you can see, the short text above is not truncated even if we have set `webkit-line-clamp` for it. 

We want to check if the text is actually truncated so that we can show "read more" button for it (we dont need to show read more for short text that is not truncated).

## Check if a text is truncated using offsetHeight and scrollHeight

