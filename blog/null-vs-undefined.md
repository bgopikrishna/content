---
title: 'Null vs Undefined in JavaScript in 1 min'
tags: ['JavaScript']
date: '2019-07-23'
path: '/blog/null-vs-undefined'
description: 'I always had a small confusion between null and undefined in JavaScript.'
cover: /assets/null-vs-undefined-cover.jpg
coverImageCredits: '<span>Photo by <a href="https://unsplash.com/@casparrubin?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Caspar Camille Rubin</a> on <a href="https://unsplash.com/s/photos/javascript?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>'
---

I always had a small confusion between `null` and `undefined` in JavaScript.

Now, Let's dive into the topic.

When we declare a variable without assigning any value to it, its value will be `undefined` by default.

```js
let color;
console.log(color); //undefined
```

But when we assign `null` to a variable, we are explicitly assigning a "nothing" or "empty" value to it.

For example, we have a `userDetails` variable which stores the details of an user.
At first, it doesn't have any data, so we are assigning `null` to it.

```js
let userDetails = null;
```

Later we fill the `userDetails` variable with the response from our function `getUserDetails`. The function may be a call to an API or accessing `localStorage` for details etc. Here it’s just a simple function which returns an object.

```js
function getUserDetails() {
  return {
    userName: 'gk',
    id: '1',
  };
}

userDetails = getUserDetails();
console.log(userDetails); // {userName:"gk", id:"1"}
```

If the value is unknown at the time of variable definition, it's always best to use `null`.

Thank you.

### Reference

[undefined](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/undefined) - MDN
| [null](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/null) - MDN

[Javascript Grammer](https://twitter.com/js_tut)
