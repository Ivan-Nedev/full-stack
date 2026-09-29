JPA, Hibernate, ORM са свързани с класове в java преобразуването им в таблици и sql команди.

Intelij Idea Ultimate позволява да се работи с базата данни вътре в идето като да се виждат таблиците, извършват операции и др.

![[spring_boot_project_structure.png]]
.mvn - вътре стои версията на maven като не е нужно да се тегли глобално,
тоест се тегли от централизирано репо като се ползва mvnw.cmd за windows което съдържа shell команди и гледа в maven-wrapper.properties версията. Това е локално за проекта и винаги когато някой вземе проекта и не знам дали с run или с друго ще се дръпне maven версията така всеки ще работи с една версия. Дори и да има изтеглена глобална версия ще се ползва зададената в maven-wrapper.properties.

resources - всички не java файлове като конфигурационни файлове като application.properties , html, css, js. 
application.properties включва ключ-стойност двойки. Пише се server port, настройки на базата и т.н.



Dependency management
Third-party libraries или frameworks.
![[spring_boot_dependencies_management_examples.png]]
В Spring Boot вместо да се добавят множество индивидуални библиотеки се ползва starter dependency
![[spring_boot_dependencies_management_starter_dependency.png]]
Включват се библиотеки, които са често ползвани заедно.
![[spring_boot_dependencies_management_starter_dependency_2.png]]

Два начина за dependency в pom.
1. В central maven се търсе dependeny и копира каквото е нужно и се слага в pom, натиската sync maven changes за да изтегли каквото е нужно като види, че го няма локално, след това не е проблем.
2. alt+insert -> dependency и се търси там, но при мен не излизат

с тези библиотеки могат да се обработват уеб заявки и създават api-та.


DI - dependency injection
![[dependency_injection_image.png]]
Аналогия:
Ресторант има готвач. Ако котвачът не е на разположение намират друг. 
Ресторант има Jhon за готвач. Ако Jhon го няма става проблем, защото има тясна връзка с него.

Така ако OrderService зависи от конкретната компания за плащане като Stripe, ако в някакъв момент се иска прехвърляне към друг като PayPal трябва да се преправя кода и кой знае какво ще последва и за свързаните класове вътре.
Решение е да се ползват общи правила, така при смяна на начина на плащане да работи изправно - OrderService -> PaymentService -> някоя компания за плащане. 
Ползват се interface-си. Инжектират се в OrderSerbice конкретната фирма спрямо интерфейса. Инжектира се зависимост вместо да е тясно свързана.
Ползват се интефейси за decouple.

Open Closed Principle - **A class should be open for extension and closed for modification.**
В превод когато се създава OrderService и ползва PaymentService за вкарване на обекти, с конструктора, които обекти идват от класове имплементиращи интерфейса PaymentService. Създаването на нови класове имплементиращи интерфейса е добре и да не се модифицира само добавят тези класове.
Това е инструмент не правило.

Следващ проблем:
Ако конкретният PaymentService като Stripe, Paypal и т.н има свои зависимости и трябва да се създадат още обекти и да се свържат заедно както тук
```java
// и в конструктора на PayPalPaymentService още обекти и т.н
orderService.setPaymentService(new PayPalPaymentService());
```
Да се управлява това механично може да стане разхвърляно, трудно за правене и доведе до грешки.

>Spring Core оправя този проблем. Едно от основните функции на Spring Core е **да създава обекти и да ги инжектира в класове автоматично.**
>Има така нареченият IoC (Inversion of Control) Container. Този контейнер управлява обектите. Тези управлявани обекти в Spring терминологията се наричат beans. Bean е java обект управляван от Spring. Spring се грижи за създаването на обекта, инжектиране на неговите зависимости (нужни обекти), дори и неговия жизнен цикъл. 
>Inversion of Control името идва от това, че контролът за създаването на обекти и неговите зависимости става от Spring. Традиционно в програмистите е контролът. Пише се код за създаване и управление на зависимостите.

```java
package com.ivan.store;  
  
import org.springframework.boot.SpringApplication;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
import org.springframework.context.ApplicationContext;  
  
@SpringBootApplication  
public class StoreApplication {  
  
    public static void main(String[] args) {  
        ApplicationContext context = SpringApplication.run(StoreApplication.class, args);  
        OrderService orderService = new OrderService(new PayPalPaymentService());  
        orderService.placeOrder();  
    }  
  
}
```


ApplicationContext е IoC контейнерът. Съхранява обекти.

```java
package com.ivan.store;  
  
import org.springframework.boot.SpringApplication;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
import org.springframework.context.ApplicationContext;  
  
@SpringBootApplication  
public class StoreApplication {  
  
    public static void main(String[] args) {  
	    // run връща ApplicationContext, което е IoC контейнерът за обектите
        ApplicationContext context = SpringApplication.run(StoreApplication.class, args);  
        // getBean зада обект, който да се управлява от Spring. 
        // Все още Spring не знае как да създаде обект от този клас
        // Има два начина за решаване на този проблем - анотации
        OrderService orderService = context.getBean(OrderService.class);   
        orderService.placeOrder();  
    }  
  
}
```


Configuring Beans
Искам Spring да управлява обекти от клас. За да стане това отивам при класа и над него го обозначавам с @Component. Component е общо предназначение анотация. Има по специализирани като
![[spring_boot_spring_core_ioc_annotations.png]]
Технически всички са alias на Component. Няма голяма разлика между тях.

Maven има 3 фази:
- clean
- default
- site
![[spring_boot_maven_phasespng.png]]

mvnw clean - трие директории и файлове
	pre-clean
	clean
	post-clean
![[spring_boot_maven_clean_phase.png]]

mvnw default - изпълнява се последователно, ако искам package първо ще се компилира и тества
	compile
	test
	package
	verify
![[spring_boot_maven_default_phase.png]]

Основи на Spring:
Spring - фреймуърк за Java, който улеснява изграждането на приложения чрез IoC, DI и модули (Web, Data, Security)

Spring Boot - надстройка на Spring, която:
- премахва тежката конфигурация
- дава auto-configuration
- позволява бърз старт на приложение

IoC (Inversion of Control) - контролът за създаване и урпавление на обекти е в Spring, не в програмиста

DI (Dependency Injection) - механизъм при който зависимостите за инектират, вместо да се ползва new

Bean - обект, който се създава, управлява и унищожава от Spring IoC контейнера

ApplicationContext - IoC контейнерът на Spring - държи всички beans и управлява техният цикъл на живот

Анотации:
WEB
@RestController (комбинация от @Controller и @ResponseBody) - връща json
@Controller - връща view-s ползва view resolver автоматично
@RequestMapping - мапва базов URL за всички endpoints в този controller клас
@GetMapping/@PostMapping/@PutMapping/@DeleteMapping - за  http методи за RestAPI
@PathVariable - извлича стойност от URL като част от адреса
взима от ```
```text
URL path (`/users/{id}`)
```
@Value - взима стойност от Spring Environment, което включва:
- `application.yml`
- `application.properties`
- environment variables
- system properties
- профили (`application-dev.yml`)
- и други property sources
ако намери по ключа някъде в тези файлове вкарва стойността
```java
public OpenAiClient(
            @Value("${openai.base-url}") String baseUrl,
            @Value("${openai.api-key}") String apiKey,
            @Value("${openai.model}") String model,
	)
```
@RequestParam - извлича query параметър. Вземи стойност от request-а по ИМЕ и я подай като аргумент като се ползва при GET да вземе от query параметрите по име, а POST като multipart/form-data взима от body параметри
@RequestParam() - в тази заявка се очаква form field с конкретно име 
@RequestBody - мапва JSON към Java обект. вземи JSON body-то  
и го превърни в Java object
ResponseEntity - обект за контрол върху HTTP статус + body + headers
@ResponseStatus(HttpStatus.CREATED) - казва при успешна регистрация (в случея е регистрация когато пиша това) върни http 201 created, не 200 ОК, защото се е създал нов ресурс
@RestControllerAdvice
- Това е “глобален interceptor” за всички `@RestController` класове
- Комбинира идеята на:
	- `@ControllerAdvice` (глобално прихващане)
	- `@ResponseBody` (връща JSON)
Логически - ако някой controller хвърли exception, пробвай да го хванеш тук и върни JSON.

@ExceptionHandler(value=IllegalArgumentException.class) -
```java
@ExceptionHandler(IllegalArgumentException.class)
public ResponseEntity<ApiError> handleBadRequest(...)
```
Това се чете така:
Ако някъде по веригата на един HTTP request се хвърли `IllegalArgumentException`,  
**изпълни този метод** и **върни HTTP отговор**, който съдържа `ApiError`

Тук **НЕ връщаш просто обект**,  
а връщаш **пълен HTTP response**.

```java
ResponseEntity<T>
```
`ResponseEntity<T>` e контайнер за HTTP отговор
Той съдържа `3 неща`:
- HTTP status code (200, 400, 409, 500…)
- HTTP headers (Content-Type, Location и др.)
- Body (payload) - в случея ApiError


IllegalArgumentException означава:
- невалидни входни данни (email формат, празна парола и т.н.)
- затова се мапва към `400 Bad Request



@Component - маркира клас като Spring bean
@Service - специализиран @Component за бизнес логика
@Repository - bean за data layer
@Autowired - инжектира зависимости автоматично (constructor/field/setter)
Constructor Injection - препоръчан начин за DI чрез конструктор

Spring Boot специфично:
@SpringBootApplication - главна анотация, която включва:
- @Configuration
- @EnableAutoConfiguration
- @ComponentScan

Auto Configuration - Spring Boot сам конфигурира beans спрямо зависимостите на проекта
Starter - готов пакет от зависимости (напр. spring-boot-starter-web)


Validation annotations
Това са правила за входните данни
@NotBlank - name не трябва да е празно. `@NotBlank` ensures a string is not null and contains at least one non-whitespace character (trims spaces). **Не може null, "", " ".**
@NotNull - стойността не може да е null
@NotEmpty - collection/list НЕ може да е null, empty []. Ползва се когато трябва да има поне 1 елемент
@Positive - стойност > 0
@Valid - валидирай и вътрешните обекти, освен, дали съществува колекцията
С `@Valid` Spring влиза вътре:

```
IngredientRequest 
↓
productId
quantity
unit
```
каквито валидации има в IngredientRequest за полетата

@Valid в контролера - @Valid @RequestBody CreateRecipeRequest request - trigger-ва validation process-а
![[validation_annotation_flow.png]]

![[validation_annotation_table.png]]

Валидацията на данни става във frontend, Controller и база данни.




![[learn_questions.png]]



Database/JPA:

JPA (Java Persistence API) - работа с база данни чрез ORM (Object Relational Mapper).
Hibernate - най-популярната JPA имплементация
ORM (Object Relational Mapper) - свързване на java обекти с таблици в базата
@Entity - маркира клас като таблица в базата
@Id първичен ключ
@GeneratedValue - автоматично генериране на ID
@Table - определя името на таблицата
@Column - определя колона и ограничения
JpaRepository - интерфейс, който дава CRUD операции без ръчен SQL
Transaction - логическа единица от операции, която се изпълнява изцяло или се rollback-ва
@Transactional - маркира метод/клас като транзакционен, при проблем с операция към базата данни се roll back-ва, тоест се връщат всякакви действия вместо да се губят стъпки и наполовина записани данни
Lazy Loading - свързаните данни се зареждат при нужда
Eager Loading - свързаните данни се зареждат веднага

@JsonIgnore - това поле няма да влезе в json при сериализация на обект
```java
@JsonIgnore
private Long userId;
```

Validation:

Bean Validation - проверка на входни данни с анотации
@NotNull/@NotBlank - проверки за празни стойности
@Size - ограничава дължината
@Email - проверка за валиден email

Maven directory structure
Maven workflow


framework работи върху основите на езика. Слоят добавя решение към често срещани проблеми и улеснява. Код написан върху core библиотеката на програмен език за решаване на проблеми.
![[framework.png]]

Spring Boot е слой над Spring framework. Spring Framework дава гъвкавост за конфигурация, но все пак е трудоемко да се конфигурира всеки нов проект. Spring Boot слоя дава възможност за автоматично конфигуриране и все пак може да се ползва функциите на Spring Framework.
![[spring_boot_framework.png]]

Spring App Layers
Spring Boot е слой, който стъпва на Spring Framework. Самият той има слоеве. Всеки layer има специфична задача и решава специфичен проблем. Ако се мисли за приложение в layers е по-лесно да се разработва и поддържа.

![[spring_boot_layers.png]]

Persistence Layer
Persistence Layer се грижи за комуникацията с базата данни. Мисли се често за Entities. Entities са java обекти, които представят таблици, колони и полета в базата данни. Изполват се за комуникацията с базата данни използвайки различни модели. Repository pattern и Data Access Objects (DAOs). Типовете функционалности, които предлагат са CRUD. Това са основи операции. За по-сложни могат да се създадат отново в двата варианта.
Целта на Persistence Layer-а е да се справи с всички различни комуникации с базата данни, тоест избраната persistence технология и се пишат добри интерфейси и хипотетично с тези интерфейси и да се смени persistence технологията остатъка от прилжонието няма да има нужда да се притеснява за това.
![[spring_boot_persistance_layer.png]]
Мислене за entities:
1. Какви обекти има?  
2. Какво съдържат?  
3. Как са свързани?  
4. Как ги идентифицирам (ID)?

Service Layer
Presentation Layer-а съдържа валидациите на данните, а Service Layer-а работи с валидните данни.
Service Layer целта е да ползва всички функционалности от Persistence Layer и ги ползва за да изпълни изискванията за които приложението е създадено. Типично се прави през интерфейси и класове ги имплементират. Тези класове се наричат Services. Функционалността в service layer може да е сложна колкото е нужно или може просто да праща заявката към Persistence Layer, което все още е важно, защото не трябва Presentation Layer да говори директно на Persistence Layer. Заявките трябва да минават винаги през Service Layer.
![[spring_boot_service_layer.png]]

Presentation Layer
Presentation Layer взима данните като резултат от Service Layer и Persistence Layer и ги показва на потребителя. Показването става с REST API където се ползват контролери или други видове API като graphql API, websockets API или някакъв друг тип. Spring Framework предлага инструменти да се създаде какъвто API е нужен. Дори и да се промени начина за представяне на данните, тоест различен API не е нужно да се правят промени в Service Layer.
![[spring_boot_presentation_layer.png]]


Modularity

Има много голяма екосистема от dependencies, тоест предварително решени проблеми.
Комуникация с базата данни с java обекти. 
- В Spring Framework трябва всички нужни dependencies в pom файла. Създават се beans ръчно, включително Entity Manager отговарящ за Entities, Transaction Manager отговаря за транзакциите и други, и това преди да се напише каквато и да е логика за приложението.
>В Spring Boot се включват dependencies. Слагат се конфигурации в properties файла като как ще бъде връзката с базата данни и създава всички beans автоматично и така може да се стартира писането на реалната логика на приложението.

Spring Boot създава тези beans ползвайки настройки по подразбиране.
![[spring_boot_persistance_layer_2.png]]
Default настройките работят за всеки layer.
Пример за REST API е нужно spring-web dependencies и се пише.

IoC (Inversion of Control)

Зеленият квадрат преставя клас. Класът зависи от други 3 класа - blue, yellow и red квадрати. Един от начините за справяне с тези зависимости е да се създадът в зеления клас с new в конструктора. Това е добре, докато не се наложи да се променят тези зависимости или сменят. Направено така ще трябва да се смени кода в зеления клас. Това не се търси.
![[spring_boot_ioc.png]]

Вместо да се създават класовете в зеления квадрат единствено се декларират интерфейси, които тези класове ползват. Нещо трябва да предостави concrete класовете, които имплементират тези интерфейси, което е spring framework - сивият квадрат. Зависи от framework-а да създаде concrete класовете и да ги инжектира където са нужни.
![[spring_boot_ioc_2.png]]

Смяна на concrete класа. Смяната на алфа класа - синия квадрат, с различен клас имплементиращ същия интерфейс, но имплементацията е малко по-различна. В сивият квадрат, framework-а, ще се махне алфа класа.
![[spring_boot_ioc_3.png]]

Създава се нов клас, който отговаря на интерфейса и framework-а го инжектира където интерфейса е деклариран. Може да е в този зелен клас или колкото са нужни още класове в приложението.

Притежанието на зависимостите е преместено от зеления квадрат към нещо, което оперира на високо ниво, framework, затова се нарича inversion of control. Има различни patterns под inversion of control, конкретно това се нарична DI (Dependency Injection). Spring Framework е dependency injection framework. Така се ползват всички тези нужни dependencies в spring ecosystem. Така се ползват всички предварително решени проблеми.


Beans
Терминът Beans се отнася за concrete класовете.

@Configuration казва на Spring да гледа в класа за bean декларация.

Един начин за деклариране на beans - C:\Users\music\OneDrive\Obsidian\spring_boot\colours

Има и други начини за декларация на beans.
@Component - C:\Users\music\OneDrive\Obsidian\spring_boot\colours - @Component

@Component и неговите приятели, които логически работатя като него. @Service, @Repository, @Controller


Component scanning
Spring ще търси за антотациите когато приложението стартира. Процесът се нарича component scanning. По време на component scan spring гледа проекта за beans и къде са нужни. Намира @Component и гледа дали има нужна от dependencies. Да кажем няма в такъв случай component scan ще създаде инстанция на този клас и ще го сложи като bean в spring ApplicationContext, което е място с beans, които ако има нужда от тях се взимат от там и се слагат където е нужно.
Друг вариант е да има зависимости компонента (bean-а). Както в упражнението RedPrinter, BluePrinter и GreenPrinter.
AutoWired специфичен термин за DI инжектиране на зависимости.

@SpringBootApplication
- @Configuration
- @ComponentScan
- @EnableAutoConfiguration


Когат се влезне в Controller, създаде DTO, има анотация @Valid, проверява се в DTO-то всички антоации за валидация. Ако даде грешка се създава `BindingResult` обект и `parameter`. След като мине през всички валидации създава `MethodArgumentNotValidException` и се подават двете. В BindingResult се съдържат `FieldError` обекти. Това са всички грешки хванати от Spring. Включват методи като `getField` за взимане на името на полето и `getDefaultMessage` за съобщението какво е трябвало да стане.

```
Jackson прави CreateProductRequest object
        ↓
Spring вижда @Valid
        ↓
Validator проверява всички constraints
        ↓
намира:
name → @NotBlank fail
price → @Positive fail
barcode → @NotNull fail
        ↓
BindingResult събира всички тях
        ↓
hasErrors() == true
        ↓
Spring създава един MethodArgumentNotValidException
        ↓
в него има BindingResult с няколко FieldError
```



Всяка заявка е нова нишка, която заявка се следи от Spring Boot. HttpServletRequest отговаря за заявката и могат да се достъпват елементи от заявката.
Exception тръгва нагоре по call stack-a в същата нишка за същата HTTP заявка.
- DispatcherServlet го хваща
- търси `@RestControllerAdvice`
- намира `@ExceptionHandler(Exception.class)`
- изпълнява метода на handler-а
- създава `ResponsekEntity<DTO което е за body>`, което е HTTP response и се преобразува към JSON с помоща на message translator като JACKSON.
- нишката се освобождава и връща в thread pool-а
- използва се отново за следваща заявка


![[spring_boot_exception_handling_flow.png]]


# JWT 
JWT (JSON Web Token)
LOGIN
→ backend създава TOKEN
→ връща го на клиента
→ клиентът го праща при всяка следваща заявка
→ backend го проверява
→ знае КОЙ си

JWT поток (mental model)
POST /auth/login
→ credentials OK
→ generate JWT
→ return JWT

GET /pdf
Authorization: `Bearer <JWT>`
→ verify token
→ extract userId
→ allow / deny

![[jwt_steps.png]]

**Protected endpoints** са API пътища (URL-и) в backend приложение, до които **нямаш достъп, ако не си автентикиран или нямаш нужните права**.



JWT = **3 части**, разделени с точка:
```css
HEADER.PAYLOAD.SIGNATURE
```

```nginx
eyJhbGciOiJIUzI1NiJ9
.eyJzdWIiOiIxIiwiZW1haWwiOiJ0ZXN0QGdtYWlsLmNvbSJ9
.abc123XYZ
```

- **HEADER** → какъв алгоритъм се ползва
- **PAYLOAD** → данните (id, email, roles, exp)
- **SIGNATURE** → защитата

HEADER
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```
Това казва:
- използвай алгоритъм **HMAC SHA-256**
- типът е JWT

`Създаване на подпис`
Взимат се HEADER и PAYLOAD
```text
BASE64URL(header) + "." + BASE64URL(payload)
```
Пример:
```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIn0
```
Прилага се алгоритъм + secret key
При HS256:
```scss
HMAC_SHA256(
    "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIn0",
    SECRET_KEY
)
```
резултатът е `signature`

Signature се добавя като трета част
```css
HEADER.PAYLOAD.SIGNATURE
```
Това е готовият JWT

Проверка на подпис
Когат се прати токена обратно Backend-а го разделя на:
- HEADER
- PAYLOAD
- SIGNATURE (оригиналния)

Backend-а пресмята нов подпис
Със същия алгоритъм и същия secret key:
```
expectedSignature =
HMAC_SHA256(BASE64(header) + "." + BASE64(payload), SECRET_KEY)
```

Сравнение
```java
if (expectedSignature.equals(signatureFromToken)) {
    // token is valid
} else {
    // token is forged or modified
}
```

Работи защото:
- Secret key е известен само на backend-а
- Клиент не може да изчисли валиден подпис
- Ако PAYLOAD-a се промени (id, role, exp) - подписът вече не съвпада
- Дори и 1 бит промяна означава различен подпис

JWT не е криптиран
Всеки може да:
- decode-не header и payload
- да види съдържанието
но 
- не може да го промени
- без да знае secret key

Затова не се слагат чувствителни данни

JWT гарантира, че 
- token-ът е създаден от backend-а
- payload-ът не е променян
- никой не може да се представи за друг

![[HS256_jwt_algorithm.png]]


Generate JWT token
Долу за System.getenv("JWT_SECRET_KEY") се ползва този алгоритъм еднократно отделно някъде за генериране на нужния ключ.
```java
import java.util.Base64;  
  
public class KeyGen{  
    static void main() {   
        byte[] key = new byte[32]; // 256 bits  
        new java.security.SecureRandom().nextBytes(key);  
        System.out.println(Base64.getEncoder().encodeToString(key));
    }  
}
```
След това се вкарва в Environment variables в IDE-то, което е за локално ползване и така се вкарва долу
```java
package com.ivan.study_flash_pdf_ai.auth.security;  
  
import io.jsonwebtoken.Claims;  
import io.jsonwebtoken.Jwts;  
import io.jsonwebtoken.SignatureAlgorithm;  
import io.jsonwebtoken.security.Keys;  
  
import javax.crypto.SecretKey;
import java.util.Base64;  
import java.util.Date;  
  
//utility class  
//helper логика затова методът е static
public class JwtUtil {  
  
    //с ключа се подписва тоукена  
    //с ключа се проверява токена по-късно    
    //Който има този ключ може да    
    //създава валидни тоукени    
    //се представи за user    
    //Затова в production не се hardcode-ва, а идва от application.properties / env variable    
    private static final String SECRET_KEY = System.getenv("JWT_SECRET_KEY");  
    //времето през което тоукентът е валиден след това не може да се ползва и трябва нов login  
    private static final long EXPIRATION_TIME = 1000 * 60 * 60; // 3,600,000 мили секунди = 1 час  
  
  
	// 
    private static Key getSigningKey() {  
        byte[] keyBytes = Base64.getDecoder().decode(SECRET_KEY);  
        return Keys.hmacShaKeyFor(keyBytes);  
    }  
    //Създава JWT за конкретен потребител.  
    public static String generateJwtToken(Long userId, String email){  
        return Jwts.builder()
	            //идентичността на потребителя  
	            //sub е стандартно JWT поле. 
				.subject(String.valueOf(userId)) //sub
				//допълнителна информация в токена                
				//това са просто данни          
                .claim("claim", email) //email
				//iat (issued at) - кога е създаден тоукена    
			    .issuedAt(new Date()) //iat
			    //до кога важи         
                .expiration(new Date(System.currentTimeMillis() + EXPIRATION_TIME)) 
                //exp  
                .signWith(getSigningKey(), SignatureAlgorithm.HS256)  
                .compact();  
    }
```

**Claims** are key-value pairs of information embedded within a security token.
![[jwt_claims.png]]




след като login е успешен, сървърът казва:
	Ето ти доказателство кой си, което можеш да носиш със себе си


JWT Filter
JWT Filter е определянето при нова заявка дали е валиден токенът и да се каже на Spring, че тази заявка е автентикирана.
Стъпките са:
- изважда се token-а т header-а
- проверка дали е валиден като се разделя на HEADER и PAYLOAD, които се декодират и се прави нов подпис като се съберат вкарат в hash алгоритъм заедно с secret key от страната на backend-а. Получава се SIGNATURE и ако двата token-а са еднакви заявката се автентикира.
- разбира се кой user е

Добавя се валидация в JwtUtil класа в който е и статичния метод за генериране на token.
Започва се с:
- да четем token
- да го валидираме
- да вадим userId от него

Ще ми трябва повече повторения на темата.


След валидацията в Токена се създава Authentication обект, но не в ApplicationContext, а в SecurityContext за конкретната заявка
- Authentication не е bean
- Не е singleton
- Не живее в ApplicationContext
- живее в SecurityContextHolder
- и то само за текущият request/нишка
Spring Security ползва ThreadLocal:
- Всяка HTTP заявка е отделна нишка
- SecurityContext е вързан за тази нишка
- след края на заявката контекстът се изчиства

Интересното е че Spring не го инжектира в Authentication интерфейса с DI както при beans, които са в ApplicationContext. Spring автоматично го падава като method parameter като ползва нареченото HandlerMethodArgumentResolver.
В обекта се достъпва каквото е вкарано предварително при валидацията на токена. Например id:
```java
new UsernamePasswordAuthenticationToken(userId, null, ...)
```
Затова:
```java
authentication.getPrincipal() → userId
```
Ползва се id, защото е уникално за всеки user. Като токена е валидиран и се вземе id-то се влиза в endpoint-а, вика се Service Layer-а от където се вика метод от Persistеnce Layer-а Repository, което комуникира с базата и ползва entity от където се създава обект, връщат се данните и се попълват полетата на обекта с празния конструктор на entity-то и рефлексия. Връща се обекта към Service Layer-а от там към Presentation Layer-а и се създава DTO за връщане на конкретни данни.

друг начин казано
тоест след валидация на токена се създава обект, който не е в ApplicationContext, а в SecurityContext за конкретната заявка. Spring взима Authentication от SecurityContext и го подава като параметър. Authentication интерфейса му се достъпва каквото е вкарано в обекта в случея за конкретната обява, нишка, id. Ползва се id, защото е уникално и с него се достъпват данните за конкретната логиа на endpoint. Ползва се през Service Layer-а Persistence Layer-а Repository за комуникация с базата като работи с entity за да се инжектират с reflection данинните от базата в полетата ползвайки protected конструктор. Връща обекта към Service Layer от там към Presentation Layer-а. Създава се DTO и връщат конкретни данни към клиента това всичко след автентикация.
id е източник на истината

filter chain-а е за всяка заявкаограничение на endpoints. Ползва се от JwtAuthenticationFilter.
```java
package com.ivan.study_flash_pdf_ai.auth.config;  
  
import com.ivan.study_flash_pdf_ai.auth.security.JwtAuthenticationFilter;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.security.config.annotation.web.builders.HttpSecurity;  
import org.springframework.security.config.http.SessionCreationPolicy;  
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;  
import org.springframework.security.crypto.password.PasswordEncoder;  
import org.springframework.security.web.SecurityFilterChain;  
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;  
  
@Configuration  
public class SecurityBeans {  
  
    @Bean  
    public PasswordEncoder passwordEncoder(){  
        return new BCryptPasswordEncoder();  
    }  
  
    //ползват се за JwtAuthenticationFilter за аутентикация на токена  
    @Bean  
    public JwtAuthenticationFilter jwtAuthenticationFilter() {  
        return new JwtAuthenticationFilter();  
    }  
  
    // достъп до endpoint-и  
    @Bean  
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {  
  
        http  
                .csrf(csrf -> csrf.disable())  
                .sessionManagement(sm ->  
                        sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS)  
                )  
                .authorizeHttpRequests(auth -> auth  
                        .requestMatchers("/auth/**").permitAll()  
                        .anyRequest().authenticated()  
                )  
                .addFilterBefore(  
                        jwtAuthenticationFilter(),  
                        UsernamePasswordAuthenticationFilter.class  
                );  
  
        return http.build();  
    }  
}
```

```java
  
package com.ivan.study_flash_pdf_ai.auth.security;  
  
import jakarta.servlet.FilterChain;  
import jakarta.servlet.ServletException;  
import jakarta.servlet.http.HttpServletRequest;  
import jakarta.servlet.http.HttpServletResponse;  
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;  
import org.springframework.security.core.context.SecurityContextHolder;  
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;  
import org.springframework.web.filter.OncePerRequestFilter;  
  
import java.io.IOException;  
import java.util.Collections;  
  
//автентификация на токена  
public class JwtAuthenticationFilter extends OncePerRequestFilter {  
  
    @Override  
    protected void doFilterInternal(  
            HttpServletRequest request,  
            HttpServletResponse response,  
            FilterChain filterChain  
    ) throws ServletException, IOException {  
  
        String authHeader = request.getHeader("Authorization");  
  
        // 1. Ако няма Authorization header → продължа  
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {  
            filterChain.doFilter(request, response);  
            return;  
        }  
  
        // 2. Изважда token-а  
        String token = authHeader.substring(7);  
  
        // 3. Валидира token-а  
        if (JwtUtil.isTokenValid(token)) {  
  
            Long userId = JwtUtil.extractUserId(token);  
  
            // 4. Създава Authentication обект  
            UsernamePasswordAuthenticationToken authentication =  
                    new UsernamePasswordAuthenticationToken(  
                            userId,  
                            null,  
                            Collections.emptyList()  
                    );  
  
            authentication.setDetails(  
                    new WebAuthenticationDetailsSource().buildDetails(request)  
            );  
  
            // 5. Казва на Spring: "този request е автентикиран"  
            SecurityContextHolder.getContext().setAuthentication(authentication);  
        }  
  
        // 6. Продължава веригата  
        filterChain.doFilter(request, response);  
    }  
}
```

```java
package com.ivan.study_flash_pdf_ai.auth.security;  
  
import io.jsonwebtoken.Claims;  
import io.jsonwebtoken.Jwts;  
import io.jsonwebtoken.SignatureAlgorithm;  
import io.jsonwebtoken.security.Keys;  
  
import javax.crypto.SecretKey;  
import java.util.Base64;  
import java.util.Date;  
  
//utility class  
//helper логика затова методът е static  
public class JwtUtil {  
  
    //с ключа се подписва тоукена  
    //с ключа се проверява токена по-късно    //Който има този ключ може да    //създава валидни тоукени    //се представи за user    //Затова в production не се hardcode-ва, а идва от application.properties / env variable    private static final String SECRET_KEY = System.getenv("JWT_SECRET_KEY");  
    //времето през което тоукентът е валиден след това не може да се ползва и трябва нов login  
    private static final long EXPIRATION_TIME = 1000 * 60 * 60; // 3,600,000 мили секунди = 1 час  
  
  
    private static SecretKey getSigningKey() {  
        byte[] keyBytes = Base64.getDecoder().decode(SECRET_KEY);  
        return Keys.hmacShaKeyFor(keyBytes);  
    }  
    //Създава JWT за конкретен потребител.  
    public static String generateJwtToken(Long userId, String email){  
        return Jwts.builder()  
                //това са claims, които са key-value pair информация вкарани в токена. Съхраняват се в Payload на токена  
                //идентичността на потребителя                
                //sub е стандартно JWT поле. 
			   .subject(String.valueOf(userId)) //sub  
			   //допълнителна информация в токена                
			   //това са просто данни
				.claim("claim", email) //email  
				//iat (issued at) - кога е създаден тоукена                .issuedAt(new Date()) 
                //iat  
                //до кога важи    
	            .expiration(new Date(System.currentTimeMillis() + EXPIRATION_TIME)) //exp  
                .signWith(getSigningKey(), SignatureAlgorithm.HS256)  
                .compact();  
    }  
  
    //Валидация  
    public static Long extractUserId(String token){  
        Claims claims = extractClaims(token);  
        return Long.parseLong(claims.getSubject());  
    }  
  
    private static Claims extractClaims(String token) {  
        return Jwts.parser()  
                .verifyWith(getSigningKey())  
                .build()  
                .parseSignedClaims(token)  
                .getPayload();  
    }  
  
    public static boolean isTokenValid(String token){  
        try {  
            extractClaims(token);  
            return true;  
        }catch (Exception e){  
            return false;  
        }  
    }  
  
  
}
```

От проекта следва PDF module
Прави се сега, защото има вече:
- JWT auth
- protected endpoints
- знае се кой е user-ът(SecurityContext)

Сега се добавя с какво реално ще работи.

PDF модулът:
- използва auth
- използва базата
- подготвя AI частта

![[study_flash_pdf_ai_pdf_module.png]]

Първият минимален Use case:
POST /pdf/upload
Authorization: Bearer `<JWT>`
multipart/form-data
→ upload PDF
→ запазваме файла
→ връщаме OK

При upload на файл не се праща JSON, а multipart/form-data request.
В Postman се прави така:
- Method: POST
- URL: /pdf/upload
- Body - raw data:
	- key: file
	- type: file
	- value: избира се .pdf файл

HTTP заявката изглежда концептуално така:
```bash
POST /pdf/upload
Content-Type: multipart/form-data; boundary=...

--boundary
Content-Disposition: form-data; name="file"; filename="test.pdf"
Content-Type: application/pdf

(binary data)
--boundary--
```

Spring разбира, че това е файл от:
- Content-Type: multipart/form-data
- има multipart boundary
- има part с name="file"

И автоматично:
- използва MultipartResolver както когато се превежда json формат автоматично от Jackson при приета заявка и връщане на отговор от заявка с response формат json
- парсва multipart request-а
- отделя всеки part

MultipartFile
```java
MultipartFile file
```
MultipartFile е Spring abstraction, която се отнася за един качен файл.

Съдържа:
- името на файла
- content type
- размера
- binary данните

Примери:
```java
file.getOriginalFilename()
file.getContentType()
file.getSize()
file.getBytes()
file.getInputStream()
```
Това е в паметта. Файла не е записан никъде все още.
Spring се занимава със създаването на обект за MultipartFile file. Това става автоматично преди да се влезне в метода.
![[MultipartFile.png]]
@RequestParam() вътре името трябва да съвпада с:
- името във form-data
- key-то в Postman
- name="file" в HTML form

Spring подава обекта автоматично като параметър

![[mime_type.png]]


![[RequestParam_vs_RequestBody.png]]

![[MultipartFile_overview.png]]


Magic Bytes още се наричат file signature или magic numbers са определени последователности от байтове в началото на файл за уникално разбиране какъв е форматът.
![[magic_bytes.png]]

Multipart - part какво е
![[multipart_part.png]]

![[multipart_part_2.png]]

multipart/form-data е като масив от данни. Body-то има множество части всяка част има свои content type и може да съдържа една заявка с този формат - file (pdf, image, audio), json, html структура,
```text
HTTP Body
└── multipart/form-data
    ├── Part 1
    │   ├── Content-Type: text/plain
    │   └── "username=ivan"
    ├── Part 2
    │   ├── Content-Type: application/json
    │   └── { "title": "Doc", "pages": 10 }
    ├── Part 3
    │   ├── Content-Type: application/pdf
    │   └── (binary pdf)
    ├── Part 4
    │   ├── Content-Type: image/png
    │   └── (binary image)
    └── Part 5
        ├── Content-Type: text/html
        └── <html>...</html>
```
всяка част има свои headers
```pqsql
Content-Disposition: form-data; name="file"; filename="doc.pdf"
Content-Type: application/pdf
```
и след него content

Така може да има различни типове в една заявка
Една part може да съдържа:
- Файл (pdf, image, audio, video)  
- JSON  
- HTML  
- String / числа  
- Base64 (ако някой реши) 
- Binary stream

Важен е Content-Type

Spring ги обработва по следния начин:
![[spring_boot_content_type_manage.png]]
За файлове Spring ползва MultipartFile
при множество parts се достъпват с множество анотации като
@RequestParam("file")
@RequestParam String 
и други

![[multipart_part_request_flow.png]]

Следва PdfService + FileStorageService
- къде се пазят файловете
- как се връзва с user

базата пази само мета данни, защото иначе би било тежко, бавно и скъпо да се съхраняват файловете там. На файловата система е по-добре. На файловата система (диска) се пази файла, а мета данните се пазят в базата.

термин -
BLOB - Binary Large Object, data type съхраняващ binary data за complex files като images и видеа

Логика
```scss
Request (JWT + file)
→ Controller (валидация)
→ Service (бизнес логика)
→ FileStorageService (запис на файл)
→ DB (по-късно: metadata)
```

UUID - Universally Unique Identifier
Ползва се за идентичност на ресурс


За големи файлове при upload се съхраняват на диска. Големи и бавни за четене са за база данни.
Реално се ползват:
- AWS S3
- Google Cloud Storage
- Azure Blob
локалният диск е мини версия


InputStream се ползва за получаване на поток от байтове, от който може да се четат данните от файла
При 
```java
Files.copy(file.getInputStream(), filePath);
```
Java чете stream-а на блоково (8 кб)
всеки блок се записва на диска
докато stream-ът свърши

четенето работи така
```java
while ((bytesRead = inputStream.read(buffer)) != -1) {
    outputStream.write(buffer, 0, bytesRead);
}
```
Какво става:
- чете **N байта**
- записва ги
- повтаря
- докато няма повече данни (`-1`)

Това е streaming  
Затова можеш да качваш големи файлове

pdf text extraction - Apache PDFBox
Разпознава страниците във pdf файла, което е много подходящо при извличане на текст от определени страници. Това помага за лимитиране на страниците, извличане на конкретни страници, контрол над AI cost за токените.
Изглежда има доста функции за pdf файлове.

PDDocument клас
- PDDocument съхранява целия pdf като структура като страници, шрифтове, обекти, потоци, метаданни .
- PDDocument е DOM на PDF, не текст  
- текстът още не е извлечен  
- Извличането на текст става с друг клас PDFTextStripper, който обхожда PDDocument, намира текстовите обекти  
- подрежда ги и връща String

обект PDDocument document съдържа:
- страници
- текстови обекти (glyphs)
- координати
- шрифтове
- drawing instructions

**Няма String текст вътре**  
Има **инструкции как да се рисува текстът**.

new File вместо String
File е реален file system обект
библиотеката може:
- да проверява съществуване на файл
- да отваря stream
- да чете метаданни

PDFTextStriper
- PDFTextStriper обекта знае как да обхожда PDF DOM от PDDocument и как да сглобява текст  
- Инструмент за изтъргване на текст от PDF структура

![[pdfteststripper_extracting_text.png]]

pdf документ е базиране на обекти затова инструменти като PDFTextStriper могат да бръкнат в структурата му и извлекат данни. Мета данни за автора на файла настроено в ползвания софтуер, софтуерът в който е написан документа като word, драйвер, който го е превърнал в pdf, дата кога е създаден и последно редактиран.

Важно е да се отбележи: ако PDF-ът е направен чрез сканиране на хартия, той не съдържа текстови обекти, а само един голям обект – **снимка**. В този случай обектите и координатите не помагат, докато не се намеси OCR (Optical Character Recognition) технология, която да „разпознае“ буквите и да ги превърне в обекти.


- AI моделите имат token лимит
- цената расте с дължината
- качеството пада при дълъг хаотичен текст

PDF → чист текст → смислени части → AI

Text cleaning
Проблеми идващи от PDF
PDFBox връща:
- много празни редове
- странни символи
- header/footer повтарящ се
- прекъснати думи

![[user_vs_ai_needs.png]]

![[user_vs_ai_needs_combination.png]]

засега с думи и да се добави по-късно идеята за страници и ако думите са повече от лимита да върне съобщение да избере други страници, защото free version предлага толкова като ограничение или по-една или две страници и на няколко пъти, което пак ще има ограничение колко може всеки да прави

Да се помисли:
- брой заявки за pdf обработка
- обем на всяка обработка (думи/страници) - да видя колко думи е нормално, колко страници типично са толкова думи и определея броя флаш карти за тези страници, думи
- брой флашкарти, които AI генерира

думи към 500 се пада средно за учебник, книга, лекция, notes, научна статия, 

chunking - разделяна на части
chunk е една част от текства, която ще се обработи от ai например 500 думи. Ползва се за token limit, което помага да се плаща по-малко, по-добро разбиране на текства като е малко количесто (качество), ако се счупи един chunk да се пробва пак


как работи в случея pagination логически и как потребителя вижда след като поиска картите, ако се пращат по 20 какво става? Идеята ппц е decks за различните файлове и за един файл да може няколко, ако се иска, но първоначално се казва име на deck и след това се слагат всички карти в него. Предполагам това с дек се случва да изгелжда визуално на frontend-а, а когато се поиска upload pdf да вземе името на pdf-а и сложи като име на дека, което може да се промени и при натискане на дека да се прави silent заявка към backend-а, клиента като браусър да поиска флашкартите за съответното user и име на дека и да ги върне и визуализира както е направено на frontend-а това не зависи от backend-а той е склад за суровини и връща данни в случея json.

rate limiting се слага на endpoints като upload и generate така да се ограничи колко заявки може да се направят. Как се правят логически ограниченията да можеш 5 файла да пратиш и получиш резултат за месец например. Това пак ли е rate limiting да се ограничи колко пъти може да се направи заявка към endpoint и това е когато сайт или приложение каже имаш еди колко си опита да направиш безплатно, ако искаш повече плати subscription и rate limiting-а става повече? Това конкретно за upload и генерирането на pdf flashcards, а на останалите endpoints по същия начин rate limiting, но не така ограничено например при get за flashcards за да се вземат от базата, пратят към frontend, визуализират както е направено и потребителя да ги ползва?

Вече вътрешни ограничение в заявката като определен брой страници и думи са отделно от rate limiting и потенциално да се види нататък, ако стане проблем със заявката какво става дали може пак да се направи и да не се зачита като пропилян опит от ограничените за месец например това го мисля като отделен rate limiting, ако не стане заявката пак да има определено време за чакане преди нова.

pagination прави склада да не изсипва всичко наведнъж.

## Защо да връщаш `id` и `createdDate` в DTO

### `id`

Полезен е за:

- **Edit/Delete** на конкретна карта (`PATCH /flashcards/{id}`, `DELETE /flashcards/{id}`)
    
- **уникален ключ** във frontend (React/Flutter списъци искат стабилен key)
    
- **избягване на дубли** при infinite scroll / “load more”
    
- **deep link** към карта или бъдещи функции (marked, learned, favorite)
    

Ако нямаш `id`, ще трябва да адресираш карта по `question+answer`, което е нестабилно (може да има еднакви).

### `createdDate`

Не е задължителен за MVP, но е полезен за:

- **сортиране** (“най-нови”, “най-стари”)
    
- **показване “генерирано преди 2 часа”**
    
- дебъг/история
    
- future: „последно обновяване“, „прогрес“
    

👉 Практика: `id` почти винаги, `createdDate` по желание. Можеш да го махнеш засега.

Без Pagination GET /decks/5/flashcards връща 10 000 карти, което е бавно, тежко и UI лагва.
Сървърът връща:
- 20 карти
- мета данни колко са общо и има ли следваща страница

Потребителят скролва/цъка "Load more":
- и се зареждат следващите 20 карти и т.н.

User вижда картите, а UI:
- показва първите 20 веднага
- после добавя следващите 20 към списъка без да маха старите
- може да показва някъде, че могат да се заредят още

има списък с decks всеки има flashcards

Business limit (free/premium) е друго:
- “max 2 страници PDF”
- “max 20 обработки/месец”
- “max 200 flashcards/deck”


тоест rate limiting ще е за достъп до endpoint-а ограничение, а вече вътрешно дори и да е свършила квотата (usage limits) се прави проверка при влизането на endpoint-а дали в базата полето, което се актуализира при успешно ползване на endpoint-а, който иска ai или каквото се иска за ограничение. Ако е над него връща грешка и съощение, че е минат free plan лимита и да покаже бутон, което се прави от frontend за premium plan за да има смисъл от повечето ползване на ai расурси, които струват пари. Тоест rate limiting е за endpoints посещаване, а вътрешно за usage limits се ползва counter при всяка успешна заявка към конкретен endpoint, което ограничава user-а

Rate limiting (endpoint-level)


@PathVariable логика
![[@PathVariable.png]]

@PathVariable - когато имената не съвпаднат в пътя на endpoint-а и метода
![[@PathVariable_2.png]]

@RequestParam query parameters от URL-a
Пример:
```bash
GET /decks/12/flashcards?page=0&size=20
```
Тук се гледа:
- page=0
- size=20

```java
@GetMapping("/{deckId}/flashcards")
public Page<FlashcardResponse> getFlashcards(
        @PathVariable Long deckId,
        @RequestParam int page,
        @RequestParam int size
) {
    ...
}
```

Spring чете query string:
```ini
page=0&size=20
```

взима стойностите по име
- page -> 0
- size -> 20

Конвертира ги:
- "0" -> int
- "20" -> int

Подава ги в метода

@PathVariable търси в path-а
@RequestParam търси в
- query params
- form-data
- multipart
@RequestBody търси в body-то
![[@PathVariable_vs_@RequestParam_vs_@RequestBody.png]]


разработчика на софтуер и абстрактно да мислиш
https://www.google.com/search?q=Long+type+в+java&newwindow=1&client=opera-gx&hs=F2Y&sa=X&sca_esv=896419ed210a4547&sxsrf=ANbL-n5r--SMLoFEQNdU_UQ3_55Avyx7mg%3A1770654327222&udm=50&fbs=ADc_l-aN0CWEZBOHjofHoaMMDiKpaEWjvZ2Py1XXV8d8KvlI3vWUtYx0DZdicpfE1faGYenqWn-q4MFiFFtvJjTKeAVxBf9XF8ByrMpEedseJb6C24e7QdJQdIE3TPpl5mEwf0EBCRLNeEhQy5amsEYIcoTye3rrZrd3IP3OYha6_rH_GIVOU8GK5eecabclKqwVhxmAmgM5&aep=1&ntc=1&ved=2ahUKEwjItcax6cySAxU5BtsEHaLjG38Q2J8OegQIDRAE&biw=2083&bih=1027&dpr=0.9&mtid=egqKafz_Ff6p9u8P8qegoQc&mstk=AUtExfA8TTgqcaJYLwR3QOFCKZzji3t7diMHNmzub2cVmkzlkiRFnhWr5VV3HDPDSHcA3JSfzcwAk_K_U-9xyI0SsDTr0LWDDQPgKpqFnat2OPYvo02bW_43FRcNhldDRmkJsJOuba71gxKyDLyIRDgdrf8ZB4mLBdQcSehJ6FfkSaJHQqd19oCE9zcVnngnZtGxLTtu5P4hw1iu9ki_K6EI7tbCQ1jMAwdpYwu3O-FKeIKVuypwOp37UO3ACpJpt9fJS5BB-E5MZbg99v8WjlVejqisk4KZd2ceThhOfUBKY-OcpjFaPZzDijwWVfIWLVoWsqzAAgA3J1qbvw&csuir=1


Page`<Flashcard>` е страница с резултати + информация за страниците
Съдържа:
- списък от Flashcard-и за текущата страница
- номер на страница
- общ брой страници
- общ брой записи
- има ли next page
- има ли previous page

Пример:
```text
Page 0 size 10
Общо записи: 438
Общо страници: 44
```

Реално се получава:
```java
page.getContent()   // List<Flashcard>
page.getTotalPages()
page.getTotalElements()
page.hasNext()
```
Page`<T>` е контейнер над List`<T>` + мета информация

Методът приема, че ще работи с някакъв тип T, който ще бъде определен при извикване.
```java
static <T>Page<T> empty()
```

Компилаторът го извежда автоматично:
```java
Page<Flashcard> page = Page.empty();
```

компилаторът прави това
```java
T = Flashcard
```
Това се нарича type inference

Page`<T>` е интерфейс, който комбинира:
1) **Данни** - List
```java
List<T> getContent();
```

2) **Метаданни** (paging info)
```java
int getNumber();          // текуща страница
int getSize();            // размер на страница
int getTotalPages();      // общо страници
long getTotalElements();  // общо записи
boolean hasNext();
boolean hasPrevious();
```

![[pagination.png]]

Начин на мислене за Page`<FlashcardResponse>` след мапването от Page`<Flashcard>`
```text
pageFlashcards =
{
  content: [
    FlashcardResponse,
    FlashcardResponse,
    ...
  ],
  pageNumber: 0,
  pageSize: 20,
  totalElements: 137,
  totalPages: 7,
  hasNext: true
}
```


Pageable обекта се създава автоматично от Spring, на база на query параметрите в URL-а.
Pageable е описание на заявка за страници, не данни
Pageable съдържа:
- page - коя страница
- size - колко елемента
- sort - по кое поле/посока
Логически поток на Spring като го създава
1. Клиентът прави HTTP заявка
```bash
GET /flashcards/my?page=0&size=20&sort=createdDate,desc
```
2. Spring получава HTTP request
Spring MVC:
- вижда @GetMapping
- вижда параметър в метода:
```java
public Page<FlashcardResponse> getMyFlashcards(Pageable pageable)
```
Вижда, че има Pageable и се опитва да го създаде
3. Търси argument resolver
Вътрешно Spring има
```nginx
HandlerMethodArgumentResolver
```
Един от тях е:
```nginx
PageableHandlerMethodArgumentResolver
```
- чете query параметрите
- създава PageRequest
4. Spring парсва query параметрите
url параметри -> влиза в Pageable
page -> page number
size -> page size
sort -> sort object

Ако няма параметри, Spring ползва default-и:
- page = 0
- size = 20
- sort = unsorted
5. Създава се обект
Вътрешно става подобно
```java
Pageable pageable =
    PageRequest.of(
        page,
        size,
        Sort.by(...)
    );
```
и го подава в метода.
Pageable е интерфейс, а реалната имплементация е PageRequest
Както Authentication, който не се създава ръчно.

query parameters са добавка към URL-а и не променят маршрута. Path-ът има значение. query параметрите не участват в избора на метода дали ще се стигне до него или не. Те са добавка.
Има две части:
```text
/path/part        ?query=params
```

Пример:
```text
/flashcards/my    ?page=0&size=20
```

/flashcards/my е маршрут (routing)
?page=0&size=20 са параметри (data)

Spring:
- избира метода само по path
- query параметрите се обработват след това

всички тези заявки отиват към един и същ метод, но query параметрите са различни
```text
GET /flashcards/my
GET /flashcards/my?page=0
GET /flashcards/my?page=1&size=20
GET /flashcards/my?page=2&size=50&sort=createdDate,desc
```



```java
@GetMapping("/my")  
@ResponseStatus(HttpStatus.OK)  
public Page<FlashcardResponse> my(Pageable pageable){  
  
    Long userId = CurrentUser.getId();  
    Page<FlashcardResponse> flashcards = flashcardService.getFlashcards(userId, pageable);  
  
    return flashcards;
     
}
```



Cache
Ако в сайт за книги 1000 души отворят една и съща книга не се пита backend-а, а cache-ът връща копие което е запазил. Не се стига до backend.

Redis cache е вътре в backend-а.
![[redis_cache.png]]

cloudflare или още CDN (Content Delivery Network) е proxy сървър при който, притежателя на уебсайт или приложение дава своя ip адрес и той го маскира и следи заявките към сървара на който се съхранява приложението на създадетеля (някаква обачна среда), ако за ip-то има много заявки не зависимо към кой endpoint след време ще върне timeout. Тоест той служи за маскиране на ip адреса и посредник между потребителя браусъра и сайта на притежателя съхранен на облачна услуга. Идеятае освен маскиране да следи за броя заявки

Cloudflare е мрежа от reverse proxy сървъри (CDN), които стоят ПРЕД твоя backend и поемат целия публичен трафик към сайта. 

```scss
Browser
   ↓
Cloudflare (CDN / Edge)
   ↓
Origin server (твоят backend в cloud / VPS / on-prem)
```
никога не се стига до сървъра на приложението, ако CloudFlare е включен

Маскиране на IP – вярно, но не основната цел
✔ Да — **реалният IP на backend сървъра се скрива**  
✔ Клиентът вижда само IP-та на Cloudflare  
✔ Това защитава от директни атаки

![[cloudflare_purposes.png]]

load reduction
CDN (Content Delivery Network) - proxy сървъри из света доставяйки по-бърз достъп.
A Content Delivery Network (CDN) is a geographically distributed network of proxy servers and data centers that caches and delivers website content (images, videos, HTML, JS) to users from the nearest location. By reducing physical distance, CDNs drastically improve website loading speeds, enhance security, decrease bandwidth costs, and prevent server crashes during high traffic.
![[cloudflare_purposes_2.png]]

```java
Browser
  ↓
Cloudflare
  ↓
Load Balancer
  ↓
Spring Boot (instance 1)
Spring Boot (instance 2)
Spring Boot (instance 3)
      ↓
    Redis
      ↓
   Database
```

VM
Виртуалната машина (Virtual Machine - VM) е софтуерна емулация на физически компютър, която работи като независима система с собствена операционна система (ОС) и приложения, използвайки ресурсите на хост компютър (процесор, RAM, дисково пространство). Тя позволява на един физически хардуер да стартира множество операционни системи едновременно (напр. Windows, Linux) в изолирана среда.



```java
.orElse(default value)
```
  - ако Optional има стойност връща я, ако няма подава default value
дори и Optional да има стойност default value пак се създава. Винаги са референтни типове, тоест обекти, което идва от Optional. При проста default стойност се ползва, но не при създаване на нов обект, правене на заявка или някаква тежка операция.
Винаги при подаване на параметри към метод се изпълняват преди извикването му
```java
public void print(String value) {
    System.out.println(value);
}

print(createValue());
```
първо се изпълнява createValue() след това се вика print()
Този проблем се решава с orElseGet(), защото се подавафункция, а не обект
```java
public T orElseGet(Supplier<? extends T> supplier)
```
Ако няма стойност — извикай supplier.get()

- Ползва се само когато defaultValue е:
	- евтин за създаване
	- примитив
	- константа

```java
.orElseGet(() -> new UserUsage(  
        userId,  
        today  
));
```
- ако Optional има стойност връща я, ако няма изпълнява ламбда функцията и връща резултата. Lambda се изпълнява само, ако Optional е празен.
- Ползва се когато се иска да се създаде нещо само, ако липсва като да ти трябва запис на обекта в базата, но ако го няма там се създава нов такъв обект като начален запис.

```java
T value = optional.orElseThrow(() -> new IllegalStateException("Not found"));

//Пример
PdfDocument pdf = pdfRepository
        .findByUserIdAndId(userId, pdfId)
        .orElseThrow(() -> new IllegalStateException("PDF not found"));

```
- Ако има стойност връща я, ако няма хвърля exception
- Ползва се когато стойността е задължителна


Spring Boot е framework (рамка). Комбинация е от много библиотеки, събрани на едно място. 

Spring Boot включва:
- Spring Core
- Spring Web
- Spring MVC
- Spring Security
- Spring Data JPA
- Jackson
- Tomcat embedded
- Logging
- Validation API
- Hibernate
- AOP
- Actuator
- десетки други модули

Както казах Spring Boot е голям пакет отделни библиотеки, които работят заедно.
*Но не ги добавям едба по една*. Spring Boot ги управлява за мен.


Maven е мениджър на проекта. Вграден е в Intelij  pom.xml (project object model) е фунтаментална единица от работата на Maven. Това е XML файл, който съдържа информация за проекта и конфигурационния детайли.
Maven e _build automation tool_, отговаря за:
- теглене на библиотеки (dependencies)
- компилиране на код
- тестове
- правене на JAR/WAR файл
- управление на ресурси
- уеднаквяване на структурата на Java проекти

Maven има собствен CLI (команден интерфейс):
```go
mvn clean package
mvn clean install
mvn dependency:tree
```

Като мозък на Java проекта.

**Как IntelliJ използва Maven?**
IntelliJ автоматично:
✔разпознава `pom.xml`  
✔ създава Maven проект  
✔ дърпа dependency-тата в `.m2/`  
✔ показва Maven прозорец (отдясно)  
✔ пуска Maven build команди

Пример:
Когат се натисне **Reload Maven Project**, IntelliJ изпълнява:
```nginx
mvn dependency:resolve
```
Когато се натисне **Build Project**, изпълнява:
```go
mvn package
```

Полезно:
**Къде се намира Maven в IntelliJ?**
Върви в:  
**View → Tool Windows → Maven**
Там виждаш:
- Lifecycle (clean, compile, test, package, install)
- Plugins
- Dependencies

![[maven_vs_intlij_manage_dependencies.png]]

![[maven_usage_in_intlij.png]]

![[maven_standard_structure.png]]

![[maven_and_green_button.png]]

Maven е софтуер (build tool), който IntelliJ използва автоматично.  
Не го стартираш ръчно — IntelliJ работи с него зад кулисите.  
Maven тегли зависимости, build-ва проекта, прави JAR-а.  
IntelliJ само ти позволява да кликнеш бутони вместо да пишеш команди.



Как Maven тегли Spring Boot библиотеките?:
В pom.xml има това:
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```
dependency е библитека, която проекта ползва.
dependencies - списък библиотеки, които проектър ползва.
Maven я тегли автоматично от Maven Central или други репота.

Термини:
- groupId - името на организацията, към което принадлежи билиботеката. Примери са:
	- org.springframework.boot
	- com.fasterxml.jackson.core
	- org.projectlombok
	- mysql
	-ползва обратен домейн за уникалност

- artifactId - името на конкретната библиотека. Това е името на реалната JAR библиотека, която искам да ползвам. Т Примери:
	- spring-boot-starter-web
	- jackson-databind
	- lombok
	- mysql-connector-j
	Това е конкретният "артефакт", който Maven ще изтегли, тоест артефакт е библиотека.

- version - версия на библиотеката. В Spring Boot може да не се пише, защото spring-boot-starter-parent я управлява. Тогава не се пише version, иначе трябва.

- scope - определя кога библиотеката се ползва
	- compile - This is the default scope, used if none is specified. Compile dependencies are available in all classpaths of a project. Furthermore, those dependencies are propagated to dependent projects.
	- runtime  - This scope indicates that the dependency is not required for compilation, but is for execution. Maven includes a dependency with this scope in the runtime and test classpaths, but not the compile classpath.
	Повече за scope-
	https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html

**groupId + artifactId + version** → директория в Maven Central.

target folder-
The target directory is **used to house all output of the build**. 
Maven папката за build-натия код. Това е официалната папка в която Maven слага всичко build-нато:
- компилирани .class файлове
- JAR/WAR файл на проекта
- ресурси
- тестови резултати
target съществува във всеки Maven проект. Това е единственото място, където Maven пише.

Разликата с out/ папката за build (IDE компилация). out е IDE папка създадена от Intelij, когато се компилира проекта без Maven (зеленият бутон). Съдържа компилирани класове. Maven не я ползва.

![[out_vs_target_folders.png]]

Maven дърпа библитеките от Maven Central или друго репо като ползва groupId, artifact, version да намери JAR архива. Ползва ги, вижда съответния url, тегли JAR-а и го **пази локално в ~/.m2/repository/**, тоест първият път изисква интернет. Следващите пъти проверява локално, дали го има, ако да ползва него, ако не тегли от репото и го пази локално.

![[maven_dependencies_processing.png]]

Build = пълна подготовка на приложението за стартиране
**Maven build включва:**
1. компилиране → `.class` файлове
2. копиране на resources → `target/classes`
3. пускане на тестове
4. създаване на JAR/WAR
5. слагане в правилните папки
6. манифест файл
7. dependency-те се добавят вътре в JAR-а (в Spring Boot)

След като Maven build-не проекта, вътре в ```target/``` има обикновен java проект ```target/my-app-1.0.jar```, тоест JAR файл. Стартира се така (пример) ```java -jar target/my-app-1.0.jar``` има и вграден начин от Spring Boot.

![[maven_vs_old_way.png]]

Това е **официалният build output**. За професионален Build винаги се ползва Maven.

Вътре в ```target/``` има:
```lua
target/
 ├── classes/              <-- компилираните .class файлове
 │    └── ...              
 ├── generated-sources/    <-- генериран код (например от Lombok)
 ├── test-classes/         <-- тестови .class файлове
 ├── surefire-reports/     <-- отчети от тестове
 ├── <project-name>.jar    <-- JAR файлът
 ├── maven-status/         <-- статус на build-а
 └── maven-archiver/       <-- инфо за packaging
```

Винаги се съдържа JAR файл след mvn package или mvn install.
За Spring Boot изглежда така:
```
target/demo-0.0.1-SNAPSHOT.jar
```
Това е fat JAR. Съдържа:
- кода
- всички dependencies
- конфигурации
- вграден сървър (примерно Tomcat)

Може да се стартира от всяка машина с:
```
java -jar demo-0.0.1-SNAPSHOT.jar
```

JAR файла съдържа цялото приложение, готов за стартиране навсеки компютър. За Spring Boot това е FAT JAR (нарича се още uber JAR).
Отново повторка:
FAT JAR файла съдържа:
-  компилираните класове
- всички dependencies (Spring Web, JPA, Lombok, MySQL driver…)
- application.properties / application.yml
- статични файлове (`static/`, `templates/`)
- манифест файл (с main class)
-  вграден сървър (Tomcat / Jetty / Netty)

**Тоест JAR файлът е напълно самостоятелно работещо приложение.**

Може да се страртира на всяка машина:
```
java -jar myapp-0.0.1.jar
```
Може да се качи и в облак(AWS, Azure), в Docker, на сървър -> ще тръгне навсякъде еднакво.

Важно е да се каже, че JAR-ът съдържа всичко нужно да работи приложението.
За deployment трябва само JAR файлът. ``` /target``` не се качва никъде.

За cloud deployment
![[maven_target_jar_file_cloud.png]]

JAR ът е самодостатъчен.
![[maven_jar_file_self_sufficient.png]]


В target освен JAR-а има ли разни неща за сегашния компютър?
- Да — това са временни build файлове, нужни само на Maven, но НЕ на приложението. Те НЕ се качват в облак.

*В практиката след build се рънва само JAR файла.
Нищо друго от target НЕ се ползва.*
Точка.
Никакви временни файлове, никакви неща за сегашния компютър.
Единствено се изпълнява JAR файлът
```
java -jar target/името-на-проекта.jar
```

Какво става с останалите файлове в target?
Примерно:
```lua
target/
 ├── classes/              <-- компилираните .class файлове
 │    └── ...              
 ├── generated-sources/    <-- генериран код (например от Lombok)
 ├── test-classes/         <-- тестови .class файлове
 ├── surefire-reports/     <-- отчети от тестове
 ├── <project-name>.jar    <-- JAR файлът
 ├── maven-status/         <-- статус на build-а
 └── maven-archiver/       <-- инфо за packaging
```

Всички без JAR-а:
- НЕ се изпълняват
- НЕ се използват от JVM
- НЕ се деплойват на облак
- НЕ стартират приложението
Те са **само build отпадък**, нужен на Maven, докато строи проекта (build).

![[spring_boot_start_a_project.png]]

![[maven_target_waste_files.png]]




Повече за pom.xml.
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
  
    <groupId>com.example</groupId>  
    <artifactId>student-enrollment-app</artifactId>  
    <version>0.0.1-SNAPSHOT</version>  
    <name>student-enrollment-app</name>  
    <description>Student-Course-Enrollment System</description>  
  
    <parent>        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-parent</artifactId>  
        <version>3.3.0</version>  
        <relativePath/>    </parent>  
    <properties>        <java.version>25</java.version>  
    </properties>  
    <dependencies>        <!-- Web -->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-web</artifactId>  
        </dependency>  
        <!-- JPA -->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-data-jpa</artifactId>  
        </dependency>  
        <!-- Security (ще го допълня по-късно) -->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-security</artifactId>  
        </dependency>  
        <!-- H2 Database за лесен старт -->  
        <dependency>  
            <groupId>com.h2database</groupId>  
            <artifactId>h2</artifactId>  
            <scope>runtime</scope>  
        </dependency>  
        <!-- Validation -->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-validation</artifactId>  
        </dependency>  
        <!-- Lombok (по избор) -->  
<!--        <dependency>-->  
<!--            <groupId>org.projectlombok</groupId>-->  
<!--            <artifactId>lombok</artifactId>-->  
<!--            <optional>true</optional>-->  
<!--        </dependency>-->  
  
        <!-- Test -->        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-test</artifactId>  
            <scope>test</scope>  
        </dependency>    </dependencies>  
    <build>        <plugins>            <plugin>                <groupId>org.springframework.boot</groupId>  
                <artifactId>spring-boot-maven-plugin</artifactId>  
            </plugin>        </plugins>    </build></project>
```

Разяснение:
- ```<project>``` - коренът на Maven. Kазва това е Maven проект.
``` xml
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">  
</project>
```
- ```<modelVersion>``` - версията на POM схемата. Винаги е 4.0.0 - стандарт за Maven.
``` xml
<modelVersion>4.0.0</modelVersion>  
```
- ```<modelVersion>``` - проектна идентификация.
``` xml
<groupId>com.example</groupId>
<artifactId>student-enrollment-app</artifactId>
<version>0.0.1-SNAPSHOT</version>
```
	- groupId - уникален идентификатор, кой е авторът, организацията, компанията, domain.
	- artifcatId - името на приложението/jar файла
		Ще стане ```student-enrollment-app-0.0.1-SNAPSHOT.jar```
	- version - версия на проекта. -Snapshot, защото е development версия (не е финална).

- name и description
``` xml 
<name>student-enrollment-app</name>
<description>Student-Course-Enrollment System</description>
```
	Чисто информативно

- ```<parent>``` - Spring Boot BOM / Dependency Management
``` xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>3.3.0</version>
	<relativePath/>
</parent>
```
Това е критично важно.
Какво прави?
✔ управлява зависимостите (versions)  
✔ задава версии на 100+ Spring Boot библиотеки  
✔ задава default конфигурации (plugins, encoding, java version…)  
✔ казва на Maven коя версия на Spring Boot се ползва

- java version - казва на Maven да компилира с Java 25
```xml
<properties>
    <java.version>25</java.version>
</properties>
```

- dependency блок - от какво зависи приложението, библиотеки.
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
	Включва:
	- Tomcat
	- Spring MVC
	- Jackson
	- HTTP handling
	- REST API

- JPA (Java Persistence API) - работа с база данни
```xml
<dependency>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```
	Включва:
	- Hibernate
	- JPA
	- Transaction Manager
	- EntityManager

- Security
```xml
<dependency>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
	Добавя:
	- Authentication / Authorization
	- Security filters
	- BCrypt
	- Default login

- H2 Database (in-memory runtime DB)
```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```
	`<scope>runtime</scope>` → използва се само при стартиране, не при                    компилация.

- Validation
```xml
<artifactId>spring-boot-starter-validation</artifactId>
```
	Добавя:
	- Jakarta Validation API
	- @NotNull @Email @Size и т.н.

- Test
```xml
<dependency>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```
	Включва:
	- JUnit 5
	- Mockito
	- Spring Test
	- JSONAssert

- Build plugins
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
Това е нещото, което създава FAT JAR-файла по-време на Build както и други неща.
Какво прави spring-boot-maven-plugin?:
	- създава FAT JAR
	- вгражда Tomcat сървър вътре
	- добавя manifest файл
	- добавя клас loader-а
	- позволява mvn spring-boot:run

Ключова част е защото без този plugin:
	- няма да имам стартиращ JAR
	- няма да имам вграден сървър


Важно е да се разбере Maven каква е ролята му.
![[maven_important_concept.png]]


Как работи Maven
![[maven_important_concept2.png]]

![[maven_important_concept3.png]]

![[maven_important_concept4.png]]

![[maven_important_concept4.1.png]]

pom.xml е рецепта за какво трябва да включи Maven в JAR файла.
![[maven_important_concept5.png]]



Да погледна:
- resources папката за какво се ползва и да направя структурата MVC.
Отговор: Всички файлове, които не са Java, но са нужни за приложението като templates паката за dynamic страници.
- дали връзката да е към MySql или да останае h2, разлики и кое е по-добре
Отговор: Ще ползвам MySql, защото ми е най-позната
- application.properties за какво са
- Какво е Hibernate и изглежда се ползва за h2 базата, тоест ORM като JPA?
- как ще изглежда визуално и е добра практика да е структурата като искам да е HTML, CSS, JavaScript и какви са вариантите, кой е най-лесен? 
Отговор: Thymeleaf се ползва за template engine
- дали за MVC да ползвам archetype за готова структура или не?
Отговор: вече я оформям сам



resources папката е за всички файлове, които не са Java код, но са нужни за приложението.
Включва:
- статични файлове (ако има web UI)
	- static/ - CSS,JS и изображения
	- templates - HTML
- конфигурации на приложението
	- application.properties
	- application.yml
- и други

MVC структура в Spring Boot има точно определена структура по конвенция
(convention over configuration)

```java
src/
 └── main/
      ├── java/
      │    └── com.example.studentenrollmentapp/
      │          ├── controller/
      │          │       StudentController.java
      │          │       CourseController.java
      │          │
      │          ├── service/
      │          │       StudentService.java
      │          │       CourseService.java
      │          │
      │          ├── repository/
      │          │       StudentRepository.java
      │          │       CourseRepository.java
      │          │
      │          ├── model/
      │          │       Student.java
      │          │       Course.java
      │          │
      │          ├── dto/
      │          │       StudentDTO.java
      │          │
      │          ├── config/
      │          │       SecurityConfig.java
      │          │
      │          └── StudentEnrollmentAppApplication.java
      │
      └── resources/
            ├── application.properties
            ├── static/  <-- CSS, JS, images (public assets)
            ├── templates/ <-- HTML templates (the real Views) - динамични страници
            ├── db/migration/
            └── messages.properties

```


нещо което не съм разбрал още 
![[spring_boot_important_resources_concepts.png]]

тук има много полезна информация-
от 
1. static файлове разбирам такива, които не се променят, но JS не се ли променя? Припомняне static страници vs non static 2.в resources е еквивалент на views?
Въпроса до след него и преди него има полезни неща
https://chatgpt.com/share/69284bdf-e14c-8011-a176-615a61a48e4c


Static файловете никога не се променят. Доставят се директно от сървъра без сървърна обработка над тях. Без обработка от backend-а.
Примери са:
- `.css`
- `.js`
- `.png`, `.jpg`
- `.svg`
- `.ico`
- `.html` (ако не е template)
- `.pdf`
- `.txt`

Сървърът не генерира нищо автоматично. 

Dynamic pages генерират се от backend-а (Java, PHP, Node, .NET и т.н)
- има логика, променливи и данни от база данни

В Spring Boot dynamic pages са в templates/ 

Дори и да има JS логиката се изпълнява в баузъра, Spring Boot просто го подава натам. Не се изпълнява в backend-а.

Важното е:
- static се изпълняват на клиента (browser)
- dynamic се изпълняват на backend-а (Spring)

В Spring Boot MVC templates/ папктата в resources е еквивалент на views (динамични страници).
Динамични страници, които Spring рендира с технология като Thymеleaf за достъп до логиката на Java.

- Дали просто archetype за повече направен вече проект както в .NET
Отговор: Spring Boot е направен силно за backend, .NET има готови шаблони в Spring Boot не като там. Обиктновенно се прави отделно UI в React/Angular/Vue например. Spring Boot не е за UI, а enterprise backend. Казва аз съм backend, UI си го направи ти. .NET има готови front-end + back-end шаблони. Не отваря браусър само пуска Tomcat сървър локално.

- Минавам въпросите сега от chat-а и каквито имам още
Минах засега въпросите.
- проверявам структурата на проекта да пасне на MVC и ако трябва да променям
- намирам какво къде и как ще се свърже в структурата
- последствие нещо нужно
- коя база данни е най-добре
Отговор: Ще ползвам MySQL
- ако е H2 как работи в Spring Boot, как се зарежда, как се създават таблици, попълват и извършват SQL заявки към нея 
Отговор: -
- H2 как да отворя да пиша директно sql заявки както в MySQL workbench
Отговор: -
- за първи администратор трябва ръчно да се направи акаунт?
- оформя логиката на приложението какво ще прави и да е логично
- изгради ER за базата данни спрямо логиката на приложението и правилата в заданието и добавя ограничения
- да помисля как ще е потребителският интерфейс и как да направя ограниченията да са като в базата данни


- Минавам въпросите сега от chat-а и каквито имам още
За role-based динамични страници трябват DYNAMIC VIEWS + TEMLATE ENGINE (Thymeleaf)
Не могат да се ползват static страници, защото статичните HTML нямат достъп до Java логиката.
Нужен е template engine - Thymeleaf.
✔ В .NET → Razor
✔ В Python → Jinja
✔ В PHP → Blade (Laravel)
✔ В Java → **Thymeleaf**

В templates/  HTML страниците **не са обикновени HTML**, а **динамични** чрез Thymeleaf.

Thymeleaf
Thymeleaf е **Java Template Engine** — позволява ти да пишеш HTML + вградени Thymeleaf/Spring изрази.

```html
<h1>Hello, [[${user.name}]]!</h1>
```
или
```html
<ul>
  <li th:each="course : ${courses}">
      <span th:text="${course.name}"></span>
  </li>
</ul>
```
НЕ пишеш Java код вътре.  
Пишеш Thymeleaf expressions → които са свързани с Java обекти.

Пример:
```java
model.addAttribute("student", studentService.getStudent(id));
```
И в Thymeleaf
```html
<p th:text="${student.fullName}"></p>
```
Това е _динамично вграждане на Java обекти в HTML_.

CSS, JS и изображенията **НЕ са в templates/**.
Те са в:
```cs
src/main/resources/static/
    css/
    js/
    images/
```
И се достъпват така
```html
<link rel="stylesheet" href="/css/style.css">
<script src="/js/app.js"></script>
<img src="/images/logo.png">
```

Тоест:
- **templates/** → HTML ги „използва“
- **static/** → къде се намират



Миграциите са скриптове за автоматично създаване/ актуализиране на базата данни.
```mysql
V1__create_tables.sql
V2__add_column_email.sql
V3__add_course_table.sql
```
Spring Boot (с нещо наречено Flyway) ще ги изпълни автоматично при стартиране.

Нужни са:
- за да не се пише ръчно CREAT TABLE
- за да може прилжението да си създаде базата данни
- за да може да се update-тват таблици при променти
- за production(което е важно)

Миграциите се слагат в:
```
resources/db/migration/
```
Spring ги изпълнява един по един.

DTO (Data Transfer Object) - папка /dto не е entity. DTO се ползват за:
- request-и (формуляри)
- response-и (API)
- безопасно извличане на данни (не се expos-ва entity-то)


за различните роли има различни views
- guest
- user
- admin

Пример:
```java
templates/
 ├── user/
 │     dashboard.html
 ├── admin/
 │     users.html
 │     manage-courses.html
 └── teacher/
       panel.html
```
Thymeleaf ги пълни с данни на потребителя:
``` html
<h1>Welcome, [[${user.fullName}]]</h1>
```

Controller-ът избира шаблона според ролята:
```java
@GetMapping("/dashboard")
public String dashboard(Model model, Authentication auth) {

    if (auth.getAuthorities().contains(new SimpleGrantedAuthority("ROLE_ADMIN")))
        return "admin/dashboard";

    return "user/dashboard";
}
```

1. Controller решава кой HTML да върне
2. Service дава данните
3. Thymeleaf ги вмъква в HTML-а
4. Потребителя получава персонализиран за него HTML
=> динамична страница



@RestController
- Класът е REST API контролер
- Всички методи връщат JSON по default
=> Няма HTML, няма views -> връща само JSON директно към браузъра


@RequestMapping("/api/students")
- Основен url за всички методи в контролера - root адрес
- GET -> /api/students
- POST → /api/students
- PUT → /api/students/{id}
- DELETE → /api/students/{id}

@Controller
- търси HTML в `resources/templates/`  
- използва Thymeleaf































