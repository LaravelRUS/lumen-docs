# Кэш

- [Настройка](#configuration)
- [Основы](#basic-usage)

<a name="configuration"></a>
## Настройка

`CACHE_DRIVER` опция в файле `.env` с помощью которой устанавливается драйвер для вашего приложения. Конечно же Lumen поддерживает весь набор драйверов Laravel, в том чиле Memcached и Redis:

- `array`
- `file`
- `memcached`
- `redis`
- `database`

> **Примечание:** Не забудьте раскомментировать строчку`Dotenv::load()` в файле `bootstrap/app.php` для того, что бы Lumen подхватил файл `.env`

### Memcached

Если вы хотите использовать драйвер Memcached, то необходимо выставить опции `MEMCACHED_HOST` и `MEMCACHED_PORT` в конфигурационном файле `.env`.

### Redis

Перед использованием Redis вам необходимо установить пакеты `predis/predis` (~1.0) и `illuminate/redis` (~5.0) через Composer.

### Database

Если вы хотите использовать базу данных для хранения данных кэша, добавьте в неё соответствующую таблицу. Она должна выглядеть примерно так:

	Schema::create('cache', function($table) {
	    $table->string('key')->unique();
	    $table->text('value');
	    $table->integer('expiration');
	});

<a name="basic-usage"></a>
## Основы

> **Примечание:** Для использования `Cache` фасадов, вам нужно раскомментировать `$app->withFacades()` в файле `bootstrap/app.php`, если конечно вы хотите их использовать.

#### Сохранение элемента в кэш

	Cache::put('key', 'value', $minutes);

#### Использование объектов Carbon для установки времени жизни элемента

	$expiresAt = Carbon::now()->addMinutes(10);

	Cache::put('key', 'value', $expiresAt);

#### Сохранение элемента в кэше если он не существует

	Cache::add('key', 'value', $minutes);

Метод `add` вернёт `true` если элемент был действительно **добавлен** в кэш. В противном случае (элемент уже существует) вернёт `false`.

#### Проверка на существование элемента в кэше

	if (Cache::has('key')) {
		//
	}

#### Получение элемента из кэша

	$value = Cache::get('key');

#### Получение элемента или возврат значения по умолчанию

	$value = Cache::get('key', 'default');

	$value = Cache::get('key', function() { return 'default'; });

#### Сохранение элемента в кэше на постоянное хранение

	Cache::forever('key', 'value');

Иногда вам может понадобиться получить элемент из кэша или сохранить его там, если он не существует. Вы можете сделать это методом `Cache::remember` method:

	$value = Cache::remember('users', $minutes, function() {
		return DB::table('users')->get();
	});

Вы так же можете комбинировать методы `remember` и `forever`:

	$value = Cache::rememberForever('users', function() {
		return DB::table('users')->get();
	});

Обратите внимание, что все кэшируемые данные сериализуются, поэтому вы можете хранить любые типы данных.

#### Изъятие элемента из кэша

Если вым необходимо получить элемент из кэша, а потом удалить его, вы можете воспользоваться методом `pull`:

	$value = Cache::pull('key');

#### Удаление элемента из кэша

	Cache::forget('key');
