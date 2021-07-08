---
id: handling-events
title: ইভেন্ট হ্যান্ডেলিং
permalink: docs/handling-events.html
prev: state-and-lifecycle.html
next: conditional-rendering.html
redirect_from:
  - "docs/events-ko-KR.html"
---

React element এর মাধ্যমে ইভেন্ট হ্যান্ডেল করা অনেকটা DOM element এর মাধ্যমে ইভেন্ট হ্যান্ডেল করার মতই। এই দুই পদ্ধতির মধ্যে কিছু গঠনগত পার্থক্য আছেঃ

* React এর ইভেন্টগুলো lowercase এর পরিবর্তে camelCase এর মাধ্যমে নামকরণ করা হয়।
* আপনি ইভেন্ট হ্যান্ডলার হিসেবে string এর পরিবর্তে JSX এর মাধ্যমে একটি ফাংশন পাস করেন।

উদাহরণস্বরূপ, নিচের HTML:

```html
<button onclick="activateLasers()">
  Activate Lasers
</button>
```

React এর ক্ষেত্রে কিছুটা ভিন্নঃ

```js{1}
<button onClick={activateLasers}>
  Activate Lasers
</button>
```

<<<<<<< HEAD
আরেকটা বড় পার্থক্য হল, React এ আপনি `false` রিটার্ন করে ডিফল্ট আচরণ প্রতিরোধ করতে পারবেন না। আপনার অবশ্যই আলাদাভাবে `preventDefault` কল করতে হবে। উদাহরণস্বরূপ, সাধারণ HTML এ একটি ডিফল্ট লিংককে একটি নতুন পৃষ্ঠা খোলা থেকে বিরত রাখতে আপনি লিখতে পারেনঃ
=======
Another difference is that you cannot return `false` to prevent default behavior in React. You must call `preventDefault` explicitly. For example, with plain HTML, to prevent the default form behavior of submitting, you can write:
>>>>>>> f3baa6d075c8de475b688abf035d7054bc8a9606

```html
<form onsubmit="console.log('You clicked submit.'); return false">
  <button type="submit">Submit</button>
</form>
```

React এ, এর পরিবর্তে কোডটা এমন হতে পারেঃ

```js{3}
function Form() {
  function handleSubmit(e) {
    e.preventDefault();
    console.log('You clicked submit.');
  }

  return (
    <form onSubmit={handleSubmit}>
      <button type="submit">Submit</button>
    </form>
  );
}
```

এখানে, `e` হল একটি কৃত্রিম ইভেন্ট। React এই কৃত্রিম ইভেন্টগুলো [W3C spec](https://www.w3.org/TR/DOM-Level-3-Events/) অনুযায়ী নির্ধারণ করে, যাতে আপনার ক্রস-ব্রাউজার কম্প্যাটিবিলিটি নিয়ে চিন্তা করতে না হয়। React ইভেন্টগুলো, ন্যাটিভ ইভেন্টের মত কাজ একইভাবে করে না। বিস্তারিত জানতে [`SyntheticEvent`](/docs/events.html) রেফারেন্স গাইডটি দেখুন।

React ব্যবহারের সময় সাধারণত আপনার `addEventListener` কল করে কোন DOM element তৈরি হওয়ার পরে listener সংযুক্ত করার প্রয়োজন হবেনা। এর পরিবর্তে, আপনি element রেন্ডারের সময় একটি listener সরবরাহ করতে পারেন।

আপনি যখন একটি কম্পোনেন্ট [ES6 class](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes) ব্যবহার করে নির্ধারণ করবেন, তখন একটি সাধারণ প্যাটার্ন হল ইভেন্ট হ্যান্ডলারটি ঐ ক্লাসের একটি মেথড হিসেবে থাকা। উদাহরণস্বরূপ, এই `Toggle` কম্পোনেন্টটি একটি বাটন রেন্ডার করে যা ইউজারকে "ON" এবং "OFF" state দুইটির মাঝে toggle করতে দেয়ঃ

```js{6,7,10-14,18}
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};

    // This binding is necessary to make `this` work in the callback
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}

ReactDOM.render(
  <Toggle />,
  document.getElementById('root')
);
```

[**CodePen এ চালিয়ে দেখুন**](https://codepen.io/gaearon/pen/xEmzGg?editors=0010)

JSX কলব্যাকগুলোর মধ্যে `this` এর অর্থ সম্পর্কে আপনার সতর্ক থাকতে হবে। জাভাস্ক্রিপ্টে, class মেথডগুলো ডিফল্টভাবে [bound](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_objects/Function/bind) থাকেনা। আপনি যদি `this.handleClick` কে bind করতে ভুলে যান এবং `onClick` এ পাস করে দেন, এক্ষেত্রে যখন ফাংশনটি কল হবে তখন `this` এর মান হবে `undefined`।

এটি React সম্পর্কিত কোন বিষয় নয়; [জাভাস্ক্রিপ্টে ফাংশন এভাবেই কাজ করে](https://www.smashingmagazine.com/2014/01/understanding-javascript-function-prototype-bind/)। সাধারণত আপনি যখন কোন ফাংশনের শেষে `()` ছাড়া উল্লেখ করবেন, যেমন `onClick={this.handleClick}`, তখন আপনার ঐ মেথডটি bind করা উচিত।

যদি `bind` কল করা আপনার জন্য বিরক্তিকর হয়, তাহলে দুই উপায়ে আপনি এটি এড়িয়ে যেতে পারেন। আপনি যদি পরীক্ষামূলক [public class fields syntax](https://babeljs.io/docs/plugins/transform-class-properties/) ব্যবহার করেন, তাহলে আপনি class fields ব্যবহারের মাধ্যমে সঠিকভাবে কলব্যাকগুলোকে bind করতে পারবেনঃ

```js{2-6}
class LoggingButton extends React.Component {
  // This syntax ensures `this` is bound within handleClick.
  // Warning: this is *experimental* syntax.
  handleClick = () => {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```

[Create React App](https://github.com/facebookincubator/create-react-app) এ এই সিনট্যাক্সটি ডিফল্টভাবেই enable করা থাকে।

আপনি যদি class fields সিনট্যাক্স ব্যবহার না করেন, তাহলে আপনি কলব্যাকে একটি [এরো ফাংশন](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions) ব্যবহার করতে পারেনঃ

```js{7-9}
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    // This syntax ensures `this` is bound within handleClick
    return (
      <button onClick={() => this.handleClick()}>
        Click me
      </button>
    );
  }
}
```

এই সিনট্যাক্সের সমস্যা হল, প্রতিবার `LoggingButton` রেন্ডারের সময় একটি ভিন্ন কলব্যাক তৈরি হবে। অধিকাংশ ক্ষেত্রে, এতে কোন সমস্যা হয়না। যাহোক, যদি এই কলব্যাকটি একটি prop হিসেবে নিচের কম্পোনেন্টগুলোতে পাস করা হয়, তাহলে ঐ কম্পোনেন্টগুলো অতিরিক্ত রি-রেন্ডারিং করতে পারে। আমরা সাধারণত এই ঝামেলা এড়াতে constructor এ bind করার অথবা class fields সিনট্যাক্স ব্যবহার করার পরামর্শ দিয়ে থাকি।

## ইভেন্ট হ্যান্ডলারে আর্গুমেন্ট পাস করুন {#passing-arguments-to-event-handlers}

একটি লুপের ভেতরে ইভেন্ট হ্যান্ডলারে অতিরিক্ত আর্গুমেন্ট পাস করা একটি সাধারণ বিষয়। উদাহরণস্বরূপ, যদি `id` একটি সারির ID হয়, নিচের যেকোন একটি পদ্ধতি অনুসরণ করলেই হবেঃ

```js
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

উপরের দুইটি লাইনই একই কাজ করে, এবং যথাক্রমে [এরো ফাংশন](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) এবং [`Function.prototype.bind`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Function/bind) ব্যবহার করে।

উভয়ক্ষেত্রেই, `e` আর্গুমেন্টটি, যা React ইভেন্ট কে তুলে ধরে, তা ID এর পর দ্বিতীয় আর্গুমেন্ট হিসেবে পাস করা হবে। এরো ফাংশনের ক্ষেত্রে আমাদের এই ইভেন্টটি আলাদাভাবে পাস করতে হয় কিন্তু `bind` এর ক্ষেত্রে অতিরিক্ত সব আর্গুমেন্টই স্বয়ংক্রিয়ভাবে ফরওয়ার্ড করা হয়।
