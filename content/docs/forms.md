---
id: forms
title: Forms
permalink: docs/forms.html
prev: lists-and-keys.html
next: lifting-state-up.html
redirect_from:
  - "tips/controlled-input-null-value.html"
  - "docs/forms-zh-CN.html"
---
এখানে HTML ফর্ম এলিমেন্টসগুলি  React এর   অন্যান্য DOM এলিমেন্টস থেকে একটু ভিন্নভাবে কাজ করে, কারণ Form এলিমেন্টস  স্বাভাবিকভাবেই কিছু internal state ধারণ করে। উদাহরণস্বরূপ, এই  HTML ফর্মটি একটি single name গ্রহণ করে:

```html
<form>
  <label>
    Name:
    <input type="text" name="name" />
  </label>
  <input type="submit" value="Submit" />
</form>
```

এই ফর্মটির default HTML আচরণ হচ্ছে ইউজারের ফর্ম সাবমিট করার সময় নতুন পৃষ্ঠায় ব্রাউজ করা। রিয়েক্টে এই বিহ্যাভিয়ারটি সহজভাবে কাজ করে। তবে বেশিরভাগ ক্ষেত্রে, ফর্মে  ইউজারের দ্বারা ইনপুট করা ডেটা অ্যাক্সেস করার জন্য জাভাস্ক্রিপ্ট ফাংশন নেওয়া সুবিধাজনক । এটি পাওয়ার জন্য প্রমাণিত পদ্ধতি হল "কন্ট্রোলড কম্পোনেন্টস" নামক একটি টেকনিক।


## Controlled Components {#controlled-components}

HTML-তে, `<input>`, `<textarea>`, এবং `<select>` এর মতো ফর্ম উপাদানগুলি সাধারণতঃ নিজস্ব state ধারন করে এবং ইউজারের ইনপুটের উপর ভিত্তি করে এটি আপডেট করে। React-এ, কম্পনেন্টের নিজের state সাধারণতঃ কম্পোনেন্টগুলির state প্রপার্টিতে সংরক্ষিত থাকে, এবং শুধুমাত্র [`setState()`](/docs/react-component.html#setstate).-এর মাধ্যমে আপডেট করা হয়।

আমরা এই দুটি সংযোজন করতে পারি রিয়েক্ট স্টেটকে "single source of truth" বানিয়ে। তারপরে রিয়েক্ট কম্পোনেন্টটি ফর্ম কে রেন্ডার করে সেই ফর্মে পরবর্তী ইউজার ইনপুটে কী ঘটছে তা নিয়ন্ত্রণ করে। এইভাবে রিয়েক্ট দ্বারা নিয়ন্ত্রিত হয়ে এমন একটি ইনপুট ফর্ম উপাদানকে "controlled component" বলা হয়।

উদাহরণস্বরূপ, যদি আমরা পূর্ববর্তী উদাহরণের মতোই নামটি সাবমিট দিয়ে লগ করতে চাই, তবে আমরা ফর্মটি একটি কন্ট্রোলড কম্পোনেন্ট হিসাবে লিখতে পারি এভাবে:

```javascript{4,10-12,21,24}
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/VmmPgp?editors=0010)

value এট্রিবিউটটি আমাদের ফর্ম element এ সেট করা হয়েছে, এবং প্রদর্শিত মান সর্বদা this.state.value হবে, এটি রিয়েক্ট স্টেটকে source of truth বানিয়ে দেয় । handleChange ফাংশনটি প্রতিটি keystroke আপডাট করার সময় চালু হয় এবং রিয়েক্ট স্টেট আপডেট করে, এবং ইউজার টাইপ করার সাথে সাথে প্রদর্শিত মানও আপডেট হয়।

একটি কন্ট্রোলড কম্পোনেন্টে, ইনপুটের মান সর্বদা React State দ্বারা নির্ধারিত হয়। এর মানে কিছু বেশি কোড লিখতে হবে, কিন্তু এখন আপনি ইনপুট ভ্যালু অন্যান্য UI এলিমেন্টসের জন্যও পাঠাতে পারবেন, বা অন্য ইভেন্ট হ্যান্ডলার থেকে রিসেট করতে পারবেন।

## The textarea Tag {#the-textarea-tag}

HTML-এ, `<textarea>` এলিমেন্ট টেক্সটটি নিজের চাইল্ডের মাধ্যমে নির্ধারণ করে।

```html
<textarea>
  Hello there, this is some text in a text area
</textarea>
```

React-এ, `<textarea>` এর পরিবর্তে value এট্রিবিউট ব্যবহার করে। এই ভাবে, `<textarea>` ব্যবহার করা একটি ফর্মকে, একটি সিঙ্গল-লাইন ইনপুট ব্যবহার করা ফর্মের মতো লিখা যেতে পারে।


```javascript{4-6,12-14,26}
class EssayForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: 'Please write an essay about your favorite DOM element.'
    };

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('An essay was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Essay:
          <textarea value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

Notice that `this.state.value` is initialized in the constructor, so that the text area starts off with some text in it.

## The select Tag {#the-select-tag}

In HTML, `<select>` creates a drop-down list. For example, this HTML creates a drop-down list of flavors:

```html
<select>
  <option value="grapefruit">Grapefruit</option>
  <option value="lime">Lime</option>
  <option selected value="coconut">Coconut</option>
  <option value="mango">Mango</option>
</select>
```

Note that the Coconut option is initially selected, because of the `selected` attribute. React, instead of using this `selected` attribute, uses a `value` attribute on the root `select` tag. This is more convenient in a controlled component because you only need to update it in one place. For example:

```javascript{4,10-12,24}
class FlavorForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: 'coconut'};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('Your favorite flavor is: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Pick your favorite flavor:
          <select value={this.state.value} onChange={this.handleChange}>
            <option value="grapefruit">Grapefruit</option>
            <option value="lime">Lime</option>
            <option value="coconut">Coconut</option>
            <option value="mango">Mango</option>
          </select>
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/JbbEzX?editors=0010)

Overall, this makes it so that `<input type="text">`, `<textarea>`, and `<select>` all work very similarly - they all accept a `value` attribute that you can use to implement a controlled component.

> Note
>
> You can pass an array into the `value` attribute, allowing you to select multiple options in a `select` tag:
>
>```js
><select multiple={true} value={['B', 'C']}>
>```

## The file input Tag {#the-file-input-tag}

In HTML, an `<input type="file">` lets the user choose one or more files from their device storage to be uploaded to a server or manipulated by JavaScript via the [File API](https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications).

```html
<input type="file" />
```

Because its value is read-only, it is an **uncontrolled** component in React. It is discussed together with other uncontrolled components [later in the documentation](/docs/uncontrolled-components.html#the-file-input-tag).

## Handling Multiple Inputs {#handling-multiple-inputs}

When you need to handle multiple controlled `input` elements, you can add a `name` attribute to each element and let the handler function choose what to do based on the value of `event.target.name`.

For example:

```javascript{15,18,28,37}
class Reservation extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isGoing: true,
      numberOfGuests: 2
    };

    this.handleInputChange = this.handleInputChange.bind(this);
  }

  handleInputChange(event) {
    const target = event.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;

    this.setState({
      [name]: value
    });
  }

  render() {
    return (
      <form>
        <label>
          Is going:
          <input
            name="isGoing"
            type="checkbox"
            checked={this.state.isGoing}
            onChange={this.handleInputChange} />
        </label>
        <br />
        <label>
          Number of guests:
          <input
            name="numberOfGuests"
            type="number"
            value={this.state.numberOfGuests}
            onChange={this.handleInputChange} />
        </label>
      </form>
    );
  }
}
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/wgedvV?editors=0010)

Note how we used the ES6 [computed property name](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer#Computed_property_names) syntax to update the state key corresponding to the given input name:

```js{2}
this.setState({
  [name]: value
});
```

It is equivalent to this ES5 code:

```js{2}
var partialState = {};
partialState[name] = value;
this.setState(partialState);
```

Also, since `setState()` automatically [merges a partial state into the current state](/docs/state-and-lifecycle.html#state-updates-are-merged), we only needed to call it with the changed parts.

## Controlled Input Null Value {#controlled-input-null-value}

Specifying the `value` prop on a [controlled component](/docs/forms.html#controlled-components) prevents the user from changing the input unless you desire so. If you've specified a `value` but the input is still editable, you may have accidentally set `value` to `undefined` or `null`.

The following code demonstrates this. (The input is locked at first but becomes editable after a short delay.)

```javascript
ReactDOM.createRoot(mountNode).render(<input value="hi" />);

setTimeout(function() {
  ReactDOM.createRoot(mountNode).render(<input value={null} />);
}, 1000);

```

## Alternatives to Controlled Components {#alternatives-to-controlled-components}

It can sometimes be tedious to use controlled components, because you need to write an event handler for every way your data can change and pipe all of the input state through a React component. This can become particularly annoying when you are converting a preexisting codebase to React, or integrating a React application with a non-React library. In these situations, you might want to check out [uncontrolled components](/docs/uncontrolled-components.html), an alternative technique for implementing input forms.

## Fully-Fledged Solutions {#fully-fledged-solutions}

If you're looking for a complete solution including validation, keeping track of the visited fields, and handling form submission, [Formik](https://jaredpalmer.com/formik) is one of the popular choices. However, it is built on the same principles of controlled components and managing state — so don't neglect to learn them.
