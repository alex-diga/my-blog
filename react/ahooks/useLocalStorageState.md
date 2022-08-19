---
title: useLocalStorageState源码
---

## 核心

`useLocalStorageState`和`useSessionStorageState`传入`localStorage`或者`sessionStorage`

createUseStorageState 方法根据传入的`storage`

```typescript
function createUseStorageState(getStorage: () => Storage | undefined) {
  function useStorageState<T = any>(
    key: string,
    options?: Options<T>
  ): StorageStateResult<T>;
  function useStorageState<T>(
    key: string,
    options: OptionsWithDefaultValue<T>
  ): StorageStateResultHasDefaultValue<T>;

  function useStorageState<T>(
    key: string,
    options?: Options<T> & OptionsWithDefaultValue<T>
  ) {
    let storage: Storage | undefined;

    // https://github.com/alibaba/hooks/issues/800
    try {
      storage = getStorage();
    } catch (err) {
      console.error(err);
    }

    const serializer = (value: T) => {
      if (options?.serializer) {
        return options?.serializer(value);
      }
      return JSON.stringify(value);
    };

    const deserializer = (value: string) => {
      if (options?.deserializer) {
        return options?.deserializer(value);
      }
      return JSON.parse(value);
    };

    function getStoredValue() {
      try {
        const raw = storage?.getItem(key);
        if (raw) {
          return deserializer(raw);
        }
      } catch (e) {
        console.error(e);
      }
      if (isFunction(options?.defaultValue)) {
        return options?.defaultValue();
      }
      return options?.defaultValue;
    }

    const [state, setState] = useState<T | undefined>(() => getStoredValue());

    useUpdateEffect(() => {
      setState(getStoredValue());
    }, [key]);

    const updateState = (value?: T | IFuncUpdater<T>) => {
      if (typeof value === "undefined") {
        setState(undefined);
        storage?.removeItem(key);
      } else if (isFunction(value)) {
        const currentState = value(state);
        try {
          setState(currentState);
          storage?.setItem(key, serializer(currentState));
        } catch (e) {
          console.error(e);
        }
      } else {
        try {
          setState(value);
          storage?.setItem(key, serializer(value));
        } catch (e) {
          console.error(e);
        }
      }
    };

    return [state, useMemoizedFn(updateState)];
  }
  return useStorageState;
}
```
