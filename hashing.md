# Хеширование

- [Введение](#introduction)
- [Основы](#basic-usage)

<a name="introduction"></a>
## Введение

Фасад `Hash` в Lumen предоставляет безопасный механизм хранения паролей с использованием Bcrypt хеширование.

<a name="basic-usage"></a>
## Введение

> **Note:** Если вы намерены использовать `Hash` фасад, то расскомментируйте `$app->withFacades()` в файле `bootstrap/app.php`.

#### Хеширование пароля с использованием Bcrypt

	$password = Hash::make('secret');

Либо можете использовать функцию-хелпер `bcrypt`:

	$password = bcrypt('secret');

#### Сравнение пароля с хешэм

	if (Hash::check('secret', $hashedPassword))	{
		// The passwords match...
	}

#### Проверка на необходимость перехеширования пароля

	if (Hash::needsRehash($hashed))	{
		$hashed = Hash::make('secret');
	}
