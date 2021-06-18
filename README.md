---

Searching Places

---

## Searching Places

Imagine you wanted to search places by typing the address into a input and showing all places in our JSON file that matched.

To do this you will need to create an input and handle the text entered. React has a special pattern for this called the controlled component pattern.

The controlled component pattern is built around state. State is a value stored inside of a component that the component tracks. Changes to state cause the component to Render. The idea is that somethings that change need to be reflected in the appearance of a component.

To make this feature work we need to work with a new React concept: **state**.

**State** is a value that is held by a component. When state changes the component will render again. There are two ways to work with state.

**Hooks and Classes**

This solution will work with **hooks.**

> [action]
>
> To use state you'll import a special function. Edit `PLACESList.js`. Update the import statement at the top to include `{ useState }`.
>
```JS
import React, { useState } from 'react'
```
>

Create the input field for the search feature.

When I did this I had already setup the grid and hadn't thought of the how the form would display with the grid.

> [action]
>
> Edit the `PLACESList` component in `PLACESList.js`
>
```JS
function PLACESList() {
    const [ query, setQuery ] = useState('')
    ...
  return (
    <div className="PLACESList">
            <form>
                <input
                    value={query}
                    placeholder="search"
                    onChange={(e) => setQuery(e.target.value)}
                />
                <button type="submit">Submit</button>
            </form>
            {places}
    </div>
  )
}
```
>

If this is working you should be able to type in the field and see the value.

While it may seem like we are doing extra work here that isn't necessary, in reality it is and best practice.

Read more about how to handle forms with React here: https://reactjs.org/docs/forms.html

The work above has created a variable `query` that holds the value you entered into the field. In the next step you will use this value to filter the list of places displayed.

To filter the list you'll use `Array.filter()` simialr to `Array.map()` filter takes a callback function as a parameter and returns a new array.

The callback used with filter receives an item from the array and expected to return true if that it is going to be included in the returned array or false if not. For example:

```JS
const lessThanThree = [1,2,3,4,5,6].filter((n) => n < 3>)
```

So how to determine if an item should be included? You can use `String.includes()` to ask if a string includes another string. For example:

```JS
if ('surreptitious'.includes('it')) {
    console.log('I found it')
}
```

Note, case matters:

```js
'Ithica'.includes('it') // returns false!
```

With this in mind what do we search? The data we have has a couple fields that might contain things people would search for.

- title - Has the name of the of the location
- address - cotnains the address

the description might be used but it contains a lot of text that might provide false positives and slow our searches.

> [action]
>
> Edit `PLACESList.js`.
>
```JS
function PLACESList() {
    ...
    const places = data.filter(obj => obj.title.includes(query) || obj.address.includes(query)).map((obj, i) => {
        ...
    })
    ...
}
```


That's pretty long and probably hard to grok. Let's take it apart.

Without the callbacks we have:

```JS
const places = data.filter().map()
```

Okay so that's not so bad. We start with `data.filter()` which returns an array and we call map() on that array: `data.filter().map()` and the array returned here is assigned to places.

So what did we do filtered the data, then turned it into `PLACESSpace` components which are rendered later.

Filter takes a callback that reecives an item from data.

```JS
data.filter(obj => /* return true if obj.title includes query */).map((obj, i) => {
        ...
    })
```

That's only half correct, we also need to check the address.

```JS
data.filter(obj => /* return true if obj.title includes query
 or if obj.address includes query */).map((obj, i) => {
        ...
    })
```

Let's break this function down into a longer form:

```JS
data.filter((obj) => {
    // true if query is in title
    const inTitle = obj.title.includes(query)
    // true if query is in address
    const inAddress = obj.address.includes(query)
    // return true if either is true
    return inTitle || inAddress
}).map((obj, i) => {
        ...
    })
```

The search should be working pretty good by now. You will notice that the search is case sensitive. For example:

```JS
'empire' !== 'Empire'
```

Try this search for names beginning with upper and lowercase letters. Til you're satisfied you're seeing the issue.

You may want to keep search case senstive. This might be important for some types of search. You may also want to avoid case sesnsitive search.

One way to avoid case sensitive search would be to convert the search query and the search text to lowercase. Here is a solution:

```JS
const places = data.filter((obj) => {
    // true if query is in title
    const inTitle = obj.title.toLowerCase().includes(query.toLowerCase())
    // true if query is in address
    const inAddress = obj.address.toLowerCase().includes(query.toLowerCase())
    // return true if either is true
    return inTitle || inAddress
}).map((obj, i) => {
    ...
})
```

### ids out sync

At this point the List page will show the search results. Filter is returning a new array with the places that match your search results **the index of these new items won't match the position of those items in the data array.**

For example:

```JS
const data = ['100 Pine', '50 Market', '101 California']
// Note the indexes
// 0:'100 Pine', 1:'50 Market', 2:'101 California'

const filteredData = data.filter((item) => item.includes('1'))
// Note the indexes - Notice 50 market is removed and 101 California is now index 1!
// 0:'100 Pine', 1:'101 California'
```

The code in our detail page relies on the index to find the information in the source data.

In the example code above if we use the index from the filtered data the details page would show 50 market instead of 101 California.

To fix this let's add an id property to our data.

> [action]
>
> Add a new file: `places-data.js` put this in the same folder as `places-data.json`.
>
> Add the following code to `places-data.js`:
>
```JS
import data from './places-data.json'

data.forEach((obj, i) => {
    obj.id = i
})

export default data
```


What happened here? This file imports `./places-data.json`, loops over the data and adds a new **id** property to each object. Where the objects originally looked like this:

```JS
{
    title: 'Transamerica Redwood Park',
    desc: 'Located in the shadow of the Transamerica Pyramid ...',
    address: '600 Montgomery St San ...',
    ...
}
```

The objects now all look like this:

```js
{
    id: 0, // NEW id added here matches the index of this element!
    title: 'Transamerica Redwood Park',
    desc: 'Located in the shadow of the Transamerica Pyramid ...',
    address: '600 Montgomery St San ...',
    ...
}
```

The last line exports the data. Our other files can now import the data from this file. You'll need to edit any of the files that import `./places-data.json` and change the import to `./places-data.js` (notice the difference `.json` to `.js`.)

> [action]
>
> Edit the following follows changes in `PLACESDetails.js`, and `PLACESList.js`:
>
```js
// Change
import data from '../../places-data.json'
// to
import data from '../../places-data.js'
```

<!--  -->

> [action]
>
> Use the **id** from data rather than the index from map as the id for each space.
>
> Edit `./components/PLACESList.js`
>
```JS
const places = data.filter(({ features, title, address }) => {
    ...
    }).map((obj) => { // remove i here
            // Add id here!
            const { id, title, address, images, hours, features } = obj
            return (
                <PLACESSpace
                    id={id} // use id here
                    key={`${title}-${id}`} // use id here
                    name={title}
                    address={address}
                    image={images[0]}
                    hours={hours}
                />
        )
    })
    ...
```
>

Here you replaced the index of the element in the array with the id of that element. Where the index might change when the array is filtered each object will always use the same value for the id.

### Style the search field

Here are a few ideas to style the search field. What you do here depends on where you positioned the search field. In our example code it ended up in the first grid cell in the list and we decided to go with it.

Here are the styles I added.

> [action]
>
> Edit `PLACESList.css` add the following:
>
```css
.PLACESList form {
  display: flex;
  align-items: flex-start;
}

.PLACESList form input, .PLACESList form button {
  padding: 0.5em;
  margin: 0;
  border: 1px solid;
  font-size: 1;
}

.PLACESList form input {
  flex: 1;
  border-top-left-radius: 0.5em;
  border-bottom-left-radius: 0.5em;
}

.PLACESList form button {
  border-top-right-radius: 0.5em;
  border-bottom-right-radius: 0.5em;
}
```

# Now Commit

> [action]

```bash
$ git add .
$ git commit -m 'searching places'
$ git push
```
