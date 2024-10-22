#smells
- [[Extract Method]]
- Use async functions
- Use promises
```js
const makeSandwich = () => {
  ...
  getBread(function(bread) {
    ...
    sliceBread(bread, function(slicedBread) {
      ...
      getJam(function(jam) {
        ...
        brushBread(slicedBread, jam, function(smearedBread) {
          ...
        });
      });
    });
  });
};

// -------------------------------------------

const getBread = doNext => {
  ...
  doNext(bread);
};

const sliceBread = doNext => {
  ...
  doNext(breadSlice);
};

...

const makeSandwich = () => {
  return getBread()
    .then(bread => sliceBread(bread))
    .then(jam => getJam(beef))
    .then(slicedBread, jam => brushBread(slicedBread, jam));
};
```