# Astro Forms Docs

## Form

The `Form` class is the only abstract class in Astro Forms, and it inherits directly from `UIView`. All Astro Forms start by subclassing `Form`. Place it inside a `UIViewController` via Interface Builder or code.

The `Form` itself is a collection of rows, and it should handle all logic related to:
- Accessing the value of rows
- Updating row view state, including showing / hiding and validation
- Collecting and validating the row input data to be passed to the `UIViewController`.

::: warning
Rows should never communicate directly with one another. Rows should implement state change delegate methods to notify the `Form` of their own updates, but leave any logic to the `Form`.
:::


## Row Tags

A row tag is a unique id for each row. Every `Row` must have a tag. Row tags are an `enum`, which provides good type safety and prevents errors from magic string access.

To create row tags for a form, start by extending enum `RowTag`.

```swift
class LoginForm: Form {
	enum LoginFormTag: RowTag {
		case 
			login,
			password,
			rememberMe
	}
	...
}
```




### Finding a Row by Tag

The `Form` class has a find function that lets you find a row by tag. You must know the type of field you are accessing to gain type information. The `findRow(tag: RowTag)` method returns an optional value to handle the case that the row cannot be found.

```swift
let passwordField: UITextField? = self.findRow(tag: LoginFormTag.password)
```

### Dynamic Row Tags

Many forms require dynamic fields based on some server side condition. Accordingly, we also require dynamic row tags, which can be generate and accessed predictably. Swift enums provide an easy way to handle this with associated values.

```swift
enum LoginFormTag: RowTag {
	case 
		login,
		password,
		rememberMe,
		dynamicField(String)
}
```

Using associated values, we can access dynamic row tags in the regular way:

```swift
let otherField: UITextField? = self.findRow(tag: LoginFormTag.dynamicField("row-1"))
```

::: tip
Only use dynamic row tags when required so that you keep the compiler protection that helps prevent errors.
:::

