# React notes on advanced topics

- React Jargon Explained by Sam Selikoff: https://www.youtube.com/watch?v=bZRqmobuJvM

## Rendering

React renders user interfaces in two phases.

- Phase 1: Render Phase
  - At the first render (new build), react executes all components to return a JSX tree as a result.
  - At subsequent renders (updates), react compares the previous and updated JSX trees to compute the diff of a JSX tree and returns this as a result. This is called reconciliation.
- Phase 2: Commit Phase
  - React will update the DOM with this JSX tree. For example, I added a new h1 tag and in the rerender, react inserts the new JSX element into the DOM using the createElement method as follows: document.createElement("h1");

## State

- State is the runtime data that react relies upon to render the app visually.
- Data that changes over time that is part of the app and not react state include:
  - scroll position
  - mouse position
  - focus elements
  - highlighed text in the browser
  - these types of data do not trigger react re-renders
- The primary way to update react state is using the updated function e.g. setState, setCount

## Side Effects

- A side effect is any code that does not directly create or manipulate JSX that include:
  - fetching data from an API
  - updating the document's title
  - using console.log
  - using date functions (each new day has a new date)

## Purity

- Render functions should be pure functions.
- A pure function has no side effects and its output depends on the input only.
- Using the useEffect hook makes react functional component a pure function.
- The useEffect hook separates side effects from the jsx.
- The side effects are defined and executed inside the useEffect hook.
- In a react functional component, the useEffect contains side effects and the render function contains the jsx.

## Idempotence

- An idempotent function has side effects that have the same effect on the overall system state no matter how many times the idempotent function gets called.
- You can run an idempotent function once or 20 times or as many times as you want and you will still get the same result.
- In react, you want to make sure side effects are idempotent.
- The useEffect hook provides a clean up functions to ensure side effects are idempotent.
- The component can mount and unmount without accruing subscriptions.
- For example, cleaning up a subscription. The subscription is calling a webhook using addEventListener. The cleanup is inside the return statement using removeEventListener.
- Example code:

```
  export default function Messages() {
    let [messages, setMessages] = useState([]);

    useEffect(() => {
      function handleMessage(message) {
        setMessages([...messages, message.data]);
      }

      websocket.addEventListener("message", handleMessage);

      // cleanup
      return () => {
        websocket.removeEventListener("message", handleMessage);
      };
    }, [messages]);

    return (
      <u1>
        {
          messages.map((message) => {
            <li key={message.id}>{message.text}</li>
          })
        }
      </ul>
    );
  }
```

- For example, a database seed is a seed function in a database file that gets called when an application is deployed.
- This could be a seed function that lists a list of countries that are displayed in a dropdown menu.
- Everytime you deploy an application, the seed functions gets called and has the same output in the application so that the list of countries in the dropdown menu is the same.
- Example code:

```
  /*
    Generate a list of countries the app relies on.
  */
  export default function idempotentSeed() {
    let name = "United States of America";

    // This check makes this function idempotent
    if (!db.country.findBy({name})) {
      db.country.create({name});
    }

    // Many ORMs have helper methods that to the same thing as the code above
    // db.country.createOrFindBy({name: "United States of America"});

  }
```

## Heuristics

- Heuristics are guiding principles for react development and reduce bugs.
- For example, the hueristic of render being pure and render being idempotent.
- React can't deterministically find side-effects in components. So, it's important to learn what a side-effect is so we can put it inside a useEffect hook and make it idempotent so it can be rendered over and over again.

## Concurrency

- Concurrent react is a set of features in React 18.
- React 17 contained a feature called synchronous rendering. This means that during an initial render or update, the execution from the render phase to the commit phase cannot be interrupted.
- React 18 introduces a new feature called Asynchronous (concurrent) rendering (to solve the above issue).
- Rendering can now be interrupted during the process of generating a diff of a jsx tree from the render phase to the commit phase of an initial render or an update render.
- For example, when a user is typing something into an input field we might need to re-render a large graph. This takes react a lot of work during the render phase to calculate a new jsx tree and it can even lock up / freeze the browser making the user interface less responsive. If react is able to keep the two phases separate, it can more intelligently create and remove jsx trees.
- The heuristic of render being pure is very important to make React 18 Asynchronous (concurrent) rendering feature useful.
- If react can assume our components are pure (taking an input, calculating a jsx tree, no side-effects, not mutating global state), reat can render different versions of our app at the same time without committing them to the browser.
- For example, preparing another screen as the user is swiping across screens in an app. A modal with buttons to move to the next screen, react can prepate the next screen in the background so that the new screen is ready to be animated in when the user clicks on a 'next' button. You don't have to wait for the user to click on a 'next' button to manipulate the DOM.
- For example, the user could also click on a 'cancel' button and the screen being prepared in the background can be removed.
- React 18 apps are now much more responsive and resilient under these heavy load / heavy render situations.
