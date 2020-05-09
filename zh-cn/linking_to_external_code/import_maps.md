## Import 映射

> 此特性尚不稳定（unstable）。
>
> 了解更多 [不稳定特性](../../runtime/unstable)。

Deno 支持 [import 映射](https://github.com/WICG/import-maps)。

import 映射须用 `--importmap=<FILE>` CLI 选项开启。

目前限制：

- import 映射一一对应
- 尚未支持多 URL 兜底备用
- Deno 不支持 `std:` 命名空间
- 仅支持 `file:`, `http:` and `https:` 协议

示例:

```js
// import_map.json

{
   "imports": {
      "http/": "https://deno.land/std/http/"
   }
}
```

```ts
// hello_server.ts

import { serve } from "http/server.ts";

const body = new TextEncoder().encode("Hello World\n");
for await (const req of serve(":8000")) {
  req.respond({ body });
}
```

```shell
$ deno run --importmap=import_map.json hello_server.ts
```
