# <img src="https://cloud.githubusercontent.com/assets/7833470/10423298/ea833a68-7079-11e5-84f8-0a925ab96893.png" width="60"> React GeoQuakes

## Introduction


In this exercise, we'll be using axios functions to make calls to a third party API.  We will be using live data from the USGS (United States Geological Survey), specifically a data set showing significant earthquakes (M4.0 or greater) from the past week.

![](https://media.giphy.com/media/3o7bubiK9vDtxXCOgU/giphy.gif)

## Objectives

Upon completing this exercise, you will be able to:

- Use [axios](https://github.com/axios/axios) to fetch data from the [USGS Earthquakes API]()
- Use the Uber [React Map GL](https://uber.github.io/react-map-gl/) API to embed a map

## GeoQuakes

#### Deliverable

Our goal is to:

- List information about each quake.
- Display a Mapbox map 
- Render a pin at the epicenter of each quake to the map.

Here's a screenshot of what the final product *could* look like:

![geoquakes](https://cloud.githubusercontent.com/assets/4304660/25784846/9905f872-3339-11e7-92c5-30775b6bb8f4.png)


## Getting Started

- Fork and clone this repository
- Run `npm install`
- Start the app using `npm start`

### Identify Components
Things to consider in the planning stage:

- How many components will this application have?
- What components is this application built of?

<details>
  <summary><strong>Open to see the components</strong></summary>

  ![](./images/react-mapbox-geoquakes-wireframe.jpeg)

Here we've identified four components on the home page:

  1. The top level component, which we'll call `App`, is boxed in red.
  2. The Map component, a sub-component of `App`, is boxed in purple. We'll call it `Map`
  3. The list of Quakes, also a sub-component of `App`, is boxed in blue. We'll called it `QuakesList`
  4. An individual quake, a sub-component of `QuakeList`, is boxed in green. We'll call it `Quake`

</details>

<details>
  <summary><strong>Open to see component hierarchy</strong></summary>

  <h4>Component Hierarchy</h4>

Given these breakdowns we have a component hierarchy that looks like this:

  - `App`
    - `Map`
    - `QuakesList`
      - `Quake`

</details>


### Review the Dataset
Take a moment to familiarize yourself with the dataset by opening it in your browser: [http://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/significant_week.geojson](http://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/significant_week.geojson).

+ What is the structure of the data?
    + How many earthquakes does it list?
    + How would you grab the first earthquake?
        * How would you grab its title?
        * How would you grab its geological coordinates:
            - *latitude*?
            - *longitude*?
        * When did it happen?
            - How many hours ago is that?

### Create the App Component

Let's start out by building the React component that will retrieve and render earthquake information available from our API.

```jsx
import React, { Component } from 'react';

class App extends Component{

  render(){
    return(
      <h1>Hello World</h1>
    )
  }
}

export default App;
```

Now we are ready to use axios to give us data from the server. 

### Retrieve Data 

We want axios to make an API call whenever React is ready to mount the `App` component. Think back to our lesson on the React Component Lifecycle, what special method would be able to do this for us?

`componentDidMount()` will allow us to initiate an API call and `setState` whenever the data gets back. Let's add that to our `App ` component.

#### Step 1: Import axios

```js
import React, { Component } from 'react';
import axios from 'axios';
```

#### Step 2: Set your state to contain a property called `earthquakes`

```js
constructor(props){
  super(props)
  
  this.state = {
    earthquakes: []
  }
}
```

#### Step 3: Use 'componentDidMount()' to make the USGS Earthquakes API call

```js
componentDidMount(){
  // fetch API data in here
}
```

#### Step 4: Use axios to retrieve data from the USGS Earthquakes server and save to the variable, `earthquakes`.

```js
const USGS_URL = 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/significant_week.geojson'

class App extends Component {

...

    axios.get(USGS_URL)
      .then((response) => {
         console.log(response.data.features);
        const earthquakes = response.data.features;
        this.setState({
          earthquakes: earthquakes
        });
      })
      .catch((error) => {
        console.error("Error: ", error);
      });
      
    ...

}
```

Check the browser console, we see that we've successfully retrieved and rendered the earthquake data.

### Pass Data to render method

In the render lifecycle method, set the data to a variable for later use in the sub-components. **Note:** console.log the variable to make sure the data you expect is being passed to the render. 

```
render() {
    console.log(this.state.earthquakes)
    const quakeData = this.state.earthquakes 
}
```


### Create `Quake` and `QuakesList` Components

Let's write a functional component, `QuakesList ` that will accept props from the App component. Nested in the `QuakesList` component will be a `Quake` component that renders the title of the earthquake to the DOM. 

```
import React, { Component } from 'react';
import Quake from './Quake'

const QuakesList = (props) => {

  console.log(props.quakeData)

  const quakes = (
    <div>
      {props.quakeData.map((quake, index) =>
            <Quake key={index} quake={quake} />
      )}
    </div>
  )

  return(
    <div>{quakes}</div>
  )
}

export default QuakesList
```

The `QuakesList` component will map over the `quakeData` being pased down from the App component and return the `Quake` component (also functional) with each earthquake data and a key passed in.

```
const Quake = (props) => {
  console.log(props.quake.properties.title)
  
  return(
    <p>{props.quake.properties.title}</p>
  )
}

export default Quake
```

> **Pro-tip**: When in doubt, work in your Chrome Javascript Console! It makes it easier to manipulate JSON and test your ideas before committing to a solution to your problem.


### Import the `QuakesList` Component to the App Component

Import the QuakesList component into the App Component and in the return, pass the quakeData down to the `QuakesList` sub-component.

```
    import QuakesList from './QuakesList'
    
    ...
    
    return (
      <div className="App">
        ...
        <div className="quakeContainer">
          <h1 className="is-size-4">Earthquakes from the past week: </h1>
            <QuakesList quakeData={quakeData} />
        </div>
      </div>
    )
``` 

Now, let's use the Mapbox GL API wrapper to add a map to the project.

### Create `Map` Component Using Mapbox

### What is Mapbox?

Mapbox GL JS is a JavaScript library that uses WebGL to render interactive maps from vector tiles and Mapbox styles. A WebGL map brings the possibility to render vector tiles on the client side, this way we get all the data accessible on the browser and we can interact with pretty much everything on the map.


#### Step 1 - Create a MapBox account

If you don't already have a Mapbox account, sign up for one [here](https://www.mapbox.com/account/).

In order to show maps from Mapbox you need to register on their website in order to retrieve an access token required by the map component, which will be used to identify you and start serving up map tiles. The service will be free until a certain level of traffic is exceeded.

Later we will use the [dotenv](https://www.npmjs.com/package/dotenv) package module to hide your environment variables (in this case, our Mapbox access token) and then we’ll put that in our `.gitignore` so no one can abuse our tokens.

Next, we will integrate React Map GL, a wrapper for Mapbox GL that was created by [Uber](https://uber.github.io/react-map-gl/#/Documentation/getting-started/get-started), into our application. In Terminal, run the following command:

```
npm install react-map-gl
```
Uber has provided some [boilerplate code](https://uber.github.io/react-map-gl/#/Documentation/getting-started/get-started?section=example) to get us started. Let’s break down the following lines. First, we need to import React methods and the Component class from the React library. Then, import React methods from the Mapbox library; we're going to be using the Popup and Marker methods.

#### Step 2 - Create Map Component

Next, we instantiatiate the component we're creating. In this case, it’s called `Map` and it’s extending the React Component class. As you know, a React component can access dynamic information in two ways: props and state. Since a component is able to decide its own state, in order to make our Map component have state, we must give it a state property and declare it inside the constructor method (or not).

```js
import {Component} from 'react';
import ReactMapGL from 'react-map-gl';
    
class Map extends Component {
    
  state = {
    viewport: {
      width: 400,
      height: 400,
      latitude: 37.7577,
      longitude: -122.4376,
      zoom: 8
    }
  };
    
  render() {
  
     const { viewport } = this.state;
     
    return (
      <ReactMapGL
        width={viewport.width}
        height={viewport.height}
        latitude={viewport.latitude}
        longitude={viewport.longitude}
        zoom={viewport.zoom}
        
        onViewportChange={(viewport) => this.setState({viewport})}
      />
    );
  }
}
    
export default Map;
```

The boilerplate code needed to start off the Map component can be found [here](https://github.com/uber/react-map-gl).

The `render` method is what renders the component to the browser, so it controls what is displayed for this component. From this function, we return what we want to display. In this case, we are rendering the Mapbox map component `<ReactMapGL />` and including the viewport properties.

The `onViewportChange` callback is fired when the user interacts with the map. User interaction and transition will not work without a valid one.

The `export` exposes the Map class to other files. This means that other files can import from this file in order to use the Map class.

The current mapbox-gl release requires its stylesheet be included **at all times**. The marker, popup and navigation components in react-map-gl also need the stylesheet to work properly. Therefore, in the `public/index.hmtl` file, add the Mapbox stylesheet to the head:

```
<!-- index.html -->
<link href='https://api.tiles.mapbox.com/mapbox-gl-js/v0.42.0/mapbox-gl.css' rel='stylesheet' />
```

#### Step 3 - Complete the ReactMapGL Component

Use the following [Github repo](https://github.com/uber/react-map-gl/tree/master/examples/interaction/src) as a guide and set the Mapbox token variable in the `index.js`. 

```
const MAPBOX_TOKEN = '';
```
There are several ways to provide a token to your app. In this example, we're passing it as a prop to the ReactMapGL instance <ReactMapGL mapboxApiAccessToken={TOKEN} />

Another recommended way is using the [dotenv](https://github.com/motdotla/dotenv) package and put your key in an untracked .env file, that will then expose it as a process.env variable, with much less leaking risks.

  - Put the API key in a .env file:

```
REACT_APP_MAPBOX_ACCESS_TOKEN='key in here'

```
  - Then, set the MAPBOX_TOKEN variable as follows:

```
const MAPBOX_TOKEN = process.env.REACT_APP_MAPBOX_ACCESS_TOKEN;
```

#### Interactive Map Properties

This component renders MapboxGL and provides full interactivity support. It is the default exported component from ReactMapGL and we need to set up the following properties on the map:

  - mapStyle: several basic templates are already provided by Mapbox here, but you can also design your own on their [Studio Platform]()
  - viewport: you can specify latitude and longitude of the object that you want to define the center of your map, as well as how zoomed in on that object you want to be.
  - mapboxApiAccessToken: once again, it’s important to obtain your unique token to access the Mapbox API

```
render() {

...

    return (
      <ReactMapGL

        ...

        mapStyle="mapbox://styles/mapbox/light-v9"
        onViewportChange={(viewport) => this.setState({viewport})}
        mapboxApiAccessToken={MAPBOX_TOKEN}
      />
    );
}

```

#### Step 4 - Import the Map Component into the App Component

```
import Map from '../Map';

...

return (
  <div className="App">
    <div className="mapContainer">
      <Map component={Map} quakeData={quakeData} />
    </div>
    
    ...
    
  </div>
)
```

#### ASSIGNMENT -- Getting Familiar with APIs is a process

Rendering data to a map is an exercise in detail and patience. Please use the following resources:

- the Uber React Map GL [documentation](https://uber.github.io/react-map-gl/#/),
- the instructions in [this repo](https://git.generalassemb.ly/WDI-Epiphany/react-mapbox-workshop) and, 
- [StackOverflow](https://stackoverflow.com/) to complete this exercise.

to figure out the following:

- **Add Markers to the Map**: We want a marker for every earthquake on the map
- **Create an Earthquake Pin**

#### Bonus Features

###### USGS 

- **Calculate how long ago the quake occurred** and add it to the page. E.g. "28 hours ago". Currently, the time that the API returns is in Unix time (seconds since 1/1/1970). That's a nice format for computers, but not a nice format for humans.
- **Parse the title to only include the location**, E.g. Instead of "M 4.2 - 1km ESE of Fontana, California", it should just say "Fontana, California."
- **Create a visual indicator of the magnitude of a quake**. For instance, maybe a 4.0 is indicated by a "yellow" dot, a 5.0 by an "orange" dot, and anything larger is "red".

###### Mapbox

- **Create Earthquake Info**: this is the information that will be displayed on the popup window
- **Render Popup window**: this will be displayed when you click on each pin

###### Styling

- Use Bulma to style the earthquake modules
