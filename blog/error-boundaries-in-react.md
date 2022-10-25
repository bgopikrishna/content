---
draft: false
date: 2022-10-25T10:30:00+05:30
title: How to use error boundaries in react?
tags:
- React
description: Error boundaries are the react components that help us to catch errors
  in the render phase. By using Error Boundaries we can also show a fallback UI if
  something goes wrong (Like an uncaught error).
cover: "/uploads/cover-1.png"
coverImageCredits: ''

---
## What are error boundaries?

Error boundaries are unique react components that help us catch the errors in a React Application. Using Error Boundaries, we can also show a fallback UI if something goes wrong (Like an uncaught error).

In plain English, Error boundaries are like try-catch blocks for the UI with some limitations (We'll see them in the latter part of this article).

For example, if a component failed to render content due to some case not being handled or you are trying to access a key in the object which doesn't exist etc.

If it's wrapped with an Error boundary when the error happens, we can render a fallback UI instead of the application crashing.

We'll see this in action during the implementation of an example application.

## How to create and use error boundaries?

To create an Error Boundary component, it should be

* A Class Component
* It should implement either (or both) of the lifecycle methods.
  * `getDerivedStateFromError()` - This static lifecycle method allows us to update the state when an error happens. Using this, we can show the fallback UI.
  * `componentDidCatch()` - This is another life cycle method we can use to log the error information to application monitoring services like Prometheus or sentry.

## Example

We will build a simple application that shows a list of superheroes and their powers. On selecting a particular hero, we're going to show his superpowers.

It looks something like this. You will find the completed application at the end of this article.

![React Application](/uploads/screenshot-2022-04-13-at-9-56-28-am.png)

Enough talk, Let's get our hands dirty.

We are going to use `create-react-app`. Open your terminal and run.

```sh
npx create-react-app hero-powers
```

Now navigate to the folder. Using your code editor, replace the contents of `src/App.js` with the following code.

```javascript
    import { useState } from 'react';
    
    const HEROES = [
      { id: 11, name: 'Dr Nice', power: ['Laser Eyes', 'Wall Crawling'] },
      { id: 12, name: 'Narco', power: ['Portal Generation', 'Solar Form'] },
      { id: 13, name: 'Bombasto', power: null },
    ];
    
    const HeroDetails = ({ heroData }) => {
      const { name, power } = heroData;
      return (
        <div className='card is-shadowless'>
          <div className='card-content'>
            <div className='media'>
              <div className='media-content'>
                <p className='title is-4'>{name}</p>
              </div>
            </div>
    
            <div className='content'><strong>Powers</strong> - {power.join(', ')}</div>
          </div>
        </div>
      );
    };
    
    function App() {
      const [selectedHero, setSelectedHero] = useState(HEROES[0]);
    
      return (
        <div className='is-flex is-flex-direction-column'>
          <h1 className='title is-3'>Heros and their powers</h1>
          <div className='column has-background-grey-lighter'>
            <div className='panel is-shadowless'>
              {HEROES.map((heroData) => (
                <span
                  className={`panel-block is-clickable ${
                    heroData.id === selectedHero.id ? 'is-active' : ''
                  }`}
                  key={heroData.id}
                  role='button'
                  onClick={() => setSelectedHero(heroData)}
                >
                  {heroData.name}
                </span>
              ))}
            </div>
          </div>
          <div className='column has-background-grey-lighter'>
              <HeroDetails heroData={selectedHero} />
          </div>
        </div>
      );
    }
    
    export default App;
```

Start the application using

```sh
npm start
```

If you look at the code written, we have a `App` component and a `HeroDetails` component.

`App` the Component renders the list of superheroes and `HeroDetails`. `HeroDetails` The Component renders the powers of a selected superhero.

To keep things simple, we are storing the hero's data in a variable called `HEROES`. But in a real-world application, this data might be coming from an API or other external sources.

Now, if you observe, when selecting `Bambasto` , the application is getting crashed and going blank. If you look at the code, For  `Bambasto` the powers are `null` (check `HEROES` variable). We did that intentionally to simulate the error.

Create a file with the name `ErrorBoundary.js`

Please copy the following contents and paste them there.

```javascript
import { Component } from "react";
 
 export default class ErrorBoundary extends Component {
   state = {
     hasError: false
   };
 
   static getDerivedStateFromError(error) {
     return {
       hasError: true
     };
   }
 
   componentDidCatch(error, errorInfo) {
     this.logErrorToService(error, errorInfo);
   }
 	
 	// logger service can be your own application monitoring tool etc
   logErrorToService(error, errorInfo) {
     console.error(error, errorInfo);
   }
 
   render() {
     const { fallback, children } = this.props;
     const { hasError } = this.state;
 
     if (hasError) {
       return fallback;
     }
 
     return children;
   }
 }
```

If you look at the code, the error boundary we created has two props

1. `fallback` - Fallback UI to show when the component crashes
2. `children` - The component which needs to wrap with an error boundary

Whenever a crash happens in our application, it gets caught by the nearest error boundary, and the fallback UI is rendered

`getDerivedStateFromError`  lifecycle method allows us to update the state to `hasError: true`

Whereas with the `componentDidCatch` lifecycle method, we can log the error to the error service. Currently, we are just logging it into the console. But ideally, we should log errors to some application monitoring service.

We can also see the whole component stack from where the error originated using `errorInfo.componentStack` .

Now let's see that in action. Wrap the `HeroDetails` with an `ErrorBoundary`

```javascript
    <ErrorBoundary
      fallback={
        <div className='message is-danger'>
          <div className='message-body'>Something went wrong</div>
        </div>
      }
    >
      <HeroDetails heroData={selectedHero} />
    </ErrorBoundary>;
```

Now on selecting `Bambasto` again. Instead of the application getting crashed, we can see the fallback UI.

![Error boundary with fallback UI](/uploads/screenshot-2022-04-14-at-10-56-27-pm.png)

## Limitations

Remember, there are some limitations to Error Boundaries. The Error Boundaries won't catch errors that happen for the following reasons.

* In event handlers (e.g., `onClick`, `onChange`, `onBlur` etc.)
* In `setTimeout` , `setInterval` or `requestAnimationFrame` callbacks
* During Server-side rendering (SSR)
* And errors caused by the error boundary itself

## Codesandbox demo

<iframe
src="https://codesandbox.io/embed/react-error-boundaries-5nzju5?			fontsize=14&hidenavigation=1&theme=dark&view=preview"
style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
title="React Error Boundaries"
allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts" />