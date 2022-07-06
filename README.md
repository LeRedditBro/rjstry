# Rjstry

![Rjstry logo](./images/rjstry.svg)

modular pattern matcher designed to reduce boilerplate when matching between objects

the motivation was to create a system to handle mapping between identifiable object instances to new instances (that may be extensions of the original instance)

## Common use cases

-   Backend sends a list of ids that need to be mapped to react elements
-   Serialized game data that needs to be instantiated as game objects

## Installation

**npm:**

```bash
npm install rjstry
```

**yarn:**

```bash
yarn add rjstry
```

**pnpm:**

```bash
pnpm add rjstry
```

## Usage

### Defining matches

the registry matches items via matcher functions, a function that receives a parameter and returns a new object if it had matched, if there is no match the matcher should return `undefined` so the next matcher may be activated, can also use any object with `match` member as matcher, see [composition](###composition)

```js
import Registry from 'rjstry';

// === registry creation ===
const reg = new Registry(
	(i) => i === 42 && 'meaning of life',
	(i) => i % 2 === 0 && 'even',
	(i) => {
		return { message: 'unknown handle', origin: i };
	}
);

// === registry invocation ===
reg.match(42); // -> 'meaning of life'
reg.match(4); // -> 'even'
reg.match('test'); // -> { message: 'unknown handle', origin: 'test' }
```

### Composition

any object with the `match` member may also be used as a matcher function.
this enables us to compose multiple sub-registries into one root registry.

this is very powerful if you need modular registries, and also allows to segment your registry to prioritized chunks

```js
import Registry from 'rjstry';

const reg1 = new Registry((i) => i === 5 && 'five');
const reg2 = new Registry((i) => 'unknown');

const root = new Registry(reg1, reg2);

root.match(5); // -> 'five'
root.match(6); // -> 'unknown'

reg1.add((i) => i === 6 && 'six');

root.match(6); // -> 'six'
```

### Lookup

a utility matcher aimed to reduce boiler if need to match the same single key to identify an object

```js
import Registry, { Lookup } from 'rjstry';

const lookup = new Lookup(x => x.id, {
	a: 'aa',
	b: 'bb',
	c: (item) => { ...item, c: 'cc' },
	d: (item, context, ...parameters) => { ...item, d: 'dd', onClick: () => { context.doSomething() }, parameters },
});

lookup.match({ id: 'a' })
// -> 'aa'
lookup.match({ id: 'c' })
// -> { id: 'c', c: 'cc' }

// lookup can also be used as a matcher in the registry
const root = new Registry(lookup);

root.match({ id: 'b' });
// -> 'bb'
```