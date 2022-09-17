
# Build Tools & CI & CD

### 1. Расскажите о фазах цикла Maven
Apache Maven — фреймворк для автоматизации сборки проектов на 
основе описания их структуры в файлах на языке POM, 
являющемся подмножеством XML. 

**Существует 3 цикла Мавен:**
- `clean`: удаление файлов сгенерированных во время предыдущего билда;
- `site`: сгенерировать проектную документацию (в формате html);
- `default`: стандартный цикл со всеми остальными фазами.

Что происходит во время каждой фазы `default` цикла? Чтобы во время
конкретной фазы что-то вообще происходило, необходимо задать 
плагину `goal`, привязанный к этой фазе. В противном случае в эту
фазу выполняться не будет ничего. 

**Пример**:
    
    <plugin>
       <groupId>org.apache.maven.plugins</groupId>
       <artifactId>maven-checkstyle-plugin</artifactId>
       <version>3.2.0</version>
       <configuration>
         <configLocation>checkstyle.xml</configLocation>
         <encoding>UTF-8</encoding>
         <consoleOutput>true</consoleOutput>
         <failsOnError>true</failsOnError>
         <linkXRef>false</linkXRef>
       </configuration>
       <executions>
         <execution>
           <id>validate</id>
           <phase>validate</phase>
           <goals>
             <goal>check</goal>
           </goals>
         </execution>
       </executions>
    </plugin>

**`Default`- цикл включает фазы:**
- `validate`: то, что необходимо сделать до компиляции - что задано 
  существующими плагинами, например, проверка код стайлинга как 
  в примере выше;
- `compile`: скомпилировать исходники;
- `test`: прогонка тестов;
- `package`: создание .jar-файла;
- `verify`: хз
- `install`: помещение .jar-файла в папку m2 к остальным зависимостям;
- `deploy`: публикация в хранилище данных (артефактов). Например, 
  в Artifactory.

### 2. Что вы знаете о Gradle, каковы его отличия от Maven?
Билд скрипт прописывется в .gradle-файле, в котором используется 
Groovy или Kotlin. Gradle малословный, императивный 
(в то время как Maven - декларативный). В Gradle 
мы можем расширить стандартный билд доп. действиями, создать свои таски. 
Maven тоже расширяем всякими плагинами, но в градле можно быстро самому написать то, 
что надо именно тебе.

_**В Gradle существует 3 билд-фазы:**_
- _Инициализация_ (Initialization). На этом этапе определяется, какой проект/подпроект будет собираться,
для каждого из них создается экземпляр `Project`. В данную фазу выполняется файл 
  `settings.gradle`.
- _Конфигурация_ (Configuration). На этом этапе все объекты `Project` конфигурируются, выполняются 
билд-скрипты всех проектов (`build.gradle`)
- _Исполнение_ (Execution). Gradle определяет сабсет тасок для исполнения, созданных и 
  сконфигурированных во время фазы конфигурации. Этот сабсет определяется 
  названиями тасок, перечисленных в команде Gradle. В примере ниже сабсет тасок это
  test1 и test2: 
  
      gradle test1 test2 
  
Пример файла `settings.gradle`:

    println 'settings.gradle execution - initialization phase'

Пример содержания файла `build.gradle` из того же проекта:

    println 'build.gradle execution - configuration phase.'

    tasks.register('test1') {
              doFirst {
                println 'build.gradle execution - doFirst - execution phase.'
              }
              doLast {
                println 'build.gradle execution - doLast - execution phase.'
              }

              println 'build.gradle execution - test1 - configuration phase'
    }

Команда:

      gradle task1

Вывод:


    settings.gradle execution - initialization phase

    > Configure project :
    build.gradle execution - configuration phase.
    build.gradle execution - test1 - configuration phase

    > Task :test1
    build.gradle execution - doFirst - execution phase.
    build.gradle execution - doLast - execution phase.

    BUILD SUCCESSFUL in 0s
    1 actionable task: 1 executed

Так же в .gradle-файле можно указать дефолтовые таски, которые будут запускаться,
если в команде не были указаны никакие названия тасок (напр, команда `gradle` 
или `gradle -q`). Если в команде указаны какие-либо названия тасок, то дефолтовые
запускаться не будут. Объявление дефолтовых тасок выглядит так:

    defaultTasks 'task2'

    tasks.register('task2') {
          doLast {
            println 'task2 !'
          }
    }

Уже существующие таски, предоставляемые грэдлом:

- tasks
- help
- dependencies
- properties
и т.д.
  
Следующим образом таска dependencies используется для прописывания зависимостей:

    dependencies {
      compile group: 'io.rest-assured', name: 'rest-assured', version: '4.3.0'
    }


Также можно прописывать в .gradle файле плагины (как показано ниже). Например, 
плагин Java будет иметь доп. таски `clean`, `build`, `jar` и т.д.

    plugins {
      id "java"
    }

### Что такое непрерывная интеграция (CI)?
Continuous integration - это практика смердживания кода разработчиков в общий mainline несколько раз в день.
Особенности: сборка проекта и прогонка тестов несколько раз в день, уведомление о сборке/прогонке, интеграция с системой баг-трекинга, результаты видны всем (CI ограничен тестированием, дальше уже - continuous delivery (то, что включает deployment).
Обязанности команды:
часто пушить код,
не пушить не рабочий или не протестированный код,
не пушить когда что-то не так с билдом,
не уходить домой пока не соберется проект после запушивания.



Преимущества: все автоматизировано, код сам билдится (может занимать и 40 минут) можно пока продолжить работу. Мы смотрим, как работает наш код с остальным кодом и не сломал ли он чего.

Jenkins - выступает как CI сервер. Код - CVS - Jenkins - CI workers.
Кроме Jenkins: TeamCity, Travis.

Jenkins as service vs process - как процесс - запускаем сами в браузере (via cmd), как сервис - можно настроить и он будет запускаться, например, при включении компьютера. Сервис в Windows - обертка над процессом, происходит то же самое, локально. В реальной работе, после merge в VCS автоматически (или вручную) запускается job в Jenkins и прогоняются тесты, там уже мы все это делаем не локально, это делаетс я удаленно на других машинах (CI workers).

Build & Deploy - 1. билдить - компилировать и упаковывать в .jar, который можно потом развернуть на сервере, не смогли сбилдить значит ошибка в исходных кодах или скрипте для сборки. 2. деплоить(разворачивать) - когда размещается на сервере и запускается.
Continuous delivery это процесс разработки, при котором 
ПО производится короткими циклами, чтобы всегда была надежная 
рабочая версия. Требует сначала вручную одобрить развертывание в 
продакшен и запустить его. Continuous deployment отличается только 
тем, что не требует мануального одобрения, а деплоится автоматически.

### Gradle phases (tasks), manefest