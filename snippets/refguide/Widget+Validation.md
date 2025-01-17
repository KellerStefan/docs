### Validation Type

This property indicates whether this widget value should be validated and, if so, how. The possible options are no validation, a predefined validation, or a custom validation.

The possible values of a predefined validation are the following:

* Required – all data types
* E-mail – string
* Positive number – decimal, float, integer, long
* Date in the future – dateTime
* Date in the past – dateTime

Custom validations are expressions that follow the [Microflow expression](expressions) syntax. Both `$currentObject` and `$value` are in a scope that refers to the current object and the current member value, respectively.

When a validation is set and it fails for this widget, a message will be shown when the user selects **Save**.

*Default value:* (none)

### Validation Message

This property determines the message that is shown to the user if widget validation is enabled and has failed. This is a translatable text (for more information, see [Translatable Texts](translatable-texts)).

<div class="alert alert-info">

For example, if an address field is required, the validation message for the text box of the address could be something like, "The address is required."

</div>
