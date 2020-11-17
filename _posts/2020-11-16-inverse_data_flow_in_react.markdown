---
layout: post
title:      "Inverse Data Flow in React"
date:       2020-11-17 02:44:56 +0000
permalink:  inverse_data_flow_in_react
---


Once we have a controlled form set up in React, we have to handle getting that data from the form and to the appropriate location to update state, so that the new information can be rendered to the DOM. Inverse data flow is a key part of making that happen.

The controlled form is where we create the input, and whenever it is changed we can keep track of it in state with an onChange function. It then takes in state as the input's value so it is dependent on state and not relying on the DOM as it's source of truth. Every time state is set this triggers a re-render. React then goes through the code checking if it needs to perform a re-render, and when it gets to
 value={this.state.username}
it realises, yes! State has been changed! I will re-render you! This is the essence of the controlled form:

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

  render() {
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
Great, so now we are keeping track of a username in the input field, now the user is ready to submit. To handle this we add an onSubmit react synthetic event handler. The first thing we need to do if we don’t want the app to refresh when a form is submitted is to use e.preventDefault. We should be familiar with this from Javascript.
```
 onSubmit = (e) => {
    e.preventDefault()
    //what goes here?
  }

render() {
    return (
      <>
        <h1>ADD YOURSELF TO THE HALL OF PET MEMES</h1>
        <form onSubmit={this.onSubmit}>
          <label>
            Username: 
            <input type='text' name='username' onChange={this.onChange} value={this.state.username} />
          </label>
          <input type='submit' value='submit' />
        </form>
      </>
    )
  }
	```
Now the goal is not to update the state here in Login, but to update the state in the parent. It is the parent’s job to keep track of who the user is for the entire app. Not the form. The form’s job is to keep track of state for the input of the username field (because it is a controlled form) which is different from who the user actually is. Our parent in this case is the MainContainer:
```
import React, {Component} from 'react'
import Login from '../components/Login'

export default class MainContainer extends Component {

  state = {
    username: ''
  }

  render() {
    return (<Login />)
  }
}
```
We can't intrusively reach into the parent and directly modify its state. Instead what we do is that we write a function inside the parent that dictates how to update state. Our function setUsername takes in a username, and calls this.setState for the username key to the argument that is passed in as username.
```
 state = {
    username: ''
  }

  setUsername = (username) => {
    this.setState({username: username})
  }
	```
By writing a function that takes in a username, and rewrites this component’s state, we can then give it to another component in the form of a prop. When we do this we are giving that other component the ability to update state. Specifically it can change state in the way that is described in the function. By giving it to the Login component as a prop we are telling Login, “here Login, this is how you can update my state, take it from your props and use it when you need it”.
```
render() {
    return (<Login setUsername={this.setUsername}/>)
  }
	```
Now the Login component can call this.props.setUsername(this.state.username) and pass in the username that we want to set in the parent’s state.
```
 onSubmit = (e) => {
    e.preventDefault()
    this.props.setUsername(this.state.username)
  }
	```
We are lending a function to allow our child to update our state for us. It's like giving a child a specific credit card with instructions on the exact thing they can purchase, and imagine this is a world where the child wouldn't try to sneak a few candy purchases. It just doesn't happen. We are not passing down the whole this.setState method (because it'd be terrible practice to give your child access to all your assets and tell them have fun), just a particular state they can change.

The act of the child component invoking the function provided by the parent that allows the child to update the parent's state is inverse data flow. Because we are sending the information upwards rather than downwards.

Now if we put that all together, you can throw in a console.log into the MainContainer's render function and see that in the MainContainer the state is being changed by the child.

MainContainer.js :
```
import React, {Component} from 'react'
import Login from '../components/Login'

export default class MainContainer extends Component {

  state = {
    username: ''
  }

  setUsername = (username) => {
    this.setState({username: username})
  }

  render() {
    console.log("in main container:", this.state.username)
    return (<Login setUsername={this.setUsername}/>)
  }
}
Login.js :
import React, { Component } from 'react'

export default class Login extends Component {

  state = {
    username: ''
  }

  onChange = (e) => {
    const {name, value} = e.target
    this.setState({[name]: value})
  }

  onSubmit = (e) => {
    e.preventDefault()
    this.props.setUsername(this.state.username)
  }

  render() {
    return (
      <>
        <h1>ADD YOURSELF TO THE HALL OF PET MEMES</h1>
        <form onSubmit={this.onSubmit}>
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

And that is the crux of inverse data flow, a parent passes a function down as prop to a child and that function has access to update the parent's state. The child can then pass in the information they have into that function and there by change the state of the parent.

Happy coding!
