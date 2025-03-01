# 互动菜单 (`menu`)

轻松地创建一个互动菜单。

## 简介

一个 inline keyboard 是一条消息下面的按钮数组。
grammY 有一个 [内置插件](./keyboard.md#inline-keyboards) 可以创建基本的 inline keybaords。

这个菜单插件将这个想法更进一步，让你能够在聊天里创建精美的菜单。
它们可以有交互的按钮，多个页面之间的导航，以及更多。

这里是一个简单的例子，不言自明。

<CodeGroup>
  <CodeGroupItem title="TS" active>

```ts
import { Bot } from "grammy";
import { Menu } from "@grammyjs/menu";

// 创建 bot
const bot = new Bot("token");

// 创建一个简单的菜单
const menu = new Menu("my-menu-identifier")
  .text("A", (ctx) => ctx.reply("You pressed A!")).row()
  .text("B", (ctx) => ctx.reply("You pressed B!"));

// 使其具有互动性
bot.use(menu);

bot.command("start", async (ctx) => {
  // 发送菜单：
  await ctx.reply("Check out this menu:", { reply_markup: menu });
});

bot.start();
```

</CodeGroupItem>
 <CodeGroupItem title="JS">

```js
const { Bot } = require("grammy");
const { Menu } = require("@grammyjs/menu");

// 创建 bot
const bot = new Bot("token");

// 创建一个简单的菜单
const menu = new Menu("my-menu-identifier")
  .text("A", (ctx) => ctx.reply("You pressed A!")).row()
  .text("B", (ctx) => ctx.reply("You pressed B!"));

// 使其具有互动性
bot.use(menu);

bot.command("start", async (ctx) => {
  // 发送菜单：
  await ctx.reply("Check out this menu:", { reply_markup: menu });
});

bot.start();
```

</CodeGroupItem>
 <CodeGroupItem title="Deno">

```ts
import { Bot } from "https://deno.land/x/grammy/mod.ts";
import { Menu } from "https://deno.land/x/grammy_menu/mod.ts";

// 创建 bot
const bot = new Bot("token");

// 创建一个简单的菜单
const menu = new Menu("my-menu-identifier")
  .text("A", (ctx) => ctx.reply("You pressed A!")).row()
  .text("B", (ctx) => ctx.reply("You pressed B!"));

// 使其具有互动性
bot.use(menu);

bot.command("start", async (ctx) => {
  // 发送菜单：
  await ctx.reply("Check out this menu:", { reply_markup: menu });
});

bot.start();
```

</CodeGroupItem>
</CodeGroup>

当然，如果你使用自定义的上下文类型，你也可以传递给 `Menu`。

```ts
const menu = new Menu<MyContext>("id");
```

## 添加按钮

菜单插件会像 [inline keyboard 插件](./keyboard.md#inline-keyboards) 一样布局你的键盘。
`InlineKeyboard` 类替换为 `Menu` 类。

下面是一个菜单的例子，它有四个按钮，按钮的布局是 1-2-1。

```ts
const menu = new Menu("movements")
  .text("^", (ctx) => ctx.reply("Forward!")).row()
  .text("<", (ctx) => ctx.reply("Left!"))
  .text(">", (ctx) => ctx.reply("Right!")).row()
  .text("v", (ctx) => ctx.reply("Backwards!"));
```

使用 `text` 来添加新的文本按钮。
你可以传递一个标签和一个处理函数。

使用 `row` 来结束当前行，并将所有后续按钮添加到新的一行。

还有许多可用的按钮类型，例如打开 URL。
请查看 [这个插件的 API 参考](https://doc.deno.land/https/deno.land/x/grammy_menu/mod.ts#MenuRange)，以及 [Telegram Bot API 参考](https://core.telegram.org/bots/api#inlinekeyboardbutton) 了解更多关于 `InlineKeyboardButton`。

## 发送菜单

你必须先安装一个菜单，然后才能发送它。
这样它就可以互动了。

```ts
bot.use(menu);
```

你现在可以直接传递菜单作为 `reply_markup` 发送消息。

```ts
bot.command("menu", async (ctx) => {
  await ctx.reply("Here is your menu", { reply_markup: menu });
});
```

## 动态标签

当你在按钮上放置标签字符串时，你也可以传递一个函数 `(ctx: Context) => string` 来在获取按钮上的动态标签。
这个函数可能是也可能不是 `async` 的（即异步）。

```ts
// 创建一个带有用户名的按钮，并且问候他们
const menu = new Menu("greet-me")
  .text(
    (ctx) => `Greet ${ctx.from.first_name}!`, // 动态标签
    (ctx) => ctx.reply(`Hello ${ctx.from.first_name}!`), // 处理函数
  );
```

由这样的函数生成的字符串被称为 _动态字符串_。
动态字符串是诸如切换按钮的理想选择。

请注意，你必须在你的按钮发生变化时，更新菜单。
调用 `ctx.menu.update()` 来确保你的菜单会被重新渲染。

```ts
// 已启用通知的用户标识符集合
const notifications = new Set<number>();

function toggleNotifications(id: number) {
  if (notifications.has(id)) notifications.delete(id);
  else notifications.add(id);
}

const menu = new Menu("toggle")
  .text(
    (ctx) => ctx.from && notifications.has(ctx.from.id) ? "🔔" : "🔕",
    (ctx) => {
      toggleNotifications(ctx.from.id);
      ctx.menu.update(); // 更新菜单！
    },
  );
```

::: tip 储存数据
上面的例子展示了如何使用菜单插件。
将用户设置储存在一个 `Set` 对象中并不是一个好主意，因为这样当你停止服务器时所有的数据都会丢失。

相反，如果你想储存数据，请考虑使用数据库或 [session 插件](./session.md)。
:::

## 更新或关闭菜单

当按钮处理函数被调用时，在 `ctx.menu` 上会有一些有用的函数。

如果你想重新渲染菜单，你可以调用 `ctx.menu.update()`。
这只会在你安装在你的菜单上的处理函数中生效。
当从其他中间件调用时，它将不会生效，因为它不能确定应该更新 _哪个_ 菜单。

```ts
const menu = new Menu("time")
  .text(
    () => new Date().toString(), // 按钮标签为当前时间
    (ctx) => ctx.menu.update(), // 点击按钮时更新时间
  );
```

你也可以通过编辑相应的消息来自动更新菜单。

```ts
const menu = new Menu("time")
  .text(
    () => new Date().toString(),
    (ctx) => ctx.editMessageText("Last updated: " + new Date.toString()),
  );
```

菜单将自动检测你想要编辑消息的文本，并利用这个机会来更新按钮。
因此，你可以通过自动更新菜单来避免显式调用 `ctx.menu.update()`。

调用 `ctx.menu.update()` 不会立即更新菜单。
相反，它只设置了一个标志并在中间件执行期间更新它。
这个叫做 _懒更新_。
如果你稍后编辑了消息本身，插件可以使用相同的 API 调用来更新按钮。
这是非常高效的，并且它确保了消息和 keyboard 同时被更新。

当然，如果你调用了 `ctx.menu.update()` 但是你没有编辑消息，菜单插件会在中间件执行完成之前自动更新按钮。

你可以使用 `ctx.menu.update({ immediate: true })` 来强制更新菜单。
请注意，`ctx.menu.update()` 将会返回一个 Promise，所以你需要使用 `await`!
使用 `immediate` 标志也可以用于所有你可以在 `ctx.menu` 上调用的操作。
这只在必要时使用。

如果你想关闭菜单，请调用 `ctx.menu.close()`。
同样，这也是懒惰地执行的。

## 菜单之间的导航

你可以很容易地创建多个页面，并且在它们之间导航。
每个页面都有自己的 `Menu` 实例。
`submenu` 按钮是一个让你导航到其他页面的按钮。
向后导航是通过 `back` 按钮完成的。

```ts
const main = new Menu("root-menu")
  .text("Welcome", (ctx) => ctx.reply("Hi!")).row()
  .submenu("Credits", "credits-menu");

const settings = new Menu("credits-menu")
  .text("Show Credits", (ctx) => ctx.reply("Powered by grammY"))
  .back("Go Back");
```

这两个按钮可以接受中间件处理函数，以便你可以响应导航事件。

你也可以使用 `ctx.menu.nav()` 来手动导航。
这个函数接受菜单标识字符串，并且将懒惰地执行导航。
类似地，向后导航通过 `ctx.menu.back()` 进行。

接下来，你需要将菜单实例连接起来，通过注册一个在另一个上。
将菜单注册到另一个菜单中，会自动设置它们的层级关系。正在注册的菜单是父菜单，注册的菜单是子菜单。
下面，`main` 是 `settings` 的父菜单，除非你显式地指定了另一个父菜单。
在向后导航时，将使用父级菜单。

```ts
// 注册设置菜单到主菜单
main.register(settings);
// 可选择设置不同的父级
main.register(settings, "back-from-settings-menu");
```

你可以注册任意多个菜单，并且可以嵌套任意深度。
菜单标识可以让你快速跳转到任何页面。

**请注意，你只需要给你的嵌套菜单设置一个交互即可。**
例如，只需要传递根菜单给 `bot.use`。

```ts
// 请这样做：
bot.use(main);

// 请不要这样做：
bot.use(main);
bot.use(settings);
```

## Payloads

你可以将短文本 payload 与所有文本按钮和导航按钮一起存储。
当相应的处理程序被调用时，payload 将在 `ctx.match` 中可用。
这是非常有用的，因为它让你可以在菜单中存储一些数据。

这里是一个记住创建者的菜单的例子。
其他用例可能是，例如，存储分页菜单的索引。

```ts
const menu = new Menu("pun-intended")
  .text(
    { text: "I know my creator", payload: (ctx) => ctx.from.first_name },
    (ctx) => ctx.reply(`I was created by ${ctx.match}!`),
  );

bot.use(menu);
bot.command("menu", async (ctx) => {
  await ctx.reply("I created a menu!", { reply_markup: menu });
});
```

Payloads 也能和动态范围一起使用。

## 动态范围

到目前为止，我们只看到了如何动态地改变按钮上的文本。
你也可以动态地调整菜单的结构，以便在任何时候添加和删除按钮。

::: danger 不要在信息处理过程中改变菜单
你不能在信息处理过程中创建或更改菜单。
所有菜单必须在你的机器人开始之前完全创建和注册。

在你的 bot 运行时，添加新的菜单会导致内存泄漏。
你的 bot 将会变得更慢，并且最终崩溃。
:::

你可以让一部分按钮在运行时动态生成（或者全部）。
我们把这部分菜单称为 _动态范围_。
创建动态范围的最简单方法是使用这个插件提供的 `MenuRange` 类。
`MenuRange` 为你提供了菜单的所有功能，但是它没有标识符，并且不能被注册。

```ts
function getRandomInt(minInclusive: number, maxExclusive: number) {
  return min + Math.floor(Math.random() * (max - min));
}

// 创建一个包含随机数量的按钮的菜单
const menu = new Menu("random");

menu.dynamic((_ctx) => {
  const range = new MenuRange();
  const buttonCount = getRandomInt(2, 9); // 2-8 buttons
  for (let i = 0; i < buttonCount; i++) {
    range
      .text(i.toString(), (ctx) => ctx.reply(`${i} selected`))
      .row();
  }
  return range;
});

menu.text("Generate New", (ctx) => ctx.menu.update());
```

The range builder function that you pass to `dynamic` may be `async`, so you can even perform API calls or do database communication.
你传递给 `dynamic` 的范围构造器可以是 `async`，所以你可以甚至进行 API 调用或数据库交互。

此外，范围构造器的第一个参数是上下文对象。
(在上面的例子中，这是不用的)。

你可以选择在 `ctx` 后面接收一个新的 `MenuRange` 实例。
如果你喜欢的话，你可以修改它而不是返回你自己的实例。

```ts
menu.dynamic((ctx, range) => {
  for (const text of ["foo", "bar", "baz"]) {
    range // 不需要 `new MenuRange()` 或者 `return`
      .text(text, (ctx) => ctx.reply(text))
      .row();
  }
});
```

## 手动回复 Callback 查询

这个插件会自动调用 `answerCallbackQuery` 来处理自己的按钮。
你可以设置 `autoAnswer: false` 来禁用这个功能。

```ts
const menu = new Menu("id", { autoAnswer: false });
```

现在你必须自己调用 `answerCallbackQuery`。
这允许你传递展示给用户的自定义消息。

## 过时的菜单和指纹

假设你有一个菜单，其中用户可以开关通知，比如在[上面](#动态标签)的例子中。
现在，如果用户发送 `/settings` 两次，他们将会得到相同的菜单两次。
但是，改变第一个消息中的通知设置将不会更新第二个消息！

很明显，我们不能在聊天中跟踪所有设置消息，并在整个聊天历史中更新所有菜单。
你需要使用很多 API 调用来实现这个，以至于 Telegram 会限制你的 bot。
你还需要大量存储来记住所有聊天中的每个菜单的所有消息标识符。
这是不现实的。

解决这个问题的办法是，在执行任何操作之前检查菜单是否过时。
这样，只有当用户真正开始点击菜单上的按钮时，我们才会更新过时的菜单。
菜单插件会自动为你处理这个问题，所以你不需要担心它。

你可以精确地配置当检测到过时菜单时会发生什么。
默认情况下，用户将会看到一条消息“菜单已过时，请重试！”，并且菜单将会被更新。
你可以在配置中的 `onMenuOutdated` 下定义自定义行为。

```ts
// 自定义消息
const menu0 = new Menu("id", { onMenuOutdated: "Updated, try now." });
// 自定义处理函数
const menu1 = new Menu("id", {
  onMenuOutdated: async (ctx) => {
    await ctx.answerCallbackQuery();
    await ctx.reply("Here is a fresh menu", { reply_markup: menu1 });
  },
});
// 完全禁用过时检测（可能运行错误的按钮处理程序）
const menu2 = new Menu("id", { onMenuOutdated: false });
```

我们有一个检测菜单是否过时的技巧。
它将会查看：

- 菜单的标识符，
- 菜单的形状，
- 被按下的按钮的位置，
- payload，如果指定的话。
- 被按下的按钮的文本。

这些数据被压缩成一个 4 字节的哈希，并且存储在每个按钮中。
然后，在任何处理程序运行之前，它将被比较以检查菜单是否过时。

有可能会出现这种情况，你的菜单可能会改变，但是上面的所有东西都会保持不变。
通常不会出现这种情况，但是如果你创建了一个这样的菜单，你应该使用一个指纹函数。

```ts
function ident(ctx: Context): string {
  // 返回一个字符串，当你的菜单改变时，它将会改变
}
const menu = new Menu("id", { fingerprint: (ctx) => ident(ctx) });
```

指纹字符串将被添加到上诉影响哈希生成的列表中。
这样，你可以确保过时的菜单总是被检测到。

## 它是如何工作的

这个插件完全不需要存储任何数据。
这对于有百万用户的大型机器人来说非常重要。
保存所有菜单的状态会占用太多内存。

当你创建菜单对象并通过 `register` 调用链接它们时，实际上没有菜单被构建。
相反，菜单插件将记住如何根据你的操作构建新的菜单。
每当一个菜单被发送时，它将重放这些操作以渲染你的菜单。
这包括布局所有动态范围和生成所有动态标签。
一旦菜单被发送，渲染的按钮数组将被忘记。

当用户按下一个菜单按钮时，我们需要找到在菜单渲染时被添加到这个按钮的处理程序。
因此，我们只需要再次渲染旧的菜单。
然而，这次，我们不需要完整的布局，我们只需要菜单的结构和一个特定的按钮。
因此，菜单插件将执行一个简单的渲染以获得更高的效率。
换句话说，菜单将只被部分渲染。

一旦再次知道被按下的按钮（并且我们已经检查了菜单没有[过时](#过时的菜单和指纹)），我们将调用处理程序。

在内部，菜单插件大量使用了 [API Transformer 函数](/advanced/transformers.md)，例如，以快速渲染出正在运行的菜单。

当你在一个大型层次结构中注册菜单以导航时，它们实际上不存储这些引用。
在内部，所有这个结构的菜单都被添加到同一个大型池中，并且这个池在所有包含的实例中共享。
每个菜单都对索引中的每个其他菜单负责，并且它们可以互相处理和渲染。
（大多数情况下，只有根菜单被传递给 `bot.use` 并且接收所有 update。
在这种情况下，这个实例将负责整个的池。）
因此，你能够在任意的菜单之间无限制地浏览，并且这个更新处理可以在 [`O(1)` 时间复杂度](https://en.wikipedia.org/wiki/Time_complexity#Constant_time)中发生，因为不需要在层次结构中搜索到正确的菜单来处理按钮点击。

## 插件概述

- 名字：`menu`
- 源码：<https://github.com/grammyjs/menu>
- 参考：<https://doc.deno.land/https/deno.land/x/grammy_menu/mod.ts>
