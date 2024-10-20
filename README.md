<h1> <img src="https://craftypixels.com/placeholder-image/60x20/2f6faf/2f6faf"> Values </h1>
The Values package is my base library for almost everything I do. It provides Values (as opposed to Objects) which are simple, immutable objects. Values can only be created but never modified. This allows for a functional programming style and simplyfies systems, since much less state has to be maintained. Especially I like to see all structure and details (of complex values) at a glance and the ease of creating test values.

### Features
* **Values** as known in the functional world:  
**immutable** and consisting only of values or primitive types. Values do not have an identity and are equal when their elements are equal. A Value always forms a simple tree without loops or references. Instead of mutating objects, values are copied with new elements. Values are created with a constructor taking all elements as arguments.
* **Literal**: every value can print itself `asSource`. This produces a string which, when evaluated, yields the same value. This is great for looking at values and for using it in code. This is only possible in Smalltalk and a few other languages (Lisp).
* **Defaults**: elements can have a default, so that they can be omited in the constructor. This simplifies systems, since only the relevant data has to be given and the defaults are hidden.
* **Named Values**: some instances of a value class may be well known and should be use in the code instead of the generic constructor. This feature should be used with caution, since changes will break code using obsolete instances. Currently, only `ColorValue` uses this for `black`, `white` and the RGB and CMY colors.

## Motivation
The project grew out of the way I model simple objects which is strongly influenced by the functional and dynamic language Lisp. I recognized a pattern and codified support infrastructure for this style. To provide the facilities, I created the class `Value` as superclass for values. The feature why I implemented this was defaults for instance variables which meant that to needed 2^^(numberOfDefaultVariables) consturctor methods to cover all possibilities. That was far to labour intensive and error prone to do by hand - hence the Values package.

With Values codified, I could write a generic printing method for values which prints them as Smalltalk source which, when evaluated, results in the same value. Values are literal. This is very useful for example and test instances as well as for writing and reading them (files or sockets). But the nicest property of this is the you can see a value with all details at a glance.

Now, I use Values for more than 10 years and I cannot do without it anymore. Most of the classes I define are Values. Objects are used for the “moving parts”, the complex stuff. Since Values are so trivial and simple, I do not need to spend much time with them - they just work very reliable. Instead I can concentrate on complicated objects at the heart of the app. By sourcing out functionalities to Values, the systems become simpler.

## Using Values
To define a new Value class:

1. create a new subclass of Value without instance variables and “Subclass responsibilities” checked
2. edit the class method #localSpecification of the new class  
  a. add a pragma for each instance variable describing the variable
3. open the popup menu on the new class and select “add Value methods…”. This generates all methods.
4. edit the class method #example to provide a useful value

Done.

Now you have:

* a class with the specified instance variables
* an accessor for each variable with the same name
* an initializer with all parameters which sets up a fresh value
* a constructor taking all parameters and the sole caller of the initializer
* (2^^<numberOfDefaultVariables>) - 1 optional constructors
* an example

Lets have an example for example:

```
Person
    name: 'Christian Haider'
    sex: #male
    birthday: (Date d: 25 m: 6 y: 1960)
```

With it you can:
* create a value with
```
person := Person example
```

* ask for its parts
```
person name  "returns 'Christian Haider' "
```

* print it as code
```
person asSource  "returns
'Person
    name: ''Christian Haider''
    sex: #male
    birthday: (Date d: 25 m: 6 y: 1960)' "
```

* get the value from its code
```
person class evaluate: person asSource "returns a value equal to person"
```

* add fancy access methods like
```
weekdayAtBirth
  ^self birthday weekday
```

## Anatomy of a Value
A value has instance variables which can only contain values. The order of the variables is relevant and is used extensively. It is recommended to order the variables by importance.

Example: Class Person (subclass of Value)

1. name
2. sex
3. birthday

Each variable has a simple getter method with a comment indicating the class.

```
Person>>name
  "<String>"
  ^name
 
Person>>sex
  "<Symbol>"
  ^sex
 
Person>>birthday
  "<Date>"
  ^birthday
```

The variables are set all at once by an initalizing method which has all initial values as parameters. The object becomes immutable after initialization and all instance variables are effectively constants.

The parameter names are the concatenated name and its class. This prevents name clashes, is systematic and still readable.

```
Person>>initializeName: nameString sex: sexSymbol birthday: birthdayDate
  name := nameString.
  sex := sexSymbol.
  birthday := birthdayDate.
  self beImmutable
```

The one initializer is called by the constructor on the class side. The constructor returns a fully initialized immutable instance.

```
Person class>>name: nameString sex: sexSymbol birthday: birthdayDate
  | inst |
  inst := self new.
  inst initializeName: nameString sex: sexSymbol birthday: birthdayDate.
  ^inst
```

Every value class has an #example method. This serves for test cases and as a nice place to copy code from.

```
Person class>>example
  ^Person
    name: 'Christian Haider'
    sex: #male
    birthday: (Date d: 25 m: 6 y: 1960)
```

The initializer, the constructor, the example, the accessor methods and a printer method (not shown) are gernerated from a specification class method containing a list of pragmas defining each variable.

```
Person class>>localSpecification
  <constant: #name class: #{String}>
  <constant: #sex class: #{Symbol}>
  <constant: #birthday class: #{Date}>
```

### Defaults
The simple values above are not very interesting. But when you define defaults for some of the variables, Values become more useful. Lets add a nickname to the specification:

```
Person class>>localSpecification
  <constant: #name class: #{String}>
  <constant: #sex class: #{Symbol}>
  <constant: #bithday class: #{Date}>
  <optional: #nickname class: #{String} default: 'self name'>
```

After generating code with:
```
Person generateMethods
```

our example responds to #nickname:
```
Person example nickname   "returns 'Christian Haider' ">
```

and there is a new constructor available:
```
Person class>>name: nameString sex: sexSymbol birthday: birthdayDate nickname: nicknameString>
```

Now you can specify a nickname:
```
Person name: 'Christian Haider' sex: #male birthday: (Date d: 25 m: 6 y: 1960) nickname: 'Chris'>
```

If you use the same parameter as the default, it is ignored
```
Person name: 'Christian Haider' sex: #male birthday: (Date d: 25 m: 6 y: 1960) nickname: 'Christian Haider'>
```

because it is equal to
```
Person name: 'Christian Haider' sex: #male birthday: (Date d: 25 m: 6 y: 1960)>
```

## Get it
The Smalltalk code lives in the [Cincom Public Store](https://github.com/PDFtalk/.github/wiki/Getting-Started#5-set-up-access-to-the-public-store) as bundle **Values Project** which you can load into your [VisualWorks](https://www.cincomsmalltalk.com/main/products/visualworks/) image.

## References
I wrote a (scientific) dry paper about it and presented it at [ESUG](http://www.esug.org/) 2009 in Brest. I think that nobody understood it… :smiley:  
You can buy the [paper from the ACM](http://dl.acm.org/citation.cfm?id=1735957) or you can see the [draft](https://wiki.pdftalk.de/lib/exe/fetch.php?media=values:complexvaluespaper.pdf) of the paper with identical content on which the ACM does not have the copyright. The slides of the talk are [here](https://wiki.pdftalk.de/lib/exe/fetch.php?media=values:complexvaluesslides.pdf).
