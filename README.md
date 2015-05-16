#angular-logger
> Enhanced $log in AngularJS

* Enhances Angular's `$log` service so that you can define **separate contexts to log for**, where the output will be prepended with the context's name and a datetime stamp.
* Further enhances the logging functions so that you can **apply patterns** eliminatinging the need of manually concatenating your strings
* Introduces **log levels**, where you can manage logging output per context or even a group of contexts
* Works as a **complete drop-in** replacement for your current `$log.log` or `console.log` statements

Based on original post of:
<a href="http://blog.projectnibble.org/2013/12/23/enhance-logging-in-angularjs-the-simple-way/" target="_blank">Enhancing $log in AngularJs the simple way by Benny Bottema</a>

## Installing

angular-logger has optional dependencies on _[momentjs](https://github.com/moment/moment)_ and _[sprintf.js](https://github.com/alexei/sprintf.js)_: without moment you can't pattern a nicely readable datetime stamp and without sprintf you can't pattern your logging input lines. Default fixed patterns are applied if either they are missing.

### Bower

Will be implemented under [issue #10](https://github.com/pdorgambide/angular-logger/issues/10)

### Manually

Include _logger.js_, _[momentjs](https://github.com/moment/moment)_ and _[sprintf.js](https://github.com/alexei/sprintf.js)_ in your web app.

## Getting Started

1. Add `logger` module as a dependency to your module:

   ```javascript
   angular.module('YourModule', ['logger'])
   ```
2. Start logging for your context

   ```javascript
   app.controller('LogTestCtrl', function ($log) {
      var notMutedLogger = $log.getInstance('Not Muted');
      var mutedLogger = $log.getInstance('Muted');
   
      $log.logLevels['Muted'] = $log.LEVEL.OFF;
   
      this.doTest = function () {
         notMutedLogger.info("This *will* appear in your console");
         mutedLogger.info("This will *not* appear in your console");
      }
   });
   ```
   ([jsFiddle](http://jsfiddle.net/plantface/d7qkaumr/))

## Applying Patterns
### Datetime stamp patterns

**For all options, see [moment.js](http://momentjs.com/docs/#/displaying/)**

Will be implemented under [issue #14](https://github.com/pdorgambide/angular-logger/issues/14)

### Logging patterns

**For all options, see [sprintf.js](https://github.com/alexei/sprintf.js)**

The placeholders in the format string are marked by % and are followed by one or more of these elements:
* An optional number followed by a `$` sign that selects which argument index to use for the value. If not specified, arguments will be placed in the same order as the placeholders in the input string.
* An optional `+` sign that forces to preceed the result with a plus or minus sign on numeric values. By default, only the `-` sign is used on negative numbers.
* An optional padding specifier that says what character to use for padding (if specified). Possible values are `0` or any other character precedeed by a `'` (single quote). The default is to pad with *spaces*.
* An optional `-` sign, that causes sprintf to left-align the result of this placeholder. The default is to right-align the result.
* An optional number, that says how many characters the result should have. If the value to be returned is shorter than this number, the result will be padded.
* An optional precision modifier, consisting of a `.` (dot) followed by a number, that says how many digits should be displayed for floating point numbers. When used on a string, it causes the result to be truncated.
* A type specifier that can be any of:
    * `%` — yields a literal `%` character
    * `b` — yields an integer as a binary number
    * `c` — yields an integer as the character with that ASCII value
    * `d` or `i` — yields an integer as a signed decimal number
    * `e` — yields a float using scientific notation
    * `u` — yields an integer as an unsigned decimal number
    * `f` — yields a float as is
    * `o` — yields an integer as an octal number
    * `s` — yields a string as is
    * `x` — yields an integer as a hexadecimal number (lower-case)
    * `X` — yields an integer as a hexadecimal number (upper-case)

Old way of logging using `$log`:
```javascript
$log.debug ("Could't UPDATE resource "+resource.name+". Error: "+error.message+". Try again in "+delaySeconds+" seconds.")
// Could't UPDATE resource ADDRESS. Error: ROAD NOT LOCATED. Try again in 5 seconds.
```

New way of logging using enhanced `$log`:
 ```javascript
var logger = $log.getInstance("SomeContext");
logger.debug("Could't UPDATE resource %s. Error: %s. Try again in %d seconds.", resource.name, error.message, delaySeconds)
// Sunday 12:13:06 pm::[SomeContext]> > Could't UPDATE resource ADDRESS. Error: ROAD NOT LOCATED. Try again in 5 seconds.
 ```
 
You can even combine pattern input and normal input:
 ```javascript
var logger = $log.getInstance('test');
logger.warn("This %s pattern %j", "is", "{ 'in': 'put' }", "but this is not!", ['this', 'is', ['handled'], 'by the browser'], { 'including': 'syntax highlighting', 'and': 'console interaction' });
// 17-5-2015 00:16:08::[test]>  This is pattern "{ 'in': 'put' }" but this is not! ["this", "is handled", "by the browser"] Object {including: "syntax highlighting", and: "console interaction"}
 ```

[working demo](https://jsfiddle.net/plantface/qkobLe0m/)

## Managing logging priority

Using logging levels, we can manage output on several levels. Contexts can be named using dot '.' notation, where the names before dots are intepreted as groups or packages.

For example for `'a.b'` and `a.c` we can define a general log level for `a` and have a different log level for only 'a.c'.

The following logging functions (left side) are available:

logging function  | mapped to: | with logLevel
----------------- | --------------- | --------------
_`logger.trace`_  | _`$log.debug`_       | `TRACE`
_`logger.debug`_  | _`$log.debug`_       | `DEBUG`
_`logger.log*`_   | _`$log.log`_        | `INFO`
_`logger.info`_   | _`$log.info`_        | `INFO`
_`logger.warn`_   | _`$log.warn`_        | `WARN`
_`logger.error`_  | _`$log.error`_       | `ERROR`
`*` maintained for backwards compatibility with `$log.log`

The level's order are as follows:
```
  1. TRACE: displays all levels, is the finest output and only recommended during debugging
  2. DEBUG: display all but the finest logs, only recommended during develop stages
  3. INFO :  Show info, warn and error messages
  4. WARN :  Show warn and error messages
  5. ERROR: Show only error messages.
  6. OFF  : Disable all logging, recommended for silencing noisy logging during debugging. *will* surpress errors logging.
```
Example:

```javascript
// config log levels before the application wakes up
app.config(function (logEnhancerProvider) {
    logEnhancerProvider.loggingPattern = '%s::[%s]> ';
    logEnhancerProvider.logLevels = {
        'a.b.c': logEnhancerProvider.LEVEL.TRACE, // trace + debug + info + warn + error
        'a.b.d': logEnhancerProvider.LEVEL.ERROR, // error
        'a.b': logEnhancerProvider.LEVEL.DEBUG, // debug + info + warn + error
        'a': logEnhancerProvider.LEVEL.WARN, // warn + error
        '*': logEnhancerProvider.LEVEL.INFO // info + warn + error
    };
    // globally only INFO and more important are logged
    // for group 'a' default is WARN and ERROR
    // a.b.c and a.b.d override logging everything-with-TRACE and least-with-ERROR respectively
});


// config log levels after the application started running
run(function ($log) {
    $log.logLevels = {
        'a.b.c': $log.LEVEL.TRACE, // trace + debug + info + warn + error
        'a.b.d': $log.LEVEL.ERROR, // error
        'a.b': $log.LEVEL.DEBUG, // debug + info + warn + error
        'a': $log.LEVEL.WARN, // warn + error
        '*': $log.LEVEL.INFO // info + warn + error
    };
});
```

Alternative notation:

```javascript
$log.logLevels['a.b.c'] = $log.LEVEL.TRACE;
$log.logLevels['a.b.d'] = $log.LEVEL.ERROR;
// etc.
```
