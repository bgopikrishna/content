---
draft: true
date: 2023-02-26T16:30:00+05:30
title: IndexedDB on steroids using Dexie.js
tags:
- IndexedDB
description: Dexie.js provides a straightforward and simplified process of creating
  databases, storing data, updating data and database migrations, etc., over the top
  of indexDB.
cover: "/uploads/stencil-facebook-cover.png"
coverImageCredits: ''

---
## What is IndexedDB

IndexedDB is a client-side, NoSQL database that allows web apps to store and retrieve data. As the data is stored locally in the browser, it's available even offline. 

The main advantage of indexedDB over local storage was we could even store files and blobs (like images, videos, etc.) along with objects.

## Dexie.js

Working with low-level indexedDB API directly is a lot of work. You can check out this [article](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB) on MDN about using indexedDB API directly. 

To make things easier, [Dexie.js](https://dexie.org/) provides a straightforward and simplified process of creating databases, storing data, updating data and database migrations, etc., over the top of indexedDB.

## Using Dexie.js

We will build a simple to-do app to understand the basics of dexie.js, like creating a database and performing CRUD operations.

### Setting up the app

Let's create an `index.html` file with the following contents.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bulma@0.9.4/css/bulma.min.css">
</head>
<body>
    <div class="content">
        <ul id="todoList">
        </ul>
    </div>
    <div>
        <form id="addTodoForm" class="column form">
            <div class="field">
                <label class="label" for="todo">Todo</label>
                <div class="control">
                    <input required class="input" name="todo" id="todo" type="text" placeholder="Todo">
                </div>
            </div>
            <div class="field is-grouped">
                <div class="control">
                    <button class="button is-link">Create todo</button>
                </div>
            </div>
        </form>
    </div>
</body>
</html>
```

Now open the html file using an HTTP server of your choice. I'm using the [Live Server](https://github.com/ritwickdey/vscode-live-server) VS Code extension. You will see something like this.

![HTML boiler plate](/Users/gk/Documents/blog/uploads/dexiejs-boilerplate-html.png)

For styling, we are using [Bulma](https://bulma.io/) CSS. Todos will be added by typing in the input box. Nothing works now, as we still need to add the javascript.

### Adding Dexie.js

Create a javascript file named `main.js,` where we will write the business logic for the app. We will not use a module bundler like webpack or parcel.js to keep things simple. Instead, we are going to use `Dexie` directly from the CDN.

Now include both of them in the html file.

```html {33-34}
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bulma@0.9.4/css/bulma.min.css">
</head>

<body>
    <div class="content">
        <ul id="todoList">
        </ul>
    </div>

    <div>
        <form id="addTodoForm" class="column form">
            <div class="field">
                <label class="label" for="todo">Todo</label>
                <div class="control">
                    <input required class="input" name="todo" id="todo" type="text" placeholder="Todo">
                </div>
            </div>
            <div class="field is-grouped">
                <div class="control">
                    <button class="button is-link">Create todo</button>
                </div>
            </div>
        </form>
    </div>
    <script src="https://unpkg.com/dexie/dist/dexie.js"></script>
    <script src="./main.js"></script>
</body>

</html>
```



### Capturing the user input

In `main.js,` we will add event listeners to capture the user input.

```javascript
const formElement = document.getElementById('addTodoForm');
const todoInput = document.getElementById('todo');
const todoList = document.getElementById('todoList');

formElement.addEventListener('submit', (e) => {
  e.preventDefault();
  const todoName = todoInput.value;    
  
  console.log('Todo to be created', todoName)
  
  todoInput.value = ''   // reset input
});

```

* `formElement` - Form where we receive user inputs to create a todo
* `todoInput` - Input where the user enters the todo
* `todoList` - Element where we are going to list all the todos

Currently, on submitting the form, we are logging the to-do to the console. But ideally, what we want is to create the to-do in the database and show it on the UI. We are going to use Dexie.js for that.

### Creating a database and tables

We can create an indexedDB database with dexie.js using. 

```javascript
const db = new Dexie('databaseName')
```

You can add tables using the `db` instance `stores` method by passing an object where the table name is the key and values are the columns that are comma separated.

```javascript
db.version(1).stores({
  tableOne: `++id, col3, col3`,
  tableTwo: `++id, col3, col3`,
});
```

You can maintain multiple versions of the database with the version number. It is similar to the [indexedDB version](https://developer.mozilla.org/en-US/docs/Web/API/IDBDatabase/version).

When declaring columns

* The first item will be the primary key.
*  `++` prefix makes it an auto-incremented primary key
* `&`  prefix makes it a unique column

Let's create a database called `TodoDatabase` 

We will store the todos in the `todo` table with the following columns.

* `id` - Primary key
* `name` - Name of the todo
* `completed` - Todo status

```javascript {1-9}
const db = new Dexie('TodoDatabase');

db.version(1).stores({
  todo: `++id, name, completed`,
});

const formElement = document.getElementById('addTodoForm');
const todoInput = document.getElementById('todo');
const todoList = document.getElementById('todoList');

formElement.addEventListener('submit', (e) => {
  e.preventDefault();

  const todoName = todoInput.value;
    
  console.log('Todo to be created', todoName)

  // reset input
  todoInput.value = ''
});

```

### Inserting into database

Let's insert the todos into the `todo` table. Insert the objects into the table using [Table.add()](https://dexie.org/docs/Table/Table.add()).

```javascript {12-21, 28}
const db = new Dexie('TodoDatabase');

db.version(1).stores({
  todo: `++id, name, completed`,
});

const formElement = document.getElementById('addTodoForm');
const todoInput = document.getElementById('todo');
const todoList = document.getElementById('todoList');


async function createTodo(todoName) {
  try {
    await db.todo.add({
      name: todoName,
      completed: false,
    });
  } catch (error) {
    console.error(error)
  }
}

formElement.addEventListener('submit', (e) => {
  e.preventDefault();

  const todoName = todoInput.value;
    
  createTodo(todoName);

  // reset input
  todoInput.value = ''
});

```

On submitting the form, we pass the todo to the `createTodo ` function.

`createTodo` function inserts the new todo using `Table.add()` method. We don't need to pass the `id` as it's auto-incremented. 

Now if you add the todos, the new todos will be inserted into the `todo` table. You can check them in the dev tools.

![IndexedDB using dexie js in developer tools](/Users/gk/Documents/blog/uploads/Screenshot%202023-01-31%20at%2010.17.54%20PM.png)



### Showing created todos on the UI

Even though the todos are being created, they are not visible on the UI. To track the changes when they are being added, updated, or deleted and show them on the UI, we will use [liveQuery](https://dexie.org/docs/liveQuery()).

 If you pass `liveQuery`,  a database query callback like `Table.where` or `Table.toArray` . Whenever database changes like creation, updation, or deletion affect the result of your query, the passed callback will be called, and the result (the return value) is emitted by observable. 

Think of this like an event listener.

 Creating a to-do observable to which we can subscribe and update the UI whenever todos are added or updated.

```javascript
const todoObservable = Dexie.liveQuery(() => db.todo.toArray());
```

Subscribing to the observable

```javascript {23, 25-41}
const db = new Dexie('TodoDatabase');

db.version(1).stores({
  todo: `++id, name, completed`,
});

const formElement = document.getElementById('addTodoForm');
const todoInput = document.getElementById('todo');
const todoList = document.getElementById('todoList');


async function createTodo(todoName) {
  try {
    await db.todo.add({
      name: todoName,
      completed: false,
    });
  } catch (error) {
    console.error(error)
  }
}

const todoObservable = Dexie.liveQuery(() => db.todo.toArray());

todoObservable.subscribe({
  next: (result) => {
    // first remove existing items
    todoList.innerHTML = ''
    
    result.forEach((todo) => {
      let newTodo = document.createElement('li');
      newTodo.dataset.id = todo.id;
      newTodo.dataset.completed = todo.completed;
      newTodo.innerText = todo.name;

      console.log(newTodo);
      todoList.appendChild(newTodo);
    });
  },
  error: (error) => console.error(error),
});


formElement.addEventListener('submit', (e) => {
  e.preventDefault();

  const todoName = todoInput.value;
    
  createTodo(todoName);

  // reset input
  todoInput.value = ''
});



```

When subscribing to the observable, we need to pass the `next` and `error` callback.

* `next` - whenever the callback is executed from `liveQuery` the result will be passed to `next`
* `error` - To handle errors for callbacks etc.

In our case, we are looping over all todos and rendering them in the UI using `Array.forEach`. Now whenever you try to add the new todo, it will be rendered in the UI

### Updating the data in the database

To mark the to-do as complete, the user clicks on the to-do item, and then the completed status will be changed to `true` and vice versa. We can update the `completed` column for the to-do using [Table.update()](https://dexie.org/docs/Table/Table.update()). `Table.update()` accepts the primary key value as the first argument and the data to be updated as the second.

```javascript
 db.todo.update(primaryKeyValue, dataToBeUpdated)
```

Let's create a function called `toggleTodoCompleteStatus` which toggles the completed to-do status.

```javascript {26-39, 51}
const db = new Dexie('TodoDatabase');

db.version(1).stores({
  todo: `++id, name, completed`,
});

const formElement = document.getElementById('addTodoForm');
const todoInput = document.getElementById('todo');
const todoList = document.getElementById('todoList');


async function createTodo(todoName) {
  try {
    await db.todo.add({
      name: todoName,
      completed: false,
    });
  } catch (error) {
    console.error(error)
  }
}

const todoObservable = Dexie.liveQuery(() => db.todo.toArray());


async function toggleTodoCompleteStatus(event) {
  const { id, completed } = event.target.dataset;
  try {
    const isUpdated = await db.todo.update(parseInt(id), {
      completed: !JSON.parse(completed), // to convert 'false` -> false etc
    });

    if (!isUpdated) {
      throw new Error('Error updating');
    }
  } catch (error) {
    console.error(error)
  }
}

todoObservable.subscribe({
  next: (result) => {
    // first remove existing items
    todoList.innerHTML = ''
    
    result.forEach((todo) => {
      let newTodo = document.createElement('li');
      newTodo.dataset.id = todo.id;
      newTodo.dataset.completed = todo.completed;
      newTodo.innerText = todo.name;
      newTodo.onclick = toggleTodoCompleteStatus;

      console.log(newTodo);
      todoList.appendChild(newTodo);
    });
  },
  error: (error) => console.error(error),
});


formElement.addEventListener('submit', (e) => {
  e.preventDefault();

  const todoName = todoInput.value;
    
  createTodo(todoName);

  // reset input
  todoInput.value = ''
});



```

We also attached this to the `onclick` method directly. The `toggleTodoCompleteStatus`  will take the to-do list HTML element and the completed status from the [dataset](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset) property of the HTML element.

All the changes are immediately reflected in the UI as we have already subscribed to `todoObservable`

### Exercise

Till now, we learned how to create a database, create a table, add data, and update data. As a small exercise, add delete todo functionality which deletes the todo item from the database by finding the proper methods from [dexiejs docs](https://dexie.org/docs/).

### Final Code



<iframe src="https://replit.com/@bgopikrishna/DexieJS-Tutorial?embed=true" width="600" height="400" />