# Prerequisites

You should understand [Mock Functions](https://jestjs.io/docs/mock-functions).

# Test Scope

### What should we cover <a name="what-should-we-cover"></a>

- Positive scenario.
- Negative/Errors scenario. Each error should be covered.

### What should we call from outside

- Life-cycle callbacks. The callbacks _connectedCallback_ and _renderedCallback_
  are called automatically during the LWC component creation.
- Event handlers.
- Methods and fields with @api.

### What should/can we assert

- HTML - **should**
- Fields with @api - *can*
- External/shared data - **should**
- Apex calls - **should**
    - How many times the apex method was called?
    - What were the arguments for the apex call?
- Dispatched events - **should**
    - How many times the event was dispatched?
    - What were the arguments of the dispatch?
- Messages (success, error, info, warning) - **should**

# Example

### Requirement

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
		
		const candidateNames = await this.apex(search, {text: text});
		
		this.isFound = !!candidateNames;
		if (this.isFound) {
			this.addSuccess(`Candidates were found.`);
			this.text = candidateNames;
			
			this.dispatchEvent(
				new CustomEvent('success', {detail: {candidateNames: candidateNames}}));
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
		// We expect the first parameter of the Apex action call has a value "Some invalid text"
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
		// We expect the first parameter of the Apex action call has a value "John"
		expect(searchApexCall.mock.calls[0][0].text).toBe(`John`);
		
		// We expect the "onsuccess" event will be called.
		expect(listenerSuccess).toHaveBeenCalled();
		// We expect the first parameter of the event has a value "John Doe, John Tyson"
		expect(listenerSuccess.mock.calls[0][0].detail.text).toBe(`John Doe, John Tyson`)
		
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

### Explanation

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
