# Existential Types

Existential types are essentially strong type aliases which only expose
a specific set of traits as their interface and the concrete type in the
background is inferred from a certain set of use sites of the existential
type.

In the language they are expressed via

```rust
existential type Foo: Bar;
```

This is in existential type named `Foo` which can be interacted with via
the `Bar` trait's interface.

Since there needs to be a concrete background type, you can currently
express that type by using the existential type in a "defining use site".

```rust
struct Struct;
impl Bar for Struct { /* stuff */ }
fn foo() -> Foo {
    Struct
}
```

Any other "defining use site" needs to produce the exact same type.
Any use site inside the existential type's "reveal scope" is able to
observe the `Struct` type (so they can coerce a `Foo` to a `Struct`).
Use sites outside the existential type's "reveal scope" will not see
the `Struct` type, but instead only see the `Foo` type.

## Reveal scope

The reveal scope of an existential type is any code *within* the parent
of the existential type definition. This includes any siblings of the
existential type and all children of the siblings.

The initiative for "not causing fatal brain damage to developers due to
accidentally running infinite loops in their brain while trying to
comprehend what the type system is doing" has decided to disallow children
of existential types to be part of the reveal scope.

Any item within the reveal scope can coerce a value of an existential type
to its concrete type. It is not possible to "simply" access the existential
type as if it were its concrete type, instead an explicit coercion is
necessary.

Any item within the reveal scope can also potentially be a defining use site
(this depends on conditions defined in the next section).

## Defining use site(s)

Currently only the return value of a function inside the reveal scope can
count as the "reveal scope" of an existential type (and only if the return
type of that function contains the existential type).