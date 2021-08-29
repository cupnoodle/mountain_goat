---
layout: post
title: How to truncate long text and show read more / less button
date: 2021-08-26 18:43 +0800
---

If you are working on an app that has user generated content (like blog posts, comments etc), there might be scenario where it is too long to display and it would make sense to truncate them and put a "read more" button below it.

This post will solely focus on the front end, which the web browser will detect if the text needs to be truncated, and only show read more / read less as needed. (dont show "read more" if the text is short enough)

TL;DR? Check out the [Demo at CodePen](https://codepen.io/cupnoodle/pen/powvObw).
## Truncate text using CSS

Referenced from [TailwindCSS/line-clamp plugin](https://github.com/tailwindlabs/tailwindcss-line-clamp).

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

Now that we managed to truncate the text, the next step is to check whether a text is truncated or not, as you can see, the short text above (second paragraph) is not truncated even if we have set `webkit-line-clamp` for it. 

We want to check if the text is actually truncated so that we can show "read more" button for it (we dont need to show read more for short text that is not truncated).

## Check if a text is truncated using offsetHeight and scrollHeight

There's two attributes for HTML elements which we can use to check if the text is truncated, [offsetHeight](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/offsetHeight) and [scrollHeight](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollHeight). From Mozilla Dev Documentation,

**offsetHeight** :
> is a measurement in pixels of the element's CSS height, including any borders, padding, and horizontal scrollbars (if rendered). 

**scrollHeight** :
> is equal to the minimum height the element would require in order to fit all the content in the viewport without using a vertical scrollbar.

Here's [a good answer](https://stackoverflow.com/a/22675563/1901264) from StackOverflow on what is offsetHeight and scrollHeight. Here's the visualized summary : 

![heights](https://rubyyagi.s3.amazonaws.com/25-how-to-truncate-long-text-and-show-read-more-less-button/heights.png)

scrollHeight is the total scrollable content height, and offsetHeight is the visible height on the screen. For an overflow view, the scrollHeight is larger than offsetHeight.

We can deduce that if the scrollHeight is larger than the offsetHeight, then the element is truncated.

Here's the javascript code to check if an element is truncated :

```javascript
var element = document.querySelector('p');

if (element.offsetHeight < element.scrollHeight ||
    element.offsetWidth < element.scrollWidth) {
    // your element has overflow and truncated
    // show read more / read less button
} else {
    // your element doesn't overflow (not truncated)
}

```

This code should be run after the paragraph element is rendered on the screen, you can put this in a `<script>` tag right before the body closing tag `</body>`.

## Toggle truncation

Assuming you already have the `.line-clamp-4` CSS class ready, we can toggle truncation by adding / removing this class on the paragraph element, the code for this can be put into `onclick` action of the read more / read less button: 
```html
<button onclick="document.querySelector('p').classList.remove('line-clamp-4')">Read more...</button>

<button onclick="document.querySelector('p').classList.add('line-clamp-4')">Read less...</button>

```

## Demo
I have made a demo for this using Alpine.js and TailwindCSS, you can check the demo at CodePen here : [https://codepen.io/cupnoodle/pen/powvObw](https://codepen.io/cupnoodle/pen/powvObw). Feel free to use this in your project.

<script async data-uid="4776ba93ea" src="https://rubyyagi.ck.page/4776ba93ea/index.js"></script>




