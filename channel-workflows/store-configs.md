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
        /**
         * @depracated Not used in file store, use filename instead
         */ 
        id?: string
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
        database?: string // defaults to 'test'
        table?: string // defaults to 'test'
        id?: string // defaults to '$MSH-10.1'
      }
    }
  | {
      mongo: {
        uri?: string // defaults to `mongodb://127.0.0.1:27017`
        database?: string // if undefined you must provide the database name in the uri connection string
        collection?: string // defaults to `test`
        warnOnError?: boolean // defaults to `false`
        /**
         * @description If true, use the normalized HL7 parsed JSON else use the looser typed HL7 parsed JSON.
         */ 
        normalized?: boolean // defaults to `false`
        verbose?: boolean // defaults to `false`
        id?: string // defaults to `UUID`
        /**
         * @see https://www.mongodb.com/docs/manual/reference/connection-string/#std-label-connections-connection-options
         */ 
        options?: MongoClientOptions // defaults to `undefined`
      }
    }
  | {
      dgraph: {
        /**
         * @description uri including port to gRPC of alpha node
         * @see https://dgraph.io/docs/deploy/security/ports-usage/
         */ 
        uri?: string // defaults to 172.0.0.1:9080
        warnOnError?: boolean // defaults to `false`
        verbose?: boolean // defaults to `false`
        id?: string // defaults to `$MSH-10.1`
      }
    }
  | {
      postgres: {
        table?: string // defaults to `test`
        schemaOwner?: string // defaults to `postgres`
        /**
         * @see https://github.com/panates/postgresql-client/blob/master/DOCUMENTATION.md#222-poolconfiguration
         */ 
        ...PoolConfiguration 
      }
    }
```

The below config settings can use values from the message using the HL7 path (See [Extrapolating Data from Messages](../msg-class/extrapolating.md)):

- File Store
  - `path`
  - `filename`
- SurrealDB Store
  - `namespace`
  - `database`
  - `table`
  - `id`
- MongoDB Store
  - `database`
  - `collection`
  - `id`
- Dgraph Store
  - `id`
- PostgreSQL Store
  - `table`
  - `id`
 
The parameter of `surreal.id`, `mongo.id`, `dgraph.id`, and `postgres.id` if set to `"UUID"` will generate universally unique identifiers.

The `file.path` and `file.filename` settings can be an array of strings that get concatenated together. Paths are concatenated using the directory traverse character (`/`). Filenames are concatenating with no separating characters. If you need a separating character, then you can add it as an element in the array.

Looking for a store that is not yet implemented? Please open a [new issue](https://github.com/gofer-engine/gofer-engine/issues). Pull Requests are welcome.

All of the stores are currently in a single npm library, `@gofer-engine/stores`. In the future, we will most likely break each store into it's own library and require users to install only the stores that they need to use to reduce unused code in project builds.
