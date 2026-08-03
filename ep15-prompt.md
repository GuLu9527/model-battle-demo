# EP15 Prompt — 8 个国产 AI 修 vue3 真实 PR

## L1 · 修注释 typo（给模型的任务描述）

```
修 vue3 编译器核心包的一个注释拼写错误。

任务：找到 packages/compiler-core/src/codegen.ts 中
CodegenSourceMapGenerator 接口的 JSDoc 注释里一处 duplicated
"to" 拼写错误（"to to" 应该是 "to"），修正它。

提示：在文件前 80 行的 JSDoc 注释里找。
```

---

## L2 · v-for + v-memo 编译崩溃（给模型的任务描述）

```
### Vue version

3.5.32 — also reproduces on every 3.5.13 through 3.5.34 (current `latest`),
and on 3.6.0-beta.12. Last working version is 3.5.12.

### Steps to reproduce

Compile the following SFC:

<script setup>
const items = [{ id: 1 }, { id: 2 }];
</script>

<template>
  <template v-for="item in items" :key="item.id" v-memo="[item]">
    <span>{{ item.id }}</span>
  </template>
</template>
```

Equivalent programmatic repro (no SFC needed):

```js
const { compileTemplate } = require('@vue/compiler-sfc');
compileTemplate({
  id: 'x',
  filename: 'x.vue',
  source: `<template v-for="item in items" :key="item.id" v-memo="[item]"><span>{{ item.id }}</span></template>`,
});
```

### What is expected?

The template compiles successfully, as it does in 3.5.12 and as the
existing test `compiler: v-memo transform > on template v-for` does for
a simple-identifier key (`:key="x"`).

### What is actually happening?

`compileTemplate` throws:

```
TypeError: Cannot read properties of undefined (reading 'trim')
    at processExpression (.../compiler-core.cjs.prod.js:4406:51)
    ...
```

The three conditions required to trigger the crash:

1. `v-for` is on a `<template>` element (not a regular element).
2. `:key` is a non-simple-identifier expression (e.g. member expression
   `item.id`, call expression `getId(item)`).
3. `v-memo` is present on the same `<template>`.

Removing any one of the three (move `v-memo` to a child, use a
destructured/simple-identifier key, or drop `v-memo`) makes the bug
disappear.

### Affected versions

| Version   | Result                       |
| --------- | ---------------------------- |
| 3.5.12    | OK                           |
| 3.5.13 → 3.5.34 (current `latest`) | THROW           |
| 3.6.0-beta.12 | THROW                      |
```

---

## L3 · keep-alive + persisted transition 钩子误触发（给模型的任务描述）

```
### Vue version

3.5.22

### Steps to reproduce

Click the button repeatedly, and you'll notice that `after-enter` is
logged in the console.

### What is expected?

The `after-enter` event should not be triggered when no transition
is applied.

### What is actually happening?

The `after-enter` event is triggered even when the element wrapped in
a `transition` is never displayed.

### System Info

```shell

```

### Any additional comments?

Discussion: #14030
```
