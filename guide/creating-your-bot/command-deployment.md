Регистрация слэш-команд

::: tip
Чтобы слэш-команды работали полностью, нужно три важных части кода:
	1.	Отдельные файлы команд, где описаны их структура и логика.
	2.	Обработчик команд, который динамически считывает файлы и выполняет нужную команду.
	3.	Скрипт деплоя команд — он регистрирует ваши слэш-команды в Discord, чтобы они появились в интерфейсе.

Эти шаги можно выполнять в любом порядке, но все три обязательны, чтобы команды функционировали корректно.

На этой странице описан Шаг 3 — регистрация команд. Не забудьте также выполнить предыдущие шаги!
:::

⸻

Регистрация команд

Слэш-команды можно зарегистрировать двумя способами:
	•	для одного конкретного сервера (guild) — удобно для тестов;
	•	глобально — для всех серверов, где есть ваш бот.

Сначала разберём регистрацию для одного сервера, потому что так проще разрабатывать и тестировать команды перед глобальным развёртыванием.

Вашему приложению нужно иметь scope applications.commands в сервере, чтобы слэш-команды появились, и чтобы вы могли их зарегистрировать без ошибок.

⚠️ Слэш-команды регистрируются только один раз, и обновляются только при изменении их определения (описание, опции и т.д.).
Поскольку есть дневной лимит на создание команд, не нужно подключать весь клиент к gateway или запускать регистрацию при каждом событии ready.
Для этого лучше использовать отдельный REST-скрипт — лёгкий и выполняемый вручную при необходимости обновить команды.

⸻

Команды для одного сервера (Guild commands)

Создайте в корне проекта файл deploy-commands.js.
Он будет использоваться для регистрации и обновления ваших слэш-команд.

Добавьте в файл config.json два новых параметра:
	•	clientId — ID вашего приложения (в Discord Developer Portal → “General Information” → Application ID)
	•	guildId — ID вашего тестового сервера (включите режим разработчика, кликните правой кнопкой по названию сервера → “Copy 
{
	"token": "ваш-токен",
	"clientId": "id-приложения",
	"guildId": "id-сервера"
}

Теперь используйте следующий код для deploy-commands.js:

<!-- eslint-skip -->

const { REST, Routes } = require('discord.js');
const { clientId, guildId, token } = require('./config.json');
const fs = require('node:fs');
const path = require('node:path');

const commands = [];
// Получаем все папки с командами
const foldersPath = path.join(__dirname, 'commands');
const commandFolders = fs.readdirSync(foldersPath);

for (const folder of commandFolders) {
	// Получаем все файлы команд из папки
	const commandsPath = path.join(foldersPath, folder);
	const commandFiles = fs.readdirSync(commandsPath).filter(file => file.endsWith('.js'));
	// Берём JSON-структуру каждой команды
	for (const file of commandFiles) {
		const filePath = path.join(commandsPath, file);
		const command = require(filePath);
		if ('data' in command && 'execute' in command) {
			commands.push(command.data.toJSON());
		} else {
			console.log(`[WARNING] В команде ${filePath} отсутствует свойство "data" или "execute".`);
		}
	}
}

// Создаём экземпляр REST клиента
const rest = new REST().setToken(token);

// Регистрируем команды
(async () => {
	try {
		console.log(`Начинается обновление ${commands.length} слэш-команд.`);

		// Метод PUT полностью обновляет список команд на сервере
		const data = await rest.put(
			Routes.applicationGuildCommands(clientId, guildId),
			{ body: commands },
		);

		console.log(`Успешно обновлено ${data.length} слэш-команд.`);
	} catch (error) {
		console.error(error);
	}
})();

Once you fill in these values, run `node deploy-commands.js` in your project directory to register your commands to the guild specified. If you see the success message, check for the commands in the server by typing `/`! If all goes well, you should be able to run them and see your bot's response in Discord!

<DiscordMessages>
	<DiscordMessage profile="bot">
		<template #interactions>
			<DiscordInteraction profile="user" :command="true">ping</DiscordInteraction>
		</template>
		Pong!
	</DiscordMessage>
</DiscordMessages>

### Global commands

Global application commands will be available in all the guilds your application has the `applications.commands` scope authorized in, and in direct messages by default.

To deploy global commands, you can use the same script from the [guild commands](#guild-commands) section and simply adjust the route in the script to `.applicationCommands(clientId)`

<!-- eslint-skip -->

```js {2}
await rest.put(
	Routes.applicationCommands(clientId),
	{ body: commands },
);
```

### Where to deploy

::: tip
Guild-based deployment of commands is best suited for development and testing in your own personal server. Once you're satisfied that it's ready, deploy the command globally to publish it to all guilds that your bot is in.

You may wish to have a separate application and token in the Discord Dev Portal for your dev application, to avoid duplication between your guild-based commands and the global deployment.
:::

#### Further reading

You've successfully sent a response to a slash command! However, this is only the most basic of command event and response functionality. Much more is available to enhance the user experience including:

* applying this same dynamic, modular handling approach to events with an [Event handler](/creating-your-bot/event-handling.md).
* utilising the different [Response methods](/slash-commands/response-methods.md) that can be used for slash commands.
* expanding on these examples with additional validated option types in [Advanced command creation](/slash-commands/advanced-creation.md).
* adding formatted [Embeds](/popular-topics/embeds.md) to your responses.
* enhancing the command functionality with [Buttons](/interactive-components/buttons) and [Select Menus](/interactive-components/select-menus).
* prompting the user for more information with [Modals](/interactions/modals.md).

#### Resulting code

<ResultingCode path="creating-your-bot/command-deployment" />
