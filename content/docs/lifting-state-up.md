---
id: lifting-state-up
title: State উপরে তোলা
permalink: docs/lifting-state-up.html
prev: forms.html
next: composition-vs-inheritance.html
redirect_from:
  - "docs/flux-overview.html"
  - "docs/flux-todo-list.html"
---

অনেক সময়, একই ডাটা পরিবর্তন বিভিন্ন কম্পোনেন্টে একসাথে প্রতিফলিত করতে হয়। আমরা এক্ষেত্রে কম্পোনেন্টগুলোর shared state কে তাদের নিকটতম ancestor এ তুলে আনার পরামর্শ দেই। চলুন আমরা দেখি কিভাবে এটি কাজ করে।

এই অনুচ্ছেদে, আমরা একটি তাপমাত্রা পরিমাপক ক্যালকুলেটর তৈরি করব যা হিসাব করবে আদৌ কোন প্রদত্ত তাপমাত্রায় পানি ফুটতে শুরু করবে কিনা।

আমরা `BoilingVerdict` নামের একটি কম্পোনেন্টের মাধ্যমে শুরু করব। এটি `celsius` তাপমাত্রা কে একটি prop হিসেবে গ্রহণ করে এবং প্রিন্ট করে আদৌ ঐ তাপমাত্রা পানি ফুটানোর জন্য যথেষ্ট কিনাঃ

```js{3,5}
function BoilingVerdict(props) {
  if (props.celsius >= 100) {
    return <p>The water would boil.</p>;
  }
  return <p>The water would not boil.</p>;
}
```

এরপর, আমরা `Calculator` নামের একটি কম্পোনেন্ট তৈরি করব। এটি একটি `<input>` রেন্ডার করে যা আপনাকে তাপমাত্রা ইনপুট করার সুযোগ করে দেয় এবং ঐ মান `this.state.temperature` এ সংরক্ষণ করে।

এছাড়াও, এটি বর্তমান ইনপুট এর মানের জন্য `BoilingVerdict` রেন্ডার করে।

```js{5,9,13,17-21}
class Calculator extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {temperature: ''};
  }

  handleChange(e) {
    this.setState({temperature: e.target.value});
  }

  render() {
    const temperature = this.state.temperature;
    return (
      <fieldset>
        <legend>Enter temperature in Celsius:</legend>
        <input
          value={temperature}
          onChange={this.handleChange} />
        <BoilingVerdict
          celsius={parseFloat(temperature)} />
      </fieldset>
    );
  }
}
```

[**CodePen এ চালিয়ে দেখুন**](https://codepen.io/gaearon/pen/ZXeOBm?editors=0010)

## দ্বিতীয় একটি ইনপুট যুক্ত করা {#adding-a-second-input}

আমাদের নতুন প্রয়োজনীয়তা হল, আমরা সেলসিয়াস ইনপুটের পাশাপাশি ফারেনহাইট ইনপুট ও সরবরাহ করব এবং তারা একে অপরের মানের সাথে মিলিয়ে চলবে।

আমরা `Calculator` কম্পোনেন্ট থেকে একটি `TemperatureInput` কম্পোনেন্ট বের করে আনার মাধ্যমে শুরু করতে পারি। আমরা এর সাথে নতুন একটি `scale` prop সংযুক্ত করব যার মান `"c"` অথবা `"f"` হবেঃ

```js{1-4,19,22}
const scaleNames = {
  c: 'Celsius',
  f: 'Fahrenheit'
};

class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {temperature: ''};
  }

  handleChange(e) {
    this.setState({temperature: e.target.value});
  }

  render() {
    const temperature = this.state.temperature;
    const scale = this.props.scale;
    return (
      <fieldset>
        <legend>Enter temperature in {scaleNames[scale]}:</legend>
        <input value={temperature}
               onChange={this.handleChange} />
      </fieldset>
    );
  }
}
```

আমরা এখন `Calculator` কে পরিবর্তন করতে পারি যাতে তা দু'টি ভিন্ন তাপমাত্রার ইনপুট রেন্ডার করেঃ

```js{5,6}
class Calculator extends React.Component {
  render() {
    return (
      <div>
        <TemperatureInput scale="c" />
        <TemperatureInput scale="f" />
      </div>
    );
  }
}
```

[**CodePen এ চালিয়ে দেখুন**](https://codepen.io/gaearon/pen/jGBryx?editors=0010)

আমাদের এখন দুইটি ইনপুট রয়েছে, কিন্তু যখন আপনি এদের যেকোন একটিতে তাপমাত্রা প্রবেশ করাবেন, অন্যটিতে কোন আপডেট হবেনা। এটি আমাদের প্রয়োজনীয়তার বিপরীতঃ আমরা এদেরকে একইসাথে আপডেট করতে চাই।

এছাড়াও আমরা `Calculator` থেকে `BoilingVerdict` দেখাতে পারবনা। `Calculator` জানেনা বর্তমান তাপমাত্রা কত কারণ এটি `TemperatureInput` এর ভেতর লুকানো রয়েছে।

## পরিবর্তনের ফাংশন লিখা {#writing-conversion-functions}

প্রথমত, আমরা দুইটি ফাংশন লিখব যা সেলসিয়াস কে ফারেনহাইটে এবং ফারেনহাইটকে পুনরায় সেলসিয়াসে রূপান্তর করতে পারেঃ

```js
function toCelsius(fahrenheit) {
  return (fahrenheit - 32) * 5 / 9;
}

function toFahrenheit(celsius) {
  return (celsius * 9 / 5) + 32;
}
```

এই দুইটি ফাংশন numbers কনভার্ট করে। আমরা আরও একটি ফাংশন লিখব যা একটি string `temperature` এবং একটি converter function কে আর্গুমেন্ট হিসেবে নেবে এবং একটি string রিটার্ন করবে। আমরা এর মাধ্যমে একটি ইনপুটের মান থেকে অন্য ইনপুটের মান নির্ণয় করব।

এটি ভুল `temperature` এর জন্য একটি খালি string রিটার্ন করবে, এবং এটি তিন দশমিক স্থান পর্যন্ত আউটপুটের মান নিখুঁতভাবে রাখবেঃ

```js
function tryConvert(temperature, convert) {
  const input = parseFloat(temperature);
  if (Number.isNaN(input)) {
    return '';
  }
  const output = convert(input);
  const rounded = Math.round(output * 1000) / 1000;
  return rounded.toString();
}
```

উদাহরণস্বরূপ, `tryConvert('abc', toCelsius)` একটি খালি string রিটার্ন করে, এবং `tryConvert('10.22', toFahrenheit)` রিটার্ন করে `'50.396'`।

## State উপরে তোলা {#lifting-state-up}

বর্তমানের, উভয় `TemperatureInput` কম্পোনেন্টই স্বাধীনভাবে তাদের মান local state এ সংরক্ষণ করেঃe:

```js{5,9,13}
class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {temperature: ''};
  }

  handleChange(e) {
    this.setState({temperature: e.target.value});
  }

  render() {
    const temperature = this.state.temperature;
    // ...  
```

কিন্তু, আমরা চাই এই দুইটি ইনপুট যাতে একে অপরের সাথে সামঞ্জস্যপূর্ণ হয়। যখনই আমরা সেলসিয়াস ইনপুট আপডেট করব, তখনই ফারেনহাইট ইনপুটও রূপান্তরিত তাপমাত্রা প্রতিফলিত করবে, এবং এর উল্টোটাও হবে।

React-এ, যেই কম্পোনেন্টগুলোর state শেয়ার করার দরকার হয় তাদের নিকটতম ancestor কম্পোনেন্টে ঐ state কে তুলে এনে state শেয়ার করা হয়। একে বলা হয় "lifting state up"। আমরা `TemperatureInput` থেকে এর লোকাল state টি মুছে ফেলব এবং একে `Calculator` কম্পোনেন্টে নিয়ে যাব।

যদি `Calculator` এই শেয়ারকৃত state নিয়ন্ত্রণ করে, এটি তখন উভয় ইনপুটেই বর্তমান তাপমাত্রার জন্য "সত্যের উৎস" হয়ে ওঠে। এটি তখন এদের নির্দেশ দিতে পারে যাতে উভয়ের মান একে অপরের সাথে সামঞ্জস্যপূর্ণ হয়। যেহেতু, উভয় `TemperatureInput` এর props গুলোই তাদের একই parent `Calculator` কম্পোনেন্ট থেকে আসছে, সেহেতু উভয় ইনপুটই সবসময় একে অপরের সাথে সামঞ্জস্যপূর্ণ থাকবে।

চলুন, ধাপে ধাপে দেখি এটি কিভাবে কাজ করে।

প্রথমত, আমরা `TemperatureInput` কম্পোনেন্টের `this.state.temperature` কে `this.props.temperature` দ্বারা প্রতিস্থাপিত করব। এখনের জন্য, ধরে নিই `this.props.temperature` বিদ্যমান, যদিও ভবিষ্যতে আমাদেরকে এটি `Calculator` থেকে পাস করতে হবেঃ

```js{3}
  render() {
    // Before: const temperature = this.state.temperature;
    const temperature = this.props.temperature;
    // ...
```

আমরা জানি, [prop হল read-only](/docs/components-and-props.html#props-are-read-only)। যখন `temperature` লোকাল state-এ ছিল, তখন `TemperatureInput` `this.setState()` কল করেই এটি পরিবর্তন করতে পারত। যাহোক, এখন যেহেতু `temperature` parent থেকে একটি prop হিসেবে আসছে, সেহেতু `TemperatureInput`-এর `temperature`-এর উপর কোন নিয়ন্ত্রণ নেই।

React-এ, সাধারণ একটি কম্পোনেন্টকে "controlled" এ রূপান্তরিত করে এটি সমাধান করা হয়। ঠিক যেমন `<input>` একটি `value` এবং `onChange` prop উভয়ই গ্রহণ করে, তেমন করেই `Calculator` parent থেকে কাস্টম `TemperatureInput` একটি `temperature` এবং `onTemperatureChange` prop উভয়ই গ্রহণ করবে।

এখন, যখন `TemperatureInput` এর তাপমাত্রা আপডেট করতে চাইবে, তখন এটি `this.props.onTemperatureChange` কল করবেঃ

```js{3}
  handleChange(e) {
    // Before: this.setState({temperature: e.target.value});
    this.props.onTemperatureChange(e.target.value);
    // ...
```

>বিঃদ্রঃ
>
>কাস্টম কম্পোনেন্টে `temperature` অথবা `onTemperatureChange` prop এর নামগুলোর বিশেষ কোন অর্থ নেই। আমরা চাইলে এদেরকে অন্য যেকোন কিছুও নাম দিতে পারতাম, যেমনঃ `value` এবং `onChange` নাম দিতে পারতাম যা একটি সাধারণ প্রচলন।

`onTemperatureChange` prop টি `temperature` prop এর সাথে `Calculator` parent কম্পোনেন্টের মাধ্যমে সরবরাহ করা হবে। এটি এর নিজস্ব local state পরিবর্তনের মাধ্যমে উভয় ইনপুট নতুন মানসহ পুনরায় রেন্ডার করবে। আমরা নতুন `Calculator` এর implementating অতি শীঘ্রই দেখব।

`Calculator` এর পরিবর্তনগুলোতে মনোনিবেশ করার আগে আমরা `TemperatureInput` কম্পোনেন্টের পরিবর্তনগুলো আরেকবার দেখে নিই। আমরা এর local state মুছে ফেলেছি এবং `this.state.temperature` এর পরিবর্তে `this.props.temperature` এর মান আমরা ব্যবহার করছি। কোন পরিবর্তনের ক্ষেত্রে আমরা `this.setState()` এর পরিবর্তে `this.props.onTemperatureChange()` কল করছি, যা `Calculator` কম্পোনেন্ট দ্বারা সরবরাহ করা হচ্ছেঃ

```js{8,12}
class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange(e) {
    this.props.onTemperatureChange(e.target.value);
  }

  render() {
    const temperature = this.props.temperature;
    const scale = this.props.scale;
    return (
      <fieldset>
        <legend>Enter temperature in {scaleNames[scale]}:</legend>
        <input value={temperature}
               onChange={this.handleChange} />
      </fieldset>
    );
  }
}
```

এখন আমরা `Calculator` কম্পোনেন্টটা দেখি।

আমরা বর্তমান ইনপুটের `temperature` এবং `scale` এর local state এ সংরক্ষণ করব। এটাই হল সেই state যা আমরা ইনপুটগুলো থেকে "উপরে তুলেছি", এবং এটি উভয়ের ক্ষেত্রে একমাত্র "সত্যের উৎস" হিসেবে কাজ করবে। এটি উভয় ইনপুট রেন্ডার করার জন্য কমপক্ষে যেই ডাটাগুলো প্রয়োজন তার একটি চিত্র।

উদাহরণস্বরূপ, যদি আমরা সেলসিয়াস ইনপুটে ৩৭ প্রবেশ করাই তাহলে `Calculator` কম্পোনেন্টের state হবেঃ

```js
{
  temperature: '37',
  scale: 'c'
}
```

যদি আমরা পরে ফারেনহাইট ইনপুটের মানকে ২১২ তে পরিবর্তন করি, তাহলে `Calculator` এর state হবেঃ

```js
{
  temperature: '212',
  scale: 'f'
}
```

আমরা চাইলে উভয় ইনপুটের মান সংরক্ষণ করতে পারতাম কিন্তু এটি এক্ষেত্রে অপ্রয়োজনীয়। সর্বশেষ পরিবর্তিত ইনপুটের মান এবং এর স্কেল সংরক্ষণ করাই এক্ষেত্রে যথেষ্ট। আমরা শুধুমাত্র বর্তমান ইনপুটের `temperature` এবং `scale` ব্যবহার করেই অপর ইনপুটের মান নির্ণয় করতে পারি।

উভয় ইনপুটই একে অপরের সাথে পরিবর্তিত হয় কারণ তাদের মান একই state থেকে নির্ণয় করা হয়ঃ

```js{6,10,14,18-21,27-28,31-32,34}
class Calculator extends React.Component {
  constructor(props) {
    super(props);
    this.handleCelsiusChange = this.handleCelsiusChange.bind(this);
    this.handleFahrenheitChange = this.handleFahrenheitChange.bind(this);
    this.state = {temperature: '', scale: 'c'};
  }

  handleCelsiusChange(temperature) {
    this.setState({scale: 'c', temperature});
  }

  handleFahrenheitChange(temperature) {
    this.setState({scale: 'f', temperature});
  }

  render() {
    const scale = this.state.scale;
    const temperature = this.state.temperature;
    const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;
    const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;

    return (
      <div>
        <TemperatureInput
          scale="c"
          temperature={celsius}
          onTemperatureChange={this.handleCelsiusChange} />
        <TemperatureInput
          scale="f"
          temperature={fahrenheit}
          onTemperatureChange={this.handleFahrenheitChange} />
        <BoilingVerdict
          celsius={parseFloat(celsius)} />
      </div>
    );
  }
}
```

[**CodePen এ চালিয়ে দেখুন**](https://codepen.io/gaearon/pen/WZpxpz?editors=0010)

এখন আপনি যেকোন ইনপুটই পরিবর্তন করেন না কেন, `Calculator` এর `this.state.temperature` এবং `this.state.scale` পরিবর্তিত হবে। একটি ইনপুটের ক্ষেত্রে মানের কোন পরিবর্তন হবেনা, তাই ইউজারের ইনপুট সংরক্ষিত থাকবে, এবং অপর ইনপুটের মান এর ভিত্তিতে নির্ণয় করা হবে।

চলুন আরেকবার দেখি কি হয় যখন আপনি কোন ইনপুট পরিবর্তন করেনঃ

* React `onChange` এ উল্লেখিত ফাংশনকে DOM `<input>` এ কল করে। আমাদের ক্ষেত্রে, এটি হল `TemperatureInput` কম্পোনেন্টের `handleChange` মেথড।
* `TemperatureInput` কম্পোনেন্টের `handleChange` মেথড `this.props.onTemperatureChange()` কে নতুন আকাঙ্ক্ষিত মানসহ কল করে। `onTemperatureChange` সহ এটির prop গুলো এর parent কম্পোনেন্ট `Calculator` দ্বারা সরবরাহ করা হয়।
* যখন এটি আগে রেন্ডার হয়েছিল তখন `Calculator` কম্পোনেন্ট উল্লেখ করে দিয়েছিল যে সেলসিয়াস `TemperatureInput` এর `onTemperatureChange` হল `Calculator`-এর `handleCelsiusChange` মেথড, এবং ফারেনহাইট `TemperatureInput` এর `onTemperatureChange` হল `Calculator`-এর `handleFahrenheitChange` মেথড। তাই আমরা কোন ইনপুট পরিবর্তন করেছি এর উপর ভিত্তি করে `Calculator` এর এই মেথডগুলোর যেকোন একটি কল করা হয়।
* এই মেথডগুলোর ভেতরে, `Calculator` কম্পোনেন্ট `this.setState()` কল করার মাধ্যমে React কে নির্দেশ দেয় নিজেকে নতুন ইনপুট এর মান এবং আমরা এইমাত্র যে ইনপুট পরিবর্তন করেছি তার স্কেল সহ পুনরায় রেন্ডার করার।
* React `Calculator` কম্পোনেন্টের `render` মেথডকে কল করে জেনে নেয় ইউজার ইন্টারফেস দেখতে কেমন হবে। উভয় ইনপুটের মানই বর্তমান তাপমাত্রা এবং স্কেলের ভিত্তিতে পুনরায় হিসাব করা হয়। তাপমাত্রার রূপান্তর এই অংশে সম্পাদিত হয়।
* React প্রতিটি `TemperatureInput` কম্পোনেন্টের `render` মেথডকে তাদের নতুন prop গুলো সহ কল করে যা `Calculator` দ্বারা সরবরাহ করা হয়। এটি জানতে পারে এদের ইউজার ইন্টারফেস দেখতে কেমন হওয়া উচিত।
* React `BoilingVerdict` কম্পোনেন্টের `render` মেথডকে কল করে সাথে এর prop হিসেবে সেলসিয়াস স্কেলে তাপমাত্রা সরবরাহ করে।
* React DOM পানি ফুটার সিদ্ধান্তসহ DOM কে পরিবর্তন করে যাতে তা ইনপুটের মানের সাথে সামঞ্জস্যপূর্ণ হয়। আমরা এইমাত্র যেই ইনপুট পরিবর্তন করেছি তা এর বর্তমান মান গ্রহণ করে, এবং অন্য ইনপুট রূপান্তরের পর পরিবর্তন করা হয়।

প্রতিটি পরিবর্তন একই ধাপগুলো অনুসরণ করে ফলে ইনপুটগুলো সবসময় একইসাথে তাল মিলিয়ে চলে।

## যা শিখলাম {#lessons-learned}

React অ্যাপ্লিকেশনে যেকোন ডাটা পরিবর্তনের জন্য শুধুমাত্র একটি "সত্যের উৎস" থাকবে। সাধারণত, state সর্বপ্রথম সেই কম্পোনেন্টে সংযুক্ত করা হয় যার রেন্ডারে এর দরকার পড়ে। এরপর, যদি অন্যান্য কম্পোনেন্টেরও এর দরকার হয় তাহলে আপনি একে এদের নিকটতম সাধারণ ancestor কম্পোনেন্টে তুলে আনতে পারেন। বিভিন্ন কম্পোনেন্টে state একইসাথে পরিবর্তন করার বদলে আপনার [top-down data flow](/docs/state-and-lifecycle.html#the-data-flows-down) এর উপর নির্ভর করা উচিত।

State উপরে তোলার ক্ষেত্রে two-way binding পদ্ধতির চেয়ে অনেক বেশি "boilerplate" কোড লিখতে হয়, কিন্তু এর একটি সুবিধা হল এতে কম পরিশ্রমে কোডের বাগগুলো খুঁজে বের করা যায়। যেহেতু যেকোন state কোন একটি কম্পোনেন্টে "বসবাস" করে এবং শুধুমাত্র ঐ কম্পোনেন্টই তা পরিবর্তন করতে পারে, কোডের বাগের বিচরণস্থল অনেকখানি কমে যায়। এরপরও আপনি ইউজার ইনপুট প্রত্যাখ্যান করার জন্য যেকোন কাস্টম লজিক দাঁড় করাতে পারেন।

যদি কোনকিছু props অথবা state থেকে বের করা সম্ভব হয়, তাহলে সম্ভবত এটি state এ না থাকা উচিত। উদাহরণস্বরূপ, `celsiusValue` এবং `fahrenheitValue` উভয়কেই সংরক্ষণ করার পরিবর্তে আমরা শুধুমাত্র সর্বশেষ পরিবর্তিত `temperature` এবং এর `scale` সংরক্ষণ করছি। অপর ইনপুটের মান সবসময়ই `render()` মেথডে এদের থেকে হিসাব করা যায়। এটি আমাদেরকে অপর ফিল্ডটি মুছে ফেলার বা মানের নির্ভুলতা সংরক্ষণ করে রাউন্ডিং করতে সাহায্য করে।

<<<<<<< HEAD
যখন আপনি ইউজার ইন্টারফেসে কিছু ভুল দেখতে পাবেন, তখন আপনি [React Developer Tools](https://github.com/facebook/react/tree/master/packages/react-devtools) ব্যবহার করে prop গুলো পরীক্ষা করে দেখতে পারেন এবং tree তে এগুলো উপরে তুলতে পারেন যতক্ষণ না আপনি state পরিবর্তনকারী কম্পোনেন্টটি খুঁজে পাচ্ছেন। এটি আপনাকে বাগগুলো এদের উৎস থেকে খুঁজে বের করতে সাহায্য করেঃ
=======
When you see something wrong in the UI, you can use [React Developer Tools](https://github.com/facebook/react/tree/main/packages/react-devtools) to inspect the props and move up the tree until you find the component responsible for updating the state. This lets you trace the bugs to their source:
>>>>>>> 0bb0303fb704147452a568472e968993f0729c28

<img src="../images/docs/react-devtools-state.gif" alt="Monitoring State in React DevTools" max-width="100%" height="100%">
