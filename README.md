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

Example 1:

```javascript
const astrosUrl = 'http://api.open-notify.org/astros.json';
const wikiUrl = 'https://en.wikipedia.org/api/rest_v1/page/summary/';
const peopleList = document.getElementById('people');
const btn = document.querySelector('button');

function getJSON(url) {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        xhr.open('GET', url);
        xhr.onload = () => {
            if (xhr.status === 200) {
                let data = JSON.parse(xhr.responseText);
                resolve(data);
            } else {
                reject( Error(xhr.statusText) );
            }
        };
        xhr.onerror = () => reject( Error('A network error occurred.') );
        xhr.send();
    });
}

function getProfiles(json) {
    const profiles = json.people.map( person => {
        return getJSON(wikiUrl + person.name);
    }); 
	return profiles;
}

function generateHTML(data) {
  const section = document.createElement('section');
  peopleList.appendChild(section);
  section.innerHTML = `
    <img src=${data.thumbnail.source}>
    <h2>${data.title}</h2>
    <p>${data.description}</p>
    <p>${data.extract}</p>
  `;
}

btn.addEventListener('click', (event) => {
  getJSON(astrosUrl)
    .then(getProfiles)
    .then( data => console.log(data) )
    .catch( err => console.log(err) )
    
  event.target.remove();
});
```

