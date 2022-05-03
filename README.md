## API. Настройка CI

Напоминаем, CI - это чаще всего отдельная система (сервер, набор серверов, облако), в котором ваш код и ваши авто-тесты собираются в автоматическом режиме (без вашего непосредственного участия). Вы лишь настраиваете CI для того, чтобы при возникновении определённых событий (например, push в репозиторий) стартовал процесс сборки и прогона тестов.

Детальнее про CI вы можете узнать погуглив "Continuous Integration", "Jenkins", "GitLab CI", "Appveyor", "Travis", "Circle CI", "GitHub Actions".

Что надо сделать: берёте проект с лекции, настраиваете для него CI (см. инструкцию ниже). Удостоверяетесь, что CI показывает, что сборка падает (в процессе сборки будут автоматически прогоняться все авто-тесты).

Важно: иногда можно настроить CI так, что там всегда будет SUCCESS smiling_imp! Не забывайте убедиться, что в CI сборка действительно падает, если вы запушите в GitHub падающий тест.

Возможно, это как-то связано с файлом gradlew и правами доступа на него. Для добавления прав на запуск файла gradlew, добавьте в CI исполнение команды chmod +x gradlew перед тем как использовать этот файл как команду для работы с гредлом.

Общая схема работы выглядит следующим образом: CI должен запустить целевой сервис в фоновом режиме (который вы и тестируете) и ваши авто-тесты. Для этого мы будем на этот раз использовать возможности Bash.

Для того, чтобы запустить целевой сервис есть несколько вариантов, самый простой из которых - положить jar-файл прямо в ваш репозиторий. Когда AppVeyor будет выкачивать исходники авто-тестов, он выкачает и ваш сервис.

Конечно, вы должны понимать, что в реальной жизни артефакты (собранный целевой сервис) хранятся в специальных системах и процесс выкачивания будет зависеть от того, где и как хранится артефакт.

Ваш целевой сервис (SUT - System under test), расположен в файле app-mbank.jar (в проекте с лекции). Вам нужно его положить в каталог artifacts вашего проекта (создайте его).

Поскольку файлы с расширением .jar находятся в списках .gitignoreб вам нужно принудительно заставить git следить за ними: git add -f artifacts/app-mbank.jar.

После чего сделать git push. Обязательно удостоверьтесь, что файл попал в репозиторий.

### AppVeyor

AppVeyor - одна из платформ, предоставляющих функциональность Continuous Integration. В базовом варианте - бесплатна.

####  Шаг 0. Конфигурация как код
Поскольку вручную настраивать каждый проект в системе Continuous Integration - лишняя трата времени, мы будем хранить всю конфигурацию для AppVeyor в специальном файле с названием .appveyor.yml.

Важно: внимательно посмотрите на структуру демо-репозитория из ваших лекций. Большинство инструментов используют подход "Configuration by exception" - т.е. конфигурируется только то, что не соответствует настройкам по умолчанию. Поэтому у вас всего два пути - либо использовать настройки по умолчанию и писать как можно меньше конфигурации, либо "идти против системы" и писать много конфигурации (а потом ещё и отлаживать её).

Файл этот должен храниться в самом репозитории на GitHub, тогда AppVeyor будет автоматически подхватывать настройки.

Yaml - формат данных, используемый многими системами для хранения конфигурации.

AppVeyor предлагает вам два вида серверов, на которых можно проводить сборку вашего приложения: под управлением Windows или под управлением Linux. Можно организовать сборку под несколькими сразу, но для упрощения мы пока остановимся только на одной ОС для каждого вашего проекта.

#### Linux Config
 ~~~
image: Ubuntu  # образ для сборки

stack: jdk 11  # версия JDK

branches:
  only:
    - master  # ветка git

build: off  # будем использовать свой скрипт сборки

install:
  # запускаем SUT (& означает, что в фоновом режиме - не блокируем терминал для запуска тестов)
  - java -jar ./artifacts/app-mbank.jar &

build_script:
  - ./gradlew test --info  # запускаем тест, флаг --info позволяет выводить больше информации
~~~

Естественно, у вас должен возникнуть вопрос, а что будет, если SUT не успеет стартовать к моменту запуска авто-тестов?

Тогда ваши тесты упадут. Что с этим делать и как классифицировать подобные случаи, мы поговорим на следующих лекциях.

Напоминаем, ваш build.gradle должен выглядеть вот так:

~~~
plugins {
    id 'java'
}

group 'ru.netology'
version '1.0-SNAPSHOT'

sourceCompatibility = 11
compileJava.options.encoding = 'UTF-8'
compileTestJava.options.encoding = 'UTF-8'

repositories {
    mavenCentral()
}

dependencies {
    testImplementation 'io.rest-assured:rest-assured:4.3.0'
    testImplementation 'org.junit.jupiter:junit-jupiter:5.6.1'
    testImplementation 'io.rest-assured:json-schema-validator:4.3.1'
}

test {
    useJUnitPlatform()
}
~~~

#### Шаг 1. Регистрация
#### Шаг 2. Регистрация через GitHub
AppVeyor предоставляет бесплатный тарифный план для публичных репозиториев GitHub (авторизация - также через GitHub):
#### Шаг 3. Разрешение доступа
При подключении необходимо разрешить AppVeyor получать уведомления
#### Шаг 4. Создание проекта
После авторизации станет доступной панель управления, где можно создать новый проект
Это даст возможность приложению получать уведомления о ваших push в репозиторий, модификации и т.д.

Детальнее об OAuth вы можете прочитать на:

* https://oauth.net/2/
* https://auth0.com/docs/protocols/oauth2

#### Шаг 5. Выбор репозитория

После авторизации достаточно будет нажать кнопку ADD напротив необходимого репозитория
После настройки всего процесса каждый push в ветку master GitHub-репозитория будет приводить к запуску сборки на AppVeyor.

#### Шаг 6. Status Badge

На странице Settings - Badges AppVeyor предлагает код для "бейджика" статуса вашего проекта
Этот badge необходимо разместить в файле README.md для отображения текущего статуса вашего проекта
Важно: убедитесь, что вы не скопировали бейджик с другого проекта! За такую "хитрость" ДЗ будет отправляться на доработку!
