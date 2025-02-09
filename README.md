# Домашнее задание к занятию «1.2. Тестирование API, CI»

## Задача №1: настройка CI

Напоминаем, CI — это чаще всего отдельная система — сервер, набор серверов, облако — в которой ваш код и ваши автотесты собираются в автоматическом режиме без вашего непосредственного участия. Вы лишь настраиваете CI для того, чтобы при возникновении определённых событий, например, push в репозиторий, стартовал процесс сборки и прогона тестов.

Детальнее про CI вы можете узнать, погуглив «Continuous integration», «Jenkins», «GitLab CI», «AppVeyor», «Travis», «CircleCI», «GitHub Actions».

Получить код проекта для решения первой и второй задач можно выполнив следующие шаги:

Клонируйте репозиторий с примерами учебного кода во временную папку на локальный компьютер
Из папки /api-ci/rest локальной копии репозитория aqa-code копируйте содержимое в папку проекта с решением задачи
Важно: иногда можно настроить CI так, что там всегда будет success 😈! Не забывайте убедиться, что в CI сборка действительно падает, если вы запушите в GitHub падающий тест.

Подсказка
Возможно, это как-то связано с файлом gradlew и правами доступа на него. Для добавления прав на запуск файла gradlew, добавьте в CI исполнение команды chmod +x gradlew перед тем как использовать этот файл как команду для работы с гредлом.

Общая схема работы выглядит следующим образом: CI должен запустить целевой сервис, который вы и тестируете, в фоновом режиме и ваши автотесты. Для этого мы будем на этот раз использовать возможности Bash.

Для того чтобы запустить целевой сервис, есть несколько вариантов, самый простой из которых — положить JAR-файл прямо в ваш репозиторий. Когда AppVeyor будет выкачивать исходники автотестов, он выкачает и ваш сервис.

Конечно, вы должны понимать, что в реальной жизни артефакты (собранный целевой сервис) хранятся в специальных системах, и процесс выкачивания будет зависеть от того, где и как хранится артефакт.

Ваш целевой сервис (SUT — System under test), расположен в файле app-mbank.jar, этот же файл используется в примерах на лекции. Вам нужно его положить в каталог artifacts вашего проекта, который необходимо создать.

Поскольку файлы с расширением .jar находятся в списках .gitignore, вам нужно принудительно заставить Git следить за ними: git add -f artifacts/app-mbank.jar.

После чего сделать git push. Обязательно удостоверьтесь, что файл попал в репозиторий.

## Задача №2: JSON Schema

JSON Schema предлагает нам инструмент валидации JSON-документов. С описанием вы можете познакомиться по адресу.

Как строится схема:
```json
{
  "$schema": "http://json-schema.org/draft-07/schema", // версия схемы: https://json-schema.org/understanding-json-schema/reference/schema.html
  "type": "array", // тип корневого элемента: https://json-schema.org/understanding-json-schema/reference/type.html
  "items": { // какие элементы допустимы внутри массива: https://json-schema.org/understanding-json-schema/reference/array.html#items
    "type": "object", // должны быть объектами: https://json-schema.org/understanding-json-schema/reference/object.html
    "required": [ // должны содержать следующие поля: https://json-schema.org/understanding-json-schema/reference/object.html#required-properties
      "id",
      "name",
      "number",
      "balance",
      "currency"
    ],
    "additionalProperties": false, // дополнительных полей быть не должно 
    "properties": { // описание полей: https://json-schema.org/understanding-json-schema/reference/object.html#properties
      "id": {
        "type": "integer" // целое число: https://json-schema.org/understanding-json-schema/reference/numeric.html#integer
      },
      "name": {
        "type": "string", // строка: https://json-schema.org/understanding-json-schema/reference/string.html
        "minLength": 1 // минимальная длина — 1: https://json-schema.org/understanding-json-schema/reference/string.html#length
      },
      "number": {
        "type": "string", // строка: https://json-schema.org/understanding-json-schema/reference/string.html
        "pattern": "^•• \\d{4}$" // соответствует регулярному выражению: https://json-schema.org/understanding-json-schema/reference/string.html#regular-expressions
      },
      "balance": {
        "type": "integer" // целое число: https://json-schema.org/understanding-json-schema/reference/numeric.html#integer
      },
      "currency": {
        "type": "string" // строка: https://json-schema.org/understanding-json-schema/reference/string.html
      }
    }
  }
}
```
Начнем с изучения проекта, проверим необходимые условия для его реализации.