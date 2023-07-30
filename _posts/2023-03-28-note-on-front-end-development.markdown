---
layout: post
title: Notes on Front-end development
date: '2023-03-28 10:06:20'
tags:
- technical
---

Front-end development is quick to get started. As projects grow and code build up, there are nuances that may help us avoid the painful road of catching intermittent bugs. Below are a few things I would like to note. Key points are to keep the application state's lean, make program flow obvious, and add checks at your build pipeline. 

## Encapsulate actions, not just components

Think about the time when you need to re-use a component in multiple screens to accomplish something. For example, to ask the user if she really wants to do something, a popup modal is often deployed across screens. One way to do that is to create a shared ConfirmModal component, and embed it wherever it is needed. 

```javascript
<ConfirmModal open={openConfirmModal} 
	message={"Are you sure you want to delete X?"}
	onAccept={positiveAction}
	onCancel={() => setOpenConfirmModal(false)}/>

function dangerousAction() {
	setOpenConfirmModal(true);
}
```
That's a first step towards [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). Yet we can take one step further. Notice that, the host component does not need to manage the open/close state of the `ConfirmModal`. Rather, encapsulate the intent as a single function, and only return to the host page what it cares about. 

```javascript
function dangerousAction() {
	const confirm = await confirm('Are you sure you want to delete X?');
	if (confirm) {
		//... continue with dangerousAction 
	}
}
```

## Choose a minimal state

Frontend application is stateful. UI components are tied to a set of flags to know if users have clicked on a checkbox or selected another tab. Some state flags are variables that get updated by interrupts (events) triggered by network callback, an interval timer. It's important to choose a minimum set of flags that reflect your application state. Avoid having state variables that are derivative of another. 

This is bad

```typescript
{
	user: User;
	isLoggedIn: boolean;
}
```

`isLoggedIn` is redundant as its value can be inferred from `user`. Having it creates responsibility to update variables when needed. 

## Avoid abusing observer pattern

Modern frameworks like Reactjs has built-in constructs to watch for changes in state variable. For example, when users select a new shipping address, another flow should be triggered to update delivery fee. I often see code along the line below, which works most of the time, except for when it does not, it leads to hard to find bug. Object comparison is not exactly the best way to check for changes. And if for some reason, user state data got refetched, the handler would be triggered again. 

```javascript

const selectAddress = () => {
	// this function get called somewhere 
	// ....
	setShippingInfo(newAddr);
}

useEffect(() => {

	updateCurrentCartShippingFee(shippingInfo);
}, [shippingInfo])
```
To me, it's way better to make the flow obvious
```javascript
const selectAddress = () => {
	// ... do something 
	setShippingInfo(newAddr);
	updateCurrentCartShippingFee(newAddr);
}
``` 		

## Invest in your build time

Developing for UI is often developing with multiple quick iterations. Ideally, one would visualize the UI arrangement in his head and churn out css for as long as possible before having to look at the actual changes. In practice, the frequency of going back and forth between the code editor and the UI is high. Bret Victor, in his talk on [inventing on principle](https://www.youtube.com/watch?v=PUv66718DII), stressed the importance of having an immediate feedback loop when creating. I think it's especially important for UI development. Maybe it's part of the reason for web technologies' presence in areas it's not initially meant for like desktop & mobile app development. The web's instant reload is faster to iterate than other frameworks like say Qt. 

So, if your development project takes long to refresh UI, view it as a critical problem. Take a look at alternative build tools, or split stable parts of the code into sub-packages that don't need to be rebuilt everytime. 


## Share data across components

Often, one component needs to communicate its change to others outside of its hierarchy. Two patterns to go about it:
- shared global state: basically, every component has access to a shared object or any of its child objects, which is the approach framework like Redux employs. Beside the component has access to the setter function of that shared object. When it needs to tell other part of the app to change, it updates the shared state. Pros: easy to debug by logging out the global state's snapshot. Cons: state is not obvious signal of action, probably you have to watch for change in state instead. 
- event bus: by implementation, event bus is also a shared global object. But it's different in that the object holds no state, except for a list of subscribers. With this pattern, an object broadcast immutable events, and other components act on those if they are interested. Each component maintains its own state instead of relying on that of the parent. Pros: modular components. Cons: manage component lifecycle to unsubscribe properly. 

## Add checks in your pipeline 

Website speed is important so it's better to be frugal on how much scripts get downloaded to the browser. More often than not, someone may add a library which is either bloated or not tree-shake friendly. When people notice the unusual bundle size, it can be a few MRs away from the causing one. Then it would be really hard to filter through past changes. In my experience, having checks that automatically fail pipeline when certain metrics degrade is really helpful. 

## Pick the right framework

Despite its popularity, I think React is more suitable for a small team of 2-3 developers than for larger project. It has many ways of doing things for users to decide, and a lot could go wrong if someone is new to the framework. For example, consider `useEffect` and its dependency array.

```javascript
const fetchProductData = useHelper();

useEffect(() => {
	// ... 
	fetchProductData();
}, [fetchProductData]);
```
Linter often suggests adding `fetchProductData` to the dependency array. But if fetchProductData is not wrapped by an useCallback, it risks pushingg CPU to 99% usage. For the unsuspectful, it's hard to notice until suffering the pain a few times. 

For larger team, I would suggest going with other more opinionated frameworks, say Angular (I have not tried the framework though).
