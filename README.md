# @objectwow/join

Join object to object like Database join

## Use case

In a microservices system where each service owns its own database, querying data requires calling multiple services to retrieve the necessary information and manually joining the data. This package simplifies the process of joining data, similar to how MongoDB handles it.

## Installation

```
npm i @objectwow/join
```

## Basic usage

```typescript
import { joinData } from "@objectwow/join";

const local = [
  { id: 1, items: [101, 102] },
  { id: 2, items: [201] },
];

const from = () => [
  { id: 101, product: "Product 101" },
  { id: 102, product: "Product 102" },
  { id: 201, product: "Product 201" },
];

const result = await joinData({
  local,
  localField: "items",
  from,
  fromField: "id",
  as: "products",
});
```

LocalData will be overwritten

```typescript
local = [
  {
    id: 1,
    items: [101, 102],
    products: [
      { id: 101, product: "Product 101" },
      { id: 102, product: "Product 102" },
    ],
  },
  {
    id: 2,
    items: [201],
    products: [{ id: 201, product: "Product 201" }],
  },
];

result = {
  allSuccess: true,
  joinFailedValues: [],
};
```

Note: see more samples in the [`tests`](https://github.com/objectwow/join/blob/main/tests/core.spec.ts)

```typescript
export interface JoinDataParam {
  /**
   * Local object or an array of local objects to be joined.
   */
  local: LocalParam;

  /**
   * A callback (async) function that returns the data from the source.
   * Data is object or an array of objects
   */
  from: (...args: any[]) => any;

  /**
   * The field name in the local object(s) used for the join,
   * can be a nested field, separated by a dot ('.')
   */
  localField: string;

  /**
   * The field name in the from object used for the join,
   * can be a nested field, separated by a dot ('.')
   */
  fromField: string;

  /**
   * An optional new field name to store the result of the join in the local object(s).
   */
  as?: string;

  /**
   * An optional mapping from the from object(s) values to the new field names in the local object(s).
   */
  asMap?: AsMap;
}

export type LocalParam = object | object[];

export type AsMap = { [key: string]: string };

export type JoinDataResult =
  | { joinFailedValues: Primitive[]; allSuccess: boolean }
  | any;
```

## Customization

With an out-of-the-box design, you can create your own function using the current structure.

```typescript
import { JoinData } from "@objectwow/join";

export class YourJoin extends JoinData {
  public execute(
    param: JoinDataParam,
    metadata?: any
  ): Promise<JoinDataResult> {}

  // Use case: Currently, deep values are split by a dot ('.'), but you can use a different symbol if needed
  protected separateSymbol: string;

  protected generateAsValue(param: GenerateAsValueParam) {}

  // Use case: Return your custom output
  protected generateResult(
    joinFailedValues: Primitive[],
    localOverwrite: LocalParam,
    metadata?: any
  ) {}

  protected getFieldValue(parent: object, path: string) {}

  protected handleLocalObj(param: HandleLocalObjParam): void {}

  protected parseFieldPath(fieldPath: string): {
    path: string;
    newPath: string;
  } {}

  // Use case: Shadow clone local data without overwriting the original.
  protected standardizeLocalParam(
    local: LocalParam,
    metadata?: any
  ): Promise<LocalParam> {}

  // Use case: Automatically call internal or external services to retrieve data based on the input
  protected standardizeFromParam(
    from: FromParam,
    metadata?: any
  ): Promise<any[]> {}

  // Use case: Throw an error if the field is invalid
  protected validateFields(
    arr: { key: string; value: any }[],
    metadata?: any
  ): void {}
}
```

- Using solution 1: Override the default JoinData instance

```typescript
SingletonJoinData.setInstance(new YourJoin())

await joinData({...})
```

- Using solution 2: create new singleton instances as needed

```typescript
import { SingletonJoinData } from "@objectwow/join";

export class YourSingletonJoinData extends SingletonJoinData{}

YourSingletonJoinData.setInstance(new YourJoin())

export async function yourJoinDataFunction(
  params: JoinDataParam,
  metadata?: any
): Promise<JoinDataResult> {
  return YourSingletonJoinData.getInstance().execute(params, metadata);
}

await yourJoinDataFunction({...})
```

- Using solution 3: you can directly use your new class without a singleton instance.

```typescript
const joinCls = new YourJoin()
await joinCls.execute({...})
```

Tips:

- You can call original function (parent function) with `super.`
- You can pass anything to the metadata

## Benchmark

- Execute: `node lib/benchmark.js`
- Report: JoinData Execution x 2,289,754 ops/sec ±3.38% (86 runs sampled)
- Device: MacbookPro M1, 16 GB RAM, 12 CPU
