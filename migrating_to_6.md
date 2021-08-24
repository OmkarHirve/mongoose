* `useNewUrlParser`, `useUnifiedTopology`, `useFindAndModify`, and `useCreateIndex` are no longer supported options. Mongoose 6 always behaves as if `useNewUrlParser`, `useUnifiedTopology`, and `useCreateIndex` are `true`, and `useFindAndModify` is `false`. Please remove these options from your code.

* Mongoose connections are no longer [thenable](https://masteringjs.io/tutorials/fundamentals/thenable). This means that `await mongoose.createConnection(uri)` **no longer waits for Mongoose to connect**. Use `mongoose.createConnection(uri).asPromise()` instead. See [#8810](https://github.com/Automattic/mongoose/issues/8810).

* The `mongoose.connect()` function now always returns a promise, **not** a Mongoose instance.

* Mongoose no longer allows executing the same query object twice. If you do, you'll get a `Query was already executed` error. Executing the same query instance twice is typically indicative of mixing callbacks and promises, but if you need to execute the same query twice, you can call `Query#clone()` to clone the query and re-execute it. See [gh-7398](https://github.com/Automattic/mongoose/issues/7398)

* In MongoDB Node.js Driver v4.x, 'MongoError' is now 'MongoServerError'. Please change any code that depends on the hardcoded string 'MongoError'.

* Mongoose now clones discriminator schemas by default. This means you need to pass `{ clone: false }` to `discriminator()` if you're using recursive embedded discriminators.

* Mongoose now saves objects with keys in the order the keys are specified in the schema, not in the user-defined object. So whether `Object.keys(new User({ name: String, email: String }).toObject()` is `['name', 'email']` or `['email', 'name']` depends on the order `name` and `email` are defined in your schema.

* Mongoose now passes the document as the first parameter to `default` functions. This may affect you if you pass a function that expects different parameters to `default`, like `default: mongoose.Types.ObjectId`. See [gh-9633](https://github.com/Automattic/mongoose/issues/9633)

* Mongoose arrays are now ES6 proxies. You no longer need to `markModified()` after setting an array index directly.

* Schema paths declared with `type: { name: String }` become single nested subdocs in Mongoose 6, as opposed to Mixed in Mongoose 5. This removes the need for the `typePojoToMixed` option. See [gh-7181](https://github.com/Automattic/mongoose/issues/7181).

* Mongoose now throws an error if you `populate()` a path that isn't defined in your schema. This is only for cases when we can infer the local schema, like when you use `Query#populate()`, **not** when you call `Model.populate()` on a POJO. See [gh-5124](https://github.com/Automattic/mongoose/issues/5124).

* When populating a subdocument with a function `ref` or `refPath`, `this` is now the subdocument being populated, not the top-level document. See [#8469](https://github.com/Automattic/mongoose/issues/8469)

* Using `save`, `isNew`, and other Mongoose reserved names as schema path names now triggers a warning, not an error. You can suppress the warning by setting the `supressReservedKeysWarning` in your schema options: `new Schema({ save: String }, { supressReservedKeysWarning: true })`. Keep in mind that this may break plugins that rely on these reserved names.

* Single nested subdocs have been renamed to "subdocument paths". So `SchemaSingleNestedOptions` is now `SchemaSubdocumentOptions` and `mongoose.Schema.Types.Embedded` is now `mongoose.Schema.Types.Subdocument`. See [gh-10419](https://github.com/Automattic/mongoose/issues/10419)

* `Aggregate#cursor()` now returns an AggregationCursor instance to be consistent with `Query#cursor()`. You no longer need to do `Model.aggregate(pipeline).cursor().exec()` to get an aggregation cursor, just `Model.aggregate(pipeline).cursor()`.

* `autoCreate` is `true` by default **unless** readPreference is secondary or secondaryPreferred, which means Mongoose will attempt to create every model's underlying collection before creating indexes. If readPreference is secondary or secondaryPreferred, Mongoose will default to `false` for both `autoCreate` and `autoIndex` because both `createCollection()` and `createIndex()` will fail when connected to a secondary.

* The `context` option for queries has been removed. Now Mongoose always uses `context = 'query'`.

* Mongoose 6 always calls validators with depopulated paths (that is, with the id rather than the document itself). In Mongoose 5, Mongoose would call validators with the populated doc if the path was populated. See [#8042](https://github.com/Automattic/mongoose/issues/8042)

* When connected to a replica set, connections now emit 'disconnected' when connection to the primary is lost. In Mongoose 5, connections only emitted 'disconnected' when losing connection to all members of the replica set.

* `Document#populate()` now returns a promise. And is now no longer chainable. Replace `await doc.populate('path1').populate('path2').execPopulate()` with `await doc.populate('path1'); await doc.populate('path2');`

* `await Model.create([])` in v6.0 returns an empty array when provided an empty array, in v5.0 it used to return `undefined`. If any of your code is checking whether the output is `undefined` or not, you need to modify it with the assumption that `await Model.create(...)` will always return an array if provided an array.

* `doc.set({ child: { age: 21 } })` now works the same whether `child` is a nested path or a subdocument: Mongoose will overwrite the value of `child`. In Mongoose 5, this operation would merge `child` if `child` was a nested path.

* Mongoose now adds a `valueOf()` function to ObjectIds. This means you can now use `==` to compare two ObjectId instances.

* If you set `timestamps: true`, Mongoose will now make the `createdAt` property `immutable`. See [gh-10139](https://github.com/Automattic/mongoose/issues/10139)

* `isAsync` is no longer an option for `validate`. Use an `async function` instead.

## TypeScript changes

* The `Schema` class now takes 3 generic params instead of 4. The 3rd generic param, `SchemaDefinitionType`, is now the same as the 1st generic param `DocType`. Replace `new Schema<UserDocument, UserModel, User>(schemaDefinition)` with `new Schema<UserDocument, UserModel>(schemaDefinition)`

* The following legacy types have been removed: `ModelUpdateOptions`, `DocumentQuery`, `HookSyncCallback`, `HookAsyncCallback`, `HookErrorCallback`, `HookNextFunction`, `HookDoneFunction`, `SchemaTypeOpts`, `ConnectionOptions`.