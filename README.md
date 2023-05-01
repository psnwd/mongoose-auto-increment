To use mongoose-auto-increment with TypeScript, follow these steps:

1. Install the extend package as a dependency using npm:

        npm i extend

2. Uninstall the mongoose-auto-increment package if it's already installed:

        npm uninstall mongoose-auto-increment

3. Create a new file named `mongoose-auto-increment.ts` (you can choose any name you want as long as it has a .ts extension). This file will contain the TypeScript types and definitions for `mongoose-auto-increment`.

4. In the mongoose-auto-increment.ts file, import the necessary packages and define the TypeScript types:

```ts
import { Schema, Connection } from "mongoose";
import extend from "extend";

export interface AutoIncrementSettings {
    model: string;
    field: string;
    startAt?: number;
    incrementBy?: number;
}

export interface AutoIncrementPlugin {
    (schema: Schema<any>, options?: AutoIncrementSettings): void;
}

export interface AutoIncrementModule {
    (connection: Connection): void;
    plugin: AutoIncrementPlugin;
}
```

5. Export the AutoIncrementModule function and the plugin function from the mongoose-auto-increment module (Just use provided files):

```ts
const autoIncrement: AutoIncrementModule = (connection: Connection) => {
    extend(true, connection.base.options, {
        useUnifiedTopology: true,
        useNewUrlParser: true,
        useFindAndModify: false,
        useCreateIndex: true,
    });
};

const plugin: AutoIncrementPlugin = function (schema: Schema<any>, options?: AutoIncrementSettings) {
    const fields = {};
    fields[options.field] = {
        type: Number,
        default: 0,
        unique: true,
    };
    schema.add(fields);

    schema.pre("save", function (this: any, next) {
        const self = this;
        if (self.isNew) {
            connection
                .model(options.model)
                .findOne({})
                .sort({ [options.field]: -1 })
                .exec(function (err, doc) {
                    if (err) {
                        return next(err);
                    }
                    self[options.field] = doc ? doc[options.field] + options.incrementBy : options.startAt;
                    next();
                });
        } else {
            next();
        }
    });
};

export default autoIncrement;
export { plugin };
```

6. In your schema file, import mongoose-auto-increment and your AutoIncrementSettings interface:
```ts
import autoIncrement, { AutoIncrementSettings } from "./mongoose-auto-increment";
```

7. Call the autoIncrement function and pass in your Mongoose connection:
```ts
autoIncrement(connection);
```

8. Define your schema and pass in the AutoIncrementSettings object to the plugin method:
```ts
const UserSchema = new Schema(schema, {
    timestamps: { createdAt: "created_at", updatedAt: "updated_at" },
});

UserSchema.plugin(plugin, {
    model: "user",
    field: "unq_id",
    startAt: 1000,
    incrementBy: 1,
});
```

9. Export your model with the TypeScript interface:
```ts
export default connection.model<UserInput>("user", UserSchema) as any;
```
