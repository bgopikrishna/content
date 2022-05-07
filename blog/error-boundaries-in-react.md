---
draft: true
date: 2022-05-17T00:00:00+05:30
title: Error Boundaries in React
tags:
- React
description: Desc Goes Here
cover: "/uploads/cover-1.png"
coverImageCredits: ''

---
# Error Boundaries in React

## Objectives

* What are error boundaries in react?
* How to use them
* A simple demo
* Usage with Sentry

## What are error boundaries?

Error boundaries are the special react components that help us to catch the errors in the render phase. By using Error Boundaries we can also show a fallback UI if something goes wrong (Like an uncaught error).

Basically in plain English, Error boundaries are like try-catch blocks for the UI(although they have some limitations which we will look at at the end of the article).

For example, if a component failed to render content due to a failed API call or you’re trying to access a key in the object which doesn’t exist etc.

If it’s wrapped with an Error boundary, when the error happens instead of the page going completely blank, we can render a fallback UI.

We'll see this in action during the implementation of an example

## How to create and use them?

To create an Error Boundary component, it must be

* A Class Component
* It must implement either (or both) of the lifecycle methods
  * `static getDerivedStateFromError()` - This lifecycle method allows us to update the state when an error happens.
  * `componentDidCatch()` - This is another life cycle method that we can use to log the information to application monitoring services like sentry etc.

## Example

We’re going to build a simple app that shows a list of superheroes and their powers. On selecting a particular hero, we’re going to show his powers.

It looks something like this.

![Screenshot 2022-04-13 at 9.56.28 AM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/aac004b3-1e40-4cc9-b879-b57a74bd690e/Screenshot_2022-04-13_at_9.56.28_AM.png)

Enough talk, Let’s get our hands dirty.

Open your terminal and run

   ```sh
   npx create-react-app hero-powers
   ```
    

Now replace the contents of `src/App.js` with the following code.
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
    

If you look at the code written, we have an `App` component and a `HeroDetails` component.

App component renders the list of superheroes. Whereas `HeroDetails` renders the powers of the selected superhero.

To keep things simple, we are storing hero's data in a variable called `HEROES`. But in a real-world scenario, this data might be coming from an API.

Now if you observe, when selecting `Bambasto` the page is getting crashed and going blank. If you look at the code, for `Bambasto` the powers are `null` (check `HEROES` variable). We did that intentionally to simulate the error.

Now let’s wrap it in with an error boundary.

Create a file with the name `ErrorBoundary.js`

Copy the following contents and paste them there.

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
        this.loggErrorToService(error, errorInfo);
      }
    	
    	// logger service can be your own application monitoring tool etc
      loggErrorToService(error, errorInfo) {
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

1. `fallback` - Fallback UI to show if the component crashed
2. `children` - The component which needs to wrap with an error boundary

Whenever a crash happens in our application, it gets caught by the nearest error boundary.

`getDerivedStateFromError` lifecycle method is allowing use to update the state and set `hasError: true`

In the `componentDidCatch` lifecycle method, we can log the error to the error service. Currently, we are just logging it into the console. But ideally, it should be logged to some application monitoring service.

We can also see the whole component stack from where the error originated using `errorInfo.componentStack` .

Now let’s see that in action, wrap the `HeroDetails` with an `ErrorBoundary`
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

Now on selecting `Bambasto` again instead of the page going completely blank, we can see the fallback UI

![Screenshot 2022-04-14 at 10.56.27 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fa2678de-1e76-4957-b49a-9f520acdace2/Screenshot_2022-04-14_at_10.56.27_PM.png)

## Limitations

These are the things that are not caught by error boundary

* Event handlers (e.g., `onClick`, `onChange`, etc.)
* `setTimeout` or `requestAnimationFramecallbacks`
* Server-side rendering (SSR)
* And errors caused by the error boundary itself