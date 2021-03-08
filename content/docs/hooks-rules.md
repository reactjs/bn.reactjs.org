---
id: hooks-rules
title: Hooks ব্যবহারের নিয়মাবলী
permalink: docs/hooks-rules.html
next: hooks-custom.html
prev: hooks-effect.html
---

*Hooks* হল React 16.8 এর নতুন সংযোজন। এটি আপনাকে ক্লাস ব্যবহার না করেই state এবং React এর অন্যান্য ফিচার ব্যবহারের সুযোগ করে দেয়।

Hooks হল জাভাস্ক্রিপ্ট ফাংশন, কিন্তু আপনাকে এটি ব্যবহারের সময় দুইটি নিয়ম মেনে চলতে হবে। আমরা একটি [linter plugin](https://www.npmjs.com/package/eslint-plugin-react-hooks) প্রদান করে থাকি যাতে স্বয়ংক্রিয়ভাবে এই নিয়মগুলো কার্যকর হয়ঃ

### শুধুমাত্র টপ লেভেলে Hooks কল করুন {#only-call-hooks-at-the-top-level}

<<<<<<< HEAD
**কোন লুপ, শর্ত বা nested ফাংশনের ভেতর Hooks কল করবেননা।** এর পরিবর্তে, সবসময় আপনার React ফাংশনের টপ লেভেলে Hooks কল করুন। এই নিয়ম মেনে চলার মাধ্যমে আপনি নিশ্চিত করেন যে, প্রতিবার রি-রেন্ডারের সময় Hooks গুলো একই ক্রমানুসারে কল করা হবে। এটি অনেকগুলো `useState` এবং `useEffect` কলের মাঝে React কে সঠিকভাবে Hooks গুলোর স্টেট সংরক্ষণ করতে সাহায্য করে। (আপনি যদি কৌতূহলী হয়ে থাকেন, আমরা [নিচে](#explanation) এটি সম্পর্কে আরও গভীর আলোচনা করব।)
=======
**Don't call Hooks inside loops, conditions, or nested functions.** Instead, always use Hooks at the top level of your React function, before any early returns. By following this rule, you ensure that Hooks are called in the same order each time a component renders. That's what allows React to correctly preserve the state of Hooks between multiple `useState` and `useEffect` calls. (If you're curious, we'll explain this in depth [below](#explanation).)
>>>>>>> 9df266413be637705d78688ffdd3697e89b102d1

### শুধুমাত্র React ফাংশনের ভেতর Hooks কল করুন {#only-call-hooks-from-react-functions}

**সাধারণ জাভাস্ক্রিপ্ট ফাংশনের ভেতর থেকে Hooks কল করবেননা।** এর পরিবর্তে আপনি:

* ✅ React ফাংশন কম্পোনেন্ট থেকে Hooks কল করুন।
* ✅ কাস্টম Hooks থেকে Hooks কল করুন (আমরা [পরের পৃষ্ঠায়](/docs/hooks-custom.html) এটি সম্পর্কে জানব)।

এই নিয়ম মেনে চলার মাধ্যমে আপনি নিশ্চিত করেন যে, একটি কম্পোনেন্টের সকল stateful logic সহজেই এর সোর্স কোড থেকে দেখা যাবে।

## ESLint Plugin {#eslint-plugin}

আমরা [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks) নামের একটি ESLint plugin প্রকাশ করেছি যা দুটি নিয়ম কার্যকর করে। আপনি এই plugin টি আপনার প্রজেক্টে সংযুক্ত করে চেষ্টা করে দেখতে পারেনঃ

এই pluginটি শুরু থেকেই [Create React App](/docs/create-a-new-react-app.html#create-react-app) এর সাথে সংযুক্ত করে দেয়া হয়েছে।

```bash
npm install eslint-plugin-react-hooks --save-dev
```

```js
// Your ESLint configuration
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error", // Checks rules of Hooks
    "react-hooks/exhaustive-deps": "warn" // Checks effect dependencies
  }
}
```

**আপনি এখন চাইলে বাকিটা বাদ দিয়ে পরের পৃষ্ঠায় [আপনার নিজের Hooks](/docs/hooks-custom.html) কিভাবে লিখবেন তা দেখে নিতে পারেন।** এই পৃষ্ঠায়, আমরা এই নিয়মগুলোর কারণ ব্যাখ্যা করব।

## ব্যাখ্যা {#explanation}

আমরা [আগেই জেনেছি](/docs/hooks-state.html#tip-using-multiple-state-variables) যে, আমরা একাধিক State অথবা Effect Hooks একটি নির্দিষ্ট কম্পোনেন্টে ব্যবহার করতে পারিঃ

```js
function Form() {
  // 1. Use the name state variable
  const [name, setName] = useState('Mary');

  // 2. Use an effect for persisting the form
  useEffect(function persistForm() {
    localStorage.setItem('formData', name);
  });

  // 3. Use the surname state variable
  const [surname, setSurname] = useState('Poppins');

  // 4. Use an effect for updating the title
  useEffect(function updateTitle() {
    document.title = name + ' ' + surname;
  });

  // ...
}
```

তাহলে React কিভাবে বুঝতে পারে কোন state এর সাথে কোন `useState` কলের সম্পর্ক রয়েছে? এর উত্তর হল **React Hooks গুলোর কলের ক্রমের উপর নির্ভর করে**। আমাদের উদাহরণ কাজ করে কারণ Hook গুলোর কলের ক্রম প্রতি রেন্ডারেই একইঃ

```js
// ------------
// First render
// ------------
useState('Mary')           // 1. Initialize the name state variable with 'Mary'
useEffect(persistForm)     // 2. Add an effect for persisting the form
useState('Poppins')        // 3. Initialize the surname state variable with 'Poppins'
useEffect(updateTitle)     // 4. Add an effect for updating the title

// -------------
// Second render
// -------------
useState('Mary')           // 1. Read the name state variable (argument is ignored)
useEffect(persistForm)     // 2. Replace the effect for persisting the form
useState('Poppins')        // 3. Read the surname state variable (argument is ignored)
useEffect(updateTitle)     // 4. Replace the effect for updating the title

// ...
```

যতক্ষণ পর্যন্ত Hook গুলো কলের ক্রম প্রতি রেন্ডারে একই থাকবে, React ততক্ষণ পর্যন্ত প্রতিটির সাথে কিছু local state যুক্ত করতে পারবে। কিন্তু তখন কি হবে যদি আমরা একটি Hook কলকে কোন শর্তের মধ্যে(উদাহরণস্বরূপ, `persistForm` effect টি) রাখি?

```js
  // 🔴 We're breaking the first rule by using a Hook in a condition
  if (name !== '') {
    useEffect(function persistForm() {
      localStorage.setItem('formData', name);
    });
  }
```

এখানে প্রথম রেন্ডারে `name !== ''` শর্তটি `true`, তাই আমরা Hook টি কল করব। কিন্তু পরবর্তি রেন্ডারে ইউজার ফর্মটি খালি করে ফেলতে পারে, এতে শর্তটি `false` হয়ে যাবে। এখন যেহেতু আমরা রেন্ডারিং এর সময় এই Hook টি বাদ দিয়ে যাই, Hook কল করার ক্রম পরিবর্তিত হয়ে যায়ঃ

```js
useState('Mary')           // 1. Read the name state variable (argument is ignored)
// useEffect(persistForm)  // 🔴 This Hook was skipped!
useState('Poppins')        // 🔴 2 (but was 3). Fail to read the surname state variable
useEffect(updateTitle)     // 🔴 3 (but was 4). Fail to replace the effect
```

React দ্বিতীয় `useState` Hook কলের জন্য কি রিটার্ন করবে বুঝতে পারবেনা। React আশা করছিল এই কম্পোনেন্টের  দ্বিতীয় Hook কল যাতে `persistForm` effect এর সাথে সঙ্গতিপূর্ণ হয়, ঠিক এর পূর্ববর্তি রেন্ডারের মত, কিন্তু এখন এরা আর সঙ্গতিপূর্ণ নয়। এর পর থেকে, আমাদের বাদ দেয়া প্রতিটি Hook কলও এক ধাপ সরে যাবে, যা পরবর্তিতে bug সৃষ্টি করবে।

**এই জন্যই Hooks গুলো সবসময় আমাদের কম্পোনেন্টের সর্বোচ্চ স্তরে কল করতে হবে।** আমরা যদি কোন effect শর্তসাপেক্ষে রান করাতে চাই, আমরা ঐ শর্ত আমাদের Hook এর *ভেতরে* রাখতে পারিঃ

```js
  useEffect(function persistForm() {
    // 👍 We're not breaking the first rule anymore
    if (name !== '') {
      localStorage.setItem('formData', name);
    }
  });
```

**উল্লেখ্য যে আপনি যদি আমাদের [দেয়া lint rule](https://www.npmjs.com/package/eslint-plugin-react-hooks) ব্যবহার করে থাকেন তাহলে আপনার এটি নিয়ে চিন্তা করার দরকার নেই।** কিন্তু আপনি এখন জানেন Hooks গুলো *কেন* এভাবে কাজ করে এবং কোন সমস্যাগুলো এই নিয়ম প্রতিরোধ করে।

## পরবর্তি ধাপসমূহ {#next-steps}

সবশেষে, আমরা আমাদের [নিজেদের Hooks লেখা](/docs/hooks-custom.html) শেখার জন্য প্রস্তুত! কাস্টম Hooks আমাদেরকে React দ্বারা সরবরাহকৃত Hooks গুলোর সাথে আমাদের নিজের abstraction এর সাথে মিলিত করে ভিন্ন ভিন্ন কম্পোনেন্টের মাঝে সাধারণ stateful logic পুনঃব্যবহার করার সুযোগ করে দেয়।
