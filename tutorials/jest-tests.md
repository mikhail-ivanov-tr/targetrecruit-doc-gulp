<!-- TOC -->
* [1. Prerequisites](#1-prerequisites)
* [2. What should we cover](#2-what-should-we-cover)
* [3. What should we call from the Jest test](#3-what-should-we-call-from-the-jest-test)
  * [3.1. Life-cycle callbacks <a name="call-callbacks"></a>](#31-life-cycle-callbacks-a-namecall-callbacksa)
  * [3.2. Event handlers <a name="call-handlers"></a>](#32-event-handlers-a-namecall-handlersa)
  * [3.3. Functions and properties with @api <a name="call-api"></a>](#33-functions-and-properties-with-api-a-namecall-apia)
* [4. What should we validate](#4-what-should-we-validate)
  * [4.1. HTML](#41-html)
  * [4.2. Fields/properties with @api <a name="validate-api"></a>](#42-fieldsproperties-with-api-a-namevalidate-apia)
  * [4.3. Shared data <a name="validate-shared-data"></a>](#43-shared-data-a-namevalidate-shared-dataa)
  * [4.4. Dispatched events <a name="validate-events"></a>](#44-dispatched-events-a-namevalidate-eventsa)
  * [4.5. Apex calls <a name="validate-apex-calls"></a>](#45-apex-calls-a-namevalidate-apex-callsa)
  * [4.6. Messages (success, error, info, warning) <a name="validate-messages"></a>](#46-messages-success-error-info-warning-a-namevalidate-messagesa)
* [5. Full Example <a name="example"></a>](#5-full-example-a-nameexamplea)
  * [5.1. Explanation <a name="example-explanation"></a>](#51-explanation-a-nameexample-explanationa)
  * [5.2. Requirements and Solution <a name="example-requirements-and-solution"></a>](#52-requirements-and-solution-a-nameexample-requirements-and-solutiona)
<!-- TOC -->

# 1. Prerequisites

You should understand [Mock Functions](https://jestjs.io/docs/mock-functions).

# 2. What should we cover

- Positive scenario.

# 3. What should we call from the Jest test

There are multiple entry points to your LWC component's code. There is the full list of the entries:

- Life-cycle callbacks
- Event handlers
- Functions, properties and fields with @api

You should call most of the entries in your Jest test and validate the calls' results.
In this section you can find examples of how to call the entry points.
In the next section you can find out how to validate the calls' results.

## 3.1. Life-cycle callbacks <a name="call-callbacks"></a>

The callbacks _connectedCallback_ and _renderedCallback_
are called automatically during the LWC component creation. 
No need to call it explicitly.

## 3.2. Event handlers <a name="call-handlers"></a>

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
		await component.dispatch('click', null, 'lightning-button');
		
		// Validate the results of the handler call here...
	});
})

```

## 3.3. Functions and properties with @api <a name="call-api"></a>

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

# 4. What should we validate

## 4.1. HTML

- Is an HTML element always available and visible? If no, then we should validate it is hidden or visible.
- Some attributes of an HTML element are set with values of fields of JS controller? If yes, then we should validate the attributes of the HTML element are set with expected values?

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

## 4.2. Fields/properties with @api <a name="validate-api"></a>

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

## 4.3. Shared data <a name="validate-shared-data"></a>

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

## 4.4. Dispatched events <a name="validate-events"></a>

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
		await component.dispatch('click', null, 'lightning-button');
		
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

## 4.5. Apex calls <a name="validate-apex-calls"></a>

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
		await component.dispatch('click', null, 'lightning-button');
		
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

## 4.6. Messages (success, error, info, warning) <a name="validate-messages"></a>

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

# 5. Full Example <a name="example"></a>

## 5.1. Explanation <a name="example-explanation"></a>

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

## 5.2. Requirements and Solution <a name="example-requirements-and-solution"></a>

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
		await component.dispatch('click', null, 'lightning-button');
		
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
		// NOTE: 2nd parameter `{detail: {}}` is not required. 
		// We can use null value instead. `{detail: {}}` is show here for demonstration purposes only.
		// It is there just to show how we can pass additional details
		// required for the functional code to the event.
		// NOTE: "await" before the function call.
		await component.dispatch('click', {detail: {}}, 'lightning-button');
		
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
		await component.dispatch('click', null, 'lightning-button');
		
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
		component.isEventDispatched(`success`, {text: `John Doe, John Tyson`});
		
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
