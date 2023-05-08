# Angular reactive form common mistakes

Reactive form is a helpful tool to manage states of every field in a form. It is used to create and update a form, validate the value of each field and more. However, you might encounter some problem while working with it if you do not fully understand the concept.

## Know the concept

- Form Control: Form Control is a smallest component of a form, each Form Control represent for a field value.
- Form Group: Form Group is a collection of Form Control instances. In Form Group object, it will have all the same method that Form Control have. However, it also have it own property and method such as: "status" and "errors". All Form Control instances are located inside the "controls" property.
- Validators: Provide a list of common used validation for form fields (more at: https://angular.io/api/forms/Validators)

## Tricks

### After set validators for all fields, want to know if your form is validate?

```javascript
checkIfFormIsValid() {
    return this.formGroup.status;
}
```

### What happen with validator when I have some fields with initial value and some are not?

If we set an initial value for input fields by setting a value for input like this one:

```plaintext
<input
    type="email"
    class="add-user__input"
    placeholder="Place holder"
    (keyup)="enableSaveButton()"
    formControlName="email"
    value="currentEmail"
/>
```

Form Control will not receive the value of the input until user focus on the field so it make the form become invalid when web page initialize. The solution is easy that we should assign the value to Form Control when you create a Form Group instances

```javascript
this.formGroup = new FormGroup({
  email: new FormControl(this.currentEmail, [Validators.email]),
});
```
