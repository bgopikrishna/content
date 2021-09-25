---
title: 'Getting started with CSS variables'
tags: ['CSS', 'JavaScript']
date: '2019-11-13'
path: '/blog/css-variables'
description: 'If you have used SASS or other CSS preprocessors, you might have already used Variables. CSS variables (also referred as custom properties) are native implementation of them in the browser.'
cover: /assets/css-variables-cover.jpg
coverImageCredits: '<span>Photo by <a href="https://unsplash.com/@pankajpatel?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Pankaj Patel</a> on <a href="https://unsplash.com/s/photos/css-variable?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>'     

---

If you have used SASS or other CSS preprocessors, you might have already used Variables. CSS variables (also referred as custom properties) are native implementation of them in the browser.

CSS variables needs to be declared with `--` prefix to the name.

```css
:root {
  --primary-color: #eaefed;
  --text-color: #001100;
}
```

When we want to access them, we just need to use the `var` function. The first argument of the `var` function is name of the variable. If variable is invalid, we can pass the fallback value as the second argument.

```css
.link {
  border: 1px solid var(--primary-color);
  color: var(--text-color, #111000);
  /* if --text-color is invalid,fallback value #111000 is used */
}
```

### Scope of the Variable

The scope of the CSS variable is the scope of the selector it is defined. For global scope we can either declare them in `:root` or `body` selector.

For example, lets declare a variable named `--btn-danger-color` in a class `btn` . If you try to access the variable `--btn-danger-color` in another class out of its scope, it will be invalid. Because scope of `--btn-danger-color` is only in `btn` class

```css
.btn {
  --btn-danger-color: #ff4136;
  background-color: var(--btn-danger-color);
}
.link {
  background-color: var(--btn-danger-color, green);
  /* Background color is green because --btn-danger is invalid */
}
```

### Accessing and Modifying CSS Variables with JavaScript

To access or modify CSS variables via JavaScript, we can use two methods

- getPropertyValue
- setProperty

### Getting the CSS Variable Value

To get the value of the CSS variable, we can use `getPropertyValue` method

```js
const btn = document.querySelector('.btn');

let btnDangerBg = getComputedStyle(element).getPropertyValue(
  '--btn-danger-color'
);

console.log(btnDangerBg); // #FF4136 - value of the css variable defined in .btn class in CSS
```

To get the CSS variables declared in `:root`, you can use `document.querySelector(':root');`

### Changing the CSS Variable Value

To set the value of the CSS variable from JavaScript we can use `setProperty` method.

```javascript
element.style.setProperty('--variable-name', value);
```

Lets write a small function to change the `Button` background colour to purple by using `setProperty` method.

```javascript
function changeColorToPurple() {
  //Set the CSS variable value
  btn.style.setProperty('--btn-danger-color', 'purple');
}
```

Don't forget to link it with HTML.

```html
<button class="btn" onclick="changeColorToPurple()">Change Color</button>
```

### Demo

I'm adding an embedded [repl](https://repl.it/@bgopikrishna/CssVariables) for the above code, So that you can play with it.

<iframe height="600px" width="100%" src="https://repl.it/@bgopikrishna/CssVariables?lite=true" scrolling="no" frameborder="no" allowtransparency="true" allowfullscreen="true" sandbox="allow-forms allow-poInter-lock allow-popups allow-same-origin allow-scripts allow-modals"></iframe>

Thank You.

### Reference

[CSS custom properties ](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties) - MDN

[Every thing you need to know about css variables](https://www.freecodecamp.org/news/everything-you-need-to-know-about-css-variables-c74d922ea855/)
