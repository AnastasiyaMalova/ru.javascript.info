# Свойства формы и методы

Формы и элементы управления, такие как `<input>` имеют большое количество специальных свойств и событий.

Работа с формами может быть более удобной, если мы их знаем.

## Навигация: формы и элементы

Document forms принадлежат специальной коллекции `document.forms`.

Это именованная коллекция: мы можем использовать имя или номер, чтобы обратиться к форме.

```js no-beautify
document.forms.my - форма с именем "my"
document.forms[0] - первая форма в документе
```

Когда у нас есть форма, любой элемент доступен в именованной коллекции `form.elements`.

Например:

```html run height=40
<form name="my">
  <input name="one" value="1">
  <input name="two" value="2">
</form>

<script>
  // объявляем форму
  let form = document.forms.my; // <form name="my"> element

  // объявляем элемент
  let elem = form.elements.one; // <input name="one"> element

  alert(elem.value); // 1
</script>
```

Может быть множество элементов с одинаковым именем, There may be multiple elements with the same name, что часто в случае с переключателями radio.

В таком случае `form.elements[name]` - коллекция, например:

```html run height=40
<form>
  <input type="radio" *!*name="age"*/!* value="10">
  <input type="radio" *!*name="age"*/!* value="20">
</form>

<script>
let form = document.forms[0];

let ageElems = form.elements.age;

alert(ageElems[0].value); // 10, the first input value
</script>
```

Эти свойства навигации не зависят от структуры тегов. Все элементы, не важно, как глубоко они лежат в форме, доступны через`form.elements`.


````smart header="Fieldsets как \"subforms\""
Форма может содержать один или несколько `<fieldset>` элементов внутри неё. У них также есть своство `elements`.

Например:

```html run height=80
<body>
  <form id="form">
    <fieldset name="userFields">
      <legend>info</legend>
      <input name="login" type="text">
    </fieldset>
  </form>

  <script>
    alert(form.elements.login); // <input name="login">

*!*
    let fieldset = form.elements.userFields;
    alert(fieldset); // HTMLFieldSetElement

    // мы может получить input как от элементов <form>, так и от <fieldset>:
    alert(fieldset.elements.login == form.elements.login); // true
*/!*
  </script>
</body>
```
````

````warn header="Короткое замечание: `form.name`"
Короткое замечание: мы можем получить доступ к элементу через `form[index/name]`.

Вместо `form.elements.login` мы можем написать `form.login`.

Это работает так же, но есть небольшая проблема: если мы обратимся к элементу, и потом поменяем его свойство `name`, он по-пережнему будет доступен под старым именем (так же, как и под новым).

Это легко увидеть на примере:

```html run height=40
<form id="form">
  <input name="login">
</form>

<script>
  alert(form.elements.login == form.login); // true, тот же самый <input>

  form.login.name = "username"; // меняем свойство name у <input>

  // form.elements обновили name:
  alert(form.elements.login); // undefined, элемента с таким именем больше нет
  alert(form.elements.username); // input

*!*
  // с прямым доступом мы можем применять оба имени: как новое, так и старое:
  alert(form.username == form.login); // true
*/!*
</script>
```

Обычно это не является проблемой, потому что мы не часто меняем имена элементов формы.

````

## Ссылка на форму element.form

For any element, the form is available as `element.form`. So a form references all elements, and elements
reference the form.

Here's the picture:

![](form-navigation.png)

For instance:

```html run height=40
<form id="form">
  <input type="text" name="login">
</form>

<script>
*!*
  // form -> element
  let login = form.login;

  // element -> form
  alert(login.form); // HTMLFormElement
*/!*
</script>
```

## Form elements

Let's talk about form controls, pay attention to their specific features.

### input and textarea

Normally, we can access the value as `input.value` or `input.checked` for checkboxes.

Like this:

```js
input.value = "New value";
textarea.value = "New text";

input.checked = true; // for a checkbox or radio button
```

```warn header="Use `textarea.value`, not `textarea.innerHTML`"
Please note that we should never use `textarea.innerHTML`: it stores only the HTML that was initially on the page, not the current value.
```

### select and option

A `<select>` element has 3 important properties:

1. `select.options` -- the collection of `<option>` elements,
2. `select.value` -- the value of the chosen option,
3. `select.selectedIndex` -- the number of the selected option.

So we have three ways to set the value of a `<select>`:

1. Find the needed `<option>` and set `option.selected` to `true`.
2. Set `select.value` to the value.
3. Set `select.selectedIndex` to the number of the option.

The first way is the most obvious, but `(2)` and `(3)` are usually more convenient.

Here is an example:

```html run
<select id="select">
  <option value="apple">Apple</option>
  <option value="pear">Pear</option>
  <option value="banana">Banana</option>
</select>

<script>
  // all three lines do the same thing
  select.options[2].selected = true;
  select.selectedIndex = 2;
  select.value = 'banana';
</script>
```

Unlike most other controls, `<select multiple>` allows multiple choice. In that case we need to walk over `select.options` to get all selected values.

Like this:

```html run
<select id="select" *!*multiple*/!*>
  <option value="blues" selected>Blues</option>
  <option value="rock" selected>Rock</option>
  <option value="classic">Classic</option>
</select>

<script>
  // get all selected values from multi-select
  let selected = Array.from(select.options)
    .filter(option => option.selected)
    .map(option => option.value);

  alert(selected); // blues,rock  
</script>
```

The full specification of the `<select>` element is available at <https://html.spec.whatwg.org/multipage/forms.html#the-select-element>.

### new Option

In the specification of [the option element](https://html.spec.whatwg.org/multipage/forms.html#the-option-element) there's a nice short syntax to create `<option>` elements:

```js
option = new Option(text, value, defaultSelected, selected);
```

Parameters:

- `text` -- the text inside the option,
- `value` -- the option value,
- `defaultSelected` -- if `true`, then `selected` attribute is created,
- `selected` -- if `true`, then the option is selected.

For instance:

```js
let option = new Option("Text", "value");
// creates <option value="value">Text</option>
```

The same element selected:

```js
let option = new Option("Text", "value", true, true);
```

```smart header="Additional properties of `<option>`"
Option elements have additional properties:

`selected`
: Is the option selected.

`index`
: The number of the option among the others in its `<select>`.

`text`
: Text content of the option (seen by the visitor).
```

## Summary

Form navigation:

`document.forms`
: A form is available as `document.forms[name/index]`.

`form.elements`  
: Form elements are available as `form.elements[name/index]`, or can use just `form[name/index]`. The `elements` property also works for `<fieldset>`.

`element.form`
: Elements reference their form in the `form` property.

Value is available as `input.value`, `textarea.value`, `select.value` etc, or `input.checked` for checkboxes and radio buttons.

For `<select>` we can also get the value by the index `select.selectedIndex` or through the options collection `select.options`. The full specification of this and other elements is at <https://html.spec.whatwg.org/multipage/forms.html>.

These are the basics to start working with forms. In the next chapter we'll cover `focus` and `blur` events that may occur on any element, but are mostly handled on forms.
