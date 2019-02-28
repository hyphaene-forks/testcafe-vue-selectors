# testcafe-vue-selectors

This plugin provides selector extensions that make it easier to test Vue components with [TestCafe](https://github.com/DevExpress/testcafe).
These extensions allow you to test Vue component state and result markup together.

## Install

`$ npm install testcafe-vue-selectors`

## Usage

#### Create selectors for Vue components

`VueSelector` allows you to select page elements by the component tagName or the nested component tagNames.

Suppose you have the following markup.

```html
<div id="todo-app">
    <todo-input />
    <todo-list>
        <todo-item priority="High">Item 1</todo-item>
        <todo-item priority="Low">Item 2</todo-item>
    </todo-list>   
    <div className="items-count">Items count: <span>{{itemCount}}</span></div>
</div>
<script>
    Vue.component('todo-input', {...});
    Vue.component('todo-list', {...});
    Vue.component('todo-item', {...});
    
    new Vue({ 
        el:   '#todo-app',
        data: {...}
    });
</script>
```

To get the root Vue node, use the VueSelector constructor without parameters.

```js
import VueSelector from 'testcafe-vue-selectors';

const rootVue = VueSelector();
```

The `rootVue` variable will contain the `<div id="todo-app">` element.


To get a root DOM element for a component, pass the component name to the VueSelector constructor.

```js
import VueSelector from 'testcafe-vue-selectors';

const todoInput = VueSelector('todo-input');
```

To obtain a component based vue ref value, pass the ref value as second parameter to the constructor

```js
import VueSelector from 'testcafe-vue-selectors';

const todoInput = VueSelector('todo-input', 'ref-todo-input1');
```

To obtain a component within another vue component(root), pass the ref of the root vue component as third parameter to the constructor

```js
import VueSelector from 'testcafe-vue-selectors';

const todoInput = VueSelector('todo-input', false, 'todo-form-1');
```

To obtain a component based on ref within another vue component(root), pass the ref of the component as second parametere and ref of the root component as third parameter to the constructor

```js
import VueSelector from 'testcafe-vue-selectors';

const todoInput = VueSelector('todo-input', 'todo-input-1', 'todo-form-1');
```

To obtain a nested component, you can use a combined selector.
```js
import VueSelector from 'testcafe-vue-selectors';

const todoItem = VueSelector('todo-list todo-item');
```

You can combine Vue selectors with testcafe `Selector` filter functions like `.find`, `.withText`, `.nth` and [other](http://devexpress.github.io/testcafe/documentation/test-api/selecting-page-elements/selectors.html#functional-style-selectors).

```js
import VueSelector from 'testcafe-vue-selectors';

var itemsCount = VueSelector().find('.items-count span');
```

Let’s use the API described above to add a task to a Todo list and check that the number of items changed.
```js
import VueSelector from 'testcafe-vue-selectors';

fixture `TODO list test`
	.page('http://localhost:1337');

test('Add new task', async t => {
    const todoTextInput = VueSelector('todo-input');
    const todoItem      = VueSelector('todo-list todo-item');

    await t
        .typeText(todoTextInput, 'My Item')
        .pressKey('enter')
        .expect(todoItem.count).eql(3);
});
```

#### Obtaining component's props, computed, state and ref

In addition to [DOM Node State](http://devexpress.github.io/testcafe/documentation/test-api/selecting-page-elements/dom-node-state.html), you can obtain `state`, `computed`, `props` or `ref` of a Vue component.

To get these data, use the Vue selector’s `.getVue()` method.

The `getVue()` method returns a [client function](https://devexpress.github.io/testcafe/documentation/test-api/obtaining-data-from-the-client.html). This function resolves to an object that contains component properties.

```js
const vueComponent      = VueSelector('componentTag');
const vueComponentState = await vueComponent.getVue();

// >> vueComponentState
//
// {
//     props:    <component_props>,
//     state:    <component_state>,
//     computed: <component_computed>,
//     ref:      <component_ref>
// }
```

The returned client function can be passed to assertions activating the [Smart Assertion Query mechanism](https://devexpress.github.io/testcafe/documentation/test-api/assertions/#smart-assertion-query-mechanism).

Example
```js
import VueSelector from 'testcafe-vue-selector';

fixture `TODO list test`
	.page('http://localhost:1337');

test('Check list item', async t => {
    const todoItem    = VueSelector('todo-item');
    const todoItemVue = await todoItem.getVue();

    await t
        .expect(todoItemVue.props.priority).eql('High')
        .expect(todoItemVue.state.isActive).eql(false)
        .expect(todoItemVue.computed.text).eql('Item 1');
});
```

As an alternative, the `.getVue()` method can take a function that returns the required property, state or computed property. This function acts as a filter. Its argument is an object returned by `.getVue()`, i.e. `{ props: ..., state: ..., computed: ..., ref: ...}`.

```js
VueSelector('component').getVue(({ props, state, computed, ref }) => {...});
```


Example
```js
import VueSelector from 'testcafe-vue-selectors';

fixture `TODO list test`
    .page('http://localhost:1337');

test('Check list item', async t => {
    const todoItem = VueSelector('todo-item');

    await t
        .expect(todoItem.getVue(({ props }) => props.priority)).eql('High')
        .expect(todoItem.getVue(({ state }) => state.isActive)).eql(false)
        .expect(todoItem.getVue(({ computed }) => computed.text)).eql('Item 1')
        expect(todoItem.getVue(({ ref }) => ref)).eql('ref-item-1');
});

```

The `.getVue()` method can be called for the VueSelector or the snapshot this selector returns.

#### Limitations
`testcafe-vue-selectors` support Vue starting with version 2.

Only props, state and computed parts of a Vue component are available.

To check if a component can be found, use the [vue-dev-tools](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd) extension for Google Chrome.

Pages with multiple Vue root nodes are not supported.
