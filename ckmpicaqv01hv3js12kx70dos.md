## React Children Map and cloneElement using TypeScript

This article shows how you can use the [React Children map](https://reactjs.org/docs/react-api.html#reactchildrenmap) function together with [React cloneElement](https://reactjs.org/docs/react-api.html#cloneelement) function to add additional props to direct children of a component. Besides, the article demonstrates how you can check the type of a child component and modify it based on their type.

This approach is useful when building a compound component in React. For example, imagine you were building the following tabs.

![TabsDemo.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1616710223179/qysuGyUvv.gif)

This component would be implemented using compound components like this:

```
<Tabs>
    <Tabs.Title>Admin</Tabs.Title>
    <Tabs.Item id="overview">Overview</Tabs.Item>
    <Tabs.Item id="settings">Settings</Tabs.Item>
</Tabs>
```

> If you are wondering how you can define and export Compound Components using TypeScript, check out my previous article [React Compound Components Typings](https://blog.cristiana.tech/react-compound-component-typings).

In this example, clicking on `Tabs.Item` makes the tab active and shows a corresponding active style. Here is how `TabsItem` was defined:
```jsx
// TabsItem.tsx
import styles from "./Tabs.module.scss";
import { FunctionComponent } from "react";
import { combineClassNames } from "../../utils/combineClassNames";

export interface TabsItemProps {
    id: string;
    isActive?: boolean;
    onClick?: () => void;
}

export const TabsItem: FunctionComponent<TabsItemProps> = ({
    children,
    isActive = false,
    onClick,
}) => (
    <div
        className={combineClassNames(
            styles.item,
            isActive ? styles.active : null
        )}
        onClick={onClick}
    >
        {children}
    </div>
);
```

The `Tabs` component is responsible for storing the active tab and ensuring that when a tab is clicked, the active tab state gets updated.

Here is the complete implementation of `Tabs`:
```jsx
// Tabs.tsx
import styles from "./Tabs.module.scss";
import {
    FunctionComponent,
    useState,
    Children,
    PropsWithChildren,
    ReactElement,
    cloneElement,
} from "react";
import { TabsTitle } from "./TabsTitle";
import { TabsItem, TabsItemProps } from "./TabsItem";

interface TabsComposition {
    Title: FunctionComponent;
    Item: FunctionComponent<TabsItemProps>;
}

const Tabs: FunctionComponent & TabsComposition = ({ children, ...props }) => {
    const [activeTab, setActiveTab] = useState<string>();
    return (
        <div className={styles.tabs}>
            {Children.map(children, (child) => {
                const item = child as ReactElement<PropsWithChildren<TabsItemProps>>;

                if (item.type === TabsItem) {
                    const isActive = item.props.id === activeTab;
                    const onClick = () => {
                        setActiveTab(item.props.id);
                        item.props.onClick?.();
                    };
                    return cloneElement(item, { isActive, onClick });
                } else {
                    return child;
                }
            })}
        </div>
    );
};

Tabs.Title = TabsTitle;
Tabs.Item = TabsItem;

export { Tabs };

``` 

Let's break it down.

In the Tabs component, we can store which tab is the active using `useState` hook.
```jsx
const [activeTab, setActiveTab] = useState<string>();
```
`Tabs` component maps over the children and modifies them. 
The simplest way of using [React's Children map API](https://reactjs.org/docs/react-api.html#reactchildrenmap) is equivalent to `{children}` and illustrated below:

```jsx
{Children.map(children, (child) => {
    return child;
})}
```
This code is traversing each direct child of the `Tabs` component and returning it as is.
However, simply returning each child is not very useful. Typically, we want to make some modifications to the child elements. For example, we might want to add an `isActive` prop in addition to the props which were already defined on the `Tab.Item` component.

To enrich child components with additional props we can use [React cloneElement](https://reactjs.org/docs/react-api.html#reactchildrenmap). This function accepts two arguments: the first is the element to be cloned, and the second argument is an object with additional props that will be added to the newly cloned element.

```jsx
{Children.map(children, (child) => {
    // ...
    return cloneElement(child, { isActive, onClick });
})}
```

To correctly set the `isActive` prop and `onClick` event so that they are based on the `activeTab` state, we need to get the `id` props of each `Tab.Item`.  Since we are using TypeScript we must explicitly set the typing `ReactElement<PropsWithChildren<TabsItemProps>>` which allows us to use the `TabsItemProps` interface when working with the child props. Thus, we get all the development benefits of type checking, IntelliSense, etc.

```jsx
{Children.map(children, (child) => {
    const item = child as ReactElement<PropsWithChildren<TabsItemProps>>;
    const isActive = item.props.id === activeTab;
    const onClick = () => {
        setActiveTab(item.props.id);
        item.props.onClick?.();
    };
    return cloneElement(child, { isActive, onClick });
})}
```

While the code above works well when we only have `Tab.Item` children, it adds props to any component regardless of their type. For example, it would add the `isActive` prop to the `Tab.Title` child element, which is not intended behavior.

The desired behavior is adding props only on the `Tab.Item` child elements. Fortunately, we can check the type of the child element components. If this type matches `TabsItem` we are able to clone it and add props, otherwise we return the child as it is.

```
if (item.type === TabsItem) {
    //...
    return cloneElement(child, { isActive, onClick });
} else {
    return child;
}
```

To sum up, when building compound components such as the `Tabs` you can leverage React's `Children.map` function together with `cloneElement` to enrich component props. Finally, you can check the type of each child and conditionally modify child elements.
