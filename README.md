# tc39-proposal-importScripts
The importScripts() method of the GlobalScope interface synchronously imports one or more scripts into the scope. (inlining)
current implementations: https://html.spec.whatwg.org/multipage/workers.html#dom-workerglobalscope-importscripts-dev

importScripts(...urls)
Fetches each URL in urls, executes them one-by-one in the order they are passed, and then returns (or throws if something went amiss).

- designed to work in Scripts and Modules 
- designed to work without additional resolve hooks


## Status
Champion(s): champion name(s) Author(s): non-champion author(s), if applicable Stage: -1
## Motivation
Why is this important to have in the JavaScript language?
sync imports are needed:
- in both front-end and back-end programming. 
- They are the only way to handle sideEffects of native code without loosing events.
- ESM is only async so it misses events from the before the first process.nextTick
- There is no universal sync Module system for ECMAScript Engines while it is needed in some Engines
- require in nodejs for example is bad because it mixes CJS Modules, importScripts, nativRequire (.node modules and c++ bindings)
- browser can only import a bunch of scripts via adding multiple script tags in the render engine there is no equal function in the JS Runtime.
- it would make experiments a lot of easyer without diskwrites as it would no longer be needed to join/merge files for tests
- is like programatical inlining offers a lot of magic.
- offers a consistent way for ECMAScript Engines to load and run scripts without using eval directly. while it is equal to eval or in NodeJS VM.runInThisContext
- is part of a bigger effort to algin ECMAScript engines and reduce code rewrites.
- good for backward compatability 
- introduces a none module based ECMAScript Environment as standard starting point which is needed in all ECMAScript Runtime Implementations
- makes user land environment modifications / mocking easyer as they happen sync in the first process.Tick 
- Allows code reuse between diffrent module systems. 

## Importent Knowleg
ECMAScripts do exist in 2 main Flavors: Scripts and Modules everything that has export {} or import is a module everything else is a Script
in Some ECMAScript Engines like NodeJS are more Module Systems Implemented for example .node modules or .cjs out of ECMAScript view this implementations are Scripts.

The ECMAScript Module System ESM is Async by Design but this is about a Sync import Process.

## Use cases

*Some realistic scenarios using the feature, with both code and description of the problem; more than one can be helpful.*

**Server-side sync import**: Say you want to import a ECMAScript Script. Then, you would normally have to create a file and inline the code from the other file. If it's in the standard, it'd be easier as you can simply create a single file that is referencing the files that contain the codes and executes them in order.

```js
var urls = ['url1','url2']
importScripts(...urls);
```

**Browser-side sync import**: browsers do only support sync script imports via <script> tags in the renderer this would be a programatical way.
also it is implemented in the web worker global scope already there it works as it is outlined here

example.html
```html
<script src="url1"></script>
<script src="url2"></script>  
```  
would get
```html
<script>importScripts('url1','url2')</script>
```
  
  
**NodeJS-side sync import**: is implemented here via multiple calls to require if you pass Script pathes to it and you never assign what ever require returns.  

main.cjs
```js
const importScripts = (args) => {
  args.forEach(require);
}
  
var urls = ['url1','url2']
importScripts(...urls);
```

**electron-side sync import**: electron is a combination of the nodejs with a inhired chromeium both usinging the same v8 ECMAScript runtime engine
This leads to a lot of confusion because people that come from a NodeJS Background are familar with require and import as it supports both module systems
while only require is usable to bootstrap a electron process when you use multiple files.
  
as the people can use require the think they can also use import but that assumption is totaly wrong. because Electron uses Native sideEffects to detect when the first process.Tick did happen see: 
- [ ] https://github.com/electron/electron/issues/21457#issuecomment-1100472723 for more details
  
importScripts can clear the dust as it makes clear that the import is sync and it allows to reuse code between both module systems without additional files. 
  
**es4x-side sync import**:
  
**just-js-side sync import**:
  
**graaljs-js-side sync import**:
  
**deno-side sync import**:
  
**v8-side sync import**:
  
**JavaScriptCore sync import**:

  
### TypeScript
in TypeScript sync import is done via require, import and so called [tripple slash references](https://www.typescriptlang.org/docs/handbook/triple-slash-directives.html) ```<reference path="..." />``` TypeScript supports
also references and options in the configuration file tsconfig.json as TypeScript is a SuperSet of ECMAScript it would profit
from importScripts as it would need to get implemented as they are a SuperSet and this way eliminates Problems for coders
  
let me explain a bit more! most coders are using Modules as a way to Author code while they need to use legacy code. they often need to do all kinds
of transpilation and adjust ments to the typescript config and the packages they depend on while the only real goal is to inline some code snippets into
the current code.
  
also we tend to over reference and explain our imports and exports that reduces productivity. Static Typing and static analyze able code is well but 
you need to pay a price for it importScripts reduces this costs a lot. And it lets you reuse your original written code in many module systems if needed.
  
**TypeScript sync import**:  
code1.js
```js
globalThis.console.log('i am a imported script')
```
  
code.js works unmodified with typescript and the browser without the need for a package.json or the nodejs module system and it even imports a other script
```js
importScripts('./code1.js')
globalThis.console.log('hi')
// i assign something for the following main.ts example
globalThis.myUltraCommonNameSpace = 'hi from code.js'
```
  
main.ts or .cts or .mts (the extension are used by TypeScript 4.7+) to guess the module system. The exact same code would work in all module systems that are supported by TypeScript which are 7+ as time of writing! more are coming to address issues! importScripts Eliminates all of them. and skips the need for additional package.json lookups. it implicit directly tells typescript to use classic resolveMode as it does not support npm-resolve.
```ts
importScripts('./code.js')
const message = globalThis.myUltraCommonNameSpace as const // Type: readonly 'hi from code.js'
```

see: https://github.com/microsoft/TypeScript/issues/46452
  
  
## Description

*Developer-friendly documentation for how to use the feature*

`importScripts(...urls)` returns undefined or throws.

## Left as Todo
```
## Comparison

*A comparison across various related programming languages and/or libraries. If this is the first sort of language or library to do this thing, explain why that is the case. If this is a standard library feature, a comparison across the JavaScript ecosystem would be good; if it's a syntax feature, that might not be practical, and comparisons may be limited to other programming languages.*

These npm modules do something like the proposal:
- [B](link)
- [C](link)

frobnicate-2018 is weird because xyz, whereas B is weird because jkl, so we take a version of the approach in C, modified by qrs.

The standard libraries of these programming languages includes related functionality:
- APL (links to the relevant documentation for each of these)
- PostScript
- Self
- XSLT
- Emacs Lisp

Our approach is pretty similar to the Emacs Lisp approach, and it's clear from a manual analysis of billions of Stack Overflow posts that this is the most straightforward to ordinary developers.

## Implementations

### Polyfill/transpiler implementations

*A JavaScript implementation of the proposal, ideally packaged in a way that enables easy, realistic experimentation. See [implement.md](https://github.com/tc39/how-we-work/blob/master/implement.md) for details on creating useful prototype implementations.*

You can try out an implementation of this proposal in the npm package [frobnicate](https://www.npmjs.com/package/frobnicate). Note, this package has semver major version 0 and is subject to change.

### Native implementations

*For Stage 3+ proposals, and occasionally earlier, it is helpful to link to the implementation status of full, end-to-end JavaScript engines. Filing these issues before Stage 3 is somewhat unnecessary, though, as it's not very actionable.*

- [V8]() (*Links to tracking issues in each JS engine*)
- [JSC]()
- [SpiderMonkey]()
- ...

## Q&A

*Frequently asked questions, or questions you think might be asked. Issues on the issue tracker or questions from past reviews can be a good source for these.*

**Q**: Why is the proposal this way?

**A**: Because reasons!

**Q**: Why does this need to be built-in, instead of being implemented in JavaScript?

**A**: We could encourage people to continue doing this in user-space. However, that would significantly increase load time of web pages. Additionally, web browsers already have a built-in frobnicator which is higher quality.

**Q**: Is it really necessary to create such a high-level built-in construct, rather than using lower-level primitives?

**A**: Instead of providing a direct `frobnicate` method, we could expose more basic primitives to compose an md5 hash with rot13. However, rot13 was demonstrated to be insecure in 2012 (citation), so exposing it as a primitive could serve as a footgun.
```
