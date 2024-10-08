# Отправка и получение сообщений

Как только вы запустите своего бота с помощью `bot.start()`, grammY предоставит
вашим слушателям сообщения, которые пользователи отправляют вашему боту. grammY
также предоставляет методы, позволяющие легко отвечать на эти сообщения.

## Получение сообщений

Самый простой способ прослушивания сообщений --- через `bot.on("message")`

```ts
bot.on("message", async (ctx) => {
  const message = ctx.message; // объект сообщения
});
```

Однако есть ряд и других вариантов.

```ts
// Обработка команд, например /start
bot.command("start", async (ctx) => {/* ... */});

// Сопоставляет текст сообщения со строкой или регулярным выражением.
bot.hears(/echo *(.+)?/, async (ctx) => {/* ... */});
```

Вы можете использовать автоподсказку в редакторе кода, чтобы увидеть все
доступные варианты, или посмотреть [все методы](/ref/core/composer) класса
`Composer`.

> [Подробнее](./filter-queries) о фильтрации определенных типов сообщений с
> помощью `bot.on()`.

## Отправка сообщений

Все методы, которые могут использовать боты
(**[важный список](https://core.telegram.org/bots/api#available-methods)**),
доступны в объекте `bot.api`.

```ts
// Отправить сообщению по ID пользователя 12345.
await bot.api.sendMessage(12345, "Привет!");
// В качестве опций вы можете передать объект options.
await bot.api.sendMessage(12345, "Привет!", {/* больше опций */});
// Просмотрите объект отправленного сообщения
const message = await bot.api.sendMessage(12345, "Привет!");
console.log(message.message_id);

// Получите информацию о самом боте.
const me = await bot.api.getMe();

// т.д.
```

Каждый метод принимает необязательный объект options типа `Other`, который
позволяет задать дополнительные параметры для ваших вызовов API. Эти объекты
опций в точности соответствуют опциям, которые вы можете найти в списке методов
по ссылке выше. Вы также можете использовать автоподсказку в вашем редакторе
кода, чтобы увидеть все доступные параметры, или посмотреть
[все методы](/ref/core/api) класса `Api`. На остальной части этой страницы
показаны некоторые примеры.

Также посмотрите [следующий раздел](./context), чтобы узнать, как объект
контекста слушателя делает отправку сообщений легким делом!

## Отправка сообщений с ответом на сообщение

Вы можете использовать функцию Telegram reply-to, указав идентификатор
сообщения, на которое нужно ответить, с помощью `reply_parameters`.

```ts
bot.hears("ping", async (ctx) => {
  // `reply` - это псевдоним для `sendMessage` в том же чате (см. следующий раздел).
  await ctx.reply("pong", {
    // `reply_parameters` задает фактическую функцию ответа.
    reply_parameters: { message_id: ctx.msg.message_id },
  });
});
```

> Обратите внимание, что отправка сообщения через `ctx.reply` не означает, что
> вы автоматически отвечаете на что-либо. Вместо этого вы должны указать
> `reply_parameters` для этого. Функция `ctx.reply` --- это всего лишь псевдоним
> для `ctx.api.sendMessage`, см.
> [следующий раздел](./context#доступные-деиствия).

Параметры ответа также позволяют вам отвечать на сообщения в других чатах, а
также цитировать части сообщения - или даже и то, и другое одновременно!
Ознакомьтесь с документацией API бота
[документация параметров ответа](https://core.telegram.org/bots/api#replyparameters).

## Отправка сообщения с форматированием

> Ознакомьтесь с
> [разделом о параметрах форматирования](https://core.telegram.org/bots/api#formatting-options)
> в Telegram Bot API, написанным командой Telegram.

Вы можете отправлять сообщения с **жирным** или _курсивным_ текстом,
использовать URL-адреса и многое другое. Есть два способа сделать это, как
описано в
[разделе о параметрах форматирования](https://core.telegram.org/bots/api#formatting-options),
а именно: Markdown и HTML.

### Markdown

> Смотрите <https://core.telegram.org/bots/api#markdownv2-style>

Отправьте сообщение с пометкой markdown в тексте и укажите
`parse_mode: "MarkdownV2"`.

```ts
await bot.api.sendMessage(
  12345,
  "*Привет\\!* _Добро пожаловать_ в [grammY](https://grammy.dev)\\.",
  { parse_mode: "MarkdownV2" },
);
```

### HTML

> Смотрите <https://core.telegram.org/bots/api#html-style>

Отправьте сообщение с HTML-элементами в тексте и укажите `parse_mode: "HTML"`.

```ts
await bot.api.sendMessage(
  12345,
  '<b>Привет!</b> <i>Добро пожаловать</i> в <a href="https://grammy.dev">grammY</a>.',
  { parse_mode: "HTML" },
);
```

## Отправка файлов

Более подробно работа с файлами описана в
[следующем разделе](./files#отправка-фаилов).

## Принудительный ответ​

> Это может быть полезно, если ваш бот работает в
> [режиме конфиденциальности](https://core.telegram.org/bots/features#privacy-mode)
> в групповых чатах.

Когда вы отправляете сообщение, вы можете сделать так, чтобы клиент Telegram
пользователя автоматически указывал это сообщение как ответ. Это означает, что
пользователь будет отвечать на сообщение вашего бота автоматически (если только
он не удалит ответ вручную). В результате ваш бот будет получать сообщения
пользователей даже при работе в режиме
[конфиденциальности](https://core.telegram.org/bots/features#privacy-mode) в
групповых чатах.

Принудительно ответить можно следующим образом:

```ts
bot.command("start", async (ctx) => {
  await ctx.reply(
    "Привет! Я могу читать только те сообщения, в которых отвечают на мои сообщения!",
    {
      // Сделайте так, чтобы Telegram клиенты автоматически показывали пользователю интерфейс ответа.
      reply_markup: { force_reply: true },
    },
  );
});
```
