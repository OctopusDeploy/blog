---
title: "Refactoring Octopus: Adding strict null checks to the Octopus front-end"
description: Learn from some of the lessons we learned adding strict null checks to the Octopus front-end codebase
author: shaun.marx@octopus.com
visibility: public
published: 2020-06-10
metaImage: refactoring-octopus.png
bannerImage: refactoring-octopus.png
bannerImageAlt: Refactoring Octopus Adding strict null checks to the Octopus front-end
tags:
  - Engineering
  - TypeScript
---

![Refactoring Octopus: Adding strict null checks to the Octopus front-end](refactoring-octopus.png)

Nulls are often said to be the billion-dollar mistake. They creep up when you least expect them, and in some pretty interesting ways. The problem isn’t in the representation of null itself, it’s more that we often forget to deal with the `null` case. It’s for this reason that some languages completely shy away from the concept of null and choose to represent the concept in a type-safe way using the `Option` monad. Functional programming isn’t always an easy sell, and in some cases, we may still have legacy codebases. This is where `strictNullChecks` compiler flag swoops in and saves the day. This single switch allows TypeScript to treat `null` and `undefined` as separate types which forces those cases to be handled. This reduces bugs surrounding `null` and `undefined` and eliminates a lot of complexity when narrowing types appropriately. It also removes the need to write tests for certain cases and provides much faster feedback. If that’s got your attention, let’s look at some strategies for enabling strict nulls in an existing codebase.

## Possible Options

Enabling strict nulls on an existing codebase should be as simple as flipping a switch, right? Unfortunately, it’s not quite that simple. In our case, simply enabling the flag resulted in around 5,000 errors, which isn’t a trivial
amount to work through. This means we had to determine the best course of action to enable strict nulls. Let’s look at a couple of options:

### Do it all in one go

There may be a number of situations where you can convert everything in one pass. Maybe you already wrote perfect strict null compliant code to begin, which seems unlikely, or maybe you have a pretty small codebase. In our particular case, the magnitude of changes we would have to make in addition to the unknowns made this a non-starter. It may sound like it would be achievable with some hard work, however, we found that unless you apply surgical precision typing changes, you are very likely to introduce many more errors for every typing change that you make. It’s not easy to estimate when numbers keep changing under you.

### Segregate our front-end codebase

With all the rage around monorepos and TypeScript supporting project references, this is rather an attractive option. This would effectively allow us to take segments of our codebase and separate these into separate projects, allowing us to enable strict nulls for each individual segment incrementally. At first glance, this option is extremely attractive, but the tooling around this didn’t feel quite complete yet. There are a number of issues surrounding project reference support in `fork-ts-checker-webpack-plugin`, while project references don’t make much sense for react based libraries at the moment. This is still something we would like to revisit in the future, but it isn’t something we had the appetite for as part of converting to strict nulls.

### Multiple tsconfigs

We could have introduced multiple `tsconfig.json` files, one that excludes strict nulls and another that includes it. During development, we could have enabled strict nulls while we used a different config to exclude strict nulls for production builds. This would let us apply downward pressure over time on non-compliant code, but it would also mean you could potentially have 7,000 odd errors. This felt less than ideal. A variation on this is to let the IDE use the tsconfig with strict nulls enabled, which keeps all the errors out of the console.

### Multiple tsconfigs with assertion

With this strategy, you would disable strict nulls for your normal development and production builds. You would then add an additional tsconfig with strict nulls enabled, which contains an explicit list of files to be compliant. This would also require an additional step in CI builds to run a `tsc` check against the tsconfig that includes the strict null checks flag. This ensures that additional errors introduced will be picked up, but you may not necessarily see these during normal development. This may work well when intentionally contributing significant engineering effort toward the endeavor over a shorter period of time.

### The non-null-assertion operator

TypeScript has the non-null assertion operator (`!`), which acts as a means to tell TypeScript that something is not null within a particular scope. This allows typing issues surrounding `null` and `undefined` to be suppressed on a case by case basis, but it requires the suppressions be done for every error when strict nulls is enabled. Enabling linting rules to prevent the operator’s usage and suppressing these on a file by file basis is also beneficial to prevent the spread of the operator. This approach allows you to effectively enable strict nulls at a file level, and the associated work to get there can be predictably estimated.

We decided to use this as our preferred option because we want to implement strict nulls in a predictable manner with as little risk and as little impact as possible.

### Caveats around this approach

There are some standout negatives to the chosen approach, which are well worth calling out:

- It requires a lot of mechanical suppressions.
- It can lead to a partially implemented strict null codebase, which never ends up being fully completed without focused effort.
- Non-null assertions can spread if they are not kept in check.
- You can end up with some nasty code smells by using the non-null assertion operator in this fashion, for example:

```
    this.state = {
        variables: null!;
    }
```

We also knew that we’d need to put in more effort in the long run to address these issues, so we felt that the pros outweighed the cons in our case.

## Lessons learned

As part of our conversion process, we didn’t leave things as they were after applying non-null assertions everywhere. We also wanted to take one feature area of our font-end and convert it to be strict null compliant. We knew we needed a better understanding of the sorts of changes we needed to make as well as patterns to apply in order to make our lives easier. What follows are some of the lessons we learned in the process.

### Intersection of types can be mutually exclusive

As previously mentioned, TypeScript with strict nulls enabled will treat `null` and `undefined` as different types and will apply stricter checks on intersections of types. An example of this is using an intersection between a `Record` and another type which contain optional properties for keys. Take the following as an example:

```
type RouteArgs = Record<string, string>;
type AllArgs = { ids?: string [] };
type SomeArgs = RouteArgs & AllArgs;
const args: SomeArgs = { ids: undefined }; //invalid
```

This sort of thing would have worked before enabling strict nulls, but afterward, you are greeted with a message stating the property ids is not compatible with the index signature and not assignable to `undefined`. This makes complete sense if you look at the signature for `Record`:

```
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```

This means we can’t provide `undefined` as a key for `Record` since `keyof` `undefined` evaluating to the bottom type `never`. This also means that the intersection between the types don’t make sense, leading to the optional prop being dropped, and as a result, TypeScript complaining. It’s worth noting that TypeScript is getting more strict regarding this sort of thing with [TypeScript 3.9](https://devblogs.microsoft.com/typescript/announcing-typescript-3-9-rc/#breaking-changes).

### Indexing into types can get tricky with undefined in the mix

Imagine you have a React component with some deeply nested state, and you want some type-safety around mutating a small slice of it while avoiding the urge to install a new dependency such as immer. Let’s call this method setChildState. A contrived example of its usage could look like the following:

```
    type Address = {
        details: string;
        postcode: string;
    };

    type Person = {
        name: string;
        surname: string;
        address: Address;
    };

    type State = {
        person: Person;
    };

    class MyComponent extends React.Component<{}, State>{
        constructor(props: Props){
            super(props);
            this.state = {
                //..state
            }
        }
        setChildState<KeyOfState extends keyof State, Child extends State[KeyOfState], KeyOfChild extends keyof Child, GrandChild extends Child[KeyOfChild], KeyOfGrandChild extends keyof GrandChild>(
            first: KeyOfState,
            second: KeyOfChild,
            state: Pick<GrandChild, KeyOfGrandChild>,
            callback?: () => void
        ) {
            this.setState(
                prev => ({
                    [first]: {
                        ...prev[first],
                        [second]: {
                            ...(prev[first] as Child)[second],
                            ...state,
                        },
                    },
                }),
                callback
            );
        }

        changeAddress(address: Address) {
            this.setChildState("person", "address", address);
        }

        render() {
            return (
                <div>
                    <button
                    onClick={() =>
                        this.changeAddress({ postcode: "555", details: "Nowhere" })
                    }
                    >
                    Change Address
                    </button>
                </div>
            );
        }
    }

```

Unfortunately, that doesn’t work very well after you enable strict nulls, and you have a null/undefined down the chain. Feel free to try this yourself with [this example](https://codesandbox.io/s/confident-bush-oudj5?file=/src/App.tsx). Making the `person` optional in `State` will break things and with good reason. If the property is optional, then you may be spreading an `undefined` object, which leaves the state in some obscure state, i.e. only part of the object would be defined. You would effectively be lying about what is defined and what is not defined at that point. Of course, if you don’t care, you can strategically sprinkle `NonNullable` throughout the signature, but you may be better off keeping `null` and `undefined` out of the picture entirely by passing the value in via props instead. We will see an example of this a bit later.

### Strict nulls also mean stricter typing around functions

TypeScript lets you enable other strict flags such as `strictFunctionTypes`, which enable stricter bivariant checks. You may find it surprising to note, however, that when enabling only `strictNullChecks` that some stricter checks may be applied to functions unrelated to `null` and `undefined`. React `refs` are a good example of this. You may previously tried to use some base type such as `HTMLElement` for your refs like so:

```
export default function App() {
  const myRef = React.useRef<HTMLElement | undefined>();

  return (
    <div ref={myRef} className="App"></div>
  );
}
```

TypeScript lets you assign directly to the base type, but it won’t be as lenient for functions so you may need to narrow the type:

```
export default function App() {
  const myRef = React.useRef<HTMLDivElement | undefined>();
//Okay
  return (
    <div ref={myRef} className="App"></div>
  );
}
```

Presto, type errors go away, and the compiler is happy.

### Let’s talk about resource differences

Remember how we mentioned that strict nulls treat `null` and `undefined` as different types? In our particular case, it also meant that we had to slightly vary the interfaces for new and existing resources. This is because creating a resource may require fewer properties than specified. Of course, TypeScript doesn’t know that, and it’s just trying to be helpful. Let’s take the following resource as an example:

```
interface TenantLinks {
    Self: string;
    Variables: string;
    Web: string;
    Logo: string;
}

export interface TenantResource {
    Id: string;
    Name: string;
    TenantTags: string[];
    ProjectEnvironments: { [projectId: string]: string[] };
    ClonedFromTenantId: string | null;
    Description: string | null;
    SpaceId: string;
    Links: LinksCollection<TenantLinks>;
}
```

After enabling strict nulls, whenever we create a resource for the first time, we need to specify the links, the ID, ClonedFromTenantId, as well as the space ID, even though none of those make sense for a new resource. This is because the associated links are provided by the server, we auto-generate the ID, automatically infer the space ID, and the ClonedFromTenantId is populated by the server when a tenant is cloned. We chose to extract the shared properties for these resources into a separate interface and change properties as needed.

This ends up looking something like this:

```
interface TenantLinks {
    Self: string;
    Variables: string;
    Web: string;
    Logo: string;
}

interface TenantResourceShared {
    TenantTags: string[];
    ProjectEnvironments: { [projectId: string]: string[] };
    Name: string;
}

export interface TenantResource extends TenantResourceShared {
    Id: string;
    SpaceId: string;
    ClonedFromTenantId: string | null;
    Description: string | null;
    Links: LinksCollection<TenantLinks>;
}

export interface NewTenantResource extends TenantResourceShared {
    Description?: string;
}

```

With this change, our repositories accept the `NewResource` while always returning the full resource to use. Likewise, any `fetch` operations will return the full resource.

### Narrow once and as soon as possible

How you create types to represent things can be quite subjective. You can create classes and inherit and discriminate based on the constructor using instanceof. You can also create objects and discriminate via properties similar to redux actions. Discriminated unions have the advantage in that they don’t necessarily have the exact same shape. In both cases, we need to narrow types order to determine what to do. The same applies to primitives where `null` and `undefined` are included as part of the set of types, for example, `string | null | undefined`. We followed a simple rule to avoid adding optional chaining and/or null checks everywhere. In order to achieve this, we narrow a type as much as possible before attempting to work with it.

The above is a simple rule to follow and reduces complexity significantly. This feels obvious, but the reason is because it reduces the number of permutations you need to deal with. The wider the type, the more cases you have to work with, and the more chance you have of getting it wrong. It’s best to narrow to the most restrictive type possible as soon as possible. As an example, we prefer `string` over `string | null` while for more complex objects and class hierarchies, we prefer something more concrete where possible.

### Loading Data

Loading data in `componentDidMount` or in a hook is a pretty common thing to do when using react. The problem is you may not have the data available for the first render, which necessitates marking the data as optional.

One solution is to pass the data in via props and only render the component when you know you have the data available. This pattern became so commonplace that we created a component that specifically loads our data and injects it via props, which coincidentally also follows our rule of narrowing once and as soon as possible. This ends up looking something like the following simplified example. This may look somewhat familiar if you worked with the GraphQL Apollo client before (if you squint hard enough):

```
const Loader = DataLoader<PersonResource>();

export const ExamplePage = () => {
  return (
    <Loader
      load={async () => {
        return await getPerson("Person-1");
      }}
      renderWhenLoaded={data => {
        return <PersonLayout initialData={data} />;
      }}
      renderAlternate={p => <div>Loading</div>}
    />
  );
};
```

### Defaults

You may find that it’s possible to use a default case for some types instead of modifying existing types to be optional, `undefined`, or `null`. For example, you could:

- Default numbers to `0`.
- Default strings to `""`.
- Default arrays to `[]`.
- Default boolean to `false`.
- Default objects with all optional properties to `{}`.
- Default objects defined as `{ [key: string]: T }` to `{}`.

This isn’t always possible, especially where arrays are involved or when code is specifically looking for `null` or `undefined`, so it’s worth checking first.

### Optional params, null or undefined

If `null` and `undefined` are treated as different types, and we have the means to specify optional props, then how do we decide whether we should be using type signatures like `{ name: string | undefined }`, `{name: string | null }`, `{ name?: string }`, or even `{ name: string | null | undefined }`? This most likely comes down to intent and whether you want consumers to be forced to think about what they are providing. We try to avoid optional props and tend toward adding `null` to the type for this reason.

In our experience, optional props, when abused, can result in some subtle bugs and are much harder to find incorrect usages. It’s also usually a good idea to prevent an optional argument to cascade far and wide by defaulting it instead, if at all possible.

## Conclusion

Starting with the most strict TypeScript compiler rules available when starting a new project is definitely the best option. If you weren’t considering it, please do. Your future self will thank you. If you aren’t so lucky and you have a large existing codebase, it may require some serious, focused effort to get there. None of the options available seems to be perfect, so it’s best to choose the option that best suits your particular scenario, however, the effort seems well worth it. We covered some of the things we learned while converting a particular area in Octopus to be strict nulls compliant. The list is in no means exhaustive, and we expect to learn more as we continue with our journey of converting the remaining areas. We’d love to hear from you regarding patterns you’ve used to deal with strict nulls and whether there are other gotchas and learnings which we have not covered in this post.

Until next time, happy deployments!
