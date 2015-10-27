---
layout: post
title: Let's Talk About Refs
---

Or, perhaps more accurately, let's talk about things to use instead of refs when using forms.

In my spare time, I like to browse reactjs [questions](http://stackoverflow.com/search?q=reactjs) on StackOverflow, and the [React forums](http://discuss.reactjs.org).

I've observed that a very common issue seems to be dealing with inputs, specifically within forms. The initial implementations and subsequent answers regular involve the use of refs.

Why?

For one, the official [React](http://facebook.github.io/react/docs/tutorial.html) tutorial uses refs to demonstrate a simple form. At this level it's fine, but once forms grow beyond maybe two inputs, this can become unwieldy very quickly. Throw things like validation into the mix and things begin to hurt.

An issue that I have with using refs is that **refs are used to reference the DOM node**, meaning that we're breaking outside of our React application to access the DOM itself.

This is probably a Bad Thing when our application is merely a description of what the DOM should look like.

Therefore, manipulating and interacting with something that isn't really there (per se) is something that I like to avoid. Fortunately, React gives us access to native browser events ([sort of](https://facebook.github.io/react/docs/events.html#syntheticevent)). So let's use them!

A solution that I like to use is to have containers for each of my inputs within a form. These containers deal with data manipulation, and render only one child component: a dumb component, used only to pass information back up to its parent container, and to render what it is told to render (more on dumb components [here](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.k0oenue6f)). This allows validations to be encapsulated within their own container components.

For the purpose of this example, however, I will write a simple, easy to implement, easy to reason about form, as a means to demonstrate an alternative to using refs.

Our container:

```` js
const MyFormContainer = React.createClass({
  getInitialState: function() {
    return {
      username: '',
      email: '',
      password: '',
      passwordConfirmation: ''
    };
  },

  render: function() {
    return (
      <MyForm
        onChange={this.handleChange}
        {...this.state}
      />
    );
  },

  handleChange: function(attribute, value) {
    this.setState({
      [attribute]: value
    });
  }
});
````

Now, our actual form:

```` js
const MyForm = React.createClass({
  render: function() {
    const {handleChange} = this;

    return (
      <form>
        <input
          type='text'
          name='username'
          onChange={handleChange}
        />
        <input
          type='email'
          name='email'
          onChange={handleChange}
        />
        <input
          type='password'
          name='password'
          onChange={handleChange}
        />
        <input
          type='password'
          name='passwordConfirmation'
          onChange={handleChange}
        />
        <button type='submit'>Submit</button>
      </form>
    );
  },

  handleChange: function(event) {
    const {name, value} = event.target;
    const {onChange} = this.props;

    onChange(name, value);
  }
});
````


We can D.R.Y this up though, right? Let's utilize composition.

```` js
const inputTypes = [
  {type: 'text', name: 'username'},
  {type: 'email', name: 'email'},
  {type: 'password', name: 'password'},
  {type: 'password', name: 'passwordConfirmation'},
];

const MyForm = React.createClass({
  render: function() {
    const inputItems = inputTypes.map(input => {
      return this.renderInput(input);
    });

    return (
      <form>
        {inputItems}
        <button type='submit'>Submit</button>
      </form>
    );
  },

  renderInput: function(options) {
    const {type, name} = options;

    return (
      <input
        type={type}
        name={name}
        onChange={this.handleChange}
      />
    );
  }

  handleChange: function(event) {
    const {name, value} = event.target;
    const {onChange} = this.props;

    onChange(name, value);
  }
});
````

Forms without refs, using the events that React provides us with; simple!
