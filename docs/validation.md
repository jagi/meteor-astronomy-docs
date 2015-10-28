{{#template name="Validation"}}
The heart of validation is the `validate()` method. It can be called with different sets of arguments causing different effects. Here is the list of allowed arguments:

- `validate()` - validate all fields and stop after the first error
- `validate(false)` - validate all fields and do not stop after the first error
- `validate(fieldName)` - validate a single field
- `validate(arrayOfFieldsNames)` - validate multiple fields and stop after the first error
- `validate(arrayOfFieldsNames, false)` - validate multiple fields and do not stop after the first error

The `validate()` method returns `true` if validation succeeded and `false` if there was any field that failed validation. Here's an example:

```js
var user = new User();
if (user.validate()) {
  user.save();
}
```

In the first step, we've created a new user document. In the next line, we check a validity of the document. If it's valid then we save it.

**Validation on the server**

We should validate a document on both client and server. Let's say we have a form template that has some events and helpers that create a new document. We can validate the document and display validation errors in the form. However, we can't solely trust validation on the client. We should always send a given document to the server and repeat validation. We should also send errors back to the client if there are any. Let's take a look at the example of a Meteor method that performs validation and returns errors back to the client.

```js
Meteor.methods({
  '/user/save': function(user) {
    if (user.validate()) {
      user.save();
      return user;
    }

    // Send errors back to the client.
    user.throwValidationException();
  }
});
```

If the validation didn't succeed, then we send validation errors back to the client by throwing an error using the `throwValidationException()` method. Now, take a look at how the client can handle this exception.

```js
Template.Form.events({
  'submit form': function() {
    var user = this;

    Meteor.call('/user/save', user, function(err) {
      if (err) {
        // Put validation errors back in the document.
        user.catchValidationException(err);
      }
   });
  }
});
```

In the context of the `Form` template we have our newly created document that was filled with values coming from the form fields. We pass the `user` document as parameter to our Meteor method. In the callback function, we check if there are any server validation errors thrown. We then put these errors back in the document using the `catchValidationException()` method.

**Optional fields**

As described in the [Optional fields](#optional-fields) section, if a field is marked as `optional` then it won't be validated if its value is `null`.
{{/template}}
