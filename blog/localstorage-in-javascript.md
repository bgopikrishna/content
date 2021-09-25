---
title: 'How to use Local Storage in JavaScript?'
tags: ['JavaScript']
date: '2019-08-04'
path: '/blog/localstorage-in-javascript'
description: "Local storage allows us to store the data in the user's browser. The data stored will not be deleted even after the browser is closed."
cover: /assets/localstorage-cover.jpg
coverImageCredits: '<span>Photo by <a href="https://unsplash.com/@tracycodes?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Tracy Adams</a> on <a href="https://unsplash.com/s/photos/localstorage-javascript?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>'      

---

Local storage allows us to store the data in the user's browser. The data stored will not be deleted even after the browser is closed.

The data will be stored in the form of key/value pairs. Both keys and values are always strings.

### Storing the data

For example, we have a user whose preferences/settings needs to be stored in the `localStorage`. Let's create a `userPreferences` object with the user settings.

```javascript
const userPreferences = {
  darkMode: true,
  accentColor: 'red',
};
```

Now, we'll store it in the browser using `localStorage.setItem` method

```js
localStorage.setItem('userData', JSON.stringfy(userPreferences));
```

Here `userData` is key and `userPreferences` is the value. The reason we are using `JSON.stringfy` method is to convert the `userPreferences` object to a string.

### Retrieving the data

When we need to use the data, we can retrieve it using `localStorage.getItem` method by passing the key as the argument. The data returned will be in the form of a string, so we need to use `JSON.parse` method to convert it into an object.

```js
let userData = localStorage.getItem('userData'); //"{ darkMode: true, accentColor:'red'}"

//convert the string to an object
userData = JSON.parse(userData);

console.log(userData); // { darkMode: true, accentColor: 'red'}
```

### Clearing the data

If you want to clear all the data stored in the local storage, we need to use `localStorage.clear` method.

```js
localStorage.clear();
```

### Removing the particular data

If you want to remove data for a particular key, we need to use `localStorage.removeItem` method by passing key as the argument.

```js
localStorage.removeItem("userData").
```

### Session Storage

`sessionStorage` is same as `localStorage` . But the main difference is, the data stored will be cleared automatically when the user closes the web page. All the methods are also same.

### Limitations

Do not store any sensitive data in the local storage as it can be accessed by any one browsing the page. And there is no form of data protection.

### Example

If you're reading this on _[Dev.to](https://dev.to)_, Open the developer tools by pressing `ctrl(cmd) + shift + I` and go to **Application &rightarrow; Local Storage**. There you'll find how _[Dev.to](dev.to)_ using local storage for storing user details with the key `current_user`.

If you're reading this on my blog, repeat the above steps and you'll see how I'm using local storage for persisting the dark mode using the key `darkMode`.

Thank You.

### Reference

[LocalStorage - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)

[Session Storage - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
