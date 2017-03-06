# Javascript
---
# React
### * Middleware
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
//never make an assumption regarding order when writing middleware - almost always use dispatch when creating a new action
```
---

# Immutable.js

---