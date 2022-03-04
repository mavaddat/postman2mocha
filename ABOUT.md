# A tool to translate Postman collections into Mocha/Chai test suites

Postman is a great tool for getting up and running quickly with API testing. Its user interface is easy to use for beginners and makes code-free exploration easy. Unfortunately, it also has its drawbacks:

- There's no way to import modules from NPM
- Modularizing common code in Postman is not straightforward
- Sharing exported collection files among a team can result in hard-to-resolve merge conflicts

Another popular toolset that can be used for API testing is the JavaScript test runner and assertion library combo [Mocha](https://www.npmjs.com/package/mocha) and [Chai](https://www.npmjs.com/package/chai). Translating a large number of test cases from Postman to Mocha can be tedious and time consuming so here we outline a tool that can be used to automate the process.

## How the translation works

The overall approach we took was to simply map the JSON in the Postman collection to a JavaScript abstract syntax tree (AST) and serialize it. Making this a bit more tricky is the existence of embedded pre-request scripts and post-request test scripts. Rather than re-implement the Postman API, we used pattern matching to convert Postman code into (hopefully) functionally idiomatic Mocha / Chai code.

Our pattern matcher works by finding and replacing fragments of JavaScript syntax tree written as rules. We felt that this would be quicker and more concise than writing functions to find and generate the code for each substitution we wanted to make. Since there's no way (at least none we could find) to parse placeholders with Babel, we're using uppercase identifiers to match branches of the syntax tree. For example, one of the rules to replace calls to `pm.expect` looks like this:

``` js
'pm.expect(X).A': 'expect(X).A',
```

### Parsing Postman components

We're using the `babel-template` library, which conveniently does allow placeholders to be specified in all caps, to generate the new AST fragments. We're also using `babel-traverse` to traverse the source tree with a single callback which calls the pattern matcher. This ensures that the rules will be matched from most-to-least specific, since traversal happens top-down. Rather than figure out a way to match multiple parameters or arbitrary length chains, we just wrote multiple versions of each rule to match all possibilities. Since the number of possible combinations is fairly small, this works well enough. (If the combinations had been larger, it might have been necessary to invoke a script to first generate the rules.)

To represent Postman environments, we're using plain `.env` environment files. A script runs before the generated tests to load the file into the node environment.

## Issues

### Whitespace squishing

So far we've had a lot of success with this method although there are limitations. A superficial but annoying one is the handling of whitespace. Since it works at the semantic level only, the syntax tree doesn't specify whitespace which means that when the code is generated it ends up squashed together. The more precise way to deal with that would be to generate a concrete syntax tree, but this involves calculating start and end offsets for each node, which is quite a bit more fiddly than generating the AST alone. The work-around we're using at the moment is to run the final output through ESLint and use [the `padding-line-between-statements` rule](https://github.com/mavaddat/postman2mocha/blob/77016914fc64e7479dc90c8df3f74fa7776c9d39/src/index.js#L28) to insert newlines in more or less the right places. It's not perfect, but it's not bad.

### Response type diversity

Another limitation is that we can only handle JSON responses at the moment. This is a side effect of only being able to call `response.json` once on a fetch API response but `pm.response.json` multiple times. By moving the code that extracts the JSON to the `before` block in the generated suite, we work around this issue &mdash; but, we end up assuming that all responses contain JSON. A more sophisticated approach would be to scan test scripts for calls to `pm.response.json`, `pm.response.text` etc and add only include the corresponding lines in the before block if necessary.

### Variables scope diversity

We're also not supporting various other API calls &mdash; including looping via `pm.setNextRequest` or globals, variables or collection variables; although, we do support `pm.environment`. This is mostly because we haven't needed these features yet, although it's hard to see how the looping could map cleanly to Mocha (any suggestions here would be welcome).

## Feedback

There's still plenty of testing and a lot more implementation left to do, but we're already finding this to be an effective approach.

If you would like to try it out or even [contribute](CONTRIBUTE.md). If you have any requests, suggestions or other feedback we'd love to hear from you at [hello@automationschool.com](http://hello@automationschool.com/).
