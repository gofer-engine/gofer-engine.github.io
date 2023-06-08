[🏠 Gofer Engine](https://gofer-engine.github.io/) > [Channel Workflows](./index.md) > Store Configs

## Store Configs

Store Configs are used to persist messages to supported stores. The interface `StoreConfig` is currently defined as:

```typescript
type StoreConfig =
  | {
      file: {
        path?: string[] // defaults to ['local']
        format?: 'string' | 'json' // defaults to 'string'
        overwrite?: boolean // default to true
        append?: boolean // defaults to false
        autoCreateDir?: boolean // defaults to true
        warnOnError?: boolean // defaults to false
        extension?: string // defaults to '.hl7'
        filename?: string | string[] // defaults to '$MSH-10.1'
        verbose?: boolean // defaults to false
      }
    }
  | {
      surreal: {
        uri?: string // defaults to http://127.0.0.1:8000/rpc
        user?: string // defaults to env.SURREALDB_USER or 'root'
        pass?: string // defaults to env.SURREALDB_PASS or 'root'
        warnOnError?: boolean // defaults to false
        verbose?: boolean // defaults to false
        namespace?: string // defaults to 'test'
        database?: strings // defaults to 'test'
        table?: string // defaults to 'test'
        id?: string // defaults to '$MSH-10.1'
      }
    }
```

The `file.path`, `file.filename`, `surreal.namespace`, `surreal.database`, `surreal.table`, and `surreal.id` settings can use values from the message using the HL7 path (See [Extrapolating Data from Messages](../msg-class/extrapolating.md).

The parameter of `surreal.id` if set to `"UUID"` will generate universally unique identifiers.

The `file.path` and `file.filename` settings can be an array of strings that get concatenated together. Paths are concatenated using the directory traverse character (`/`). Filenames are concatenating with no separating character. If you need a separating character, then you can add it as an element in the array.

In the future, more stores will be added. We are open to pull requests for new stores at: [gofer-stores](https://github.com/amaster507/gofer-stores)
