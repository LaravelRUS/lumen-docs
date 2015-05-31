# Работа E-MAIL

- [Конфигурирование](#configuration)
- [Основы](#basic-usage)
- [Inline вложения](#embedding-inline-attachments)
- [Работа с очередью](#queueing-mail)

<a name="configuration"></a>
## Конфигурирование

> **Примечание:** По умолчанию, пакет `illuminate/mail` не входит в Lumen, по этому вам необходимо добавить `illuminate/mail` зависимость в `composer.json`.

Lumen использует библиотеку Laravel которая обеспечивает чистый и простой API к библиотеке  [SwiftMailer](http://swiftmailer.org).

Настроить работу с почтой можно в файле `.env`, для этого предназначены строчки `MAIL_*`. Как вы можете видеть, по умолчанию, SMTP сконфигурирован, но ничто не мешает настроить его как вам нравится.

Если вы хотите использовать PHP функцию `mail` для отправки почты, измените опцию `MAIL_DRIVER` в `mail`. Так же доступен драйвер `sendmail`.

### API драйвер

Lumen включает в себя драйверы отправки почты через сервисы Mailgun и Mandrill HTTP API. Как правило, отправка через API быстрее, чем через SMTP сервис. Оба этих драйвера требую что бы библиотека Guzzle 4 HTTP была установлена в вашем приложении. Вы можете добавить Guzzle 4 в ваш проект, добавив строку которая указана ниже в файл `composer.json` либо используя команды Composer:

	"guzzlehttp/guzzle": "~4.0"

#### Mailgun драйвер

Для использования Mailgun драйвера, установите опцию`MAIL_DRIVER` в `mailgun`. Затем создайте конфигурационный файл  `config/services.php` если его ещё не существует. Убедитесь, что он содержит следующие параметры:

	'mailgun' => [
		'domain' => 'your-mailgun-domain',
		'secret' => 'your-mailgun-key',
	],

#### Mandrill драйвер

Для использования Mandrill драйвера, установите опцию`MAIL_DRIVER` в `mandrill`. Затем создайте конфигурационный файл  `config/services.php` если его ещё не существует. Убедитесь, что он содержит следующие параметры:

	'mandrill' => [
		'secret' => 'your-mandrill-key',
	],

### Log драйвер

Опция `MAIL_DRIVER` может быть установлена в `log`, в этом случае все почтовые сообщения будут записаны в ваш лог-файл, а не отправлены получателю. Это прежде всего полезно для быстрой отладки и проверки содержания письма.

<a name="basic-usage"></a>
## Основы

> **Предупреждение:** Если вы хотите использовать `Mail` фасад, расскомментируйте строку `$app->withFacades()` в файле `bootstrap/app.php`.

Метод `Mail::send` используется для отправки e-mail сообщения:

	Mail::send('emails.welcome', ['key' => 'value'], function($message)	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

Первый аргумент метода `send` имя шаблона, который будет будет использоваться в содержании письма. Второй это данные которые будут переданы в виде ассоциативного массива, эти данные будут доступны в шаблоне по переменной `$key`. Третий аргумент это замыкание, которое позволяет вам установить специфические настройки сообщения.

> **Примечание:** Переменная `$message` всегда передаётся в шаблон, а так же доступна в Inline вложениях. Поэтому лучше всего избегать передачи переменной `message` через второй аргумент метода `send`, в этом нет необходимости.

Кроме HTML шаблона вы можете так же указать шаблон для простого текста:

	Mail::send(['html.view', 'text.view'], $data, $callback);

Или использовать один тип шаблона, для этого используйте ключи `html` и `text`:

	Mail::send(['text' => 'view'], $data, $callback);

Вы так же можете указывать другие опции, такие как копирование, вложение.

	Mail::send('emails.welcome', $data, function($message) {
		$message->from('us@example.com', 'Laravel');

		$message->to('foo@example.com')->cc('bar@example.com');

		$message->attach($pathToFile);
	});

Для прикрепляемых файлов вы можете указать MIME тип и отображаемое имя:

	$message->attach($pathToFile, ['as' => $display, 'mime' => $mime]);

Если вам нужно просто отправить строку используйте метод `raw`:

	Mail::raw('Text to e-mail', function($message) {
		$message->from('us@example.com', 'Laravel');

		$message->to('foo@example.com')->cc('bar@example.com');
	});

> **Примечание:** Объект $message, передаваемый функции-замыкания метода Mail::send, наследует класс сообщения SwiftMailer, это позволяет вызывать любые методы из класса для создания e-mail сообщения.

<a name="embedding-inline-attachments"></a>
## Inline вложения

Встраивание inline изображений в сообщение как правило дело не из лёгких, однако Lumen (В оригинале Laravel) обеспечивает лёгкий и удобный способ что бы встроить изображение в письмо и получить CID вложения.

#### Встраивание картинки в шаблон сообщения

	<body>
		Тут какая то картинка:

		<img src="<?php echo $message->embed($pathToFile); ?>">
	</body>

#### Встраивание Raw данных в шаблон сообщения

	<body>
		Тут находится какая то картинка состоящая из строки данных:

		<img src="<?php echo $message->embedData($data, $name); ?>">
	</body>

Обратите внимание, что переменная `$message` всегда передаётся в шаблон сообщения в виде фасада `Mail`.

<a name="queueing-mail"></a>
## Работа с очередью

#### Очередь отправки письма

> **Примечание:** Прежде чем использовать очередь вам необходимо добавить зависимость `jeremeamia/superclosure` (~2.0) в файл `composer.json`.

Отправка сообщения может резко негативна сказаться на отклике вашего приложения, поэтому многие разработчики используют очереди для отправки сообщений в фоне. С Lumen и Laravel делать это легко, благодаря [unified queue API](/docs/queues). Для того что бы поставить сообщение в очередь на отправку, просто используйте метод `queue` на фасаде `Mail`:

	Mail::queue('emails.welcome', $data, function($message)	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

с помощью метода `later` вы также можете задать количество секунд через которое будет отправлено сообщение: 

	Mail::later(5, 'emails.welcome', $data, function($message) {
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

Используйте методы `queueOn` и `laterOn` если вы хотите поместить сообщение в определённую очередь:

	Mail::queueOn('queue-name', 'emails.welcome', $data, function($message)	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});
