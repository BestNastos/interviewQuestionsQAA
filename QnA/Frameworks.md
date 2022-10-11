# Frameworks

### 1. Что вы знаете о TestNG?
TestNG - бесплатный фреймворк для тестирования. 

Предоставляет:
- ассерты
- многопоточность 
- дата провайдеры 
- аннотации

**Ассерты**:

    Assert.assertEquals(actual, expected, message);
    Assert.assertNotEquals(actual, expected, message);
    Assert.assertNotNull(object, message);
    Assert.assertNull(object, message);
    Assert.assertFalse(condition, message);
    Assert.assertTrue(condition, message);
    Assert.assertSame(object, object, message);
    Assert.assertNotSame(object, object, message);
    Assert.assertEqualsNoOrder(array, array, message);
    // etc.

**Soft Assert vs Hard Assert:**

`SoftAssert` требует создания объекта. Тест не падает сразу,
если софт ассерт не прошел. Тест доходит до конца и только после вызова
метода `assertAll()` падает, если есть хотябы 1 непрошедший ассерт.


    SoftAssert softAssert = new SoftAssert();
    softAssert.assertEquals(actual, expected, message);
    softAssert.assertNull(object, message);
    softAssert.assertAll();

**TestNG vs JUnit:** 

В JUnit нельзя группировать тесты без помощи доп. тулов.

В JUnit нет dependsOn (но, с другой стороны, есть @Order).

**Аннотации:**

Аннотации TestNG ставятся над методами. 

`@Test` - данная аннотация ставится над самим тестом. 
Может иметь следующие параметры: <br />
`groups`<br />
`enabled`<br /> 
`dependsOnGroups` / `dependsOnMethods`<br />
`invocationCount` & `threadPoolSize`<br />
`dataProvider`&`DataProvideClass`<br /> 
`retryAnalyzer` - для использования этого параметра 
нужно имплементировать `IRetryAnalyzer` )<br />

Следующие аннотации указывают на очередность выполнения методов:
`@BeforeGroups`  <br /> 
`@BeforeClass` <br />
`@BeforeTest` - before all methods within <test> tag, specified in testng.xml<br />
`@BeforeMethod` - before each test<br />
`@BeforeSuite` - before whole test run once<br />


`@DataProvider`(parallel по количеству подмассивов)<br />
`@Parameters`(parameter name from xml)<br />

**Дата провайдеры:** 
Это - способ параметризовать тестовый метод. 
Дата провайдеры - методы, возвращающие двумерный массив, 
каждый подмассив которого подается в виде параметров 
соответствующему тестовому методу. Помечаются соответствующей 
аннотацией:

    @DataProvider
    private Object[][] simpleDataProvider(){
        return new Object[][]{
                         {7, "hello"},
                         {9, "world"}
        };
    }


    @Test(dataProvider = "simpleDataProvider")
    public void simpleTest(int i, String str) {
        System.out.println("int = " + i + "\n" + "String = " +str);
    }


TestNG XML - конфигурационный файл, который определяет 
test suite, в котором указывется тэг, по которому 
будут запускаться те или иные группы тестов, там же 
можно передавать параметры в тест или делать параллельный запуск.

Параллельный запуск тестов. Если тесты с дата провайдером, то parallel=true можно указать в аннотации DataProvider, либо же к аннотации @Test вместе с invokationCount и threadPoolSize  или в конфигурационном файле (by methods, threadcount).

**Исключения testng:** 

При ошибке валидации кидает джавовый `java.lang.AssertionError`.

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

### 2. Что вы знаете о TestRail?
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
