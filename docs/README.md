**1\. Введение**

Тема проекта: ВК-бот

Цель проекта - изучить принципы создания Вк-бота на языке программирования C#, реализовать работающий чат-бот, взаимодействующее с ВК Bot API.

Функциональность:

• Обработка команд /start, /help, /info.\
• Ведение диалога с пользователями.\
• Возможность расширения (Календарь, напоминания и тд).

**2\. Выбор технологий**

| Компонент | Технология | Описание |
| --- | --- | --- |
| Язык программирования | C# | Современный язык программирования |
| VK API | VK.Bot | Популярная .NET-библиотека для работы с Bot API |
| IDE | Visual Studio / Rider | Для разработки |
| Хранение данных | JSON / SQLite / PostgreSQL | Выбирается в зависимости от проекта |
| Хостинг | Render / Railway / VPS | Развёртывание VK-бота |
| Документация | Markdown | Для технического описания |

**3\. Исследование: как работает VK-бот на C#**

**3.1 Как работает VK Bot API**

VK Bot API — это HTTP-интерфейс, через который можно получать и отправлять сообщения, обрабатывать команды и взаимодействовать с пользователями. Есть два основных способа получения обновлений:

• Long Polling — бот запрашивает новые сообщения с интервалом\
• Webhook — Telegram сам отправляет обновления на указанный URL\

Для простоты мы будем использовать Long Polling, так как он не требует публичного сервера на первом этапе.

**3.2 Основы библиотеки VK.Bot для C#**

VK.Bot (https://github.com/aiogram/aiogram) — Ресурс для создания ВК-бота. Возможности:

• Отправка и получение сообщений.
• Обработка команд.

**4\. Пошаговое создание ВК-бота на C#**

**Шаг 1: Регистрация бота в ВК**

1\. Откройте ВК и найдите бота

2\. Отправьте команду /newbot

3\. Укажите имя и username (например, MyTestBot)

4\. Скопируйте токен, он выглядит так:

123456789:ABCDefGhIJKLmNoPQRsTUVwxyZ

<img width="677" height="458" alt="скрин бота" src="https://github.com/user-attachments/assets/32bc1e2b-a156-4c3a-b337-a5dce8fb1571" />

**Шаг 2: Создание проекта**

1\. Откройте Visual Studio

2\. Создайте консольное приложение:

File → New → Project → Console App (.NET Core или .NET 6/8)

3\. Установите NuGet-пакет:

dotnet add package VK.Bot

Шаг 3: Базовая реализация бота**

Создайте файл Program.cs и вставьте следующий код:

using Vk.Bot;

using VK.Bot.Types;

using Vk.Bot.Polling;

using Vk.Bot.Types.Enums;

class Program

{

static async Task Main()

{

var botClient = new VkBotClient("YOUR\_TOKEN\_HERE");

using var cts = new CancellationTokenSource();

var receiverOptions = new ReceiverOptions

{

AllowedUpdates = Array.Empty<UpdateType>() 

};

botClient.StartReceiving(

HandleUpdateAsync,

HandleErrorAsync,

receiverOptions,

cancellationToken: cts.Token

);

var me = await botClient.GetMeAsync();

Console.WriteLine($"Бот запущен: {me.Username}");

Console.ReadLine();

cts.Cancel();

}

static async Task HandleUpdateAsync(IVkBotClient botClient, Update update, CancellationToken cancellationToken)

{

if (update.Message is not { } message || message.Text is not { } messageText)

return;

Console.WriteLine($"Сообщение от {message.Chat.Id}: {messageText}");

string response = messageText switch

{

"/start" => "Привет! Я ВК-бот на C#.",

"/help" => "Доступные команды: /start, /help, /info",

"/info" => "Этот бот написан на C# с использованием VK.Bot.",

\_ => $"Вы написали: {messageText}"

};

await botClient.SendTextMessageAsync(

chatId: message.Chat.Id,

text: response,

cancellationToken: cancellationToken);

}

static Task HandleErrorAsync(ITelegramBotClient botClient, Exception exception, CancellationToken cancellationToken)

{

Console.WriteLine($"Ошибка: {exception.Message}");

return Task.CompletedTask;

}

}

**Шаг 4: Запуск**

1\. Вставьте свой токен в строку:

var botClient = new VkBotClient("YOUR\_TOKEN\_HERE");

2\. Запустите проект: Ctrl + F5

3\. Напишите боту в VK команду /start

**Итог:**

На этом этапе у нас есть базовый VK-бот, который:

• Работает через Long Polling

• Обрабатывает команды /start, /help, /info

• Отвечает на любые текстовые сообщени

**5\. Иллюстрации и диаграммы**

**5.1. UML-диаграмма компонентов**

<img width="960" height="600" alt="ffdr83yesqcv78hua6zxt45tbye" src="https://github.com/user-attachments/assets/8bdfb64a-6684-45ec-8a30-bc3cf9a9673b" />


**5.2. Диаграмма последовательности обработки сообщений**

<img width="960" height="953" alt="CheckEmail svg" src="https://github.com/user-attachments/assets/c542062a-d925-41e9-8681-e8cdfe0e5b71" />


**5.3. Схема логики обработки команд**

\[Старт\]

\[Получено сообщение\]

\[Это команда?\] -- нет --> \[Ответ: "Вы написали..."\]

да

\[Команда /start?\] --> Ответ: "Привет!"

\[Команда /help?\] --> Ответ: "Доступные команды..."

\[Команда /info?\] --> Ответ: "Информация о боте"

**6\. Модификация проекта: Ведение дневника**

**Цель модификации**

Добавить в Vk-бота возможность сохранять и просматривать дневниковые записи пользователя. Пользователь сможет:

• Добавить запись: /add Сегодня был тяжёлый день

• Посмотреть все записи: /list

• Очистить записи: /clear

**Техническая реализация**

Мы используем:

• Словарь Dictionary<long, List<string>> для хранения записей в памяти

• Команды: /add, /list, /clear

Позже это можно заменить на SQLite или сохранение в файл.

**Пример кода**

Добавим в Program.cs:

static Dictionary<long, List<string>> diary = new();

static async Task HandleUpdateAsync(IVkBotClient botClient, Update update, CancellationToken cancellationToken)

{

if (update.Message is not { } message || message.Text is not { } messageText)

return;

var chatId = message.Chat.Id;

var text = messageText.Trim();

if (!diary.ContainsKey(chatId))

diary\[chatId\] = new List<string>();

if (text.StartsWith("/add "))

{

string entry = text.Substring(5);

diary\[chatId\].Add(entry);

await botClient.SendTextMessageAsync(chatId, "Запись добавлена!", cancellationToken: cancellationToken);

}

else if (text == "/list")

{

if (diary\[chatId\].Count == 0)

{

await botClient.SendTextMessageAsync(chatId, "У вас нет записей.", cancellationToken: cancellationToken);

}

else

{

string allEntries = string.Join("\\n— ", diary\[chatId\]);

await botClient.SendTextMessageAsync(chatId, $"Ваши записи:\\n— {allEntries}", cancellationToken: cancellationToken);

}

}

else if (text == "/clear")

{

diary\[chatId\].Clear();

await botClient.SendTextMessageAsync(chatId, "Все записи удалены.", cancellationToken: cancellationToken);

}

else

{

string response = text switch

{

"/start" => "Привет! Я бот-дневник. Используйте /add, /list, /clear",

"/help" => "Команды: /add <запись>, /list, /clear",

"/info" => "Этот бот сохраняет ваши личные записи.",

\_ => "Неизвестная команда. Введите /help"

};

await botClient.SendTextMessageAsync(chatId, response, cancellationToken: cancellationToken);

}

}

**Идеи для будущей доработки**

• Более качественные и развернутые ответы\
• Добавить фильтрацию по дате\
• Сохранять данные в базу\
• Напоминания на основе записей

