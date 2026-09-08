1. React used to run on class components earlier but now runs on functional components - {


    This is about how the way you *write* React components changed over time — not a totally different framework, just a different pattern for building the same kinds of components.

    **Class components (the older way)**

    Before 2019, if you wanted a component with state or lifecycle logic (like "do something when this mounts" or "do something when props change"), you had to write it as a JavaScript class:

    ```jsx
    class Counter extends React.Component {
    constructor(props) {
        super(props);
        this.state = { count: 0 };
    }

    increment = () => {
        this.setState({ count: this.state.count + 1 });
    };

    componentDidMount() {
        console.log("Component mounted");
    }

    render() {
        return (
        <div>
            <p>{this.state.count}</p>
            <button onClick={this.increment}>+</button>
        </div>
        );
    }
    }
    ```

    State lived in `this.state`, you updated it with `this.setState()`, and lifecycle events (mount, update, unmount) were separate methods like `componentDidMount`, `componentDidUpdate`, `componentWillUnmount`.

    **Functional components + Hooks (the current way)**

    In React 16.8 (early 2019), React introduced **Hooks** — functions like `useState` and `useEffect` that let plain JavaScript functions have state and lifecycle behavior, without needing a class at all:

    ```jsx
    function Counter() {
    const [count, setCount] = useState(0);

    useEffect(() => {
        console.log("Component mounted");
    }, []);

    return (
        <div>
        <p>{count}</p>
        <button onClick={() => setCount(count + 1)}>+</button>
        </div>
    );
    }
    ```

    Same behavior, much less boilerplate — no `this`, no constructor, no binding methods.

    Why the shift happened

    - `this` in JavaScript classes is genuinely confusing (binding issues were a constant source of bugs)
    - Class components made it hard to reuse stateful logic between components — you ended up with awkward patterns like "render props" or "higher-order components" just to share logic
    - Related logic in `componentDidMount` and `componentDidUpdate` often had to be split across different lifecycle methods even though it was the same concern. `useEffect` lets you keep it together.
    - Functions are just easier to read, test, and reason about than classes for most people
}



2. What are hooks in React - {

    Hooks are special functions that let functional components "hook into" React features — like state and lifecycle behavior — that used to only be available in class components. They all start with `use` (that's a naming convention React enforces, not just style).

    Think of a plain function component without hooks as a function that runs, returns some JSX, and then forgets everything — it has no memory between renders. Hooks are how React gives that function memory and a way to plug into things happening around it (mounting, updates, browser APIs, etc.).

    **The two you already know, formalized:**

    **`useState`** — gives a component memory (state) that persists across re-renders.
    ```jsx
    const [count, setCount] = useState(0);
    ```
    `count` is the current value, `setCount` is how you update it, and calling `setCount` triggers React to re-render the component with the new value. Without this hook, a regular variable inside a function would just reset to its initial value every render.

    **`useEffect`** — lets you run "side effects" — things that reach outside of rendering, like API calls, subscriptions, timers, or manually touching the DOM.
    ```jsx
    useEffect(() => {
    fetchSongs();
    }, []);
    ```
    The empty `[]` at the end means "only run this once, when the component mounts" — this is what replaces `componentDidMount`. If you put variables in that array, the effect re-runs whenever those variables change.

    **A few other common ones you'll run into:**

    - **`useRef`** — holds a mutable value that persists across renders but *doesn't* trigger a re-render when changed. Often used to directly reference a DOM element (like grabbing an `<audio>` or `<input>` tag directly).
    - **`useContext`** — lets a component read shared data (like a logged-in user or theme) without passing props down through every level manually.
    - **`useMemo` / `useCallback`** — performance hooks that "remember" a computed value or function between renders so it isn't recalculated unnecessarily.

    **The core rule of hooks:** they must be called at the top level of a component, in the same order every render — never inside loops, conditions, or nested functions. React relies on call order to match each hook to its stored data internally, so breaking that rule causes bugs that are confusing to track down.

    **One more concept: custom hooks.** Once you're comfortable with the built-in ones, you can write your own hook (a function starting with `use` that calls other hooks inside it) to package up reusable logic — e.g. a `useFetch(url)` hook you write once and reuse across components instead of copy-pasting `useEffect` + `useState` logic everywhere.

    Hooks are just functions that let a React component "remember" things and "do" things — without needing to write a class.

    - **`useState`** = gives a component memory. Store a value, and when you update it, the screen re-renders with the new value.
    ```jsx
    const [count, setCount] = useState(0);
    ```

    - **`useEffect`** = do something extra after the component shows up — like fetching data or setting up a timer.
    ```jsx
    useEffect(() => {
        fetchSongs();
    }, []);
    ```

    That's really it at the core: **state** (remember stuff) and **effects** (do stuff outside rendering, like API calls). Everything else (`useRef`, `useContext`, etc.) is a variation for more specific situations.

    One rule to keep in mind: always call hooks at the top of your component, never inside an `if` or a loop.

}

3. Components and Props in React - {

    **Components** are the building blocks of a React app — reusable pieces of UI, each written as a function that returns JSX (HTML-like syntax).

    Think of a webpage like Lego: instead of one giant HTML file, you break the UI into small, reusable pieces — a `Navbar`, a `SongCard`, a `Button` — and combine them.

    ```jsx
    function Button() {
    return <button>Click me</button>;
    }

    function App() {
    return (
        <div>
        <Button />
        <Button />
        </div>
    );
    }
    ```

    `Button` here is a component. `App` uses it twice — that's the whole point: write once, reuse anywhere.

    **Props** ("properties") are how you pass data *into* a component — like function arguments, but for JSX.

    ```jsx
    function Button(props) {
    return <button>{props.label}</button>;
    }

    function App() {
    return (
        <div>
        <Button label="Play" />
        <Button label="Pause" />
        </div>
    );
    }
    ```

    Same `Button` component, but each usage gets different data (`label`) passed to it, so it renders differently. Props are **read-only** — a component receives them but can't change them itself (that's what `useState` is for, inside the component itself).

    **The mental model that ties it together:**
    - **Components** = the reusable UI pieces (like functions)
    - **Props** = the inputs to those pieces (like function arguments)
    - **State** = memory the component keeps *inside itself*, that changes over time


}

4. Why this is wrong in React - <button onClick={seta(a + 1)}>Change a</button> ? - {

    The bug is here:

    <button onClick={seta(a + 1)}>Change a</button>

    `onClick` expects a **function reference** — something to call later, when the button is clicked. But `seta(a + 1)` *calls* `seta` immediately, during render, instead of passing a function.

    So what actually happens: every time `App` renders, `seta(a + 1)` runs right away → state updates → component re-renders → `seta(a + 1)` runs again → infinite loop. You'd likely see React throw a "Maximum update depth exceeded" error, or the page just freeze/crash.

    **Fix:** wrap it in an arrow function so it only runs *on click*, not during render:

    ```jsx
    <button onClick={() => seta(a + 1)}>Change a</button>
    ```

    This creates a new function each render, but that function only calls `seta` when the button is actually clicked — not immediately.

    **Rule of thumb:** `onClick={someFunction}` = pass the function itself (fine if it takes no arguments and needs no closure). `onClick={someFunction()}` = calls it immediately during render (almost always wrong). `onClick={() => someFunction(arg)}` = wrap it, so it runs later, with whatever arguments you need.

    Look at this line:

    ```jsx
    <button onClick={seta(a + 1)}>Change a</button>
    ```

    `seta(a + 1)` — with the parentheses — means "run this right now." So React runs it the moment it builds the button, not when you click it. That changes `a`, which makes the component re-render, which runs `seta(a + 1)` again... forever. It just loops and crashes.

    You want to say "run this *when clicked*," not "run this *right now*." So wrap it in a mini function:

    ```jsx
    <button onClick={() => seta(a + 1)}>Change a</button>
    ```

    `() => seta(a + 1)` means "here's something to do later." Now it only runs when you actually click the button.

    **Simple rule:** if you're calling a function with `()`, do it inside an arrow function `() => ...` when it's in `onClick`. Otherwise it fires immediately instead of waiting for the click.

    Let's slow this down step by step, tracing through what actually happens when React renders this:


    <button onClick={seta(a + 1)}>Change a</button>

    **Step 1:** React needs to build the button, so it evaluates everything inside the curly braces — including `seta(a + 1)`. Since it has `()`, it *executes* right there, immediately. It's not "the thing to run on click" — it's "run me now, and whatever I return becomes the value of `onClick`."

    **Step 2:** `seta(a + 1)` runs → state updates from `10` to `11`.

    **Step 3:** Whenever state changes, React re-renders the component — it calls `App()` again from scratch to rebuild the UI with the new value.

    **Step 4:** But rebuilding the UI means evaluating that same JSX line again — and `seta(a + 1)` gets executed again, immediately, just like Step 1. This time it updates state from `11` to `12`.

    **Step 5:** That state change triggers another re-render → which hits `seta(a + 1)` again → another state change → another re-render...

    This keeps happening, non-stop, because every render *itself* causes the next state change, and every state change causes another render. Nothing ever waits for you to click — the button never even gets a chance to display before it's already looping.

    **Why wrapping it in `() => ...` fixes this:**

    ```jsx
    <button onClick={() => seta(a + 1)}>Change a</button>
    ```

    `() => seta(a + 1)` is not executed during render. It's just *created* — like writing down an instruction on a sticky note without doing it yet. React stores that sticky note as `onClick`. Nothing runs, state doesn't change, no re-render happens. The instruction only actually executes later, when the click event fires, and only *then* does `seta` get called — once per click, not infinitely.

    **The one-line version:** `seta(a+1)` = "do it now." `() => seta(a+1)` = "do it later, when I say so."
}


5.  What is Named Import/Export and Default Import/Export ? - {


    The core difference is **how many** you can have per file, and **how strictly the name is tied** on import.

    ## Named Export/Import

    **Export:** You can have **multiple** named exports per file.
    ```js
    // utils.js
    export const PI = 3.14;
    export function add(a, b) { return a + b; }
    ```

    **Import:** You must use the **exact same name**, wrapped in curly braces `{ }`.
    ```js
    import { PI, add } from './utils.js';
    ```

    If you want a different name, you must explicitly rename with `as`:
    ```js
    import { PI as CircleConstant } from './utils.js';
    ```

    ## Default Export/Import

    **Export:** Only **one** default export allowed per file.
    ```js
    // greet.js
    export default function greet(name) {
    console.log(`Hi, ${name}`);
    }
    ```

    **Import:** No curly braces — and you can name it **anything you want**, since there's only one thing being imported.
    ```js
    import greet from './greet.js';
    import sayHello from './greet.js';  // also valid, same import
    import whatever from './greet.js';  // still works
    ```

    ## Side-by-side

    | | Named | Default |
    |---|---|---|
    | Per file | Multiple allowed | Only 1 allowed |
    | Import syntax | `import { name }` (curly braces) | `import name` (no braces) |
    | Import name | Must match export (unless using `as`) | Can be anything |
    | Common use | Utility functions, constants, multiple related exports | The "main thing" a module provides (a component, a class) |

    ## In practice (React example)

    ```jsx
    // Button.jsx
    export default function Button() { ... }       // the component itself
    export const BUTTON_SIZES = ['sm', 'md', 'lg']; // a helper constant

    // Importing elsewhere
    import Button, { BUTTON_SIZES } from './Button';
    ```

    This pattern — default export for the main component, named exports for helpers/constants — is very common in React codebases, so it's worth getting comfortable with mixing the two.
}


6.  Explain Real and Virtual DOM ?  - { 

            ## Real DOM

        The **DOM (Document Object Model)** is the browser's live, tree-structured representation of your HTML page. Every element, attribute, and text node becomes an object in this tree, and JavaScript can read/manipulate it directly.

        ```html
        <div id="app">
        <h1>Hello</h1>
        <p>World</p>
        </div>
        ```

        This becomes a tree of node objects that the browser renders visually. When you do:

        ```js
        document.getElementById('app').innerHTML = '<h1>Hi</h1>';
        ```

        ...you're directly mutating the *real* DOM.

        ### Why manipulating the real DOM is expensive

        Every time you change something in the real DOM, the browser potentially has to:
        1. **Recalculate styles** (which CSS rules apply)
        2. **Reflow/Layout** (recompute size/position of elements — and often their neighbors too)
        3. **Repaint** (redraw pixels on screen)

        This pipeline is costly. If you're updating the DOM frequently (e.g., in a list with 1,000 items, changing one item), doing it naively — say, in a loop — can cause the browser to reflow/repaint repeatedly, making the UI slow and janky.

        ```js
        // Bad: triggers reflow/repaint on every iteration
        for (let i = 0; i < 1000; i++) {
        document.getElementById('list').innerHTML += `<li>${i}</li>`;
        }
        ```

        ## Virtual DOM

        The **Virtual DOM (VDOM)** is a concept popularized by React (also used by Vue). It's a lightweight, in-memory **JavaScript representation** of the real DOM — essentially a plain JS object tree that mirrors what the UI *should* look like.

        ```js
        // Simplified idea of what a VDOM node looks like
        {
        type: 'div',
        props: { id: 'app' },
        children: [
            { type: 'h1', props: {}, children: ['Hello'] }
        ]
        }
        ```

        ### How it works (the reconciliation process)

        1. **Render:** When state/props change, React creates a **new** virtual DOM tree representing the updated UI.
        2. **Diff:** React compares (diffs) this new tree against the **previous** virtual DOM tree — this is called **reconciliation**.
        3. **Patch:** React calculates the *minimal* set of changes needed and applies **only those changes** to the real DOM — not a full re-render.

        ```jsx
        function Counter({ count }) {
        return <div>Count: {count}</div>;
        }
        ```

        If `count` changes from 1 to 2, React doesn't recreate the whole `<div>` in the real DOM — it diffs the trees, sees only the text changed, and updates just that text node.

        ## Why Virtual DOM helps

        | | Real DOM directly | Virtual DOM (React) |
        |---|---|---|
        | Updates | Every change can trigger reflow/repaint | Changes are batched and diffed first |
        | Cost | Expensive DOM operations happen often | Cheap JS object comparisons happen often; DOM touched minimally |
        | Developer experience | You manually track *what* changed | You just describe the *desired* UI state; React figures out the diff |

        The key insight: **JS object creation/comparison is fast**; **real DOM manipulation is slow**. The Virtual DOM lets you do all your "what changed?" computation in fast JS-land, and only touch the slow real DOM for the actual minimal update.

        ### A quick analogy
        Imagine editing a 500-page document. Instead of tearing out and reprinting every page each time you fix a typo (real DOM), you keep a draft copy, compare it to the last printed version, and only reprint the *one page* that changed (virtual DOM).

        ---

        Since you're coming from the Sigma course and moving into React, this diffing/reconciliation concept is exactly what's happening under the hood every time you call `setState` or update a hook — worth keeping in mind as you build components, since it explains *why* React encourages patterns like keys in lists (`key` helps the diffing algorithm match elements efficiently).


   }


   7. Concept of Reconciliation - {

            **Reconciliation** is the algorithm React uses to figure out *what changed* between two virtual DOM trees, so it can update the real DOM as efficiently as possible.

        ## The core problem it solves

        Every time state or props change, React doesn't know exactly what's different — it just re-runs your component function(s) and gets a **brand new virtual DOM tree**. Reconciliation is the process of comparing this new tree to the **previous** tree to compute the minimal set of real DOM changes needed.

        ```
        Old Virtual DOM Tree  →  [DIFFING ALGORITHM]  →  Minimal changes  →  Real DOM
        New Virtual DOM Tree  ↗
        ```

        ## How the diffing works

        Comparing two arbitrary trees node-by-node is theoretically very expensive (O(n³) in the general computer science sense). React makes this fast by using a **heuristic algorithm** based on two assumptions:

        ### 1. Different element types → tear down and rebuild
        If the root element type changes, React doesn't try to compare children — it just destroys the old subtree entirely and builds a new one from scratch.

        ```jsx
        // Before
        <div><Counter /></div>

        // After  
        <span><Counter /></span>
        ```

        Since `div` became `span`, React unmounts everything inside (losing `Counter`'s state) and mounts fresh.

        ### 2. Same element type → compare and update in place
        If the type stays the same, React keeps the underlying DOM node and just **updates the changed attributes/props**.

        ```jsx
        // Before
        <div className="old" title="hi" />

        // After
        <div className="new" title="hi" />
        ```

        React sees it's still a `div`, so it keeps the same real DOM node and only patches `className` — `title` is left untouched.

        ### 3. Lists → this is where `key` matters

        For a list of children, React by default compares them **by position** (index). This causes problems:

        ```jsx
        // Before
        <li>Apple</li>
        <li>Banana</li>

        // After (inserted "Mango" at the start)
        <li>Mango</li>
        <li>Apple</li>
        <li>Banana</li>
        ```

        Without keys, React compares index-by-index: it thinks item 0 changed from "Apple" to "Mango", item 1 changed from "Banana" to "Apple", and a new item 2 "Banana" was added. That's 3 updates for what should've been 1 insertion.

        With a stable `key` on each item:

        ```jsx
        <li key="apple">Apple</li>
        <li key="banana">Banana</li>
        ```

        React matches elements by key across renders, realizes "Apple" and "Banana" just moved down, and only inserts the new "Mango" node — much cheaper, and it correctly preserves component state tied to each item.

        ## Putting it together — the phases

        1. **Render phase:** React calls your components, builds the new virtual DOM tree, and diffs it against the old one using the rules above. This is pure computation — no real DOM touched yet.
        2. **Commit phase:** React takes the list of computed changes ("patches") and applies them to the real DOM in one batch.

        Splitting it this way means React can even pause/interrupt the render phase (this is what enables features like concurrent rendering) since nothing has actually touched the screen yet — only the commit phase causes visible, irreversible DOM changes.

        ## Why this matters practically

        - Always give list items a **stable, unique `key`** (not array index if the list can reorder) — this is the single most common reconciliation gotcha.
        - Conditionally rendering different element types at the same position (`<div>` vs `<span>`) forces full remounts — sometimes surprising if you expected state to persist.
        - Understanding this explains *why* `setState` doesn't update the DOM synchronously — it schedules a re-render, and reconciliation decides what actually gets touched.


   }
