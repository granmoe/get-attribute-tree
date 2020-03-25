# Data Tree Snapshot 🌳🤳

## Intro

Create the smallest possible representation of the tree structure of the values of any attribute or property contained in a DOM element. `data-tree-snapshot` only preserves ancestor/descendant relationships because it is used to assert that a given element _contains_ a certain structure. This compacted tree structure is created recursively for the entire DOM tree within the given DOM element.

## Installation

```
npm install data-tree-snapshot -D
```

## API

### `printTextTree` (Text Snapshotting, named export)

```js
import React from 'react'
import { render } from '@testing-library/react'
import { printTextTree } from 'data-tree-snapshot'

test('has correct textContent', () => {
  // This is just an example. Normally, the JSX output would be a result of rendering
  // your app or a subset of your app and taking actions to get it into a given state
  const { container } = render(
    <div>
      Heading
      <div>
        Subheading
        <div>Paragraph 1</div>
        <div>Paragraph 2</div>
        <div>Paragraph 3</div>
      </div>
    </div>,
  )

  const tree = printTextTree(container)

  expect(tree).toMatchInlineSnapshot(`
    "
    Heading
      Subheading
        Paragraph 1
        Paragraph 2
        Paragraph 3
    "
  `)
})
```

#### Implementation

```js
import makePrintDataTree from '../make-print-data-tree'
// 👆The default export of data-tree-snapshot, used to create data tree printers

export default makePrintDataTree({
  propertyName: 'textContent',
  filter: t => typeof t === 'string' && t.length > 0,
})
```

### `printTestIdTree` (Test Id Tree Snapshotting, named export)

```js
import React from 'react'
import { render } from '@testing-library/react'
import { printTestIdTree } from 'data-tree-snapshot'

test('shows correct components for scenario', () => {
  render(
    // This is just an example. Normally, the JSX output would be a result of rendering
    // your app or a subset of your app and taking actions to get it into a given state
    <div data-testid="root">
      <div>
        <div data-testid="parent">
          <div data-testid="child">
            <div>
              <div>
                <span data-testid="deeply-nested-child">some stuff</span>
              </div>
            </div>
          </div>
          <br data-testid="another-child" />
          <div>
            <br data-testid="yet-another-child" />
          </div>
        </div>
      </div>
    </div>,
  )

  // 👇 You can pass either a data-testid string or an element itself
  expect(printTestIdTree('root')).toMatchInlineSnapshot(`
    "
    parent
      child
        deeply-nested-child
      another-child
      yet-another-child
    "
  `)
})
```

(Lots more good info about this helper [on the wiki page!](https://github.com/granmoe/data-tree-snapshot/wiki/printTestIdTree))

#### Implementation

```js
import makePrintDataTree from '../make-print-data-tree'
// 👆The default export of data-tree-snapshot, used to create data tree printers

// If you use an attribute, the default string selector will be for that attribute
// e.g. document.querySelector('[data-testid="your-test-id-name"]') will be used for printTestIdTree
export default makePrintDataTree({
  attributeName: 'data-testid',
  filter: t => typeof t === 'string' && t.length > 0,
})
```

### `printQaIdTree`

Same as `printTestIdTree`, except it looks at `data-qa` instead of `data-test-id`.

### `makePrintDataTree`

This is a function that creates a custom printDataTree function based on the following options object, represented here as a TypeScript interface:

```ts
interface options {
  // formats the value before printing it
  // value is the attribute or property we've gotten from the element
  // based on attributeName or propertyName
  format?: (value) => boolean
  // Only values / elements for which this returns true will be included
  filter?: (value, element) => boolean
  attributeName?: string
  propertyName?: string
}
```

The only requirement is that either `attributeName` or `propertyName` are passed (but not both).

The type of the returned print function is as follows `elementOrString => string`, where `elementOrString` is either an HTML element or a string value to be used as a selector for the given `attributeName` (passing a string selector for property tree printers is currently unsupported).

For example, when calling `printTestIdTree('root')`, `document.querySelector('[data-testid="root"]')` will be called to get the root element for which to print the test id tree.

Full example:

```js
const getLabelTree = makePrintDataTree({
  attributeName: 'label',
  filter: l => typeof l === 'string' && l.length > 0,
})

printLabelTree('some-label-value') // document.querySelector(`[label="some-label-value"]`) will be called to get the root element
```

## Upcoming Features

- Include multiple pieces of data per element
- Configure selectors for property printers
