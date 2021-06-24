---
layout: post
title: How to get started with TailwindCSS
date: 2021-06-24 22:44 +0800
---

# How to get started with TailwindCSS

This article assumes you have heard of [TailwindCSS](https://tailwindcss.com), and interested to try it out but have no idea where to start. 




## Try it out online on Tailwind Play

If you just want to try out TailwindCSS and don't want to install anything on your computer yet, [the official Tailwind Play](https://play.tailwindcss.com) online playground is a good place to start!



## Try it on your HTML file without fiddling with the nodeJS stuff

This section is for when you want to use TailwindCSS on a static website, but you don't want to deal with the nodeJS NPM stuff.




Thankfully, [Beyond Code](https://beyondco.de/blog/tailwind-jit-compiler-via-cdn) has released TailwindCSS JIT via CDN. You can read more on how it works on the linked article here : [https://beyondco.de/blog/tailwind-jit-compiler-via-cdn](https://beyondco.de/blog/tailwind-jit-compiler-via-cdn)



To use TailwindCSS classes on your HTML file, simply include this script in between the `<head>...</head>` tag  :

```html
<!-- Include CDN JavaScript -->
<script src="https://unpkg.com/tailwindcss-jit-cdn"></script>
```



Then you can use TailwindCSS class names on your HTML file and it will work!


Keep in mind that this JIT javascript file is > 375 KB , which is quite sizeable if you want to use it for production. 

> The bundle size is pretty big (375kb gzipped) when compared to a production built CSS that is usually ~10kb. But some people don't mind this.



If you would like to compile a production TailwindCSS, you would have to deal with NPM / nodeJS (I promise it would be painless) , which will go into in the next section.



## Compile and purge TailwindCSS into a .css file for production use

Say you already have the HTML ready with TailwindCSS classes, and now you want to build a production ready CSS file, this is where we need to npm install some stuff and run some npm build script.




If you haven't already, open terminal, and navigate to your project folder root, and **initiate npm** :

```
npm init -y
```

This will create a package.json file in your project folder root.




Next, we are going to install TailwindCSS, PostCSS and AutoPrefixer package : 

```
npm install -D tailwindcss@latest postcss@latest autoprefixer@latest
```



Next, create a PostCSS config file on the project folder root, the file name should be **postcss.config.js** , and add TailwindCSS and Autoprefixer plugin into this config file.

```javascript
// postcss.config.js

module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  }
}
```



If you have custom configuration for TailwindCSS (eg: extension or replacement of default class/font), you can create a Tailwind config file using this command :

```
npx tailwindcss init
```



This will create a  `tailwind.config.js` file at the root of your project folder.



You can customize this file to extend / replace existing settings, let's open it and change the **purge** dictionary to include all html files :

```javascript
// tailwind.config.js
module.exports = {
  purge: [
    './**/*.html'
  ],
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {},
  plugins: [],
}
```


This will tell TailwindCSS to search for all .html and .js files in the project folder for TailwindCSS classes (eg: mx-4 , text-xl etc), and remove classes that did not appear in the .html and .js files, this can cut down the generated production css file later.



Next, create a .css file if you don't have one already, and insert these tailwind directive into it : 

```css
/* style.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* your custom css class/ids here */
.custom-class {
  
}
```



These directives (eg: `@tailwind base`) will be replaced with tailwind styles later on.



Now you can run the following line in terminal to generate the production purged Tailwind CSS file : 

```
NODE_ENV=production npx tailwindcss -i ./style.css  -c ./tailwind.config.js -o ./dist.css
```



This will read the original CSS file (**style.css**), use the config located in the **tailwind.config.js**  file, and save the production CSS file into **dist.css** file (on the project folder root).



Then you can use this production css file in the HTML `<head>` part like this :

```html
<head>
  <!-- production compiled CSS -->
  <link href="dist.css" rel="stylesheet">
</head>
```



It might be tedious to type out the long command above to generate production css, especially if you need to generate it more than one time.



To save some typing, you can move the command above into the "scripts" section of the **package.json** file :

```javascript
// package.json
{
  "name": "your-project-name",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "css": "NODE_ENV=production npx tailwindcss -i ./style.css  -c ./tailwind.config.js -o ./dist.css"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "autoprefixer": "^10.2.6",
    "postcss": "^8.3.5",
    "tailwindcss": "^2.2.2"
  }
}
```



Notice that I have added "**css**" and the command to generate production css into the "**scripts**" section.



Now you can simply type `npm run css` on your terminal to generate the production css! 🙌



I have also created a repo here : [https://github.com/rubyyagi/tailwindcss-generator](https://github.com/rubyyagi/tailwindcss-generator) , which you can simply clone it, replace the repo with your own HTML files, and run `npm run css` to generate production CSS!


## Using TailwindCSS on Ruby on Rails

If you are interested to use TailwindCSS on a Rails app, you can refer [this article on how to install and use TailwindCSS on a Rails project](https://rubyyagi.com/tailwindcss2-rails6/).


<script async data-uid="4776ba93ea" src="https://rubyyagi.ck.page/4776ba93ea/index.js"></script>





