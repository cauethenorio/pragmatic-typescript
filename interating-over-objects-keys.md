# Iterating over object keys

---

## Problem: `Object.keys` doesn't work because returns an array of strings, not a union of all the keys

```typescript
interface User {
    name: string;
    age: number;
}

function printUser(user: User) {
  Object.keys(user).forEach((key) => {
    // Doesn't work!
    console.log(user[key]);
  })
};
```

Errors in code
> Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'User'.
  No index signature with a parameter of type 'string' was found on type 'User'.

[Playground Link](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgKoGdrIN4ChkHIhwC2EAXMumFKAOYDc+hcdFRAriQEbRMC+uXDA4gEYYAHsQyAA61wGaAAoOmKJSVQAlDmYB5bgCsI4gHQBrCAE90q9drMxJUAKKIAFsuVXrugLwAfHqEyAD0YcgAIpIQ6CAA5GDIAO4uFgCEzAQI0uiSADYQZgWSdPbQANq+ALraTAT82rj8DEA)

The same problem occurs when using `for...in` loops:

```typescript
interface User {
    name: string;
    age: number;
}

function printUser(user: User) {
  for (const key in user) {
    // error here
    console.log(user[key]);
  }
};
```

Errors in code
> Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'User'.
  No index signature with a parameter of type 'string' was found on type 'User'.

[Playground Link](https://www.typescriptlang.org/play?ssl=10&ssc=3&pln=1&pc=1#code/JYOwLgpgTgZghgYwgAgKoGdrIN4ChkHIhwC2EAXMumFKAOYDc+hcdFRAriQEbRMC+uXDA4gEYYAHsQyAA61wGaAAoOmKJSVQAlDmYxJUZMoTTqyANYQAnslDI10XXkLJTIdJIA2EAHRfJOlV1AG0rawBdbSYCQX4GIA)


## Solutions

### 1. Use `Object.keys` and cast the result to `Array<keyof T>`

```typescript
interface User {
    name: string;
    age: number;
}

function printUser(user: User) {
  const userKeys = Object.keys(user) as Array<keyof User>;

  userKeys.forEach((key) => {
    console.log(user[key]);
  })
};
```
[Playground Link](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgKoGdrIN4ChkHIhwC2EAXMumFKAOYDc+hcdFRAriQEbRMC+uXDA4gEYYAHsQyAA61wGaAAoOmKJSVQAlDmYJp1ZGugBpCAE90yALzIA8twBWEcQDoA1pfSr1uuNYAglBQcBYAPF4WkjBo6gB8TMwmUOZWbjCSUACiiAAWyspRujbxeoTIBiDokgA2EG61knS+0ADaUQC62kwE-Nq4-AxAA)

### 2. Type Predicates

By using a isKey helper, we can check that the key is actually on the object before indexing into it.

```typescript
interface User {
    name: string;
    age: number;
}

function isKey<T extends object>(
  x: T,
  k: PropertyKey
): k is keyof T {
  return k in x;
}

function printUser(user: User) {
  Object.keys(user).forEach((key) => {
    if (isKey(user, key)) {
      console.log(user[key]);
    }
  })
};
```
[Playground Link](https://www.typescriptlang.org/play?ssl=19&ssc=3&pln=1&pc=1#code/JYOwLgpgTgZghgYwgAgKoGdrIN4ChkHIhwC2EAXMumFKAOYDc+hcdFRAriQEbRMC+uXDA4gEYYAHsQyYOgDSEAJ4AeACrIIAD0ggAJumSTuAKwjiAfAApmWymoA0zANaUAClEkAHaGCWKlXABKSmdZQ2dlSRhkDTwCKAgwDigZMNBkLQEhETEJaWQvWnAMaCsOTChKUqggnGYAeVNzMAA6SKV0csqg1phJKABRRAALKysOuoBeC3rCWRirOQDu6AdkSbr4+eQEaXRJABsIVsPJOlWoAG0OgF0gpnnBAn4g3H4GIA)

## References
- https://www.totaltypescript.com/iterate-over-object-keys-in-typescript
- https://twitter.com/mattpocockuk/status/1681267079977000961