gulp-mejs
====
> Mejs plugin for gulp

[![NPM version][npm-image]][npm-url]
[![Build Status][travis-image]][travis-url]

## [Mejs](https://github.com/teambition/mejs) -- Moduled and Embedded JavaScript templates

## Install

Install with [npm](https://npmjs.org/package/gulp-mejs)

```
npm install --save-dev gulp-mejs
```

## Usage

```js
var gulpMejs = require('gulp-mejs')

gulp.task('mejs', function () {
  return gulp.src('test/fixtures/*.html')
  .pipe(gulpMejs({filename: 'templates.js'}))
  .pipe(gulp.dest('test'));
})

gulp.task('render-page', function () {
  return gulp.src('test/fixtures/*.html')
  .pipe(gulpMejs.render('header', {
    title: 'test render',
    user: {name: 'zensh'}
  }))
  .pipe(gulp.dest('test'))
})

gulp.task('render-pages', function () {
  return gulp.src('test/fixtures/*.html')
  .pipe(gulpMejs.render(['header', 'user'], function (tplName) {
    switch (tplName) {
      case 'header':
        return {
          title: 'test render',
          user: {name: 'zensh'}
        }
      case 'user':
        return {
          name: 'test-man'
        }
    }
  }))
  .pipe(gulp.dest('test'))
})
```

options and Mejs class API: https://github.com/teambition/mejs

## Demo

`test/fixtures/header.html`:
```html
<p><%= it.title || 'gulp' %> module</p>
<%- include('user', it.user) %>
```
`test/fixtures/user-list.html`:
```html
<ul>
  <% it.users.forEach(function(user) { -%>
    <li>
      <%= user.name %>
    </li>
  <% }) -%>
</ul>
```
`test/fixtures/user.html`:
```html
<h1><%= it.name %></h1>
```

precompile to `test/templates.js`(Run it in node.js/io.js/browers):
```js
// **Github:** https://github.com/teambition/mejs
//
// **License:** MIT
/* global module, define, window */

// Mejs is a compiled templates class, it can be run in node.js or browers

;(function (root, factory) {
  'use strict'

  if (typeof module === 'object' && module.exports) module.exports = factory()
  else if (typeof define === 'function' && define.amd) define([], factory)
  else root.Mejs = factory()
}(typeof window === 'object' ? window : this, function () {
  'use strict'

  var hasOwn = Object.prototype.hasOwnProperty
  var templates = {}
  var htmlEscapes = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#39;',
    '`': '&#96;'
  }

  function Mejs (locals) {
    this.locals = locals || {}
    this.templates = copy({}, templates, true)
  }

  Mejs.import = function (tpls) {
    copy(templates, tpls, true)
    return this
  }

  var proto = Mejs.prototype
  proto.copy = copy

  proto.render = function (tplName, data) {
    var template = this.get(tplName)
    if (typeof template !== 'function') throw new Error(tplName + ' is not found')
    return template.call(this, this.copy(data, this.locals), tplName)
  }

  proto.get = function (tplName) {
    return hasOwn.call(this.templates, tplName) ? this.templates[tplName] : null
  }

  proto.add = function (tplName, tplFn, overwrite) {
    if (!overwrite && hasOwn.call(this.templates, tplName)) throw new Error(tplName + ' exist')
    this.templates[tplName] = tplFn
    return this
  }

  proto.remove = function (tplName) {
    delete this.templates[tplName]
    return this
  }

  proto.import = function (ns, mejs, overwrite) {
    if (typeof ns !== 'string') {
      overwrite = mejs
      mejs = ns
      ns = '/'
    } else ns = ns.replace(/\/?$/, '\/')
    for (var tplName in mejs.templates) {
      if (hasOwn.call(mejs.templates, tplName)) {
        this.add(this.resolve(ns, tplName), mejs.get(tplName), overwrite)
      }
    }
    return this
  }

  proto.resolve = function (parent, current) {
    parent = this.stringify(parent)
    current = this.stringify(current).replace(/^([^\.\/])/, '\/$1')
    current = /^\//.test(current) && !/\/$/.test(parent) ?
      current : (parent.replace(/[^\/]*\.?[^\/]*$/, '').replace(/\/?$/, '\/') + current)
    current = current.replace(/\/\.?\/+/g, '\/')
    while (/\/\.\.\//.test(current)) current = current.replace(/[^\/]*\/\.\.\//g, '')
    return current.replace(/^[\.\/]*/, '')
  }

  proto.escape = function (str) {
    return this.stringify(str).replace(/[&<>"'`]/g, function (match) {
      return htmlEscapes[match]
    })
  }

  proto.stringify = function (str) {
    return str == null ? '' : String(str)
  }

  function copy (dst, src, overwrite) {
    dst = dst || {}
    for (var key in src) {
      if (hasOwn.call(src, key) && (overwrite || !hasOwn.call(dst, key))) dst[key] = src[key]
    }
    return dst
  }

  templates['header'] = function (it, __tplName) {
    var ctx = this, __output = ""
    var include = function (tplName, data) { return ctx.render(ctx.resolve(__tplName, tplName), ctx.copy(data, it)) }
    ;__output += "<p class=\"test\">";;__output += ctx.escape(it.title || 'gulp');__output += " module</p>\n";;__output += ctx.stringify(include('user', it.user));__output += "\n";
    return __output.trim()
  };

  templates['user-list'] = function (it, __tplName) {
    var ctx = this, __output = ""
    ;__output += "<ul>\n  ";;it.users.forEach(function(user) {
  ;__output += "    <li>\n      ";;__output += ctx.escape(user.name);__output += "\n    </li>\n  ";;})
  ;__output += "</ul>\n";
    return __output.trim()
  };

  templates['user'] = function (it, __tplName) {
    var ctx = this, __output = ""
    ;__output += "<h1>";;__output += ctx.escape(it.name);__output += "</h1>\n";
    return __output.trim()
  };

  return Mejs
}))
```

## License

MIT © [Teambition](http://teambition.com)

[npm-url]: https://npmjs.org/package/gulp-mejs
[npm-image]: http://img.shields.io/npm/v/gulp-mejs.svg

[travis-url]: https://travis-ci.org/teambition/gulp-mejs
[travis-image]: http://img.shields.io/travis/teambition/gulp-mejs.svg
