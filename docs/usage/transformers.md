---
title: Transformers
weight: 3
---

Transformers are the heart of the package. They take a PHP class and will determine if it can be transformed to Typescript, when possible, the transformer will transform the PHP class to tyupescript.

## Default transformers

Although writing your own transformers isn't that difficult we've added a few transformers to get started:

- `MyclabsEnumTransformer`: this converts a `myclabs\enum`
- `DtoTransformer`: a powerful transformer that transforms classes and their properties, you can read more about it [here](https://docs.spatie.be/typescript-transformer/v1/dtos/transforming-dtos/)

The laravel package has some extra transformers:

- `SpatieEnumTransformer`: this converts a `spatie\enum`
- `SpatieStateTransformer`: this converts `spatie\laravel-model-states`

## Writing transformers

A transformer is a class which implements the `Transformer` interface:

```php
use Spatie\TypescriptTransformer\Transformers\Transformer;

class EnumTransformer implements Transformer
{
    public function canTransform(ReflectionClass $class): bool
    {
        // can this transformer handle the given class?
    }

    public function transform(ReflectionClass $class, string $name): Type
    {
        // get the typescript representation of the class
    }
}
```

In the `canTransform` you should decide if this transformer can convert the class. In the `transform` method, you should return a `Type`, the transformed type. Let's take a look at how we can create them.

### Creating types

```php
Type::create(
    ReflectionClass $class, // The reflection class
    string $name, // The name of the Type
    string $transformed // The Typescript representation of the class
);
```

What about creating types that depend on other types? It is possible to add a fourth argument to the `create` method:

```php
Type::create(
    ReflectionClass $class,
    string $name,
    string $transformed,
    MissingSymbolsCollection $missingSymbols
);
```

A `MissingSymbolsCollection` will contain links to other types. The package will replace these links with correct Typescript types. So, for example, say you have this class:

```php
/** @typescript **/
class User extends DataTransferObject
{
    public string $name;
    
    public RoleEnum $role;
}
```

As you can see it has a RoleEnum as a property, which looks like this:

```php
/** @typescript **/
class RoleEnum extends Enum
{
    const GUEST = 'guest';
    
    const ADMIN = 'admin';
}
```

When transforming this class we don't know what the `RoleEnum` will be, but since it is also converted to Typescript the package will produce the following output:

```typescript
export type RoleEnum = 'guest' | 'admin';

export type User = {
    name : string;
    role : RoleEnum;
}
```

In your transformers you should add such linked properties to the `MissingSymbolsCollection` as such:

```php
$type = $missingSymbols->add(RoleEnum::class); // Will return {%RoleEnum::class%}
```

The `add` method will return a token that can be used in your transformed type, to be replaced later. It's the link we described above between the types.

When in the end, no type was found(because it wasn't converted to Typescript, for example), it will be replaced with the `any` Typescript type.

#### Inline types

It is also possible to create an inline type, these types will not create a whole new Typescript type but just replace a type inline in another type. In our previous example, if we would transform `Enum` classes with an inline type, the generated Typescript would look like this:

```typescript
export type User = {
    name : string;
    role : 'guest' | 'admin';
}
```

Inline types can be created like the regular types, but they do not need a name:

```php
Type::createInline(
    ReflectionClass $class,
    string $transformed
);
```

When needed you can also add a `MissingSymbolsCollection`:

```php
Type::createInline(
    ReflectionClass $class,
    string $transformed,
    MissingSymbolsCollection $missingSymbols
);
```

When you create a new transformer, do not forget to add it to the list of transformers in your configuration!