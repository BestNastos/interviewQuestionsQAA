### 1. Что такое RestAssured?
Это Java библиотека для тестирования RESTful сервисов.

### 2. RestAssured vs Postman
RestAssured - это Java библиотека для автоматизации тестирования, Postman - отдельный 
инструмент. RestAssured является более гибким и многофункциональным, 
а так же позволяет лучше настроить репортинг. 

### 3. Что такое RequestSpecification и как ее использовать?
Класс, помогающий составить запрос. При составлении реквест-спецификации 
можно указать: параметры, URL, тип контента, или даже тело запроса. Для каждого запроса
реквест-спецификаця будет своя, но разные запросы могут иметь сходства, например, 
общий базовый URL.
Тогда общее можно вынести в отдельный метод, который будет составлять универсальную 
спецификацию с общими свойствами и возвращать ее: 

    public static RequestSpecification requestSpecification() {
        return new RequestSpecBuilder()
                .setAccept(ContentType.JSON)
                .setBaseUri(BASE_URI)
                .build()
                .filter(new AllureRestAssured());
    }


А каждый отдельный тест к этой спецификации может добавлять что-то свое: спецификация 
передается в статический метод `RestAssured.given(specification)`, непосредственно при выполнении запроса.
Затем к ней добавляются частности, например, тело запроса:

    @Step("Create Pet")
    public Response create(Pet pet) {
        return RestAssured
                .given(requestSpecification())
                .log()
                .parameters()
                .body(pet.getPayload(), ObjectMapperType.GSON)
                .contentType(ContentType.JSON)
                .put(ADD_OR_UPDATE_PET)
                .prettyPeek();
    }

Если ничего не передавать в метод `RestAssured.given()`, то он вернет “пустую” спецификацию,
и у этой спецификации мы будем вызывать методы, чтобы ее "заполнить" нужной информацией (такой как 
URL, при необходимости - тело запроса, параметры и т.п.):

    RestAssured.given()
                .queryParams("status", "sold")
                .get("https://petstore.swagger.io/v2/pet/findByStatus")
                .prettyPeek()

### 4. Назовите основные классы в RestAssured, которые вы использовали.

**_1. RequestSpecification (and RequestSpecBuilder)_**

Позволяют при помощи следующих методов построить запрос и организовать логирование:

_Логирование_:

    log().all()
    log.ifError()
    log().ifValidationFails()

_Для формирования запроса:_

    queryParams()
    pathparams()
    contentType()
    body()

_Для отправки запроса:_

    post()
    put()
    get()
    delete()

_**2. Response**_

Имеет следующие опции, унаследованные от ResponseOptions:

    getBody(); (.as(class), .asString())
    getHeaders();
    getResponseType();
    getStatusCode();
    и т.д.

    а так же:
    prettyPeek()
    then() 

Метод `then()` возвращает объект ValidatableResponse, на котором можно делать валидацию:

       .then()
       .assertThat() // эта строка - синтаксический сахар
       .statusCode(200)
       .body("category.id", equalTo(5));
    

_**3. ResponseSpecification (And ResponseSpecBuilder)**_

На объекте `ValidatableResponse` также можно вызвать метод `.spec() `
и подать туда объект ResponseSpecification для валидации, с такими общими 
“требованиями” к ответу:

    expectStatusCode()
    expectHeader()
    expectResponseTime()
    expectContentType()
    expectBody()


_**4. JsonPath**_

Класс JsonPath используется для парсинга тела ответа в формате Json:  

    JsonPath jsPath = response.jsonPath();
    jsPath.get(“script”)
    // casts script to int/List etc:
    jsPath.getInt(“script”)
    jsPath.getList(“script”).size()

Вызов метода `response.path(“script”)` эквивалентен `response.jsonPath().get(“script”)`.
Для парсинга json-файла в качестве параметра можно передать groovy script:

    response.path("find{it->it.name==\"Grumpy Cat\"}.id");
    List<Integer> list = response
        .path("findAll{it->it.category.id > 10 && (it.category.id < 1000)}.category.id");

### 5. Напишите простой тест на GET
   

    @Test
    public void testGet() {

        RestAssured.given()
                .queryParams("status", "sold")
                .get("https://petstore.swagger.io/v2/pet/findByStatus")
                .prettyPeek()

                .then()
                .assertThat()
                .statusCode(200)
                .body("[0].status", equalTo("sold"))
    }


### 6. Напишите простой тест на PUT / POST

    @Test
    public void testPut() { // same for post

        Map<String, Object> mapPayload = new Pet()
                .withCategory(5, "name")
                .withName("name").getPayload();

        RestAssured.given()
                .contentType(ContentType.JSON)
                .body(mapPayload)
                .put("https://petstore.swagger.io/v2/pet")
                .prettyPeek()

                .then()
                .assertThat()
                .statusCode(200)
                .body("category.id", equalTo(5));
    }

### 7. Напишите простой тест на DELETE

    @Test
    public void testDelete() {

        RestAssured.given()
                // use pathParam instead of queryParam when you
                   deal with a placeholder:
                .pathParam("petId", 5)
                .delete("https://petstore.swagger.io/v2/pet/{petId}")
                .prettyPeek()

                .then()
                .assertThat()
                .statusCode(200);
    }

### 8. Как десериализовать объект с помощью RestAssured?

    List<Employee> returnedEmployees = Arrays
                        .asList(response.getBody().as(Employee[].class));

### 9. Какие ассершены есть в RestAssured?

    equalTo()

    startsWith()
    endsWith()
    lessThan()

    contains()
    hasItemInArray()

    not()
    is()

### 10. Какие типы аутентификации вы знаете?

- _Basic Auth_ - при каждома запросе логин и пароль передеются в хэдэре, только по защищенному https.
- _ОAuth 1.0, OAuth 2.0_ - манипулируют токеном, который подается в запросе в качестве параметра.

Пример платформы для управления доступом: _OpenAM_. 
Поддерживает около 20 способов аутентификации.
Пример аутентификации при помощи OpenAM: 
отправка отдельного POST запроса на OpenAM-сервер, 
с логином и паролем в виде хэдеров 
и получение токена в ответе.

