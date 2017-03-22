# Setup and Notes
## Setup
1. `create-rect-app ReactRouterV4` sets up the project
2. Delete logo, index.css, and test files.
3. `npm i react-router-dom@next --save-dev` (@next to install latest version)
4. `npm start`

Notes created by following Egghead.io course at the url below:
`https://egghead.io/courses/add-routing-to-react-apps-using-react-router-v4`

## Notes
### Path Qualifiers
```javascript
<Router>
  <div>
    <Route exact path="/" component={Home} />
    <Route path="/about" component={Home} />
  </div>
</Router>
```
Using `exact path` will prevent the Home component from being rendered twice when we are at the `'/about'` path. The `exact` qualifier does not care about trailing forward slashes, however the `strict` qualifier does care about them.

### render vs. children Route props
```javascript
<Route path="/about" render={() => <h1>About</h1>} />
<Route path="/about" children={() => <h1>About</h1>} />
```
The `render` prop works as expected; the render callback is executed upon visiting the `'/about'` path. The `children` prop on the other hand fires on every path.

```javascript
<Route path="/about"
  children={({match}) => match && <h1>About</h1>} />
```
The `match` property is available in the Route props `render`, `children`, and `component`, and can be extracted to check if the current URL matches the Route path.

### Link
```javascript
const Links = () => (
  <nav>
    <Link to="/">Home</Link>
    <Link to={{pathname: '/about'}}>About</Link>
    <Link replace to="/contact">Contact</Link>
  </nav>
)

const App = () => (
  <Router>
    <div>
      <Links />
      <Route exact path="/" render={() => <h1>Home</h1>} />
      <Route path="/about" render={() => <h1>About</h1>} />
      <Route path="/contact" render={() => <h1>Contact</h1>} />
    </div>
  </Router>
);
```
The `replace to` acts like `Router.replace(new_url)` as opposed to `Router.push(new_url)`. The `to` prop can accept a string or an object.

### NavLink
Similar to Link but allows us to identify our current route and decide if we want this link to have an active state.
```javascript
const isActiveFunc = (match, location) => {
  console.log(match, location);
  return match
}

const Links = () => (
  <nav>
    <NavLink exact activeClassName="active" to="/">Home</NavLink>
    <NavLink activeStyle={{color: 'green'}} to="/about">About</NavLink>
    <NavLink isActive={isActiveFunc}
      activeClassName="active"
      to="/contact">Contact</NavLink>
  </nav>
)
```
`NavLink` behaves similar to `Route` in that the `activeClassName` property for the `Home` NavLink will be true on all pages unless the `exact` qualifier is specified.
The `activeStyle` property can be used to specify CSS properties instead of providing an `activeClassName` property.
The `isActive` property takes a function, and will fire anytime the URL changes.

### URL Parameters
```javascript
const App = (props) => (
  <Router>
    <div>
      <Route path="/:page?/:subpage?" render={({match}) => (
        <h1>
          PAGE: {match.params.page || 'Home' }
          <br/>
          SUBPAGE: {match.params.subpage}
        </h1>
      )} />
    </div>
  </Router>
)
```
Putting a `:` before words in the `path` change them from ordinary strings into URL parameters. Adding a `?` after the URL parameters makes it optional.

In the example above, going to the route `/react/router` will have the HTML text:
```html
PAGE: react
SUBPAGE: router
```
The `path` property can also be changed to a dash format `path="/:page?-:subpage?"`. This will yield the same HTML text when visiting the route `/react-router`.

### Regular Expressions with Routes
```javascript
const App = (props) => (
  <Router>
    <div>
      <Route path="/:a(\d{2}-\d{2}-\d{4}):b(\.[a-z]+)" render={({match}) => (
        <h1>
          paramA: {match.params.a}
          <br/>
          paramB: {match.params.b}
        </h1>
      )} />
    </div>
  </Router>
)
```
Regular expressions can be used to validate url parameters. The example above when going to the route `/03-21-2017.html` prints the HTML text:
```html
paramA: 03-21-2017
paramB: .html
```

### Parsing Query Parameters
```javascript
const Links = () => (
  <nav>
    <Link to="/?id=123">Inline</Link>
    <Link to={{pathname: '/', search: 'id=456'}}>Object</Link>
  </nav>
)

const App = (props) => (
  <Router>
    <div>
      <Links />
      <Route path="/" render={({match, location}) => (
        <div>
          <p>root</p>
          <p>{JSON.stringify(match)}</p>
          <p>{JSON.stringify(location)}</p>
          <p>{new URLSearchParams(location.search).get('id')}</p>
        </div>
      )} />
    </div>
  </Router>
)
```
Query strings can be manually constructed via question mark syntax when passing a string to the `to` prop of `Link`, or by specifying the key and value without the question mark and nesting it under the `search` key in an object. Attempting to add a query string to the `pathname` property will not achieve the same results; the query string will be under the `search` key of the `location` object.

The example above prints the following HTML text when clicking on the `Object` link:
```html
root
{"path":"/","url":"/","isExact":true,"params":{}}
{"pathname":"/","search":"?id=456","hash":"","key":"v1oz70"}
456
```

### Catching Unknown Routes with Switch Component
```javascript
const Links = () => (
  <nav>
    <Link to="/">Home</Link>
    <Link to="/about">About</Link>
    <Link to="/contact">Contact</Link>
  </nav>
)

class App extends React.Component {
  render() {
    return (
      <Router>
        <div>
          <Links />
          <Switch>
            <Route exact path="/" render={() => <h1>Home</h1>} />
            <Route path="/about" render={() => <h1>About</h1>} />
            <Route render={() => <h1>Page not found</h1>} />
          </Switch>
        </div>
      </Router>
    );
  }
}
```
Creating a `Route` component with no path will cause it to match all URLs. The `Switch` component will only render the first of its child components that matches the current path.

### Conditionally Rendering a Route with Switch Component
```javascript
class App extends React.Component {
  render() {
    return (
      <Router>
        <div>
          <Switch>
            <Links />
            <Route exact path="/" render={() => <h1>Home</h1>} />
            <Route path="/about" render={() => <h1>About</h1>} />
            <Route path="/contact" render={() => <h1>Contact</h1>} />
            <Route path="/:itemid"
              render={({match}) => <h1>Item: {match.params.itemid}</h1>} />
          </Switch>
        </div>
      </Router>
    );
  }
}
```
Similar to the previous section introducing `Switch`, the `/:itemid` route will only fire if none of the other routes match.

### Render Multiple Components for the Same Route
```javascript
const Links = () => (
  <nav>
    <Link to="/home">Home</Link>
    <Link to="/about">About</Link>
    <Link to="/contact">Contact</Link>
  </nav>
)

const Header = ({match}) => (
  <div className="header">
    <Route path="/:page"
      render={({match}) => (
        <h1>{match.params.page} header</h1>)} />
  </div>
)

const Content = ({match}) => (
  <div className="content">
    <Route path="/:page"
      render={({match}) => (
        <p>{match.params.page} content</p>)} />
  </div>
)

class App extends React.Component {
  render() {
    return (
      <Router>
        <div>
          <Links />
          <Header />
          <Content />
        </div>
      </Router>
    );
  }
}
```
Pretty self explanatory.

### Rendering Nested Components
```javascript
const Home = () => <h1>Home</h1>;
const Menu = () => (
  <div>
    <h1>Menu</h1>
    <Link to="/menu/food">Food</Link>
    <Link to="/menu/drinks">Drinks</Link>
    <Link to="/menu/sides">Sides</Link>
    <Route path="/menu/:section"
      render={({match}) => <h2>{match.params.section}</h2>} />
  </div>
);

class App extends React.Component {
  render() {
    return (
      <Router>
        <div>
          <Link to="/">Home</Link>
          <Link to="/menu">Menu</Link>
          <Route exact path="/" component={Home} />
          <Route path="/menu" component={Menu} />
        </div>
      </Router>
    );
  }
}
```
Pretty self explanatory.

### Redirecting To Another Page
```javascript
const Links = () =>
  <nav>
    <Link to="/">Home</Link>
    <Link to="/old/123">Old</Link>
    <Link to="/new/456">New</Link>
    <Link to="/protected">Protected</Link>
  </nav>

const loggedin = false;

class App extends React.Component {
  render() {
    return (
      <Router>
        <div>
          <Links />
          <Switch>
            <Route exact path="/" render={() => (<h1>Home</h1>)} />
            <Route path="/new/:str"
              render={({match}) => (<h1>New: {match.params.str}</h1>)} />

            <Route path="/old/:str" render={({match}) => (
              <Redirect to={`/new/${match.params.str}`} />
            )} />

            <Route path="/protected" render={() => (
              loggedin ? <h1>Welcome to the protected page</h1>
              : <Redirect to="/new/Login" />
            )} />

            {/* <Redirect from="old" to="/new" /> */}

          </Switch>
        </div>
      </Router>
    );
  }
}
```
Pretty self explanatory.

### Intercepting Route Change
```javascript
const Home = () => (<h1>Home</h1>)
class Form extends React.Component {
  state = {dirty: false};
  setDirty = () => this.setState({dirty: true});
  render() {
    return (
      <div>
        <h1>Form</h1>
        <input type="text" onInput={this.setDirty} />
        <Prompt message="Data will be lost!"
          when={this.state.dirty} />
      </div>
    )
  }
}

const App = () => (
  <Router>
    <div>
      <Link to="/">Home</Link>
      <Link to="/form">Form</Link>
      <Route exact path="/" component={Home}/>
      <Route path="/form" component={Form}/>
    </div>
  </Router>
)
```
The `Prompt` component has a `message` prop and a `when` prop. The `message` prop is simply the text that the user will see in the pop-up alert.  The `when` prop takes a boolean, and triggers the alert when its value is `true`.

### Different React Router Types
#### BrowserHistory
```javascript
const LinksRoutes = () => (
  <div>
    <Link to="/">Home</Link>
    <Link to="/about">About</Link>
    <Route exact path="/" render={() => <h1>Home</h1>} />
    <Route path="/about" render={() => <h1>About</h1>} />
  </div>
)

const forceRefresh = () => {
  console.log(new Date());
  return true;
}

const BrowserRouterApp = () => (
  <BrowserRouter forceRefresh={forceRefresh()}>
    <LinksRoutes />
  </BrowserRouter>
)
```
`BrowserHistory` is meant to be used in environments where the `HTML5 history` API is supported. The `forceRefresh` prop will trigger a refresh when its value is `true`.

#### HashRouter
```javascript
const HashRouterApp = () => (
  <HashRouter hashType="hashbang">
    <LinksRoutes />
  </HashRouter>
)
```
`HashRouter` is useful in environments where the `HTML5 history` API is not supported. It has a property `hashType` that by default is set to `"slash"`, which makes the root URL begin with `"/#"`. Other types include `"noslash"` and `"hashbang"`, which make the root URL begin with `"#"` and `"#!"`, respectively.

#### MemoryRouter
```javascript
const MemoryRouterApp = () => (
  <MemoryRouter initialEntries={['/', '/about']}
    initialIndex={1}>
    <LinksRoutes />
  </MemoryRouter>
)
```
`MemoryRouter` is mainly useful for testing; it keeps all the history in RAM and does not change the URL upon navigating to different URLs. It has two properties `initialEntries` and `initialIndex` that take an array and array index, respectively, to represent history and a starting path.

#### StaticRouter
`StaticRouter` is meant for server-side rendering.

#### NativeRouter
`NativeRouter` is meant for mobile applications built with React-Native.
