# react-accessible-headings

## The Problem

[WCAG WAI says](https://www.w3.org/WAI/tutorials/page-structure/headings/),

> Skipping heading ranks can be confusing and should be avoided where possible: Make sure that a `<h2>` is not followed directly by an `<h4>`, for example.

However developers often hardcode specific heading levels into their components, limiting their flexibility.

By using `react-accessible-headings` you can have components with dynamic headings that fit the appropriate heading level, allowing you to more easily create accessible headings that don't skip levels.

Could you avoid this library and perhaps make component props that set the heading level, or use `children` in each instance so that the heading level is correct? Sure, but this is an alternative approach that makes it easier to refactor and 'indent' heading levels arbitrarily. See the <a href="#Examples">Examples</a> section for ideas on how this can be done.

This library is less than 1 kilobyte (minified and compressed) and comes with TypeScript types.

## Usage

```jsx
import React from "react";
import { Level, H } from "react-accessible-headings";

export default function() {
  return (
    <div>
      <H>This will be a heading 1</H>
      <Level>
        <H>and this a Heading 2</H>
        <H>another Heading 2</H>
        <Level>
          <H>a Heading 3</H>
        </Level>
        <H>yet another Heading 2</H>
      </Level>
    </div>
  );
}
```

## API

All APIs have TypeScript types available.

### `Level` component

Props: `value`: _(Optional)_ a **number** to override the level. An exception will be thrown if attempting to set an invalid value such as `7` as HTML only has h1-h6. There are no other props, except `children`.

This component doesn't render any HTML except `children`.

### `H` component

Props: `offset`: _(Optional)_ a **number** to offset the heading level (see _Examples: The 'Offset' Example_ for more). All other valid props for an heading are also accepted.

This component renders either `<h1>`, `<h2>`, `<h3>`, `<h4>`, `<h5>`, or `<h6>`. An exception will be thrown if attempting to render invalid HTML such as `<h7>`.

### `LevelContext` context

If for some reason you'd like to inspect the current `level` value then `useContext(LevelContext)`.

## Limitations

While this library facilitates dynamic heading levels it doesn't detect skipped heading levels through incorrect usage such as,

```jsx
<h1>Heading 1</h1>
<Level>
  <Level>
    <Level>
      <H>this will be a heading 3</H>
    </Level>
  </Level>
</Level>
```

Testing in [Axe](https://www.deque.com/axe/) will reveal this error. It's unlikely that this project will introduce a runtime check for analysing heading levels as Axe already does this. Also, because webpages could have a static HTML `h1` with a React app rendering only `h2`s (a perfectly valid and accessible approach) then any check would need to analyse the whole DOM and have nothing to do with React or this project, so a run-time check was added this would be a separate standalone package, but replicating Axe functionality would probably be pointless.

## Further reading

### Prior art

[DocBook](https://docbook.org/), the ill-fated [XHTML 2](https://www.w3.org/TR/xhtml2/mod-structural.html#sec_8.5.), and [HTML5's abandoned 'outline'](http://blog.paciellogroup.com/2013/10/html5-document-outline/) had a very similar idea. Also check out the 2014 project [html5-h](https://github.com/ThePacielloGroup/html5-h).

### References

#### [WCAG 2: G141: Organizing a page using headings](https://www.w3.org/TR/2012/NOTE-WCAG20-TECHS-20120103/G141),

> To facilitate navigation and understanding of overall document structure, authors should use headings that are properly nested (e.g., h1 followed by h2, h2 followed by h2 or h3, h3 followed by h3 or h4, etc.).

#### [Axe: Heading levels should only increase by one](https://dequeuniversity.com/rules/axe/3.4/heading-order)

> Ensure headings are in a logical order. For example, check that all headings are marked with `h1` through `h6` elements and that these are ordered hierarchically. For example, the heading level following an `h1` element should be an `h2` element, not an `h3` element.

## Examples

### The 'Card' Example

Consider a 'card' component that might be coded as,

```jsx
export function Card({ children, heading }) {
  return (
    <div className="card">
      <h3 className="card__heading">{heading}</h3>
      {children}
    </div>
  );
}
```

But then you want to reuse that card in two places with two different heading levels, so you might refactor the code like,

```jsx
export function Card({ children, heading, headingLevel }) {
  return (
    <div className="card">
      {headingLevel === 2 ? (
        <h2 className="card__heading">{heading}</h2>
      ) : headingLevel === 3 ? (
        <h3 className="card__heading">{heading}</h3>
      ) : null}
      {children}
    </div>
  );
}
```

or

```jsx
export function Card({ children, heading, headingLevel }) {
  return (
    <div className="card">
      {React.createElement(
        "h" + headingLevel,
        { className: "card__heading" },
        heading
      )}
      {children}
    </div>
  );
}
```

or perhaps you'd use `children`,

```jsx
export function Card({ children }) {
  return <div className="card">{children}</div>;
}
```

but now the parent component needs to know about the `"card__heading"` class and the implementation details of `<Card>` are leaking; there's less encapsulation.

```jsx
// usage
<h1>Cards</h1>
<Card>
  <h2 className="card__heading">text</h2>
  <p>body</p>
</Card>
<h2>See also</h2>
<Card>
  <h3 className="card__heading">text</h3>
  <p>body</p>
</Card>
```

Alternatively, with `react-accessible-headings` the implementation details of `<Card>` can stay encapsulated and look like,

```jsx
export function Card({ children, heading }) {
  return (
    <div className="card">
      <H className="card__heading">{heading}</H>
      {children}
    </div>
  );
}

// usage
<H>Cards</H>
<Level>
  <Card heading="text">
    <p>body</p>
  </Card>
  <H>See also</H>
  <Level>
    <Card heading="text">
      <p>body</p>
    </Card>
  </Level>
</Level>
```

And then consider if there's an <abbr title="information architecture">IA</abbr> change that lowers the heading level of all of these because there's a new `h1` in the page. It's now easy to add a `<Level>` wrapper to indent everything and you're done. Much easier than updating lots of `h*` numbers around the code to realign them all.

```jsx
<Level>
  <H>Cards</H>
  <Level>
    <Card heading="text">
      <p>body</p>
    </Card>
    <H>See also</H>
    <Level>
      <Card heading="text">
        <p>body</p>
      </Card>
    </Level>
  </Level>
</Level>
```

So it's an alternative composition technique that may make it easier to refactor code.

### The 'Level Query' Example

If you want to programatically query the current level you can,

```jsx
import { LevelContext, H } from "react-accessible-headings";

const level = useContext(LevelContext); // level is an integer

return (
  <div className={`heading--${level}`}>
    <H>text</H>
  </div>
);
```

### The 'Offset' Example

If you want to have heading levels dynamic yet related to one another you can provide an `offset` prop

```jsx
<div className="card">
  <H className="card__heading">This will be the current heading level</H>
  <H offset={1} className="card__sub-heading">
    This will be one level deeper
  </H>
  <H offset={2} className="card__sub-sub-heading">
    This will be two levels deeper. I don't know why you'd want this!
  </H>
  {children}
</div>
```
