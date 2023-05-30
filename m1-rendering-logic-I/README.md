# Rendering Logic I

## built-in and inheritance

HTML tags do include a few minimal styles. For example, here are the built-in styles for `<a>` tags, in Chrome 86:

```css
a {
  color: -webkit-link;
  cursor: pointer;
  text-decoration: underline;
}
```

- **Inheritance**

```css
p {
  color: deeppink;
}
```

```html
<p>I know <em>you</em> are, but what am I?</p>
```

![images1](/images/m1/1.png)

How come the word `you` is colored pink? If you think about it, we haven't applied any styles to the em element, only to paragraphs.

The answer is that certain CSS properties **`inherit`**. When I apply a color to an element, that value gets passed down to all children and grand-children.

Not all CSS properties are inheritable, though. Let's look at another example:

```css
p {
  border: 1px solid hotpink;
}
```

```html
<p>I know <em>you</em> are, but what am I?</p>
```

![images2](/images/m1/2.png)

Notice that the `border` value doesn't get passed down to the `em`. We still only have 1 border, and it's around the entire paragraph.

If we explicitly add the same `border` value to both paragraphs and emphasis tags, we see a second border within our paragraph :

```css
p, em {
  border: 1px solid hotpink;
}
```

![images3](/images/m1/3.png)

This behaviour probably isn't surprising to you, but it's a bit weird when you think about it!

The people who wrote the CSS spec opted to make certain properties inheritable for convenience. It's a DX (Developer Experience) thing.

Most of the properties that inherit are typography-related, like color, font-size, text-shadow, and so on.

### Forcing inheritance

Occasionally, you may wish to have a property inherit even when it wouldn't normally do so.

A good example is link colors. By default, anchor tags have built-in styles that give unvisited, inactive links a blue hue:

```html
<p>
  This paragraph contains <a href="#">a hyperlink</a>!
</p>
```

![images3](/images/m1/4.png)

the `built-in` style for `<a>` tags are like this:

```css
a {
  color: -webkit-link;
  cursor: pointer;
  text-decoration: underline;
}
```

The trouble is that even though color is an inheritable property, it's being overwritten by the default style, color: `-webkit-link`

We can fix this by explicitly telling anchor tags to inherit their containing text color:

```css
a {
  color: inherit;
}
```

```html
<p>
  This paragraph contains <a href="#">a hyperlink</a>!
</p>
<p style="color: red;">
  This is a red paragraph with <a href="#">another link</a>.
</p>
```

![images3](/images/m1/5.png)

## The Cascade

```html
<style>
  p {
    font-weight: bold;
    color: hsl(0deg 0% 10%);
  }

  .introduction {
    color: violet;
  }
</style>

<p class="introduction">
  Hello world
</p>
```

code above was creating 2 rules, one targeting a `<p>` tag, another targeting class `introduction`.

You may already know what happens here: we wind up with a bold, violet paragraph. It plucks the font-weight declaration from the p tag, and the color declaration from the .introduction class.

This example shows the browser's **cascade algorithm** at work.

The CSS language includes many different selectors, and each selector has a relative power. For example, classes are "more specific" than tags, so if there is a conflict between a class and a tag, the class wins. IDs, however, are more specific than classes.

### Similarities with JS merging

Here's a fun thought experiment: how might we implement the cascade in JavaScript?

It turns out, JS has the perfect tool for the job: [spread syntax](`<https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax>`).

In principle, the cascade algorithm works something like this:

```js
const appliedStyles = {
  ...inheritedStyles,
  ...tagStyles,
  ...classStyles,
  ...idStyles,
  ...inlineStyles,
  ...importantStyles
}
```

or if we break down variable above will like this

```js
const tagStyles = {
  fontWeight: 'bold',
  color: 'hsl(0deg 0% 10%)',
};
const classStyles = {
  color: 'violet',
}

const appliedStyles = {
  ...tagStyles,
  ...classStyles,
}
```

The order that they're merged in is determined by specificity; class styles are more specific than tag styles, so they're merged in later. This way, they overwrite any conflicting styles. All non-conflicting styles are kept.

## Block and Inline Directions

The web was built for displaying inter-linked documents. A lot of CSS mechanics and terminology are inherited from the print world.

English is a `left-to-right language`, meaning that the words are placed side-by-side, from left to right. Individual words are combined into blocks, like headings and paragraphs.

Pages are constructed out of blocks, placed one on top of the other. When a new paragraph is added to the page, it's inserted below the previous block element.

**CSS** builds its sense of direction based on this system. It has a `block direction` (vertical), and an `inline direction` (horizontal).

### Logical properties

we've learn about built-in style --- these are the rules that each browser comes with out-of-the-box, defined in the user-agent stylesheet.

Here are the built-in styles for `<p></p>` tags, in Chrome:

```css
p {
  display: block;
  margin-block-start: 1em;
  margin-block-end: 1em;
  margin-inline-start: 0px;
  margin-inline-end: 0px;
}
```

These properties are equivalent to the 4 cardinal directions: `margin-top`, `margin-bottom`, `margin-left`, and `margin-right`.

`margin-block` refers to the vertical axis. `margin-block-start` refers to the top, since block elements are stacked top-to-bottom.

Why use these alternatives? Because not all languages are left-to-right, top-to-bottom. If you were to switch your browser's language to Arabic, margin-inline-start would add spacing to the right instead of to the left, since Arabic is a right-to-left language.

These alternatives are known as **logical properties**. It's not just margin: there are logical variants for padding, border, and overflow as well.

#### When should we use logical properties?

Logical properties are intended to entirely replace their conventional alternatives. In the future, we might all write `margin-inline-start` instead of `margin-left`!

The main selling point for logical properties is *internationalization*. If you want your product to be available in a left-to-right language like English and a right-to-left language like Arabic, you can save yourself a lot of trouble by using logical properties.

## The Box Model

The most common CSS-related job-interview question is probably this:
**Can you explain the box model?**

Typically, though, the answer given is quite shallow, and glosses over a lot of details. This is unfortunate, since the Box Model is a critical part of CSS' rendering model!

### Winter Layers

The four aspects that make up the box model are:

1. Content
2. Padding
3. Border
4. Margin

A helpful analogy is to imagine a person out for a winter walk, wearing a big poofy coat:

<img src="/images/m1/person-walking.png" height="30%"/>

- The `content` is the person themselves, the human being inside the coat.
- The `padding` is the polyester stuffing in the coat. The more stuffing there is, the more poofed-up the coat will be, and the more space the person will take up.
- The `border` is the material of the coat. It has a thickness and a color, and it affects the person's appearance.
- The `margin` is the person's “personal space”. As we've learned in recent years, it's good to have 2 meters (6 feet) of space around us.

### Box Sizing

```html
<style>
  section {
    width: 500px;
  }
  .box {
    width: 100%;
    padding: 20px;
    border: 4px solid;
  }
</style>

<section>
  <div class="box"></div>
</section>
```

code above will creating box `548px x 48px` total.

![image](/images/m1/box-model-thing-dimensions.png)

When we set our `.box` to have `width: 100%`, we're saying that the box's content size should be equal to the available space, `500px`. The padding and border is added on top.

Our box winds up being 548px wide because it adds 20px of padding and 4px of border to each side: `500 + (20 * 2) + (4 * 2)`.

The same thing happens with `height`: because the element is empty, it has a `content size of 0px`, with the same border and padding added on top.

This behaviour is surprising, and generally not what we want as developers! Thankfully, browsers provide an escape hatch.

The `box-sizing` CSS property allows us to change the rules for size calculations. The default value (`content-box`) only takes the inner content into account, but it offers an alternative value: `border-box`.

With `box-sizing: border-box`, things behave much more intuitively.

#### **A new default**

Instead of having to remember to swap box-sizing on every layout element, we can set it as the default value for all elements with this handy CSS snippet:

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

Why the pseudo-elements?
You might be wondering: why do we need to add `*::before` and `*::after`? Shouldn't the wildcard selector (`*`) select everything?

Well, yes and no. The `*` selector will select all elements, but `::before` and `::after` aren't considered elements. They're pseudo-elements. And so we need to select them explicitly.

## Padding

A helpful way to think about padding is that it's "inner space".

```css
.some-fella {
  padding: 48px;
  background-color: tomato;
}
```

css above will creating like this:

![image](/images/m1/6.png)

Padding can be set for all directions at once, or it can be specified for individual directions:

```css
.even-padding {
  padding: 20px;
}

.asymmetric-padding {
  padding-top: 20px;
  padding-bottom: 40px;
  padding-left: 60px;
  padding-right: 80px;
}

/* The same thing, but using ✨ logical properties ✨ */
.asymmetric-logical-padding {
  padding-block-start: 20px;
  padding-block-end: 40px;
  padding-inline-start: 60px;
  padding-inline-end: 80px;
}
```

### Units

When applying padding, we can pick from a pretty wide range of units. The most common ones are:

- `px`
- `em`
- `rem`

Many developers believe that pixels are bad for accessibility. This is true when it comes to font size, but I actually think pixels are the best unit to use for padding (and other box model properties like margin/border).

```css
header {
  font-size: 2rem;
  padding: 32px;
  background-color: peachpuff;
}
main {
  background: silver;
  padding: 32px;
}
```

```html
<header>
  My App
</header>
<main>
  <p>
    These are some words you might find on a web application.
    They are indeed words, and nobody can refute that.
  </p>
</main>
```

![image](/images/m1/7.png)

### Shorthand properties

The padding property has a couple tricks up its sleeve. It can be used to set asymmetric padding, in a few different ways.

```css
.two-way-padding {
  padding: 15px 30px;
}

.asymmetric-padding {
  padding: 10px 20px 30px 40px;
}
```

If two values are passed in `two-way-padding`, the first value is used for both vertical directions (top/bottom), and the second value is used for horizontal directions (left/right).

If all 4 values are passed in `asymmetric-padding`, it applies them in a specific order: top, right, bottom, left.

The easiest way to remember this order is to picture a clock. We start at the top (12:00), and go clockwise for the remaining values:

![image](/images/m1/shorthand-clock.png)

If only 3 values, we set top/right/bottom explicitly, and mirror the right value to the left.

#### Overwriting values

Let's say we want to produce an element with only 3 padded sides:

![imgage](/images/m1/8.png)

We could do this with our shorthand property:

```css
.box {
  padding: 48px 48px 0;
}
```

There is another way to represent the same intent, which is arguably clearer:

```css
.box {
  padding: 48px;
  padding-bottom: 0;
}
```

"Long-form" properties can *overwrite* the relevant value in shorthand properties. The effect is the same, but it's a bit more semantic; instead of a random string of numbers, we're declaring that we want 48px of padding, hold the bottom.

## Border

Border is a bit of an odd duck in the trinity of padding/border/margin—unlike the other two, it has a visual/cosmetic component.

There are three styles specific to border:

- Border width (eg. `3px`, `1em`)
- Border style (eg. `solid`, `dotted`)
- Border color (eg. `hotpink`, `black`)

They can be combined into a shorthand:

```css
.box {
  border: 3px solid hotpink;
}
```

The only required field is border-style. Without it, no border will be shown!

```css
.not-good {
  /* 🙅‍♀️ Won't work – needs a style! */
  border: 2px pink;
}

.good {
  /* 🙆‍♀️ Will produce a black, 3px-thick border */
  border: solid;
}
```

As with padding, you can overwrite broad properties with specific ones. This is useful for creating several "variants" of an element.

```html
<div class="box one"></div>
<div class="box two"></div>
<div class="box three"></div>
```

```css
.box {
  width: 50px;
  height: 50px;
  border: 10px solid;
}

.box.one {
  border-color: deeppink;
}

.box.two {
  border-color: gold;
}

.box.three {
  border-color: turquoise;
}
```

![image](/images/m1/9.png)

If we don't specify a border color, it'll use the font's color by default. This isn't well-known, but it can be useful in cases where those things should be synchronized!

### Border radius

The CSSWG (CSS Working Group, the organization that maintains and advances the CSS language.) has published a list of mistakes they've made with the CSS language. One of these mistakes is listed:
**border-radius should have been corner-radius.**

It's not hard to understand the rationale; the border-radius property rounds an element even if it has no border!

Like `padding`, `border-radius` accepts discrete values for each direction. Unlike padding, it's focused on specific corners, not specific sides. Here are some examples:

```html
<style>
  .box {
    width: 100px;
    height: 100px;
    border: 4px solid hotpink;
    border-radius: 25px;
  }
</style>

<div class="box"></div>
```

![image](/images/m1/10.png)

```html
<style>
  .box {
    width: 100px;
  height: 100px;
  border: 4px solid hotpink;
  border-radius: 10px 10px 40px 40px;
  }
</style>
<div class="box"></div>
```

![image](/images/m1/11.png)

You can also use percentages; 50% will turn your shape into a circle or oval, since each corner's radius is 50% of the total width/height:

```html
<style>
  .box {
    width: 100px;
  height: 100px;
  border: 4px solid hotpink;
  border-radius: 50%;
  }
</style>
<div class="box"></div>
```

![image](/images/m1/12.png)

### Border vs. Outline

A common stumbling block for devs is the distinction between `outline` and `border`. In some respects, they're quite similar! They both add a visual edge to a given element.

The core difference is that **outline doesn't affect layout**. Outline is kinda more like box-shadow;

Outlines share many of the same properties:

- `border-width` becomes `outline-width`
- `border-color` becomes `outline-color`
- `border-style` becomes `outline-style`

Outlines are stacked outside border, and can sometimes be used as a "second border", for effect:

```css
.box {
  width: 100px;
  height: 100px;
  border: 4px solid darkviolet;
  outline: 4px solid deeppink;
}
```

![image](/images/m1/13.png)

A couple more quick tidbits about outlines:

- Outlines will follow the curve set with border-radius in all browsers except Safari. This is a recent change; before September 2021, most browsers kept outlines straight and boxy.
- Outlines have a special `outline-offset` property. It allows you to add a bit of a gap between the element and its outline.

```css
.outline-offset {
  width: 100px;
  height: 100px;
  border: 4px solid darkviolet;
  outline: 4px solid currentColor;
  outline-offset: 4px;
}
```

![image](/images/m1/14.png)

*One more thing*: `Outlines` are sometimes used as focus indicators, for folks who use the keyboard (or other non-pointer devices) to navigate.

## Margin

Margin increases the space around an element, giving it some breathing room. As we saw earlier, margin is "personal space".

In some ways, margin is the most amorphous and mysterious. It can do wacky things, like pull an element outside a parent, or center itself within its container.

The syntax for margin looks an awful lot like padding:

```css
.spaced-box {
  margin: 20px;
}

.asymmetrically-spaced-box {
  margin: 20px 10px;
}

.individually-specified-box {
  margin-top: 10px;
  margin-left: 20px;
  margin-right: 30px;
  margin-bottom: 40px;
}
.logical-box {
  margin-block-start: 20px;
  margin-block-end: 40px;
  margin-inline-start: 60px;
  margin-inline-end: 80px;
}
```

### Negative margin

With padding and border, only positive numbers (including 0) are supported. With margin, however, we can drop into the negatives.

```css
main {
  width: 200px;
  height: 200px;
  border: 3px solid;
}

.pink-box {
  width: 50%;
  height: 50%;
  border: 3px solid deeppink;
  background: white;
  margin-top: -32px;
  margin-left: -32px;
}
```

```html
<main>
  <div class="pink-box"></div>
</main>
```

![image](/images/m1/15.png)

Negative margin can affect the position of all siblings. Check out what happens when we apply negative margin to the first box in a series:

```html
<main>
  <div class="box one"></div>
  <div class="box two"></div>
  <div class="box three"></div>
</main>
```

```css
main {
  width: 200px;
  height: 200px;
  border: 3px solid silver;
}

.box {
  width: 25%;
  height: 25%;
  border: 3px solid;
  background: white;
}

.box.one {
  border-color: deeppink;
  margin-top: -24px;
}
```

![image](/images/m1/16.png)

The interesting thing is those two black boxes: they "follow" the deep pink box up.

### Auto margins

Margins have one other trick up their sleeve: they can be used to center a child in a container.

```html
<main>
  <section class="content">
    Hello World
  </section>
</main>
```

```css
.content {
  width: 50%;
  margin-left: auto;
  margin-right: auto;
  background: palevioletred;
  padding: 16px;
}

main {
  width: 100%;
  height: 200px;
  border: 3px solid silver;
}
```

![img](/images/m1/17.png)

The `auto` value seeks to fill the **maximum available space**. It works the same way for the width property, as we'll discover shortly.

If you take the free space around an element and distribute it evenly on both sides, you wind up centering that element. This is a happy byproduct of this mechanism!

Two caveats:

- **This only works for horizontal margin.** Setting top/bottom margin to auto is equivalent to setting it to 0px.
- **This only works on elements with an explicit width**. Block elements will naturally grow to fill the available horizontal space, so we need to give our element a width in order to center it.

## Exercises

1. Thinking outside the box

Your mission in this exercise is to reproduce the following effect:

```css
body {
  background: #222;
  padding: 32px;
}

.card {
  background-color: white;
  padding: 32px;
  border-radius: 8px;
}

h1 {
  background: deeppink;
  padding: 16px 32px;
  margin-top: -48px;
  margin-bottom: 24px;
  font-size: 2rem;
  text-align: center;
  border-radius: 4px;
}
```

```html
<div class="card">
  <h1>An Otter Essay</h1>
  <p>
    Otters have long, slim bodies and relatively short limbs. Their most striking anatomical features are the powerful webbed feet used to swim, and their seal-like abilities holding breath underwater.
  </p>
</div>
```

![image](/images/m1/18.png)

## Flow Layout

Every HTML element will have its layout calculated by a layout algorithm. These are known as *“layout modes”*, and there are 7 distinct ones.

There is several layout including Positioned layout, “Flexible Box” layout (AKA Flexbox), and Grid layout (AKA CSS Grid).

There are two main element types in Flow layout:

1. **Block elements**: things like headings, paragraphs, footers, asides. The chunks of content that make up a page.
2. **Inline elements**: things like links, or a string of bold text.

Each HTML tag has a default type. `<div>` and `<header>` are block elements, span and `<a>` are inline elements.

```css
/* Transform a particular <a> tag from `inline` to `block`: */
a.nav-link {
  display: block;
}
```

There's also a third value, inline-block, which is a sort of fusion between the two types.

### Inline elements don't want to make a fuss

You can shift things in the inline direction with `margin-left` and `margin-right`, since that pushes it around in the inline direction, but you can't give it a `width` or `height`. When it comes to layout, an inline element is where it is, and there's not much we can do about it.

#### **Replaced elements**

There's an exception to this rule: **replaced elements**.
A replaced element is one that embeds a "foreign" object. This includes:

- `<img />`
- `<video />`
- `<canvas />`

### Block elements don't share

When you place a block level element on the page, its content box greedily expands to fill the entire available horizontal space.

A heading might only need 150px to contain its letters, but if you put it in an 800px container, it will expand to fill 800px of width.

```html
<h2>
  Hello World
</h2>
```

```css
h2 {
  border: 2px dotted;
}
```

![img](/images/m1/19.png)

What if we force it to shrink down to the minimum size required for the letters? We can do this with the special width keyword fit-content;

```html
<h2>
  Hello World
</h2>
<div class="red-box"></div>
```

```css
h2 {
  width: fit-content;
  border: 2px dotted;
}

.red-box {
  width: 50px;
  height: 25px;
  background: red;
}
```

![img](/images/m1/20.png)

Even though there's plenty of space left on that first row, the red box sits underneath our heading. The `h2` does not want to share any inline space.

In other words, elements that are display: block will stack in the block direction, regardless of their size.

### Inline elements have “magic space”

Inline elements are a bit sneaky, though.

the example is when have fixed image size of 300x300 pixels, sitting in a plain `div`.
![img](/images/m1/cat-300px.jpg)

then when you open the browser devtools you'll notice something peculiar… The image is taking up the correct size, 300×300, but the parent has a slightly larger height.

The image is 300px tall, but its parent `<div>`is 306px tall. Where are those extra few pixels coming from?? It's not padding, it's not border, it's not margin…

The reason for this extra **“magic space”** is that the browser treats inline elements as if they're typography.

There are two ways we can fix this problem:

1. Set images to `display: block` — if you're noticing this problem, there's a good chance your images aren't interspersed with text, so setting them to display as blocks makes sense.
2. Set the `line-height` on the wrapping div to `0`:

This space is proportional to the height of each line, so if we reduce the line height to 0, this **“magic space”** goes away. Because our container doesn't contain any text, this property has no other effect.

### Space between inline elements

There's another unrelated way that inline elements have a bit of extra spacing. Take a look at this example; notice that there are gaps between the 3 images:

```html
<img
  src="/cfj-mats/placekitten-100.jpg"
  width="100"
  alt="Cat photo"
/>
<img
  src="/cfj-mats/placekitten-100.jpg"
  width="100"
  alt="Cat photo"
/>
<img
  src="/cfj-mats/placekitten-100.jpg"
  width="100"
  alt="Cat photo"
/>
```

![img](/images/m1/21.png)

This space is caused by the whitespace between elements. If we squish our HTML so that there are no newlines or whitespace characters between images, this problem goes away:

```html
<!--
  This is exactly the same HTML,
  but without any whitespace
-->
<img src="/cfj-mats/placekitten-100.jpg" width="100" alt="Cat photo" /><img src="/cfj-mats/placekitten-100.jpg" width="100" alt="Cat photo" /><img src="/cfj-mats/placekitten-100.jpg" width="100" alt="Cat photo" />
```

![img](/images/m1/22.png)

This happens because HTML is space-sensitive, at least to an extent. The browser can't tell the difference between whitespace added to separate words in a paragraph, and whitespace added to indent our HTML and keep it readable.

### Inline elements can line-wrap

Inline elements have one pretty big trick up their sleeves; they can line-wrap.

Unlike block elements, an inline element can produce shapes other than boxes.

![img](/images/m1/flow-layout-inline-wrapping.png)

If we add a border, we can see that we don't get 2 discrete rectangles, but rather a single rectangle cut in half and repositioned:

```css
strong {
  border: 2px solid;
}
```

```html
<p>
  This is a paragraph with <strong>some very bolded words in it</strong>.
</p>
```

![img](/images/m1/23.png)

### The deal with inline-block

One of the most confusing things about Flow layout is the Frankenstein `display: inline-block` value.

```css
strong {
  display: inline-block;
  color: white;
  background-color: red;
  width: 100px;
  margin-top: 32px;
  text-align: center;
}

strong:hover {
  transform: scale(1.2);
}
```

```html
<p>
  <strong>Warning:</strong> Alpaca may bite.
</p>
```

![img](/images/m1/24.png)

In this example, we've set our `<strong>` to be inline-block. Because this tag is now secretly a block-level element, it has access to the full universe of CSS.

#### Inline-block doesn't line-wrap

This is a neat effect, but it relies on CSS properties like clip-path, properties that don't work with inline elements. In order for this to work, we need to switch to `display: inline-block`.

Here's the big downside with `inline-block`: It disables `line-wrapping`.

This might not seem like a big tradeoff, but consider what happens when we try to use this on longer-length links:

![img](/images/m1/inline-block-links.png)

Because the link can't line-wrap, it forces the entire link onto the next line, adding an awkward gap.

## Width Algorithms

We learned how block-level elements like h1 and p will expand to fill the available space.

It's easy to assume that this means that they have a default width of `100%`, but that wouldn't quite be right.

```html
<h1>
  Hello World
</h1>
```

```css
h1 {
  /* width: 100%; */
  margin: 0 16px;
  background-color: chartreuse;
}
```

![img](/images/m1/25.png)

When we enable `width: 100%`, we cause the heading to pop outside of our `frame`. This happens because of the `margin`.

When we use percentage-based widths, those percentages are **based on the parent element's content space**. If the body tag makes `400px` of space available, any child with `100%` width will become `400px` wide, regardless of any other circumstances.

### Keyword values

Broadly speaking, there are two kinds of values we can specify for width:

1. Measurements (100%, 200px, 5rem)
2. Keywords (auto, fit-content)

Measurement-based values are either completely explicit (eg. `200px`), or relative to the parent's available space (eg. `50%`).
We've already seen how `auto` will let our element greedily consume the available space while respecting any constraints. `

#### min-content

When we set `width: min-content`, we're specifying that we want our element to become as narrow as it can, based on the child contents.

```css
h1 {
  width: min-content;
  background-color: fuchsia;
}
```

```html
<h1>
  This heading is shrunk down.
</h1>
```

![img](/images/m1/26.png)

#### max-content

This value is similar in principle, but it takes an opposite strategy: it *never* adds any line-breaks.

```css
h1 {
  width: max-content;
  background-color: mediumspringgreen;
}
```

```html
<h1>
  This heading is constrained using max-content, which causes the line to extend far longer than it otherwise would!
</h1>
```

![img](/images/m1/27.png)

As you can see, an element with `width: max-content` pays no attention to the constraints set by the parent. It will size the element based purely on the length of its unbroken children.

It produces a nice effect when the content is short enough to fit within the parent:

![img](/images/m1/28.png)

Unlike with `auto`, `max-content` doesn't fill the available space. If we want to add a background color only around the letters, this would be a neat way to do it!

#### fit-content

Here's how it works: like `min-content` and `max-content`, the width is based on the size of the children. If that width can fit within the parent container, it behaves just like `max-content`, not adding any `line-breaks`.

```css
h2 {
  width: fit-content;
  background-color: peachpuff;
  margin-bottom: 16px;
  padding: 8px;
}
```

```html
<h2>Short</h2>
<h2>A mid-length heading</h2>
<h2>The longest heading you've ever seen in your life, will it ever end, ahhhhh ohmigod 😬😬😬😬😬😬😬</h2>
```

![img](/images/m1/29.png)

#### Min and max widths

We can add constraints to an element's size using min-width and max-width.

```css
.box {
  width: 50%;
  min-width: 170px;
  max-width: 300px;
  margin: 0 auto;
  border: solid hotpink;
}
```

```html
<div class="box">
  Hello World
</div>
```

Try resizing this result to see how the box changes size!

|   |   |
|---|---|
| ![image](/images/m1/30.png) | ![img](/images/m1/31.png) |

#### **Exercises**

**Max Width Wrapper**

Solution from me:

```html
<style>
  .max-width-wrapper {
    max-width: 350px;
    margin: 0 auto;
    padding: 0 16px;
  }
</style>

<div class="max-width-wrapper">
  <div class="card">
    <p>
      Otters have long, slim bodies and
      relatively short limbs. Their most
      striking anatomical features are
      the powerful webbed feet used to
      swim, and their seal-like
      abilities holding breath
      underwater.
    </p>
  </div>
</div>
```

**Figures and captions**

The `<figure>` HTML element is fairly niche, but super useful. It allows us to display any sort of “non-typical” content: images, videos, code snippets, widgets, etc. It also lets us caption that content with `<figcaption>`.

Here's how it's used:

```html
<figure>
  <img
    src="/graph.jpg"
    alt="A bar graph showing the number of cats per capita"
  />
  <figcaption>
    Source: Cat Scientists Ltd.
  </figcaption>
</figure>
```

`<figure>` elements are block-level elements, which means they fill the available horizontal space.

solution:

```css
figure {
  padding: 8px;
  border: 1px solid;
  margin-bottom: 32px;
  margin-left: auto;
  margin-right: auto;
  width: min-content;
}

figure img {
  margin-bottom: 8px;
}

figcaption {
  text-align: center;
  color: #666666;
}
```

## Height Algorithms

The default `"width"` behaviour of a block-level element is to fill all the available width, whereas the default `"height"` behaviour is to be as small as possible while fitting all of the element's content; it's closer to `width: min-content` than `width: auto`!

Also, we tend to treat height as "more dynamic" than width. We might feel comfortable setting our main content wrapper to have a max width of 750px, but we wouldn't usually do this with height; We generally want to avoid setting fixed heights.

Our goal is to have a .wrapper element that will be no smaller than 100% of the available height. But `min-height: 100%` doesn't work:

```css
.wrapper {
  min-height: 100%;
  border: solid;
}
```

```html
<section class="wrapper">
  <p>
    I'm not very tall!
  </p>
</section>

```

As we saw earlier, the default behaviour of an element in terms of height is to be as small as possible, to contain its children.

Our section sits inside the `<body>` tag, and so when we set a percentage-based height or min-height, the percentage is based on that parent height. `<body>` doesn't have a specific height set, which means it uses the default behaviour: stay as short as possible, while still containing all the children.

In other words, we have an impossible condition: we're telling the `<section>` to be a percentage of the `<body>`, and the `<body>` wants to base its size off of the `<section>`. They're both looking to each other for guidance.

Here's how to fix it:

1. Put `height: 100%` on every element before your main one (including `html` and `body`)
2. Put `min-height: 100%` on that wrapper
3. Don't try and use percentage-based heights within that `wrapper`

When html is given `height: 100%`, it takes up the height of the viewport. That serves as our base. The body tag's 100% is based on that base size.

```css
html, body {
  height: 100%;
}
.wrapper {
  min-height: 100%;
  border: solid;
}x
```

```html
<div class="wrapper">
  <p>
    I fill the viewport now!
  </p>
</div>
```

![image](/images/m1/32.png)

**What about the vh unit?**

You may be familiar with the vh unit, a unit designed exactly for this purpose. If you set height: 100vh, your element will inherit its height from the viewport size.

## Margin Collapse

Margins work in a similar fashion—adjacent margins will sometimes "collapse", and overlap.

### Rules of Margin Collapse

### Only vertical margins collapse

Here's the "canonical" margin-collapse example — multiple paragraphs in a row:

```html
<style>
  p {
    margin-top: 24px;
    margin-bottom: 24px;
  }
</style>

<p>Paragraph One</p>
<p>Paragraph Two</p>
```

Each paragraph has `24px` of vertical margin (`margin-top` and `margin-bottom`), and that margin collapses. The paragraphs will be `24px` apart, not `48px` apart.

Hover over (or focus) this visualization. This page is full of these interactive illustrations that show how margins do (or don't) overlap.

![image](/images/m1/33.png)

When margin-collapse was added to the CSS specification, the language designers made a curious choice: horizontal margins (`margin-left` and `margin-right`) shouldn't collapse.

So that's our first rule: only vertical margins collapse.

![image](/images/m1/34.png)

### Margins only collapse in Flow layout

The web has multiple layout modes, like positioned layout, flexbox layout, and grid layout.
Margin collapse is unique to Flow layout. If you have children inside a `display: flex` parent, those children's margins will never collapse.

### Only adjacent elements collapse

It is somewhat common to use the `<br/>` tag (a line-break) to increase space between block elements.

```html
<style>
  p {
    margin-top: 32px;
    margin-bottom: 32px;
  }
</style>

<p>Paragraph One</p>
<br />
<p>Paragraph Two</p>
```

![image](/images/m1/35.png)

The `<br/>` tag is invisible and empty, but any element between two others will block margins from collapsing. Elements need to be adjacent in the DOM for their margins to collapse.

### The bigger margin wins

What about when the margins are asymmetrical? Say, the top element wants `72px` of space below, while the bottom element only needs `24px`?

![image](/images/m1/36.png)

The bigger number wins.

### Nesting doesn't prevent collapsing

Alright, here's where it starts to get weird.

```html
<style>
  p {
    margin-top: 48px;
    margin-bottom: 48px;
  }
</style>

<div>
  <p>Paragraph One</p>
</div>
<p>Paragraph Two</p>
```

We're dropping our first paragraph into a containing `<div>`, but the margins will still collapse!

![image](/images/m1/37.png)

It turns out that many of us have a misconception about how margins work.

Margin is meant to increase the distance between siblings. It is not meant to increase the gap between a child and its parent's bounding box; that's what padding is for.

### Blocked by padding or border

You can think of padding/border as a sort of wall; if it sits between two margins, they can't collapse, because there's an obstruction in the way.

![image](/images/m1/38.png)

This visualization shows padding, but the same thing happens with border.

Even 1px of padding or border will cause margins not to collapse.

### Blocked by a gap

As we saw in Height Algorithms, a parent will "vacuum-seal" around a child. A 150px-tall single child will have a 150px-tall parent, with no pixels to spare on either side.

But what if we explicitly give our parent element a height? Well, that would create a gap underneath the child

![image](/images/m1/39.png)

### Blocked by a scroll container

If the parent element creates a scroll container, with a declaration like `overflow: auto` or `overflow: hidden`, it will disable margin collapse if the margins are on either side of the scroll container.

## Margins can collapse in the same direction

Margins can collapse in the same direction

![image](/images/m1/40.png)

```html
<style>
  .parent {
    margin-top: 72px;
  }

  .child {
    margin-top: 24px;
  }
</style>

<div class="parent">
  <p class="child">Paragraph One</p>
</div>
```

The child margin is getting “absorbed” into the parent margin.

## More than two margins can collapse

Margin collapse isn't limited to just two margins! In this example, 4 separate margins occupy the same space:

![image](/images/m1/41.png)

It's hard to see what's going on, but this is essentially a combination of the previous rules:

1. Siblings can combine adjacent margins (if the first element has margin-bottom, and the second one has margin-top)
2. A parent and child can combine margins in the same direction

example code:

```html
<style>
  header {
    margin-bottom: 10px;
  }

  header h1 {
    margin-bottom: 20px;
  }

  section {
    margin-top: 30px;
  }

  section p {
    margin-top: 40px;
  }
</style>

<header>
  <h1>My Project</h1>
</header>
<section>
  <p>Hello World</p>
</section>

```

![image](/images/m1/42.png)

The space between our `<header>` and `<section>` has 4 separate margins competing to occupy that space!

here the explanation

1. The `header` wants space below itself
2. The `h1` in the `header` has bottom margin, which collapses with its parent
3. The `section` below the `header` wants space above itself
4. The `p` in the section has top margin, which collapses with its parent

Ultimately, *the paragraph has the largest cumulative margin*, so it wins, and `40px` separates the `header` and `section`.

### Negative margins

As we saw when we looked at Margins, a negative margin will pull an element in the **opposite direction**. A sibling with a negative `margin-top` might overlap its neighbor:

![image](/images/m1/43.png)

the negative margin quite similar to positive ones. The negative margin will share a space and that space determined by the most significant negative margin.

In this example, the elements overlap by 75px, since the more-negative margin (-75px) was more significant than the other (-25px).

![image](/images/m1/44.png)

so, what if we mixed the positive margin and negative margin ? in that case it will **added together**.

![image](/images/m1/45.png)

Why would we want to apply margins that have no effect?! Well, sometimes you don't control one of the two margins. Maybe it comes from a legacy style, or it's tightly ensconced in a component. By applying an inverse negative margin to the parent, you can "cancel out" a margin.

### Multiple positive and negative margins

What if we have multiple margins competing for the same space, and some are negative?

If there are more than 2 margins involved, the algorithm looks like this:

1. All of the positive margins collapse together (eg. 10px and 50px collapse into a single 50px margin).
2. All of the negative margins collapse together (eg. -20px and -30px collapse into a single -30px margin).
3. Add those two numbers together (50px + (-30px) = 20px).
