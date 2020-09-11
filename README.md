#### Ecma Script 2015

1. Arrow functions:

   

```js
function Person(){
  this.age = 0;

  setInterval(() => {
    this.age++; // |this| properly refers to the Person object
  }, 1000);
}

var p = new Person();
```

```javascript
var elements = [
  'Hydrogen',
  'Helium',
  'Lithium',
  'Beryllium'
];

// This statement returns the array: [8, 6, 7, 9]
elements.map(function(element) {
  return element.length;
});

// The regular function above can be written as the arrow function below
elements.map((element) => {
  return element.length;
}); // [8, 6, 7, 9]

// When there is only one parameter, we can remove the surrounding parentheses
elements.map(element => {
  return element.length;
}); // [8, 6, 7, 9]

// When the only statement in an arrow function is `return`, we can remove `return` and remove
// the surrounding curly brackets
elements.map(element => element.length); // [8, 6, 7, 9]

// In this case, because we only need the length property, we can use destructuring parameter:
// Notice that the `length` corresponds to the property we want to get whereas the
// obviously non-special `lengthFooBArX` is just the name of a variable which can be changed
// to any valid variable name you want
elements.map(({ length: lengthFooBArX }) => lengthFooBArX); // [8, 6, 7, 9]

// This destructuring parameter assignment can also be written as seen below. However, note that in
// this example we are not assigning `length` value to the made up property. Instead, the literal name
// itself of the variable `length` is used as the property we want to retrieve from the object.
elements.map(({ length }) => length); // [8, 6, 7, 9] 
```



#### Promises

Always in one of three states.

1. Pending - default
2. Fulfilled - completed succesfully
3. Rejected - failed

Create using Promise() constructor.

Define what happens when promise is fulfilled or rejected.

Consume the value of a fulfilled promise (then() on Promise object), or provide a rejection reason for a rejected promise (catch() on Promise object).

Using fulfilled promises:

```javascript
const breakfastPromise = new Promise( () => {
    setTimeout(() => {
        resolve('Your order is ready. Come and get it!');
    }, 3000);
});

console.log(breakfastPromise);
// then return fulfilled value val
breakfastPromise.then( val => console.log(val) );
```

Catching rejected promises:

```javascript
const breakfastPromise = new Promise( () => {
    setTimeout(() => {
        reject( Error('Oh no! There was a problem with your order.') );
    }, 3000);
});

console.log(breakfastPromise);
// then return fulfilled value
// catch return rejected value
breakfastPromise
    .then( val => console.log(val)
    .catch( err => console.log(err) ) );
```

It is possible to chain promises and pass the return value of one to input value of another:

```javascript
function addFive(n) {
  return n + 5;
}
function double(n) {
  return n * 2;
}
function finalValue(nextValue) {
  console.log(`The final value is ${nextValue}`);
}

const mathPromise = new Promise( (resolve, reject) => {
  setTimeout( () => {
    // resolve promise if 'value' is a number; otherwise, reject it
    if (typeof value === 'number') {
      resolve(value);
    } else {
      reject('You must specify a number as the value.')
    }
  }, 1000);
});

const value = 5;
mathPromise
  .then(addFive)
  .then(double)
  .then(finalValue)
  .catch( err => console.log(err) )

// The final value is 20
```

### Example 1:

```javascript
const astrosUrl = 'http://api.open-notify.org/astros.json';
const wikiUrl = 'https://en.wikipedia.org/api/rest_v1/page/summary/';
const peopleList = document.getElementById('people');
const btn = document.querySelector('button');

function getProfiles(json) {
    const profiles = json.people.map( person => {
        const craft = person.craft;
        // method fetch() gets data from endpoint in form of json
        // it is possible to access data related to request as well as data itself
        return fetch(wikiUrl + person.name)
        		.then( response => response.json() )
        		.then( profile => {
            		return { ...profile, craft }
        		})
        		.catch( err => console.log('Error fetching Wiki: ', err) )
    }); 
    // this is returning array or promises
	return profiles;
    // this is waiting for all promises to resolve
    // and returns a Promise containing as value an array of returned objects (data)
    // it allways follow the order in which the promises have been passed
    return Promise.all(profiles);
}

function generateHTML(data) {
  data.map( person => {
    const section = document.createElement('section');
    peopleList.appendChild(section);
    section.innerHTML = `
    <img src=${person.thumbnail.source}>
	<span>${person.craft}</span>
    <h2>${person.title}</h2>
    <p>${person.description}</p>
    <p>${person.extract}</p>
    `;
  });
}

btn.addEventListener('click', (event) => {
  event.target.textContent = "Loading...";
  fetch(astrosUrl)
    .then( response => response.json() )
    .then(getProfiles)
    .then(generateHtml)
    .catch( err => {
      peopleList.inngerHTML = '<h3>Something went wrong!</h3>';
      console.log(err)
  } )
    .finally( () => event.target.remove() )
});
```

### Async / await.

Await:

1. Pauses the execution of an async function and waits for the resolution of a promise.
2. Resumes execution and returns the resolved value.
3. Pausing execution is not going to cause blocking behavior.
4. Valid only inside functions marked async.

Async function always returns a promise.

That promise resolves with the value returned by the async function, or rejects with an error thrown from within the function.

```javascript
async function fetchData(url) {
    const response = await fetch(url);
    const data = await response.json();
    
    return data;
}

// to use data (Promise) returned by async funtion it needs to be passed to a then method
fetchData('dog.ceo/dog-api/breeds')
	.then( data => console.log(data) )
```

### Example:

```javascript
async function getPeopleInSpace() {
    const peopleResponse = await fetch(url);
    const peopleJSON = await peopleResponse.json();
    
    const profiles = peopleJSON.people.map( async (person) => {
        const craft = person.craft;
        const profileResponse = await fetch(wikiUrl + person.name);
        const profileJSON = await profileResponse.json();
        
        return { ...profileJSON, craft };
    })
    
    return Promise.all(profiles);
}

btn.addEventListener('click', (event) => {
    event.target.textContent = 'Loading...';
    
    getPeopleInSpace(astrosUrl)
    	.then(generateHTML)
    	.finally( () => event.target.remove() )
})
```

### Exceptions and error handling.

```javascript
// Handle all fetch requests
async function getJSON(url) {
    try {
        const response = await fetch(url);
        return await response.json();
    } catch (error) {
        throw error;
    }
}

async function getPeopleInSpace() {
    const peopleJSON = await getJSON(url);
    
    const profiles = peopleJSON.people.map( async (person) => {
        const craft = person.craft;
        const profileJSON = await getJSON(wikiUrl + person.name);
        
        return { ...profileJSON, craft };
    })
    
    return Promise.all(profiles);
}


btn.addEventListener('click', (event) => {
    event.target.textContent = 'Loading...';
    
    getPeopleInSpace(astrosUrl)
    	.then(generateHTML)
    	.catch( e => {
        	peopleList.innerHTML = '<h3>Something went wrong!</h3>';
        	console.error(e);
    	})
    	.finally( () => event.target.remove() )
})
```

## Web APIs

**The call stack handles asynchronous tasks in a different way than synchronous tasks**. Whenever the JavaScript environment encounters code that needs to run and execute at a later time, like a `setTimeout()` callback or a network request, that code is handed off to a special Web API that processes the async operation. Meanwhile, the call stack moves on to other tasks then revisits the async task when it's ready to be  dealt with.

It's important to note that **the asynchronous behavior of JavaScript does not come from the JavaScript engine itself**. JavaScript engines like Chrome's V8, for instance, do not have  asynchronous capabilities. Asynchronicity actually comes from the  runtime environment. Runtime environments *(like web browsers and Node)* provide APIs that let JavaScript run tasks asynchronously.

For example, async development in the browser (which is the focus of  this course) happens in a number of places using Web APIs like `setTimeout()`, XMLHttpRequest (XHR), Fetch API, and the DOM event API.

Where does the async callback go before being pushed onto the call  stack? It doesn't immediately go back into the call stack. Instead the  callback gets pushed into something called the **callback queue**.

## How the Callback Queue Works

You can think of the callback queue as a holding area for all the  callbacks that are waiting to be put on the call stack and eventually  executed. Only callbacks for (non-blocking) operations that are  scheduled to complete execution at a later time like `setTimeout`, `setInterval` and AJAX requests, for example, are added to the callback queue

The web APIs add each async task to the callback queue where they wait  to be pushed onto the call stack and executed one at a time. 

## The Event Loop

The event loop has one important job: monitor the call stack and  callback queue. Anytime the call stack is empty, the event loop takes  the first task from the callback queue and pushes it onto the call  stack, and runs it.

## Default Parameters

```javascript
function greet(name = 'Piotr', timeOfDay = 'Day') {
    console.log(`Good ${timeOfDay}, ${name}!`);
}

// we can pass undefined to function with defined default parameters
// and the function will use default value for this parameter
greet(undefined, 'Afternoon');
```

## Rest Parameters and Spread Operator

#### Rest Parameter

```javascript
// rest parameter is three dots ... before variable params
// variable name can be any name, it doesn't have to be params
// Rest Operator have to be assigned to the last function argument
function myFunction(name, ...params) {
    console.log(name, params);
}

// when calling function with rest parameter we can call it with more arguments
// in this example I call 2 argument function using 6 arguments
// rest parameter makes argument ...params contain an array of all arguments
// that have been passed after the first one
myFunction('Piotr', 1, 2, 3, 4, 'Dawid')
```

#### Spread Operator

```javascript
'use strict';

// spread operator enables arrays to be nested
// anything added to oldFlavors or newFlavors will be added to inventory as well
const oldFlavors = ['Chocolate', 'Vanilla'];

const newFlavors = ['Strawberry', 'Mint Chocolate Chip'];

const inventory = ['Rocky Road', ...oldFlavors, 'Neapolitan', ...newFlavors];

console.log(inventory);
```

```javascript
'use strict';

function myFunction(name, iceCreamFlavor) {
    console.log(`${name} really likes ${iceCreamFlavor} ice cream.`)
}

let args = ['Piotr', 'Chocolate'];

// it can also be used to get arguments for array and pass it to a function call
myFunction(...args)
```

## De-structuring

```javascript
'use strict';

let toybox = { item1: 'car', item2: 'ball', item3: 'frisbee'};

// de-structuring lets you take pieces of object and place them in a variable
let {item1, item2} = toybox;

console.log(item1, item2);

// it can also create new variables inside parentheses
let {item3: disc} = toybox;

console.log(disc);
```

```javascript
'use strict';

let widgets = ['widget1', 'widget2', 'widget3', 'widget4', 'widget5'];

let [a, b, c, ...d] = widgets;

console.log(d);
```

```javascript
function getData({ url, method = 'post' } = {}, callback) {
    callback(url, method);
}

getData({ url: 'myposturl.com' }, function (url, method) {
    console.log(url, method);
});

getData({ url: 'myputurl.com', method: 'put' }, function (url, method) {
    console.log(url, method);
});
```

```javascript
'use strict';

let parentObject = {
    title: 'Super Important',
    childObject: {
        title: 'Equally Important'
    }
}

let { title, childObject: { title: childTitle } } = parentObject

console.log(childTitle);
```

