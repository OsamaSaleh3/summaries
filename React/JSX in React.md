
## 1. Core Objective

The primary goal of this video is to demystify the foundational mechanics of React by building a bare-bones application entirely from scratch, bypassing modern build tools like Vite or Webpack. The viewer learns how the React library interacts directly with the browser's Document Object Model (DOM) using pure JavaScript, establishing a deep understanding of what happens under the hood before abstractions like JSX are introduced.

## 2. Key Concepts Explained

- **React vs. React-DOM Separation of Concerns:** The instructor highlights a critical architectural decision in React. The `react` package is an abstract, universal interface that manages components, state, and the Virtual DOM. It is environment-agnostic. The `react-dom` package is the specific "renderer" that translates React's internal state into actual HTML elements for the web browser. This is why React Native or React 360 can exist; they use the same `react` core but swap out the rendering engine.
    
- **Zero-Build React via UMD:** Instead of using an NPM build process, the code imports React using Universal Module Definition (UMD) builds via `unpkg` (a CDN). This demonstrates that React, at its core, is just a JavaScript library that can be executed directly by the browser engine.
    
- **The Component vs. Element Paradigm (The Stamp Analogy):** The instructor uses a "stamp" analogy to explain instantiation. A React _Component_ (like the `App` function) is the blueprint or the physical rubber stamp. A React _Element_ (the output of `React.createElement`) is the ink left on the page after the stamp is pressed.
    
- **Predictive Debugging:** A vital mental model introduced is pausing to ask, "What do I expect to happen?" before executing code. This forces the developer to trace the execution context and the component tree mentally, rather than relying strictly on browser output.
    

## 3. Code Logic & Architecture

The instructor builds the foundation using raw HTML and pure JavaScript. Here is the architectural breakdown of the provided logic.

### The HTML Host



```HTML
<div id="root">not rendered</div>
```

This is the mounting point for the React application. The instructor strategically places "not rendered" inside the div. When React successfully mounts, it entirely replaces the contents of this node. If the developer sees "not rendered" on the screen, it acts as an immediate visual failure indicator.

### Component Definition (`App.js`)



```JavaScript
const App = () => {
  return React.createElement(
    "div",
    {},
    React.createElement(
      "h1", 
      {}, 
      "Padre Gino's"
    )
  );
};
```

This function is a React Component. The `React.createElement` function signature accepts three primary arguments:

1. **Type:** The HTML tag as a string (e.g., `"div"`) or another React component.
    
2. **Props:** An object representing attributes (like `id`, `className`, or custom props). Passed as an empty object `{}` here.
    
3. **Children:** What goes inside the element. This can be text or nested `React.createElement` calls.
    

This code is the exact output that modern bundlers produce when they transpile JSX.

### The Rendering Layer (`App.js`)



```JavaScript
const container = document.getElementById("root");
const root = ReactDOM.createRoot(container);
root.render(React.createElement(App));
```

Here is where `react` meets `react-dom`:

- The script grabs the physical DOM node.
    
- `ReactDOM.createRoot` initializes React 18's concurrent rendering engine on that specific node.
    
- `root.render()` instructs React to take the `App` component blueprint, create an element instance out of it via `React.createElement`, and paint it onto the browser's DOM.
    

## 4. Best Practices & Pitfalls

- **Best Practice: Fail-Visible UI:** Putting fallback text inside the root div ensures mounting failures are caught immediately without needing to open the console.
    
- **Best Practice: Mental Execution:** The instructor emphasizes predicting output before saving. This builds a robust mental model of the Virtual DOM and execution flow.
    
- **Anti-Pattern Addressed:** Writing React without a build step using `React.createElement` directly is highly discouraged for production. It is verbose, unmaintainable, and lacks modern developer experience features (like Hot Module Replacement). The instructor explicitly notes that no one does this in reality, but it is taught purely to strip away the "magic" of bundlers and JSX.
    

## 5. Key Takeaways (TL;DR)

- React is fundamentally just a JavaScript library; it does not strictly require Node.js, Webpack, or a build step to function in a browser.
    
- `react` is the abstract engine; `react-dom` is the web-specific paint brush.
    
- `React.createElement` is the core engine function that turns your component blueprints into renderable instances.
    
- Understanding raw React components without JSX is essential for understanding how React actually parses and constructs the UI tree.
  
  
  ---

### 1. Core Objective

The primary goal of this video is to introduce the concept of component reusability and dynamic data injection in React using "Props." The viewer learns how to encapsulate UI logic into a single, flexible component and render multiple distinct instances of it, while also gaining insight into the historical design philosophy that makes React unique.

### 2. Key Concepts Explained

- **Component Encapsulation & Reusability:** The instructor expands on the "stamp" analogy from previous lessons. By creating a `Pizza` component, the logic and UI structure for a pizza are isolated in one place. This makes the component highly reusable across different parts of an application (e.g., a menu page, an order history page) without duplicating code.
    
- **Returning Sibling Elements (Arrays):** In pure React, a component typically returns a single node. To return multiple sibling elements (like an `h1` and a `p` tag side-by-side) without wrapping them in an unnecessary parent `div`, you can return an array of `React.createElement` calls.
    
- **Props (Properties):** Props are the mechanism React uses to pass data from a parent component down to a child component. This turns static components into dynamic templates that render different outputs based on the configuration objects passed to them.
    
- **React's Architectural Philosophy vs. MVC:** The instructor provides valuable historical context. Older frameworks (like Backbone or Angular 1.x) strictly separated the Model, View, and Controller (or ViewModel). React challenged this by observing that grouping the View and ViewModel together into small, encapsulated components—similar to early PHP page models—makes UI development much faster and more maintainable.
    

### 3. Code Logic & Architecture

Here is the underlying logic of the application being built, translated from the instructor's raw `React.createElement` explanations.

**The Child Component (`Pizza`)**



```JavaScript
const Pizza = (props) => {
  return [
    React.createElement("h1", {}, props.name),
    React.createElement("p", {}, props.description)
  ];
};
```

- **How it works:** The `Pizza` component is declared as a function that accepts a single argument: `props`. Instead of hardcoding "Pepperoni", it reads `props.name` and `props.description`. It returns an array of two React elements, allowing the heading and paragraph to render as direct siblings.
    

**The Parent Component (`App`)**



```JavaScript
const App = () => {
  return React.createElement(
    "div",
    {},
    [
      React.createElement("h1", {}, "Padre Gino's"),
      React.createElement(Pizza, {
        name: "The Pepperoni Pizza",
        description: "Some dope pizza"
      }),
      React.createElement(Pizza, {
        name: "The Americano",
        description: "French fries and hot dogs"
      }),
      React.createElement(Pizza, {
        name: "The Hawaiian",
        description: "Pineapple and ham"
      })
    ]
  );
};
```

- **How it works:** Inside the `App` component, we pass an array of children to the main wrapper `div`. When calling `React.createElement(Pizza, {...})`, the second argument—normally reserved for HTML attributes—is used to pass the `props` object to the `Pizza` component. React handles taking those key-value pairs and injecting them into the `Pizza` function.
    

### 4. Best Practices & Pitfalls

- **Pitfall - Missing Keys in Arrays:** The instructor briefly notes that React will yell at you in the console about "keys" when rendering arrays. While he says to "ignore it for now," it's a critical React fundamental to know: when mapping over arrays or rendering lists of elements, React requires a unique `key` prop on each item to optimize the Virtual DOM diffing algorithm.
    
- **Best Practice - High Cohesion (Separation of Concerns):** The transcript highlights the core React best practice of grouping by _concern_ rather than by _technology_. Instead of keeping all HTML in one file and all JS in another, React encourages keeping the JS and UI logic for a specific feature (like a Pizza card) entirely encapsulated within its own component.
    

### 5. Key Takeaways (TL;DR)

- **Props make components dynamic:** They allow parent components to pass custom data to child components.
    
- **Arrays allow sibling returns:** You can return an array of `React.createElement` calls to avoid adding unnecessary HTML wrapper `div`s to your DOM.
    
- **Think in components:** React's power comes from breaking the UI down into small, isolated, and reusable pieces rather than treating the frontend as one massive, separated Model-View-Controller architecture.
    
- **Mental execution is key:** The instructor reiterates the importance of predicting the exact DOM output (e.g., counting the exact number of expected `h1` and `p` tags) before checking the browser.
  
  ____

### 1. Core Objective

The primary goal of this lesson is to introduce **JSX** as a developer-friendly syntactic sugar over `React.createElement` and to demonstrate how to transition from a single-file script into a modular, multi-file application structure. The viewer learns how to write declarative markup inside JavaScript, dynamically inject data using expressions, and properly export/import components using ES Modules.

### 2. Key Concepts Explained

- **JSX (JavaScript XML):** The instructor explains that JSX is not a new language, but rather a very thin abstraction layer on top of JavaScript. Initially met with skepticism for mixing "HTML into JavaScript," it was widely adopted because it allows developers to write code that visually mimics the intended DOM structure. Under the hood, the build tool transpiles every JSX tag back into the raw `React.createElement` calls seen in previous lessons.
    
- **JSX Expressions (`{}`):** To make JSX dynamic, developers can execute standard JavaScript inside their markup using curly braces. The instructor clarifies a crucial technical boundary: you can only place JavaScript _expressions_ inside these braces (anything that resolves to a value, like `props.name.toUpperCase()`), not _statements_ (like `if/else` or `for` loops).
    
- **DOM API Naming Conventions (`className`):** In standard HTML, CSS classes are applied using the `class` attribute. In JSX, you must use `className`. This is because JSX is fundamentally JavaScript, and `class` is a strictly reserved keyword in JS (used for ES6 classes). `className` directly maps to the standard browser DOM API (`document.getElementById().className`).
    
- **ES Modules (Default vs. Named Exports):** The video breaks down how to share code between files:
    
    - **Default Exports (`export default Component`):** Imported without curly braces (`import Pizza from './Pizza'`). This aligns well with the "one file, one component" architectural pattern.
        
    - **Named Exports (`export const Component`):** Imported with curly braces (`import { Pizza } from './Pizza'`). This is useful when a single file exports multiple distinct utility functions or components.
        
- **File Extensions (`.js` vs `.jsx`):** The instructor provides historical context on file naming. While `.js` became popular for a time, modern, high-performance bundlers like Vite strictly require the `.jsx` extension to explicitly know when to run the JSX transpiler.
    

### 3. Code Logic & Architecture

Here is the architectural transition from the raw React API to modular JSX.

**The Child Component (`Pizza.jsx`)**



```JavaScript
// We no longer need: import React from 'react';

const Pizza = (props) => {
  return (
    <div className="pizza">
      <h1>{props.name.toUpperCase()}</h1>
      <p>{props.description}</p>
    </div>
  );
};

export default Pizza;
```

- **How it works:** 1. The component returns JSX instead of an array of `React.createElement` calls. The parentheses `()` following the `return` statement are used to allow the JSX to span multiple lines safely without triggering JavaScript's automatic semicolon insertion (ASI).
    
    2. The `{props.name.toUpperCase()}` syntax demonstrates a JavaScript expression evaluating in real-time before the `h1` is painted to the DOM.
    
    3. The file concludes with a `default` export, packaging the `Pizza` function so it can be consumed by other parts of the application.
    

**The Parent Component (`App.jsx`)**



```JavaScript
import Pizza from './Pizza'; // Importing the default export

// ... inside the App component return:
<Pizza name="The Pepperoni Pizza" description="Some dope pizza" />
```

- **How it works:** The `App` file imports the `Pizza` component. Because it was exported as a `default`, we can name it whatever we want upon import, though keeping the name consistent (`Pizza`) is standard practice. When used in JSX, attributes like `name` and `description` are automatically bundled into the `props` object and passed to the child component.
    

### 4. Best Practices & Pitfalls

- **Best Practice - The "One File, One Component" Rule:** The instructor highly recommends keeping components isolated in their own individual files and utilizing `export default`. This makes the codebase highly navigable and prevents files from becoming massive and unmaintainable.
    
- **Best Practice - Tooling Pragmatism:** Developers often argue over `.js` versus `.jsx` or default versus named exports. The instructor's core advice is to avoid these "religious wars" and simply follow what your specific build tools (like Vite) and team conventions require.
    
- **Historical Pitfall Avoided - The React Import:** In older codebases, omitting `import React from 'react'` in a file containing JSX would cause a crash, because the transpiled code referenced the `React` object. Modern React (versions 17+) introduced an automatic JSX runtime transform, completely removing the need to manually import React just to write UI markup.
    

### 5. Key Takeaways (TL;DR)

- JSX is syntactic sugar that allows you to write HTML-like structures inside JavaScript; it is strictly compiled down to `React.createElement` logic.
    
- Use curly braces `{}` to evaluate any valid JavaScript expression directly within your JSX markup.
    
- Always use `className` instead of `class` to assign CSS classes to elements, avoiding collisions with JS keywords.
    
- Default exports are ideal for isolating single components per file.
    
- Modern bundlers like Vite require the `.jsx` file extension to process React markup, but you no longer need to manually `import React` to use it.
  
  
  ___

### 1. Core Objective

The primary goal of this lesson is to configure ESLint to intelligently lint React code and to finalize the application's migration from pure JavaScript (`React.createElement`) to modern JSX. The viewer learns the strict syntactical rules of JSX, how to configure ESLint's modern "flat" configuration for React, and how to properly render custom components using JSX syntax.

### 2. Key Concepts Explained

- **JSX Syntax Strictness & "Gotchas":** * **Reserved Keywords:** Because JSX compiles to JavaScript, you cannot use reserved JS keywords as HTML attributes. Therefore, the standard HTML `class` becomes `className` (to avoid colliding with JS ES6 classes), and the label attribute `for` becomes `htmlFor` (to avoid colliding with `for` loops).
    
    - **Self-Closing Tags:** Browsers are forgiving with unclosed HTML tags like `<input>` or `<img>`. JSX is strictly parsed XML; it will throw a fatal error if tags are not explicitly closed. You must use self-closing syntax (e.g., `<input />`).
        
- **The JSX Runtime transform:** Historically, React required developers to include `import React from 'react'` at the top of every file containing JSX, so the compiler could access `React.createElement`. Modern React toolchains feature an automatic "JSX runtime" that injects this behind the scenes, rendering the manual import obsolete.
    
- **Component Naming Conventions (Capitalization):** React uses capitalization to distinguish between native DOM elements and custom React components.
    
    - **Lowercase (e.g., `<h1>`, `<div>`):** Tells React to render a standard HTML DOM node.
        
    - **Uppercase (e.g., `<Pizza />`, `<App />`):** Tells React that this is a custom, user-defined component that needs to be executed to resolve its underlying UI.
        
- **ESLint Flat Configuration:** The instructor configures ESLint using its modern "flat" configuration syntax (`eslint.config.js`). This requires specific ordering and merging of configuration objects using the spread operator (`...`).
    

### 3. Code Logic & Architecture

Here are the two major architectural shifts demonstrated in the code.

**1. ESLint Configuration (`eslint.config.js`)**



```JavaScript
import reactPlugin from 'eslint-plugin-react';

export default [
  // ... other configs
  {
    ...reactPlugin.configs.flat.recommended,
    settings: {
      react: {
        version: 'detect' // Automatically reads package.json to find the React version
      }
    }
  },
  reactPlugin.configs.flat['jsx-runtime'], // Disables the 'React must be in scope' rule
  {
    rules: {
      'react/no-unescaped-entities': 'off', // Allows normal quotes (') instead of forcing &apos;
      'react/prop-types': 'off' // Disables prop-type validation (largely replaced by TypeScript today)
    }
  }
];
```

- **How it works:** The configuration injects React's recommended ruleset. It dynamically detects the installed React version so it lints accurately. It then applies the `jsx-runtime` ruleset to stop ESLint from warning about missing React imports. Finally, it explicitly disables two overly strict default rules to improve developer experience.
    

**2. The JSX App Component (`App.jsx`)**



```JavaScript
import Pizza from './Pizza';

const App = () => {
  return (
    <div>
      <h1>Padre Gino's - Order Now</h1>
      <Pizza name="Pepperoni" description="Pep, cheese, stuff" />
      <Pizza name="Hawaiian" description="Ham, pineapple" />
      <Pizza name="Americano" description="French fries" />
    </div>
  );
};

// Rendering the root component
root.render(<App />);
```

- **How it works:** `App.js` is renamed to `App.jsx`. The verbose array of `React.createElement` calls is replaced by a clean, HTML-like structure. The `Pizza` function is invoked by treating it as an HTML tag (`<Pizza />`), and its arguments (props) are passed identically to standard HTML attributes (`name="Pepperoni"`).
    

### 4. Best Practices & Pitfalls

- **Best Practice - Version Detection:** Always configure the ESLint React plugin to `detect` the React version. Hardcoding the version in the linter settings creates technical debt when you eventually upgrade React.
    
- **Best Practice - Disabling Friction:** The instructor advocates turning off `react/no-unescaped-entities`. While technically safer for avoiding rogue characters breaking the parser, forcing developers to write `&apos;` instead of a simple apostrophe drastically slows down development. Modern tooling usually handles text encoding safely.
    
- **Pitfall - Renaming File Extensions:** A common bug when transitioning to JSX is renaming `App.js` to `App.jsx` but forgetting to update the corresponding `<script src="./src/App.js">` tag in the `index.html` file. The entry point must match exactly.
    
- **Pitfall - Lowercase Custom Components:** If you name your component `pizza` and write `<pizza />`, React will attempt to find a native HTML tag called `<pizza>`, which doesn't exist, leading to a silent failure or broken UI. Always capitalize custom components.
    

### 5. Key Takeaways (TL;DR)

- JSX is strict: you must use `className`, `htmlFor`, and explicitly self-close empty tags like `<input />`.
    
- React relies on capitalization to know if a tag is a standard HTML element (`<div>`) or a custom React component (`<Pizza />`).
    
- The modern JSX runtime eliminates the need to `import React` at the top of your files.
    
- ESLint configurations should be updated to use the `jsx-runtime` preset and actively `detect` the installed React version to prevent false-positive errors.
  
  
  ---

### 1. Core Objective

The primary goal of this lesson is to establish a bridge between the frontend React application and a backend API during local development. The viewer learns how to run a provided local Node.js API and, crucially, how to configure the Vite development server to act as a reverse proxy. This setup seamlessly routes frontend requests to the backend, circumventing complex security policies like CORS.

### 2. Key Concepts Explained

- **Client-Server Separation:** The architecture of the project is split. The frontend React app runs on one development server (Vite via port 5173), while the backend API (Fastify/Node.js) runs on an entirely separate server (port 3000).
    
- **CORS (Cross-Origin Resource Sharing):** Browsers have strict security features that prevent a frontend running on one origin (e.g., `localhost:5173`) from making direct requests to a backend on a different origin (e.g., `localhost:3000`). The instructor highlights proxying as the modern, painless way to bypass this development headache without having to write complex CORS configurations on the backend.
    
- **Reverse Proxying via Vite:** Instead of the React app making a request directly to the API port, it makes the request to its _own_ port. The Vite server intercepts any request matching specific URL paths (like `/api`) and secretly forwards it to the backend server. To the browser, it looks like the frontend and backend are running on the exact same domain and port.
    

### 3. Code Logic & Architecture

Here is the underlying logic of the proxy configuration added to the Vite setup.

**Vite Configuration (`vite.config.js`)**



```JavaScript
export default defineConfig({
  // ... plugins and other configurations
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true,
      },
      '/public': {
        target: 'http://localhost:3000',
        changeOrigin: true,
      }
    }
  }
});
```

- **How it works:**
    
    - **`server.proxy`:** This object tells Vite's internal development server to listen for specific incoming request paths.
        
    - **The Keys (`'/api'`, `'/public'`):** These act as the triggers. If the React app fetches data from `http://localhost:5173/api/pizzas`, Vite intercepts it because the path starts with `/api`.
        
    - **`target`:** This dictates where Vite should send the intercepted request. In this case, it forwards the request to the local Fastify server running on port 3000.
        
    - **`changeOrigin: true`:** This modifies the host header of the request to match the target URL. This is a crucial setting that ensures the backend server doesn't reject the proxy request due to host mismatching.
        

### 4. Best Practices & Pitfalls

- **Best Practice - Mirroring Production:** Using a proxy in local development is highly recommended because it mirrors how applications often run in production (where the built frontend and the API sit behind a single load balancer or domain like `padreginos.com` and `padreginos.com/api`).
    
- **Best Practice - Cloud Target Flexibility:** The instructor correctly points out that the `target` does not have to be `localhost`. If your team has a shared staging database or a cloud API (like AWS Lambdas) available at `staging.myapi.com`, you can simply point your local Vite proxy target there to develop against live data.
    
- **Pitfall - Port Conflicts:** A common developer error is leaving older backend processes or different project servers running in the background. If another app is occupying port 3000, the Fastify server will fail to start, breaking the entire proxy chain. Always ensure your terminal ports are clear before booting up the stack.
    

### 5. Key Takeaways (TL;DR)

- Running full-stack apps locally often requires managing multiple servers on different ports (e.g., Vite on 5173, Node on 3000).
    
- Fetching data across different local ports triggers CORS errors in the browser.
    
- Vite has built-in server proxying that intercepts frontend requests and forwards them to the backend, entirely eliminating CORS issues during development.
    
- Once the proxy is configured in `vite.config.js`, your React app should fetch from its _own_ port (`localhost:5173/api/...`), and Vite will handle routing it to the backend behind the scenes.
  
  
  
  ___

### 1. Core Objective

The primary goal of this lesson is to demonstrate how to dynamically render media within reusable React components by passing file paths through props. It also reinforces the dual-server local development workflow by proving that the Vite proxy configuration successfully serves static assets (like images and CSS) from the backend API to the frontend client.

### 2. Key Concepts Explained

- **Dynamic Media via Props:** Props are not limited to just rendering text. By passing a file path string down as a prop, developers can instruct a single, generic component to render completely different images.
    
- **Dynamic Accessibility (Alt Text):** The instructor uses the existing `props.name` to dynamically populate the `alt` attribute of the image. This is a fundamental concept in writing accessible React code; as components become dynamic, their accessibility markers must scale dynamically as well.
    
- **Static Asset Proxying:** The images and CSS files are technically hosted on the backend Fastify API under a `/public` directory. Because of the proxy configured in the previous lesson, the React frontend can request `/public/pizzas/pepperoni.webp` on its _own_ port (5173), and Vite seamlessly fetches it from the backend port (3000).
    
- **Concurrent Server Execution:** Modern full-stack development usually requires running multiple processes simultaneously. The instructor explicitly clarifies the need to split terminal windows to run both the Node API and the Vite development server at the same time.
    

### 3. Code Logic & Architecture

Here is the architectural update to the application's UI and styling layers.

**The Child Component (`Pizza.jsx`)**



```JavaScript
const Pizza = (props) => {
  return (
    <div className="pizza">
      <h1>{props.name}</h1>
      <p>{props.description}</p>
      <img src={props.image} alt={props.name} />
    </div>
  );
};
```

- **How it works:** An `<img>` tag is added to the component blueprint. Instead of a hardcoded string, the `src` attribute evaluates the JavaScript expression `{props.image}`. The `alt` attribute smartly reuses `{props.name}`. Note that the `<img>` tag must be explicitly self-closed (`/>`), as JSX enforces strict XML syntax.
    

**The Parent Component (`App.jsx`)**



```JavaScript
// Inside the App component return:
<Pizza 
  name="Pepperoni" 
  description="Pep, cheese, stuff" 
  image="/public/pizzas/pepperoni.webp" 
/>
<Pizza 
  name="Hawaiian" 
  description="Ham, pineapple" 
  image="/public/pizzas/hawaiian.webp" 
/>
```

- **How it works:** The parent component dictates exactly which image each `Pizza` instance should display by passing the relative URL string into the `image` prop.
    

**Global Styling (`index.html`)**



```HTML
<head>
  <link rel="stylesheet" href="/public/style.css" />
</head>
```

- **How it works:** Rather than importing CSS directly into the JavaScript (which is common in modern bundlers but hasn't been introduced yet), the application taps directly into the proxied API server to fetch a global stylesheet via a standard HTML `<link>` tag.
    

### 4. Best Practices & Pitfalls

- **Best Practice - Dynamic Alt Text:** Automatically mapping `props.name` to the image's `alt` attribute ensures that every dynamically generated image is compliant with screen readers, without requiring an entirely separate `props.altText` to be passed down.
    
- **Best Practice - Modern Image Formats:** The project uses `.webp` images. WebP is a modern image format developed by Google that provides superior lossless and lossy compression for images on the web, significantly improving page load times compared to standard JPEGs or PNGs.
    
- **Pitfall - Unclosed Tags:** Just like the `<input />` tag in the previous lesson, forgetting the trailing slash on an `<img>` tag will cause the JSX transpiler to throw a fatal error.
    
- **Pitfall - Dead Servers:** A common point of confusion for beginners is seeing broken image links or unstyled HTML. This almost always happens because the terminal running the backend API was accidentally closed, meaning the proxy has nothing to fetch the `/public` assets from.
    

### 5. Key Takeaways (TL;DR)

- Props are the standard mechanism for passing varied asset paths (like images) into reusable UI templates.
    
- You can (and should) reuse existing text props to dynamically generate descriptive `alt` tags for accessibility.
    
- JSX strictly requires standard HTML tags like `<img>` to be explicitly self-closing.
    
- Vite's proxy feature routes static assets (CSS, WebP images) exactly the same way it routes API data requests.
    
- Full-stack local development requires concurrent terminal sessions—one keeping the backend alive, and one serving the frontend.