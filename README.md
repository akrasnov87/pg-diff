# Добро пожаловать в документацию pg-diff
По сути, этот инструмент командной строки помогает вам во время разработки проекта на базе базы данных PostgreSQL собирать изменения объектов, создавать исправленный сценарий SQL и сохранять версии вместе с кодом приложения.

[Оригинал документации тут](https://michaelsogos.github.io/pg-diff/)

Является [fork'ом](https://github.com/michaelsogos/pg-diff)

## Краткое описание того, зачем это использовать

Если вы применяете какие-либо передовые методы DevOps при разработке программного обеспечения и работаете в команде, то, конечно, вы сталкивались с разногласиями по поводу того, как лучше обрабатывать изменения в объектах базы данных. Вообще говоря, вы можете работать в двух разных ситуациях:

* вы используете своего рода ORM, который, вероятно, будет интегрировать стратегию миграции. Этот подход называется «сначала код». В этом случае эта библиотека вам не подходит.
* вы работаете с объектами базы данных напрямую, обычно с помощью инструментов с графическим интерфейсом, таких как pgAdmin или DBeaver. Такой подход называется «сначала база данных». Если это ваш случай, то эта библиотека станет для вас отличным дополнением.

На основе нашего многолетнего опыта, пройдя через множество способов обработки изменений объектов базы данных (не только в PostgreSQL, мы поняли, что как при управлении версиями кода, так и при выпуске программного обеспечения изменения кода должны быть тесно связаны с изменениями объектов базы данных.
Таким образом, при обновлении программного обеспечения будет легко применять изменения объектов базы данных.

Ещё одна проблема, которую мы решили, — это сохранение числовой версии SQL-скриптов, которая не работает в команде, потому что участники команды не должны знать о «следующем порядковом номере», а также чтобы избежать конфликтов.
По этой причине наша стратегия заключается в создании числовой версии SQL-скриптов, генерируемых на основе даты и времени, связанных с текущим моментом в часовом поясе UTC; это также помогает при совместной работе с удалёнными командами (например, с итальянской командой, которая работает над тем же проектом, что и индийская команда).

Конечно, эта библиотека создаёт качественный SQL-код только для PostgreSQL, но этого недостаточно. Мы не нашли инструментов (а мы много чего пробовали :смайлик:), которые могли бы гарантировать правильность сгенерированных SQL-скриптов, потому что существует множество ситуаций, в которых фрагмент SQL-кода не может быть выполнен (например, добавление нового столбца без значения по умолчанию в таблицу без указания значения по умолчанию или изменение прав доступа для пользователя, которого нет в целевой базе данных). Поэтому перед фиксацией изменений или публикацией необходимо __проверить SQL-скрипты__.

По этой причине библиотека добавляет подсказку (все подсказки начинаются со слов «ПРЕДУПРЕЖДЕНИЕ:» или «ОШИБКА:») в качестве комментария рядом с соответствующей строкой скрипта SQL, чтобы было проще выявлять потенциально проблемные команды SQL.
Это, конечно, наша __ГЛАВНАЯ ВАЖНАЯ ФУНКЦИЯ__, которой нет ни в одной из протестированных библиотек и которая может значительно ускорить проверку разработчиком соответствия скрипта SQL требованиям.

В итоге эта библиотека __НАСТОЛЬКО ПОМОГАЕТ ОБНАРУЖИВАТЬ ИЗМЕНЕНИЯ В БАЗЕ ДАННЫХ__, что в нашем случае она экономит более 50% времени, затрачиваемого на синхронизацию баз данных.

## Подготовьте свое окружение

В нашей системе DevOps, чтобы лучше работать с этой библиотекой, избежать каких-либо проблем в процессе разработки и сделать всё просто, разумно и легко, мы настроили на каждом компьютере разработчика две базы данных: одна из них — это база данных нашего приложения, которую мы называем __TARGET__, а другая — это база данных, в которую мы вносим изменения, которую мы называем __SOURCE__.
Для любых изменений, внесённых в базу данных __SOURCE__, библиотека сгенерирует скрипт исправления для базы данных __TARGET__.

Конечно, когда член команды проверяет последние изменения в коде с помощью управления версиями (нам нравятся GIT и GITFLOW, но вы можете использовать любую другую систему, например SVN и т. д.), он должен запустить\выполнить исправления SQL-скрипта в базе данных __SOURCE__ перед внесением изменений. Таким образом, все члены команды (в том числе удалённые) могут быть в курсе последних изменений в базе данных.

## Как работает эта библиотека

На самом деле эта библиотека — просто инструмент командной строки (потому что мы любим Node.js, но также работаем с другими языками и фреймворками), который не зависит от вашей IDE. Таким образом, вы можете использовать эту библиотеку, даже если не работаете с Node.js.

В качестве инструмента командной строки и для упрощения (чтобы было проще интегрироваться с вашими собственными devops) всё, что вам нужно сделать, это:

* наличие файла конфигурации JSON с именем "__pg-diff-config.json__" в котором вы укажете всё, что нужно знать библиотеке
* запустите из командной строки "__pg-diff__" с указанием, какой __configuration to use__ и какой __name__ файл сценария нужно создать

Вот и все, не торопитесь выпить чашечку кофе!

## Приступая к работе

Установите библиотеку с помощью:

<pre>
git clone https://github.com/akrasnov87/pg-diff
cd pg-diff
npm install
</pre>

Создайте конфигурационный файл в папке вашего проекта, как показано в приведенном ниже примере:

<pre>
{
  "development": {
    //Должна существовать хотя бы одна конфигурация, но их может быть несколько
    "sourceClient": {
      //Укажите здесь подключение к исходной базе данных
      "host": "localhost", //IP-адрес или имя хоста
      "port": 5432, //Порт сервера
      "database": "my-source-db", //Имя базы данных
      "user": "postgres", //Имя пользователя для доступа к базе данных, лучше иметь права администратора для доступа к схеме pg_catalog
      "password": "put-password-here", //Пароль для доступа к базе данных
      "applicationName": "pg-diff-cli" //Пользовательская строка для лучшей идентификации сеанса приложения, подключенного к серверу PostgreSQL
    },
    "targetClient": {
      //Укажите здесь подключение к целевой базе данных
      "host": "localhost",
      "port": 5432,
      "database": "my-target-db",
      "user": "postgres",
      "password": "put-password-here",
      "applicationName": "pg-diff-cli"
    },
    "compareOptions": {
      //Этот раздел обязателен
      "author": "your-name-or-nickname-or-anything-else", //Этот параметр обязателен, но строка может быть пустой
      "outputDirectory": "sqlscripts", //Абсолютный или относительный путь к каталогу, где сохраняются патчи SQL, в случае недопустимых значений (например, null, пустая строка, не строка) будет использоваться текущий рабочий каталог
      "getAuthorFromGit": false, //Если true, будет игнорироваться "author" и будет пытаться получить его из вашего GIT CONFIG (сначала из локальной конфигурации проекта, затем из глобальной конфигурации)
      "schemaCompare": {
        "namespaces": ["public", "other-namespace"], //Простая строка, содержащая только одно имя схемы или массив разделенных запятыми имен схем, для которых извлекаются объекты для сравнения, если назначено null или любое другое недопустимое объект, он автоматически извлечет все доступные схемы из базы данных
        "dropMissingTable": false, //При значении true будут обнаружены таблицы, которые существуют только в целевой базе данных, в случае, если будет сгенерирован оператор DROP
        "dropMissingView": false, //При значении true будут обнаружены представление и материализованное представление, которые существуют только в целевой базе данных, в случае, если будет сгенерирован оператор DROP
        "dropMissingFunction": false, //При значении true будут обнаружены функции, которые существуют только в целевой базе данных, в случае, если будет сгенерирован оператор DROP
        "dropMissingAggregate": false, //При значении true будут обнаружены агрегаты, которые существуют только в целевой базе данных, в случае, если будет сгенерирован оператор DROP
        "roles": [] //Список имен ролей, разделенных запятыми, для которых извлекаются разрешения GRANT и REVOKE на объекты базы данных. Если пусто, патч не будет содержать никаких разрешений
      },
      "dataCompare": {
        //Этот параметр обязателен
        "enable": true, //False для отключения сравнения записей
        "tables": [
          //Этот параметр обязателен, если указанный выше параметр "enable" имеет значение true
          {
            "tableName": "my-table-name", //Имя таблицы без схемы
            "tableSchema": "public or any-other-namespace", //Имя схемы, в которой существует таблица, если не указано иное, будет использоваться "public"
            "tableKeyFields": ["list-of-key-fields-name"] //Список имен полей, разделенных запятыми, который можно использовать для уникальной идентификации строк
          },
          {
            "tableName": "my-other-table-name",
            "tableSchema": "public or any-other-namespace",
            "tableKeyFields": ["list-of-key-fields-name"]
          }
        ]
      }
    },
    "migrationOptions": {
      //Этот раздел обязателен, только если вы хотите использовать нашу стратегию миграции
      "patchesDirectory": "db_migration", //Папка, в которой будут извлечены исправления скрипта SQL
      "historyTableName": "migrations", //Это имя таблицы, в которой будет сохранена "история миграций"
      "historyTableSchema": "public" //Это имя схемы, в которой будет создана таблица "истории миграций"
    }
  }
}
</pre>

Запустите инструмент ввода текста в командной строке:

<pre>
node main.js -c development initial-script -f ./your-config-file.json
</pre>

Будет создан файл `20180828103045123_initial-script.sql` в папке `{outputDirectory}`.

Если вам нужна помощь, введите:
</pre>
node main.js -h
</pre>

### Отличие

* добавлена возможность обернуть скрипт в транзакцию `compareOptions.transaction=true`;
* добавлена возможность пропустить генерацию схемы: `compareOptions.schemaCompare.disable=true`;
* добавлено игнорирование колонок: compareOptions.dataCompare.tables.ignoreFields:String[].

## Получение последних изменений

<pre>
git remote add upstream https://github.com/michaelsogos/pg-diff
git pull upstream master
</pre>

## Тегирование

<pre>
git tag 3.0.0
git push origin tag 3.0.0
</pre>