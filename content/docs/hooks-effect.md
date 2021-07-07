---
id: hooks-state
title: Effect Hook এর ব্যবহার
permalink: docs/hooks-effect.html
next: hooks-rules.html
prev: hooks-state.html
---

*Hooks* - React 16.8 এ নতুন সংযোজন। তারা আপনাকে ক্লাস না লিখে state এবং অন্যান্য React ফিচারকে ব্যবহার করতে দেয়।

*Effect Hook* আপনাকে ফাংশন কম্পোনেন্টকে পার্শ্ব React সম্পাদন করতে দেয়:

```js{1,6-10}
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

এই স্নিপেট উপর ভিত্তি করে [counter example from the previous page](/docs/hooks-state.html), তবে আমরা এতে একটি নতুন বৈশিষ্ট্য যুক্ত করেছি: আমরা ক্লিকের সংখ্যা সহ কাস্টম বার্তায় নথির শিরোনাম সেট করেছি।

তথ্য আনা, একটি সাবস্ক্রিপশন সেট আপ, এবং React কম্পোনেন্টকে ম্যানুয়ালি DOM পরিবর্তন করা পার্শ্ব React গুলির উদাহরণ। আপনি এই অপারেশনগুলিকে "পার্শ্ব প্রতিক্রিয়া" (or just "effects"), বলার জন্য অভ্যস্ত কিনা বা না, আপনি সম্ভবত আপনার কম্পোনেন্টকে এগুলি সম্পাদন করেছেন।

>পরামর্শ
>
>আপনি যদি React শ্রেণীর জীবনচক্র পদ্ধতিগুলির সাথে পরিচিত হন তবে আপনি `UseEffect` হুককে `componentDidMount`, `componentDidUpdate` এবং `componentWillUnmount` সম্মিলিত হিসাবে ভাবতে পারেন।

React কম্পোনেন্টকে দুটি সাধারণ ধরণের পার্শ্ব React রয়েছে: সেগুলির পরিষ্কার-পরিচ্ছন্নতার প্রয়োজন হয় না এবং সেগুলি করে। আসুন আরও বিস্তারিত এই পার্থক্যটি দেখি।

## Effects Without Cleanup {#effects-without-cleanup}

কখনও কখনও, আমরা চাই **রিঅ্যাক্টটি DOM আপডেট করার পরে কিছু অতিরিক্ত কোড চালাও।** নেটওয়ার্ক requests, ম্যানুয়াল DOM রূপান্তর এবং লগিং এফেক্টগুলির সাধারণ উদাহরণ যা পরিষ্কার করার প্রয়োজন হয় না। আমরা বলি কারণ আমরা তাদের চালাতে পারি এবং তাত্ক্ষণিকভাবে সেগুলি ভুলে যেতে পারি। তুলনা করি, কীভাবে ক্লাস এবং হুকস আমাদের এ জাতীয় পার্শ্ব React প্রকাশ করতে দেয়।

### ক্লাস ব্যবহার করার উদাহরণ {#example-using-classes}

React শ্রেণীর কম্পোনেন্টকে, `render` পদ্ধতি নিজেই কোনও পার্শ্ব React সৃষ্টি করে না। এটি খুব তাড়াতাড়ি হবে - আমরা সাধারণত React ডমকে আপডেট করার *পরে* আমাদের প্রভাবগুলি সম্পাদন করতে চাই।

এই কারণেই React শ্রেণিতে আমরা পার্শ্ব Reactগুলিকে `componentDidMount` এবং `componentDidUpdate` আমাদের উদাহরণে ফিরে আসছি, এখানে একটি React পাল্টা ক্লাস উপাদান রয়েছে যা DOM পরিবর্তন React করার পরে ডকুমেন্টের শিরোনামটি আপডেট করে:

```js{9-15}
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

নোট করুন: **ক্লাসে আমাদের এই দুটি জীবনচক্র পদ্ধতির মধ্যে কোডটি নকল করতে হবে**

এটির কারণ কেবলমাত্র উপাদানটি কেবল মাউন্ট করা হয়েছে, বা এটি আপডেট হয়েছে কিনা তা নির্বিশেষে আমরা অনেক ক্ষেত্রে একই পার্শ্ব React সম্পাদন করতে চাই। ধারণামূলকভাবে, আমরা এটি প্রতিটি রেন্ডারের পরে ঘটতে চাই - তবে React শ্রেণীর উপাদানগুলির মতো এই পদ্ধতি নেই। আমরা একটি পৃথক পদ্ধতিতে নিষ্কাশন করতে পারি তবে আমাদের এখনও এটি দুটি জায়গায় কল করতে হবে।

এখন আসুন আমরা কীভাবে `UseEffect` হুক দিয়ে একই কাজ করতে পারি তা দেখি।

### হুক ব্যবহার করার উদাহরণ {#example-using-hooks}

আমরা ইতিমধ্যে পৃষ্ঠার শীর্ষে এই উদাহরণটি দেখেছি, তবে আসুন এটি ঘনিষ্ঠভাবে দেখি:

```js{1,6-8}
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

**`UseEffect` কি করে?** এই হুক ব্যবহার করে, আপনি React জানান যে আপনার উপাদানটি রেন্ডারের পরে কিছু করা দরকার। React আপনি যে ক্রিয়াকলাপটি পাস করেছেন তা মনে রাখবে (আমরা এটিকে আমাদের "effect" হিসাবে উল্লেখ করব), এবং DOM আপডেটগুলি সম্পাদন করার পরে এটি পরে কল করব। এই প্রভাবটিতে, আমরা ডকুমেন্ট শিরোনাম সেট করেছি, তবে আমরা ডেটা আনতে পারফর্ম করতে পারি বা অন্য কোনও অত্যাবশ্যকীয় API কল করতে পারি।

**কেন `useEffect` একটি উপাদানের ভিতরে বলা হয়?** উপাদানটির ভিতরে `useEffect` স্থাপন করা আমাদের প্রভাব থেকে ডানদিকে `count` state পরিবর্তনশীল (বা কোনও প্রপস) অ্যাক্সেস করতে দেয়। এটি পড়তে আমাদের একটি বিশেষ API দরকার নেই - এটি ইতিমধ্যে ফাংশনের সুযোগে। হুকস জাভাস্ক্রিপ্ট বন্ধের আলিঙ্গন করে এবং জাভাস্ক্রিপ্ট ইতিমধ্যে একটি সমাধান সরবরাহ করে এমন React-নির্দিষ্ট API প্রবর্তন এড়ানো।

**`useEffect` কি প্রতি রেন্ডার পরে চলে** হ্যাঁ! গতানুগতিক, এটি প্রতিটি রেন্ডার *and* প্রতিটি আপডেটের পরে উভয়ই চালিত হয়। (আমরা পরে কথা বলব [এটি কীভাবে কাস্টমাইজ করা যায়](#tip-optimizing-performance-by-skipping-effects)) "mounting" এবং "updating" এর ক্ষেত্রে বিবেচনা করার পরিবর্তে, "after render" প্রভাবগুলি ঘটেছিল তা ভাবতে আপনার পক্ষে আরও সহজ হতে পারে। React গ্যারান্টি দেয় যে প্রভাবগুলি চালানোর সময় দিয়ে DOM আপডেট হয়ে গেছে।

### বিস্তারিত ব্যাখ্যা {#detailed-explanation}

এখন আমরা effects সম্পর্কে আরও জানি, এই লাইনগুলি বোঝা উচিত:

```js
function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
}
```

আমরা `count` state পরিবর্তনশীল ঘোষণা করি এবং তারপরে আমরা Reactটি বলি আমাদের একটি effect ব্যবহার করা দরকার। আমরা `useEffect` হুক এ একটি ফাংশন পাস করি। এই ফাংশনটি আমরা পাস *করি* আমাদের effect। আমাদের effects ভিতরে, আমরা `document.title` ব্রাউজার API ব্যবহার করে ডকুমেন্ট শিরোনাম সেট করি। আমরা effects এর মধ্যে সর্বশেষতম `count` পড়তে পারি কারণ এটি আমাদের ফাংশনের সুযোগে। যখন Reactটি আমাদের উপাদানটিকে রেন্ডার করে, এটি আমাদের ব্যবহৃত প্রভাবটি স্মরণ করবে এবং তারপরে DOM আপডেট করার পরে আমাদের প্রভাব চালাবে। এটি প্রথম রেন্ডার সহ প্রতিটি রেন্ডারের ক্ষেত্রে ঘটে।

অভিজ্ঞ জাভাস্ক্রিপ্ট বিকাশকারীরা লক্ষ্য করতে পারেন যে `UseEffect` এ ফাংশনটি প্রতিটি রেন্ডারে আলাদা হতে চলেছে। এটা ইচ্ছাকৃত। প্রকৃতপক্ষে, এটি আমাদের বাসী হয়ে যাওয়ার চিন্তা না করে প্রভাবের ভিতরে থেকে `count` মানটি পড়তে দেয়। যতবার আমরা আবার রেন্ডার করি, আমরা একটি _আলাদা_ effects শিডিউল করি, আগেরটি প্রতিস্থাপন করে। একটি উপায়ে, এটি effects রেন্ডার ফলাফলের অংশের মতো আচরণ করে - প্রতিটি effects একটি নির্দিষ্ট রেন্ডারের সাথে "সম্পর্কিত"। কেন এটি কার্যকর তা আমরা আরও স্পষ্ট দেখতে পাব [পরে এই পৃষ্ঠায়](#explanation-why-effects-run-on-each-update).

>পরামর্শ
>
> `componentDidMount` বা `componentDidUpdate`-এর বিপরীতে, `UseEffect` এর সাথে নির্ধারিত প্রভাবগুলি ব্রাউজারটিকে স্ক্রিনটি আপডেট করতে আটকাবে না। এটি আপনার অ্যাপ্লিকেশনটিকে আরও Reactশীল মনে করে। বেশিরভাগ প্রভাবগুলির সিঙ্ক্রোনালি হওয়ার দরকার নেই। অস্বাভাবিক ক্ষেত্রে যেখানে তারা করেন (যেমন লেআউটটি পরিমাপ করা), সেখানে আলাদা রয়েছে [`useLayoutEffect`](/docs/hooks-reference.html#uselayouteffect) `UseEffect` এর অনুরূপ একটি API সহ হুক`

## Effects with Cleanup {#effects-with-cleanup}

এর আগে, আমরা কীভাবে এমন কোনও পার্শ্ব React প্রকাশ করতে দেখলাম যাতে কোনও পরিষ্কারের প্রয়োজন নেই। যাইহোক, কিছু effect আছে। উদাহরণস্বরূপ, **আমরা কোনও বাহ্যিক ডেটা উত্সে একটি সাবস্ক্রিপশন সেট করতে চাই**। সেক্ষেত্রে পরিষ্কার করা জরুরী যাতে আমরা কোনও মেমরি ফুটো প্রবর্তন করি না! আসুন আমরা কীভাবে ক্লাস এবং হুকসের সাথে এটি করতে পারি তা তুলনা করি।

### ক্লাস ব্যবহার করে উদাহরণ {#example-using-classes-1}

একটি React শ্রেণিতে, আপনি সাধারণত `componentDidMount` একটি সাবস্ক্রিপশন সেট আপ করতে এবং এটি `componentWillUnmount` সাফ করুন উদাহরণস্বরূপ, আসুন আমরা বলি যে আমাদের একটি `ChatAPI` মডিউল রয়েছে যা আমাদের বন্ধুর অনলাইন স্ট্যাটাস সাবস্ক্রাইব করতে দেয়। ক্লাস ব্যবহার করে আমরা কীভাবে সেই স্থিতিটি সাবস্ক্রাইব এবং প্রদর্শন করতে পারি তা এখানে:

```js{8-26}
class FriendStatus extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }

  render() {
    if (this.state.isOnline === null) {
      return 'Loading...';
    }
    return this.state.isOnline ? 'Online' : 'Offline';
  }
}
```

লক্ষ্য করুন কীভাবে কীভাবে `componentDidMount` এবং `componentWillUnmount` একে অপরকে mirror করা দরকার। উভয় ক্ষেত্রে ধারণাগত কোড একই effects এর সাথে সম্পর্কিত হলেও লাইফাইসাইকেল পদ্ধতিগুলি আমাদের এই যুক্তিকে বিভক্ত করতে বাধ্য করে।

>মন্তব্য
>
>ঈগল চোখের পাঠকরা লক্ষ্য করতে পারেন যে এই উদাহরণটির পুরোপুরি সঠিক হওয়ার জন্য একটি `componentDidUpdate` পদ্ধতিও প্রয়োজন। আমরা আপাতত এটিকে উপেক্ষা করব তবে এটি [পরবর্তী বিভাগে](#explanation-why-effects-run-on-each-update) ফিরে আসব এই পৃষ্ঠার।

### হুক ব্যবহার করার উদাহরণ {#example-using-hooks-1}

আসুন দেখি কীভাবে আমরা হুকস সহ এই উপাদানটি লিখতে পারি।

আপনি হয়ত ভাবছেন যে ক্লিনআপটি সম্পাদন করার জন্য আমাদের একটি পৃথক effect প্রয়োজন। তবে সাবস্ক্রিপশন যুক্ত এবং অপসারণের কোডটি এতটা দৃঢ়ভাবে সম্পর্কিত যে `useEffect` এটিকে একসাথে রাখার জন্য ডিজাইন করা হয়েছে। যদি আপনার effect কোনও ফাংশন দেয়, Reactটি পরিষ্কার করার সময় হলে এটি চালিত হবে:

```js{6-16}
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // Specify how to clean up after this effect:
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

**আমরা কেন আমাদের effects থেকে কোনও ফাংশনে ফিরলাম?** এটি effects এর জন্য ঐচ্ছিক ক্লিনআপ প্রক্রিয়া। প্রতিটি effects এমন কোনও ফাংশন ফিরিয়ে দিতে পারে যা এর পরে পরিষ্কার হয়ে যায়। এটি আমাদের সাবস্ক্রিপশন একে অপরের নিকটে যুক্ত এবং অপসারণের জন্য যুক্তি রাখতে দেয়। তারা একই প্রভাব অংশ!

**React কখন একটি effects পরিষ্কার করে?** উপাদানটি আনমাউন্ট করার সময় React পরিচ্ছন্নতা সম্পাদন করে। তবে, যেমনটি আমরা আগে জেনেছি, effects প্রতিটি রেন্ডারের জন্য চালানো হয় এবং একবারে নয়। এই কারণেই React *পরবর্তী* প্রভাবগুলি পরবর্তী সময় চালানোর আগে পূর্ববর্তী রেন্ডার থেকে effects পরিষ্কার করে। আমরা আলোচনা করব [কেন এটি বাগগুলি এড়াতে সহায়তা করে](#explanation-why-effects-run-on-each-update) এবং [এটি কার্য সম্পাদনের সমস্যা তৈরির ক্ষেত্রে কীভাবে এই আচরণ থেকে বেরিয়ে যায়](#tip-optimizing-performance-by-skipping-effects) পরে নীচে।

>মন্তব্য
>
>effect থেকে আমাদের কোনও নামকৃত ফাংশন ফিরিয়ে দিতে হবে না। এর উদ্দেশ্যটি স্পষ্ট করার জন্য আমরা এটিকে এখানে `cleanup` বলেছি, তবে আপনি কোনও তীর ফাংশনটি ফিরে আসতে পারেন বা এটিকে আলাদা কিছু বলতে পারেন।

## সংক্ষিপ্তবৃত্তি {#recap}

আমরা শিখেছি যে `useEffect` আমাদের একটি উপাদান সরবরাহের পরে বিভিন্ন ধরণের পার্শ্ব React প্রকাশ করতে দেয়। কিছু প্রভাব পরিষ্কার করার প্রয়োজন হতে পারে যাতে তারা কোনও ফাংশন ফেরত দেয়:

```js
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
```

অন্যান্য effects এর কোনও ক্লিনআপ পর্ব নাও থাকতে পারে এবং কিছুই ফেরত দেয় না।

```js
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
```

effect হুক উভয়ই একক API সহ কেস ব্যবহার করে।

-------------

**আপনার যদি মনে হয় যে আপনার কীভাবে প্রভাব হুক কাজ করে বা আপনার যদি অভিভূত বোধ হয় সে সম্পর্কে আপনার কাছে যথাযথ উপলব্ধি রয়েছে তবে আপনি এখানে যেতে পারেন [হুকসের বিধি সম্পর্কে পরবর্তী পৃষ্ঠা](/docs/hooks-rules.html) এখন।**

-------------

## effects ব্যবহার করার জন্য টিপস {#tips-for-using-effects}

অভিজ্ঞ Reactর ব্যবহারকারীরা সম্ভবত কৌতূহল বোধ করবেন এমন `UseEffect` - এর কিছু দিকগুলি গভীরতার সাথে আমরা এই পৃষ্ঠাটি চালিয়ে যাব। এখনই এগুলিতে খননের বাধ্যবাধকতা বোধ করবেন না। এফেক্ট হুক সম্পর্কে আরও বিস্তারিত জানতে আপনি এই পৃষ্ঠায় সর্বদা ফিরে আসতে পারেন।

### টিপ: উদ্বেগ আলাদা করতে একাধিক effects ব্যবহার করুন {#tip-use-multiple-effects-to-separate-concerns}

সমস্যার মধ্যে একটিরূপে আমরা রূপরেখা করি [প্রেরণা](/docs/hooks-intro.html#complex-components-become-hard-to-understand) হুকস জন্য হল ক্লাস লাইফ সাইকেল পদ্ধতিগুলিতে প্রায়শই সম্পর্কিত নয় এমন যুক্তি থাকে তবে সম্পর্কিত যুক্তি বিভিন্ন পদ্ধতিতে বিভক্ত হয়ে যায়। এখানে এমন একটি উপাদান যা পূর্ববর্তী উদাহরণগুলি থেকে কাউন্টার এবং বন্ধুর স্ট্যাটাস সূচক যুক্তিকে সংযুক্ত করে:

```js
class FriendStatusWithCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0, isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }
  // ...
```

`document.title` সেট করে এমন যুক্তিটি কীভাবে `componentDidMount` এবং `componentDidUpdate` এর মধ্যে বিভক্ত হবে তা নোট করুন। সাবস্ক্রিপশন লজিকটি `componentDidMount` এবং `componentWillUnmount` মধ্যেও ছড়িয়ে পড়ে এবং `componentDidMount` উভয় কাজের জন্য কোড ধারণ করে।

সুতরাং, হুকস কীভাবে এই সমস্যার সমাধান করতে পারে? ঠিক যেমন [আপনি একাধিকবার *state* হুক ব্যবহার করতে পারেন](/docs/hooks-state.html#tip-using-multiple-state-variables), আপনি বেশ কয়েকটি effects ব্যবহার করতে পারেন। এটি আমাদের সম্পর্কিত সম্পর্কগুলিকে বিভিন্ন effects পৃথক করতে দেয়:

```js{3,8}
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
  // ...
}
```

**হুকস আমাদের কী করছে তার উপর ভিত্তি করে কোডটি বিভক্ত করা যাক** পরিবর্তে একটি জীবনচক্র পদ্ধতির নাম। Reactটি উপাদান দ্বারা ব্যবহৃত *প্রতিটি* প্রভাব প্রয়োগ করা হবে, তারা নির্দিষ্ট করা ক্রমে।

### ব্যাখ্যা: প্রতিটি আপডেটে কেন প্রভাব চলতে পারে {#explanation-why-effects-run-on-each-update}

আপনি যদি ক্লাসে অভ্যস্ত হন, আপনি ভাবতে পারেন যে প্রতি রি-রেন্ডারের পরে কেন প্রভাব পরিষ্কারের ধাপ হয়, এবং আনমাউন্ট করার সময় একবার নয়। এই ডিজাইনটি কেন কম বাগ সহ উপাদান তৈরি করতে আমাদের সহায়তা করে তা দেখতে একটি ব্যবহারিক উদাহরণ দেখুন।

[আগে এই পৃষ্ঠায়](#example-using-classes-1), আমরা একটি উদাহরণ `FriendStatus` উপাদানটি উপস্থাপন করেছি যা দেখায় যে কোনও বন্ধু অনলাইনে আছে কিনা। আমাদের ক্লাসটি `this.prop` থেকে `friend.id` পড়ে, উপাদানটি মাউন্ট করার পরে বন্ধুত্বের স্ট্যাটাস সাবস্ক্রাইব করে এবং আনমাউন্ট করার সময় সদস্যতা গ্রহণ করে:

```js
  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
```

**তবে `friend` প্রপ পরিবর্তন হলে কী হয়** কম্পোনেন্টটি স্ক্রিনে থাকা অবস্থায়? আমাদের উপাদানটি অন্য কোনও বন্ধুর অনলাইন স্ট্যাটাস প্রদর্শন করা চালিয়ে যাবে। এটি একটি বাগ। আনসাবস্ক্রাইব করা কলটি যদি ভুল বন্ধু আইডি ব্যবহার করে তবে আনমাউন্ট করার সময় আমরা মেমরি ফাঁস বা ক্রাশও করতে পারি।

শ্রেণীর কম্পোনেন্টকে, এই কেসটি হ্যান্ডেল করতে আমাদের `componentDidUpdate` যুক্ত করতে হবে:

```js{8-19}
  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate(prevProps) {
    // Unsubscribe from the previous friend.id
    ChatAPI.unsubscribeFromFriendStatus(
      prevProps.friend.id,
      this.handleStatusChange
    );
    // Subscribe to the next friend.id
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
```

`componentDidUpdate` সঠিকভাবে পরিচালনা করতে ভুলে যাওয়া রিয়েক্ট অ্যাপ্লিকেশনগুলিতে বাগগুলির একটি সাধারণ উত্স।

এখন এই উপাদানটির সংস্করণটি বিবেচনা করুন যা হুকগুলি ব্যবহার করে:

```js
function FriendStatus(props) {
  // ...
  useEffect(() => {
    // ...
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
```

এটি এই বাগ থেকে ভোগেনা। (তবে আমরা এতে কোনও পরিবর্তনও করি নি))

আপডেটগুলি পরিচালনা করার জন্য কোনও বিশেষ কোড নেই কারণ `useEffect` এগুলি *ডিফল্টরূপে* পরিচালনা করে। এটি পরবর্তী প্রভাবগুলি প্রয়োগ করার আগে পূর্ববর্তী প্রভাবগুলি পরিষ্কার করে। এটি চিত্রিত করার জন্য, এখানে সাবস্ক্রাইব এবং সাবস্ক্রাইব করা কলগুলির ক্রম রয়েছে যা এই উপাদানটি সময়ের সাথে সাথে উত্পাদন করতে পারে:

```js
// Mount with { friend: { id: 100 } } props
ChatAPI.subscribeToFriendStatus(100, handleStatusChange);     // Run first effect

// Update with { friend: { id: 200 } } props
ChatAPI.unsubscribeFromFriendStatus(100, handleStatusChange); // Clean up previous effect
ChatAPI.subscribeToFriendStatus(200, handleStatusChange);     // Run next effect

// Update with { friend: { id: 300 } } props
ChatAPI.unsubscribeFromFriendStatus(200, handleStatusChange); // Clean up previous effect
ChatAPI.subscribeToFriendStatus(300, handleStatusChange);     // Run next effect

// Unmount
ChatAPI.unsubscribeFromFriendStatus(300, handleStatusChange); // Clean up last effect
```

এই আচরণটি ডিফল্টরূপে ধারাবাহিকতা নিশ্চিত করে এবং আপডেট যুক্তি অনুপস্থিতির কারণে শ্রেণি কম্পোনেন্টকে প্রচলিত বাগগুলি প্রতিরোধ করে।

### টিপ: স্কিপিং effects দ্বারা পারফরম্যান্স অনুকূলকরণ {#tip-optimizing-performance-by-skipping-effects}

কিছু ক্ষেত্রে, পরিষ্কার করা বা প্রতিটি রেন্ডার পরে প্রভাব প্রয়োগ করা একটি পারফরম্যান্স সমস্যা তৈরি করতে পারে। শ্রেণীর কম্পোনেন্টকে, আমরা `prevProps` বা `prevDate` এর ভিতরে `componentDidUpdate` এর সাথে অতিরিক্ত তুলনা লিখে এটি সমাধান করতে পারি:

```js
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```

এই প্রয়োজনীয়তাটি যথেষ্ট সাধারণ যে এটি `useEffect` হুক API এ অন্তর্নির্মিত। আপনি Reactটিকে *এড়িয়ে যান* বলতে পারেন কোনও প্রভাব প্রয়োগ করে যদি নির্দিষ্ট রেন্ডারগুলির মধ্যে কিছু মান পরিবর্তন না হয়। এটি করতে, `UseEffect`- এ ঐচ্ছিক দ্বিতীয় আর্গুমেন্ট হিসাবে একটি অ্যারে পাস করুন:

```js{3}
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // Only re-run the effect if count changes
```

উপরের উদাহরণে, আমরা দ্বিতীয় আর্গুমেন্ট হিসাবে `[count]` pass পাস করি। এটার মানে কি? যদি `count` `5` হয় এবং তারপরে আমাদের উপাদানটি এখনও `count` সমান `5` এর সাথে পুনরায় রেন্ডার করে, React পূর্ববর্তী রেন্ডার থেকে `[5]` এবং পরবর্তী রেন্ডার থেকে `[5]` তুলনা করবে । অ্যারেতে থাকা সমস্ত আইটেম সমান (`5 === 5`), Reactটি effects এড়িয়ে যাবে। এটাই আমাদের অপ্টিমাইজেশন।

যখন আমরা `count` কে আপডেট করে `6` এ আপডেট করি তখন React পূর্ববর্তী রেন্ডার থেকে `[5]` অ্যারেতে থাকা আইটেমগুলির সাথে পরবর্তী রেন্ডার থেকে `[6]` অ্যারে আইটেমগুলির সাথে তুলনা করবে। এবার `5! == 6` এর কারণে Reactটি পুনরায় প্রয়োগ করবে অ্যারেতে যদি একাধিক আইটেম থাকে তবে Reactগুলি কেবলমাত্র তার মধ্যে একটি আলাদা হলেও কার্যকরভাবে পুনরায় চালিত হবে।

এটি ক্লিনআপ পর্বের effects এর জন্যও কাজ করে:

```js{10}
useEffect(() => {
  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
  };
}, [props.friend.id]); // Only re-subscribe if props.friend.id changes
```

ভবিষ্যতে, দ্বিতীয় যুক্তি একটি বিল্ড-টাইম রূপান্তর দ্বারা স্বয়ংক্রিয়ভাবে যুক্ত হতে পারে।

>মন্তব্য
>
>আপনি যদি এই অপটিমাইজেশন ব্যবহার করেন তবে অ্যারে অন্তর্ভুক্ত রয়েছে তা নিশ্চিত করুন **উপাদান ব্যবস্থার সমস্ত মান (যেমন props এবং state) যা সময়ের সাথে সাথে পরিবর্তিত হয় এবং যা প্রভাব দ্বারা ব্যবহৃত হয়**. অন্যথায়, আপনার কোড পূর্ববর্তী রেন্ডারগুলির থেকে বাসি মানগুলি উল্লেখ করবে। এই সম্পর্কে আরও জানো [কিভাবে কার্যাদি মোকাবেলা করতে](/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies) এবং [অ্যারে খুব ঘন ঘন পরিবর্তিত হলে কী করবেন](/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often).
>
>আপনি যদি কোনও প্রভাব চালাতে চান এবং এটি একবারে পরিষ্কার করতে চান (মাউন্ট এবং আনমাউন্টে), আপনি একটি খালি অ্যারে (`[]`) দ্বিতীয় আর্গুমেন্ট হিসাবে পাস করতে পারেন। এটি React জানায় যে আপনার effect props বা state *কোনও* মানের উপর নির্ভর করে না, সুতরাং এটি পুনরায় চালানোর প্রয়োজন হয় না। এটি কোনও বিশেষ কেস হিসাবে পরিচালিত হয় না - নির্ভরতা অ্যারে সর্বদা কীভাবে কাজ করে তা সরাসরি তা অনুসরণ করে।
>
>আপনি যদি খালি অ্যারে (`[]`) পাস করেন তবে প্রপসের ভিতরে থাকা props এবং state সর্বদা তাদের প্রাথমিক মান থাকবে। আর্গুমেন্ট `[]` পাশ করার সময় দ্বিতীয় আর্গুমেন্টটি পরিচিত `componentDidMount` এবং `componentWillUnmount` মানসিক মডেলের কাছাকাছি থাকায় সাধারণত রয়েছে [উত্তম](/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies) [সমাধান](/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often) পুনরায় চলমান effects প্রায়শই এড়াতে। এছাড়াও, ভুলে যাবেন না যে ব্রাউজারটি আঁকার পরে React `UseEffect` চলমান স্থগিত করে, সুতরাং অতিরিক্ত কাজ করা কোনও সমস্যা কম।
>
>আমরা ব্যবহার করার পরামর্শ দিচ্ছি [`exhaustive-deps`](https://github.com/facebook/react/issues/14920) আমাদের অংশ হিসাবে নিয়ম [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation) প্যাকেজ। নির্ভরতাগুলি ভুলভাবে নির্দিষ্ট করা হয় এবং একটি সমাধানের পরামর্শ দেয় এটি সতর্ক করে।

## পরবর্তী পদক্ষেপ {#next-steps}

অভিনন্দন! এটি একটি দীর্ঘ পৃষ্ঠা ছিল, তবে আশা করি শেষ পর্যন্ত আপনার effects সম্পর্কে বেশিরভাগ প্রশ্নের উত্তর দেওয়া হয়েছিল। আপনি state হুক এবং effect হুক উভয়ই শিখে ফেলেছেন, এবং উভয়কে সম্মিলিত করে আপনি করতে পারেন এমন একটি *অনেকগুলি* রয়েছে। তারা ক্লাসগুলির জন্য বেশিরভাগ ব্যবহারের ক্ষেত্রে কভার করে - এবং যেখানে তারা না দেয় আপনি সম্ভবত এটি সন্ধান করতে পারেন [অতিরিক্ত হুকস](/docs/hooks-reference.html) সহায়ক।

হুকস কীভাবে [প্রেরণা](/docs/hooks-intro.html#motivation) এর মধ্যে বর্ণিত সমস্যাগুলি সমাধান করে তাও আমরা দেখতে শুরু করেছি। আমরা দেখেছি কীভাবে কার্যকরভাবে ক্লিনআপ `componentDidUpdate` এবং `componentWillUnmount` সদৃশতা এড়ানো যায়, সম্পর্কিত কোডকে একসাথে নিয়ে আসে এবং বাগগুলি এড়াতে আমাদের সহায়তা করে। আমরা তাদের লক্ষ্য দ্বারা কীভাবে প্রভাবগুলি পৃথক করতে পারি তাও আমরা দেখেছি, যা এমন কিছু যা আমরা ক্লাসে মোটেই করতে পারিনি।

এই মুহুর্তে আপনি প্রশ্ন করছেন যে হুকস কীভাবে কাজ করে। কোন `useState` কলটি পুনরায় রেন্ডারগুলির মধ্যে কোন state ভেরিয়েবলের সাথে সম্পর্কিত তা কীভাবে React জানবে? প্রতিটি আপডেটে পূর্ববর্তী এবং পরবর্তী effects কীভাবে React হয় "মিলিত হয়"? **পরবর্তী পৃষ্ঠায় আমরা শিখব [হুকসের বিধি](/docs/hooks-rules.html) -- তারা হুকসকে কাজ করার জন্য প্রয়োজনীয়।**
