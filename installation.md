git b85c0d51af158b30aaeee02a53be4fa96c46ea49

---

# Установка

- [Установка Composer](#install-composer)
- [Установка Lumen](#install-lumen)
- [Требования к серверу](#server-requirements)

<a name="install-composer"></a>
## Установка Composer

Lumen использует [Composer](http://getcomposer.org) для управления зависимостями. Таким образом, перед тем как использовать Lumen, убедитесь, что у вас установлен Composer.

<a name="install-lumen"></a>
## Установка Lumen

### Через установщик Lumen

Сначала скачайте установщик Lumen с помощью Composer'а.

	composer global require "laravel/lumen-installer=~1.0"

Убедитесь, что указали путь `~/.composer/vendor/bin` в переменной PATH чтобы исполнительный файл `lumen` мог быть обнаружен вашей системой.

После установки, простая команда `lumen new` создаст новую копию Lumen в указанной вами папке. Например, `lumen new service` создаст папку с названием `service` содержащую в себе свежую копию Lumen со всеми необходимыми зависимостями. Этот способ установки намного быстрее, чем установка через Composer:

	lumen new service

### Через Composer Create-Project

Вы, так же можете установить Lumen, используя команду Composer'а `create-project` в терминале:

	composer create-project laravel/lumen --prefer-dist

<a name="server-requirements"></a>
## Требования к серверу

Фреймворк Lumen обладает некоторыми требованиями к серверу:

- PHP >= 5.4
- Расширение PHP Mcrypt
- Расширение PHP OpenSSL
- Расширение PHP Mbstring
- Расширение PHP Tokenizer

<a name="configuration"></a>
## Настройка

Lumen не требует практически никакой настройки из коробки. Вы сразу можете начать разработку!

Возможно, вы захотите настроить некоторые дополнительные компоненты Lumen, такие как:

- [Кэширование (Cache)](/docs/cache#configuration)
- [База данных (Database)](/docs/database#configuration)
- [Очереди (Queue)](/docs/queues#configuration)
- [Сессии (Session)](/docs/session#configuration)

<a name="permissions"></a>
### Разрешения

Lumen может потребоваться настройка некоторых разрешений: папки в директории `storage` должны быть открыты для записи.

<a name="pretty-urls"></a>
## Красивые URL'ы

### Apache

Фреймворк поставляется с файлом `public/.htaccess`, который нужен для того, чтобы использовать URL без `index.php`. Если вы используете Apache, для управления вашим приложением на Lumen, убедитесь, что активировали расширение `mod_rewrite`.

Если файл `.htaccess`, поставляющийся с Lumen не работает с вашим Apache, попробуйте следующий вариант:

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]

### Nginx

Следующие директивы в Nginx в настройках вашего сайта позволят использовать "Красивые" URL'ы:

	location / {
		try_files $uri $uri/ /index.php?$query_string;
	}

И конечно же, при использовании [Homestead](http://laravel.com/docs/homestead), красивые URL'ы будут сконфигурированы автоматически.
