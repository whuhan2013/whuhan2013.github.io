---
layout: post
title: React Basic Demos
date: 2016-7-5
categories: blog
tags: [React Native]
description: React Basic Demo
---


**This papper mainly includes the following contents**

1. Render JSX 
2. Use JavaScript in JSX    
3. Use array in JSX
4. Define a component 
5. this.props.children  
6. PropTypes  
7. Finding a DOM node    
8. this.state  
9. Form           
10. Component Lifecycle      
11. Ajax            
12. Display value from a Promise

**Render JSX**           
The template syntax in React is called JSX. It is allowed in JSX to put HTML tags directly into JavaScript codes. ReactDOM.render() is the method which translates JSX into HTML, and renders it into the specified DOM node.

```
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('example')
);
```

Attention, you have to use script type="text/babel" to indicate JSX codes, and include browser.min.js, which is a browser version of Babel and could be get inside a babel-core@5 npm release, to actually perform the transformation in the browser.

Before v0.14, React use JSTransform.js to translate <script type="text/jsx">. It has been deprecated.


**Use JavaScript in JSX**       
You could also use JavaScript in JSX. It takes angle brackets (<) as the beginning of HTML syntax, and curly brackets ({) as the beginning of JavaScript syntax.

```
var names = ['Alice', 'Emily', 'Kate'];

ReactDOM.render(
  <div>
  {
    names.map(function (name) {
      return <div>Hello, {name}!</div>
    })
  }
  </div>,
  document.getElementById('example')
);
```

**Use array in JSX**          
If a JavaScript variable is an array, JSX will implicitly concat all members of the array.

```
var arr = [
  <h1>Hello world!</h1>,
  <h2>React is awesome</h2>,
];
ReactDOM.render(
  <div>{arr}</div>,
  document.getElementById('example')
);
```

**Define a component**           
React.createClass() creates a component class, which implements a render method to return an component instance of the class. You don't need to call new on the class in order to get an instance, just use it as a normal HTML tag.

```
var HelloMessage = React.createClass({
  render: function() {
    return <h1>Hello {this.props.name}</h1>;
  }
});

ReactDOM.render(
  <HelloMessage name="John" />,
  document.getElementById('example')
);
```

Please remember the first letter of the component's name must be capitalized, otherwise React will throw an error. For instance, HelloMessage as a component's name is OK, but helloMessage is not allowed. And a React component should only have one top child node.

```
// wrong
var HelloMessage = React.createClass({
  render: function() {
    return <h1>
      Hello {this.props.name}
    </h1><p>
      some text
    </p>;
  }
});

// correct
var HelloMessage = React.createClass({
  render: function() {
    return <div>
      <h1>Hello {this.props.name}</h1>
      <p>some text</p>
    </div>;
  }
});
```

** this.props.children**      
React uses this.props.children to access a component's children nodes.

```
var NotesList = React.createClass({
  render: function() {
    return (
      <ol>
      {
        React.Children.map(this.props.children, function (child) {
          return <li>{child}</li>;
        })
      }
      </ol>
    );
  }
});

ReactDOM.render(
  <NotesList>
    <span>hello</span>
    <span>world</span>
  </NotesList>,
  document.getElementById('example')
);
```

Please be mindful that the value of this.props.children has three possibilities. If the component has no children node, the value is undefined; If single children node, an object; If multiple children nodes, an array. You should be careful to handle it.

React gave us an utility React.Children for dealing with the this.props.children's opaque data structure. You could use React.Children.map to iterate this.props.children without worring its data type being undefined or object. Check official document for more methods React.Children offers.


**PropTypes**              
Components have many specific attributes which are called ”props” in React and can be of any type.

Sometimes you need a way to validate these props. You don't want users have the freedom to input anything into your components.

React has a solution for this and it's called PropTypes.

```
var MyTitle = React.createClass({
  propTypes: {
    title: React.PropTypes.string.isRequired,
  },

  render: function() {
     return <h1> {this.props.title} </h1>;
   }
});
```

The above component of MyTitle has a props of title. PropTypes tells React that the title is required and its value should be a string.

Now we give Title a number value.

```
var data = 123;

ReactDOM.render(
  <MyTitle title={data} />,
  document.getElementById('example')
);
```            

It means the props doesn't pass the validation, and the console will show you an error message.

Warning: Failed propType: Invalid prop `title` of type `number` supplied to `MyTitle`, expected `string`.

P.S. If you want to give the props a default value, use getDefaultProps().

```
var MyTitle = React.createClass({
  getDefaultProps : function () {
    return {
      title : 'Hello World'
    };
  },

  render: function() {
     return <h1> {this.props.title} </h1>;
   }
});

ReactDOM.render(
  <MyTitle />,
  document.getElementById('example')
);
```

**Finding a DOM node**       
Sometimes you need to reference a DOM node in a component. React gives you the ref attribute to find it.  

```
var MyComponent = React.createClass({
  handleClick: function() {
    this.refs.myTextInput.focus();
  },
  render: function() {
    return (
      <div>
        <input type="text" ref="myTextInput" />
        <input type="button" value="Focus the text input" onClick={this.handleClick} />
      </div>
    );
  }
});

ReactDOM.render(
  <MyComponent />,
  document.getElementById('example')
);
```

The desired DOM node should have a ref attribute, and this.refs.[refName] would return the corresponding DOM node. Please be mindful that you could do that only after this component has been mounted into the DOM, otherwise you get null.

**this.state**             
React thinks of component as state machines, and uses this.state to hold component's state, getInitialState() to initialize this.state(invoked before a component is mounted), this.setState() to update this.state and re-render the component.

```
var LikeButton = React.createClass({
  getInitialState: function() {
    return {liked: false};
  },
  handleClick: function(event) {
    this.setState({liked: !this.state.liked});
  },
  render: function() {
    var text = this.state.liked ? 'like' : 'haven\'t liked';
    return (
      <p onClick={this.handleClick}>
        You {text} this. Click to toggle.
      </p>
    );
  }
});

ReactDOM.render(
  <LikeButton />,
  document.getElementById('example')
);
```

**Form**            
According to React's design philosophy, this.state describes the state of component and is mutated via user interactions, and this.props describes the properties of component and is stable and immutable.

Since that, the value attribute of Form components, such as <input>, <textarea>, and <option>, is unaffected by any user input. If you wanted to access or update the value in response to user input, you could use the onChange event.

```
var Input = React.createClass({
  getInitialState: function() {
    return {value: 'Hello!'};
  },
  handleChange: function(event) {
    this.setState({value: event.target.value});
  },
  render: function () {
    var value = this.state.value;
    return (
      <div>
        <input type="text" value={value} onChange={this.handleChange} />
        <p>{value}</p>
      </div>
    );
  }
});

ReactDOM.render(<Input/>, document.getElementById('example'));
```

**Component Lifecycle**              
Components have three main parts of their lifecycle: Mounting(being inserted into the DOM), Updating(being re-rendered) and Unmounting(being removed from the DOM). React provides hooks into these lifecycle part. will methods are called right before something happens, and did methods which are called right after something happens.

```
var Hello = React.createClass({
  getInitialState: function () {
    return {
      opacity: 1.0
    };
  },

  componentDidMount: function () {
    this.timer = setInterval(function () {
      var opacity = this.state.opacity;
      opacity -= .05;
      if (opacity < 0.1) {
        opacity = 1.0;
      }
      this.setState({
        opacity: opacity
      });
    }.bind(this), 100);
  },

  render: function () {
    return (
      <div style={{opacity: this.state.opacity}}>
        Hello {this.props.name}
      </div>
    );
  }
});

ReactDOM.render(
  <Hello name="world"/>,
  document.getElementById('example')
);
```

**The following is a whole list of lifecycle methods.**

- componentWillMount(): Fired once, before initial rendering occurs. Good place to wire-up message listeners. this.setState doesn't work here.                     
- componentDidMount(): Fired once, after initial rendering occurs. Can use this.getDOMNode().
- componentWillUpdate(object nextProps, object nextState): Fired after the component's updates are made to the DOM. Can use this.getDOMNode() for updates.                   
- componentDidUpdate(object prevProps, object prevState): Invoked immediately after the component's updates are flushed to the DOM. This method is not called for the initial render. Use this as an opportunity to operate on the DOM when the component has been updated.                            
- componentWillUnmount(): Fired immediately before a component is unmounted from the DOM. Good place to remove message listeners or general clean up.             
- componentWillReceiveProps(object nextProps): Fired when a component is receiving new props. You might want to this.setState depending on the props.
- shouldComponentUpdate(object nextProps, object nextState): Fired before rendering when new props or state are received. return false if you know an update isn't needed.


**Ajax**          
How to get the data of a component from a server or an API provider? The answer is using Ajax to fetch data in the event handler of componentDidMount. When the server response arrives, store the data with this.setState() to trigger a re-render of your UI.

```
var UserGist = React.createClass({
  getInitialState: function() {
    return {
      username: '',
      lastGistUrl: ''
    };
  },

  componentDidMount: function() {
    $.get(this.props.source, function(result) {
      var lastGist = result[0];
      if (this.isMounted()) {
        this.setState({
          username: lastGist.owner.login,
          lastGistUrl: lastGist.html_url
        });
      }
    }.bind(this));
  },

  render: function() {
    return (
      <div>
        {this.state.username}'s last gist is
        <a href={this.state.lastGistUrl}>here</a>.
      </div>
    );
  }
});

ReactDOM.render(
  <UserGist source="https://api.github.com/users/octocat/gists" />,
  document.getElementById('example')
);
```

**Display value from a Promise**              
This demo is inspired by Nat Pryce's article "Higher Order React Components".

If a React component's data is received asynchronously, we can use a Promise object as the component's property also, just as the following.

```
ReactDOM.render(
  <RepoList
    promise={$.getJSON('https://api.github.com/search/repositories?q=javascript&sort=stars')}
  />,
  document.getElementById('example')
);
```

The above code takes data from Github's API, and the RepoList component gets a Promise object as its property.

Now, while the promise is pending, the component displays a loading indicator. When the promise is resolved successfully, the component displays a list of repository information. If the promise is rejected, the component displays an error message.

```
var RepoList = React.createClass({
  getInitialState: function() {
    return { loading: true, error: null, data: null};
  },

  componentDidMount() {
    this.props.promise.then(
      value => this.setState({loading: false, data: value}),
      error => this.setState({loading: false, error: error}));
  },

  render: function() {
    if (this.state.loading) {
      return <span>Loading...</span>;
    }
    else if (this.state.error !== null) {
      return <span>Error: {this.state.error.message}</span>;
    }
    else {
      var repos = this.state.data.items;
      var repoList = repos.map(function (repo) {
        return (
          <li>
            <a href={repo.html_url}>{repo.name}</a> ({repo.stargazers_count} stars) <br/> {repo.description}
          </li>
        );
      });
      return (
        <main>
          <h1>Most Popular JavaScript Projects in Github</h1>
          <ol>{repoList}</ol>
        </main>
      );
    }
  }
});
```

