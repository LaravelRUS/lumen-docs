# Файловая система / Облачное хранилище

- [Введение](#introduction)
- [Конфигурация](#configuration)
- [Основы](#basic-usage)

<a name="introduction"></a>
## Введение

Lumen даёт прекрасные механизмы для работы с файловой системой благодаря пакету [Flysystem](https://github.com/thephpleague/flysystem), написанным Frank de Jonge. Интеграция этого пакета позволяет легко работать как с локальной файловой системой так и с Amazon S3, Rackspace облачными хранилищами. Ещё лучше то, что вы можете переключаться между этими видами хранилищ используя один и тот же API!

<a name="configuration"></a>
## Конфигурация

Конфигурационные опции находятся в файле `.env`. Как их использовать, можно посмотреть в файле `.env.example`.

Before using the S3 or Rackspace drivers, you will need to install the appropriate package via Composer:
Перед использованием драйвера S3 или Rackspace, вам необходимо установить соответствующий пакет с помощью Composer:

- Amazon S3: `league/flysystem-aws-s3-v2 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

Обратите внимание, что все операции с локальным хранилищем производятся в отношении директории `storage/app`. Таким образом, следующий метод будет хранить данные в файле `storage/app/file.txt`:

	Storage::disk('local')->put('file.txt', 'Contents');

<a name="basic-usage"></a>
## Основы

> **Примечание:** Если вы намерены использовать `Storage` фасад, то расскомментируйте `$app->withFacades()` в файле `bootstrap/app.php`.

`Storage` фасад может быть использован для взаимодействия с любым из настроенных дисков. Кроме того, возможно внедрить в конструктор класса объект, реализующий `Illuminate\Contracts\Filesystem\Factory`, используя [сервис-контейнер](/docs/container).

#### Подключение диска

	$disk = Storage::disk('s3');

	$disk = Storage::disk('local');

#### Определение существования файла

	$exists = Storage::disk('s3')->exists('file.jpg');

#### Выполнение метода на диске по умолчанию

	if (Storage::exists('file.jpg')) {
		//
	}

#### Получение содержимого файла

	$contents = Storage::get('file.jpg');

#### Запись в файл

	Storage::put('file.jpg', $contents);

#### Добавление контента в начало файла

	Storage::prepend('file.log', 'Prepended Text');

#### Добавление контента в конец файла

	Storage::append('file.log', 'Appended Text');

#### Удаление файла/файлов

	Storage::delete('file.jpg');

	Storage::delete(['file1.jpg', 'file2.jpg']);

#### Копирование файла

	Storage::copy('old/file1.jpg', 'new/file1.jpg');

#### Перемещение файла

	Storage::move('old/file1.jpg', 'new/file1.jpg');

#### Получение размера файла

	$size = Storage::size('file1.jpg');

#### Получить время последнего изменения файла (UNIX)

	$time = Storage::lastModified('file1.jpg');

#### Получить все файлы в директории

	$files = Storage::files($directory);

	// Рекурсивно...
	$files = Storage::allFiles($directory);

#### Получить все поддиректории в директории

	$directories = Storage::directories($directory);

	// Рекурсивно...
	$directories = Storage::allDirectories($directory);

#### Создание директории

	Storage::makeDirectory($directory);

#### Удаление директории

	Storage::deleteDirectory($directory);
