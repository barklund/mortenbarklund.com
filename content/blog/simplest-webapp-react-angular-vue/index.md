---
title: "The simplest possible webapp implemented in React, Angular, and Vue"
date: 2020-08-31T00:00:00-04:00
publishDate: 2020-08-31T00:00:00-04:00
weight: 150
covercaption: "We all have to start small before we go exploring the huge and scary world of frontend"
coveralt: "A puppy looking up"
tags: ["webapp", "react", "vue", "angular"]
---

Are you new to frontend development and feel that all the popular libraries and frameworks are too daunting to get started with? Do you always balk at tutorials, when they mention installing a bunch of programs and libraries and running scripts, compilers, and bundlers when trying to learn something new?

Or are you an experienced developer, but just wish that you could test out something in a framework without installing all sorts of complex machinery? Just develop something like in the good old days - an HTML file, a script, a webserver, and that's it?

It's your lucky day, both of you, because I've got you covered! Inspired by a recent post by [Kent C. Dodds](https://twitter.com/kentcdodds/) on [a super simple React starter setup](https://kentcdodds.com/blog/super-simple-start-to-react/), I expanded the notion of the super simple webapp setup to also include Angular and Vue.

Thus I can present to you all, the **Simplest possible webapp in React, Angular, and Vue**!

In all three instances, the entire app is just an HTML file and a script file. Simply serve both files statically over a webserver and your app will compile and run right in the browser. No need for any tooling, no scripts running in the background, no installing packages and versions, no fighting over incompatibilities!

**NB**: Sorry, the puppy above is only for grabbing your attention - there'll be no more puppies.

---

## Show me!

You can download the necessary files from the Github repository, [`barklund/simple-webapp`](https://github.com/barklund/simple-webapp). And even better, you can experience it right in your own browser:

* [React](https://barklund.github.io/simple-webapp/public/react/) - a full app in just two files
* [Angular](https://barklund.github.io/simple-webapp/public/angular/) - a full app in just two files
* [Vue](https://barklund.github.io/simple-webapp/public/vue/) - a full app in just two files

A few notes on the sample application implemented in all 3 languages:

1. It uses multiple components
1. It has and updates internal state
1. It uses the language's own template format
1. It is probably the minimal possible application doing all the above

If you have suggestions on how to simplify or optimize any of the apps even further or how mistakes or bad patterns can be fixed, please [ping me on Twitter](https://twitter.com/barklund)! However bear in mind, that transparency and understanding are key design goals here.

## How do I use it?

You download the two files relevant for your desired flavor of webapp and serve them over a local webserver. That's it!

The _"serve them over a local webserver"_ part might sound tricky, but it's not that hard. If you have no idea how to do this, my suggestion is to download [Mongoose](https://www.cesanta.com/binary.html) (just the free version), copy it to the same folder as the above two files, and start the application. It'll automatically open your favorite browser to the correct page and show you the app running. Then just edit the HTML or script file as you see fit, reload the browser and voilá - you've just developed something!

If you're an experienced developer, you probably have a webserver of choice in your arsenal - you could for instance simply run `python -m http.server` in the relevant folder. Or `npm i -G http-server && http run http-server .` if you prefer NodeJS.

### How to get started with React

1. Download [`react/index.html`](https://raw.githubusercontent.com/barklund/simple-webapp/main/public/react/index.html) and [`react/script.js`](https://raw.githubusercontent.com/barklund/simple-webapp/main/public/react/script.js)
2. Start a webserver in the folder where you downloaded the above two files

That's it! 2 steps! That beats most (if not all) other guides, that require 10+ steps (and often even more than that if you don't have the correct versions of required software).

With this setup, you can get started trying out all basic React features in 1 minute flat!

### How to get started with Angular

1. Download [`angular/index.html`](https://raw.githubusercontent.com/barklund/simple-webapp/main/public/angular/index.html) and [`angular/script.ts`](https://raw.githubusercontent.com/barklund/simple-webapp/main/public/angular/script.ts)
2. Start a webserver in the folder where you downloaded the above two files

That's it! 2 steps! See if you can find another Angular guide that'll get you started as quickly - I highly doubt it!

**NB**: Note that the script file has the extension `.ts` rather than `.js`. This is by design, as it is a TypeScript file.

With this setup, you can get started trying out all basic Angular features in a jiffy!

### How to get started with Vue

1. Download [`vue/index.html`](https://raw.githubusercontent.com/barklund/simple-webapp/main/public/vue/index.html) and [`vue/script.js`](https://raw.githubusercontent.com/barklund/simple-webapp/main/public/vue/script.js)
2. Start a webserver in the folder where you downloaded the above two files

That's it! 2 steps! And if you act now, you'll get a 24 piece steak knife set included for free! _(Disclaimer: no actual steak knife set is available, sorry)_

With this setup, you can get started trying out all basic Vue features in two shakes of a lamb's tail!

--- 

## But how does it work?

This works by loading the libraries as UMD files. UMD files are universal modules that can run anywhere - even in the browser!

However, for both React and Angular, the source code has to be compiled, as it is not normally written in regular valid JavaScript. For those, we also need to load some special (UMD) transpilers, that can load the slightly exotic JavaScript flavors (for React it's JSX/Babel, for Angular it's TypeScript) and convert it to fully valid JavaScript, that your browser understands.

React requires just two modules - and a babel transpiler. Angular requires 8 modules to run and 2 more to transpile the source code. Vue is by far the simplest - requiring just a single external file and no transpiling, as the input is already pure JavaScript.

--- 

## Caveats

This of course has a huge number of caveats.

1. First of all, this is only for educational purposes! Never should you ever put one of these apps in production with any expectation of scale or speed. It's crude and inefficient - just as intended.
1. You cannot easily upgrade to a new version nor can you easily add any plugins or libraries - but that's also not the intention.
1. This is slow! The source code is not pre-compiled, thus it has to be compiled every time you open the webapp. For such a tiny app and you as the only user, that's not a problem, but for anything but this, it would be.
1. This is not the recommended way to structure an app. Almost all apps will quickly grow way too big to fit in a single file like this. When you find a framework or library you like and want to do anything even slightly serious in, start a new project in the right way following whatever process said project recommends.
1. You cannot easily extend this to include CSS-in-JS or anything like that. You could fairly simply add images and CSS files in the mix, but of course not anything automagically generated from source.

---

## Wait, there are more puppies!

Not to worry, I did find some more puppies for you:

<figure>
<a href="./puppies.jpg"><img src="./puppies.jpg" alt="7 puppies" /></a>
<caption><em><small>Here are no less than <strong>7 more puppies</strong> for you. And they're all completely exhausted after trying out 3 different webapp frameworks in a few minutes!</small></em></caption>
</figure>

