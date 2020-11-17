---
layout: post
title:      "Controlled Forms in React"
date:       2020-11-17 02:42:24 +0000
permalink:  controlled_forms_in_react
---


Controlled forms are an important part of writing code when using React. So I read. Many times in the React docs. It wasn't until my instructor gave a rather silly example that all the pieces of a controlled form clicked for me.

Let's say you have a Login component where you can enter a username to gain access to another page on a website.
import React, { Component } from 'react'

```
export default class Login extends Component {

  render() {
    return (
      <>
        <h1>Enter Your Username</h1>
        <form>
          <label>
            Username: 
            <input type='text' name='username' />
          </label>
          <input type='submit' value='submit' />
        </form>
      </>
    )
  }
}
```
The question posed is this, how can we use the input we see on the DOM and make user inputs work in a "React-y" way, more formally, give React it's beloved single source of truth. To do this we use the React provided jsx attribute "onChange" which keeps track of key strokes and knows to run whatever function it gets passed when it detects a change. So we add that to the input in the jsx.
   <input type='text' name='username' onChange={this.onChange} />

We then need the Login class to keep track of its internal state so we will add that to the top of its class with a default value of an empty string.
```
state = {
 username: '',
}
```
As well as the onChange function that will update state with the user's input.
 ```
 onChange = (e) => {
  this.setState({username: e.target.value})
}
```
Now if we
```
console.log(this.state.username)
```
inside the return we can see state changing every time a new key is entered into the input.

(Side note: need to add multiple fields to a form? Use this trick with destructuring to reuse your 
```
onChange method for multiple inputs) ->
 onChange = (e) => {
    const {name, value} = e.target
    this.setState({[name]: value})
  }
	```
But this is still not a controlled form. It is uncontrolled, because what we rendered to the DOM is not necessarily what is in state. Confused? I was. So let's alter our onChange function a little bit:
```
 onChange = (e) => {
     let {name, value} = e.target
     value = value.split('').filter(char => char !=='e').join('')
     this.setState({[name]: value})
   }
	 ```
What this does is filter out a lowercase 'e' every time it is typed. If you implement this and are still console logging your state, they are no longer the same! In the input you can see "Merry Gentlemen" but in the console.log state it is registering as "Mrry Gntlmn". Ok...so this is rather contrived, but it made me see how a user's input and the updating state are different. And that's because the source of the user's input is the DOM itself, and not React.

To solve this, we add a value attribute to the input and set it equal to state.
```
<input type='text' name='username' onChange={this.onChange} value={this.state.username} />
```
Now if you type input into the username field, you will see as a user "Mrry Gntlmn" which matches what is being console.logged. Setting the value is a very important piece to turning this form from uncontrolled to controlled. Why does React and we as developers care so much about controlled forms? React is powerful enough to re-render elements as needed in the background, and you don't want your user to all of a sudden lose their input before its submitted. You can also use it for validations before a user even submits input. But as with many areas of developing, it's not always necessary, but it's good practice and a valuable skill to keep tucked into your back pocket.

Congrats! Now you can psych your friends out that their keys aren't working. And then explain that you're just using your react skills to render state as their input.

Happy coding!

The final file:
```
import React, { Component } from 'react'

export default class Login extends Component {

  state = {
    username: ''
  }

  onChange = (e) => {
    const {name, value} = e.target
    this.setState({[name]: value})
  }

  //silly onChange example
  // onChange = (e) => {
  //   let {name, value} = e.target
  //   value = value.split('').filter(char => char !=='e').join('')
  //   this.setState({[name]: value})
  // }

  render() {
    console.log(this.state.username)
    return (
      <>
        <h1>ADD YOURSELF TO THE HALL OF PET MEMES</h1>
        <form>
          <label>
            Username: 
            <input type='text' name='username' onChange={this.onChange} value={this.state.username} />
          </label>
          <input type='submit' value='submit' />
        </form>
      </>
    )
  }
}
```
