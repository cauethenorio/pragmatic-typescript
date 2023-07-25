# React Polymorphic Components & forwardRef

---

## Problem: React Polymorphic components are hard to type correctly

Polymorphic component are the ones that can be rendered with a different container element/node.

Example: `<Box as="div" />`, `<Box as="span" />` and `<Link as="button">Click me</Link>`.

The `as` prop can be any valid HTML element or a React component and other possible props will depend on it's value.
It means a polymorphic component can have different props depending on the `as` prop value.

## Solutions

### Polymorphic component that can render any HTML element

```tsx
import React, { ElementType } from 'react';

interface Props<T extends ElementType> {
  as?: T;
  theme: 'dark' | 'light';
}

type DistributiveOmit<T, TOmitted extends PropertyKey> = T extends any
  ? Omit<T, TOmitted>
  : never;

function StyledButton<T extends ElementType>(
  props: Props<T> &
    DistributiveOmit<
      React.ComponentProps<ElementType extends T ? 'button' : T>,
      'as'
    >
) {
  const { as: Comp = 'button', theme, ...rest } = props;
  return <Comp {...rest}></Comp>;
}

// examples

<StyledButton as="a" theme="dark" href="https://www.sun.com">
  🌝
</StyledButton>;

<StyledButton theme="light" form="button-only-prop">
  ☀
</StyledButton>;

```
**Playground Link:** [Provided](https://www.typescriptlang.org/play?#code/JYWwDg9gTgLgBAJQKYEMDGMA0cDecCiANkiEgHYwAqAnmEnAL5wBmUEIcA5FKhpwNwAoQcApIozdPQAKbMAGcAPJThIAHjHIATeQWKkKNOgD5cguHBTyA-AC44lIRZgALEknuctKKAGtOcAA+XITAAOYuMAKCDMIwtPQAIsDyMFDAAEYArjDAAG5IAPIgwDDK2JTFpZpaqhraurIQdLDUANJI1KYAvA51mmQ6lmTU5nDWcFVllBVTNcZj9mRIBVBCgsxZZBjAEGRwAMrxxFoAQjkwe8r9DXruhgnGABRjYHLy9k0KyqYAZGMWZKpdLZXIFKaKAEWRC8GAAOgAwuxIMsKF8lER7lQEjdBroVBNOKDLmQAvZKMZMFCLJwrJwoQsAJRmCxoPapXCWD5wJHgOC9IkXPacbCudzYOGSngcpi9N7NeROOA8GBZKD7RS8sC4SVw6UwBjGRQAei1xiEsUExuNdRQ4GI8mEiiO1BO5xgJK53QARChvXAxaQfd4-P6XDxmD7IjAFLZrQB3RNw+RbOFskDehYWQA8G4BcfcEJpdbqFZHNTqLSDOJYDbiD3tCERg-uY0BAPuJewAtHtCNRO-KwJmxoAAMkA8H8F40VqsevZloA)

### With `forwardRef`


```tsx
import React, { ElementType, forwardRef, useRef } from 'react';

type FixedForwardRef = <T, P = {}>(
  render: (props: P, ref: React.Ref<T>) => JSX.Element
) => (props: P & React.RefAttributes<T>) => JSX.Element;

const fixedForwardRef = forwardRef as FixedForwardRef;

// Type to distribute the Omit across a union  (https://stackoverflow.com/a/57103940/780262)
type DistributiveOmit<T, TOmitted extends PropertyKey> = T extends any
  ? Omit<T, TOmitted>
  : never;

interface Props<T extends ElementType> {
  as?: T;
  additionalProp: string;
  optionalProp?: string;
}

const Link = fixedForwardRef(function UnwrappedLink<T extends ElementType>(
  props: Props<T> &
    DistributiveOmit<
      React.ComponentProps<ElementType extends T ? 'a' : T>,
      'as'
    >
) {
  const { as: Comp = 'a', ...rest } = props;
  return <Comp {...rest}></Comp>;
});

/**
 * Should work without specifying 'as'
 */

function StyledButton<T extends ElementType>(
  props: {
    as?: T;
    theme: 'dark' | 'light';
  } & React.ComponentProps<ElementType extends T ? 'button' : T>
) {
  const { as: Comp = 'button', theme, ...rest } = props;
  return <Comp {...rest}></Comp>;
}

const Example1 = () => {
  const ref = useRef<HTMLAnchorElement>(null);

  return (
    <>
      <Link additionalProp="a" ref={ref} />

      <Link
        additionalProp="a"
        optionalProp="b"
        // e should be inferred correctly
        onClick={(e) => e.currentTarget.href}
      ></Link>

      <Link
        as="button"
        additionalProp="a"
        onClick={(e) => e.currentTarget}
      ></Link>
    </>
  );
};

// Should work with Custom components

const Custom = forwardRef(
  (
    props: { thisIsRequired: boolean },
    ref: React.ForwardedRef<HTMLDivElement>
  ) => {
    return <div ref={ref} />;
  }
);

const Example2 = () => {
  const ref = useRef<HTMLDivElement>(null);
  return (
    <>
      <Link as={Custom} thisIsRequired={true} additionalProp="a" ref={ref} />
    </>
  );
};

```
**Playground Link:** [Provided](https://www.typescriptlang.org/play?#code/JYWwDg9gTgLgBAJQKYEMDGMA0cDecCiANkiEgHYwAqAnmEtgGbQDuKUAJsg9gK4DOSLnAC+cBlAgg4AciioM0gNwAoZTFpI4AMWAAPJOy0s2nJAzgBeOAB5K2AAqXcwgHwAKZXDhyy7JFAAuODcwCTA+IPtsOQYg5HQYADouWxcASksXOAApAGUADUSiEnIYZQyLLJCwiLhHADJEeSSuAEEYGChgACMeGCQ+VIqsvMLi0goVZTQIMj54Bj0DIyhWDiErJlWTIRQ+bSXDY3WzKYB6M7gaOjgYCDh2YHmu3v7bgAtNAHkQYHh0CR8fYoOA8MjAWZeNzvDrhAIXeboADWEAAbv4GIQIMxEjMQGcUGcAKwAdgAjAAGADMAE4ACwUs4kgAcFIATAA2NlpNQaOAAESenR6fWA6J+f1s2EoEo6BjgSF0-V8+3sYX86gA0khqFkrJQFUryOxgWRqJ44AB+OCyqVXWX9dguC1BMhIdFQKbACgY9CaNUQcK2Q3Kk0EYgTKgaLI4C17S1BSgqLwodiPGAQsgoQgBsBBZ7egDmybggYzs2zuYTcALZGLymEqhmc3gABlvUinIt9EdticGG4GGCMJm4ABVMjMKAoMB0djtshI4OK0P7cala5IdwW0KB2q5waULL1C1eQUF15ipC209eJoJRIAYUkkDdFAP1nXFE3IeN+wN1rSCg0hwImLiYLeXhAXw0i3s6GSxl4zbzLgcB7EEz7gE4QHSNgiT4XIKGiFYu7hCWcgwDwUBkDYmFgLg+GJIRMCuNYZx0S4KjCGk5wAFS8Z4vFwLk7wQDwhDsHAzDQJ2zB-KJfQ1nQaDAAw1BFjIeywXAvFnKoQ5kCOkK5OoxDsAAQn0dxkMuRoquGJTftGHheKRtSIXe8aJiWXgwJ8pBBNI7BsEiIEAD4yIQwCFjCSgWqIjTxBgT4vrMpQfl+UY3Cuf5XFaMivNZIFgeUuAWsh8B4OhcB0dhhWzLhHwlHhBEDPAxFwG55FIJR1G0S+DGtfMrHsS+nENk2swofgugoOAxBkk4bjDGVSFTfAMROPwghmNYAASlAALKtq0hmiVAmXuGQ4mEDxqheBRVE0S5d7WM6d6vQunapummaVmEFgAEQoID3hmBYOAxKIZzOpBNhfXDKZpn8f05gDwOA4jpZgOWWZo4GQPdJjH0fRcCo1gpElwN0mjegw-hyJJMxQHIGCEOaJN3rMj5RWgSIQ24SArUguJUT4VBsIWPWJO8UNwy4bFfbDnPWAjnMpnwhNWbMxPqz9KMVvjYBAyDWPc7z-M4ILwuiyzG6Sz1jacwrZxK7ebHvXAd3COclwiWJVPSVAsnyTV-B3FIeKvqUfCTS2YfzJIXbHKYA4Wi9rk1EEeB+U8ACSfDIAAjjwwCM0E3QQBAxAoDRwgQR9MRxM0iQrGsfip-tR2toKqKXRaK0eQ9PVPTYjyomDDAQ1DcAwyWjZ3dM60ELN81IGyS2D+Vy+bVY20pAdx295dbjXYQt3db1z3u57H2qx2aGazgj7h5Ioi53wBfF6XjMQ50PBIFEPrXG-0CYY0ntPMw0Nb42BhgPLiKggA)
      


## References
- https://github.com/total-typescript/react-typescript-tutorial/blob/main/src/08-advanced-patterns/72-as-prop-with-forward-ref.solution.tsx
- https://www.freecodecamp.org/news/build-strongly-typed-polymorphic-components-with-react-and-typescript/
- https://stackoverflow.com/questions/57103834/typescript-omit-a-property-from-all-interfaces-in-a-union-but-keep-the-union-s
- https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types