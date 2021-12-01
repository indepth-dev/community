---
title: How to upgrade React Router in 4 steps
author_name: Imran Farooq
author_link: https://imranfarooqreactjs.medium.com/
discussion_link: https://github.com/indepth-dev/community/discussions/216
display_name: How to upgrade React Router in 4 steps
---

# **How to upgrade React Router in 4 steps**

React-router version 6 was released and this is quite important because react-router is one of the most used and most important react packages that you find out there a lot of react projects need routing and therefore a lot of react projects do use react-router in this article I will walk you through what's new with react-router version 6 and of course I will also show you how you could update an existing react app that's using react-router version 5 to react-router version 6.

Now to learn about react-router version 6 you can of course check out the official website reactrouter.com and the documentation you find there and specifically there also is an upgrading guide where you will find detailed upgrading steps and where you also learn what's new and what changed and this is a quite long document and if you want to have all the details you should definitely also dive into it.

But in the end, it's really simple to upgrade and not a lot changed when it comes to the code that we write under the hood version 6 is a lot better than version 5, and therefore if you can upgrade you should of course strongly consider doing that.

Now to see what changed and write some code I created a little snapshot, a little project snapshot which git repo link will be given at the end of this article does use react-router version 5 so which does not use version 6.

So once you downloaded and extracted that snapshot you should run npm install to install all the core dependencies that come with that project and once you did that you should install react-router version 6 and you do this by running `npm install react-router-dom` and that's important you want `react-router-dom` which is the browser version of react-router and then add `@6` which ensures that you install the latest version `6` and that's what we can do here:

![](https://images.indepth.dev/tutorials/react/react-router-1.png)

So with that installed we got react-router version 6 and if I now run npm start you will see that this project won't work if I try to visit this page I now get an error that switch is not exported from react-router-dom:

![](https://images.indepth.dev/tutorials/react/react-router-2.png)

And that's one of the first changes when using react-router version 5 we use this switch component which is provided by the react-router package to wrap our routes and make sure that only one of these routes is loaded at the same time instead of all matching routes and typically that is what you want you to want to define multiple routes but only one route should be active at a given point of time:

![](https://images.indepth.dev/tutorials/react/react-router-3.png)

Now with react-router version 6 switch doesn't exist anymore instead this now becomes routes, so you replace the switch with routes and you, therefore, import routes from `react-router-dom` instead of `switch`

![](https://images.indepth.dev/tutorials/react/react-router-4.png)

By the way, what hasn't changed is that you still import browser router from `react-router-dom` and you wrap that around your app where you then plan to use routing that's exactly the same syntax as you know it:

![](https://images.indepth.dev/tutorials/react/react-router-5.png)

With that, the error goes away but if I click welcome or products you see that the URL changes but I still don't see anything on a screen:

![](https://images.indepth.dev/tutorials/react/react-router-6.png)

The reason for this is that the way how we define our routes also changed we still have the route component and it still takes a path prop:

![](https://images.indepth.dev/tutorials/react/react-router-7.png)

But the component that should be loaded when a given path becomes active so in this case, the welcome component, for example, is no longer a child of your route but instead on the route you add a new element prop and then you pass a dynamic value to element and that dynamic value is that to be rendered component as jsx that's important:

![](https://images.indepth.dev/tutorials/react/react-router-8.png)

So you don't pass welcome as a string you don't pass a pointer to that component function instead you pass the jsx element hence the name element it wants a jsx element and then this can, of course, become a self-closing component this route component and we do the same here for products we also add the element prop and we then move products into this prop whereas a value we set it as a value for this prop and we turn this into a self-closing component and last but not least we also do this for this last route and please note that this is a dynamic route and that still works as you learned it so you can still have dynamic path parameters like product id as you could with version five that did not change:

![](https://images.indepth.dev/tutorials/react/react-router-9.png)

With that, once the routes were restructured now you see this welcome page content here and I can switch to products back and forth:

![](https://images.indepth.dev/tutorials/react/react-router-10.png)

![](https://images.indepth.dev/tutorials/react/react-router-11.png)

If I select a product I also do see those product details so those detail routes also work and therefore a simple app using routes like this can easily be updated to version 6 by replacing the switch with routes and moving the to be loaded component into a new element prop.

## Github Repo Link

https://github.com/imran-farooq7/react-router-v6-update

