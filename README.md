 # Writing your own CSS-only Games
# Part 1: Counters

CSS is not a programming language. Not _officially_. But it [is possible to create games with only HTML and CSS](https://www.youtube.com/watch?v=-e5nWsGgZXQ). And there are even some tutorials on how to make a specific game: [a click 'em all](https://medium.com/cssclass-com/how-to-create-pure-css-games-2a29c777bf4), [a jigsaw](https://css-tricks.com/how-i-made-a-pure-css-puzzle-game/), [a maze](https://scriptraccoon.dev/blog/create-css-maze), and maybe more that I have not found.

My aim here is different. In this series of articles, I want to get you to experiment with the different techniques that you can use to create your own games. By the end of each article, you won't have created a game, but you will have understood why some things work and some things don't.

In this article, you'll be looking at the following ideas:

1. Using checkboxes and radio buttons to store state
2. Using animations to change state
3. Using `::before` and `::after` pseudo-elements
4. Using `z-index` to re-order elements
5. Using the `:hover` pseudo-class to intercept the actions of the mouse
6. Writing selectors that control the logic
7. Using custom CSS properties
8. **Working with counters**

My main focus here will be on understanding how **CSS counters work**. You need to use counters, after all, to number things: to show the score or to display a timer, for example. These are simple ways to add urgency and excitement to a game. And if you think laterally, you can also get counters to do some quite unexpected things for you, as you will see.

I won't treat the other seven ideas is such great depth here. I'll be writing other articles about them specifically.

###  Starting with questions

Before you start, you can play a little with the CSS-only activities below. Ask yourself these questions:
- How can I create a countdown timer in CSS?
- How can I calculate a score in CSS?
- How is a counter used for the _letters_ in the word game?
- Really!? I can use _counters_ to animate emojis? How soon can I use that in a project?
- How does a counter know where the mouse is?

If you start with questions, each time you find an answer, you get a little buzz of satisfaction.

These are not all complete games or puzzles. You can think of them as illustrations of the concepts that you will be reading more about below.

![train](images/word.webp)  
[Link](https://merncraft.github.io/word/)

![spin](images/spin.webp)  
[Link](https://merncraft.github.io/spin/)

![map](images/map.webp)  
[Link](https://merncraft.github.io/map/)

You might also have other questions, like:
- How do I detect when the player has won?
- How do I stop the game when the player's time runs out?
- How do I make an animation glitch?
- How do I ensure that whichever button the player clicks last behaves differently from the others?
- Must I solve the word game to find a link to the repo?

 Keep these questions in mind. You'll find answers to them in forthcoming tutorials... unless you can work out the answers for yourself first : )

### What I expect you to know already

I'm assuming that you already know how to create an HTML page and link a CSS file to it. I'm assuming that you already understand how to write CSS selectors, and how to structure [nested CSS](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_nesting/Using_CSS_nesting). Nested CSS has been [available in all major browsers since December 2023](https://caniuse.com/?search=nested%20css), and it's available to over 80% of all users, so it should work in your development browser. It allows me to write shorter CSS code.

If you don't yet know about the recently-introduced `:has()` pseudo-class, don't worry. You'll see plenty of examples on how to use it.




## Learning to Count
### The Origin Story: ordered lists

If you create an `<ol>` ordered list, your browser will automatically number the lines. Try it:
```html
<h1>TO DO LIST</h1>
<ol>
  <li>Buy a tortoise</li>
  <li>Call it 'The Speed of Light'</li>
  <li>Tell people I can run faster than The Speed of Light</li>
</ol>
```
You can change the number that the list starts with by using the `start` attribute in your HTML:
```html
<ol start="10">
  <li>Buy a tortoise</li>
  <li>Call it 'The Speed of Light'</li>
  <li>Tell people I can run faster than The Speed of Light</li>
</ol>
```
![ol start](images/ol.webp)  
Figure 1.

This works because under the hood, the browser uses a  [counter](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_counter_styles/Using_CSS_counters) called `list-item`. You can control this counter through CSS. With the following ruleset, all your `<ol>` elements will start from `101`... even those where you set the `start` property in your HTML file. The `counter` property takes precedence.
```css
ol {
  counter-set: list-item 101;
}
```
### Free for all

Ordered lists don't claim exclusive rights over counters. Browsers make counters available to any element. They give you full control of any kind of numbering process. Here's how you could create a similar (not identical) result using a `<ul>` unordered list.
```html
<ul>
  <li>For the money</li>
  <li>For the show</li>
  <li>To get ready and go cat go</li>
</ul>
```
```css
ul {
  list-style-type: none;
  counter-set: item;

  li {
    counter-increment: item;
  }

  li::before {
    content: "Item " counter(item) ". ";
  }
}
```
The CSS elves see that a `counter` called `item` has been set for the `<ul>` element. By default, they give it the value `0`. They apply the rule `counter-increment: line` each time they meet a new `<li>` item in the list where the `item` counter was set. They then read the current value of `counter(item)` and use this number as the [`content`](https://developer.mozilla.org/en-US/docs/Web/CSS/content) of the [`::before`](https://developer.mozilla.org/en-US/docs/Web/CSS/::before) pseudo-element.

![numbered unordered list](images/go-cat-go.webp)  
Figure 2.

> Note that [you can provide several items](https://developer.mozilla.org/en-US/docs/Web/CSS/content#syntax) in the value for the `content` attribute, like `"Item " counter(item) ". "` and the CSS elves will concatenate them together for you. 

> Note that I use [`counter-set`](https://developer.mozilla.org/en-US/docs/Web/CSS/counter-set) and not `counter-reset`, because the current version of Firefox (128.0.3) does not always handle `counter-reset` as you would expect it to.

### Unpacking how CSS works

Warning: What I show below is _not good HTML_. You are not supposed to use `<b>` tags inside a `<ul>` list, and there's not a real-life case for leaving them empty. But this make-shift HTML is good enough to show you how the CSS elves work.
```html
<b></b>
<ul>
  <b></b>
  <li>For the money</li>
  <b></b>
  <li>For the show</li>
  <b></b>
  <li>To get ready and go cat go</li>
  <b></b>
</ul>
```
In my first example, I gave no initial value for the `item` counter, so it started with the default value of `0`. The CSS elves then found the first list item ("For the money"), and incremented the `item` counter by the default amount: `1`. So when the first list item was shown, the `content` of its `::before` pseudo-element was "Item 1."

In the CSS below, I create a custom `counter` called `item`, with an initial value of `10`. I also use `counter-increment: item 5;` each time an `<li>` item is treated, so the first item starts with an `item` value of 15.

```css
ul {
  list-style-type: none;
  counter-set: item 10;

  li {
    counter-increment: item 5;
  }

  li::before {
    content: "Item " counter(item) ". ";
  }
}

b::before {
  content: "—[" counter(item) "]—";
}
```
Here's what this looks like:

![Numbered items](images/counters.webp)  
Figure 3.

Notice the values of `counter(item)` that are shown in the `<b>` elements.
- The `item` counter is not available to the first `<b>` element (outside the `<ul>` element where `item` is declared, so it shows the default value of `0`.
- The `<b>` element that appears in the HTML page before the first `<li>` item shows the initial value of `10`.
- The other `<b>` elements show the value for `item` that was accumulated by all the `<list>` items that came before it.


### White space matters
Try adding a new line to the `b` ruleset:
```css
  b::before {
    counter-increment: item+1; /* new line */
    content: "—[" counter(item) "]—";
  }
```

You'll see that the CSS elves also now happily add `1` for each `<b>` element they encounter. Notice that there is no space between `item` and `+1`.

![b counters](images/b-counters.webp)  
Figure 4.

But what if you change the `+` to a`-` sign?

```css
counter-increment: item-1; /* new line */
```

The CSS elves get confused. They think that you are referring to a counter named `item-1`. If you'd used `item-one`, it would have had the same effect. You didn't declare a counter called `item-1`, so they ignore it. The result will be just the same as what you saw in Figure 1.

If you add a white space, then everything is clear again.

```css
counter-increment: item -1; /* altered line */
```

> Note that there is no `counter-decrement` property. You have to increment by a negative number. And a space.

Each `b` element decreases `item` by 1, in each and every case, even outside the `<ul>` where the `item` counter is declared. In effect, the CSS elves are very accommodating. You didn't declare a `counter` for this particular element before you started using it? No worries, they will create one for you implicitly. Browers are designed to be forgiving, so you can write sloppy code.

And yes, my code here is deliberately sloppy. I find that you learn more about how to write good code if you deliberately try to break the rules, and see what the effect is. [If you always do things perfectly, you don't find out what the rules are](https://www.youtube.com/watch?v=vKA4w2O61Xo).

And, as before, each `<li>` item increases `item` by 5.

![decrementing a counter](images/counter-1.webp)  
Figure 5.


## Interacting with the User

A web experience cannot be consider to be a game if it has no user interaction. In this section, you'll be seeing how to use checkboxes, radio buttons and sibling combinators ( [`~`](https://developer.mozilla.org/en-US/docs/Web/CSS/Subsequent-sibling_combinator) and [`+`](https://developer.mozilla.org/en-US/docs/Web/CSS/Next-sibling_combinator) ) to change what the CSS is counting, depending on what the user clicks on.

### Not on display

The CSS elves will only count what they can see. You can try to trick them with another sloppy line of HTML:
```html
  <ul>
    <b></b>
    <li>For the money</li>
    <b></b>
    <input type="checkbox"> <!-- Linters don't like this in a <ul> -->
    <li>For the show</li>
    <b></b>
    <li>To get ready and go cat go</li>
    <b></b>
  </ul>
```

And you can add a new ruleset that will hide the `<li>` item immediately after the checkbox when you check it.

(Note that I've also removed the `counter-increment: item -1;` from the `b::before` ruleset, to keep things simple.)
```css
ul {
  list-style-type: none;
  counter-set: item 10;

  li {
    counter-increment: item 5;
  }

  li::before {
    content: "Item " counter(item) ". ";
  }

  /* New ruleset */
  input:checked + li {
    display: none;
  }
}

b::before {
  content: "—[" counter(item) "]—";
}
```

What do you think will happen when you click on the checkbox?

![checked counter](images/checked-counter.webp)  
Figure 6.

If an element is not displayed, then no counter will count it.

However, if you simply hide a list item, like this...

```css
  /* New ruleset */
  input:checked + li {
    visibility: hidden; /* new rule */
  }
```

... then the CSS elves _do_ count it:

![hidden item](images/hidden-item.webp)  
Figure 7.

So here's a trick that you can use: You can set the `display` value of certain HTML elements to `none` if particular checkboxes or radio buttons are `checked`. The CSS elves will not count the elements with `display: none`, so you can display the number of such elements that are visible.

### Time to Play: counting clicks

If all this makes sense to you, the perhaps you'd like to create a really simple activity. (If it doesn't make sense, then you can go back and read the links that I provided for each new concept, and you try to make different mistakes to irritate your browser, until you can work out what rules it really wants you to follow.)

This time, you can leave lists behind, and count checkboxes directly.

You can create an new HTML file and give it these elements:
```html
  <label>
    <input type="checkbox">
    <span></span>
  </label>
  <label>
    <input type="checkbox">
    <span></span>
  </label>
  <label>
    <input type="checkbox">
    <span></span>
  </label>
  
  <p>You have clicked on <span>/</span> circles.</p>
```

Attach a CSS stylesheet with the following rulesets:

```css
body {
  counter-set: total;
}

label span {
  --size: 25vh;
  display: block;
  width: var(--size);
  height: var(--size);
  border-radius: var(--size);
  background-color: gold;
  opacity: 0.1;

  counter-increment: total;
}  

input:checked ~ span {
  opacity: 1;
}

p span::before {
  content: "0"
}

p span::after {
  content: counter(total)
}
```

Before you launch your new web page, do your best to predict what you will see.
- Each `<label>` contains both a checkbox and a `<span>`. This means that the `<span>` is part of the  `<label>`, and a click on this `<span>` inside the  `<label>` will toggle the checkbox.
- The [`~` sibling combinator](https://developer.mozilla.org/en-US/docs/Web/CSS/Subsequent-sibling_combinator) in the selector `input:checked ~ span` selects a span only when a preceding `<input>` sibling has been checked. When a checkbox is checked, the `opacity` of the associated gold circle will be set to `1`. When the checkbox is not checked, the gold circle will appear dim.
- There are different rulesets for the  `<span>` elements inside a `<label>` and for  the `<span>` inside the  `<p>` element. This is because they are used for different purposes.
* The [custom CSS property](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties) `--size` allows you to set one value that is used for the `width`, `height` and `border-radius` of all the `<span>` elements which are inside `<label>` elements. This means that the `<span>`s will be round. 
* There is a `counter` called `total` which starts with a default value of `0`. Each `<label>` element increments this value by the default increment of `1`. There are three `<label>` elements, so the final value of `total` should be 3.
* The `<span>` inside the `<p>` element contains only a `/` slash, but the CSS adds both `::before`  and `::after` pseudo-elements to it.

You should see three pale gold circles, each with a checkbox at its top left, and text at the bottom of your page should say: "You have clicked on 0/3 circles." You can click on the checkboxes and circles as much as you like, but the final sentence will not change.

![three circles](images/3-circles.webp)  
Figure 8.

The value `3` appears because of this rule:
```css
p span::after {
  content: counter(total)
}
```
Can you imagine how to add another `counter` that will count how many `<input>` elements have been checked? Can you imagine how to display  the value of this new `counter` in the `::before` pseudo-element of the `<p>` element?

Can you do it on your own, without reading the solution below?

#### Solution

Here are the lines of CSS that I changed or added in my version, in order to make this feature work:

```css
body {
  counter-set: total clicked;
}

input:checked {
  counter-increment: clicked;
}

p span::before {
  content: counter(clicked)
}
```
![clicked circles](images/clicked-circles.webp)  
Figure 9.

>Note: You might think that declaring each `counter` on a different line might be valid CSS...
```css
body {
  counter-set: total;
  counter-set: clicked;
}
```
>... and it _is_ valid, but it doesn't behave the way you might expect. The second `counter-set` property overwrites the first, so `counter-set: total;` will be ignored.

This is the way CSS works for all other properties. In this ruleset...
```css
label span {
  background-color: gold;
  background-color: silver;
}
```
... the `silver` background will overwrite the `gold`, just as you would expect.

If you want to declare different `counter` properties on different lines, just use a single semi-colon at the end of the multi-line rule:
```css
body {
  counter-set:
    total
    clicked;
}
```
Now when you click on a gold circle, or on its checkbox, the number shown in the `::before` pseudo-element changes.

### One-way clicking

If you click on the same gold circle a second time, the checkbox will be deselected, and the number of "clicked" circles will go down. What is the simplest way to ignore any subsequent clicks?

In a group of radio buttons, only one radio button can be checked at any given time. Clicking on that button again does not uncheck it. If you have a group with only one radio button in it, then you can switch it from unchecked to checked, but you can't toggle it back. Initially, a group of radio button does not need to have any buttons checked. This means that when you use radio buttons, the initial state can have all the buttons unchecked, just like you can do with checkboxes.

To test this, change the `type` of each `input` from `checkbox` to `radio` and refresh your page.
```html
<label>
    <input type="radio">
    <span></span>
  </label>
  <label>
    <input type="radio">
    <span></span>
  </label>
  <label>
    <input type="radio">
    <span></span>
  </label>
  
  <p>You have clicked on <span>/</span> circles.</p>
```
Now only the first click on a gold circle will have any effect, and the number of "clicked" circles cannot decrease.

### Who keeps order?

If you follow my instructions perfectly, your web page should work. But if it works, what exactly have you learned? You have definitely learned to follow instructions, but you probably knew how to do that already.

So what if you deliberately do something different, something disruptive, something that I didn't ask you to do? For example: what would happen if you put the "You have clicked on ... circles" message at the _beginning_ of your HTML page?

And just as a double-check, you can add a new line between the second and the third label, like this:
```html
  <p>You have clicked on <span>/</span> circles.</p> <!-- Now at the top -->
  
  <label>
    <input type="radio">
    <span></span>
  </label>
  <label>
    <input type="radio">
    <span></span>
  </label>
  <p><span>/</span> so far</p> <!-- new line -->
  <label>
    <input type="radio">
    <span></span>
  </label>
```
Suddenly, your message insists "You have clicked on 0/0 circles", no matter how hard you click. The new line you have inserted fails to count the circle in the layer above it, or any clicks on that circle.

But were you really being disruptive? Or were you just following instructions again? I wasn't watching.

> Notice how I use the word "above" in the text above. The third circle is shown on the screen _below_ the other two, and lower down in the HTML file. However, it is _drawn_ in a layer that covers the circles that appear earlier in the HTML file. In the _z-direction_ —from the screen to your eye— it is _above_ the other circles, even if in the y-direction, it is closer to the bottom of your screen.
> 
> You can see what I mean if you add this ruleset at the end of your file:
> ```css
>  label:last-of-type span {
    position: relative;
     top: -10vh;
     border: 10px solid red
   }
> ```
> ![above](images/above.webp)
> Figure 10.
> 
> So yes, in one sense, the bottom circle is "above" the others.

From the test that you did above, you can deduce that **the order in which elements appear _in the HTML file_ affects the order in which they can increment a `counter`**.

But what about the order of rulesets in the CSS stylesheet? What happens if you change the order in which the `counter` properties are set and read, as shown below?
```css
/* Read the value of the counter properties... */
p span::before {
  content: counter(clicked)
}

p span::after {
  content: counter(total)
}

/* ...before you declare the counter properties */
body {
  counter-set:
    total
    clicked;
}

/* The CSS below is unchanged */
label span {
  --size: 25vh;
  display: block;
  width: var(--size);
  height: var(--size);
  border-radius: var(--size);
  background-color: gold;
  opacity: 0.1;

  counter-increment: total;
}

input {
  position: absolute;
  left: -999px;
}

input:checked {
  counter-increment: clicked;
}

input:checked ~ span {
  opacity: 1;
}
```
Does this solve the problem that you just created? No. But if you place a copy of the  element`<p>You have clicked on <span>/</span> circles.</p>` back at the end, everything works fine,  at least in this final position. This _suggests_ that the order in which rulesets appear _in the CSS file_ **does not** affect the order in which a `counter` is incremented.

But don't let this give you a false sense of security. As Mr Weasley said: "Never trust something that can think for itself if you can't see where it keeps its brain".

![first and last](images/first-last.webp)  
Figure 10.

### Checking it twice

Whenever I am following a tutorial to learn a new technique, I do the tutorial twice. The first time, I follow the instructions carefully, so that I can see that the results are what the writer promised. 

> Sometimes, tutorials contain errors — a missing step, a missing line, a missing semi-colon — and sometimes the technology has changed since the tutorial was written, and instructions that used to work no longer do. When I encounter a tutorial like this, I find that it is often worth persisting, and searching online for solutions. The process of searching is where a lot of my learning happens.

The second time I do the tutorial, I try not to read it at all. I do as much as I can from memory, and inevitably I make mistakes because there is something that I did not understand, or did not pay attention to, or assumed that I could ignore. I don't _mean_ to be disruptive. I just allow disruption to happen.

It is the mistakes that I make the second time that show me how much I still have to learn. If I don't make these mistakes, I can deceive myself into thinking that I have understood. Solving my mistakes takes effort (especially if I don't look back at the tutorial). And something that you have made an effort to do is something that you will remember. 

An app that works perfectly the first time is less helpful than an app where I have to struggle to get it to work.
### Traffic Lights, anyone?

This arrangement of three circles, one above the other, might make you think of traffic lights. And you might be tempted now to get distracted and see if you can create a set of interactive traffic light with only CSS. 

And why not? It's fun to play, and play is great for learning.

The sequence of traffic lights is not the same in every country. Here's a CSS-only illustration of the traffic light sequences in France and in the UK.

![lights](images/lights.webp)  
[Link](https://merncraft.github.io/traffic-lights/)

If you want to try this yourself, here's a hint: You'll be placing a checkbox `<input>` and a `<span>` inside a `<label>` for each light, as you have just been doing, but the `<span>` inside (at least) one label will need a `z-index` setting, so that a click on this `<span>` will check a radio button in a lower layer.

But _this_ article is about **counting** and you won't need to count anything with traffic lights. So when you are ready, perhaps you would like to continue?

### Custom CSS properties

You can also use [custom CSS properties](https://developer.mozilla.org/en-US/docs/Web/CSS/--*)  to hold numbers. To create custom CSS property, you use a property name that starts with `--x`, where "x" is any letter.

I've already used a custom CSS property to set the diameter of the circles in the last example:
```css
label span {
  --size: 25vh;
  display: block;
  width: var(--size);
  height: var(--size);
  border-radius: var(--size);
  /* more lines skipped */
}
```
A custom property lets you define a value once, and use it in many places. It helps you follow the DRY principle: Don't Repeat Yourself. If you decided that you wanted circles of a different size, you could simply change the value of `--size`, in just one place, and the `width`, `height` and `border-radius` properties would all adjust their values.

#### My Opinion on the Term ~~CSS Variables~~

Some people use the term `CSS variables` as a synonym for custom properties. But I don't.

In a programming language, a _variable_ can have various values at different times. And so can custom properties. But:

- A custom CSS property is often reset to its initial value, as you will see in a moment
- You can't display the value of a custom CSS property in the `content` property of a `::before` or `::after` element
- A custom CSS property cannot be used recursively, to reset its value to a value calculated from itself, as the next demonstration shows.

In JavaScript, you can do this, and the JavaScript elves know how to handle it:
```javascript
var x = 1
x = x * 2
```
After these two lines of code have been executed, `x` will have the value `2`.

In CSS, you can set a custom property to a different value. This works fine:
```css
label span {
  --size: 25vh;
  --size: 15vh;
  display: block;
  width: var(--size);
  height: var(--size);
  border-radius: var(--size);
  /* more lines skipped */
}
```
But in the snippet below, the second rule causes the CSS elves to panic, as if they see themselves in an [infinity mirror](https://en.wikipedia.org/wiki/Infinity_mirror). They simply stop working for you.
```css
label span {
  --size: 25vh;
  --size: calc(0.6 * var(--size));
  display: block;
  width: var(--size);
  height: var(--size);
  border-radius: var(--size);
  /* more lines skipped */
}
```
Try both of these changes in the clicking-on-circles exercise that you have just done. 

Custom CSS properties do not behave like variables do in other languages. So _I_ always say "custom CSS ***properties***", to make this distinction clear. But if you call them "CSS variables", I won't say anything. I'll just sigh.

### Setting a starting value for a counter

To explore how custom CSS properties can interact with counters, create a new HTML file with the following body:
```html
  <b></b>
  <p>Tinker</p>
  <p>Tailor</p>
  <p>Soldier</p>
  <p>Sailor</p>
  <hr>
  <label>
    Add 10: <input id="add10" type="checkbox">
  </label>
  <label>
    Add 100: <input id="add100" type="checkbox">
  </label>
  <label>
    Add 1000: <input id="add1000" type="checkbox">
  </label>
```
Here's a challenge: Before you read on, can you use everything you have learned so far about CSS counters to write the CSS that will make your page look like this?

![property layout](images/property-layout.webp)
Figure 11.  

**DON'T READ ANY FURTHER UNTIL YOU HAVE WRITTEN YOUR OWN CSS.**

Are you ready now? Or are you being disruptive, like I suggested you should be?

Did you write something like this?
```css
body {
  counter-set: item 
}

b::before {
  content: "—[" counter(item) "]—";
}

p {
  counter-increment: item;
}

p::before {
  content: counter(item) ": ";
}

label {
  display: block;
}
```
I imagine that:
- You chose a different name for your counter
- You might have given it an initial value and an explicit increment value
- Your rulesets might be in a different order.

That's good. It means that you won't be able to simply copy and paste the CSS that I provide below. You'll have to adapt it to work with your property names. You'll have to make an effort : )

### Activating the inputs

To activate the checkboxes, can you do these three steps?
1. Add a custom CSS property to the ruleset for the `body` selector, and set its value to `0`. Below, I've used the name `--start`, but you can use whatever name you like.
2. Use your custom CSS property as the initial value for your counter.
```css
/* Create a CSS property and use it to initialize the counter */
body {
  --start: 0;
  counter-reset: item var(--start);
}
```
3. Add three new rulesets that will change the value of your custom CSS property.
  **Note that I have placed ruleset for the  the `#add1000`input first. Please do the same, even if you are feeling disruptive. You'll see why in a moment.**
```css
/* Use the checkboxes to change the value of --start */
body:has(#add1000:checked)  {
  --start: 1000
}

body:has(#add10:checked)  {
  --start: 10
}

body:has(#add100:checked)  {
  --start: 100
}
```
Notice that I have wrapped  the reference to the `<input>` elements with the new [`:has(...)` pseudo-class](https://developer.mozilla.org/en-US/docs/Web/CSS/:has). Basically, this says: "if the `body` has a checked input, then set the value of `--start` for the `body` and all its children". 

What would happen if you don't use the `:has()` pseudo-class, like this?
```css
#add100:checked  {
  --start: 100
}
```
The rule above would say "set the value of `--start` for `input#add100` and all its children". But `<input>` elements don't have any children, so the value of `--start` would change only for the `<input>` itself, and the `<input>` itself ignores it. So nothing would happen.

Refresh your page, and then click on each of the checkboxes in turn. You should see the numbers change. Do they change the way you would expect them to?

When you check Add 10, you should see `——[10]——`.
If you check both Add 10 and Add 100, you should see `——[100]——`, not `110`
And if you check Add 1000 _on its own_, you should see `——[1000]——`.

But if you check Add 1000 _and one of the other checkboxes_, you do **not** see `——[1000]——`. You see `——[100]——` or `——[10]——` instead.

![add to --start](images/add-to-start.webp)  
Figure 12.

If you see something different, then check that your ruleset for `#add1000` does appear first in your CSS file, as it does in my code listing above.

I deliberately placed the ruleset for `#add1000` first, so that you can see that the order in which the CSS rulesets are written **does** matter when you set the value of custom CSS property. Even if you change the order of the `<input>` elements in the HTML file, as shown below, the order of the CSS rulesets still takes priority.
```html
  <label>
    Add 1000: <input id="add1000" type="checkbox">
  </label>
  <label>
    Add 100: <input id="add100" type="checkbox">
  </label>
  <label>
    Add 10: <input id="add10" type="checkbox">
  </label>
```
Try it and see.

### Resetting a counter

As you have just seen, you can set a `counter` to a value given by a custom CSS property. If you change the value of the custom CSS property, the values of all the counters will be recalculated.

But you can also use `counter-set` explicitly anywhere in your CSS. Will there be a conflict of interests? Who will win? The custom CSS property or the value give by `counter-set`?

Here's how you could test this. Add a new line to your HTML:

```html
  <p>Tinker</p>
  <p>Tailor</p>
  <b class="resetter"></b> <!-- new line -->
  <p>Soldier</p>
  <p>Sailor</p>
```

Replace your current ruleset for `body` with these two rulesets... which may or may not be in a logical order. (That's what we want to find out.)
```css
b.resetter {
  counter-set: item 9999;
}

body {
  counter-set: item var(--start);
  --start: 0;
}

/* The other lines do not change */
```

Refresh your page.

![counter-set](images/counter-set.webp)  
Figure 13.

In the screenshot above, the custom CSS property `--start` is set to `100`, and this determines what value the `item` counter has, until the `<b class=resetter>` element is displayed. At this point, the rule...
```css
b.resetter {
  counter-set: item 9999;
}
```
... is applied, and the initial value set by the custom CSS property `--start` is forgotten. Once again, it is the order of elements _in the HTML source_ that decides what value a `counter` will have at any particular point.

> To summarize:
> - When you use CSS to alter the value of a custom CSS property, it is the precedence in the CSS which determines the value
> - When you use CSS to alter the value of a CSS counter, it is the order of elements in the HTML file which determines the value.

This is, in fact, logical, but it can be confusing.

### Counting without numbers

So far, every time you've seen a counter, it's looked like a number. Well... that's not quite true. Right at the beginning, in the little sneak-peek demos, you saw counters that look like letters and emojis. You may not even have realized they were counter tokens, although I did give a big hint.

Here's a CSS-only presentation of how the [`@counter-style`](https://developer.mozilla.org/en-US/docs/Web/CSS/@counter-style) at-rule can be used to change the tokens that are used to number items in an `<ol>` list. Click on the style names on the right to see how it changes the counter tokens on the left.

The `<ol>` list on the left contains only empty `<li>` items, so all you see are the `::marker` tokens.

![symbols](images/symbols.webp)  
[Link](https://merncraft.github.io/counter-styles/)

The default value for `list-style-type` is decimal. Your browser has [over 100 built-in alternatives](https://w3c.github.io/predefined-counter-styles/), to cater to writing systems all around the world. You can see them in action [here](https://mdn.github.io/css-examples/counter-style-demo/). In order to create so many styles, the [`@counter-style`](https://developer.mozilla.org/en-US/docs/Web/CSS/@counter-style) at-rule became[available in all major browsers since September 2023](https://caniuse.com/?search=%40counter-style). 

Here, for example, is the built-in `@counter-style` at-rule for Thai numeric symbols. This gives the Unicode code points for each numeral:
```css
@counter-style thai { 
  system: numeric;
  symbols: '\E50' '\E51' '\E52' '\E53' '\E54' '\E55' '\E56' '\E57' '\E58' '\E59';
}
```
The `@counter-style` below has exactly the same effect. It uses the actual [Thai characters](https://compart.com/en/unicode/search?q=thai#characters) instead of their code points.
```css

@counter-style thai { 
  system: numeric;
  symbols: '๐' '๑' '๒' '๓' '๔' '๕' '๖' '๗' '๘' '๙';
}
```
And just like the `counter` property has been made generic so that all HTML elements have access to it, so the `@counter-style` has been made generic, so that you can create your own styles. Here's my rule for English weekday abbreviations:
```css
@counter-style weekdays {
  system: cyclic;
  symbols: "Mon" "Tue" "Wed" "Thu" "Fri" "Sat" "Sun";
  suffix: "";
}
```
By default, each `counter-style` has `"."` as its suffix. If you want to remove this (as I did for the weekdays), or change it to something else (like `")"`), you can use the `suffix` property.

Because you can use _any text_, you can also use emojis. Or [mathematical symbols](https://www.compart.com/en/unicode/category/Sm). Or Unicode [shapes](https://www.compart.com/en/unicode/block/U+25A0) and [arrows](https://www.compart.com/en/unicode/block/U+2190). The W3C specifications also describes the use of images, but at the time of writing, the major browsers have not implemented this for counters yet. (You can set the `list-style-image` as bullet points for a whole list or for an individual list item, though, but this is outside the current topic of counters.)

### `::marker`styling

In the example above, all the Thai list-items appear in red, and the abbreviations "Sat" and "Sun" appear in gold. This is not an effect of the `@counter-style` that is used. This is achieved by [styling the `::marker` pseudo-element](https://developer.mozilla.org/en-US/docs/Web/CSS/::marker).

You can only change a few properties of the `::marker` pseudo-element. You can't change its alignment or position, for instance, but you can change the `color` and even the `content`. Here's the CSS that I've used to make weekends stand out, and react to being rolled over by the mouse:
```css
#weekdays:checked {
  & ~ ol {
    list-style-type: weekdays;

    li:nth-child(7n + 6)::marker,
    li:nth-child(7n)::marker {
      color: gold;
    }

    li:nth-child(7n + 6):hover::marker,
    li:nth-child(7n):hover::marker {
      content: "Weekend!";
    }
  }
}
```
Note that the `:hover` pseudo-class applies to the list item itself; it cannot be applied to the `::marker` pseudo-element.

![hovering on Saturday](images/hover-sat.webp)  
[Link](https://merncraft.github.io/counter-styles/)

The `::marker` pseudo-element only applies to list elements (`<ul>`, `<ul>` and `<dl>`). You can provide CSS rulesets for the  `::before` and `::after` pseudo-elements of any visible element, and these can be styled in many more ways than the `::marker` pseudo-element.


## No-click Actions

So far, you've seen how to count checkboxes, radio buttons and other elements which are displayed on the page. This requires the user to interact physically with your game, and interaction is what gives your players the feeling that they are in control. (But you know that _you_, the game developer, you are _really_ the one who is in control of their experience.)

Often, though, you want certain actions of your game to happen automatically: a countdown timer, for example, or a spinning reel on a fruit machine, or the expression on an emoji face. Or you may want to trigger something without the player performing an action as deliberate as a conscious click.

In this section, I'll describe two solutions: CSS animations and the `:hover` pseudo-class.


## An animated countdown

The easiest project for exploring how CSS animations work with CSS counters is a countdown timer. For this new topic, you can create a new folder with the name "Countdown" and create inside it an HTML file and a linked CSS file. Here's enough HTML to get started:
```html
<a href="/" class="refresh">Refresh Page</a>
```
This is enough to see that your page is loaded in your browser, and to be able to reload it with a simple click. Here is some equally minimal CSS:
```css
body {
  counter-set: countdown 10;
}

a::after {
  content: counter(countdown);
  font-size: 50vmin;
}
```
The `font-size` is not strictly necessary, but it makes the display more dramatic. Your page should now show: 

> <u>Refresh Page 10</u> 

CSS Animations are defined by a [`@keyframes`](https://developer.mozilla.org/en-US/docs/Web/CSS/@keyframes) at-rule. Here is a rule that creates an animation with the name `timer`:
```css
@keyframes timer {
   0% { counter-increment: countdown   0; }
  10% { counter-increment: countdown  -1; }
  20% { counter-increment: countdown  -2; }
  30% { counter-increment: countdown  -3; }
  40% { counter-increment: countdown  -4; }
  50% { counter-increment: countdown  -5; }
  60% { counter-increment: countdown  -6; }
  70% { counter-increment: countdown  -7; }
  80% { counter-increment: countdown  -8; }
  90% { counter-increment: countdown  -9; }
 100% { counter-increment: countdown -10; }
}
```
You can add this at the end of your CSS file.

>Note that I did not type all of this out by hand. I used a hack. My code editor is VS Code, and this comes with a built-in plug-in called Emmet. ([Emmet](https://www.etymonline.com/search?q=emmet) is an old or dialectical word for [ant](https://www.etymonline.com/word/ant).) Emmet provides shortcuts for HTML and CSS code that you might need to type regularly. As of the time of writing, it does not provide shortcuts for generating CSS keyframe animations. So I cheated. I used a shortcut for creating a series of HTML elements with incrementing values. I typed this in my HTML page...
```html
div{ $% counter-increment: countdown -$ }*10
```
>... and then pressed Enter. (Tab also works.) This produced the following output:
```html
<div> 1% counter-increment: countdown -1 </div>
<div> 2% counter-increment: countdown -2 </div>
<div> 3% counter-increment: countdown -3 </div>
<div> 4% counter-increment: countdown -4 </div>
<div> 5% counter-increment: countdown -5 </div>
<div> 6% counter-increment: countdown -6 </div>
<div> 7% counter-increment: countdown -7 </div>
<div> 8% counter-increment: countdown -8 </div>
<div> 9% counter-increment: countdown -9 </div>
<div> 10% counter-increment: countdown -10 </div>
```
>The [curly `{}` brackets](https://docs.emmet.io/abbreviations/syntax/#text) define the text that I want to appear inside the HTML element, [the dollar `$` sign](https://docs.emmet.io/abbreviations/syntax/#item-numbering) is replaced by a line number and the [multiplication `*` sign](https://docs.emmet.io/abbreviations/syntax/#multiplication) tells Emmet how many elements I want.
>
>The output is not exactly what I want, but I then used VS Code's built-in [multi-cursor selection](https://code.visualstudio.com/docs/getstarted/tips-and-tricks#_multi-cursor-selection)to quickly remove the parts that I did not want, to add curly `{}` brackets where I did want them, and to add a `0` between the numbers on the left and the following percentage `%` sign.

An `@keyframes` rule simply gives instructions on what should change after at what point in the animation. It does not give any information on which element this animation should be applied to, or how fast it should run or whether it should repeat, or a number of other things.

To get the countdown timer to work, you can add a single line to your CSS that provides these extra details:
```css
a::after {
  content: counter(countdown);
  animation: timer 10s forwards; /* new line */
}
```
(Yes, I've removed the `font-size` rule, for clarity, but you can keep it if you want.)

This new line says:
- Apply the animation named `timer` to the element identified by `body::after`
- Play the animation for [`10s`](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-duration)
- When the animation reaches the end, don't rewind. Just play it [`forwards`](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-fill-mode).

The animation should start as soon as you load the page, and continue until the counter shows a big `0`. The word `forwards` is one of the values of the [`animation-fill-mode`](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-fill-mode) property. It tells the CSS elves to leave everything as they were told to by the last keyframe they read.

Now you might understand why I started with the `Refresh Page` link. If you click on it, you can play the animation again.

### Transition between frames

You can click on this link to restart the animation. Do you notice that the step from `10` to `9` plays faster than the following steps? This is because, by default, CSS applies a transition between each keyframe. If the property that the animation changes can have intermediate values, then the animation will move it smoothly from one keyframe to the next.

>The word _keyframe_ is used to refer the precise values to use at precise times. The browser will interpolate other (non-key) frames between each keyframe. 

You can observe this interpolation process by adding second `@keyframes timer` at-rule after the first:
```css
@keyframes timer {
   0% {  background-color: #f00; }
  10% {  background-color: #f80; }
  20% {  background-color: #8f0; }
  30% {  background-color: #0f0; }
  40% {  background-color: #0f8; }
  50% {  background-color: #08f; }
  60% {  background-color: #00f; }
  70% {  background-color: #80f; }
  80% {  background-color: #f0f; }
  90% {  background-color: #f08; }
 100% {  background-color: #f00; }
}
```
Because this second at-rule appears later in the CSS file than the other, the CSS elves will ignore the first  `@keyframes timer`. They won't increment the countdown counter any more, they will only change the `background-color`.

Now when you refresh your page, you should see the `background-color` moving smoothly  from one colour to the next. Actually, the default [`transition-timing-function`](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-timing-function) is `ease-in-out`, so there is a pause on each intermediate `background-color`. If you replace your `animation` rule with this...
```css
  animation: timer 10s forwards linear;
```
... then the changes become even smoother.

### Step-wise frames

The value of your `counter` changes by a whole integer at each step. With the default settings, the CSS elves get half-way towards the next way-point and they think "We're closer to next way-point now. Let's change already!" As a result, with your counter, the changes happen after 0.5s, 1.5s, 2.5s, and so on. That's why the change from `10` to `9` happens faster than you would expect.

Keep the `background-color` frames, but change your `animation` rule to this:
```css
  animation: timer 10s forwards step-end;
```
With the `step-end` timing function, you should see an abrupt change to the next `background-color`at the end of every second. Comment out the `background-color` timer, so that the original `counter-increment` plays instead. You should see that change from `10` to `9` now happens the way you would expect.

>What happens if you use [`end`](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-timing-function#end), or [`jump-end`](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-timing-function#jump-end), or  [step-start](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-timing-function#step-start) as the timing function instead? Some of these have the same effect, one does something different.
>
>Whenever you meet a new idea, it is good to try a number of variants. This helps you to remember. The human brain is very sensitive to changes. Your eyes are constantly twitching, so that the signal they send to the brain is always slightly different. [If the image on your retina does not change, your vision becomes poorer](https://www.rochester.edu/newscenter/small-eye-movements-are-critical-for-20-20-vision-415522/). When the motor of your refrigerator is running, your brain stops hearing it. When the motor stops, you suddenly become aware of the lack of noise.
>
>If you give your brain the chance to compare and contrast ideas, your brain will pay more attention.

### Starting an animation

For now, to restart the animation, you have to click on the link to reload the page. Reloading the page will reset everything in your game, so you only want to do this if the player wants to start over from the beginning.

Here's a change you can make so that the animation will restart when you move your mouse over the `<a>` link element. I've given the complete CSS code, so you can compare with what you have in your project, but only a small part has changed.

Note that I have reduced the duration of the animation to just `1s`, because your time is precious, and you don't want to wait ten whole seconds to see the final result.
```css
body {
  counter-set: countdown 10;
}

a::after {
  content: counter(countdown);
  font-size: 50vmin;
}

/* animation rule moved to here... and made to run faster */
a:hover::after {
  animation: timer 1s forwards step-end;
}

@keyframes timer {
  0% { counter-increment: countdown   0; }
 10% { counter-increment: countdown  -1; }
 20% { counter-increment: countdown  -2; }
 30% { counter-increment: countdown  -3; }
 40% { counter-increment: countdown  -4; }
 50% { counter-increment: countdown  -5; }
 60% { counter-increment: countdown  -6; }
 70% { counter-increment: countdown  -7; }
 80% { counter-increment: countdown  -8; }
 90% { counter-increment: countdown  -9; }
100% { counter-increment: countdown -10; }
}

/* @keyframes timer {
   0% {  background-color: #f00; }
  10% {  background-color: #f80; }
  20% {  background-color: #8f0; }
  30% {  background-color: #0f0; }
  40% {  background-color: #0f8; }
  50% {  background-color: #08f; }
  60% {  background-color: #00f; }
  70% {  background-color: #80f; }
  80% {  background-color: #f0f; }
  90% {  background-color: #f08; }
 100% {  background-color: #f00; }
} */
```
Now, if you move your mouse over the text `Refresh Page`, or the `1` of the number `10`, the animation will start. If you move your mouse over the `0` of the number `10`, the CSS elves will start running around in circles. When the mouse is over the `0`, the animation will start, and they will replace the number `10` with the number `9`, which isn't so wide, so the mouse is no longer over the `<a>` link, so the animation will stop, and they will show `10` again, and restart the animation. Over and over.

Depending on which browser you are using, you may see the display flicker between `10` and `9`, or you may just see the cursor flicker between a pointer and an arrow.

### Restarting when the state changes

If you hold the mouse in a place where the animation will run to the end, you should see a big `0`. If you then move the mouse away from the link, it will revert to showing `10`.

Before, it played all the way to the end, and then stayed at `0`. What has changed?

Before, the animation started as soon as the page was loaded. The page remained loaded after the animation finished, so the CSS elves left everything the way they were told to by the final `100%` keyframe of the animation. Now, when you move the mouse off the link, they consider the animation to be switched off again, and reset everything to its original state.

### Pausing an animation

Instead of `:hover`, you can use the state of a checkbox or a radio button to change the state of an animation. Edit your HTML page so that it looks like this:
```html
  <input type="radio" name="play-state" id="play">
  <input type="radio" name="play-state" id="paused">
  <input type="radio" name="play-state" id="stop">
  <a href="/" class="refresh">Refresh Page</a>
```
Notice that there are three radio buttons which all appear before the `<a>` link. These radio buttons share the same `name` property, so only one can be on at a time, but they all have different `id` values.

In your CSS page, comment out the ruleset for `a:hover::after` and add a new rule for the radio buttons:
```css
/* a:hover::after {
  animation: timer 1s forwards step-end;
} */

input:checked ~ a::after {
  animation: timer 1s forwards step-end;
}
```
The selector `input:checked ~ a::after` uses the `~` sibling combinator. It says: "Apply the following rule to any `<a>` link where there is an `<input>` that is `:checked` with the same parent earlier in the HTML page". All the radio buttons appear before the  `<a>` link, so if any of them is checked, the `animation` rule will apply.

Refresh your page so that no radio buttons are checked, and then click any one of them. The animation plays to the end, and stops on `0`. Click on any of the other radio buttons.

Nothing happens.

The rules says: "If any  of the radio button `<input>`s  is `:checked`, play the animation. If not, stop the animation." So it doesn't matter which button is checked. They all agree with each other.

What happens if you specify that it only the button with an `id` of `play` is to make the animation play? Try this:
```css
input#play:checked ~ a::after {
  animation: timer 1s forwards step-end;
}
```
Now, if you click on the first button, the animation plays to the end. If you click on one of the other buttons, the display reverts to showing `10`.

Add a ruleset for the `#paused` button, as shown below. (Note that I have reset the duration for both buttons to the same value of `10`, so that you have time to click on different buttons while the animation is in progress.)
```css
input#play:checked ~ a::after {
  animation: timer 10s forwards step-end;
}

input#paused:checked ~ a::after {
  animation: timer 10s forwards step-end;
  animation-play-state: paused;
}
```
This also plays the animation when it is selected, but it sets the [`animation-play-state`](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-play-state) to `paused`. So if you refresh your page and then click the middle radio button, nothing happens. (The CSS elves are in fact busy, but they are busy doing nothing.) If you click on the left-most (`#play`) radio button, the animation will start. If you click on the middle (`#pause`) button, it will ... pause. You can continue the animation from this by clicking on the left-most button again. Or you can reset the counter to `10` by clicking on the right-most (`stop`) button.

### Who owns the animation?

What happens if you comment out the line that makes the `#paused` play the animation:
```css
input#paused:checked ~ a::after {
  /* animation: timer 10s forwards step-end; */
  animation-play-state: paused;
}
```
If you start the animation with the first button and then click on the `#pause` button, the animation reverts to its initial state, just as if you had clicked on the `#stop` button. The `animation-play-state` no longer knows which animation it refers to.

Actually, I think that this version is more elegant:
```css
input#paused:checked ~ a::after,
input#play:checked ~ a::after {
  animation: timer 10s forwards step-end;
}

input#paused:checked ~ a::after {
  animation-play-state: paused;
}
```
Why? DRY! The duration of `10s` is defined in only one place. In the earlier version, you could set a different duration in each of the rulesets, and unexpected things could happen.

The selector that you use for the different rulesets must apply to exactly the same element. The animation "belongs" to that element. The selectors used can look very different, so long as they refer to the same animated element. These rulesets would also work:
```css
body:has(input#paused:checked) a::after,
input#play:checked ~ a::after {
  animation-name: timer;
  animation-duration: 10s;
  animation-fill-mode: forwards;
  animation-timing-function: step-end;
}

body:has(input#paused:checked) a::after {
  animation-play-state: paused;
}
```
The more complex selector for the `#paused` button means that it could now be anywhere in the HTML page, even after the animated element, or with a different parent:
```html
  <input type="radio" name="play-state" id="play">
  <input type="radio" name="play-state" id="off">
  <a href="/" class="refresh">Refresh Page</a>
  <div>
    <input type="radio" name="play-state" id="paused">
  </div>
```

### Triggering a timeout
### Animating a counter-style
### Using an animation to cycle a list smoothly

## Incrementing with `:hover`


# Complete a Game by Adding Counters

I wrote at the beginning of this article that I would not be showing you, step by step, how to build a particular game. However, I do want to give you the chance to test whether my explanations have made good sense to you.

I've created a game that uses the techniques that I have describe above. You can find the GitHub Repository [here](https://github.com/MERNCraft/path).

![path](images/path.webp)  
[Link](https://MERNCraft.github.io/path/)

If you clone the repository and launch the game on your own computer, you will find that the scoring and timing system is missing. Using the knowledge that you have just acquired, can you get your local version of the game to behave the same as the online version?

The logic of the game (starting and stopping it, picking up the gold tokens, showing the result when you reach the end) is already written. You'll find the CSS for this in a file called `styles.css`. You should not need to change anything in this file or in the `index.html` file.

There is a second CSS file attached to the HTML page. It's called `counters.css`, and it currently contains no CSS at all. You should now be able to write CSS of your own that:
- Shows the score
- Shows an emoji that gets happier as the score increases
- Shows how many seconds have passed since the player clicked on the green Start button
- Stops the timer when the game is over.
  
And of course, you can use this as the starting point for your own remixed version of the game. I'm sure that, artistically, your CSS power does exceed my own...