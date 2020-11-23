---
id: uncontrolled-components
title: Uncontrolled Components
permalink: docs/uncontrolled-components.html
---

বেশিরভাগ ক্ষেত্রেই, ফর্ম প্রস্তুতের জন্য আমরা [controlled components] ব্যবহার করার পরামর্শ দিয়ে থাকি (/docs/forms.html#controlled-components)। একটা কন্ট্রোল্ড কম্পোনেন্টে, ফর্ম ডাটা হ্যান্ডেল করা হয় React কম্পোনেন্টের মাধ্যমে। এর বিকল্প হচ্ছে আনকন্ট্রোল্ড কম্পোনেন্ট, যেখানে DOM নিজে ফর্ম ডাটা হ্যান্ডেল করে।

একটি আনকন্ট্রোল্ড কম্পোনেন্ট লিখতে, প্রতি state আপডেটে ইভেন্ট হ্যান্ডেলার লেখার পরিবর্তে আপনি [ref](/docs/refs-and-the-dom.html) ব্যবহার করতে পারেন DOM থেকে ভ্যালু পাওয়ার জন্য।

উদাহরণ হিসেবে, এই কোডটি একটি আনকন্ট্রোল্ড কম্পোনেন্টে একটি name নেয়ঃ

```javascript{5,9,18}
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
    this.input = React.createRef();
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.input.current.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={this.input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

[**CodePen এ চালিয়ে দেখুন**](https://codepen.io/gaearon/pen/WooRWa?editors=0010)

যেহেতু একটি আনকন্ট্রোল্ড কম্পোনেন্ট DOM এর ভিতর আসল উৎসটি ধারণ করে তাই React এবং non-React কোড ইন্টিগ্র্যাট করার সময় প্রায়শই এটা সহজতর হয়। যদি আপনি দ্রুত এবং একটু অগোছালো কোড চান তাহলে একটু কম কোডেও কাজ হয়, তানাহলে কন্ট্রোল্ড কম্পোনেন্ট ব্যবহার করাই শ্রেয়।

যদি এটা এখনো পরিষ্কার না হয় যে কোন পরিস্থিতিতে কি ধরনের কম্পোনেন্ট ব্যাবহার করা উচিত তাহলে আপনার জন্য [controlled versus uncontrolled inputs](https://goshakkk.name/controlled-vs-uncontrolled-inputs-react/) এর উপর আর্টিকেলটি সহায়ক হতে পারে।

### ডিফল্ট ভ্যালু {#default-values}

<<<<<<< HEAD
React রেন্ডারিং লাইফসাইকেল এ, ফর্ম element এ `value` attribute, DOM এর ভ্যালুকে ওভাররাইড করে। একটি আনকন্ট্রোল্ড কম্পোনেন্ট এর দ্বারা আপনি React ইনিশিয়াল ভ্যালুটি নির্দিষ্ট করে রাখতে পারেন এবং পরবর্তী আপডেটগুলো আনকন্ট্রোল্ড রাখতে পারেন। এই ব্যাপারটি সামলানোর জন্য আপনি `value` এর পরিবর্তে একটি `defaultValue` নির্দিষ্ট করে দিতে পারেন।
=======
In the React rendering lifecycle, the `value` attribute on form elements will override the value in the DOM. With an uncontrolled component, you often want React to specify the initial value, but leave subsequent updates uncontrolled. To handle this case, you can specify a `defaultValue` attribute instead of `value`. Changing the value of `defaultValue` attribute after a component has mounted will not cause any update of the value in the DOM.
>>>>>>> 8f9ef00db1b36ee3e5a0e6072eb601257a6f8ccb

```javascript{7}
render() {
  return (
    <form onSubmit={this.handleSubmit}>
      <label>
        Name:
        <input
          defaultValue="Bob"
          type="text"
          ref={this.input} />
      </label>
      <input type="submit" value="Submit" />
    </form>
  );
}
```

একইভাবে `<input type="checkbox">` এবং `<input type="radio">` সাপোর্ট করে `defaultChecked`, এবং `<select>` এবং `<textarea>` সাপোর্ট করে `defaultValue`।

## ফাইল ইনপুট ট্যাগ {#the-file-input-tag}

HTML এ, একটি `<input type="file">`, ব্যবহারকারীকে তাদের ডিভাইস স্টোরেজ থেকে এক বা একাধিক ফাইল কোন সার্ভারে আপলোড করতে বা জাভাস্ক্রিপ্ট দ্বারা [File API](https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications) এর মাধ্যমে ম্যানিপুলেটেড করতে সুযোগ করে দেয়।

```html
<input type="file" />
```

React এ, একটি `<input type="file" />` সবসময় আনকন্ট্রোল্ড কম্পোনেন্ট, কারণ প্রোগ্র্যাম্যাটিক্যালি নয় বরং একজন ইউজারই পারে এর ভ্যালু সেট করতে।

আপনি ফাইলসমূহ নিয়ে কাজ করতে চাইলে File API ব্যাবহার করতে পারেন। নিচের উদাহারণটি দেখাবে কিভাবে একটি [DOM নোডের ref](/docs/refs-and-the-dom.html) প্রস্তুত করতে হয় সাবমিট হ্যান্ডেলারের ফাইলসমূহ এক্সেস করার জন্য:

`embed:uncontrolled-components/input-type-file.js`

[](codepen://uncontrolled-components/input-type-file)

