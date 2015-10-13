# Forms, user input, and methods

One of the hallmarks of an "app" compared to a "website" is that apps are built around user input, interaction, and collaboration. One of Meteor's best features is a great building block for user inputs: the Meteor method.

The methods in your Meteor app describe the surface area of all inputs and database changes your app will accept. Think of it as enabling the create, update, and delete API of a typical REST backend, but with some extra features to enable a great user experience for your users.

## Defining a method with optimistic UI and validation

Several features are crucial to make your app feel modern and user-friendly.

1. **Optimistic UI**: When appropriate, the user interface should respond immediately. For example, in a chat app, you would want the chat message to show up immediately to give the user feedback on their action. However, this is not always appropriate - for example, you wouldn't want to do the same for a bank transaction; the appropriate response there would be to show a processing screen until the transaction is confirmed.
1. **Instant input validation**: Users should get immediate feedback about their inputs. They should never end up in a situation where they spent 10 minutes filling out a form only to get an error message - it should be clear what the form expects and feedback should be instant.

You have probably written several methods before, especially if you've completed the Meteor getting started tutorial. Now let's go over how to make a simple method that enables the features above.

### Reality

```js
Meteor.methods({
  // XXX method naming?
  "/widgets/new"() {

  }
});
```

XXX call with throwStubExceptions :[

```js
try {
  Meteor.call("/widgets/new", {
    throwStubExceptions: true
  }, (err, res) => {
    if (err) {
      // Handler server-side error
    } else {
      // Success!
    }
  });
} catch (validationError) {
  // Handle client-side validation error
}
```

### Pseudocode below

```js
// Defining validated method
const myMethod = Methods.define({
  // Name needs to be unique? can we get around this
  // Actually if you want to be able to call this method from a native
  // app or something, it needs a well-defined global name.
  name: "/widgets/new",

  // All args are keyword args
  // Also, note that schema is only the stuff that the client knows how
  // to validate; you might need to add additional server-side validation
  // logic in the server-side implementation.
  schema: {
    name: {
      type: String,
      required: true,
      maxLength: 10,
      isNotFive: (val) => {
        if (val === '5') {
          // I would like to not have UI strings here
          return "Value is 5";
        }
      }
      // ...
    },
    content: {
      type: String,
      maxLength: 500,
      // ...
    }
  },

  // Can also be split into simulation() and server() or something?
  run({name, content}) {
    Widgets.insert({
      name,
      content,
      createdAt: new Date()
    });
  }
});
```

```js
// Or, you can just `require` the original object?
const myMethod = Methods.getMethod("/widgets/new");

// Usage
myMethod.getValidationErrors({
  name: "Hello",
  content: "Stuff"
});

// For a specific field
myMethod.getValidationErrorsForField("name", "Hello");

// Returns `null` if there are no errors, so you can do
// const errors = myMethod.getValidationErrors(args);
// if (! errors) { ... }

// If there are errors, returns JS object of a form similar to
{
  name: {
    value: "Hello",
    type: true, // Let's not hardcode error messages...
  }
}

myMethod.call({
  name: "Hello",
  content: "Stuff"
}, (err, res) => {
  // Get the `err` result here even if it's a client-side error, and if
  // there is a client side error it is never called on the server
  // Validation error has the data structure above attached
});

// Available on the server only
myMethod.callWithUserId(userId, {
  name: "Hello",
  content: "Stuff"
}, callback);
```

1. Definitely solves client-side realtime validation via `getValidationErrors` - and you can do it per-field if you need to
2. You could auto-generate forms via the `schema` property
3. Saving intermediate inputs... you could have a per-user collection and have each form have a unique ID; then you have a `submit` button that actually runs the method validation?
4. Does this help for uploading files at all?
5. What if you want to do different validation on the client and server? I guess the validation data is only stuff that is shared between client and server...

The ideal solution for forms would probably have a couple components:

1. A form generator
2. A form reader - parse the form and get an object that can be validated, reactively when user input happens; so that we can display realtime errors or save the user's progress
3. The validating method infrastructure above
