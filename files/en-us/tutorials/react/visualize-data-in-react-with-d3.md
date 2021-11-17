---
title: How to visualize data in React with D3.js
author_name: Imran Farooq
author_link: https://imranfarooqreactjs.medium.com/
discussion_link: https://github.com/indepth-dev/community/discussions/199
display_name: How to visualize data in React with D3.js
---

# **How to visualize data in React with D3.js**

When I want to visualize data on a web app, my preferred environment is using D3.js in a React application. But these two technologies are quite hard to combine. The reason is that they both want to handle the DOM.

Let’s start it!

### Creating SVG elements
When we want to visualize data in the browser, we will most probably want to work with SVG elements because they are very expressive and are absolutely positioned.

To begin, we shall render a simple <svg> element.

```javascript
const Svg = () => {
   return (
           <svg style={{
              border: "2px solid #1cff00"
           }} />
   )
}
```

![](https://images.indepth.dev/tutorials/react/vizualize_data_d3_1.png)

I have added a green border so we can see our <svg> element.
For visualizing data, we shall have to represent data points as shapes. Let’s start with a simple basic shape: a <circle>.

```javascript
const Circle = () => {
   const ref = useRef()

   useEffect(() => {
      const svgElement = d3.select(ref.current)
      svgElement.append("circle")
              .attr("cx", 150)
              .attr("cy", 70)
              .attr("r",  50)
   }, [])

   return (
           <svg
                   ref={ref}
           />
   )
}
```

![](https://images.indepth.dev/tutorials/react/vizualize_data_d3_2.png)

### Explanation of above-mentioned Circle.jsx

1. Uses a ref to store a reference to our rendered <svg> element
2. Runs D3 code when the component mounts.
3. Uses `d3.select()` to turn our ref into a D3 selection object.
4. Uses our D3 selection object to append a <circle> element.

But this is a large amount of code to draw a single shape. And don’t I have to use React refs as sparingly as possible?
   
Thanks to the React core team, all SVG elements are supported in JSX with the release of React 15 so we can create a circle with a <circle> element.

```javascript
const Circle = () => {
   return (
           <svg>
              <circle
                      cx="150"
                      cy="77"
                      r="40"
              />
           </svg>
   )
}
```

![](https://images.indepth.dev/tutorials/react/vizualize_data_d3_2.png)

### What are the benefits of using standard JSX instead of running D3 code on mount?
1. **Declarative instead of imperative**
   The code describes what is being drawn, instead of how to draw it.
2. **Less code**
   Our second Circle component has fewer than two-thirds the number of lines of our first iteration.
3. **Less hacky**
   React is, mainly, a rendering library, and has a lot of optimizations to keep our web apps performant. When adding elements using D3, we’re hacking around React, and importantly have to fight against those optimizations.

### Creating many SVG elements

   One of the main concepts of D3.js is to bind data to DOM elements.
   Now, we will generate a dataset of 10 random [x, y] coordinates.

### How to generate this dataset
   I have developed a `generateDataset()` function that outputs an array of 10 arrays.

```javascript
const generateDataset = () => (
        Array(10).fill(0).map(() => ([
           Math.random() * 80 + 10,
           Math.random() * 35 + 10,
        ]))
)
```
What will it look like if I drew a <circle> at each of these locations?
Beginning with the naive D3 code:

```javascript

const Circles = () => {
   const [dataset, setDataset] = useState(
    generateDataset()
)
const ref = useRef()

useEffect(() => {
   const svgElement = d3.select(ref.current)
   svgElement.selectAll("circle")
           .data(dataset)
           .join("circle")
           .attr("cx", d => d[0])
           .attr("cy", d => d[1])
           .attr("r",  3)
}, [dataset])

useInterval(() => {
   const newDataset = generateDataset()
   setDataset(newDataset)
}, 2000)

return (
        <svg
                viewBox="0 0 100 50"
                ref={ref}
        />
)
}
```

![](https://images.indepth.dev/tutorials/react/vizualize_data_d3_3.png)

### Explanation of above-mentioned Circles.jsx
1. I am producing a selection of all <circle> elements and using our D3 selection’s `join()` method to insert a circle for each data point.
2. I am re-running my d3 code whenever the dataset changes.
3. I am utilizing `useInterval()` (a React hook) to re-calculate my dataset every 2 seconds.
But we still have the original issues: our code is quite imperative, verbose, and hacky. What will this look like using React to render our <circle>?


```javascript
const Circles = () => {
   const [dataset, setDataset] = useState(
           generateDataset()
   )

   useInterval(() => {
      const newDataset = generateDataset()
      setDataset(newDataset)
   }, 2000)

   return (
           <svg viewBox="0 0 100 50">
              {dataset.map(([x, y], i) => (
                      <circle
                              cx={x}
                              cy={y}
                              r="3"
                      />
              ))}
           </svg>
   )
}
```

![](https://images.indepth.dev/tutorials/react/vizualize_data_d3_3.png)
### Explanation of above-mentioned Circles.jsx
1. Looping over every data point.
2. Rendering a <circle /> at [x, y].

We all know that D3 is good at keeping track of what elements are new, and animating elements in and out.
Now, take a look at transitions:


```javascript
const AnimatedCircles = () => {
   const [visibleCircles, setVisibleCircles] = useState(
           generateCircles()
   )
   const ref = useRef()

   useInterval(() => {
      setVisibleCircles(generateCircles())
   }, 2000)

   useEffect(() => {
      const svgElement = d3.select(ref.current)
      svgElement.selectAll("circle")
              .data(visibleCircles, d => d)
              .join(
                      enter => (
                              enter.append("circle")
                                      .attr("cx", d => d * 15 + 10)
                                      .attr("cy", 10)
                                      .attr("r", 0)
                                      .attr("fill", "cornflowerblue")
                                      .call(enter => (
                                              enter.transition().duration(1200)
                                                      .attr("cy", 10)
                                                      .attr("r", 6)
                                                      .style("opacity", 1)
                                      ))
                      ),
                      update => (
                              update.attr("fill", "lightgrey")
                      ),
                      exit => (
                              exit.attr("fill", "tomato")
                                      .call(exit => (
                                              exit.transition().duration(1200)
                                                      .attr("r", 0)
                                                      .style("opacity", 0)
                                                      .remove()
                                      ))
                      ),
              )
   }, [dataset])

   return (
           <svg
                   viewBox="0 0 100 20"
                   ref={ref}
           />
   )
}
```
Well, this is a large amount of code!

You don’t need to run through all of it — the gist is that we have 6 <circle>s, and every 2 seconds, we randomly choose some of them to show up.

Okay, so you can see that the above gist is hard to scan. But how can we implement this using React?


```javascript
const AnimatedCircles = () => {
   const [visibleCircles, setVisibleCircles] = useState(
           generateCircles()
   )

   useInterval(() => {
      setVisibleCircles(generateCircles())
   }, 2000)

   return (
           <svg viewBox="0 0 100 20">
              {allCircles.map(d => (
                      <AnimatedCircle
                              key={d}
                              index={d}
                              isShowing={visibleCircles.includes(d)}
                      />
              ))}
           </svg>
   )
}

const AnimatedCircle = ({ index, isShowing }) => {
   const wasShowing = useRef(false)

   useEffect(() => {
      wasShowing.current = isShowing
   }, [isShowing])

   const style = useSpring({
      config: {
         duration: 1200,
      },
      r: isShowing ? 6 : 0,
      opacity: isShowing ? 1 : 0,
   })

   return (
           <animated.circle {...style}
                            cx={index * 15 + 10}
                            cy="10"
                            fill={
                               !isShowing          ? "tomato" :
                                       !wasShowing.current ? "cornflowerblue" :
                                               "lightgrey"
                            }
           />
   )
}
```
Animating elements is not very simple in React. I keep all of the <circle>s rendered, and give them an opacity if they are not in the currently shown circles.
### Explanation of above-mentioned Transitions.jsx
- Loop over our allCircles array and create a <AnimatedCircle> for each item.
- Define an AnimatedCircle component that takes two props: `index` (for positioning), and `isShowing`.
- Caching the last `isShowing` value, so I can see whether the <circle> is entering or exiting.
- To animate our <circle>’s use animated from react-spring and spread our animated values as element attributes. 

You can say that this code is not very short compared to using D3.js code, but it is easy to read.

### Axis
The D3.js API is extensive, and we can become dependent on it to do the heavy stuff for us. Especially with the variety of methods that can make several DOM elements for us.

For example, the `.axisBottom()` function will create a whole chart axis in one line of code.


```javascript
const Axis = () => {
   const ref = useRef()

   useEffect(() => {
      const xScale = d3.scaleLinear()
              .domain([0, 100])
              .range([10, 290])

      const svgElement = d3.select(ref.current)
      const axisGenerator = d3.axisBottom(xScale)
      svgElement.append("g")
              .call(axisGenerator)
   }, [])

   return (
           <svg
                   ref={ref}
           />
   )
}
```

![](https://images.indepth.dev/tutorials/react/vizualize_data_d3_4.png)

### Explanation of above-mentioned Axis.jsx
- Create a scale that converts from the data values (0–100) to the corresponding physical location (10px — 290px).
- Store the <svg> element in a ref and develop a D3 selection object containing it.
- Pass the scale to `.axisBottom()` to make an axisGenerator.
- `.call()` our axisGenerator on our new <g> element.

I think it is pretty easy but we usually prefer to keep things React-ish.
So if we do not want to use `.axisBottom()` to create our axis DOM elements, what would we do?

```javascript
const Axis = () => {
   const ticks = useMemo(() => {
      const xScale = d3.scaleLinear()
              .domain([0, 100])
              .range([10, 290])

      return xScale.ticks()
              .map(value => ({
                 value,
                 xOffset: xScale(value)
              }))
   }, [])

   return (
           <svg>
              <path
                      d="M 9.5 0.5 H 290.5"
                      stroke="currentColor"
              />
              {ticks.map(({ value, xOffset }) => (
                      <g
                              key={value}
                              transform={`translate(${xOffset}, 0)`}
                      >
                         <line
                                 y2="6"
                                 stroke="currentColor"
                         />
                         <text
                                 key={value}
                                 style={{
                                    fontSize: "10px",
                                    textAnchor: "middle",
                                    transform: "translateY(20px)"
                                 }}>
                            { value }
                         </text>
                      </g>
              ))}
           </svg>
   )
}
```

While we do not want to use a D3 function (such as .axisBottom())that creates DOM elements, we might use the D3 methods that D3 uses internally to create axes!

### Explanation of above-mentioned Axis.jsx
- Make a scale that converts from the data values (0–100) to the corresponding physical location (10px — 290px).
- Utilize our D3 scale’s `.ticks()` function.
- Mapping over our array of tick values and make an object that contains the value and xOffset (converted using xScale).
- Make a <path> element that marks that top of our axis. It begins at [9, 0] and moves horizontally to [290, 0].
- For each of the ticks, I want to make a group that is shifted the proper number of pixels to the right.
- Each of my groups will contain a tick <line> and <text> containing the tick value.


Okay! So this is for sure more code. But that makes sense since we’re fundamentally duplicating some of the D3 library code in our own codebase.
  
But here is the thing: the new code is very readable — we know what elements we’re rendering just by looking at the return statement. Plus, we can extract all of this logic into a single Axis component. This we can customize however we like, without having to think about this extra logic again.
  
What should a more reusable Axis component look like?


```javascript
const Axis = ({
                 domain=[0, 100],
                 range=[10, 290],
              }) => {
   const ticks = useMemo(() => {
      const xScale = d3.scaleLinear()
              .domain(domain)
              .range(range)

      const width = range[1] - range[0]
      const pixelsPerTick = 30
      const numberOfTicksTarget = Math.max(
              1,
              Math.floor(
                      width / pixelsPerTick
              )
      )

      return xScale.ticks(numberOfTicksTarget)
              .map(value => ({
                 value,
                 xOffset: xScale(value)
              }))
   }, [
      domain.join("-"),
      range.join("-")
   ])

   return (
           <svg>
              <path
                      d={[
                         "M", range[0], 6,
                         "v", -6,
                         "H", range[1],
                         "v", 6,
                      ].join(" ")}
                      fill="none"
                      stroke="currentColor"
              />
              {ticks.map(({ value, xOffset }) => (
                      <g
                              key={value}
                              transform={`translate(${xOffset}, 0)`}
                      >
                         <line
                                 y2="6"
                                 stroke="currentColor"
                         />
                         <text
                                 key={value}
                                 style={{
                                    fontSize: "10px",
                                    textAnchor: "middle",
                                    transform: "translateY(20px)"
                                 }}>
                            { value }
                         </text>
                      </g>
              ))}
           </svg>
   )
}
```

The Axis component will take two props: domain and range.

### Explanation of above-mentioned Axis.jsx
- I check the range to dynamically alter the number of ticks that I am aiming for (which I can set by giving the number to .ticks()).
- I need to recalculate my ticks when my props change. I have to pay attention to the values of my domain and range arrays, instead of the array reference, so I will .join() them into a string. For example, I have to check a string of “0–100” instead of a reference to the array [0, 100]. This will give me the capability to make the domain and range arrays within Axis’s parent component.
- A little bit of change, but quite important. I will add a duplicate first and last tick mark, in case my ticks don’t cover the top or bottom of our domain.

Now, my Axis component specifically works for axes at the bottom of a chart, at the moment. But for sure this gives you enough of an idea of how simple it is to duplicate D3’s axis drawing functions.

I use this function for recreating any D3 methods that make a number of elements. In addition to the usual advantages of using React to render elements (declarative and less hacky), I find that this code is easier for other developers who are little familiar with the D3 API to understand.

I truly get the best of both worlds, since the D3 API surfaces many of its internal functions.

### Sizing and Responsivity

Sizing charts can be difficult. Because we want to exactly position our data elements, we can’t utilize our usual web development tricks that depend on responsive sizing of <div>s and <spans>.

If you’ve read through many D3.js examples, you will know that there is a common way of sizing charts.

![](https://images.indepth.dev/tutorials/react/vizualize_data_d3_5.png)

### Explanation of above-mentioned image
- Wrapper is the extent of the chart, and the dimensions of the <svg> element.
- Bound contains the data elements, but excludes margins and legends.
- Margins determine the padding around the bounds.

We have to split up our wrapper and bounds boxes because we want to know their exact dimensions when building a chart with <svg>.

```javascript
const chartSettings = {
   "marginLeft": 75
}
const ChartWithDimensions = () => {
   const [ref, dms] = useChartDimensions(chartSettings)

   const xScale = useMemo(() => (
           d3.scaleLinear()
                   .domain([0, 100])
                   .range([0, dms.boundedWidth])
   ), [dms.boundedWidth])

   return (
           <div
                   className="Chart__wrapper"
                   ref={ref}
                   style={{ height: "200px" }}>
              <svg width={dms.width} height={dms.height}>
                 <g transform={`translate(${[
                    dms.marginLeft,
                    dms.marginTop
                 ].join(",")})`}>
                    <rect
                            width={dms.boundedWidth}
                            height={dms.boundedHeight}
                            fill="lavender"
                    />
                    <g transform={`translate(${[
                       0,
                       dms.boundedHeight,
                    ].join(",")})`}>
                       <Axis
                               domain={xScale.domain()}
                               range={xScale.range()}
                       />
                    </g>
                 </g>
              </svg>
           </div>
   )
}
```

This example is a little complex; let me explain what’s going on. The main parts to pay attention to here are as follows:

### Explanation of above-mentioned ChartWithDimensions.jsx

- Utilize a custom hook to calculate the dimensions of our wrapper and bounds (more on this later).
- Utilize the dms object with the calculated dimensions to make an x scale.
- Utilize the React ref from our custom hook to give a non-SVG wrapping element that is the size we need our wrapper to be.
- Change the main part of our chart to respect our top and left margins.

Now that we have an idea of how we could use wrapper, bounds, and margins, let’s see what our custom hook is doing.

```javascript
import ResizeObserver from '@juggle/resize-observer'

const useChartDimensions = passedSettings => {
   const ref = useRef()
   const dimensions = combineChartDimensions(
           passedSettings
   )

   const [width, setWidth] = useState(0)
   const [height, setHeight] = useState(0)

   useEffect(() => {
      if (dimensions.width && dimensions.height)
         return [ref, dimensions]

      const element = ref.current
      const resizeObserver = new ResizeObserver(
              entries => {
                 if (!Array.isArray(entries)) return
                 if (!entries.length) return

                 const entry = entries[0]

                 if (width != entry.contentRect.width)
                    setWidth(entry.contentRect.width)
                 if (height != entry.contentRect.height)
                    setHeight(entry.contentRect.height)
              }
      )
      resizeObserver.observe(element)

      return () => resizeObserver.unobserve(element)
   }, [])

   const newSettings = combineChartDimensions({
      ...dimensions,
      width: dimensions.width || width,
      height: dimensions.height || height,
   })

   return [ref, newSettings]
}
```

When we provide a settings object to our custom useChartDimensions hook, it will do the following:

### Explanation of above-mentioned useChartDimensions.js

- Filling in missing margins by presetting default values.
- Defer to the given height and width, if specified in the passedSettings.
- Utilize a ResizeObserver to recalculate the dimensions when the given element changes size.
- Hold the height and width of a containing <div> for our wrapper dimensions.
- Calculating the dimensions of our bounds (named boundedHeight and boundedWidth). 

The important thing to remember is that any settings that we don’t set are being filled in automatically.

Hope this provides you an idea of how to take care of chart dimensions in a responsive, easy way.

### Maps

So you’ve seen great examples of people using D3 to make detailed maps, and globes that you can spin around. And you want to do that too.

Do not worry! We will let D3 do a lot of the heavy lifting, and have a map in no time!

```javascript
const Map = ({ projectionName = "geoArmadillo" }) => {
   // grab our custom React hook we defined above
   const [ref, dms] = useChartDimensions({})

   // this is the definition for the whole Earth
   const sphere = { type: "Sphere" }

   const projectionFunction = d3[projectionName]
           || d3GeoProjection[projectionName]
   const projection = projectionFunction()
           .fitWidth(dms.width, sphere)
   const pathGenerator = d3.geoPath(projection)

   // size the svg to fit the height of the map
   const [
      [x0, y0],
      [x1, y1]
   ] = pathGenerator.bounds(sphere)
   const height = y1

   return (
           <div
                   ref={ref}
                   style={{
                      width: "100%",
                   }}
           >
              <svg width={dms.width} height={height}>
                 <defs>
                    {/* some projections bleed outside the edges of the Earth's sphere */}
                    {/* let's create a clip path to keep things in bounds */}
                    <clipPath id="Map__sphere">
                       <path d={pathGenerator(sphere)} />
                    </clipPath>
                 </defs>

                 <path
                         d={pathGenerator(sphere)}
                         fill="#f2f2f7"
                 />

                 <g style={{ clipPath: "url(#Map__sphere)" }}>
                    {/* we can even have graticules! */}
                    <path
                            d={pathGenerator(d3.geoGraticule10())}
                            fill="none"
                            stroke="#fff"
                    />

                    {countryShapes.features.map((shape) => {
                       return (
                               <path
                                       key={shape.properties.subunit}
                                       d={pathGenerator(shape)}
                                       fill="#9980FA"
                                       stroke="#fff"
                               >
                                  <title>
                                     {shape.properties.name}
                                  </title>
                               </path>
                       )
                    })}
                 </g>
              </svg>
           </div>
   )
}
```

### Explanation of above-mentioned useChartDimensions.js
- First, we want to create a projection. This is our map between our country shape definitions and the process we draw those 3D shapes on our 2D screen. We’ll utilize the .`fitWidth()` function to size our map within the width of the component, and also make a pathGenerator to create path definitions for our Earth & country shapes using `d3.geoPath()`.
- Next, we’ll find the dimensions of the entire Earth (sphere) in our projection, and give the height of our SVG to the height of the sphere.
- Some projections’ shapes got outside of the edges of the Earth, so we’ll keep them in bounds utilizing a clipPath.
- We can utilize our pathGenerator method to turn GeoJSONshape definitions into <path> d attribute strings. First, we’ll draw the entire Earth in a light gray.
- D3-geo has some great functions, like `d3.geoGraticule10()` which will aid us to draw graticule lines for reference.
- Last but not the least, we will draw our country shapes! We can draw various types of geographic shapes by giving GeoJSON definitions to our pathGenerator method. For example, we’re importing a list of country definitions, then creating <path> elements with their shape.

Once you know the basics down, this is a quite flexible way to draw geography! The trick is to think of D3 as a series of tools.

### Conclusion
- We’re through with the basics! We’ve covered:
- How to draw SVG elements.
- How to draw many SVG elements.
- How to replicate built-in D3 methods for drawing complex elements like axes.
- How to size our charts easily.
- How to draw maps.

From my experience, the most important rule is to know your tools. Once you’re comfortable drawing with SVG, using D3 as a utility library, and building React code, you’ll truly be able to make whatever you can imagine. This is the beauty of learning the fundamentals, instead of just grabbing a chart library — it’s a lot more work to learn, but is way more powerful.
