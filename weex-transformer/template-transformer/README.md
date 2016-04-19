# Weex `<template>` Transformer

## Features

- convert a `<template>` element to JSON-like object
- autofix common mistakes & friendly warnings
    + tag name
    + attr
    + text content
- parse data binding and expressions
- make sure only one root node

## API

- `parse(code, done)`
- `validate(json, done)`

```javascript
/**
 * Parse `<template>` code to a JSON-like Object and log errors & warnings
 * 
 * @param {string} code
 * @param {function} done
 */
function parse(code, done) {}

/**
 * Validate a JSON-like Object and log errors & warnings
 * 
 * @param {object} json
 * @param {function} done
 */
function validate(json, done) {}

/**
 * Result callback
 *
 * data
 * - jsonTemplate{type, attr, style, events, shown, repeat}
 * - deps[tagname]
 * - log[{line, column, reason}]
 * 
 * @param {Error} err
 * @param {object} data
 */
function done(err, data) {}
```

## Validation

- root check: only one root element (ignore others)
- tagname check: native or hyphenated (autofix or warn non-hyphenated custom tagname)
- child/parent check: elements required certain child/parent (error)
- attr value check: special tag[attr] value (error or autofix or warn)
- text content check: only text element allows text content (error)
- data binding check:
    + normal value (exp converting)
    + event: non-mustache -> mustache (autofix)
    + style: `k: {{v}}; ...` (`styler.validate` needed)
    + class: `a {{v}} c`

## Demo

```javascript
var templater = require('weex-templater')

var code = '<template><container><text>Hello</text><img class="a {{x}} c" src="{{y}}" /><image style="opacity: {{z}}"></image></container></template>'

templater.parse(code, function (err, data) {
  // syntax error
  // format: {line, column, reason, ...}
  err
  // result
  // {
  //   type: 'container',
  //   children: [
  //     {
  //       type: 'text',
  //       value: 'Hello'
  //     },
  //     {
  //       type: 'img',
  //       class: function () {return ['a', this.x, 'c']},
  //       attr: {
  //         src: function () {return this.y}
  //       }
  //     },
  //     {
  //       type: 'img',
  //       style: {
  //         opacity: function () {return this.z}
  //       }
  //     }
  //   ]
  // }
  data.jsonTemplate
  // ['container', 'text', 'img']
  data.deps[]
  // format: {line, column, reason}
  // - Error: `image` tag should have a `src` attr
  // - NOTE: autofixed `image` tag name to `img`
  data.log[]
})

var json = {
  type: 'container',
  children: [
    {
      type: 'text',
      value: 'Hello'
    },
    {
      type: 'image',
      class: 'a {{x}} c',
      attr: {
        src: '{{y}}'
      }
    },
    {
      type: 'img',
      style: {
        opacity: '{{z}}'
      }
    }
  ]
}

styler.validate(json, function (err, data) {
  err
  data.jsonTemplate
  data.deps[]
  data.log[]
})
```