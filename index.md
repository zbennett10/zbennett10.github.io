
# Javascript
---
# React
* Middleware
```Javascript
export default function({dispatch}) {
    return next => action => {
        if(typeof action.payload.then === 'function') {
            //resolve promise and call next with new action
            action.payload
                .then((response) => {
                    //send this new action back through entire middleware chain
                    return dispatch({type: action.type, payload: response.data});
                })
        }
        next(action);

    }
}
//never make an assumption regarding order when writing middleware
//almost always use dispatch when creating a new action
```
---
# Mocha/Chai
* test_helper.js setup

```Javascript
import jsdom from 'jsdom';
import jquery from 'jquery';
import TestUtils from 'react-addons-test-utils'; 
import ReactDOM from 'react-dom';
import chai, {expect} from 'chai';
import React from 'react';
import {Provider} from 'react-redux';
import {createStore} from 'redux';
import reducers from '../src/reducers';
import chaiJquery from 'chai-jquery';

//setup testing environment to run like a browser in the command line
//fake browser in command line (nodejs env)
global.document = jsdom.jsdom('<!doctype html><html><body></body></html>');
global.window = global.document.defaultView;
const $ = jquery(global.window); //create instance of jquery that only reaches out and touches our fake dom



//build 'renderCOmponent' helper that should render a given react class
function renderComponent(ComponentClass, props, state) {
  const componentInstance = TestUtils.renderIntoDocument(
    <Provider store={createStore(reducers, state)}>
      <ComponentClass {...props}/>
    </Provider>
  );
  return $(ReactDOM.findDOMNode(componentInstance)); //gives access to jquery-chai matchers - wrap component html with jquery
}


//build helper for simulating events
$.fn.simulate = function(eventName, value) {
  if(value) {
    this.val(value);
  }
  TestUtils.Simulate[eventName](this[0]);
}

//set up chai-jquery
chaiJquery(chai, chai.util, $);


export {renderComponent, expect};
```
---

# Immutable.js

---