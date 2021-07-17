---
id: hooks-intro
title: হুক - সূচনা
permalink: docs/hooks-intro.html
next: hooks-overview.html
---

*হুক* React 16.8 এ একটি নতুন সংযোজন। এটি ক্লাস না লিখেই state এবং Reactর আরও অন্যান্য ফিচার ব্যবহার করতে দেয়।

```js{4,5}
import React, { useState } from 'react';

function Example() {
  // "count" নামের একটি নতুন স্টেট ভ্যারিয়েবল বানানো হলো
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>{count} বার ক্লিক হয়েছে</p>
      <button onClick={() => setCount(count + 1)}>
        ক্লিক করো
      </button>
    </div>
  );
}
```

`useState` নামক এই নতুন ফাংশন দিয়েই আমরা প্রথম "হুক" বুঝবার চেষ্টা করব, তবে এই উদাহরণ অতি সাধারণ। পরবর্তী আলোচনার মাধ্যমে আমরা এটা আরো ভালো করে বুঝবো!

**হুক সন্বন্ধে বিশদে জানতে দেখুন [পরবর্তী পর্যায়](/docs/hooks-overview.html)।** এখানে আমরা হুকের প্রয়োজনীয়তা এবং তা ব্যবহার করে কিভাবে অ্যপ লেখা সম্ভব তাই নিয়ে আলোচনা করব।

>মনে রাখবেন
>
>React 16.8.0 এ হুক প্রথম বার উপলব্ধ. অতএব ব্যবহার করবার সময় অন্যান্য প্যাকেজগুলি, যেমন React DOM আপডেট করতে হবে।
>React Native এ ইতিমধ্যেই হুক উপলব্ধ [React Native এর 0.59 ভার্সন থেকে](https://reactnative.dev/blog/2019/03/12/releasing-react-native-059)।

## ভিডিও দেখুন {#video-introduction}

2018 সালে React কনফারেন্সে, সোফি অ্যালবর্ট এবং ড্যান আব্রামভ হুকের উপস্থাপনা করেন। পরবর্তীতে রায়ান ফ্লোরেন্স তার অ্যাপের মাধ্যমে হাতেকলমে হুকের প্রয়োগ করে দেখান. বিশদে দেখুন:

<br>

<iframe width="650" height="366" src="//www.youtube.com/embed/dpw9EHDh2bM" frameborder="0" allowfullscreen></iframe>

## কোনো ব্রেকিং পরিবর্তন নেই {#no-breaking-changes}

আমরা চালিয়ে যাওয়ার আগে মনে রাখবেন:

* **সম্পূর্ণরূপে ঐচ্ছিক:** আপনি কোনও বিদ্যমান কোড পুনর্লিখন না করে কিছুমাত্র কম্পনেন্টে হুকস ব্যবহার চেষ্টা করতে পারেন। আপনি না চাইলে আপনাকে এখনই হুকস শিখতে বা ব্যবহার করতে হবে না।
* **কম্প্যাটিবল:** হুকগুলিতে কোনও ব্রেকিং পরিবর্তন নেই।
* **উপলভ্যতা:** হুকগুলি এখন v16.8.0 প্রকাশের সাথে উপলব্ধ।

**React থেকে ক্লাস অপসারণ করার কোনও পরিকল্পনা নেই:** আপনি এই পৃষ্ঠার [নীচের অংশে)(#gradual-adoption-strategy) হুকের জন্য ধাপে ধাপে পরিবর্তনের কৌশল সম্পর্কে আরও পড়তে পারেন।

**হুক আপনার React ধারণাগুলির কোনো বদল করে না** বরং, হুকগুলি আপনার ইতিমধ্যে জানা React-র ধারণাগুলির আরও সহজ API সরবরাহ করে: Props, State, Context, Ref এবং LifeCycle। আমরা পরে দেখাব যে, হুকস তাদের একসাথে ব্যবহার করার জন্য একটি নতুন শক্তিশালী উপায়ও সরবরাহ করে।

**আপনি যদি কেবল হুকস শিখতে শুরু করতে চান তবে নির্দ্বিধায় [সরাসরি পরবর্তী পৃষ্ঠায় ঝাঁপুন!](/docs/hooks-overview.html)** এই পৃষ্ঠায় আমরা আলোচনা করব যে কিভাবে আমরা হুক ব্যবহারের সিদ্ধান্তে এলাম, এবং কিভাবে কোনও বিদ্যমান কোড পুনর্লিখন না করে হুকস ব্যবহার করা সম্ভব।

## অনুপ্রেরণা {#motivation}

হুকগুলি React-র এমন কিছু আপাতদৃষ্টিতে অসংলগ্ন সমস্যার সমাধান করে, যা গত পাঁচ বছর ধরে অসংখ্য component লিখে এবং তা মেইন্টেন করতে গিয়ে আমরা অনুভব করে থাকি. আপনি React-র নবিশ হন, অথবা প্রতিদিন এটি ব্যবহার করে থাকেন, বা এমনকি অন্য কোনও অনুরূপ Component-মডেলযুক্ত একটি ভিন্ন লাইব্রেরি পছন্দ করুন না কেন, আপনি এই সমস্যার কিছুটি চিনতে পারেন।

### বিভিন্ন Components-র মধ্যে stateful logic-র পুনর্ব্যবহার {#its-hard-to-reuse-stateful-logic-between-components}

React নিজে থেকে কোনও component-র সাথে পুনরায় ব্যবহারযোগ্য লজিক "সংযুক্ত" করার কোনও উপায় সরবরাহ করে না (উদাহরণস্বরূপ, কোনও স্টোরের সাথে সংযুক্ত করে)। আপনি যদি কিছুকাল যাবত React নিয়ে কাজ করে থাকেন তবে আপনি এই ধরণের নিয়মগুলির সাথে পরিচিত হতে পারেন, [render props](/docs/render-props.html) এবং [higher-order components](/docs/higher-order-components.html), যা কিনা এই যাবতীয় সমস্যা সমাধানের চেষ্টা করে। তবে এই নিদর্শনগুলির জন্য আপনার componentগুলি ব্যবহার করার সময় আপনাকে পুনর্লিখন করা দরকার যা পরিশ্রমসাধ্য হতে পারে এবং কোড অনুসরণ করা আরও শক্ত করে তোলে। আপনি যদি React DevTool-এ একটি সাধারণ React অ্যাপ্লিকেশন দেখেন, তবে আপনি সম্ভবত providers, consumers, higher-order components, render props এবং অন্যান্য abstraction-র স্তর দ্বারা বেষ্টিত Component-গুলির একটি "wrapper hell"-কে দেখতে পাবেন। যদিও আমরা এগুলি [DevTools-এ ফিল্টার](https://github.com/facebook/react-devtools/pull/503) করতেই পারি, এটি একটি গভীর অন্তর্নিহিত সমস্যার দিকে ইঙ্গিত করে: stateful logic পুনরায় ব্যবহারযোগ্য করে তোলার জন্য React-এ প্র​য়োজন উত্তমতর প্রক্রিয়া।

এমত অবস্থায় হুক ব্যবহার করে, **আমরা component থেকে stateful logic আলাদা করে নিজের মত টেস্ট ও পুনর্ব্যবহার করতে পারি এবং এর জন্য component-র শ্রেণিবিন্যাসের কোনো পরিবর্তনের প্র​য়োজন হয়না।** এর ফলে বিভিন্ন component-র মধ্যে হুক ব্যবহার করা অতি সহজ এবং অন্যান্যদের সাথে শেয়ার করা সম্ভব।

[পরবর্তীতে](/docs/hooks-custom.html) আমরা এই নিয়ে আরও অধিক আলোচনা করব.

### Complex components become hard to understand {#complex-components-become-hard-to-understand}

We've often had to maintain components that started out simple but grew into an unmanageable mess of stateful logic and side effects. Each lifecycle method often contains a mix of unrelated logic. For example, components might perform some data fetching in `componentDidMount` and `componentDidUpdate`. However, the same `componentDidMount` method might also contain some unrelated logic that sets up event listeners, with cleanup performed in `componentWillUnmount`. Mutually related code that changes together gets split apart, but completely unrelated code ends up combined in a single method. This makes it too easy to introduce bugs and inconsistencies.

In many cases it's not possible to break these components into smaller ones because the stateful logic is all over the place. It's also difficult to test them. This is one of the reasons many people prefer to combine React with a separate state management library. However, that often introduces too much abstraction, requires you to jump between different files, and makes reusing components more difficult.

To solve this, **Hooks let you split one component into smaller functions based on what pieces are related (such as setting up a subscription or fetching data)**, rather than forcing a split based on lifecycle methods. You may also opt into managing the component's local state with a reducer to make it more predictable.

We'll discuss this more in [Using the Effect Hook](/docs/hooks-effect.html#tip-use-multiple-effects-to-separate-concerns).

### Classes confuse both people and machines {#classes-confuse-both-people-and-machines}

In addition to making code reuse and code organization more difficult, we've found that classes can be a large barrier to learning React. You have to understand how `this` works in JavaScript, which is very different from how it works in most languages. You have to remember to bind the event handlers. Without unstable [syntax proposals](https://babeljs.io/docs/en/babel-plugin-transform-class-properties/), the code is very verbose. People can understand props, state, and top-down data flow perfectly well but still struggle with classes. The distinction between function and class components in React and when to use each one leads to disagreements even between experienced React developers.

Additionally, React has been out for about five years, and we want to make sure it stays relevant in the next five years. As [Svelte](https://svelte.dev/), [Angular](https://angular.io/), [Glimmer](https://glimmerjs.com/), and others show, [ahead-of-time compilation](https://en.wikipedia.org/wiki/Ahead-of-time_compilation) of components has a lot of future potential. Especially if it's not limited to templates. Recently, we've been experimenting with [component folding](https://github.com/facebook/react/issues/7323) using [Prepack](https://prepack.io/), and we've seen promising early results. However, we found that class components can encourage unintentional patterns that make these optimizations fall back to a slower path. Classes present issues for today's tools, too. For example, classes don't minify very well, and they make hot reloading flaky and unreliable. We want to present an API that makes it more likely for code to stay on the optimizable path.

To solve these problems, **Hooks let you use more of React's features without classes.** Conceptually, React components have always been closer to functions. Hooks embrace functions, but without sacrificing the practical spirit of React. Hooks provide access to imperative escape hatches and don't require you to learn complex functional or reactive programming techniques.

>Examples
>
>[Hooks at a Glance](/docs/hooks-overview.html) is a good place to start learning Hooks.

## Gradual Adoption Strategy {#gradual-adoption-strategy}

>**TLDR: There are no plans to remove classes from React.**

We know that React developers are focused on shipping products and don't have time to look into every new API that's being released. Hooks are very new, and it might be better to wait for more examples and tutorials before considering learning or adopting them.

We also understand that the bar for adding a new primitive to React is extremely high. For curious readers, we have prepared a [detailed RFC](https://github.com/reactjs/rfcs/pull/68) that dives into motivation with more details, and provides extra perspective on the specific design decisions and related prior art.

**Crucially, Hooks work side-by-side with existing code so you can adopt them gradually.** There is no rush to migrate to Hooks. We recommend avoiding any "big rewrites", especially for existing, complex class components. It takes a bit of a mindshift to start "thinking in Hooks". In our experience, it's best to practice using Hooks in new and non-critical components first, and ensure that everybody on your team feels comfortable with them. After you give Hooks a try, please feel free to [send us feedback](https://github.com/facebook/react/issues/new), positive or negative.

We intend for Hooks to cover all existing use cases for classes, but **we will keep supporting class components for the foreseeable future.** At Facebook, we have tens of thousands of components written as classes, and we have absolutely no plans to rewrite them. Instead, we are starting to use Hooks in the new code side by side with classes.

## Frequently Asked Questions {#frequently-asked-questions}

We've prepared a [Hooks FAQ page](/docs/hooks-faq.html) that answers the most common questions about Hooks.

## Next Steps {#next-steps}

By the end of this page, you should have a rough idea of what problems Hooks are solving, but many details are probably unclear. Don't worry! **Let's now go to [the next page](/docs/hooks-overview.html) where we start learning about Hooks by example.**
