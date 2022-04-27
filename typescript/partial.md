# typescript 之 Partial

## 深度 Partial

```ts
type Partial1<T> = {
  [P in keyof T]?: T[P];
};

type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends Object ? DeepPartial<T[P]> : T[P];
};

type Person = {
  name: string;
  age: string;
  contact: {
    email: string;
    phone: number;
  };
};

let partialPerson: DeepPartial<Person> = {
  name: "tom",
  contact: {
    phone: 10086,
  },
};
```

[深度 Partial](https://segmentfault.com/a/1190000019758521)
