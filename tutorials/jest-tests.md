# Table of Contents

- [1. Prerequisites](#prerequisites)
- [2. Test Scope](#test-scope)
    + [2.1. What should we cover](#what-should-we-cover)
    + [2.2. What should we call from outside](#call)
        - [2.2.1. Life-cycle callbacks](#call-callbacks)
        - [2.2.2. Event handlers](#call-handlers)
        - [2.2.3. Functions and properties with @api](#call-api)
    + [2.3. What should we validate](#validate)
        - [2.3.1. HTML](#validate-html)
        - [2.3.2. Fields/properties with @api](#validate-api)
        - [2.3.3. Shared data](#validate-shared-data)
        - [2.3.4. Dispatched events](#validate-events)
        - [2.3.5. Apex calls](#validate-apex-calls)
        - [2.3.6. Messages (success, error, info, warning)](#validate-messages)
- [3. Full Example](#example)
    + [3.1. Requirement](#example-requirement)
    + [3.2. Explanation](#example-explanation)

# 1. Prerequisites <a name="prerequisites"></a>

You should understand [Mock Functions](https://jestjs.io/docs/mock-functions).

# 2. Test Scope <a name="test-scope"></a>

### 2.1. What should we cover <a name="what-should-we-cover"></a>

- Positive scenario.
- Negative/Errors scenario. Each error should be covered.

### 2.2. What should we call from outside <a name="call"></a>

There are multiple entry points to your LWC component's code. There is the full list of the entries:

- Life-cycle callbacks
- Event handlers
- Functions, properties and fields with @api

Most of the entries you should call in your Jest test and validate the calls results.
In this section you can find examples of how to call the entry points.
How to validate the calls results will find out in next section.

#### 2.2.1. Life-cycle callbacks <a name="call-callbacks"></a>

The callbacks _connectedCallback_ and _renderedCallback_
are called automatically during the LWC component creation.

#### 2.2.2. Event handlers <a name="call-handlers"></a>

***yourLwc.html***

```html

<template>
	<lightning-button label="Button" onclick={handleButtonClicked}></lightning-button>
</template>
```

***yourLwc.js***

```javascript
import CustomElement from 'c/customElement';

export default class YourLwc extends CustomElement {
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
	// NOTE: The test function is "async".
	it('call-button-clicked-handler', async () => {
		// We'd like to test our c-your-lwc component. 
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component.
		// NOTE: "await" before the function call.
		let component = await createComponent('c-your-lwc', YourLwc);
		
		// We would like to emulate the button click. 
		// To achieve this we should query for the button element and then dispatch it's onclick event.
		// Use {@link JestHtmlElement#query} or {@link JestHtmlElement#queryAll} to query child components of the current component.
		// NOTE: "await" before the function call.
		let buttonElement = await component.query('lightning-button');
		// To call the handler function, use {@link JestHtmlElement#dispatch} function 
		// to dispatch the onclick event which the handler is subscribed to.
		// NOTE: "await" before the function call.
		await buttonElement.dispatch('click');
		
		// Validate the results of the handler call here...
	});
})
```

#### 2.2.3. Functions and properties with @api <a name="call-api"></a>

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
		let component = await createComponent('c-your-lwc', YourLwc);
		
		// The c-your-lwc component has the public @api property. 
		// We should:
		//   - set a value to the property
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

### 2.3. What should we validate <a name="validate"></a>

#### 2.3.1. HTML <a name="validate-html"></a>

- Is the expected element available and visible in HTML? We should validate it if the element can be
  hidden in some cases.
- Are the attributes of the element set with proper values?

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
			// Some logic here...
			
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
		// To achieve this we should query for the "div" element and validate it was queried.
		
		// Use {@link JestHtmlElement#query} or {@link JestHtmlElement#queryAll} to query child components of the current component.
		// NOTE: "await" before the function call.
		let divElement = await component.query('div');
		// Validate the queried element isn't null.
		expect(divElement).not.toBeNull();
		
		// We can't access the "inputValue" field in the test code. 
		// But instead we can validate the "lightning-input"'s value attribute has an expected value. 
		// To achieve this we should query for the "lightning-input" element and validate it's value attribute.
		let inputElement = await component.query('lightning-input');
		expect(inputElement.value).toBe(`Some value`);
	});
})
```

#### 2.3.2. Fields/properties with @api <a name="validate-api"></a>

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
		// If we expect the value of the field was modified then we should:
		//   - get the value of the field and validate the value
		const fieldValue = component.attributeField;
		expect(fieldValue).toBe(`Expected value`);
	});
})
```

#### 2.3.3. Shared data <a name="validate-shared-data"></a>

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

#### 2.3.4. Dispatched events <a name="validate-events"></a>

- How many times the event was dispatched?
- What were the arguments of the dispatch?

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
		const text = this.template.querySelector(`lightning-input`).value;
		this.dispatchEvent(
			new CustomEvent('buttonclicked', {detail: {text: text}}));
	}
}
```

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

#### 2.3.5. Apex calls <a name="validate-apex-calls"></a>

- How many times the apex method was called?
- What were the arguments for the apex call?

To mock and validate Apex calls properly, your LWC component should call Apex action method
using {@link CustomElement#apex} like in the example below.

In your Jest test you should import {@link SharedTestHelpers#apexMock} function
and use it to mock the Apex method call.

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

#### 2.3.6. Messages (success, error, info, warning) <a name="validate-messages"></a>

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

# 3. Full Example <a name="example"></a>

### 3.1. Requirement <a name="example-requirement"></a>

There is a LWC component which shows a text input element and a search button.
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
	
	<lightning-input label="Text" type="text"></lightning-input>
	<lightning-button label="Search" onclick={handleSearchButtonClicked}></lightning-button>
	
	<div if:true={isFound}>{candidateNames}</div>
</template>
```

***yourLwc.js***

```javascript
import CustomElement from 'c/customElement';
import search from '@salesforce/apex/YourController.search';

export default class YourLwc extends CustomElement {
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
			
			this.dispatchEvent(
				new CustomEvent('success', {detail: {candidateNames: this.candidateNames}}));
		} else {
			this.addError(`No one candidate was found.`);
		}
	}
}
```

***yourLwc.test.js*** <a name="yourLwc-test-js"></a>

```javascript
import {createComponent, apexMock, getMessages} from "shared";
import YourLwc from 'c/your-lwc';

describe('c-your-lwc', () => {
	// NOTE: The test function is "async".
	it('text-not-entered', async () => {
		// We'd like to test our c-your-lwc component. 
		// Use {@link SharedTestHelpers#createComponent} to create an instance of the LWC component.
		// NOTE: "await" before the function call.
		let component = await createComponent('c-your-lwc', YourLwc);
		
		// The c-your-lwc component calls Apex action to do the search. 
		// So, we should validate that the Apex action was called and the parameters of the call are correct.
		// Mock apex action call with {@link SharedTestHelpers#apexMock} function. 
		// This function returns a mock function, which will be called 
		// when the apex action will be called by the component. 
		// If the mock function was called it means the "search" Apex action was called.
		// NOTE: it will work correctly if you use for Apex calls the {@link CustomElement#apex} only.
		let searchApexCall = apexMock(`search`);
		
		// The c-your-lwc component dispatches the onsuccess event. 
		// Use {@link JestHtmlElement#listen} function to listen for events which are dispatched by the component. 
		// This function returns a mock function, which will be called 
		// when the event will be dispatched by the component.
		// If the mock function was called it means the onsuccess event was dispatched.
		// So, we should validate that for this unit test the event will not be dispatched. 
		// We should assert that the mock function wasn't called.
		let listenerSuccess = component.listen(`success`);
		
		// We would like to emulate a Search button click. 
		// To achieve this we should query for the button element and then dispatch it's onclick event.
		// Use {@link JestHtmlElement#query} or {@link JestHtmlElement#queryAll} to query child components of the current component.
		// NOTE: "await" before the function call.
		let buttonElement = await component.query('lightning-button');
		// To call a handler function, use {@link JestHtmlElement#dispatch} function 
		// to dispatch the onclick event which the handler is subscribed to.
		// NOTE: "await" before the function call.
		await buttonElement.dispatch('click');
		
		// Once the button's onclick event is dispatched and the handler function is called, we can validate:
		// - Was "onsuccess" event dispatched? If yes, then what are the values of the parameters of the event?
		// - Was "search" Apex action called? If yes, then what are the values of the parameters of the call?
		// - Does the component shows the list of the found candidates to the user?
		// - Does the component shows success/error messages?
		
		// We don't expect the "search" Apex action will be called.
		expect(searchApexCall).not.toHaveBeenCalled();
		
		// We don't expect the "onsuccess" event will be called.
		expect(listenerSuccess).not.toHaveBeenCalled();
		
		// We don't expect the "div" with candidate names will be shown.
		// NOTE: "await" before the function call.
		let divElement = await component.query('div');
		expect(divElement).toBeNull();
		
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
		
		// The c-your-lwc component calls Apex action to do the search. 
		// So, we should validate that the Apex action was called and the parameters of the call are correct.
		// Mock apex action call with {@link SharedTestHelpers#apexMock} function. 
		// This function returns a mock function, which will be called 
		// when the apex action will be called by the component. 
		// If the mock function was called it means the "search" Apex action was called.
		// NOTE: it will work correctly if you use for Apex calls the {@link CustomElement#apex} only.
		let searchApexCall = apexMock(`search`, ``);
		
		// The c-your-lwc component dispatches the onsuccess event. 
		// Use {@link JestHtmlElement#listen} function to listen for events which are dispatched by the component. 
		// This function returns a mock function, which will be called 
		// when the event will be dispatched by the component.
		// If the mock function was called it means the onsuccess event was dispatched.
		// So, we should validate that for this unit test the event will not be dispatched. 
		// We should assert that the mock function wasn't called.
		let listenerSuccess = component.listen(`success`);
		
		// We would like to emulate the text input element is set with some value.
		// NOTE: "await" before the function call.
		let textElement = await component.query('lightning-input');
		textElement.value = `Some invalid text`;
		
		// We would like to emulate a Search button click. 
		// To achieve this we should query for the button element and then dispatch it's onclick event.
		// Use {@link JestHtmlElement#query} or {@link JestHtmlElement#queryAll} to query child components of the current component.
		// NOTE: "await" before the function call.
		let buttonElement = await component.query('lightning-button');
		// To call a handler function, use {@link JestHtmlElement#dispatch} function 
		// to dispatch the onclick event which the handler is subscribed to.
		// NOTE: 2nd parameter `{detail: {}}` is not required.
		// It is there just to show how we can pass additional details
		// required for the functional code to the event.
		// NOTE: "await" before the function call.
		await buttonElement.dispatch('click', {detail: {}});
		
		// Once the button's onclick event is dispatched and the handler function is called, we can validate:
		// - Was "onsuccess" event dispatched? If yes, then what are the values of the parameters of the event?
		// - Was "search" Apex action called? If yes, then what are the values of the parameters of the call?
		// - Does the component shows the list of the found candidates to the user?
		// - Does the component shows success/error messages?
		
		// We expect the "search" Apex action will be called.
		expect(searchApexCall).toHaveBeenCalled();
		// We expect the first parameter of the Apex action call has a text field with a value "Some invalid text"
		expect(searchApexCall.mock.calls[0][0].text).toBe(`Some invalid text`);
		
		// We don't expect the "onsuccess" event will be called.
		expect(listenerSuccess).not.toHaveBeenCalled();
		
		// We don't expect the "div" with candidate names will be shown.
		// NOTE: "await" before the function call.
		let divElement = await component.query('div');
		expect(divElement).toBeNull();
		
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
		
		// The c-your-lwc component calls Apex action to do the search. 
		// So, we should validate that the Apex action was called and the parameters of the call are correct.
		// Mock apex action call with {@link SharedTestHelpers#apexMock} function. 
		// This function returns a mock function, which will be called 
		// when the apex action will be called by the component. 
		// If the mock function was called it means the "search" Apex action was called.
		// NOTE: it will work correctly if you use for Apex calls the {@link CustomElement#apex} only.
		let searchApexCall = apexMock(`search`, `John Doe, John Tyson`);
		
		// The c-your-lwc component dispatches the onsuccess event. 
		// Use {@link JestHtmlElement#listen} function to listen for events which are dispatched by the component. 
		// This function returns a mock function, which will be called 
		// when the event will be dispatched by the component.
		// If the mock function was called it means the onsuccess event was dispatched.
		// So, we should validate that for this unit test the event will be dispatched 
		// and the parameters of the event are correct. 
		// We should assert that the mock function was called with correct parameters.
		let listenerSuccess = component.listen(`success`);
		
		// We would like to emulate the text input element is set with some value.
		// NOTE: "await" before the function call.
		let textElement = await component.query('lightning-input');
		textElement.value = `John`;
		
		// We would like to emulate a Search button click. 
		// To achieve this we should query for the button element and then dispatch it's onclick event.
		// Use {@link JestHtmlElement#query} or {@link JestHtmlElement#queryAll} to query child components of the current component.
		// NOTE: "await" before the function call.
		let buttonElement = await component.query('lightning-button');
		// To call a handler function, use {@link JestHtmlElement#dispatch} function 
		// to dispatch the onclick event which the handler is subscribed to.
		// NOTE: "await" before the function call.
		await buttonElement.dispatch('click');
		
		// Once the button's onclick event is dispatched and the handler function is called, we can validate:
		// - Was "onsuccess" event dispatched? If yes, then what are the values of the parameters of the event?
		// - Was "search" Apex action called? If yes, then what are the values of the parameters of the call?
		// - Does the component shows the list of the found candidates to the user?
		// - Does the component shows success/error messages?
		
		// We expect the "search" Apex action will be called.
		expect(searchApexCall).toHaveBeenCalled();
		// We expect the first parameter of the Apex action call has a text field with a value "John"
		expect(searchApexCall.mock.calls[0][0].text).toBe(`John`);
		
		// We expect the "onsuccess" event will be called.
		expect(listenerSuccess).toHaveBeenCalled();
		// We expect the first parameter of the event has a detail.text field with a value "John Doe, John Tyson"
		expect(listenerSuccess.mock.calls[0][0].detail.text).toBe(`John Doe, John Tyson`);
		
		// We expect the "div" will be shown.
		// NOTE: "await" before the function call.
		let divElement = await component.query('div');
		expect(divElement).not.toBeNull();
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

### 3.2. Explanation <a name="example-explanation"></a>

We should import helper functions from {@link SharedTestHelpers} library.
We know that we should create a LWC component, so we need a {@link
SharedTestHelpers#createComponent} function,
which is a powerful wrapper
for [createElement](https://developer.salesforce.com/docs/component-library/documentation/en/lwc/lwc.unit_testing_using_jest_create_tests)
function.
Also, we know that the LWC will call Apex code, so we need an {@link SharedTestHelpers#apexMock}
function.
Also, we know that the LWC will show success/error messages, so we need an {@link
SharedTestHelpers#getMessages} function.

***yourLwc.test.js***

```javascript
import {createComponent, apexMock, getMessages} from "shared";
import YourLwc from 'c/your-lwc';
```

We should identify what are the scenarios we should test.
According to [What should we cover](#what-should-we-cover) section we should test positive scenarios
as well as all possible errors.

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

See the unit [tests content](#yourLwc-test-js) comments above to get more details about the things
which should be tested. 
