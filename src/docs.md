# Documentation

## Form

A `Form` in Astro Forms maintains a collection of rows, responds to row changes, and validates rows.

### Creating a Form

To create a form, simply subclass `Form`.

```swift
import AstroForms

class LoginForm: Form {
	// ...
}
```

As `Form` is a subclass of `UIView`, just add the class in Interface Builder to add it to your `UIViewController`.

### Adding a Row

Forms have `add(_:)` and `insert(_: at:)` instance methods for adding rows to a form.

```swift
public func add(_ row: AnyRow)
```

```swift
public func insert(_ row: AnyRow, at index: Int)
```

To add a row, simply call the add/insert method at any point, for example:

```swift
import AstroForms

class LoginForm: Form {
	
	enum LoginFormTag: RowTag, Equatable {
		case fullName
	}

	override func awakeFromNib() {
		super.awakeFromNib()

		let fullNameRow = TextFieldRow(tag: LoginFormTag.fullName) {
            $0.view.label.text = "Full Name"
            $0.view.textField.placeholder = "Enter your name..."
        }

        // Adding the row here
        add(fullNameRow)

	}

}
```

Once added, these rows are available via the `rows` property, an instance variable of type `[AnyRow]`.

### Row Tags in Forms

In the above example, you'll notice when creating `fullNameRow` we used a `LoginFormTag` enumerated type defined in `LoginForm`. 

Tags are an important concept in Astro Forms:
- Tags provide a unique ID for every `Row`
- Tags can be dynamic using associated values
- Every `Row` must have a tag
- Every `Form` has an enum that declares it's set of row tags.

Create tags by implementing `RowTag` and `Equatable`. For example a set of tags might be:

```swift
enum LoginFormTag: RowTag, Equatable {
	case email, password, submit
}
```

A `Form`'s row tags type must implement `Equatable` so they can be compared with each other.

### Finding a Row

Because every row has a tag you can access them later with the `findRow(tag:)` instance method:

```swift
let fullNameRow: TextFieldRow? = self.findRow(tag: LoginFormTag.fullName)
```

If rows are rendered dynamically, associated types provide a convenient way to declare tags for rows that may be generated base on conditions.

```swift
enum LoginFormTag: RowTag, Equatable {
	case dynamicTag(String)
}

// Later, inside an instance method:

for i in 1...4 {
	add(TextFieldRow(tag: LoginFormTag.dynamicTag("address-line-\(i)")))
}

let foundRow: TextFieldRow? = self.findRow(
	tag: LoginFormTag.dynamicTag("address-line-4")
)
```

::: tip
Only use dynamic row tags when absolutely required to avoid runtime errors from mistyped strings.
:::

### Removing a Row

A Form contains an remove function that will remove a row, and any associated views from the `Form`.

```swift
public func remove(at index: Int) 
```

### Responding to Row Changes

A `Form` receives all Row updates in a overrideable method:
```swift
func rowUpdate(type: RowUpdate, row: AnyRow)
```
There are several RowUpdate types, that map to common validation requirements, however can be used for any arbitrary form update (for example show/hide). 

These are:

- **live**
	`rowUpdate` called every time the row is changed
- **onResignActive**
	`rowUpdate` called every time the row is blurred
- **onResignActiveAfterChange**
	`rowUpdate` called on blur after the row has changed at least once
- **regular**
	`rowUpdate` called every time the row is changed after the row has been blurred at least once, and every time it is blurred


By example, to update the disabled state of a row that contains a button on every row change (ButtonRow):

```swift
override func rowUpdate(type: RowUpdate, row: AnyRow) {

    guard let tag = row.tag as? LoginFormTag else { return }
    
    switch type {
        case .live:
            
            // When any row is updated, update button row validity
            guard let buttonRow: ButtonRow = findRow(
                tag: LoginFormTag.submit
            ) else {
                return
            }
            
            updateButtonEnabledStateUI(row: buttonRow)
        
        default: break
    }
    
}
```

## Row

A Row in Astro Forms is a class that manages any `UIView` that is inside a `Form`. This is commonly a view with an input like a `UITextField`, but it also might just be a view with your company logo, some images, a map - anything.

### Creating a Row

To be added to a `Form` a `Row` only needs implement the `Row` protocol. The requirements for this protocol are:
- To define a `UIView` type associated with the `Row` 
- Add an property to refer to the `UIView` for this row
- Add a `tag` property to store this row's `RowTag` 

Here's an example for a row that shows a company logo:
```swift
class CompanyLogoRow: Row {

	// Define the view type
	associatedtype View = CompanyLogoView

	// The `UIView instance the user will see
	var view: View

	// Every Row must have a RowTag
	var tag: RowTag

	init(tag: RowTag, config: ((CompanyLogoRow) -> Void)? = nil) {
		let view: View = View.fromXib()
		self.view = view
		self.tag = tag
		config?(self)
	}

}
```

In this example:
- The `CompanyLogoRow` includes a view, tag, and a basic initialization method
- A `CompanyLogoView` (defined somewhere else), is just a UIView
- A Xib for the `CompanyLogoView` 

That's it, this row is ready to be added to a `Form` with the `add(_:)` instance method.


### Value Rows

A row can have a value of any type, simply by implementing the `ValueRow` protocol. This protocol contains a `value` instance property for getting and setting the value from the underlying `UIView`. In the below example this is a Boolean based on a `UISwitch` state, for a `UITextField` based row, it would get and set the String `text` property on the field. 

This protocol also contains some instance properties and methods (with default implementations) for delegating view updates to the `Form`.

By example, a SwitchRow (containing a `UISwitch`) with a value:

```swift
import UIKit
import AstroForms

class SwitchRow: Row, ValueRow {

	// ValueRow protocol requirements

	typealias Value = Bool
    
    var valueHasStartedEditing: Bool = false
    
    var valueHasChanged: Bool = false
    
    var valueHasEndedEditing: Bool = false
    
    var value: Value {
        
        get { return view.switch.isOn }
        
        set { view.switch.setOn(newValue, animated: false) }
        
    }

    // Row protocol requirements

    var tag: RowTag

    var view: SwitchRowView
    
    init(tag: RowTag, config: ((SwitchRow) -> Void)? = nil) {
    	...
    }
    
}
```

In implementing `ValueRow` for this `SwitchRow`:

1. First, we set the Value typealias to a Bool (for when the switch is on or off)

2. Next, we create a virtual property that gets / sets the switch value, defined in the `SwitchRowView`

3. Finally, there are three variables `valueHasStartedEditing`, `valueHasChanged`, `valueHasEndedEditing`. These are to manage the validation state for this row. To satisfy the protocol, just assign these default values of `false`.

Now, in the `Form`, we have access to the row, with a Bool value:

```swift
// Inside the Form subclass
let row: SwitchRow? = findRow(tag: MyFormTag.mySwitchRow)

let value: Bool? = row?.value
```

### Responding to View Changes

When the value of a `ValueRow` changes, the `ValueRow` automatically passes this change information up to a `Form`. This update is then available in the Form's `rowUpdate(type: RowUpdate, row: AnyRow) `. 

However, given a Row's view can be any `UIView` implementation, these changes could take many shapes. For example, a `UISwitch` notifies a `UIView` of updates using the target-action pattern for`UIControl.Event.valueChanged`, however a UITextField uses a combination of delegate methods and the target-action pattern for `UIControl.Event.editingChanged`.

For this reason, a `ValueRow` normalizes all changes through several instance methods to call from your view.

```swift
// The value of the view has changed
func valueDidEdit()

// The use has started editing the view, relevant for UITextField etc...
func valueDidStartEditing()

// The user is done editing. This is the correct method for views that have 
// instantly changing values too - like a `UISwitch` or `UIButton`
func valueDidEndEditing()

```

Call these on a row from the row's UIView implementation, for example:

```swift
class SwitchRowView: UIView {
    
    @IBOutlet weak var `switch`: UISwitch!
    
    @IBOutlet weak var label: UILabel!
    
    // Store a weak reference to a row so you can call methods on it. 
    // This can be assigned during Row initialization.
    weak var row: SwitchRow?
    
    override func awakeFromNib() {
        super.awakeFromNib()
        
        `switch`.addTarget(
            self,
            action: #selector(switchValueChanged(_:)),
            for: .valueChanged
        )
        
    }
    
    @objc func switchValueChanged(_ sender: UISwitch) {
    	// Call the row's valueDidEndEditing() method, 
    	// so it can pass this event up for validation etc.
        row?.valueDidEndEditing()
    }
    
}
```

### Focusable Rows

 Astro Forms exposes a `FocusableRow` protocol to:
 - Enable switching between focusable rows (like text inputs) with next/previous buttons
 - Allow rows to define custom a `focusRect`. A focusRect is the `CGRect` that you want form's `UIScrollView` to ensure is visible on screen when an input is focused. This is useful for ensuring for example, both email and password are visible above the keyboard when the email field is focused. No more hunting behind the keyboard for fields.

 To implement this protocol:

 1. Use a block to return the `UIResponder` that should be focused when a `Row` is asked to take focus - for example when the "Next" button on the keyboard is tapped on the previous `Row`.

```swift
 class TextFieldRow: Row, ValueRow, FocusableRow {
 	// ...
    var focusElement: UIResponder { return view.textField }
    // ...
}
```

2. Provide a default implementation for focusRect to satisfy the protocol. If this is nil, the `Form` will focus the `CGRect` for the whole `Row`. It's almost always best to set this to nil, and provide custom focusRect's when adding rows (so you can reference other rows etc).

```swift
var focusRect: () -> CGRect? = { return nil }
```

Later when adding rows in your Form, override this block:

```swift
emailRow.focusRect = {
    CGRect(
        x: emailRow.baseView.frame.origin.x,
        y: emailRow.baseView.frame.origin.y,
        width: emailRow.baseView.frame.width,
        height: emailRow.baseView.frame.height + passwordRow.baseView.frame.height
    )
}
```

Now when the email row `UITextField` is the `firstResponder`, both the emailRow and passwordRow will be visible above the keyboard.

### Helpers

A Helper is a reusable `UIView` that a Row manages that appears after the Row's `UIView` in a `UIStackView`. A clear use case for a Helper is error message views that need to show and hide on validation errors. The interface between a `Row` and a helper is the ability to show and hide the helper.

This is done through two methods:

```swift
func showHelper<T: UIView>(
	viewType: T.Type, 
	animated: Bool, 
	config: ((T) -> Void)?
)

func hideHelper(
	animated: Bool = false, 
	callback: (() -> Void)? = nil
)
```

By example, showing an error using the example helper `ErrorView` is as simple as calling the show method, and configuring the view:

```swift
guard validate(row: row, ValidationRule.required) else {
    row.showHelper(viewType: ErrorView.self, animated: true) { errorView in
        errorView.label.text = "This field is required"
    }
    return
}
```

The only unusual thing here is the use of the class type instead of an existing instance as a parameter when showing a helper. This is to avoid the need to store instances of helper views - Astro Forms will manage replacing the view if necessary, or reusing the existing view if it is of the same type.

## Validation

Validation is handled by several instance methods available on `Form`, and does not enforce any particular architecture. 

However, as in the example project, the logical place is to handle validation in methods called from `func rowUpdate(type: RowUpdate, row: AnyRow)` on the form.

### Validation Methods

There are several validation methods available on a form for validating input, with method signatures all beginning `validate`.

### Basic Validation

The simplest kind of validation takes a variadic parameter of blocks that return boolean values. If all the blocks return true, then the result is true. Each block is passed the row value as it's parameter.

```swift
   func validate<R: ValueRow>(
        row: R,
        _ rules: ValidationRuleBlock<R>...) -> Bool
```

By example, consider a `StringRow` with a value of string:

```swift
let isValid: Bool = form.validate(
    row: stringRow,
    { $0 == "astro" }, // $0 is the value of the stringRow
    { $0.count == 5 }  // typed to String based on the Row's Value associatedtype
)
```
Here, two blocks are passed in, one comparing the `StringRow` value to "astro", and another counting the characters.

### Validation Messages

If there are multiple rules being used in a chain, it can be helpful to use a tuple with messages, so rather than just know the chain failed overall, you also have a message for the specific rule that failed.

```swift
func validate<R: ValueRow>(
        row: R,
        _ rules: ValidationRuleMsgBlock<R>...) -> (Bool, String?)
```

By example, the following will return `(false, "The character count must be correct")`, for the input string "astro":

```swift 
let isValid = form.validate(
    row: stringRow,
    ({ $0 == "astro" }, "The string should equal astro"),
    ({ $0.count == 10 }, "The character count must be correct")
)
```

### Validation List

In some situations, you may want to return a full list of messages and true / false for those that pass or fail. A common use case for the structure is a password field with many rules, and you want to show the user which ones they have successfully filled, and those they have yet to pass.

```swift
    func validateList<R: ValueRow>(
        row: R,
        _ rules: ValidationRuleMsgBlock<R>...) -> [(Bool, String?)]
```

By example, with a password list:

```swift

// Where stringRow.value == "astroforms"

let passwordValidityList = form.validateList(
    row: stringRow,
    ({$0 == "*" }, "The password should contain a *."),
    ({$0.count > 6}, "The password should have more than 6 characters."),
    ({$0.count < 30}, "The password should have less than 30 characters.")
)

// Returns:
// [(false, "The password should contain a *."),
// (true, "The password should have more than 6 characters."),
// (true, "The password should have less than 30 characters.")]

```

### Named Validation Rules

Inline blocks are convenient however not very reusable. Astro Forms contains a factory of `ValidationRule` methods, these can be mixed with inline blocks. This can be extended with your own methods via an extension.

```swift
let isValid: Bool = validate(
	row: stringRow,
	{ $0.count > 0 },
	ValidationRule.isEmail
)
```

## Theming

You can create themes in a type safe way easily in Astro Forms for any kind of property. Themes are defined on a form and inherited by views automatically, although any given instance of a row can override the theme to allow multiple themes to be used in the same form.

Theming in Astro Forms can be used independently of forms too, you can apply the `Themeable` protocol you create to any `UIView`, simply by implementing the `theme` instance property and updating themes when you set it. If the given `UIView` is in a subview of any `Form`, it won't inherit a theme automatically so you will need to define it on each `UIView`. 

Here an example `UIImageView` subclass that implements a different backgroundImage per theme in the Astro Forms example project:

```swift
class ThemeableImageView: UIImageView, Themeable {
    
    var theme: AstroTheme? {
        didSet { updateTheme() }
    }
    
    func updateTheme() { self.image = image(.formBackground) }
    
}
```

### Creating a Theme

To create a theme, you need to create your own `Themeable` protocol, composed of various theming protocols that define themable behaviour (color, size) and also those that define how to apply the theme to your `UIView` subclasses (`Form` or `Row` views). These are constrained to concrete enumerated types implementing:
- The number and names of the different themes in your project (for example: light, dark etc.)
- The properties that should be applied within the themes (primaryColor, smallMarginSize etc.)

This then exposes easy to use methods with a convenience syntax in any UIView that inherits your new protocol.

#### Included Themable Protocols

The color function is exposed by `ThemeableColorTraits`:
```swift
func color(_ requirement: ThemeColorType) -> UIColor
```

An example usage is: `textView.tintColor = color(.primaryTint)`,

The image function is exposed by `ThemeableImageTraits`:
```swift
func image(_ requirement: ThemeImageType) -> UIImage
```

An example usage is: `imageView.image = image(.formBackground)`.

In both these cases, a color and image will be returned for the currently active theme.
