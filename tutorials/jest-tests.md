<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->

- [1. Mock objects and data](#1-mock-objects-and-data)
    * [1.1. Apex calls](#11-apex-calls)
    * [1.2. @api functions of child elements](#12-api-functions-of-child-elements)
- [2. What should we cover](#2-what-should-we-cover)
    * [2.1. New story or enhancement](#21-new-story-or-enhancement)
    * [2.2. Bug or defect](#22-bug-or-defect)
- [3. What should we call](#3-what-should-we-call)
    * [3.1. Life-cycle callbacks](#31-life-cycle-callbacks)
    * [3.2. Event handlers](#32-event-handlers)
    * [3.3. Functions and properties with @api](#33-functions-and-properties-with-api)
- [4. What should we validate](#4-what-should-we-validate)
    * [4.1. HTML](#41-html)
    * [4.2. Fields/properties with @api](#42-fieldsproperties-with-api)
    * [4.3. Shared data](#43-shared-data)
    * [4.4. Dispatched events](#44-dispatched-events)
    * [4.5. Apex calls](#45-apex-calls)
    * [4.6. Messages (success, error, info, warning)](#46-messages-success-error-info-warning)
- [5. Best Practices](#5-best-practices)
    * [5.1. Existence validation and further usage](#51-existence-validation-and-further-usage)
    * [5.2. Difficult validation logic](#52-difficult-validation-logic)
- [6. Full Example](#6-full-example)
    * [6.1. Explanation](#61-explanation)
    * [6.2. Requirements and Solution](#62-requirements-and-solution)

<!-- TOC end -->

<!-- TOC --><a name="1-mock-objects-and-data"></a>
# 1. Mock objects and data

There are a few entities you can mock in your Jest tests. We use a few helper functions to easily mock the entities. More details about how you can mock in Jest you can read [here](https://jestjs.io/docs/mock-functions).

<!-- TOC --><a name="11-apex-calls"></a>
## 1.1. Apex calls

***yourLwc.html***

```html
<template>
	<lightning-input label="User Name" value={userName}></lightning-input>
	<lightning-button label="Search" onclick={handleSearchButtonClicked}></lightning-button>
</template>
```

***yourLwc.js***

```javascript
import CustomElement from 'c/customElement';

import init from '@salesforce/apex/YourController.init';
import search from '@salesforce/apex/YourController.search';

export default class YourLwc extends CustomElement {
	userName = ``;
	
	async connectedOnceCallback() {
		this.userName = await this.apex(init);
	}
	
	async handleSearchButtonClicked() {
		this.userName = await this.apex(search, {text: this.query(`lightning-input`).value});
	}
}
```

***yourLwc.test.js***

```javascript
import {createComponent} from "shared";
import YourLwc from 'c/your-lwc';

describe('c-your-lwc', () => {
	// NOTE: The test function is "async".
	it('call-apex', async () => {
		// We'd like to test our c-your-lwc component.
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component.
		// 4th parameter of the function is used to mock returns of the Apex methods.
		// It is an object with keys as Apex action names and values as the return values.
		// The mock return value can be an array, object, a simple type like string or number, or a function.
		// NOTE: "await" before the function call.
		let component = await createComponent('c-your-lwc', YourLwc, null, {
			init: `Current User Name`,
			search: args => {
				if (args.text === `1`) {
					return `Search result 1`;
				} else {
					return `Search result 2`;
				}
			}
		});
		
		// Validate the value of the text input equals to the value
		// which came from `init` mock of Apex action.
		const textElement = await component.query(`lightning-input`);
		expect(textElement.value).toBe(`Current User Name`);
		
		// Set value of the text element to `1`.
		textElement.value = `1`;
		
		// We emulate the click of the button.
		await component.dispatch('click', null, null,'lightning-button');
		
		// Validate the value of the text input equals to the value
		// which came from `search` mock of Apex action.
		// As text element value was equal to `1` before we clicked the button,
		// we expect the new value of the text element to be equal to `Search result 1`,
		// because the mock function for search Apex action
		// returns `Search result 1` if args.text == '1'.
		expect(textElement.value).toBe(`Search result 1`);
		
		// Set value of the text element to `2`.
		textElement.value = `2`;
		
		// We emulate the click of the button.
		await component.dispatch('click', null, null, 'lightning-button');
		
		// Validate the value of the text input equals to the value
		// which came from `search` mock of Apex action.
		// As text element value was equal to `2` before we clicked the button,
		// we expect the new value of the text element to be equal to `Search result 2`,
		// because the mock function for search Apex action
		// returns `Search result 2` if args.text != '1'.
		expect(textElement.value).toBe(`Search result 2`);
	});
})
```

<!-- TOC --><a name="12-api-functions-of-child-elements"></a>
## 1.2. @api functions of child elements

***yourLwc.html***

```html
<template>
	<lightning-datatable data={records} key-field="Id" columns={columns}></lightning-datatable>
	<lightning-button label="Send" onclick={handleSendButtonClicked}></lightning-button>
</template>
```

***yourLwc.js***

```javascript
import CustomElement from 'c/customElement';

export default class YourLwc extends CustomElement {
	records = [{Id: `id1`}, {Id: `id2`}];
	columns = [{label: `ID`, fieldName: 'Id'}];
	
	async handleSendButtonClicked() {
		const ids = this.query('lightning-datatable').getSelectedRows().map(record => record.Id);
		
		this.dispatch('send', {ids: ids});
	}
}
```

***yourLwc.test.js***

```javascript
import {createComponent} from "shared";
import YourLwc from 'c/your-lwc';

describe('c-your-lwc', () => {
	// NOTE: The test function is "async".
	it('mock-child-function', async () => {
		// We'd like to test our c-your-lwc component.
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component.
		// NOTE: "await" before the function call.
		let component = await createComponent(`c-your-lwc`, YourLwc);
		
		// We mock `getSelectedRows` function of `lightning-datatable` element 
		// to always return [{Id: `idTest1`}].
		// NOTE: "await" before the function call.
		await component.mock(`lightning-datatable`, `getSelectedRows`, () => [{Id: `idTest1`}])
		
		// We would like to emulate the button click.
		// NOTE: "await" before the function call.
		await component.dispatch(`click`, null, null, `lightning-button`);
		
		// We expect the "send" event was called
		// and detail.ids field contains IDs from mocked `getSelectedRows` function 
		// of the datatable element.
		// Use the function {@link JestHtmlElement#isEventDispatched}
		component.isEventDispatched(`send`, {ids: [`idTest1`]});
	});
})
```

<!-- TOC --><a name="2-what-should-we-cover"></a>
# 2. What should we cover

<!-- TOC --><a name="21-new-story-or-enhancement"></a>
## 2.1. New story or enhancement
We should write a test with a basic ***positive*** scenario of the new functionality. No need to test wrong behaviour of the end-user.

<!-- TOC --><a name="22-bug-or-defect"></a>
## 2.2. Bug or defect
We should write a new test or update an existing one to reproduce the reported issue.

<!-- TOC --><a name="3-what-should-we-call"></a>
# 3. What should we call

There are multiple entry points to your LWC component's code. There is the full list of the entries:

- Life-cycle callbacks
- Event handlers
- Functions, properties and fields with @api

You should call most of the entries in your Jest test and validate the calls' results.
In this section you can find examples of how to call the entry points.
In the next section you can find out how to validate the calls' results.

<!-- TOC --><a name="31-life-cycle-callbacks"></a>
## 3.1. Life-cycle callbacks

The callbacks _connectedCallback_ / _connectedOnceCallback_ and _renderedCallback_ / _renderedOnceCallback_
are called automatically during the LWC component creation.
No need to call it explicitly.

<!-- TOC --><a name="32-event-handlers"></a>
## 3.2. Event handlers

***yourLwc.html***

```html

<template>
	<lightning-button label={label} onclick={handleButtonClicked}></lightning-button>
</template>
```

***yourLwc.js***

```javascript
import {api} from 'lwc';
import CustomElement from 'c/customElement';

export default class YourLwc extends CustomElement {
	@api label = ``;
	
	handleButtonClicked() {
		// Some logic here...
	}
}
```

***yourLwc.test.js***

```javascript
import {createComponent} from "shared";
import YourLwc from 'c/your-lwc';

describe('c-your-lwc', () => {
	// NOTE: The test function mush be "async".
	it('call-button-clicked-handler', async () => {
		// We'd like to test our c-your-lwc component.
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component.
		// NOTE: "await" before the function call.
		let component = await createComponent(
			'c-your-lwc', YourLwc, {label: `Apply`});
		
		// We would like to run the "handleButtonClicked" handler of our LWC component.
		// To achieve this, we should emulate the button click. There are 2 ways to do this:
		
		// #### FIRST WAY - longer ####
		// We can query for the button element and then dispatch it's "onclick" event.
		// Use {@link JestHtmlElement#query} or {@link JestHtmlElement#queryAll}
		// to query child components of the current component.
		// NOTE: "await" before the function call.
		let buttonElement = await component.query('lightning-button');
		// To call the handler function, use {@link JestHtmlElement#dispatch} function
		// of the button element to dispatch the "onclick" event which the handler is subscribed to.
		// NOTE: "await" before the function call.
		await buttonElement.dispatch('click');
		
		// #### SECOND WAY - shorter ####
		// We can call {@link JestHtmlElement#dispatch} function of our LWC component
		// with "onclick" event and specify a selector of our button element.
		// NOTE: "await" before the function call.
		await component.dispatch('click', null, null, 'lightning-button');
		
		// Validate the results of the handler call here...
	});
})

```

<!-- TOC --><a name="33-functions-and-properties-with-api"></a>
## 3.3. Functions and properties with @api

***yourLwc.html***

```html

<template>
	<!-- Some code here... -->
</template>
```

***yourLwc.js***

```javascript
import {api} from 'lwc';
import CustomElement from 'c/customElement';

export default class YourLwc extends CustomElement {
	@api set attributeProperty(value) {
		this.__attributeProperty = value;
		
		// Some logic here...
	};
	
	get attributeProperty() {
		// Some logic here...
		
		return this.__attributeProperty;
	}
	
	@api doSomeStuff() {
		// Some logic here...
	}
}
```

***yourLwc.test.js***

```javascript
import {createComponent} from "shared";
import YourLwc from 'c/your-lwc';

describe('c-your-lwc', () => {
	// NOTE: The test function is "async".
	it('call-api-function-property', async () => {
		// We'd like to test our c-your-lwc component. 
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component.
		// NOTE: "await" before the function call.
		let component = await createComponent(
			'c-your-lwc', YourLwc, {attributeProperty: `some value`});
		
		// The c-your-lwc component has the public @api property. 
		// The property was set during the call of {@link SharedTestHelpers#createComponent} above. 
		// Also, the property can be set explicitly, like right below.
		// We should:
		//   - set a value to the property (implicitly by {@link SharedTestHelpers#createComponent} or explicitly)
		//   - validate the results of the assignment 
		//   - get the value of the property and validate the value
		component.attributeProperty = `some value`;
		
		// Validate the results of the property setter call here...
		
		const value = component.attributeProperty;
		
		// Validate the retrieved value of the property getter here...
		
		// The c-your-lwc component has the public @api function. 
		// We should:
		//   - call the function
		//   - validate the results of the call
		component.doSomeStuff();
		
		// Validate the results of doSomeStuff() call here...
	});
})
```

<!-- TOC --><a name="4-what-should-we-validate"></a>
# 4. What should we validate

<!-- TOC --><a name="41-html"></a>
## 4.1. HTML

- Is an HTML element always available and visible? If no, then we should validate it is hidden or visible.
- Some attributes of an HTML element are set with values of fields of JS controller? The values are not static/hardcoded but generated from database data or from user input? If answer is "yes" for both questions, then we should validate the attributes of the HTML element are set with expected values?

***yourLwc.html***

```html

<template>
	<div if:true={showDiv}>
		<lightning-input value={inputValue}></lightning-input>
	</div>
</template>
```

***yourLwc.js***

```javascript
import CustomElement from 'c/customElement';

export default class YourLwc extends CustomElement {
	showDiv = false;
	inputValue = ``;
	
	connectedCallback() {
		try {
			this.showDiv = true;
			this.inputValue = `Some value`;
		} catch (e) {
			this.addError(e);
		}
	}
}
```

***yourLwc.test.js***

```javascript
import {createComponent} from "shared";
import YourLwc from 'c/your-lwc';

describe('c-your-lwc', () => {
	// NOTE: The test function is "async".
	it('validate-div-is-shown-input-has-proper-value', async () => {
		// We'd like to test our c-your-lwc component. 
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component.
		// NOTE: "await" before the function call.
		let component = await createComponent('c-your-lwc', YourLwc);
		
		// We can't access the "showDiv" field in the test code. 
		// But instead we can validate the "div" is shown.
		
		// There are 2 ways to do this:
		
		// #### FIRST WAY - longer ####
		
		// We should query for the "div" element and validate it was retrieved.
		
		// Use {@link JestHtmlElement#query} or {@link JestHtmlElement#queryAll} 
		// to query child components of the current component.
		// NOTE: "await" before the function call.
		let divElement = await component.query(`div`);
		// Validate the queried element isn't null.
		expect(divElement).not.toBeNull();
		
		// #### SECOND WAY - shorter ####
		
		// There is a function {@link JestHtmlElement#isGenerated}. 
		// It will throw an exception if the element doesn't exist.
		// NOTE: "await" before the function call.
		await component.isGenerated(`div`);
		
		// Also, you can validate an opposite case, when the element shouldn't exist in the HTML.
		// There is a function {@link JestHtmlElement#isNotGenerated}. 
		// It will throw an exception if the element exists.
		// NOTE: "await" before the function call.
		await component.isNotGenerated(`div`);
		
		// We can't access the "inputValue" field in the test code. 
		// But instead we can validate the "lightning-input"'s value attribute has an expected value.
		
		// To achieve this we should query for the "lightning-input" element,
		// and validate it's value attribute.
		let inputElement = await component.query('lightning-input');
		expect(inputElement.value).toBe(`Some value`);
		
		// Also, we can use another, single function to query an element 
		// and, with the same function, validate the element exists.
		// The function is {@link JestHtmlElement#queryUnique}
		let inputElement = await component.queryUnique('lightning-input');
		expect(inputElement.value).toBe(`Some value`);
	});
})
```

<!-- TOC --><a name="42-fieldsproperties-with-api"></a>
## 4.2. Fields/properties with @api

***yourLwc.html***

```html

<template>
	<!-- Some code here... -->
</template>
```

***yourLwc.js***

```javascript
import {api} from 'lwc';
import CustomElement from 'c/customElement';

export default class YourLwc extends CustomElement {
	@api set attributeProperty(value) {
		this.__attributeProperty = value;
		
		// Some logic here...
	};
	
	get attributeProperty() {
		// Some logic here...
		
		return this.__attributeProperty;
	}
	
	@api attributeField;
}
```

***yourLwc.test.js***

```javascript
import {createComponent} from "shared";
import YourLwc from 'c/your-lwc';

describe('c-your-lwc', () => {
	// NOTE: The test function is "async".
	it('validate-property-and-field-values', async () => {
		// We'd like to test our c-your-lwc component. 
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component.
		// NOTE: "await" before the function call.
		let component = await createComponent('c-your-lwc', YourLwc);
		
		// The c-your-lwc component has the public @api property. 
		// If we expect the value of the property was modified then we should:
		//   - get the value of the property and validate the value
		const propertyValue = component.attributeProperty;
		expect(propertyValue).toBe(`Expected value`);
		
		// The c-your-lwc component has the public @api field. 
		// If we expect the value of the field was modified, then we should:
		//   - get the value of the field and validate the value
		const fieldValue = component.attributeField;
		expect(fieldValue).toBe(`Expected value`);
	});
})
```

<!-- TOC --><a name="43-shared-data"></a>
## 4.3. Shared data

LWC components, which extend _CustomElement_ class, can have a shared data.
One of the components can set the data and another component will read the data.
The data is accessible by {@link CustomElement#staticData}. To validate the data was set properly by
your LWC component, in your Jest test you can use {@link SharedTestHelpers#staticData} object.

***yourLwc.html***

```html

<template>
	<!-- Some code here... -->
</template>
```

***yourLwc.js***

```javascript
import CustomElement from 'c/customElement';

export default class YourLwc extends CustomElement {
	connectedCallback() {
		this.staticData.dataToShareForYourLwc = {value: `val`};
	}
}
```

***yourLwc.test.js***

```javascript
// Import staticData from "shared"
import {createComponent, staticData} from "shared";
import YourLwc from 'c/your-lwc';

describe('c-your-lwc', () => {
	// NOTE: The test function is "async".
	it('validate-shared-data', async () => {
		// We'd like to test our c-your-lwc component. 
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component.
		// NOTE: "await" before the function call.
		let component = await createComponent('c-your-lwc', YourLwc);
		
		// Validate the staticData equals to the expected.
		expect(staticData.dataToShareForYourLwc).toEqual({value: `val`});
	});
})
```

<!-- TOC --><a name="44-dispatched-events"></a>
## 4.4. Dispatched events

We should validate:
- How many times the event was dispatched?
- What were the arguments of the dispatch?

NOTE: To mock and validate dispatched event properly, your LWC component should dispatch the event
using {@link CustomElement#dispatch}, like in the example below.

***yourLwc.html***

```html

<template>
	<lightning-input label="Text" type="text"></lightning-input>
	<lightning-button label="Button" onclick={handleButtonClicked}></lightning-button>
</template>
```

***yourLwc.js***

```javascript
import CustomElement from 'c/customElement';

export default class YourLwc extends CustomElement {
	async handleButtonClicked() {
		const inputValue = this.template.querySelector(`lightning-input`).value;
		this.dispatch('buttonclicked', {detail: {text: inputValue}});
	}
}
```

NEW APPROACH:

***yourLwc.test.js***

```javascript
import {createComponent} from "shared";
import YourLwc from 'c/your-lwc';

describe('c-your-lwc', () => {
	// NOTE: The test function is "async".
	it('validate-event-dispatchd', async () => {
		// We'd like to test our c-your-lwc component. 
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component.
		// NOTE: "await" before the function call.
		let component = await createComponent('c-local-development', YourLwc);
		
		// The c-your-lwc component dispatches the onbuttonclicked event 
		// once the lightning-button is clicked. 
		// The onbuttonclicked event is dispatched with a detail.text field.
		// So, we should validate 2 things:
		//   - the event was dispatched.
		//   - the event's detail.text field has an expected value. 
		
		// To achive this, we should set the lightning-input's value attribute with some value, 
		// then emulate the lightning-button click, 
		// and then validate the 2 things explained above. 
		
		// We would like to emulate the text input element is set with some value.
		// NOTE: "await" before the function call.
		let textElement = await component.query('lightning-input');
		textElement.value = `John`;
		
		// We would like to emulate the button click. 
		// NOTE: "await" before the function call.
		await component.dispatch('click', null, null, 'lightning-button');
		
		// We expect the "onbuttonclicked" event was called 
		// and detail.text field has the value "John"
		// Use the function {@link JestHtmlElement#isEventDispatched}
		component.isEventDispatched('buttonclicked', {text: `John`});
	});
})
```

OLD APPROACH:

***yourLwc.test.js***

```javascript
import {createComponent} from "shared";
import YourLwc from 'c/your-lwc';

describe('c-your-lwc', () => {
	// NOTE: The test function is "async".
	it('validate-event-dispatchd', async () => {
		// We'd like to test our c-your-lwc component. 
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component.
		// NOTE: "await" before the function call.
		let component = await createComponent('c-local-development', YourLwc);
		
		// The c-your-lwc component dispatches the onbuttonclicked event. 
		// Use {@link JestHtmlElement#listen} function to listen for events which are dispatched by the component. 
		// This function returns a mock function, which will be called 
		// when the event will be dispatched by the component.
		// If the mock function was called it means the onbuttonclicked event was dispatched.
		// So, we should validate that for this unit test the event will be dispatched 
		// and the parameters of the event are correct. 
		// We should assert that the mock function was called with correct parameters.
		let listenerSuccess = component.listen(`buttonclicked`);
		
		// We would like to emulate the text input element is set with some value.
		// NOTE: "await" before the function call.
		let textElement = await component.query('lightning-input');
		textElement.value = `John`;
		
		// We would like to emulate the button click. 
		// To achieve this we should query for the button element and then dispatch it's onclick event.
		// Use {@link JestHtmlElement#query} or {@link JestHtmlElement#queryAll} to query child components of the current component.
		// NOTE: "await" before the function call.
		let buttonElement = await component.query('lightning-button');
		// To call a handler function, use {@link JestHtmlElement#dispatch} function 
		// to dispatch the onclick event which the handler is subscribed to.
		// NOTE: "await" before the function call.
		await buttonElement.dispatch('click');
		
		// We expect the "onbuttonclicked" event was called.
		expect(listenerSuccess).toHaveBeenCalled();
		// We expect the first parameter of the event has a detail.text field with a value "John"
		expect(listenerSuccess.mock.calls[0][0].detail.text).toBe(`John`);
	});
})
```

<!-- TOC --><a name="45-apex-calls"></a>
## 4.5. Apex calls

We should validate:
- How many times the apex method was called?
- What were the arguments for the apex call?

NOTE: To mock and validate Apex calls properly, your LWC component should call Apex action method
using {@link CustomElement#apex}, like in the example below.

***yourLwc.html***

```html

<template>
	<lightning-input label="Text" type="text"></lightning-input>
	<lightning-button label="Search" onclick={handleSearchButtonClicked}></lightning-button>
</template>
```

***yourLwc.js***

```javascript
import CustomElement from 'c/customElement';
import search from '@salesforce/apex/YourController.search';

export default class YourLwc extends CustomElement {
	async handleSearchButtonClicked() {
		let text = this.template.querySelector(`lightning-input`).value;
		const candidateNames = await this.apex(search, {searchText: text});
		
		// Other code here...
	}
}
```

NEW APPROACH:

***yourLwc.test.js***

```javascript
import {createComponent} from "shared";
import YourLwc from 'c/your-lwc';

describe('c-your-lwc', () => {
	// NOTE: The test function is "async".
	it('validate-apex-call', async () => {
		// We'd like to test our c-your-lwc component. 
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component.
		// NOTE: "await" before the function call.
		let component = await createComponent('c-your-lwc', YourLwc);
		
		// The c-your-lwc component calls Apex action to do the search.
		// once the lightning-button is clicked. 
		// The Apex code is called with "searchText" argument.
		// So, we should validate 2 things:
		//   - the Apex method was called.
		//   - the Apex method's "searchText" argument has an expected value. 
		
		// To achive this, we should set the lightning-input's value attribute with some value, 
		// then emulate the lightning-button click, 
		// and then validate the 2 things explained above. 
		
		// We would like to emulate the text input element is set with some value.
		// NOTE: "await" before the function call.
		let textElement = await component.query('lightning-input');
		textElement.value = `John`;
		
		// We would like to emulate the button click. 
		// NOTE: "await" before the function call.
		await component.dispatch('click', null, null, 'lightning-button');
		
		// We expect the "search" Apex action was called 
		// and there is an argument "searchText" with the value "John".
		// Use the function {@link JestHtmlElement#isApexCalled}
		component.isApexCalled('search', {searchText: `John`});
	});
})
```

OLD APPROACH:

In your Jest test you should import {@link SharedTestHelpers#apexMock} function
and use it to mock the Apex method call.

***yourLwc.test.js***

```javascript
// Import apexMock from "shared"
import {createComponent, apexMock} from "shared";
import YourLwc from 'c/your-lwc';

describe('c-your-lwc', () => {
	// NOTE: The test function is "async".
	it('validate-apex-call', async () => {
		// We'd like to test our c-your-lwc component. 
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component.
		// NOTE: "await" before the function call.
		let component = await createComponent('c-your-lwc', YourLwc);
		
		// The c-your-lwc component calls Apex action to do the search. 
		// So, we should validate that the Apex action was called and the parameters of the call are correct.
		// Mock apex action call with {@link SharedTestHelpers#apexMock} function. 
		// This function returns a mock function, which will be called 
		// when the apex action will be called by the LWC component. 
		// If the mock function was called it means the "search" Apex action was called.
		// NOTE: it will work correctly if you use for Apex calls the {@link CustomElement#apex} only.
		let searchApexCall = apexMock(`search`);
		
		// We would like to emulate the text input element is set with some value.
		// NOTE: "await" before the function call.
		let textElement = await component.query('lightning-input');
		textElement.value = `John`;
		
		// We would like to emulate the button click. 
		// To achieve this we should query for the button element and then dispatch it's onclick event.
		// Use {@link JestHtmlElement#query} or {@link JestHtmlElement#queryAll} to query child components of the current component.
		// NOTE: "await" before the function call.
		let buttonElement = await component.query('lightning-button');
		// To call a handler function, use {@link JestHtmlElement#dispatch} function 
		// to dispatch the onclick event which the handler is subscribed to.
		// NOTE: "await" before the function call.
		await buttonElement.dispatch('click');
		
		// We expect the "search" Apex action was called.
		expect(searchApexCall).toHaveBeenCalled();
		// We expect the first parameter of the Apex action call has a searchText field with a value "John".
		expect(searchApexCall.mock.calls[0][0].searchText).toBe(`John`);
	});
})
```

<!-- TOC --><a name="46-messages-success-error-info-warning"></a>
## 4.6. Messages (success, error, info, warning)

You can validate the messages which were shown using
{@link CustomElement#addError},
{@link CustomElement#addSuccess},
{@link CustomElement#addWarning},
{@link CustomElement#addInfo} functions.

To validate the messages in your Jest test you should import from "shared" the {@link
SharedTestHelpers#getMessages} function.
Then use it to retrieve a list of all messages sent by the LWC component life-cycle.
Then you can assert the retrieved messages.

***yourLwc.html***

```html

<template>
	<c-messages messages={messagesGlobal} type="toast"></c-messages>
</template>
```

***yourLwc.js***

```javascript
import CustomElement from 'c/customElement';

export default class YourLwc extends CustomElement {
	connectedCallback() {
		this.addError(`Error message.`);
		this.addSuccess(`Success message.`);
		this.addWarning(`Warning message.`);
		this.addInfo(`Info message.`);
	}
}
```

***yourLwc.test.js***

```javascript
// Import getMessages from "shared"
import {createComponent, getMessages} from "shared";
import YourLwc from 'c/your-lwc';

describe('c-your-lwc', () => {
	// NOTE: The test function is "async".
	it('validate-messages', async () => {
		// We'd like to test our c-your-lwc component. 
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component.
		// NOTE: "await" before the function call.
		let component = await createComponent('c-your-lwc', YourLwc);
		
		// We expect:
		// - 1 error message with text "Error message."
		// - 1 success message with text "Success message."
		// - 1 warning message with text "Warning message."
		// - 1 info message with text "Info message."
		
		// Use getMessages function imported from "shared" to get a list of all messages 
		// sent by the LWC component life-cycle. 
		let messages = getMessages();
		
		// We expect 4 messages to be shown by now.
		expect(messages.length).toBe(1);
		
		// 1st message has a severity "ERROR" and a text "Error message."
		expect(messages[0].severity).toBe(`ERROR`);
		expect(messages[0].title).toBe(`Error message.`);
		
		// 2nd message has a severity "SUCCESS" and a text "Success message."
		expect(messages[1].severity).toBe(`SUCCESS`);
		expect(messages[1].title).toBe(`Success message.`);
		
		// 3rd message has a severity "WARNING" and a text "Warning message."
		expect(messages[2].severity).toBe(`WARNING`);
		expect(messages[2].title).toBe(`Warning message.`);
		
		// 4th message has a severity "INFO" and a text "Info message."
		expect(messages[3].severity).toBe(`INFO`);
		expect(messages[3].title).toBe(`Info message.`);
	});
})
```

<!-- TOC --><a name="5-best-practices"></a>
# 5. Best Practices

<!-- TOC --><a name="51-existence-validation-and-further-usage"></a>
## 5.1. Existence validation and further usage

No need to check the existence of an element if you will operate with the element in the code later. E.g. you click a button in the Jest test to validate the event handler of the button. So, no need to validate the button exists. You will get an exception if you click a button which does not exist.

***yourLwc.html***

```html
<template>
	<lightning-button lwc:if={show} label="Button" onclick={handleButtonClicked}></lightning-button>
</template>
```

***yourLwc.js***

```javascript
import CustomElement from 'c/customElement';

export default class YourLwc extends CustomElement {
	show = false;
	
	connectedOnceCallback() {
		this.show = true;
	}
	
	async handleButtonClicked() {
		// Some logic here...
	}
}
```

***yourLwc.test.js***

```javascript
import {createComponent} from "shared";
import YourLwc from 'c/your-lwc';

describe('c-your-lwc', () => {
	it('no-validation-when-usage', async () => {
		let component = await createComponent(`c-your-lwc`, YourLwc);
		
		// NO NEED to check the existence of the button.
		// With the following command (await component.dispatch) we click the button
		// and exception will be thrown if the button does not exist. 
		// So, the next command (await component.isGenerated) should be removed.
		await component.isGenerated(`lightning-button`);
		
		// Click the button.
		await component.dispatch(`click`, null, null, `lightning-button`);
		
		// Validate a result of the button click here ...
	});
})
```

<!-- TOC --><a name="52-difficult-validation-logic"></a>
## 5.2. Difficult validation logic

Try to avoid loops and "if"s for validations.

No need for difficult validation logic. As an author of your test you know your test data. You know records of the data you have.

***yourLwc.html***

```html
<template>
	<div for:each={contacts} for:item="contact" key={contact.name}>{contact.name}</div>
</template>
```

***yourLwc.js***

```javascript
import CustomElement from 'c/customElement';

import init from '@salesforce/apex/YourController.init';

export default class YourLwc extends CustomElement {
	contacts = [];
	
	async connectedOnceCallback() {
		this.contacts = await this.apex(init);
	}
}
```

***yourLwc.test.js***

```javascript
import {createComponent} from "shared";
import YourLwc from 'c/your-lwc';

const TEST_DATA = [{name: `John`}, {name: `Adam`}];

describe(`c-your-lwc`, () => {
	it(`difficult-validation-logic`, async () => {
		// TEST_DATA is a response from "init" Apex call.
		let component = await createComponent(`c-your-lwc`, YourLwc, null, {init: TEST_DATA});
		
		const divElements = await component.queryAll(`div`);
		
		// This is a difficult to understand logic. We should AVOID doing so.
		for (let i = 0; i < TEST_DATA.length; i++) {
			expect(divElements[i].textContent).toBe(TEST_DATA[i].name);
		}
		
		// This approach is much easier to understand of what we expect in the HTML.
		// Let's USE this approach.
		expect(divElements[0].textContent).toBe(`John`);
		expect(divElements[1].textContent).toBe(`Adam`);
	});
})
```

<!-- TOC --><a name="6-full-example"></a>
# 6. Full Example

<!-- TOC --><a name="61-explanation"></a>
## 6.1. Explanation

We should import helper functions from {@link SharedTestHelpers} library.
We know that we should create a LWC component, so we need a {@link
SharedTestHelpers#createComponent} function,
which is a powerful wrapper
for [createElement](https://developer.salesforce.com/docs/component-library/documentation/en/lwc/lwc.unit_testing_using_jest_create_tests)
function.
Also, we know that the LWC will show success/error messages, so we need an {@link
SharedTestHelpers#getMessages} function.

***yourLwc.test.js***

```javascript
import {createComponent, getMessages} from "shared";
import YourLwc from 'c/your-lwc';
```

We should identify what are the scenarios we should test.
According to [What should we cover](#what-should-we-cover) section we should test positive scenarios. Also, we can validate all possible errors.

So, we will test 3 scenarios:

- User didn't set a text input value (_text-not-entered_ test)
- User set the text input value but there is no search result (_text-invalid_ test)
- User set the text input value and candidates were found (_text-valid_ test)

Please note, the test functions are _**async**_. This is not like in standard LWC documentation.

***yourLwc.test.js***

```javascript
describe('c-your-lwc', () => {
	it('text-not-entered', async () => {
	
	});
	
	it('text-invalid', async () => {
	
	});
	
	it('text-valid', async () => {
	
	});
})
```

See the unit [tests content](#yourLwc-test-js) comments below to get more details about the things
which should be tested.

<!-- TOC --><a name="62-requirements-and-solution"></a>
## 6.2. Requirements and Solution

There is a LWC component with 2 attributes: input-label and button-label.
The labels should be set by a developer.
The LWC component shows a text input element and a search button.
The user can set some text to the input element and click the *Search* button.
Once the button is clicked it will search for candidates in Salesforce database
by the candidates' first name via Apex call.
The Apex call returns a string with full names of the found candidates.

If user clicked the button without setting any value for input field then:

- it should show an error message *Please enter some text.*

If user set the input field value, clicked the button and there is no any candidate found then:

- it should show an error message *No one candidate was found.*

If user set the input field value, clicked the button and any of the candidates are found then:

- it should show a success message *Candidates were found.*
- send an onsuccess event with the list of the candidate full names
- show the list of the candidate full names on the component layout inside the *div* element

***yourLwc.html***

```html

<template>
	<c-messages messages={messagesGlobal} type="toast"></c-messages>
	
	<lightning-input label={inputLabel} type="text"></lightning-input>
	<lightning-button label={buttonLabel} onclick={handleSearchButtonClicked}></lightning-button>
	
	<div if:true={isFound}>{candidateNames}</div>
</template>
```

***yourLwc.js***

```javascript
import {api} from 'lwc';
import CustomElement from 'c/customElement';
import search from '@salesforce/apex/YourController.search';

export default class YourLwc extends CustomElement {
	@api inputLabel = ``;
	@api buttonLabel = ``;
	
	candidateNames = ``;
	isFound = false;
	
	async handleSearchButtonClicked() {
		this.text = ``;
		
		let text = this.template.querySelector(`lightning-input`).value;
		if (!text) {
			this.addError(`Please enter some text.`);
			return;
		}
		
		this.candidateNames = await this.apex(search, {text: text});
		
		this.isFound = !!this.candidateNames;
		if (this.isFound) {
			this.addSuccess(`Candidates were found.`);
			this.text = this.candidateNames;
			
			this.dispatch('success', {detail: {candidateNames: this.candidateNames}});
		} else {
			this.addError(`No one candidate was found.`);
		}
	}
}
```

***yourLwc.test.js*** <a name="yourLwc-test-js"></a>

```javascript
import {createComponent, getMessages} from "shared";
import YourLwc from 'c/your-lwc';

describe('c-your-lwc', () => {
	// NOTE: The test function is "async".
	it('text-not-entered', async () => {
		// We'd like to test our c-your-lwc component. 
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component.
		// NOTE: "await" before the function call.
		let component = await createComponent(
			'c-your-lwc', YourLwc, {inputLabel: `input label text`, buttonLabel: `button label text`});
		
		// We would like to emulate a Search button click.
		await component.dispatch('click', null, null, 'lightning-button');
		
		// Once the button's onclick event is dispatched and the handler function is called, we can validate:
		// - Was "onsuccess" event dispatched? If yes, then what are the values of the parameters of the event?
		// - Was "search" Apex action called? If yes, then what are the values of the parameters of the call?
		// - Does the component show the list of the found candidates to the user?
		// - Does the component show success/error messages?
		
		// We don't expect the "search" Apex action was called.
		component.isApexCalled(`search`, null, 0);
		
		// We don't expect the "onsuccess" event was dispatched.
		component.isEventDispatched(`success`, null, 0);
		
		// We don't expect the "div" with candidate names will be shown.
		// NOTE: "await" before the function call.
		await component.isNotGenerated('div');
		
		// We expect an error message "Please enter some text." will be shown.
		let errorMessages = getMessages();
		// NOTE: we also validate that this is the only message to be shown.
		expect(errorMessages.length).toBe(1);
		expect(errorMessages[0].severity).toBe(`ERROR`);
		expect(errorMessages[0].title).toBe(`Please enter some text.`);
	});
	
	// NOTE: The test function is "async".
	it('text-invalid', async () => {
		// We'd like to test our c-your-lwc component. 
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component. 
		// NOTE: "await" before the function call.
		let component = await createComponent('c-local-development', YourLwc);
		
		// We would like to emulate the text input element is set with some value.
		// NOTE: "await" before the function call.
		let textElement = await component.query('lightning-input');
		textElement.value = `Some invalid text`;
		
		// We would like to emulate a Search button click. 
		// To call a handler function, use {@link JestHtmlElement#dispatch} function 
		// to dispatch the onclick event which the handler is subscribed to.
		// NOTE: 2nd parameter `{}` is not required. 
		// We can use null value instead. `{}` is show here for demonstration purposes only.
		// It is there just to show how we can pass additional details
		// required for the functional code to the event.
		// NOTE: "await" before the function call.
		await component.dispatch('click', {}, null, 'lightning-button');
		
		// Once the button's onclick event is dispatched and the handler function is called, we can validate:
		// - Was "onsuccess" event dispatched? If yes, then what are the values of the parameters of the event?
		// - Was "search" Apex action called? If yes, then what are the values of the parameters of the call?
		// - Does the component shows the list of the found candidates to the user?
		// - Does the component shows success/error messages?
		
		// We expect the "search" Apex action will be called,
		// and the text argument of the Apex call is set to a value "Some invalid text"
		component.isApexCalled(`search`, {text: `Some invalid text`});
		
		// We don't expect the "onsuccess" event was dispatched.
		component.isEventDispatched(`success`, null, 0);
		
		// We don't expect the "div" with candidate names will be shown.
		// NOTE: "await" before the function call.
		await component.isNotGenerated('div');
		
		// We expect an error message "No one candidate was found." will be shown.
		let errorMessages = getMessages();
		// NOTE: we also validate that this is the only message to be shown.
		expect(errorMessages.length).toBe(1);
		expect(errorMessages[0].severity).toBe(`ERROR`);
		expect(errorMessages[0].title).toBe(`No one candidate was found.`);
	});
	
	// NOTE: The test function is "async".
	it('text-valid', async () => {
		// We'd like to test our c-your-lwc component. 
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component.
		// NOTE: "await" before the function call.
		let component = await createComponent('c-local-development', YourLwc);
		
		// We would like to emulate the text input element is set with some value.
		// NOTE: "await" before the function call.
		let textElement = await component.query('lightning-input');
		textElement.value = `John`;
		
		// We would like to emulate a Search button click. 
		// To call a handler function, use {@link JestHtmlElement#dispatch} function 
		// to dispatch the onclick event which the handler is subscribed to.
		// NOTE: "await" before the function call.
		await component.dispatch('click', null, null, 'lightning-button');
		
		// Once the button's onclick event is dispatched and the handler function is called, we can validate:
		// - Was "onsuccess" event dispatched? If yes, then what are the values of the parameters of the event?
		// - Was "search" Apex action called? If yes, then what are the values of the parameters of the call?
		// - Does the component shows the list of the found candidates to the user?
		// - Does the component shows success/error messages?
		
		// We expect the "search" Apex action will be called,
		// and the text argument of the Apex call is set to a value "John"
		component.isApexCalled(`search`, {text: `John`});
		
		// We expect the "onsuccess" event was dispatched,
		// and the text argument of the event is set to a value "John Doe, John Tyson"
		component.isEventDispatched(`success`, {candidateNames: `John Doe, John Tyson`});
		
		// We expect the "div" will be shown.
		// We can use a single function to query an element 
		// and, with the same function, validate the element exists.
		// The function is {@link JestHtmlElement#queryUnique}
		// NOTE: "await" before the function call.
		let divElement = await component.queryUnique('div');
		// We expect the "div" has candidate names as it's content.
		expect(divElement.textContent).toBe(`John Doe, John Tyson`);
		
		// We expect a success message "Candidates were found." will be shown.
		let errorMessages = getMessages();
		// NOTE: we also validate that this is the only message to be shown.
		expect(errorMessages.length).toBe(1);
		expect(errorMessages[0].severity).toBe(`SUCCESS`);
		expect(errorMessages[0].title).toBe(`Candidates were found.`);
	});
})
```
