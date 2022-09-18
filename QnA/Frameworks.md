###### Что такое Cucumber и Gherkin?

Cucumber & Gherkin notation. Cucumber это инструмент для написания автотестов.
Это слой сценария, логики тест-кейса. BDD. В Cucumber для написания тестов
используется Gherkin-нотация, которая определяет структуру теста и набор
ключевых слов. У Cucumber есть свои @Before и @After (Hooks), которые
помещаются в какой-нибудь класс из Step Definition. Класс Runner
наследуется от AbstractTestNGCucumberTests. Там же пишем следующее над
классом-раннером:
@CucumberOptions( features = { "classpath:hw6" }, tags =
{ "@DifferentElementsInterface}, glue = { "classpath:Homework.hw6.ex1" } )
@DifferentElementsInterface - ставится на тест в .feature файле.
Запускать через runner или через testng.xml где прописан путь к раннеру.
Аннотация @Test не используется.

### Что вы знаете о TestRail?
TestRail - инструмент для управления тестированием ПО.

Позволяет:
- заводить тест-кейсы и прогонять их мануально (как ALM);
- мониторить статистику;
- разбивать тесты по разным проектам / компонентам / веткам;
- интегрировать кейсы с системой баг-трекинга (например, с Jira);
- использовать TestRail API для интеграции с автотестами
  (смены статуса passed/failed при прогоне автотестов).

###### Что такое Selenide?
Selenide Page object. Страница инициализируется в @Before, специальный конструктор не нужен:
open(URL.toString()); → открывает браузер. Закрывается он сам.
homePage = page(HomePage.class); → инициализация страницы.

Selenide - обертка для Селениума.
Основные методы - это open(url), page(Page.class), $ и $$ (класс Selenide);
$(By/css) / $$(By/css) -- в документации написано doesn’t start search yet? и ссылка на find(string); find(css/By) вместо driver.findElement(By). Методы $ и $$ есть у класса Selenide и интерфейса SelenideElement, но не ElementCollection. Чтобы найти что-то внутри ElementCollection используем findBy(text(“hello”));
Помимо того, что можно на элементах сделать click(), getText() и т.д., он добавляет дополнительные удобочитаемые проверки и Conditions (интерфейс SelenideElement):
$(".profile-photo").shouldHave(text("Piter Chailovskii"));
$().should(Condition... condition) / shouldHave/ shouldBe / shouldNotBe
Conditions: visible(), text(), checked(), attribute().
shouldHave - возвращает SelenideElement чтобы можно было делать цепочки асертов.
open(String url);
homePage = page(HomePage.class)
getWebDriver();

Класс Configuration - настройки. Все поля статические.
Configuration.browser = "CHROME"; // по умолчанию
Configuration.timeout = 4000; // сколько ждем пока появится элемент
Configuration.pollingInterval = 100; // как часто провер-м появился ли
Configuration.startMaximized = true;
Configuration.browserSize = "1920x1080";

Selenide best practices
ассерты в РО,
енамы вместо хардкода,
степ-комментарии,
абстрактный класс base для наследования,
внятные названия классов, методов и переменных.
