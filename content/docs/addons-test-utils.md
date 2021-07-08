---
id: test-utils
title: টেস্ট ইউটিলিটি
permalink: docs/test-utils.html
layout: docs
category: Reference
---

**ইম্পোর্ট করা**

```javascript
import ReactTestUtils from 'react-dom/test-utils'; // ES6
var ReactTestUtils = require('react-dom/test-utils'); // ES5 with npm
```

## সারমর্ম {#overview}

`ReactTestUtils` React কম্পোনেন্টগুলোকে আপনার পছন্দের টেস্টিং ফ্রেমওয়ার্কে টেস্ট করার কাজটি সহজ করে দেয়। ফেসবুকে আমরা সহজ জাভস্ক্রিপ্ট টেস্টিং এর জন্য [Jest](https://facebook.github.io/jest/) ব্যবহার করে থাকি। Jest দিয়ে কিভাবে শুরু করবেন তা Jest ওয়েবসাইটের [React টিউটোরিয়াল](https://jestjs.io/docs/tutorial-react) অংশে শিখতে পারবেন।

> বিঃদ্রঃ
>
<<<<<<< HEAD
> আমরা [React Testing Library](https://testing-library.com/react) ব্যবহার করার পরামর্শ দেই যেটা এমনভাবে ডিজাইন করা হয়েছে যাতে আপনি আপনার কম্পোনেন্টগুলোকে এদের ব্যবহারকারীদের মত টেস্ট করতে পারবেন।
>
> অন্যথায়, Airbnb [Enzyme](https://airbnb.io/enzyme/) নামের একটি টেস্টিং ইউটিলিটি প্রকাশ করেছে, যেটা আপনার React কম্পোনেন্টের আউটপুট assert, manipulate এবং traverse করার কাজগুলো সহজ করে দেয়।
=======
> We recommend using [React Testing Library](https://testing-library.com/react) which is designed to enable and encourage writing tests that use your components as the end users do.
> 
> For React versions <= 16, the [Enzyme](https://airbnb.io/enzyme/) library makes it easy to assert, manipulate, and traverse your React Components' output.


>>>>>>> abcf0358d43caa0772e599949458df9e6578489a

 - [`act()`](#act)
 - [`mockComponent()`](#mockcomponent)
 - [`isElement()`](#iselement)
 - [`isElementOfType()`](#iselementoftype)
 - [`isDOMComponent()`](#isdomcomponent)
 - [`isCompositeComponent()`](#iscompositecomponent)
 - [`isCompositeComponentWithType()`](#iscompositecomponentwithtype)
 - [`findAllInRenderedTree()`](#findallinrenderedtree)
 - [`scryRenderedDOMComponentsWithClass()`](#scryrendereddomcomponentswithclass)
 - [`findRenderedDOMComponentWithClass()`](#findrendereddomcomponentwithclass)
 - [`scryRenderedDOMComponentsWithTag()`](#scryrendereddomcomponentswithtag)
 - [`findRenderedDOMComponentWithTag()`](#findrendereddomcomponentwithtag)
 - [`scryRenderedComponentsWithType()`](#scryrenderedcomponentswithtype)
 - [`findRenderedComponentWithType()`](#findrenderedcomponentwithtype)
 - [`renderIntoDocument()`](#renderintodocument)
 - [`Simulate`](#simulate)

## রেফারেন্স {#reference}

### `act()` {#act}

একটি কম্পোনেন্টকে assertions এর জন্য প্রস্তুত করতে ঐ কম্পোনেন্টের রেন্ডারিং এবং আপডেটের জন্য লিখা সকল কোডকে একটি `act()` কলের মধ্যে অন্তর্ভুক্ত করুন। এটি আপনার টেস্টগুলো React ব্রাউজারে যেভাবে কাজ করে তার অনেকটা কাছাকাছিভাবে চালনা করে।

>বিঃদ্রঃ
>
>আপনি যদি `react-test-renderer` ব্যবহার করে থাকেন, এটিও একটি `act` export সরবরাহ করে থাকে যা একইভাবে কাজ করে।

উদাহরণস্বরূপ, ধরে নিই আমাদের এই `Counter` কম্পোনেন্টটি আছেঃ

```js
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: 0};
    this.handleClick = this.handleClick.bind(this);
  }
  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }
  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }
  handleClick() {
    this.setState(state => ({
      count: state.count + 1,
    }));
  }
  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={this.handleClick}>
          Click me
        </button>
      </div>
    );
  }
}
```

আমরা একে নিচের মত টেস্ট করতে পারিঃ

```js{3,20-22,29-31}
import React from 'react';
import ReactDOM from 'react-dom';
import { act } from 'react-dom/test-utils';
import Counter from './Counter';

let container;

beforeEach(() => {
  container = document.createElement('div');
  document.body.appendChild(container);
});

afterEach(() => {
  document.body.removeChild(container);
  container = null;
});

it('can render and update a counter', () => {
  // Test first render and componentDidMount
  act(() => {
    ReactDOM.render(<Counter />, container);
  });
  const button = container.querySelector('button');
  const label = container.querySelector('p');
  expect(label.textContent).toBe('You clicked 0 times');
  expect(document.title).toBe('You clicked 0 times');

  // Test second render and componentDidUpdate
  act(() => {
    button.dispatchEvent(new MouseEvent('click', {bubbles: true}));
  });
  expect(label.textContent).toBe('You clicked 1 times');
  expect(document.title).toBe('You clicked 1 times');
});
```

- আমাদের মনে রাখতে হবে DOM ইভেন্ট dispatch করলে তা শুধু তখনই কাজ করবে যখন DOM কন্টেইনারটি `document` এ সংযুক্ত করা হবে। আপনি বয়লারপ্লেট কোড কমাতে [React Testing Library](https://testing-library.com/react) এর মত একটি লাইব্রেরি ব্যবহার করতে পারেন।

- [`recipes`](/docs/testing-recipes.html) পৃষ্ঠায় `act()` কিভাবে আচরণ করে তা বিস্তারিতভাবে উদাহরণ এবং ব্যবহারপদ্ধতি সহ দেওয়া আছে।

* * *

### `mockComponent()` {#mockcomponent}

```javascript
mockComponent(
  componentClass,
  [mockTagName]
)
```

একটি mocked কম্পোনেন্ট মডিউলকে এই মেথডে পাস করে এর সাথে বিভিন্ন প্রয়োজনীয় মেথড জুড়ে দিতে পারবেন যা একে একটি কৃত্রিম React কম্পোনেন্ট হিসেবে ব্যবহার করার সুযোগ করে দেবে। সাধারণভাবে রেন্ডার করার পরিবর্তে কম্পোনেন্টটি এর সরবরাহকৃত children সহ একটি সাধারণ `<div>` (যদি `mockTagName` সরবরাহ করা হয় তাহলে অন্য ট্যাগে) এ পরিণত হবে।

> বিঃদ্রঃ
>
> `mockComponent()` হল একটি legacy এপিআই। আমরা এর পরিবর্তে [`jest.mock()`](https://facebook.github.io/jest/docs/en/tutorial-react-native.html#mock-native-modules-using-jestmock) ব্যবহার করার পরামর্শ দিয়ে থাকি।


* * *

### `isElement()` {#iselement}

```javascript
isElement(element)
```

`element` যদি কোন React element হয়ে থাকে তাহলে রিটার্ন করে।

* * *

### `isElementOfType()` {#iselementoftype}

```javascript
isElementOfType(
  element,
  componentClass
)
```

`element` যদি একটি React element হয় যার টাইপ হল একটি React `componentClass` এর তাহলে `true` রিটার্ন করে।

* * *

### `isDOMComponent()` {#isdomcomponent}

```javascript
isDOMComponent(instance)
```

`instance` যদি একটি DOM কম্পোনেন্ট হয়ে থাকে (যেমনঃ `<div>` অথবা `<span>`) তাহলে `true` রিটার্ন করে।

* * *

### `isCompositeComponent()` {#iscompositecomponent}

```javascript
isCompositeComponent(instance)
```

`instance` যদি একটি ইউজার দ্বারা ডিফাইনকৃত কম্পোনেন্ট, যেমনঃ একটি ক্লাস অথবা একটি ফাংশন হয়ে থাকে তাহলে `true` রিটার্ন করে।

* * *

### `isCompositeComponentWithType()` {#iscompositecomponentwithtype}

```javascript
isCompositeComponentWithType(
  instance,
  componentClass
)
```

`instance` যদি React `componentClass` টাইপের একটি কম্পোনেন্ট হয় তাহলে `true` রিটার্ন করে।

* * *

### `findAllInRenderedTree()` {#findallinrenderedtree}

```javascript
findAllInRenderedTree(
  tree,
  test
)
```

`tree` তে থাকা সকল কম্পোনেন্টকে ট্রাভার্স করে এবং যেসকল কম্পোনেন্টের ক্ষেত্রে `test(component)` এর মান `true` হয় সেগুলোকে একত্রিত করে। এটি নিজে থেকে তেমন একটা দরকারি হয়না, কিন্তু অন্যান্য টেস্ট ইউটিলিটির ক্ষেত্রে এটি একটি প্রারম্ভিক মান হিসেবে কাজ করে।

* * *

### `scryRenderedDOMComponentsWithClass()` {#scryrendereddomcomponentswithclass}

```javascript
scryRenderedDOMComponentsWithClass(
  tree,
  className
)
```

রেন্ডারকৃত ট্রি থেকে কম্পোনেন্টের সকল DOM elements কে খুঁজে বের করে যাদের class name সরবরাহকৃত `className` এর সাথে মিলে যায়।

* * *

### `findRenderedDOMComponentWithClass()` {#findrendereddomcomponentwithclass}

```javascript
findRenderedDOMComponentWithClass(
  tree,
  className
)
```

[`scryRenderedDOMComponentsWithClass()`](#scryrendereddomcomponentswithclass) এর মতই কিন্তু এটি শুধুমাত্র একটি ফলাফলই আশা করে এবং ঐ ফলাফল রিটার্ন করে, অন্যথায় যদি একের অধিক ফলাফল পাওয়া যায় এক্সেপশন থ্রো করে।

* * *

### `scryRenderedDOMComponentsWithTag()` {#scryrendereddomcomponentswithtag}

```javascript
scryRenderedDOMComponentsWithTag(
  tree,
  tagName
)
```

রেন্ডারকৃত ট্রি থেকে সকল DOM elements খুঁজে বের করে যাদের tag name এর সাথে `tagName` মিলে যায়।

* * *

### `findRenderedDOMComponentWithTag()` {#findrendereddomcomponentwithtag}

```javascript
findRenderedDOMComponentWithTag(
  tree,
  tagName
)
```

[`scryRenderedDOMComponentsWithTag()`](#scryrendereddomcomponentswithtag) এর মতই কিন্তু এটি শুধুমাত্র একটি ফলাফলই আশা করে এবং ঐ ফলাফল রিটার্ন করে, অন্যথায় যদি একের অধিক ফলাফল পাওয়া যায় এক্সেপশন থ্রো করে।

* * *

### `scryRenderedComponentsWithType()` {#scryrenderedcomponentswithtype}

```javascript
scryRenderedComponentsWithType(
  tree,
  componentClass
)
```

`componentClass` টাইপের সকল কম্পোনেন্ট ইন্সট্যান্স খুঁজে বের করে।

* * *

### `findRenderedComponentWithType()` {#findrenderedcomponentwithtype}

```javascript
findRenderedComponentWithType(
  tree,
  componentClass
)
```

[`scryRenderedComponentsWithType()`](#scryrenderedcomponentswithtype) এর মতই কিন্তু এটি শুধুমাত্র একটি ফলাফলই আশা করে এবং ঐ ফলাফল রিটার্ন করে, অন্যথায় যদি একের অধিক ফলাফল পাওয়া যায় এক্সেপশন থ্রো করে।

***

### `renderIntoDocument()` {#renderintodocument}

```javascript
renderIntoDocument(element)
```

ডকুমেন্টের একটি বিচ্ছিন্ন DOM নোডে একটি React element রেন্ডার করে। **এই ফাংশনের একটি DOM এর প্রয়োজন হয়।** এটি কার্যকারভাবে নিচের মত কাজ করেঃ

```js
const domContainer = document.createElement('div');
ReactDOM.render(element, domContainer);
```

> বিঃদ্রঃ
>
> আপনি `React` ইম্পোর্ট করার **আগে** আপনার `window`, `window.document` এবং `window.document.createElement` globally এভেইলেভল থাকতে হবে। অন্যথায় React মনে করবে এটি DOM এ প্রবেশ করতে পারছেনা এবং `setState` এর মত মেথডগুলো কাজ করবেনা।

* * *

## অন্যান্য ইউটিলিটি {#other-utilities}

### `Simulate` {#simulate}

```javascript
Simulate.{eventName}(
  element,
  [eventData]
)
```

একটি DOM নোডে `eventData` ইভেন্ট ডাটা সহ একটি event dispatch এর অনুকরণ করুন।

[React যেসকল ইভেন্ট বুঝতে পারে](/docs/events.html#supported-events) প্রত্যেকটির জন্যই `Simulate` এ একটি মেথড রয়েছে।

**একটি element কে ক্লিক করা**

```javascript
// <button ref={(node) => this.button = node}>...</button>
const node = this.button;
ReactTestUtils.Simulate.click(node);
```

**একটি ইনপুট ফিল্ডের মান পরিবর্তন করা এবং এরপর ENTER চাপা।**

```javascript
// <input ref={(node) => this.textInput = node} />
const node = this.textInput;
node.value = 'giraffe';
ReactTestUtils.Simulate.change(node);
ReactTestUtils.Simulate.keyDown(node, {key: "Enter", keyCode: 13, which: 13});
```

> বিঃদ্রঃ
>
> আপনার কম্পোনেন্টে ব্যবহৃত যেকোন ইভেন্ট প্রোপার্টি আপনাকেই সরবরাহ করতে হবে (যেমনঃ keyCode, which, ইত্যাদি...) যেহেতু React আপনার জন্য এর কোনটিই তৈরি করছেনা।

* * *
