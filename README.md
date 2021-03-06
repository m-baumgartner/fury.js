# Fury.js

[![Circle CI](https://circleci.com/gh/apiaryio/fury.js.svg?style=svg)](https://circleci.com/gh/apiaryio/fury.js)
[![Coverage Status](https://coveralls.io/repos/apiaryio/fury.js/badge.svg)](https://coveralls.io/r/apiaryio/fury.js)
[![Dependency Status](https://david-dm.org/apiaryio/fury.js.svg)](https://david-dm.org/apiaryio/fury.js)
[![devDependency Status](https://david-dm.org/apiaryio/fury.js/dev-status.svg)](https://david-dm.org/apiaryio/fury.js#info=devDependencies)

API Description SDK

> _Wardaddy: [Best job I ever had](http://www.imdb.com/title/tt2713180/quotes?item=qt2267083)._

Fury provides uniform interface to API description formats such as
[API Blueprint][https://apiblueprint.org] and [Swagger](http://swagger.io/).

Note: Fury requires *adapters* to support parsing and serializing. You will need to install at least one adapter along with Fury. You can [find Fury adapters](https://www.npmjs.com/search?q=fury-adapter) via npm.

## Usage

### Install

Fury.js is available as an npm module.

```sh
$ npm install --save fury
```

### Refract Interface

Fury.js offers an interface based on the [Refract Project](https://github.com/refractproject/refract-spec) element specification and makes use of the API description and data structure namespaces. Adapters convert from formats such as API Blueprint into Refract elements and Fury.js exposes these with API-related convenience functionality. For example:

```js
import fury from 'fury';
import apibParser from 'fury-adapter-apib-parser';

// The input as a string
const source = 'FORMAT: 1A\n# My API\n...';

// Use the API Blueprint parser adapter
fury.use(apibParser);

// Parse the input and print 'My API'
fury.parse({source}, function(err, result) {
  console.log(result.api.title);
});
```

Once you have a parsed API it is easy to traverse:

```js
api.resourceGroups.forEach(function (resourceGroup) {
  console.log(resourceGroup.title);

  resourceGroup.resources.forEach(function (resource) {
    console.log(resource.title);

    resource.transitions.forEach(function (transition) {
      console.log(transition.title);

      transition.transactions.forEach(function (transaction) {
        const request = transaction.request;
        const response = transaction.response;

        console.log(`${request.method} ${request.href}`);
        console.log(`${response.statusCode} (${response.header('Content-Type')})`);
        console.log(response.messageBody);
      });
    });
  });
});
```

It is also possible to do complex document-wide searching and filtering. For example, to print out a listing of HTTP methods and paths for all defined example requests:

```js
/*
 * Prints out something like:
 *
 * POST /frobs
 * GET /frobs
 * GET /frobs/{id}
 * PUT /frobs/{id}
 * DELETE /frobs/{id}
 */
function filterFunc(item) {
  if (item.element === 'httpRequest' && item.statusCode === 200) {
    return true;
  }

  return false;
}

console.log('All API request URIs:');

api.find(filterFunc).forEach(function (request) {
  console.log(`${request.method} ${request.href}`)
});
```

Reference:

* [Minim](https://github.com/refractproject/minim)
* [API description elements](https://github.com/refractproject/minim-api-description)
* [Parse result elements](https://github.com/refractproject/minim-parse-result)

#### Multiple Fury Instances

There may come a day when you need to have multiple Fury instances with different adapters or other options set up in the same program. This is possible via the `Fury` class:

```js
import {Fury} from 'fury';

const fury1 = new Fury();
const fury2 = new Fury();

fury1.parse(...);
```

#### Writing an Adapter

Adapters convert from an input format such as API Blueprint into refract elements. This allows a single, consistent interface to be used to interact with multiple input API description formats. Writing your own adapter allows you to add support for new input formats.

Adapters are made up of a name, a list of media types, and up to three optional public functions: `detect`, `parse`, and `serialize`. A simple example might look like this:

```js
export const name = 'my-adapter';
export const mediaTypes = ['text/vnd.my-adapter'];

export function detect(source) {
  // If no media type is know, then we fall back to auto-detection. Here you
  // can check the source and see if you think you can parse it.
  return source.match(/some-test/i) !== null;
}

export function parse({minim, generateSourceMap, source}, done) {
  // Here you convert the source into refract elements. Use the `minim`
  // variable to access refract element classes.
  const Resource = minim.getElementByClass('resource');
  // ...
  done(null, elements);
}

export function serialize({api, minim}, done) {
  // Here you convert `api` from javascript element objects to the serialized
  // source format.
  // ...
  done(null, outputString);
}

export default {name, mediaTypes, detect, parse, serialize};
```

Now you can register your adapter with Fury.js:

```js
import fury from 'fury';
import myAdapter from './my-adapter';

// Register my custom adapter
fury.use(myAdapter);

// Now parse my custom input format!
fury.parse({source: 'some-test\n...'}, function (err, api) {
  console.log(api.title);
});
```

### Legacy Interface

This is the older "legacy" interface for API Blueprint and Apiary Blueprint parsing.

#### API Blueprint Parsing

```js
var parser = require('fury').legacyBlueprintParser;
var source = '# My API\n';

parser.parse({ code: source }, function(error, api, warnings) {

    console.log(api.name);
});
```
#### Markdown Rendering

The legacy interface also offers access to Markdown rendered as used internally
by API and Apiary Blueprint parsers.

```js
var markdownRenderer = require('fury').legacyMarkdownRenderer;
var source = '# My API\n';


markdownRenderer.toHtml(source, {}, function(error, html) {

    console.log(html);
});
```

## Development

### Building & Testing
Parts of Fury.js are written in Coffeescript, so you must build the final library before it can be used. All of the build/test/etc commands are run through npm:

```sh
# Build the library
npm run compile

# Run the unit and integration tests
npm test

# Generate a coverage report
npm run coverage

# Open the HTML report
open coverage/lcov-report/index.html
```

[API Blueprint]: http://apiblueprint.org
