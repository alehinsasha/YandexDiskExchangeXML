# 1С SDK для Яндекс.Диск REST API
SDK представлен обработкой, которая позволит 1С взаимодействовать с файлами на Яндекс.Диске. Реализованы только основные методы API. Обработка имеет графический интерфейс, в котором можно проверить работоспособность.

## Возможности

- OAuth-авторизация
- Обзор файлов
- Доступ к папке приложения
- Добавление папок
- Загрузка файлов на Яндекс.Диск
- Скачивание файлов с Яндекс.Диска
- Удаление файлов и папок

## Использование

1. Зарегистрируйте свое приложение на OAuth-сервере https://oauth.yandex.ru/, включите необходимые права на вкладке "Яндекс.Диск REST API", оставьте Callback URL по умолчанию — https://oauth.yandex.ru/verification_code
2. Выгрузите обработку из конфигурации этого проекта
3. Загрузите обработку в конфигурацию или подключите ее через подсистему дополнительных отчетов и обработок из БСП.
4. Получите код авторизации
    41. Перейдите по адресу https://oauth.yandex.ru/authorize?response_type=code&client_id=<ID приложения> от имени пользователя, аккаунт которого вы хотите использовать.
    42. Разрешите доступ к Яндекс.Диску вашему приложению, код авторизации появится на экране.
    43. Можете реализовать автоматическое получение кода авторизации аналогично тому, как это сделано на основной форме обработки по команде ЗаполнитьКодАвторизации.
5. Используйте процедуру "Токен" из модуля объекта обработки для обмена полученного кода авторизации на OAuth-токен. Токен выдается на один год и используется во всех прикладных методах.
6. Используйте программный интерфейс объекта обработки, модуль объекта содержит подробные комментарии.

## Папка приложения
REST API, в отличие от Web DAV, позволяет использовать "Папки приложений". Если при регистрации приложения на OAuth-сервере задать соответствующий доступ, то приложение сможет использовать только данные из своей собственной папки Приложения/<Название приложения>. Эту возможность можно использовать, например, при организации обменов между базами, избавляя пользователя от необходимости выбирать папку для обмена.

## Примеры использования
### Инициализация
	// Инициализируем объект из конфигурации
	ЯндексДиск = Обработки.ОбменЯндексДиск.Создать();
	
	// Или инициализируем объект из подсистемы доп обработок БСП 2.3
	ЯндексДиск = ДополнительныеОтчетыИОбработки.ОбъектВнешнейОбработки(СсылкаНаДопОбработку);
	
	ЯндексДиск.IDПриложения = Константы.IDПриложенияЯндексДиск.Получить();
	ЯндексДиск.ПарольПриложения = Константы.ПарольПриложенияЯндексДиск.Получить();
	ЯндексДиск.КодАвторизации = КодАвторизации;
	
	ЯндексДиск.Токен();

### Просмотр и скачивание файлов
	// Получаем список файлов в папке приложения
	ЯндексДиск.СписокФайлов("app:/");

	ВременныйФайл = ПолучитьИмяВременногоФайла("xml");

	// Ищем файл для загрузки
	Для Каждого Стр Из ЯндексДиск.СписокФайлов Цикл
		Если Стр.Тип = "file" И СтрНачинаетсяС(Стр.Имя, "Message1to2") Тогда

			// Скачиваем файл
			АдресФайла = ЯндексДиск.СкачатьФайл(Стр.Путь);
			ДанныеФайла = ПолучитьИзВременногоХранилища(АдресФайла);
			ДанныеФайла.Записать(ВременныйФайл);

			// Читаем данные из файла
			ЧтениеXML = Новый ЧтениеXML;
			ЧтениеXML.ОткрытьФайл(ВременныйФайл);

			// (...) Обработка объекта ЧтениеXML

		КонецЕсли;
	КонецЦикла;

	УдалитьФайлы(ВременныйФайл);

### Загрузка файлов на Яндекс.Диск

	ВременныйФайл = ПолучитьИмяВременногоФайла("xml");

	// Подготавливаем файл для выгрузки
	ЗаписьXML = Новый ЗаписьXML;
	ЗаписьXML.ОткрытьФайл(ВременныйФайл);
	ЗаписьXML.ЗаписатьБезОбработки(ТекстXML);
	ЗаписьXML.Закрыть();

	// Выгружаем данные
	АдресФайла = ПоместитьВоВременноеХранилище(Новый ДвоичныеДанные(ВременныйФайл));
	ЯндексДиск.ЗагрузитьФайл("app:/Message2to1.xml", АдресФайла, Истина);

	УдалитьФайлы(ВременныйФайл);

## Ограничения и особенности

- Версия платформы 8.3.6 и выше
- Используются кроссплатформенные объекты для работы с HTTP запросами
