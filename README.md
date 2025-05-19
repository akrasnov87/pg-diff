# pg-diff
PostgreSQL schema and data comparing tool

[Documentation is here](https://michaelsogos.github.io/pg-diff/)

## Fork

Является [fork'ом](https://github.com/michaelsogos/pg-diff)

## Применение

Пример конфигурации указан в файле `pg-diff-config.json`.

<pre>
npm install
node main.js -c development initial-script -f ./workspace/your-config-file.json
</pre>

### Отличие

* добавлена возможность обернуть скрипт в транзакцию `compareOptions.transaction=true`;
* добавлена возможность пропустить генерацию схемы: `compareOptions.schemaCompare.disable=true`;

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