---
layout: api
title: addParser
permalink: addParser/index.html
filename: api/addParser.md
---

<p class="alert alert-danger">
    <strong>Advanced Code Ahead!</strong>
    Adding new date parsers is a complex topic and you should only do this 
    if you are a confident developer, with a good understanding of types. It
    is recommended to read through the code to understand this better!
</p>

The `addParser` method extends the functionality of the  [set](/api/set) method
(which in turn extends the  functionality of the [constructor](/api/Tempus)
method).  If you require the ability to parse a custom date format through
Tempus,  this is the best way to do that.

When parsing a date, you have access to the current Tempus instance.   The date
will be set to the current time, so it is the parser's job to  run any date
manipulation functions to set the date based on the set of arguments.

### Parser Testability ###


Rather than having parsers blindly try to parse Tempus invocations, the
`addParser` method provides certain "testability" criteria.  The three criteria
are (in order): The amount of arguments Tempus has  been invocated with
(optional), the types of those arguments, and a  tester function to do more
complex custom tests such as  `RegExp`. See the following two points for more
details:

### Argument Type Matching ###

`addParser` requires new parsers to declare what type of  arguments they expect,
this is a great way to make quick assertions on  if the parser can sucessfully
parse the potential date. The first  argument type is required, so you __must__
declare what  type you expect the first argument to be. The list of types are:

<div class="row">
    <ul class="span2">
        <li>string</li>
        <li>number</li>
        <li>array</li>
        <li>object</li>
    </ul>
    <ul class="span2">
        <li>undefined</li>
        <li>global</li>
        <li>boolean</li>
        <li>function</li>
    </ul>
    <ul class="span2">
        <li>regexp</li>
        <li>date</li>
        <li>math</li>
        <li>error</li>
    </ul>
</div>

{% capture code %}
// Here I am adding a parser which accepts "object" as the first argument
Tempus.addParser( testerFn, parserFn, 'object' );

// My parser will never get run for this date, because the first argument is "number"
var tempus_object = new Tempus( 2008 );

// My parser WILL run for this date - the first argument type is "object"
var tempus_object = new Tempus( { year: 2008 } );
{% endcapture %}{% include code.html %}

You can express types for as many arguments as you expect, or only the 
first. For example

{% capture code %}
// This parser expects 3 arguments, 1st is 'number', 2nd: 'string', 3rd: 'number'
Tempus.addParser( testerFn, parserFn, 'number', 'string', 'number' );

// My parser will never get run for this date, because it has only 1 argument
var tempus_object = new Tempus( 2008 );

// My parser WILL run for this date - all three arguments match the types
var tempus_object = new Tempus( 2008, 'Dec', 2 );

// My parser will ALSO run for this date, as there is no strictness on 4th arg.
var tempus_object = new Tempus( 2008, 'Dec', 2, '12' );
{% endcapture %}{% include code.html %}

### Arguments Length ###

If you handle multiple types as arguments, but still have a minimum set 
of arguments, then you can express the 3rd argument as the minimum 
argument length. For example

{% capture code %}
// This parser expects 3 arguments, 1st is 'object', 2nd and 3rd are mixed
Tempus.addParser( testerFn, parserFn, 3, 'object' );

// My parser will never get run for this date: it has only 1 argument
var tempus_object = new Tempus( {} );

// My parser will never get run for this date: it has only 2 argument
var tempus_object = new Tempus( {}, function () {} );

// My parser will never get run for this date: 1st argument is not an 'object'
var tempus_object = new Tempus( 'String', {}, function () {} );

// My parser WILL run for this date: all three arguments, and the 1st is 'object'
var tempus_object = new Tempus( {}, function () {}, 2 );

// My parser will ALSO run for this date, as there is no strictness on max length
var tempus_object = new Tempus( {}, 'String', 1, /^regexp$/, function () {} );
{% endcapture %}{% include code.html %}

### Parse Ordering ###

Date parsers added through the addParser method are always put at the  bottom of
the [set](/api/set) stack. The order in which  parsers are added using the
`addParser` method, is the order  in which they are looped through.

If you plan on adding many parsers to your project, you should add them  strict-
most to loose-most, so that your strict functions have a chance  of executing.

### Test Function ###

For more complex tests a mandatory test function is provided which 
should return <var>true</var> if your parser function can parse 
this date, or <var>false</var> if it cannot.

### Putting it together ###

{% capture code %}
// This parser takes 1 argument, which is an object.
Tempus.addParser(
    // The testFN checks to see if every property in "obj" is a function of Tempus
    function (obj) { 
        for (i in obj) {
            // So if Tempus doesn't have a method we'll return false
            // meaning we cannot parse this
            if (typeof Tempus.prototype[i] !== 'function') return false;
        }
        // We got to here, so we can return true
        return true;
    }, 
    // Our parserFN will now parse the date from the object
    function (obj) {
        for (i in obj) {
            this[i](obj[i]);
        }
    },
    'object' // we expect 1 object.
);

// Using this parser...
var tempus_object = new Tempus( {

    year: 2012,
    month: 0, // Jan
    date: 1,
    hour: 6,
    minute: 12,
    second: 30

} );
{% endcapture %}{% include code.html %}