# 连接外部代码

正如《[入门](../getting_started)》该章所示，我们已知悉 Deno 能够执行 URL 脚本。仿似浏览器中的 JavaScript，Deno 支持直接从 URL 导入脚本。参看下方示例，从 URL 导入一个测试断言库：

test.ts
```ts
import { assertEquals } from "https://deno.land/std/testing/asserts.ts";

assertEquals("青山悠悠路遥遥", "青山悠悠路遥遥");
assertEquals("扬鞭快马惊栖鸟", "扬鞭快马惊栖鸟");

console.log("面朝大海，春暖花开");
```

试运行之：

```shell
$ deno run test.ts
Compile file:///mnt/f9/Projects/github.com/denoland/deno/docs/test.ts
Download https://deno.land/std/testing/asserts.ts
Download https://deno.land/std/fmt/colors.ts
Download https://deno.land/std/testing/diff.ts
面朝大海，春暖花开
```

即使我们没有给 test.ts 授予 `--allow-net` 权限，它仍能访问网络去下载、导入及缓存远程模块到磁盘，全因运行时（Runtime）手持丹书铁券，特享免检免审待遇。

远程模块会被缓存到环境变量 `$DENO_DIR` 指定的特殊目录之中，既已缓存，再次运行则无需下载，如无改动，亦不会反复编译。如未设置 `$DENO_DIR` 则默认为下方的系统缓存目录：

- Linux/Redox: `$XDG_CACHE_HOME/deno` 或 `$HOME/.cache/deno`
- Windows: `%LOCALAPPDATA%/deno` (`%LOCALAPPDATA%` = `FOLDERID_LocalAppData`)
- MacOS: `$HOME/Library/Caches/deno`
- 如遇失败，则设为 `$HOME/.deno`

## FAQ

### `https://deno.land/` 万一当机，为之奈何？

开发时倚仗外部服务施惠显然相当便捷，然而在生产环境下势必受制于人，必须打包所有依赖项，自给自足。故此应将依赖项的目录纳入版本控制，并使 `$DENO_DIR` 设定为该目录。

### 如何信任可能会变更的 URL？

通过使用 `--lock` 命令行选项，可以锁定该文件，以确保运行的都是期望的代码。详情可参阅[此处](./integrity_checking)。

### 如何导入指定版本？

可在 URL 中指定版本。例如这个 URL： `https://unpkg.com/liltest@0.0.5/dist/liltest.js`，它指定了运行所需的版本。结合之前所述 `$DENO_DIR` 的应对计策，可在无需网络访问的情况下，全面把握所需运行的代码于股掌之中。

### 随处都可导入 URL 看似不够灵巧。

> 如果某个 URL 连接到某个截然不同的库版本该怎么办？

> 大型项目中到处需要维护 URL 是否容易谬误百出？

治理之道，是使用集中式的 `deps.ts` 导入依赖并重新导出外部模块（与 Node `package.json` 文件有异曲同工之意）。

譬如您在大型项目中使用了测试断言库：`"https://deno.land/std/testing/asserts.ts"`，应创建一个名为 `deps.ts` 的文件，导入该 URL 并同时导出该模块，而不必四处导入：

deps.ts
```ts
export {
  assert,
  assertEquals,
  assertStrContains,
} from "https://deno.land/std/testing/asserts.ts";
```

然后在此项目中，自始至终均从 `deps.ts` 导入模块，防止相同 URL 存在过多的引用：

```ts
import { assertEquals, runTests, test } from "./deps.ts";
```

这种设计规避了包管理、集中式代码存储库和多余的文件格式带来的过多复杂性。
