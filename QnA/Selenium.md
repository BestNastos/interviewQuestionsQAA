# Selenium Q&A

### 1. Что такое Selenium?

**_Selenium_** - фреймворк для тестирования web UI. 
Поддерживает много языков (Java, Perl, Python, C#, Ruby, Groovy, JavaScript), 
операционных систем, девайсов и браузеров (Internet Explorer, Chrome, Firefox, 
Opera, and Safari).

**_Недостатки_**: 
* подходит только для _web_ приложений
* требует знания языков программирования
* не имеет тех поддержки т.к. он _open source_

**_Selenium Software_** - это коллекция инструментов:
* _Selenium WebDriver_ - инструмент для автоматизации действий веб-браузера;
* _Selenium Remote Control (RC)_ - предшественник Selenium WebDriver;
* _Selenium IDE_ - плагин к браузеру, 
  который может записывать действия пользователя, воспроизводить их, а также 
  генерировать код для WebDriver, в котором выполняются те же самые действия. 
  В общем, это «Selenium-рекордер» с функцией record-and-playback;
* _Selenium Server and Selenium Grid_ - 
  позволяют поднять сервер, на котором можно гонять тесты в несколько потоков, 
  на разных окружениях (нодах).

Source: https://habr.com/ru/post/152653/

### 2. Что такое Selenium Server и Selenium Grid?

Это сервер, который позволяет управлять браузером с удалённой машины, по сети. 
Сначала на той машине, где должен работать браузер, устанавливается и запускается сервер. 
Затем на другой машине (технически можно и на той же самой, конечно) запускается программа, 
которая, используя специальный драйвер RemoteWebDriver, соединяется с 
сервером и отправляет ему команды. Он в свою очередь запускает браузер 
и выполняет в нём эти команды, используя драйвер, соответствующий этому браузеру.
Selenium Grid – это кластер, состоящий из нескольких Selenium-серверов. 
Он предназначен для организации распределённой сети, позволяющей параллельно запускать 
много браузеров на большом количестве машин.
Selenium Grid имеет топологию «звезда», то есть в его составе имеется выделенный сервер, 
который носит название «хаб» или «коммутатор», а остальные сервера называются «ноды» или «узлы».

Source: https://habr.com/ru/post/152653/

**P.S. Мини-инструкция, как попробовать использовать Selenium Grid локально на своей машине:**
1. Скачать Selenium Server
2. Разместить скачанный .jar файл в отдельной папке
3. Скачать chromedriver
4. Chromedriver можно разместить в той же папке
5. Добавить chromedriver в system path
6. Запустить сервер при помощи команды: 
   `"java -jar selenium-server-.jar standalone"`, или при помощи
   комбинации следующих команд:
   `"java -jar selenium-server-.jar hub"` и 
   `"java -jar selenium-server-.jar node"`
7. В тестах при создании `RemoteWebDriver` в конструкторе передать ссылку на хаб:
   `driver = new RemoteWebDriver(new URL("http://localhost:4444/wd/hub"), capabilities);`

### 3. Что такое Selenese?

Это язык для написания скриптов в Selenium IDE (устаревший).

### 4. Как работает WebDriver?

Скрипт вызывает вебдрайвер, вебдрайвер автоматизирует действия 
браузера (путь к драйверу указывается при помощи метода `setProperty`):

`System.setProperty("webdriver.chrome.driver", 
        ".\\src\\test\\resources\\driver99\\chromedriver.exe");`

Интерфейс `WebDriver` расширяет интерфейс `SearchContext`, в котором 
изначально содержатся методы `findElement()` и `findElements()`.
Из конкретных имплементаций драйвера существуют:
`ChromeDriver, RemoteWebDriver` и т.д.
Пример использования: 

    driver.findElement(By.id("submit-id"));

При использовании удаленного вебдрайвера ссылка на хаб прописывается в 
конструкторе при создании драйвера:

    driver = new RemoteWebDriver(new URL("http://localhost:4444/wd/hub"), capabilities);

### 5. Расскажите про DesiredCapabilities 

С помощью DesiredCapabilities мы задаем желаемые опции для браузера. Например имя, версию, платформу:

    capabilities = new DesiredCapabilities(); 
    capabilities.setBrowserName("chrome");
    capabilities.setVersion("100.0");
    capabilities.setPlatform(Platform.WIN10);
    driver = new RemoteWebDriver(new URL("http://localhost:4444/wd/hub"), capabilities);

### 6. Расскажите про ChromeOptions 

Класс ChromeOptions, так же как и класс DesiredCapabilities, 
наследуется от MutableCapabilities. ChromeOptions имеет доп.опции, 
например, прогон тестов в "headless chrome"-режиме, т.е. без отрисовки в браузере:<br/>

        chromeOptions.setHeadless(boolean headless)

У ChromeOptions и у DesiredCapabilities есть метод `merge`
(унаследованный от Capabilities), через который к одним capability можно добавить 
другие:

        chromeOptions.merge(Capabilities extraCapabilities)

### 7. Расскажите про fluent wait, explicit (conditional) wait и implicit wait.

**_Implicit wait_** - неявное ожидание. Задается в коде один раз и действует до тех пор,
пока работает браузер. Минусы - замедляет тесты, т.к. заставляет каждую команду 
ждать указанное количество времени:

        driver.manage().timeouts().implicitlyWait(3, TimeUnit.SECONDS);

_**Explicit wait**_ -  это ожидание, которое прописывается явно. Скрипт ждет 
выполнения определенного условия с заданным таймаутом, и только затем 
продолжается выполнение программы, если условие удовлетворено, в противном случае 
получаем исключение:

        WebDriverWait wait = new WebDriverWait(driver, 5); // 5 sec - timeout
        wait.until(ExpectedConditions.visibilityOfElementLocated(By.xpath("locator")));

Вместо `visibilityOfElementLocated` возможно использование других методов:<br/>

    elementToBeClickable
    elementToBeSelected
    presenceOfElementLocated
    visibilityOfElementLocated
    titleContains
    titleIs
    urlToBe

**_Fluent wait_** - гибкое ожидание, которое прописывается явно. Скрипт ждет
выполнения определенного условия с заданным таймаутом и polling-интервалом 
(частота опроса страницы на предмет выполнения условия), 
и только после завершения ожидания продолжается выполнение программы 
(либо выпадает исключение, если таймаут истек, а условие не выполнено):

    Wait wait = new FluentWait(driver)
                .withTimeout(5, SECONDS) // метод deprecated
                .pollingEvery(1, SECONDS) // метод deprecated
                .ignoring(StaleElementReferenceException.class)
                .until(ExpectedConditions.visibilityOfElementLocated(By.xpath("locator")));;

Класс WebDriverWait наследуется от класса FluentWait и позволяет сделать все 
то же самое даже лаконичнее:
    
    // 5 sec - timeout, 1000 millisec - polling interval
    new WebDriverWait(driver, 5, 1000) 
        .ignoring(StaleElementReferenceException.class)
        .until(ExpectedConditions.visibilityOfElementLocated(By.xpath("locator")));

### 8. Назовите самые часто используемые методы WebDriver-a

    driver.get(String)
    driver.getTitle()

    driver.findElement(By)
    driver.findElements(By)

    driver.close() // close current window
    driver.quit() // close all windows

    driver.getWindowHandle(); // for windows and tabs 
    driver.getWindowHandles();

    driver.navigate()
    driver.switchTo()
    driver.manage()

![img](E:/MyProjects/interviewQuestionsQAA/src/images/driver.png)

### 9. Расскажите про метод driver.manage()

Возвращает объект с типом ссылки `WebDriver.Options` (интерфейс), 
а у него есть методы возвращающие `WebDriver.Timeouts` 
и `WebDriver.Window` (тоже интерфейсы):

![img](E:/MyProjects/interviewQuestionsQAA/src/images/driverManage.png)

    driver.manage().window().setSize(); 
    driver.manage().window().maximize();
    driver.manage().timeouts().implicitlyWait(3, TimeUnit.SECONDS);
    driver.manage().timeouts().pageLoadTimeout(10, TimeUnit.SECONDS);

### 10. Как используется метод driver.navigate()?

Возвращает объект с типом ссылки `WebDriver.Navigation` (интерфейс),
а тот, в свою очередь, позволяет выполнять следующие действия навигации:

    driver.navigate().to(url)
    driver.navigate().to(String url)
    driver.navigate().forward()
    driver.navigate().back()
    driver.navigate().refresh()

### 11. Как используется метод driver.switchTo()

Возвращает `WebDriver.TargetLocator`, который позволяет 
переключаться между окнами и не только:

    driver.switchTo().frame(id or str)
    driver.switchTo().alert()
    driver.switchTo().window(str)

### 12. Напишите небольшой тест
Базовый класс:

    public class BaseTest {

       public WebDriver driver;

       @BeforeSuite
       public void beforeSuite(){
           System.setProperty("chromedriver", "path");
       }
    
       @BeforeMethod
       public void beforeMethod(){
           driver = new ChromeDriver();
       }
    
       @AfterMethod
       public void afterMethod(){
          driver.quit();
       }
    }

Класс с тестом:

    public class TestSelen extends BaseTest{

        @Test
        public void test(){
            driver.get("url");
            WebElement elem = driver.findElement(By.id("elementId"));
            elem.click();
            Assert.assertTrue(elem.isSelected(), "Verify that element is selected");
        }
    }

### 13. Расскажите про класс By

`By` - абстрактный класс, предоставляющий механизм поиска по локаторам. 
Не наследуется ни от чего. `By` предоставляет нам для работы 
статические методы, например:`cssSelector(String)`, `xpath(String)`,
`id(String)` и т.д.:

    driver.findElement(By.id(“456”));

А эти методы, в свою очередь, делегируют выполнение соответствующим
классам, расположенным внутри класса `By`:

    public abstract class By {

        public static By id(String id) {
            return new ById(id);
        }
    ...

Имплементируется класс `By` этими же своими внутренними классами: 
`ById`,`ByName`, `ByXpath` и т.д.:

![img](E:/MyProjects/interviewQuestionsQAA/src/images/by.png)

Устройство внутренних классов на примере класса `ById`:

    public static class ById extends By implements Serializable {

        private final String id;

        public ById(String id) {
            if (id == null) {
            throw new IllegalArgumentException("Cannot find elements when the id is null.");
        }

        this.id = id;
    } 
        
    ... // class also has methods toString() , findElement(), 
        findElements(), and field long serialVersionUID and that's it.


Так же класс `ById`, как и все внутренние классы, входящие в By, 
имеет методы `findElement()`, `findElements()`, 
которым `driver.findElement()` делегирует поиск элемента.

### 14. Что такое WebElement?

WebElement - это часть вебстраницы. Вебэлементы могуть быть:
* простыми
* составными (composite) - включают в себя несколько элементов (например, хэдер);
* сложными (complex) - элементы, для доступа к которым нужно более одного локатора (например, элементы дропдауна).

Из чего состоит веб элемент:

        <ul id="5" class="className">Hello</ul>

    ul - тэг
    id - аттрибут
    5 - значение аттрибута id
    class - аттрибут
    className - значение аттрибута class
    Hello - текст вебэлемента


Интерфейс WebElement как и WebDriver, расширяет интерфейс SearchContext, а
также TakesScreenshot. 

    public interface WebElement extends SearchContext, TakesScreenshot

Представляет собой HTML елемент. 
Из конкретных имплементаций есть: RemoteWebElement и EventFiringWebElement.

_**Самые часто используемые методы:**_

`click()`<br />
`clear()`<br />
`sendKeys()`<br />
`submit()`<br />

`findElement()`<br />
`findElements()`<br />

`getText()`<br />
`getTagName()`<br />
`getAttribute(String)`<br />

`isDisplayed()`<br />
`isSelected()`<br />
`isEnabled()`<br />

![img](E:/MyProjects/interviewQuestionsQAA/src/images/webelement.png)

### 15. Что такое Action builder?

Класс `Actions` помогает состроить действие целиком, чтобы потом применить его одним
действием `perform`. Располагает такими методами, как `clickAndHold`, `dragAndDrop`.
Для создания экземпляра класса, в конструктор нужно передать драйвер:

    new Actions(driver)
        .clickAndHold(element)
        .moveByOffset(5, 0)
        .release()
        .perform();

### 16. Что такое JavascriptExecutor и для чего он может быть полезен?

в Java есть интерфейс `JavascriptExecutor`, который позволяет производить команды Javascript. 
Например, поменять значение атрибута в HTML или кликнуть по элементу. Клик, произведенный
при помощи стандартного селениумовского метода `webElement.click()` имитирует клик юзера,
то есть он более "человекоподобный", и если нам нужно кликнуть на элемент, который,
например, перекрыт всплывающим окном, то он не подойдет. 
А клик через Javascript с этим справится.

Метод `executeScript()` принадлежит интерфейсу `JavascriptExecutor`, соответственно, чтобы его
вызвать на драйвере со ссылкой типа `WebDriver` (который тоже является интерфейсом), 
сначала надо скастить драйвер к его конкретной имплементации или к самому `JavascriptExecutor`-у:

    JavascriptExecutor js = (JavascriptExecutor) driver;
    js.executeScript("arguments[0].click();", element);

Так же при помощи JavascriptExecutor можно скроллить:

    js.executeScript("window.scrollBy(0,300)", "");

### 17. Как сделать скриншот?

Метод `getScreenshotAs()` принадлежит интерфейсу `TakesScreenshot`, соответственно, 
чтобы его вызвать на драйвере со ссылкой типа `WebDriver` 
(который тоже является интерфейсом),
сначала надо скастить драйвер к его конкретной имплементации или к самому `TakesScreenshot`-у:

    TakesScreenshot sc = (TakesScreenshot) driver;
    File screensFile = sc.getScreenshotAs(FILE); //or as BYTES in Listener for Allure

При использовании AllureReport можно следующим образом написать Listener 
для прикрепления скриншотов к отчетам (перед тестовым классом при этом ставим 
`@Listeners(Listener.class)`):

    public class Listener extends TestListenerAdapter {

        @Attachment(value = "Attachment:", type = "image/png")
        public byte[] printScreen() {
            byte[] array = {1};
            try {
                return ((TakesScreenshot) getWebDriver()).getScreenshotAs(OutputType.BYTES);
            } catch (WebDriverException e) {
            e.printStackTrace();
            }
            return array;
        }

        @Override
        public void onTestFailure(ITestResult tr) {
            printScreen();
        }
    }

### 18. Какие аннотации в Selenium вы знаете?

@FindBy (name, id, class, className, tagName, linkText, partialLinkText, css, xpath)<br/>
@FindAll - элементы должны соответствовать хотя бы 1 критерию.<br/>
@FindBys - элементы должны соответствовать всем критериям.<br/>
@CacheLookup (указывает что элемент не меняется)<br/>
@PageFactoryFinder

Пример:

    @FindBys({
    @FindBy(className = "class1")
    @FindBy(className = "class2")   })
    private List<WebElement> elements;

### 19. Какие исключения в Selenium вам встречались в работе?

    ElementNotVisibleException (элементы нераскрытого дропдауна?)
    NoSuchElementException
    WebDriverException: Connection refused
    StaleElementReferenceException

### 20. Расскажите о PageFactory

PageFactory это класс, который предоставляет статические фабричные методы для 
инициализации элементов Page Object-a, например: 

    initElements(driver, this/class);
    instantiatePage(driver, this/class);

### 21. Что такое фабрика / фабричный метод?

При сложной инициализации объекта, эта инициализация может быть обернута в 
отдельный статический метод (**фабричный метод**), скрывающий эту логику.

    PageFactory.initElements(driver, this);

**Фабрика** - похожий по своей цели паттерн. Для его создания нам нужен интерфейс, например, 
с дженерифицированным методом `create()`:

    public interface Factory<T> {
        public T create();
    }

У этого интерфейса могут быть различные реализации, которые в методе `create()` 
будут создавать что-то свое. Например, вот имплементация со строкой, 
для простоты понимания:

    class FactoryImpl implements Factory<String>{
        public String create(){
            return "Factory String";
        }
    }

Применение:

    class TestFactory{
        public static void main(String[] args) {
            new FactoryImpl().create();
        }
    }

В отличие от фабричного метода, метод фабрики - не статичный и требует создания 
объекта.

### 22. Что такое Page Object?

Page object - паттерн проектирования. Реализуется в виде класса, включающего в себя
элементы определенной страницы приложения,
а так же методы для работы с этой страницей.

    public class TopMenuPageObject extends BaseClass {

        @FindBy(id = "name")
        private WebElement loginBox;
        @FindBy(id = "password")
        private WebElement passwordBox;
        @FindBy(id = "login-button")
        private WebElement submitButton;

        public TopMenuPageObject(WebDriver driver) {
            super(driver);
            PageFactory.initElements(driver, this);
        }

        public TopMenuPageObject login(User user) {
            loginBox.sendKeys(user.getUsername());
            passwordBox.sendKeys(user.getPassword());
            submitButton.click();
            return new TopMenuPageObject(driver);
        }
    }

### 23. Что такое локатор?
Locator - это строка, уникально идентифицирующая веб-элемент. 

### 24. Какие бывают локаторы?

**A) DOM**.

DOM - Document Object Model - объектная модель, используемая для 
XML/HTML-документов. Согласно DOM-модели, документ является иерархией, деревом. 
Каждый HTML-тег образует узел дерева с типом «элемент». Вложенные в него теги 
становятся дочерними узлами. Для представления текста создаются узлы с 
типом «текст». В консоли chrome dev tools можно при помощи java script 
найти веб элементы следующими способами:

    по id: getElementById
    по имени: getElementsByName
    по классу: getElementsByClassName
    по тэгу: getElementsByTagName

Синтаксис:

    document.getElementById("reviewDialog")

Этот тип локаторов можно использовать и в Selenium:

    @FindBy(id = "name")
    private WebElement name;

DOM - локаторы самые быстрые и лаконичные, но они ограничены в своих возможностях:
нельзя отталкиваясь от одного вебэлемента подняться вверх по верстке или опуститься вниз.

**B) CSS.**

CSS - относительно быстрые и лаконичные локаторы. Гибкие - отталкиваясь 
от элемента можно опуститься вниз по DOM-модели и получить дочерний элемент. 
В отличие от XPATH, CSS не позволяет подниматься по DOM-модели вверх и 
получать таким образом родительские элементы. 
Так же, в отличие от XPATH, CSS не позволяет искать вебэлемент по тексту.

Пример нахождения локатора через консоль chrome dev tools:

    $$("span[class='benefit-txt']")
    $$("ul.panel-body-list.logs > li")
    $$("#name")
    $$(".browser > div:nth-child(4)")

Пример в Selenium:

    @FindBy(css = "#lfootercc > script")
    private WebElement element;

**C) XPATH.**

XPATH - язык запросов к элементам XML-документа. XPATH локаторы являются самыми 
универсальными, но самыми медленными. Отталкиваясь от элемента можно ходить вверх
или вниз по DOM-одели, получая 
таким образом родительские или дочерние элементы. Можно искать по тексту при помощи методов 
text() и contains().

    $x(“//span[@class='icons-benefit-base']/parent::*/following-sibling::*”)
    $x("//div[contains(text(), 'hello')]")
    $x("//div[text()='hello']")

XPATH-локатор состоит из: 

    axis::node[predicate]
    //div[text()='hello']

_**axis (ось)**_ - определяет отношение между текущим нодом и тем, 
который мы хотим выбрать (примеры осей: child, descendant, following, following-sibling): `//`<br/>
_**node (нода)**_ - идентифицирует ноду после указания оси:`div`<br/>
_**predicate (условие)**_ - условие, по которому дальше фильтруются найденные оси: `[@name=‘Jo’]`,
`[text()='hello']`. Существуют так же _**функции**_, которые могут использоваться в качестве предиката: 
`text()=‘hello’`, `contains(text(), 'hello')`




  



