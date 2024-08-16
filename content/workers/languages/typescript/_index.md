---
pcx_content_type: navigation
title: TypeScript
weight: 2
meta:
  title: Write Cloudflare Workers in TypeScript
---

## TypeScript

TypeScript is a first-class language on Cloudflare Workers. Cloudflare publishes type definitions to [GitHub](https://github.com/cloudflare/workers-types) and [npm](https://www.npmjs.com/package/@cloudflare/workers-types) (`npm install -D @cloudflare/workers-types`). All APIs provided in Workers are fully typed, and type definitions are generated directly from [workerd](https://github.com/cloudflare/workerd), the open-source Workers runtime.

### Dynamic Runtime Types (Experimental)

There is a possibility that the types defined in `@cloudflare/workers-types` might not exactly match your project's runtime APIs. This is because runtime APIs are dynamic, and their exact configuration depends on:

1. The `compatibility_date` defined in your `wrangler.toml`.
2. Any `compatibility_flags` defined in your `wrangler.toml`.

For example, if you have `compatibility_flags = ["nodejs_als"]` in your `wrangler.toml`, then the runtime will allow you to use the [AsyncLocalStorage](https://nodejs.org/api/async_context.html#class-asynclocalstorage) class in your worker code, but you will not see this reflected in the type definitions in `@cloudflare/workers-types`.

In order to solve this issue, and to ensure that your type definitions are always up-to-date with your compatibility settings, you can dynamically generate the runtime types (as of `wrangler 3.66.1`):

```bash
wrangler types --experimental-runtime

# You could also use the alias
wrangler types --x-runtime
```

This will generate a `d.ts` file and (by default) save it to `.wrangler/types/runtime.d.ts`. You will be prompted in the command's output to add that file to your `tsconfig.json`'s `compilerOptions.types` array.

If you would like to commit the file to git, you can provide a custom path. Here, for instance, the `runtime.d.ts` file will be saved to the root of your project:

```bash
wrangler types --x-include-runtime="./runtime.d.ts"
```

Note that the `--x-runtime` command replaces the need for the `@cloudflare/workers-types` package, so that package should be uninstalled to avoid any potential for conflict.

### Type Checking (Experimental)

We provide a flag for the `wrangler types` command that will run `tsc` on your project's type definitions after the types have been generated. This is useful in CI because it allows you to verify that your project's types are valid without having to run the full build process and before you deploy your Worker. Here are the available commands:

```bash
# Generate your Env types and run tsc
wrangler types --x-check

# Generate your Env types, saving to a custom path, and run tsc
wrangler types --x-check /path/to/env-types.d.ts

# Generate your Env and runtime types, and run tsc
wrangler types --x-check --x-runtime

# Generate your Env and runtime types, saving the runtime types to a custom path, then run tsc
wrangler types --x-check --x-runtime=/path/to/runtime-types.d.ts

# Generate your Env and runtime types, saving both the Env and runtime types to custom paths, then run tsc
wrangler types --x-check /path/to/env-types.d.ts --x-runtime=/path/to/runtime-types.d.ts
```

The `--x-check` flag runs the TypeScript compiler (tsc) to check your project's types. This command will pass any additional flags through to `tsc`.

### TypeScript in Development (Experimental)

Wrangler provides several options to enhance your TypeScript development experience when using `wrangler dev`. These options allow you to generate and check types on the fly, ensuring that your development environment closely matches your production environment.

In the context of `wrangler types`, the flags are namespaced with `-types`. As an example, if you wanted to automatically generate your runtime types and check them on the fly, you would run:

```bash
# Generate Env types
wrangler dev --x-types

# Generate Env and runtime types, then run tsc
wrangler dev --x-types-check --x-types-runtime
```

See [the full list of available flags](/workers/wrangler/commands/#types) for more details.

### Known issues

#### Transitive loading of `@types/node` overrides `@cloudflare/workers-types`

You project's dependencies may load the `@types/node` package on their own. As of `@types/node@20.8.4` that package now overrides `Request`, `Response` and `fetch` types (possibly others) specified by `@cloudflare/workers-types` causing type errors.

The way to get around this issue currently is to pin the version of `@types/node` to `20.8.3` in your `package.json` like this:

```json
---
filename: package.json
---
{
	"overrides": {
		"@types/node": "20.8.3"
	}
}
```

For more information, refer to [this GitHub issue](https://github.com/cloudflare/workerd/issues/1298).

### Resources

- [TypeScript template](https://github.com/cloudflare/workers-sdk/tree/main/templates/worker-typescript)
- [@cloudflare/workers-types](https://github.com/cloudflare/workers-types)
- [Runtime APIs](/workers/runtime-apis/)
- [TypeScript Examples](/workers/examples/?languages=TypeScript)
