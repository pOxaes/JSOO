# POO in JS

# What

3 ways for a function to return an Object:

* Constructor functions
* ES6 Class (made up constructor)
* Factory

## common

* methods stored in a shared prototype
* private data via closures

## examples

### _constructor_

```
function Drums () {}

Drums.prototype.play = function () {
  console.log('BOUM');
};

const drumInstance = new Drums();
drumInstance.play(); // BOUM
```

### _class_

```
class Drums {
  play () {
    console.log('BOUM');
  }
}

const drumInstance = new Drums();
drumInstance.play(); // BOUM
```

Under the hood, still a constructor:

```
typeof class DoubleRainbow {} // function
```

Babel

```
var Drums = function () {
  function Drums() {
    _classCallCheck(this, Drums);
  }

  // Attach play method to Drums prototype
  _createClass(Drums, [{
    key: 'play',
    value: function play() {
      console.log('BOUM');
    }
  }]);

  return Drums;
}();

var drumInstance = new Drums();
```

### _factory_

#### objects with its own properties: custom part of objects

* more flexible
* can create objects with initial data

```
function createDrum() {
  return {
    play() {
      console.log('BOUM');
    }
  }
}
const drumInstance = createDrum();
drumInstance.play(); // BOUM

// play is a drumInstance's property
console.log(drumInstance) // { play: ƒ }
```

#### objects with assigned prototype: common part of objects

* costs less space & memory (it doesn't create properties)

> [Mozilla, Object.create ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
>
> The Object.create() method creates a new object with the specified prototype object and properties.

```
function createDrum() {
  const prototypeLike = {
    play () {
      console.log('BOUM');
    }
  };
  return Object.create(prototypeLike);
};

const drumInstance = createDrum();
drumInstance.play(); // BOUM

// play is a drumInstance.prototype's property
console.log(drumInstance) // {}
```

# Private properties

* Closures, closures and closures

Examples with...

_...class..._

```
class Drums {
  constructor() {
    const sound = 'BOUM'; // private workaround
    this.getSound = () => sound;
  }

  play () {
    console.log(this.getSound());
  }
}

const drumInstance = new Drums();
drumInstance.play(); // BOUM
```

using [Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)

```
const _name = Symbol('name');

class Test {
  constructor(name) {
    this[_name] = name;
  }

  get name() { return this[_name]; }
}
```

_...factory..._

```
function createDrum() {
  const sound = 'BOUM'; // private variable

  return {
    play () {
      console.log(sound);
    }
  };
};

const drumInstance = createDrum();
drumInstance.play(); // BOUM
```

# Inheritance

## Class and constructor

Prototype is a chain, populated with each extension

* protoA inherit from protoB
* protoB inherit from protoC
* ...
* protoZ inherit from Object's prototype

Reminder: When you try to access a property on an object, it checks the object’s own properties first. If it doesn’t find it there, it checks the prototype.

#### with a class

```
class A {}

class B extends A {}
```

#### with a constructor

```
function Parent() {}

function Child() {}
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;
```

## Factory

#### objects with assigned prototype

```
function createDrum() {
  return Object.create({
    whoAreYou() {
      console.log('A Drum!');
    },
    play () {
      console.log('BOUM');
    }
  })
}

function createSmallDrum() {
  return Object.create(Object.assign(createDrum.call(this), {
    play() {
      console.log('little boum')
    }
  }));
};

const smallDrumInstance = createSmallDrum();
smallDrumInstance.play(); // little boum
smallDrumInstance.whoAreYou(); // A Drum!
```

> Note: if you want to use Object spread, createSmallDrum becomes:

```
const drum = createDrum.call(this);
return Object.create({
  ...drum.__proto__,
  play() {
    console.log('little boum')
  }
});
```

> `__proto__` property of an object is a pointer to the object's constructor function's prototype property
>
> `foo.__proto__ === foo.constructor.prototype`

#### objects with custom properties

```
function createDrum() {
  return {
    sound: 'BOUM',
    whoAreYou() {
      console.log('A Drum!');
    },
    play() {
      console.log(this.sound);
    }
  }
}

function createSmallDrum() {
  return Object.assign(createDrum.call(this), {
    sound: 'little boum'
  });
};

const smallDrumInstance = createSmallDrum();
smallDrumInstance.play(); // little boum
smallDrumInstance.whoAreYou(); // A Drum!
```

> Note that you should not use arrow function in the returned Object because the context will be the function and not the object itself.

### More...

* return an object of any subtype of their return type: the object to be returned could be of several different types depending on some parameter
* mix custom properties + prototype
* compose multiple objects: more flexible

```
const createHuman() {
  return {
    canWalk: true,
    canTalk: true,
    canSee: false,
    canSleepWithoutPeeing: true
  }
}

const createBaby() {
  return {
    canSleepWithoutPeeing: false
  }
}

const createAlien() {
  return {
    canFly: true,
    canTalk: false
  }
}

const createSuperMixedEntity() {
  return Object.assign(createHuman(), createBaby(), createAlien())
}
```

# Benchmark

https://jsperf.com/prototype-vs-factory-vs-class

* Factory: 1 093 Ops / sec
* Prototype: 4 370 Ops / sec
* Class: 4 414 Ops / sec

**Factory 75% slower**
