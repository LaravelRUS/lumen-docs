# HTTP посредники (Middleware)

- [Введение](#introduction)
- [Применение посредников](#defining-middleware)
- [Регистрация посредников](#registering-middleware)
- [Завершающие посредники](#terminable-middleware)

<a name="introduction"></a>
## Введение

HTTP-посредники предоставляют удобный механизм для фильтрации HTTP-запросов, приходящив в ваше приложение. Например, Lumen включает посредник, который проверяет CSRF-ключ вашего приложения.

Конечно же, посредники могут быть написаны для осуществления задач более широкого круга, чем CSRF-валидации. Посредник CORS, может быть ответственным за добавление заголовков ко всем ответам от вашего приложения. Логгирующий посредник может записывать в логи все входящие запросы к вашему приложению.

Все посредники обычно находятся в папке `app/Http/Middleware`.

<a name="defining-middleware"></a>
## Применение посредников

Чтобы создать новый посредник, просто создайте класс с методом `handle`, как в следующем примере:

	public function handle($request, $next)
	{
		return $next($request);
	}

Например, давайте создадим посредник, который разрешает доступ к роуту, если возраст `age` больше 200. Иначе, перенаправить пользователя назад, к URI "home".

	<?php namespace App\Http\Middleware;

	class OldMiddleware {

		/**
		 * Запустить фильтр запросов.
		 *
		 * @param  \Illuminate\Http\Request  $request
		 * @param  \Closure  $next
		 * @return mixed
		 */
		public function handle($request, Closure $next)
		{
			if ($request->input('age') < 200) {
				return redirect('home');
			}

			return $next($request);
		}

	}

Как вы можете увидеть, если возраст `age` меньше `200`, посредник вернет клиенту HTTP-перенаправление; В противном случае, запрос пройдет дальше к приложению. Для отправки запроса глубже в приложение (позволяя проверке посредника быть "пройденной"), просто вызовите коллбэк `$next` с параметром `$request`.

Проще понять посредники, если представить их как набор "слоев", которые должен пройти HTTP-запрос, прежде чем воспользоваться приложением. Каждый "слой" может проверять запрос, и даже отвергать его целиком.

### *Before* / *After* посредники

Будет ли посредник запущен до или после запроса, зависит от самого посредника. Данный посредник будет выполнять задачи **до** того, как запрос будет обрабатываться приложением:

	<?php namespace App\Http\Middleware;

	class BeforeMiddleware implements Middleware {

		public function handle($request, Closure $next)
		{
			// Выполнить действие

			return $next($request);
		}
	}

Тогда как этот посредник будет выполнять свои задачи **после** обработки запроса приложением:

	<?php namespace App\Http\Middleware;

	class AfterMiddleware implements Middleware {

		public function handle($request, Closure $next)
		{
			$response = $next($request);

			// Выполнить действия

			return $response;
		}
	}

<a name="registering-middleware"></a>
## Регистрация посредников

### Глобальные посредники

Если вы хотите, чтобы посредники выполнялись при каждом запросе к вашему приложению, просто перечислите ваши классы посредников в вызове `$app->middleware()` ы файле `bootstrap/app.php`.

### Применение посредников к роутам

Если вы хотите применить поссредники к определенным роутам, вам следует сначала присвоить посреднику короткий ключ-псевдоним в файле `bootstrap/app.php`. По умолчанию, вызов метода `$app->routeMiddleware()` этого файла уже содержит посредников для роутов, заданных вашим приложением. Чтобы добавить свои собственные, просто допишите  их к этому списку и присвойте ключ по вашему выбору. Например:

    $app->routeMiddleware([
        'old' => 'App\Http\Middleware\OldMiddleware',
    ]);

Теперь, если задать посредник в HTTP-ядре, вы сможете использовать ключ `middleware` в опциях роута:

	$app->get('admin/profile', ['middleware' => 'old', function() {
		//
	}]);

<a name="terminable-middleware"></a>
## Завершающие посредники (terminable)

Иногда, посредникам может потребоваться проделать некоторую работу после того как ответ уже отправлен в браузер. Например, посредник "session", включенный в Laravel и Lumen, записыват данные сессии в хранилище _после_ того, как запрос отправится в браузер. Чтобы использовать это, задайте посредник как "terminable", реализуя контракт `Illuminate\Contracts\Routing\TerminableMiddleware`:

	use Closure;
	use Illuminate\Contracts\Routing\TerminableMiddleware;

	class StartSession implements TerminableMiddleware {

		public function handle($request, Closure $next)
		{
			return $next($request);
		}

		public function terminate($request, $response)
		{
			// Сохранить данные сессии...
		}

	}

Как вы можете видеть, в дополнение к методу `handle`, контракт `TerminableMiddleware` требует реализацию метода `terminate`. Этот метоб получает и запрос и ответ. После того, как вы создали завершающий посредник , необходимо добавить его в список глобальных посредников в вашем HTTP-ядре.
