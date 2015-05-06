git d85007e94210beae6413fee9f5e951d14eff219f

---

# HTTP Роутинг

- [Базовый роутинг](#basic-routing)
- [Параметры роутинга](#route-parameters)
- [именованные роуты](#named-routes)
- [Группы роутов](#route-groups)
- [Префиксы роутов](#route-prefixing)
- [защита от CSRF](#csrf-protection)
- [Подмена методов (Method Spoofing)](#method-spoofing)
- [Отображение ошибок 404](#throwing-404-errors)

<a name="basic-routing"></a>
## Базовый роутинг

Вы будете задавать большинство роутов для своего приложения в файле `app/Http/routes.php`, который по умолчанию загружается в файле `bootstrap/app.php`. Как и Laravel, большинство основных роутов Lumen принимают URI и `Closure` (Замыкания):

#### Стандартный роут GET

	$app->get('/', function() {
		return 'Hello World';
	});

#### Другие страндартные роуты

	$app->post('foo/bar', function() {
		return 'Hello World';
	});

	$app->patch('foo/bar', function() {
		//
	});

	$app->put('foo/bar', function() {
		//
	});

	$app->delete('foo/bar', function() {
		//
	});

Часто вам будет необходимо генерировать URL'ы к вашим роутам. Вы это можете сделать с помощью хэлпера `url`:

	$url = url('foo');

#### Запросы роутов к контроллерам

Если вы заинтересованы в запросах роутов в классам, прочитайте документацию о [контроллерах](/docs/controllers).

<a name="route-parameters"></a>
## Парметры роутов

Конечно же, вы можете получать сегменты запрашиваемого URI в вашем роуте:

#### Базовые параметры роута

	$app->get('user/{id}', function($id) {
		return 'User '.$id;
	});

#### Ограничения параметров регулярными выражениями

> **Внимание:** Это единственная чать Lumen, которую невозможно напрямую скопировать в Laravel. Если вы решите обновить ваше приложение на Lumen в приложение на Laravel, то эти регулярные выражения должны быть перемещены в вызов метода `where` в роуте.

	$app->get('user/{name:[A-Za-z]+}', function($name) {
		//
	});

<a name="named-routes"></a>
## Именованные роуты

именованные роуты позволяют вам удобным споссобом генерировать URL'ы или перенаправлять к определенным роутам. Вы можете задать имя для роута с помощью ключа массива `as`:

	$app->get('user/profile', ['as' => 'profile', function() {
		//
	}]);

Вы, так же можете задать имя роута для метода контроллера:

	$app->get('user/profile', [
		'as' => 'profile', 'uses' => 'App\Http\Controllers\UserController@showProfile'
	]);

Теперь вы можете использовать название роута при генерации URL'ов или редиректе:

	$url = route('profile');

	$redirect = redirect()->route('profile');

<a name="route-groups"></a>
## Группы роутов

Иногда вам может потребоваться применить middleware к группе роутов. Вместо того, чтобы указывать middleware для каждого роута, вы можете использовать группу роутов.

Общие аттрибуты указываются в виде массива в первом параметре метода `$app->group()`.

<a name="route-group-middleware"></a>
### Middleware

Middleware применяется ко всем роутам внитри группы, если передавать массив аттрибутов группы роутов с ключом `middleware` и списком нужных middleware. Middleware будут применены в порядке, указанном вами в массиве:

	$app->group(['middleware' => 'foo|bar'], function($app)
	{
		$app->get('/', function() {
			// Использует Foo и Bar Middleware
		});

		$app->get('user/profile', function() {
			// Использует Foo и Bar Middleware
		});
	});

<a name="route-group-namespace"></a>
### Пространства имен

Вы можете использовать параметр `namespace` в массиве атрибутов группы, чтобы указать пространство имен для всех контроллеров внутри группы:

	$app->group(['namespace' => 'App\Http\Controllers\Admin'], function($app) {
		// Контроллеры в пространстве имен "App\Http\Controllers\Admin"
	});

<a name="route-prefixing"></a>
### Префиксы роутов

Группа роутов может иметь префикс, указываемый с помощью аттрибута `prefix` для группы:

	$app->group(['prefix' => 'admin'], function()
	{
		$app->get('users', function()
		{
			// Соответствует УРЛу "/admin/users"
		});
	});

Вы, так же, можете использовать параметр `prefix` для передачи общих параметров в ваш роут:

#### Параметры URL'а в префиксах

	$app->group(['prefix' => 'accounts/{account_id}'], function()
	{
		$app->get('detail', function($account_id)
		{
			//
		});
	});

<a name="csrf-protection"></a>
## Защита от CSRF 

> **Внимание:** Вам необходимо [активировать сессии](/docs/session#session-usage) чтобы иметь воспользоваться этой возможностью Lumen.

Lumen, как и Laravel, позволяет легко защитить ваше приложение от [межсайтовой подделки запросов (CSRF)](http://en.wikipedia.org/wiki/Cross-site_request_forgery). CSRF - это один из видов вредоносного использования команд неавторизованным пользователем, от имени авторизованного.

Lumen автоматически генерирует CSRF-ключ для каждой активной пользовательской сессии, используемой в приложении. Этот ключ позволяет убедиться в том, что только авторизованный пользователь делает эти запросы к приложению.

#### Вставка CSRF-ключа в форму

	<input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">

И конечно же, используя [шаблонизатор Blade](/docs/templates):

	<input type="hidden" name="_token" value="{{ csrf_token() }}">

Нет необходимости вручную проверять CSRF-ключ при запросах POST, PUT, или DELETE. Если это активировано в файле `bootstrap/app.php`, то `Laravel\Lumen\Http\Middleware\VerifyCsrfToken` [посредник](/docs/middleware) сам проверит то, что ключ пришедший в запросе, и хранящийся в сессии - один и тот же.

#### X-CSRF-TOKEN

В дополнение к проверке CSRF-ключа, как параметра "POST", посредник, так же будет проверять параметр заголовка запроса `X-CSRF-TOKEN`. Вы можете, например, указывать ключ в мета-тегах и добавлять его в заголовки при использовании jQuery:

	<meta name="csrf-token" content="{{ csrf_token() }}" />

	$.ajaxSetup({
		headers: {
			'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
		}
	});

Теперь все запросы AJAX будут автоматически включать в себя CSRF-ключ:

	$.ajax({
	   url: "/foo/bar",
	})

#### X-XSRF-TOKEN

Lumen, также хранит CSRF-ключ в куки `XSRF-TOKEN`. Вы можете использовать значение в куки, чтобы задать заголовок запроса `X-XSRF-TOKEN`. Некоторые Javascript-фреймворки, такие как Angular, делают это автоматичеки.

> Замечание: Разница между `X-CSRF-TOKEN` и `X-XSRF-TOKEN` в том, что первый использует простое текстовое значение, а второй использует зашифроавнное значение, так как куки в Lumen всегда зашифрованы, когда активированы глобальные посредники в файле `bootstrap/app.php`.

<a name="method-spoofing"></a>
## Подмена методов

HTML формы не поддерживают отправку с помощью методов `PUT`, `PATCH` или `DELETE`. Таким образом, при использовании роутов `PUT`, `PATCH`, либо `DELETE` вызываемых из HTML-формы, вам необходимо добавлять скрытое поле с именем `_method` к форме.

Значение, отправленное в поле с именем `_method`, будет использовано как метод отправки запроса. Например:

	<form action="/foo/bar" method="POST">
		<input type="hidden" name="_method" value="PUT">
		<input type="hidden" name="_token" value="{{ csrf_token() }}">
	</form>

<a name="throwing-404-errors"></a>
## Отобразить ошибку 404

Есть 2 способа вручную выбросить исключение 404 из роута. Во-первых, вы можете использовать хэлпер `abort`:

	abort(404);

Хэлпер `abort`, просто выбрасывает исключение `Symfony\Component\HttpFoundation\Exception\HttpException` с нужным кодом статуса.

Во-вторых, вы можете вручную выбросить экземпляр исключения `Symfony\Component\HttpKernel\Exception\NotFoundHttpException`.

Больше информации по применению исключений 404 и использованию собственных ответов на эти ошибки можно найти в документации в разделе [ошибок](/docs/errors#http-exceptions).
