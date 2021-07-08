---
id: lists-and-keys
title: Lists and Keys
permalink: docs/lists-and-keys.html
prev: conditional-rendering.html
next: forms.html
---

চলুন প্রথমে আমরা পর্যালোচনা করি কিভাবে আপনি জাভাস্ক্রিপ্টে লিস্ট রুপান্তর করবেন।

নিচে প্রদত্ত কোড অনুযায়ী আমরা [`map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) ফাংশনটি ব্যবহার করে একটি `numbers` array নিয়ে তার মানকে দিগুণ করে দেই। `map()` ফাংশনটি দ্বারা প্রাপ্ত মানকে আমরা `doubled` ভ্যারিয়েবল নামে সংজ্ঞায়িত করে একে কনসোলে লগ করিঃ

```javascript{2}
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((number) => number * 2);
console.log(doubled);
```

এই কোড কনসোলে `[2, 4, 6, 8, 10]` প্রকাশ করে।

React এ array কে লিস্ট [elements](/docs/rendering-elements.html) এ রূপান্তর করাও প্রায় একই রকম।

### একাধিক কম্পোনেন্ট রেন্ডার করা।{#rendering-multiple-components}

আপনি কারলি ব্র্যাকেট `{}` ব্যবহার করে elements এর সম্ভার গঠন করতে পারবেন এবং তাদেরকে [JSX এর অন্তর্ভুক্ত করতে পারবেন](/docs/introducing-jsx.html#embedding-expressions-in-jsx)

নিচে আমরা জাভাস্ক্রিপ্ট [`map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) ফাংশন ব্যবহার করে `numbers` array কে লুপ করি। প্রতিটি আইটেমের জন্য আমরা একটি `<li>` element রিটার্ন করি। পরিশেষে আমরা প্রাপ্ত array elements কে `listItems` এর অন্তর্ভুক্ত করিঃ

```javascript{2-4}
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li>{number}</li>
);
```

সম্পূর্ণ `listItems` array কে আমরা একটি `<ul>` element এর ভিতরে রাখি এবং [এটিকে DOM এ রেন্ডার করিঃ](/docs/rendering-elements.html#rendering-an-element-into-the-dom) 

```javascript{2}
ReactDOM.render(
  <ul>{listItems}</ul>,
  document.getElementById('root')
);
```

[**CodePen এ চালিয়ে দেখুন**](https://codepen.io/gaearon/pen/GjPyQr?editors=0011)

এই কোড ১ থেকে ৫ পর্যন্ত সংখ্যা গুলোকে বুলেট লিস্ট আকারে প্রকাশ করে।

### বেসিক লিস্ট কম্পোনেন্ট {#basic-list-component}

সাধারণত আপনি লিস্টগুলোকে [কম্পোনেন্টের](/docs/components-and-props.html) মধ্যে রেন্ডার করবেন।

আমরা পূর্বের উদাহারণকে এমন একটি কম্পোনেন্টে রিফ্যাক্টর করতে পারি যেটা `numbers` array কে গ্রহণ করবে এবং একটি elements লিস্ট আউটপুট করবে ।

```javascript{3-5,7,13}
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li>{number}</li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

আপনি যখন এই কোড রান করবেন তখন একটি সতর্কীকরণ বার্তা পাবেন যে লিস্ট আইটেমগুলোর জন্য একটি করে  `key` দরকার। "Key" হল একটি বিশেষ `string` attribute যা আপনার তখনই দরকার যখন আপনি elements এর লিস্ট তৈরি করবেন। এটি কি কারণে গুরুত্বপূর্ণ তা আমরা পরবর্তী অনুচ্ছেদে আলোচনা করব।

আসুন `numbers.map()` এর মধ্যে আমাদের লিস্ট আইটেমকে `key` দিয়ে দেই এবং 'key' না থাকার ইস্যুর সমাধান করি।

```javascript{4}
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

[**CodePen এ চালিয়ে দেখুন**](https://codepen.io/gaearon/pen/jrXYRR?editors=0011)

## Keys {#keys}

Keys, কোন আইটেমটি পরিবর্তিত হয়েছে, কোনটি যোগ হয়েছে অথবা কোনটি মুছে ফেলা হয়েছে তা সনাক্ত করতে React কে সাহায্য করে। Elements কে স্থির পরিচয় দেয়ার জন্য array এর মধ্যে elements কে keys প্রদান করতে হবেঃ

```js{3}
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>
    {number}
  </li>
);
```

Key নেয়ার সবচেয়ে ভালো উপায় হল এমন string ব্যবহার করা যেগুলো পাশাপাশি অবস্থিত লিস্ট আইটেমগুলোকে স্বতন্ত্রভাবে চিহ্নিত করতে পারে। অধিকাংশ সময় আপনি আপনার ডাটা থেকে আইডিগুলোকে keys হিসাবে ব্যবহার করবেনঃ

```js{2}
const todoItems = todos.map((todo) =>
  <li key={todo.id}>
    {todo.text}
  </li>
);
```

যখন আপনার রেন্ডারড আইটেমের জন্য কোন স্থিতিশীল আইডি থাকবেনা তখন আপনি শেষ অবলম্বন হিসাবে আইটেমের ইনডেক্সকে key হিসাবে ব্যবহার করতে পারেনঃ

```js{2,3}
const todoItems = todos.map((todo, index) =>
  // Only do this if items have no stable IDs
  <li key={index}>
    {todo.text}
  </li>
);
```

যে ক্ষেত্রে আইটেমের অর্ডার পরিবর্তন হতে পারে সে ক্ষেত্রে আমরা ইনডেক্সকে ব্যবহার করার পরামর্শ দেবনা। এটি পারফরমেন্সের উপর নেতিবাচক প্রভাব ফেলতে পারে এবং কম্পোনেন্টের state এর সমস্যা হতে পারে। এই বিষয় Robin Pokorny's এর আর্টিকেল [ইনডেক্সকে key হিসাবে ব্যবহারের নেতিবাচক প্রভাব নিয়ে বিস্তারিত ব্যাখ্যা](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318) দেখুন। আপনি যদি লিস্ট আইটেমকে সুস্পষ্ট কোন key নির্ধারণ করে না দেন তাহলে React ডিফল্ট হিসেবে ইনডেক্সকে keys হিসেবে ব্যবহার করবে।

আপনি যদি বিস্তারিত জানতে চান তাহলে [keys কেন দরকারি এ বিষয়ে গভীর আলোচনাটি](/docs/reconciliation.html#recursing-on-children) দেখতে পারেন।

### কম্পোনেন্টেকে Keys সহ পৃথক করা {#extracting-components-with-keys}

Keys এর ব্যবহার তখনি অর্থপূর্ণ হয়, যখন আপনার context এ arrays থাকে।

উদাহারণস্বরূপ যদি আপনি একটি `ListItem` কম্পোনেন্টেকে [পৃথক](/docs/components-and-props.html#extracting-components) করেন তাহলে আপনি array এর key কে `ListItem` এর `<li>`element এর পরিবর্তে `<ListItem />` element এই দিয়ে দিবেন।

**উদাহারণ: Key এর ভুল ব্যবহার**

```javascript{4,5,14,15}
function ListItem(props) {
  const value = props.value;
  return (
    // Wrong! There is no need to specify the key here:
    <li key={value.toString()}>
      {value}
    </li>
  );
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // Wrong! The key should have been specified here:
    <ListItem value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

**উদাহারণ: Key এর সঠিক ব্যবহার**

```javascript{2,3,9,10}
function ListItem(props) {
  // Correct! There is no need to specify the key here:
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // Correct! Key should be specified inside the array.
    <ListItem key={number.toString()} value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

[**CodePen এ চালিয়ে দেখুন**](https://codepen.io/gaearon/pen/ZXeOGM?editors=0010)

একটি ভাল প্রচলিত রীতি হল `map()` ফাংশন  কলের মধ্যের elements গুলোর keys দরকার।

### পারিপার্শ্বিক elements গুলোর মধ্যে Keys কে ইউনিক হতে হবে {#keys-must-only-be-unique-among-siblings}

<<<<<<< HEAD
Array এর মধ্যে ব্যবহৃত keys তাদের পারিপার্শ্বিক অন্যান্য আইটেমগুলোর মধ্যে ইউনিক হতে হবে কিন্ত গ্লোবাল ভাবে তাদের ইউনিক হওয়া দরকারি নয়। আমরা দুইটি ভিন্ন ভিন্ন array এর ক্ষেত্রে একই keys ব্যবহার করতে পারিঃ
=======
Keys used within arrays should be unique among their siblings. However, they don't need to be globally unique. We can use the same keys when we produce two different arrays:
>>>>>>> 25f756d8e3800afb032cb31ce3626d6134e31e38

```js{2,5,11,12,19,21}
function Blog(props) {
  const sidebar = (
    <ul>
      {props.posts.map((post) =>
        <li key={post.id}>
          {post.title}
        </li>
      )}
    </ul>
  );
  const content = props.posts.map((post) =>
    <div key={post.id}>
      <h3>{post.title}</h3>
      <p>{post.content}</p>
    </div>
  );
  return (
    <div>
      {sidebar}
      <hr />
      {content}
    </div>
  );
}

const posts = [
  {id: 1, title: 'Hello World', content: 'Welcome to learning React!'},
  {id: 2, title: 'Installation', content: 'You can install React from npm.'}
];
ReactDOM.render(
  <Blog posts={posts} />,
  document.getElementById('root')
);
```

[**CodePen এ চালিয়ে দেখুন**](https://codepen.io/gaearon/pen/NRZYGN?editors=0010)

 React এ সংকেত হিসাবে keys ব্যবহৃত হয় কিন্ত এটি আপনার কম্পোনেন্টের মধ্যে দিয়ে পাস হয়না। আপনার কম্পোনেন্টের যদি ওই একই মান প্রয়োজন হয় তাহলে এটাকে ভিন্ন নামে prop হিসেবে কম্পোনেন্টে সুস্পষ্ট ভাবে পাস করে দিনঃ

```js{3,4}
const content = posts.map((post) =>
  <Post
    key={post.id}
    id={post.id}
    title={post.title} />
);
```

উপরের উদাহরণ অনুসারে `Post` কম্পোনেন্টটি `props.id` পড়তে পারে কিন্ত `props.key` পড়তে পারে না।

### map() কে JSX এ স্থাপন করা {#embedding-map-in-jsx}

উপরের উদাহরণে আমরা একটি পৃথক `listItems` ভ্যারিয়েবল ডিক্লেয়ার করি এবং এটাকে JSX এর অন্তর্ভুক্ত করিঃ

```js{3-6}
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <ListItem key={number.toString()}
              value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```

JSX কারলি ব্র্যাকেটে [যেকোন এক্সপ্রেশনকে স্থাপন করার](/docs/introducing-jsx.html#embedding-expressions-in-jsx) অনুমতি দেয় তাই আমরা `map()`এর ফলাফল সারিবদ্ধভাবে প্রকাশ করতে পারিঃ

```js{5-8}
function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
      {numbers.map((number) =>
        <ListItem key={number.toString()}
                  value={number} />
      )}
    </ul>
  );
}
```

[**CodePen এ চালিয়ে দেখুন**](https://codepen.io/gaearon/pen/BLvYrB?editors=0010)

এর ফলে মাঝেমাঝে পরিচ্ছন্ন কোড পাওয়া যায়, কিন্ত এই স্টাইলের অপব্যবহারও হতে পারে। জাভাস্ক্রিপ্টের মত এটাও আপনার সিদ্ধান্ত যে পড়ার সুবিধার জন্য আপনার এখান থেকে একটি ভ্যারিয়েবল বের করা যুক্তিযুক্ত কিনা। মনে রাখা দরকার `map()` বডি যদি অনেক বেশি nested হয়, তাহলে এটি একটি [কম্পোনেন্ট বের করে আনার](/docs/components-and-props.html#extracting-components) জন্য উপযুক্ত সময়।