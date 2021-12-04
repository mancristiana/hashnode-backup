## Building Compound Components using React Context API and TypeScript

When creating a React project, you might need to share some state between components of your application. [React Context API](https://reactjs.org/docs/context.html) offers a mechanism for passing data through the component tree without relying on props (also known as props drilling). 

I have seen teams implement a lightweight global store for their React application using React Context as an alternative to using a state management library such as Redux.

This article shows how you can build reusable `Accordion` compound components that rely on React Context to share state and control the visibility of collapsible content. 

> You can find the completed project on [GitHub](https://github.com/mancristiana/blog-app-context).

You can see the `Accordion` in action:
![Accordion.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1616782955565/P413FpEUr.gif)

The building blocks of the `Accordion` are `CollapseButton` and `ExpandButton,` which toggle the visibility of the `Collapsible` content when clicked.

In the main App file we use the `Accordion` as follows:
```
<Accordion>
    <h1>Accordion</h1>
    <Accordion.CollapseButton>Show less</Accordion.CollapseButton>
    <p>
        In pharetra nibh massa, ...
        <Accordion.Collapsible>
            Vestibulum feugiat ...
        </Accordion.Collapsible>
    </p>
    <Accordion.ExpandButton>Show more...</Accordion.ExpandButton>
    <Accordion.CollapseButton>Show less</Accordion.CollapseButton>
</Accordion>;
```

Since the `Accordion` exposes compound components, we get the flexibility to place the `ExpandButton,` `CollapseButton,` and `Collapsible` content in any order or configuration we need. From my experience, creating compound components that enable this kind of composability has the benefits of reducing the number of props, complexity, and potential bugs introduced. Thus, it is generally a good practice.

Let's take a look at the implementation of `Accordion`:

```
const Accordion: FunctionComponent & AccordionComposition = ({ children }) => {
    const [isExpanded, setIsExpanded] = useState(false);
    const expand = () => setIsExpanded(true);
    const collapse = () => setIsExpanded(false);

    return (
        <AccordionProvider value={{ isExpanded, expand, collapse }}>
            {children}
        </AccordionProvider>
    );
};
```

> If you are wondering about the typings in the first line `const Accordion: FunctionComponent & AccordionComposition`, you can check out my previous article [React Compound Components Typings](https://blog.cristiana.tech/react-compound-component-typings). It explains how you can define and export Compound Components using TypeScript.

The `Accordion` component is responsible for storing the `isExpanded` state, and defining means through which the state can be updated (via `expand,` `collapse` functions). As you will see further in this article, the `Collapsible`, `ExpandButton` and `CollapseButton` need to know and modify this state. 

This is where React Context comes into play and shines!
The `AccordionProvider` is an important part of the React Context mechanism because it gives children in the tree access to a value or the shared state. For example, the value is an object with three properties: `isExpanded`, `expand` and `collapse`. The type of this object is defined as follows:
```
interface AccordionContextValue {
    isExpanded: boolean;
    expand: () => void;
    collapse: () => void;
}
```

To understand how `AccordionProvider` is defined we must first understand how the `AccordionContext` is created. We rely on `createContext` function imported from `React`.
```
import { createContext } from "react";
```

The `createContext` function accepts a default value as an argument. In this case, I chose the `undefined` value. We then specify the type of the value passed through the context which can either be `AccordionContextValue` or the `undefined` value.
 
```
const AccordionContext = createContext<AccordionContextValue | undefined>(undefined);
```

The `AccordionContext` exposes a `Provider` which we can export as `AccordionProvider`

```
export const AccordionProvider = AccordionContext.Provider;
```

According to [React Documentation](https://reactjs.org/docs/context.html#contextprovider):

> Every Context object comes with a Provider React component that allows consuming components to subscribe to context changes.

> The Provider component accepts a value prop to be passed to consuming components that are descendants of this Provider. One Provider can be connected to many consumers. Providers can be nested to override values deeper within the tree.

> All consumers that are descendants of a Provider will re-render whenever the Provider’s value prop changes. 


To consume the value provided through `AccordionProvider` we can use `useContext` hook:

```
import { useContext } from "react";
const context = useContext(AccordionContext);
```

It is a good idea to check whether we have an undefined context since it is an indication that `Collapsible`, `ExpandButton` or `CollapseButton` were used outside of an `Accordion` parent. Such a usage is unintended. The `useAccordionContext ` custom hook abstracts away this logic and allows reusing it in all places we need:
```
export const useAccordionContext = () => {
    const context = useContext(AccordionContext);
    if (!context) {
        throw new Error(
            "Accordion.Collapsible, Accordion.ExpandButton and Accordion.CollapseButton must be used within an Accordion"
        );
    }
    return context;
};
```

We can then destructure the `isExpanded` value from `useAccordionContext` and conditionally render `children` of the `Collapsible` component:
 
```
const Collapsible: FunctionComponent = ({ children }) => {
    const { isExpanded } = useAccordionContext();
    if (isExpanded) {
        return <>{children}</>;
    } else {
        return null;
    }
};
```

In the `ExpandButton` component, we are also using the `expand` function when someone clicks the `button` to toggle the visibility of the collapsible content. Besides, we control when `ExpandButton` is rendered depending on whether `isExpanded` is false.
```
const ExpandButton: FunctionComponent = ({ children }) => {
    const { isExpanded, expand } = useAccordionContext();
    if (!isExpanded) {
        return (
            <button className={styles.button} onClick={expand}>
                {children}
            </button>
        );
    } else {
        return null;
    }
};
```
Similarly, in the `CollapseButton` we are using the `collapse` function and also controlling when `CollapseButton` is rendered depending on `isExpanded`.

```
const CollapseButton: FunctionComponent = ({ children }) => {
    const { isExpanded, collapse } = useAccordionContext();
    if (isExpanded) {
        return (
            <button className={styles.button} onClick={collapse}>
                {children}
            </button>
        );
    } else {
        return null;
    }
};
```

To conclude, React Context API is a powerful tool for passing state through the tree of children. We looked at an example of how to build `Accordion` compound components that share and modify some state and how we can abstract that mechanism away for consumers of those components.

Thanks for reading!
