## React Compound Component Typings

Imagine you were building a compound component in React. 

```
<Tabs>
    <Tabs.Title>Admin</Tabs.Title> 
    <Tabs.Item>Overview</Tabs.Item>
    <Tabs.Item>Settings</Tabs.Item>
</Tabs>
```

The type of `Tabs` is `FunctionComponent<TabsProps>` so how can we export the `Title` and `Item` components?

We can define a `TabsComposition` interface:
```
interface TabsComposition {
    Title: FunctionComponent<TitleProps>;
    Item: FunctionComponent<ItemProps>;
}
```

Then the declaration of `Tabs` becomes:

```
const Tabs: FunctionComponent<TabsProps> & TabsComposition = ({children, ...props}) => (
  <>{children}</>
);
```

We can then assign the Title and Item components like you would normally [work with objects in JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects):

```
Tabs.Title = Title;
Tabs.Item = Item;
```

Finally, you can export the `Tabs` component: 
```
export {Tabs};
```
