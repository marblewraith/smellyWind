# Tailwind: Over Hyped and Oddly focused

## UPDATE 2024/09/12

I just found [pandaCSS](https://panda-css.com/) and i think it might be my new favorite!

## UPDATE 2024/03/19

It's been over a year since smellyWind was published. In that time:

- [styleX](https://github.com/facebook/stylex) from meta was published
- The [material UI team is creating their own zero-runtime CSS-in-JS lib](https://twitter.com/olivtassinari/status/1765719897152127315) retaining the name [Pigment CSS](https://github.com/mui/material-ui/tree/master/packages/pigment-css-react)

Neither of these solutions resemble anything close to Tailwind or its "composible" philosophy. They are both zero-runtime CSS-in-JS libs that are closer in behaviour and syntax to the others that were already mentioned in smellyWind:

- [Vanilla Extract](https://vanilla-extract.style/)
- [ecsstatic](https://www.ecsstatic.dev/)

## Table of Contents

- [TL;DR Acceptable Tailwind Use Case?](#tldr-acceptable-tailwind-use-case)
- [Styling](#styling)
  - [Common](#common)
  - [Dynamic Style Composition (DSC)](#dynamic-style-composition-dsc)
  - [Tailwind](#tailwind)
- [Component Paradigm](#component-paradigm)
  - [React](#react)
  - [Component SFC formats](#component-sfc-formats)
- [Tailwind Marketing](#tailwind-marketing)
  - [Alleged Benefits](#alleged-benefits)
  - [Why not just use inline styles?](#why-not-just-use-inline-styles)
  - [Maintainability concerns](#maintainability-concerns)
- [Conclusion](#conclusion)

## TL;DR Acceptable Tailwind Use Case?

I am not a proponent of Tailwind. Quite the reverse, and have been for a long time for reasons that we'll get into. But for the sake of people wanting the juicy low-down:

> *"Is there a TL;DR for acceptable ways to use Tailwind?"*

***Yes.***

1. If you want to colocate your styles inside the components of a framework that does *not* have a single file component (SFC) format available (React, Angular, solidJS, etc). Tailwind is one of the more hacky solutions that facilitates this.

2. The other use case being, if you need to prototype something, but know absolutely nothing about design, and need some prebuilt design system, that already has curated colors, etc.

Typically CSS is the implementation of a design system, for the OOP programmers; design = class, CSS = object. Tailwinds "composable" philsophy presents the opportunity to integrate the design system into CSS more directly and by doing so, lets you "compose *design*" on the fly (iterate prototypes).

With that out of the way, let's dive in an examine Tailwind from first principles.

## Styling

Tailwind or not, it can be said CSS augmentations are generally designed to enhance the *UX of devs* (DevX) when reading / writing code (real-time), and organizating it for access and maintenance (future). Which all translates to predictable behaviour, thus reliablity.

No matter what kind of augmentation (library, framework, paradigm, convention) you use, by the time you run it through uglification / minification build processes, with a little massaging, it should come out pretty similar by the end.

So. You're coding along, let's say you want to create a sphere, what approaches are there?...

### Common

Aggregate everything under a class `.sphere`. Perhaps if some styles like a particular color are frequently reused in multiple places and/or change with difference instances of the element, you may refactor those **certain** styles out into utility classes, (`.sphere .red`) for DRY code.

```CSS
.sphere {
  --size: 100px;
  position: absolute;
  top: calc(50% - (var(--size) / 2));
  left: calc(50% - (var(--size) / 2));
  border-radius: 50%;
  width: var(--size);
  height: var(--size);
}

.red {
  background: radial-gradient(circle at 65% 15%, white 1px, pink 3%, red 60%, maroon 100%);
}
```

Implementing it would look something like this:

``` HTML
<div class="red sphere"></div>
<!-- or -->
<div class="sphere red"></div>
```

Short, readable, concise. Tells you exactly what it is.

### Dynamic Style Composition (DSC)

But lets say you want to retain full ability to *dynamically compose* styles in the markup itself while also being maximally DRY, how would you do it?

```CSS
.shape {
  border-radius: var(--rad);
  width: var(--w);
  height: var(--h);
}

.color {
  background: radial-gradient(circle at 65% 15%, white 1px, var(--c1) 3%, var(--c2) 60%, var(--c3) 100%);
}

.position {
  position: absolute;
  top: calc(var(--t) - (var(--size) / 2));
  left: calc(var(--l) - (var(--size) / 2));
}
```

With the above styles, to implement a sphere in HTML / CSS looks like:

```HTML
<div  id="sphere1" class="shape color position" style="--rad:50%; --w:100px; --h:100px; --c1:pink; --c2:red; --c3:maroon; --t:50%; --l:50%"></div>
```

You could parameterize some more in the style definitions, like using the longhand of border radius for individual corners. Could also be written better, perhaps do some clever things with calc so you only have to specify one HSL color value. But this much already proves the point... *It sucks!*

**1. Contiguity**: Custom properties are separate from the associated classes. Not great for DevX because there's 2 things in different attributes (`class`, `style`) even tho' they're dependent on each other.

**2. Complexity**: Hopefully your editor LSP and Linter can account for the above? That way you can write left-to-right, do all the classes, then all the styles, rather than jumping back and forth. If not, either someone would have to make the tooling work and/or you'd just have to "know the code" (increased mental overhead).

**3. Convolution**: It's also not great because it lacks descriptive detail, for example if you didn't need any unique JS hook and `id` was removed like this:

```HTML
<div class="shape color position" style="--rad:50%; --w:100px; --h:100px; --c1:pink; --c2:red; --c3:maroon; --t:50%; --l:50%"></div>
```

This is *bad code* and developer etiquette. Why? There is not way to tell *what* shape the div is producing without seeing the output (in browser), and/or being forced to read significant amounts of syntax. Perhaps the above is still somewhat agreeable to you? But this is only a simple example, imagine *even more* classes and custom properties... it quickly faces issues of scale.

Well written source code should be obvious in what it does, its should scream at you "this is my purpose!". Not always possible for some low level functions / declarative languages, but that's also why *naming* comes in handy. The obvious question: How important are naming abilities variables / functions give to programmers?

*Hugely important*. To the extent we added variables to CSS, first via preprocessors, then natively (custom properties) and the equivalent of lexical scope with css modules and container queries, name schemes (BEM, ITCSS, SMACSS) being an earlier hack to artificially impose scope (namespaces).

I digress...

What's the specific effect of the above? It makes it difficult to "opt out early", that is, read the minimal amount to understand the whole (what it's doing), with the option to drill for detail (where, how, and why).

The common way (`<div class="sphere red"></div>`) tells you *exactly* what the div is producing, encapsulating the identifiable detail in the name.

### Tailwind

[The homepage](https://Tailwindcss.com/) says the following:

> A utility-first CSS framework packed with classes like `flex`, `pt-4`, `text-center` and `rotate-90` that can be composed to build any design, directly in your markup.

The "can be composed", is synonymous with the composition in DSC we just went through. That is, Tailwind is trying to be a better version of it. Time to compare and contrast... and whoops. Tailwind has no native classes to handle radial gradients. No problem i guess we can [make some](https://blog.logrocket.com/guide-adding-gradients-tailwind-css/#adding-radial-background-gradients), but spoiler, this is a blight on colocation.

```HTML
<!-- DSC -->
<div class="shape color position" style="--rad:50%; --w:100px; --h:100px; --c1:pink; --c2:red; --c3:maroon; --t:50%; --l:50%"></div>
<!-- Tailwind -->
<div class="rounded-full w-100px h-100px absolute top-1/2 left-1/2 bg-gradient-radial from-rose-100 via-rose-700 to-maroon-800">
```

**1. Contiguity** has been mostly resolved, hyphenated property-values / descriptive utility class syntax, it's all under 1 attribute and grouped appropriately, thus the DevX *when writing* Tailwind is an improvement... In favor of making everything else worse.

![MalcolmSmirk](https://user-images.githubusercontent.com/45786628/216475991-55a33c55-86bc-4fe7-8623-4b9d34dc8b09.png)

When *reading* Tailwind, talking about efficiency in trying grok what it's doing, and the opt-out early principle i mentioned previously, neither of them fair well as you can see with this...

![Tailwind class hell image1](https://www.aleksandrhovhannisyan.com/assets/images/SlYrQIW6RE-1261.webp)

... all that orange, 71 classes just to style a checkbox. Full credit for this screenshot to [Aleksandr Hovhannisyan's article](https://www.aleksandrhovhannisyan.com/blog/why-i-dont-like-Tailwind-css/), for further reading of Tailwind's shortcomings.

And for those saying it's not a realistic use case, as stated in that article, that screengrab was "taken from netlify's dashboard", and is entirely consistent with tailwinds "composable" philosophy. Another one:

https://twitter.com/hi__mayank/status/1584261503670448128

It's so bad to the point people have made [editor plugins](https://marketplace.visualstudio.com/items?itemName=moalamri.inline-fold) to hide the, quote: "utility classes that often disfigure code visual structure".

**2. Complexity** has been increased. Tailwind is effectively a "custom grammar". That grammar must be defined so that LSPs and linters understand it. Not to mention you also have to make sure it works flawlessly embedded inside other languages / frameworks, and they're all updated in sync with Tailwind itself. This is all added complexity, dependencies, and potential for failure in the dev workflow. Is this a problem? Well...

https://twitter.com/gyfchong/status/1587038177352523777

I suspect this would be quite frustrating when trying migrate code from one project to another. It's true Tailwind is not solely responsible for this, but they do hold a share of blame as the proponent of their product / why we should use it over anything else.

**3. Convolution** has been increased. Without looking at output (in browser) you *must* read all the Tailwind classes to grok what it's doing (no "opt out early"), and it can get quite verbose as we've seen. Also the Tailwind syntax (mental overhead mapping Tailwind to CSS) that cannot be ignored despite their best efforts to use mnemonics.

> *"But what do you mean "without looking at the browser"? I always have a browser window open..."*

If you are fortunate enough to have extra screen real estate available, and you can use all the "dev toys" (live reload, HMR, etc) maybe this isn't as much of big deal to you.

However in cases with more limited screen real estate (just a laptop / tablet), being able to easily understand the code itself without having to render, is an advantage in my book. At the very least you're saved from switching between the code editor and the browser window via `alt+tab` all the time (DevX benefit).

## Component Paradigm

> *"Aha! Gotcha! You've been talking about Vanilla HTML all this time. But what kinda pleb n00b devs do you take us for? Everyone's coding with some framework using the component paradigm right? Surely Tailwind prevails there!..."*

Not really. The DevX issues mentioned will persist, no matter where you use Tailwind. But there are indeed some mitigating offests when used within component framworks that make the obnoxious problems more tolerable, let's reset.

You're coding along, you want to create and style a sphere *component in React*, what approaches are there?

### React

Well you could use inline styles inside your jsx/tsx, but implementing CSS-in-JS comes with it's own set of problems. If you want to read about those, there's a great [article by Sam Magura](https://dev.to/srmagura/why-were-breaking-up-wiht-css-in-js-4g9b).

And so you got 2 options at this point:

- Write CSS-in-JS for source files, but you need some kind of build step that extracts styles from source into an external file, and placeholds where they were with refs (class / id attributes). There are a few libs that do this, however they too can face [issues with compatibility](https://github.com/withastro/astro/issues/4432) in embedded formats and so my choice would be [Vanilla Extract](https://vanilla-extract.style/), with [ecsstatic](https://www.ecsstatic.dev/) being a close second.

- Structure the component as you would a typical HTML file, that is, separate the style definitions into an external CSS file and use class refs to point to them, which are parsed independently in the browser, and applied once the component is rendered. CSS modules can be used with great effect here.

Tailwind operates closer to the latter, you're not doing CSS-in-JS, you're composing mnemonic class name refs that point to existing and/or generates the necessary styles (based on the names used).

Question: What's the most "painful" part about that workflow?... Or asked another way: What does Tailwind enable you to do, that couldn't be done by just using regular CSS (or scss, postcss, etc)?

**Colocation.**

Tailwind effectively colocates both the *style definition* (i.e. the properties + values) inside the *style implementation* (where the styles are being invoked), as the class names are effectively CSS declarations in and of themselves. With that, there is no external CSS file to create / switch between / manage for each component file.

Granted you still have shared globals and styles (in `Tailwind.config.js`) to account for, but these shouldn't change frequently to be much of an imposition. And so, people argue the trade-off of learning the Tailwind syntax (abstraction of an entire language) is worth being able to sit in a single file (most of the time) and be able to do everything, is comfortable in terms of DevX.

This is why there's such contention around the Tailwind, because *comfort* can have a strong subjective motif.

**1. Contiguity** and **2. Complixity** issues are still present.

**3. Convolution** however has been reduced if not eliminated. Why? Because React components themselves provide surrounding context. That is, rather than html being designated by `class="sphere red"` or an id, instead *the component itself* would be named appropriately like `sphere.jsx`, or perhaps `shape.jsx` if you wanted to pass `sphere` in via props. This establishes scope / context, making Tailwind fit for use.

### Component SFC formats

> *"Yeah! See we told you Tailwinds good with component frameworks, so there!"*

Not so fast. Other component frameworks / libs that exist would have come across the same issue, so how did they solve it?

For Svelte and Vue, they baked colocation into their single file component (SFC) formats, that is, the "extraction" build step i mentioned above when talking about Vanilla Extract; Svelte and Vue have this capability integrated, no need for 3rd party libs.

**1. Contiguity** is slightly inferior to Tailwind in JSX/TSX. Separate sections for style definiton vs implementation, but this shouldn't be hard to solve with standard editor features (`Goto Symbol`), and the mandatory plugin / extension that would be needed for syntax highlighting / formatting anyway. Also contiguity isn't actually a problem if you abandon the whole composable style paradigm, because it means you won't have that many classes to contend with anyway.

**2. Complexity** is still increased, but less so by comparison. There is still a "custom grammar" in the sense that native styling (css, scss, etc) is embedded in the SFC format, however existing work on LSP's / linters can be more easily leveraged, without having to take into account the names of classes. Also it means changes to the CSS spec can be usable more quickly without a lib such as tailwind "in the way".

**3. Convolution**... Again if you ditch the composable style paradigm, possible (because the SFC formats took care of colocation) convolution is a non-issue since you can (and should) name your classes. We'll address the Tailwind arguments about naming ("premature abstraction") in a bit.

> *"So wait a minute, with frameworks that aren't React, you said if we don't use the composition paradigm for styling, the code should be much more readable? Doesn't this mean since Tailwind is based on the idea of composable styling that it's fundamentally flawed?"*

Yes! 😂

Or rather, Tailwind seems oddly focused at targeting a shortcoming that's present in *only* in frameworks that do not treat styling as a first class tennant of web dev... which is rather stupid considering how much of a role it plays, and makes it deeply ironic that front-end devs would be singing praises about it 🤣

We could further get into ripping apart the philosophy behind Tailwind and the mistakes made there, but that would extend this already lengthy article even further. If there's enough interest ill consider writing some more.

## Tailwind Marketing

Alright, with this understanding so far. Let's take a look at the claims Tailwind uses to justify its existence. Reading through the "Core Concepts" sidebar starting from [Utility-First Fundamentals](https://Tailwindcss.com/docs/utility-first) (v3.2.1 of docs at time of writing):

![wannaSell](https://user-images.githubusercontent.com/45786628/216475142-e2418ab0-ce7f-4684-a874-ab80a40d0b83.gif)

### Alleged Benefits

> [1] You aren't wasting energy inventing class names. No more adding silly class names like `sidebar-inner-wrapper` just to be able to style something, and no more agonizing over the perfect abstract name for something that's really just a flex container.

To put this in context. We've accepted component based architecture is basically "the way" to do things. Hundreds of frameworks and libs and even new native features features (CSS modules / container queries) to support its use further.

Not to mention tailwind themselves acknowlegde such architecture, [suggesting it](https://tailwindcss.com/docs/reusing-styles#extracting-components-and-partials) as a workaround to the fact tailwind is WET in markup... and you have to *name* those components / files.

Even if we discount CSS modules / scoped CSS of SFC files, and used the old way of BEM namespacing. All you have to do is use the component name (which you *must* have figured out already) as the "block" to namespace the styles. After that you should only encounter naming conflicts locally (in the same file).

If there are so many styles in the same component, such that you can make a duplicate naming error, i'd say you have bigger problems (i.e. component is too big / should be refactored).

But they're still saying: "no no, naming is hard and takes alot of effort?"... Huuuuh? 🤨

Not to mention, while you may not be "wasting energy inventing class names", instead you'll be wasting energy learning the pre-named class mnemonics and architecture of Tailwind.

🦕💩 score = 0 : 1

---

> [2] Your CSS stops growing. Using a traditional approach, your CSS files get bigger every time you add a new feature. With utilities, everything is reusable so you rarely need to write new CSS.

...But it doesn't matter?

During dev we're normally splitting styles into arbitrary files anyway, and the length of any single CSS or component file is flexible.

Furthermore even if there were duplications in the final output, to borrow an argument Tailwind themselves use, who cares? We have CSS nano and other optimizers, that would take care of it for prod.

Also in terms of load performance for prod, it turns out some duplication is fine. As demonstrated by [Harry Roberts: Extends vs Mixins](https://csswizardry.com/2016/02/mixins-better-for-performance/). In SCSS `mixins` produce more classes overall than `extends` does. Yet by the time compression (gzip / brotli) gets done with it, notice the filesize?

🦕💩 score = 0 : 2

---

> [3] Making changes feels safer. CSS is global and you never know what you're breaking when you make a change. Classes in your HTML are local, so you can change them without worrying about something else breaking.

False... do i actually have to explain how the cascade works? If you remove specificity (for the sake of brevity) CSS still has scope depending on *where* it's defined. Also no recognition or distinction made between CSS in dev vs CSS in prod?

CSS in dev isn't necessarily global, [CSS modules are a thing](https://github.com/css-modules/css-modules) and locally scoped css in component frameworks has been possible for some time now ([Vue example](https://vuejs.org/api/sfc-css-features.html#scoped-css)).

Furthermore even if the component framework does *not* support locally scoped styles, there are better CSS-in-JS solutions mentioned before: [Vanilla Extract](https://vanilla-extract.style/) that don't require pre-named class mnemonics or conventions.

🦕💩 score = 0 : 3

### Why not just use inline styles?

> [4] Designing with constraints. Using inline styles, every value is a magic number. With utilities, you're choosing styles from a predefined design system, which makes it much easier to build visually consistent UIs.

Those magic numbers didn't disappear in Tailwind, they just moved. Now they're in `Tailwind.config.js`, which is no different to what other CSS frameworks (Boostrap, Foundation, Bourbon, Bulma, etc) had done before it. Defining the design system via overridable global defaults.

The only difference being Tailwind added more dynamism for processing. But it's not like we haven't seen that before either. Notice [bootstraps grid system](https://getbootstrap.com/docs/5.2/layout/grid/) and how you can specify a number on the end of the class name prefix to determine the number of columns? What's that?... Dare i say it... is it composable?! 🤯

Credit where it's due, out of all the CSS frameworks i've seen, Tailwind's means of [using "arbitrary properties"](https://Tailwindcss.com/docs/adding-custom-styles#arbitrary-properties), that is the dynamism for processing above for overrides, is the best i've seen in terms of DevX.

For this i'll grant half a point because it's also worth bearing in mind, this is a double edged sword. Ease of overriding also means it becomes easy for devs (particularly inexperienced ones) to stray from design systems and do anything they want.

🦕💩 score = 0.5 : 4

---

> [5] Responsive design. You can't use media queries in inline styles, but you can use Tailwind's responsive utilities to build fully responsive interfaces easily.

True. But again this is only relevant for frameworks without SFC formats and/or if you're not using one of the CSS-in-JS libs mentioned.

So i guess it's another half point? Because if you have those features at your disposal, you're free to define styles inside the component using native syntax, that includes media queries. You can even mimic similar behaviour in native HTML if you're using [progressive rendering techniques](https://csswizardry.com/2018/11/css-and-network-performance/) by specifying the media attribute in the link tag used in the body. Tho' the markup syntax gets "tailwind ugly", very polluted.

Also I don't consider media queries something that needs to be dynamically composed anyway? Using proper responsive / fluid design patterns means the number of media queries should be kept to a minimum in the first place. Can't speak for everyone, but by doing this i find i'm consistently using *all* queries that have been defined anyway, without a need to tweak any of them.

🦕💩 score = 1 : 5

### Maintainability concerns

[Extracting components and partials](https://tailwindcss.com/docs/reusing-styles#extracting-components-and-partials)

> [6 (paraphrased)] Utility classes are WET / can be extremley verbose. You can just use component frameworks to make it DRY.

I find this a bit hypocritical. Going on and on about CSS being global / scary before, and why Tailwind solves all the associated problems. Then in the next breath? Oh just use component frameworks... which provide you with scoped css for components anyway?

This failure of a sidestep doesn't solve one of the actual issues... as stated above, 71 classes to style a checkbox.

Of course people that know Tailwind will say: "oh just use the other tailwind features in the cases where class names get too lengthy"... So we're supposed to create an even stronger link to a 3rd party dependency (tailwind) to be able to use arbitrary naming (via @apply, etc) when we would've had that capability anyway without it?

According to them apparently yes 😑 . And then doubling down they say naming is a  ["Premature Abstraction"](https://Tailwindcss.com/docs/reusing-styles#avoiding-premature-abstraction) underneath, citing the same points we've already been through:

- You have to think up class names all the time
- You have to jump between multiple files to make changes
- Changing styles is scarier (I'm not making this up, they actually say "scarier" 🤣)
- Your CSS bundle will be bigger

The irony being Tailwind utility classes themselves are *still* an abstraction.

That is, we've gone from encapsulating styling via conceptual abstraction, to abstracting *an entire language* with Tailwind, to avoid having to name stuff early... which in most cases we'd be naming anyway (component file names, JS hooks, etc).

Trade off isn't worth it IMO.

🦕💩 score = 1 : 6

---

[Using editor and language features](https://tailwindcss.com/docs/reusing-styles#using-editor-and-language-features)

> [7 (paraphrased)] Utility classes are WET / can be extremley verbose. Doesn't matter, editor features will compensate.

***Sorry what?!*** Since when did we lose our pride as devs? One liners aren't a thing anymore? OK i'm being hyperbolic 😁

But seriously, Being able to write something that presents as short, concise, and easy to grok in the fewest amount of chars, that's not important?

Someone will have to *read* that code at some point! Be polite to your fellow devs and/or yourself in 6 months.

Not to mention relying on editor features as a crutch, instead of writing reasonably DRY code from the outset to make it easy to edit and refactor... Seems a bit ends justifying the means.

Also maybe it's just me, but the example they use, the code sample is a bit too "mint". The places that require multiple cursors are all right next to each other, in reality using multicursors are way more janky than that, not so quick and simple, and even find/replace comes with its own caveats.

🦕💩 score = 1 : 7

## Conclusion

Does tailwinds claims hold up? No, they're over 80% dino-shit.

![jurassic park image](https://user-images.githubusercontent.com/45786628/216474017-769141a2-9581-4637-81e2-b9e76f32719a.gif)

Is it worth using Tailwind? ([See TL;DR Acceptable Tailwind Use Case?](#tldr-acceptable-tailwind-use-case)).

Is it worth prioritizing learning Tailwind? Not unless you're working on something that has it as a dependency already. It has a limited scenario's where it's useful (Angluar / React) and for those cases there are better alternatives (Vanilla Extract, ecsstatic, etc). Furthermore it's not a very transferrable skill, that is, not every project's gonna have tailwind.

We spent all this time getting to the point of componentizing websites, only to tie another 3rd party dependency (tailwind) to it so the code can't work natively if you move it to another project without tailwind? I thought the whole point of components was to DRY things out, what's the point if you're going to make things WET over multiple projects? 😑

Note: I'm speaking to new devs with this next paragraph, experienced devs you've probably heard / seen this.

Short of JSX / Tailwind being adopted by the W3C itself and becoming a web standard (unlikely), learning and using it now is akin to implementing tech debt. Short term DevX gains, for long term pain when it comes to support / migration (that will inevitably need to happen), just as it did with Jquery, Bootstrap, momentJS, Grunt, Gulp, Webpack or whatever other dated libs / tools / frameworks you can think of.

Is tailwind capitalizing on the flaws of frameworks that didn't consider styling a first class citizen?

Speculation on my part here, but i'm gonna say yes. While i do tip my hat 🎩 🧐 to their criminal genius, when i look at devs enamoured and frothing at the mouth claiming Tailwind is the best...

https://user-images.githubusercontent.com/45786628/216472537-c4c96d5c-04a9-4138-8983-80f19bcd5b61.mp4


