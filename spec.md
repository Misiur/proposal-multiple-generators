Multiple generators syntax proposal
===

Stage: Draft
---

Author: Marcin Misiurski  
Repository: [Misiur/proposal-multiple-generators](https://github.com/Misiur/proposal-multiple-generators)  
See also: [tc39/proposal-async-iteration](https://github.com/tc39/proposal-async-iteration/issues/103)  
See also: [Misiur/proposal-spread-async](https://github.com/Misiur/proposal-spread-async)

Synopsis
---

In this document, we propose extending existing syntax to support a better way for handling cases with generator class exposing multiple iterators for developers.

### Should
* Support default generator without any changes in code
* Allow for an easier way for handling alternative asynchronous iterator
* Allow for an easier way for choosing a different iterator

### May not
* Break any existing syntax

Existing syntax
---
```js
class Multi
{
  alternativeIterator = Symbol('multi_alternativeiterator');

  constructor() {
    this[this.alternativeIterator] = function* () {
      yield 1;
      yield 2;
      yield 3;
    }
  }

  *[Symbol.iterator]() {
    yield 1;
    yield 2;
    yield 3;
  }

  async *otherIterator() {
    yield 4;
    yield 5;
    yield 6;
  }
}

(async () => {
  const multi = new Multi();

  const gens = [...multi];
  const ogens = [];

  for await (const item of multi.otherIterator()) {
    ogens.push(item);    
  }

  for (const item of multi[multi.alternativeIterator]()) {
    ogens.push(item);
  }

  console.log(gens, ogens);
})();
```

Proposed syntax
---
```js
class Multi
{
  alternativeIterator = Symbol('multi_alternativeiterator');

  constructor() {
    @iterator(alt)
    this[this.alternativeIterator] = function* () {
      yield 1;
      yield 2;
      yield 3;
    }
  }

  *[Symbol.iterator]() {
    yield 1;
    yield 2;
    yield 3;
  }

  @iterator(other)
  async *otherIterator() {
    yield 4;
    yield 5;
    yield 6;
  }
}

(async () => {
  const multi = new Multi();
  const gens = [...multi];
  const ogens = [...await multi[^other], ...multi[^alt]];

  console.log(gens, ogens);
})();
```