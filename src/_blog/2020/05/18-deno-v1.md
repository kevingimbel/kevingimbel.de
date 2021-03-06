---
title: "Deno 1.0 released"

type: blog
categories:
    - coding
tags:
    - javascript
    - node
    - runtime
    - rust 
    - code

date: "2020-05-18"
lastmod: "2020-05-18"
---

Deno made its first stable release with the release of v1! 

I've heard of Deno in the past but I never really cared for it. My work with JavaScript has always been in the browser and I'm not too much of a NodeJS fan - mainly because of the security and sandbox aspects of the runtime (or more, the lack of them). Deno aims to do better and fix some (or all?) of the NodeJS mistakes, like making access to the filesystem and network impossible unless allowed. In NodeJS, a script can read, write, send and receive network packages and basically do whatever it wants - there's little restriction and this has lead to malicious code executions in the past. 

## Quick overview

Deno is a JavaScript runtime that supports TypeScript out of the box. We can write regular JavaScript and execute it through the Deno toolchain, as the following example shows.

```ts
// file: hello.ts
console.log("Hello from Deno!");
```

If we save the file as `hello.ts` we can just run it.

```bash
$ deno run hello.ts
Hello from Deno!
```

This means with Deno all the benefits of TypeScript can be used out of the box, like ... Types. 😬

```ts
// file: sum.ts
function sum(a: number, b: number): number {
  return a + b;
}
```

By annotating the parameters (`a: number`) we can tell the compiler what type of parameter this functions expects. If we try to execute it with a wrong type, the compiler throws an error.

```ts
// file: ts_example.ts
import * as sum from "./sum.ts";

sum(4, "6");
```
When we try to tun the script with Deno, we get a nice error message:

```bash
$ deno run ts_example.ts
Compile file:///Users/kevingimbel/Development/private/deno/hello-world/ts_example.ts
error: Uncaught TypeError: sum is not a function
console.log(sum(4, "6"));
            ^
    at file:///Users/kevingimbel/Development/private/deno/hello-world/ts_example.ts:3:13
```

Types are incredibly helpful and in my opinion TypeScript is a blessing to the JavaScript world. Strong types are also one of the things I love about Rust - it just makes things clear and clean, even if it is hard sometimes. 

## File system access

Besides out-of-the-box TypeScript support, Denos security concept makes it very interesting, especially when you think about a Enterprise context and running code on servers where tight sandboxing and access control is important.

Here's another example. We use the std library `fs` module and the async function `exists` to check if a directory exists. If we run the code as it is with `deno run dir.ts` it will fail.

```ts
// file: dir.ts
import { exists } from "https://deno.land/std/fs/mod.ts";

const my_dir_exists = await exists("./foo"); // returns a Promise<boolean>
if (my_dir_exists) {
  console.log("Found directory!");
} else {
  console.log("Directory doesn't exist");
}
```

```bash
$ deno run dir.ts
error: Uncaught PermissionDenied: read access to "/Users/kevingimbel/Development/private/deno/hello-world/foo", run again with the --allow-read flag
    at unwrapResponse ($deno$/ops/dispatch_json.ts:43:11)
    at Object.sendAsync ($deno$/ops/dispatch_json.ts:98:10)
    at async lstat ($deno$/ops/fs/stat.ts:69:16)
    at async exists (https://deno.land/std/fs/exists.ts:8:5)
    at async file:///Users/kevingimbel/Development/private/deno/hello-world/dir.ts:8:23
```

As it turns out, we do not have write access so the script cannot check if a directory exists - this also means none of our dependencies can access the file system! To give the script read access the `--allow-read` flag can be used.

```bash
$ deno run --allow-read dir.ts 
Directory doesn't exist
```

`--allow-read` takes a directory as parameter, so the read access can be restricted to only a certain directory and sub-directories, as the following example shows:

Given the directory structure

```bash
├── dir.ts
├── test1
├── test2
```

Access to can be restricted to only `test1` with `--allow-read=./test1`.

```ts
// file: dir_test1.ts
import { exists } from "https://deno.land/std/fs/mod.ts";

const my_dir_exists = await exists("./test1"); // returns a Promise<boolean>
if (my_dir_exists) {
  console.log("Found directory!");
} else {
  console.log("Directory doesn't exist");
}
```

Here the script is executed with both directories - for some reason I needed the `--unstable` flag when a directory is passed to `--allow-read`.
```bash
$ deno run --allow-read=./test2 --unstable dir_test1.ts 
error: Uncaught PermissionDenied: read access to "/Users/kevingimbel/Development/private/deno/hello-world/test1", run again with the --allow-read flag
    at unwrapResponse ($deno$/ops/dispatch_json.ts:43:11)
    at Object.sendAsync ($deno$/ops/dispatch_json.ts:98:10)
    at async lstat ($deno$/ops/fs/stat.ts:69:16)
    at async exists (https://deno.land/std/fs/exists.ts:8:5)
    at async file:///Users/kevingimbel/Development/private/deno/hello-world/dir_test1.ts:8:23

$ deno run --allow-read=./test1 --unstable dir_test1.ts 
Found directory!
```

I'm incredibly excited for the future of Deno and I hope it will shift the JavaScript world into a more sandboxed, secure future. So far it is very promising.

The full announcement blog post can be found [on the deno blog](https://deno.land/v1)
