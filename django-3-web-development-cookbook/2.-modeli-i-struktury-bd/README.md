# 2. Модели и структуры БД

В этой главе мы рассмотрим следующие темы:

* Использование миксинов модели
* Создание миксина модели с методами, связанными с URL
* Создание миксина модели для обработки создания и модификации дат
* Создание миксина модели для управления метатегами
* Создание миксина модели для обработки общих отношений
* Работа с многоязычными полями
* Работа с таблицами перевода модели
* Избегайте циклических зависимостей
* Добавление ограничений базы данных
* Использование миграций
* Изменение внешнего ключа на поле «многие ко многим»

## Введение

Когда вы запускаете новое приложение, первое, что вы делаете, — это создаете модели, представляющие структуру вашей базы данных. Мы предполагаем, что вы уже создали приложения Django или, по крайней мере, прочитали и поняли официальное руководство по Django. В этой главе вы познакомитесь с несколькими интересными методами, которые сделают структуру вашей базы данных единообразной для различных приложений вашего проекта. Затем вы увидите, как обрабатывать интернационализацию данных в вашей базе данных. После этого вы узнаете, как избежать циклических зависимостей в ваших моделях и как установить ограничения базы данных. В конце главы вы увидите, как использовать миграции для изменения структуры вашей базы данных в процессе разработки.

## Технические требования

Для работы с кодом из этой книги вам понадобится последняя стабильная версия Python, база данных MySQL или PostgreSQL и проект Django с виртуальной средой.

Вы можете найти весь код для этой главы в каталоге **ch02** в [репозитории GitHub по адресу](https://github.com/PacktPublishing/Django-3-Web-Development-Cookbook-Fourth-Edition).
