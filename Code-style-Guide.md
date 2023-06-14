Please follow the following style rules when writing code, in order to minimize unnecessary back-and-forth during code review. (Note that most, but not all, of the rules on this page are baked into the default linters and the pre-push hook.)

If you use [Sublime Text](http://www.sublimetext.com/), consider installing the SublimeLinter, [SublimeLinter-jscs](https://github.com/SublimeLinter/SublimeLinter-jscs) and [SublimeLinter-pylint](https://github.com/SublimeLinter/SublimeLinter-pylint) plugins, following the instructions on their respective pages.

## General
- Ensure that your code looks consistent with the code surrounding it.
- Strings should use single quotes (`'`) throughout Python and JavaScript.
- Prefer having comments on their own line (above the code that's being commented on), as opposed to next to a line. The exception is when you need to disable a pylint warning for a specific line.
- The last character in each file should be a newline. (If you're using Sublime, you can enforce this locally by adding `"ensure_newline_at_eof_on_save": true` to your user preferences file.)
- Avoid introducing `TODO (#XYZ): ...` comments in the files and instead try to do things correctly the first time. If you are going to add a TODO comment in any file then there needs to be (at minimum) a full comment and justification explaining what has been tried and what the issue is. The TODo should also reference an issue created on GitHub for thracking the problem.

## Design tips
- Avoid referencing elements of a list by a hardcoded index number, e.g. `item[0]`, `item[1]`. This is because the reader typically has no idea what is significant about the element index in question. If the values in the list are of different types, consider using a domain object instead to model the item being passed around.
- Avoid passing raw "dictionaries" (Python dicts or JS objects) between functions, because it's possible to add new fields to them midway through their lifecycle, which can get confusing for readers of the code. Use domain objects instead, since they have a fixed set of fields.
  - Similarly, if you are passing in two lists of variables and you require both lists to be the same length (because the elements need to correspond with each other), consider using one list of composite domain objects instead.
- Avoid redefining the same variable more than once. Use different names to represent different variables, since each variable (conceptually) stores a different thing.
- Functions that start with "get", or which have GET semantics, should, under no circumstances, update or delete anything. They should be safe to call and have no side effects.


## Python
- Consider using a frozenset or tuple to a list, if the data structure is not meant to be subsequently modified. This applies especially to constants.
- If you need to raise an Exception, just do `raise Exception` -- no need to define custom exceptions. We tend to use exceptions fairly sparingly, though.
- Do not use `str()` under any circumstances. If you need to cast ints/floats to strings, please use `python_utils.UNICODE()` instead. Avoid casting strings to other types of strings using str(), unicode(), etc. Also, there should be no need to prefix any string literals with b' or u', since all string literals in Python files are prefixed with `u'` by default (due to the import of unicode_literals at the top of the file).
- Otherwise, please follow the [Google Python style guide](https://github.com/google/styleguide/blob/gh-pages/pyguide.md). In particular:
  - There should be two empty lines before any top-level class or function definition.
  - It's OK for the initial documentation string to be more than one line long.
  - Prefer string interpolation over concatenation -- e.g. prefer: `'My string %s' % varname` to `'My string ' + varname`.
  - When indenting from an open parenthesis ('('), prefer indenting by 4 rather than indenting from the position of the parenthesis. e.g., prefer:

    ```
      my_function_with_a_really_long_name(
          'abc', 'def', None)
    ```

    over

    ```
      my_function_with_a_really_long_name('abc',
                                          'def',
                                          None)
    ```
  - Docstrings should start with """ and end with """. The content of each docstring should be left-aligned. For example:

    ```
      """Check whether an email message with the same recipient_id,
      email_subject and (cleaned) email_html_body has been sent in
      the last DUPLICATE_EMAIL_INTERVAL_MINS.
      """
    ```
    Docstrings should also contain `Args`, `Returns` and `Raises` whenever applicable in a method. For example:

    ```
    def function_name(arg1, arg2):
        """Brief description about the function.

        Args:
            arg1: type. Short description.
            arg2: type. Short description.

        Returns:
            type. Short description.

        Raises:
            TypeOfException: Short description.
        """
    ```
  - Never use backslashes to end a line. It's hard to tell whether they're escaping newlines, spaces, or something else. Use parentheses instead to break the line up, e.g.:

    ```
       my_variable = (
           my_very_long_module_name.my_really_long_function_name())
    ```
  - Be careful [not to use mutable objects](https://google.github.io/styleguide/pyguide.html?showone=Default_Argument_Values#Default_Argument_Values) as default values in the function or method definition. i.e., don't do things like `def foo(a, b=[]):`.

  - Imports should be in three groups: standard libraries, files within the Ambu codebase, and third-party files. Each group should be separated by a single newline. Within each group, imports should be organized alphabetically. If you have additional questions, feel free to reference the [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html#313-imports-formatting).

    ````

## JavaScript
_General note: We use the ES2017 standard for our JavaScript/TypeScript code.

- We use extra parentheses if a statement breaks across multiple lines, similar to Python. In particular, when code in '(...)' or '[...]' spans more than one line, make a line break after the opening parentheses or bracket.
- The indentation is always 2 spaces.
- Try to start only function names with verbs to help distinguish them from variables. Conversely, do not start variable names with verbs.

   For example:

   - For a boolean variable to check if a card is displayed:
        - Correct: `cardIsDisplayed`
        - Wrong: `isCardDisplayed`

   - For a function to check if a card is displayed:
        - Correct: `isCardDisplayed()`
        - Wrong: `cardIsDisplayed()`

- The dependencies mentioned in strings and functional parameters of controllers, directives and factories should be in the following manner: dollar imports (e.g. ```$log, $scope``` etc.), regular imports (e.g. ```ContextService, PageService``` etc.), and constant imports (e.g ```COLLECTION_TAGS, DELETE_COLLECTION``` etc.) all in sorted order.

    For Example:
    ```javascript
    ambu.thing('ThingName', [
      '$sortedDollarImports', 'SortedRegularImports',
      'SORTED_CONSTANT_IMPORTS',
      function(
          $sortedDollarImports, SortedRegularImports,
          SORTED_CONSTANT_IMPORTS) {
        // The implementation of `ThingName`.
      }]);
    ```
- For asynchronous functions that return a promise, use the following convention:
  - At the function declaration, use the keyword `async` (see [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)).
  - Add 'Async' to the function name. example:
    ```ts
    const getUserInfoAsync = async function() {
      return new Promise(resolve => {
        setTimeout(function() {
          resolve("something");
        }, 2000);
       });
    }
    ```
- For functions or variables that are private and that should not be exposed outside their immediate controller/service, prefix their names with an underscore (`_`) and add the `private` keyword.
- Keep line lengths to at most 80 characters (with the exception of lines containing URLs, which are allowed to have a length of greater than 80 characters).
- Declare a variable before usage. For instance:

  **Wrong usage:**
  ```javascript
  exampleVar = true;
  if (someCondition) {
    exampleVar = false;
  }
  ```
  **Right usage:**
  ```javascript
  var exampleVar = true;
  if (someCondition) {
    exampleVar = false;
  }
  ```
- All loop variables should be declared. For instance:

  **Wrong usage:**
  ```javascript
  for (item in itemList) {
    ...
  }
  ```
  **Right usage:**
  ```javascript
  for (var item in itemList) {
    ...
  }
  ```

- Always initialize a variable at declaration. If you do not want a specific value at declaration, initialize the variable with a null value. For instance:

  **Wrong usage:**
  ```javascript
  var person;
  if (someCondition) {
    person = 'name';
  }
  ```
  **Right usage:**
  ```javascript
  var person = null;
  if (someCondition) {
    person = 'name';
  }
  ```
- Do not overwrite the variable with a different type. Instead create a new variable whenever you have a different use case. For instance:

  **Wrong usage:**
  ```javascript
  var person = {
    name: 'name',
    schoolName: 'school name'
  };
  ...
  person = {
    name: 'name',
    officeName: 'office name'
  };
  ```
  **Right usage:**
  ```javascript
  var personForSchool = {
    name: 'name',
    schoolName: 'school name'
  };
  ...
  var personForOffice = {
    name: 'name',
    officeName: 'office name'
  };
  ```

## CSS
- Do not include units if the value is 0. E.g. `margin-left: 0` instead of `margin-left: 0px`.
- Within each CSS rule, attributes should be alphabetized (e.g. 'height' before 'margin' before 'top'). This makes it easy to find the value of an attribute if there are lots of them.
- Avoid using `!important` as much as possible.
- For colours, use hex values (like "#012345") or rgb(a) values, instead of names (like "white"). When using hex colour codes, try to use the 3 char version if possible (see [example](https://google.github.io/styleguide/htmlcssguide.html#Hexadecimal_Notation)).
- If the CSS class is ambu-specific, prefix it with `ambu-`. This helps distinguish it from CSS classes used by other third-party libraries.

----

### Note for Sublime Text users

If you use Sublime Text, the following settings may be useful for your "Preferences.sublime-settings -- User" file (go to Preferences > Settings)

```
{
    "ensure_newline_at_eof_on_save": true,
    "font_size": 9,
    "highlight_line": true,
    "rulers":
    [
        80
    ],
    "shift_tab_unindent": true,
    "spell_check": true,
    "tab_size": 4,
    "translate_tabs_to_spaces": true,
    "trim_trailing_white_space_on_save": true,
    "update_check": false
}
```