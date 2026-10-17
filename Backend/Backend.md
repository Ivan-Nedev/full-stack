защо се ползва post, а не put?

За **Approve** конкретно използваме POST, а не PUT, защото не казваме просто „замени/обнови данните на този PendingProduct“. Казваме на backend-а:

> **Изпълни бизнес действието „Approve“ върху този PendingProduct.**

При теб:

`POST /admin/pending-products/5/approve`

задейства цяла операция:

`PendingProduct #5 ↓ validateReadyForApproval() ↓ провери за duplicate barcode ↓ създай НОВ Product ↓ PendingProduct.status = APPROVED ↓ reviewedAt = now`

Това е **команда/действие**, затова POST е естественият избор.

За сравнение твоят Save е:

`PUT /admin/pending-products/5`

Там смисълът е:

> „Ето новото състояние на данните на PendingProduct #5 — обнови го.“

Например:

```
{
  "name": "Milk",
  "brand": "Vereya",
  "calories": 60
}
```

Затова при SFS можеш да го запомниш така:

`PUT /pending-products/5 → обнови данните на resource #5 POST /pending-products/5/approve → извърши действието Approve върху #5`

И още една важна разлика: PUT по принцип се стремим да е **idempotent** — ако изпратиш една и съща update заявка два пъти, крайният резултат трябва да е същият.

Approve обаче има страничен ефект:

```
productService.createFromPending(pendingProduct);
```

тоест **създава друг resource — Product**. Затова POST пасва още по-добре.

Не е технически невъзможно да направиш approve с PUT; HTTP няма да те спре, ако backend endpoint-ът е @PutMapping. Просто POST /{id}/approve описва по-добре семантиката на API-то ти.





къде се пазят env файловете? Локално на компютъра с който се работи и в production server-а, а отделно за тестове dev profile, което в SpringBoot с правилната антоация ползва конфигурационните файлове в application.properties, които са за конкретната зададена среда?

Да, логиката ти е правилна, но има една важна разлика между **.env файлове**, **environment variables** и **Spring profiles**.

При локална разработка често имаш .env файл на твоя компютър, например:

`frontend/ ├── src/ ├── package.json └── .env`

Пример:

`VITE_API_URL=http://localhost:8080`

Обикновено .env се добавя в:

`.env`

за да не влиза в Git, особено ако съдържа secrets.

В production не е задължително изобщо да имаш физически .env файл. По-често самата среда — Docker, cloud provider, operating system и т.н. — предоставя environment variables:

`DB_URL=... DB_USERNAME=... DB_PASSWORD=...`

и приложението ги чете.

При Spring Boot например можеш да имаш:

`spring.datasource.url=${DB_URL} spring.datasource.username=${DB_USERNAME} spring.datasource.password=${DB_PASSWORD}`

Тук:

`application.properties ↓ ${DB_URL} ↓ взима стойността от environment variable`

Това разделя:

`конфигурационната структура от реалните чувствителни стойности`

Spring profiles са малко друго нещо. Те позволяват различна **конфигурация според средата**.

Например:

`application.properties application-dev.properties application-test.properties application-prod.properties`

Ако активираш:

`spring.profiles.active=dev`

Spring използва общото:

`application.properties`

плюс:

`application-dev.properties`

Ако е:

`prod`

ползва:

`application.properties + application-prod.properties`

При тестове често ще видиш:

```
@ActiveProfiles("test")
```

например:

```
@SpringBootTest
@ActiveProfiles("test")
class ProductServiceTest {
}
```

Тогава Spring може да зареди:

`application.properties + application-test.properties`

Тоест @ActiveProfiles("test") не означава:

> „зареди .env.test“

а:

> „активирай Spring profile test.“

Това е различен механизъм.

Много често архитектурата изглежда така:

`КОНФИГУРАЦИЯ │ application.properties │ ┌────────────────┼────────────────┐ ↓ ↓ ↓ application-dev application-test application-prod .properties .properties .properties │ │ │ ↓ ↓ ↓ DEV env TEST env PROD env`

А истинските secrets могат да идват отвън:

`Environment variables ↓ DB_PASSWORD SUPABASE_URL JWT_SECRET API_KEY ↓ Spring Boot`

Например production конфигурацията може да съдържа:

`spring.datasource.url=${DB_URL}`

но реалният:

`DB_URL=jdbc:postgresql://...`

е зададен само на production server-а.

Така един и същ код може да бъде:

`Git repository ↓ ├── твоя компютър │ DB_URL → development DB │ ├── automated tests │ DB_URL → test DB │ └── production server DB_URL → production DB`

Това е и една от основните причини конфигурацията да се отделя от кода.

И още едно важно уточнение за твоя React frontend: **не трябва да слагаш истински secrets във frontend .env.** При Vite променливи като:

`VITE_SOMETHING=...`

могат да попаднат в JavaScript bundle-а, който браузърът получава. Потребителят може да ги види.

Затова например:

`Frontend: VITE_API_URL ✅ public Supabase URL ✅ public/anon key според предназначението му ✅ Backend: database password ✅ private API keys ✅ private signing secrets ✅`

Най-кратко:

> **.env / environment variables държат стойности, зависещи от машината/средата. Spring profiles определят коя конфигурация да бъде активна за dev/test/prod. application-*.properties описват конфигурацията и могат да четат стойностите от environment variables.**

Трите механизма често работят заедно, но не са едно и също.

тоест env variables са такива записани върху машина, която е средата. Ако машината се ползва за разработка се dev например, ако е в production е за production и тези стойности се ползват в конфигурационните файлове така не се показва на други машини и е локално и затова application properties взимат env variables, които са на машината малки променливи запазени в рамта?

Да, почти точно. Само едно важно уточнение: environment variables не са задължително „записани постоянно в RAM“, а са **част от средата на конкретния процес**.

Може да го мислиш така:

`машина / OS ↓ задава environment variables ↓ стартира Spring Boot процес ↓ Spring Boot процесът получава копие на тези стойности ↓ application.properties ги чете`

Пример на dev машината:

`DB_URL=jdbc:postgresql://localhost:5432/sfs_dev DB_USER=dev_user DB_PASSWORD=...`

а на production:

`DB_URL=jdbc:postgresql://prod-server:5432/sfs DB_USER=prod_user DB_PASSWORD=...`

Кодът ти остава един и същ:

`spring.datasource.url=${DB_URL} spring.datasource.username=${DB_USER} spring.datasource.password=${DB_PASSWORD}`

Така application.properties не съдържа конкретната тайна стойност, а само казва:

> „Вземи стойността от environment variable с това име.“

Това е причината различните машини да могат да ползват различни стойности без да променяш кода.

За RAM: когато процесът вече работи, тези стойности са достъпни в паметта на процеса. Но първоначално могат да идват от различни места:

`OS environment .env файл Docker configuration cloud secret manager CI/CD settings terminal session`

Например:

```
export DB_URL=...
java -jar app.jar
```

Тогава java процесът наследява DB_URL.

И да, ако една машина е dev среда, на нея ще имаш dev стойности. Ако е production машина/контейнер — production стойности.

Така избягваш това:

`spring.datasource.password=myRealProductionPassword`

в Git.

По-точният модел е:

`Git: application.properties → съдържа ${DB_PASSWORD} Dev машина: DB_PASSWORD=dev123 Prod машина: DB_PASSWORD=superSecretProdValue`

Един и същ код, различни среди.

Само внимавай с израза „не се показва на други машини“ — ако env variable е само на една машина, тя естествено не се прехвърля автоматично другаде. Но човек с достатъчно права върху тази машина или процес може да я прочете. Тоест env variables са **начин за конфигурация и отделяне на secrets от кода**, не магическа защита сами по себе си.




така могат да се ползват различни бази данни без да се променя кодът? Dev се разработва и чупи кога, а production при по-стабилна версия?

Да, точно това е една от основните идеи.

Можеш да имаш един и същ код, но различни среди:

`DEV → dev база → тестови данни → може да чупиш, триеш, мигрираш, експериментираш TEST → test база → автоматични тестове → може да се създава и изтрива често PROD → production база → реални потребители и реални данни → промените трябва да са контролирани`

Например кодът е един:

`spring.datasource.url=${DB_URL}`

Но на dev машината:

`DB_URL = sfs_dev`

а на production:

`DB_URL = sfs_prod`

Така не пипаш Java кода.

И да — dev средата е тази, в която:

`правиш нов feature ↓ нещо може да не работи ↓ може да счупиш schema ↓ може да изтриеш тестови данни ↓ може да върнеш назад`

Докато production трябва да е версията, която вече е минала през проверки.

Обичайният flow е:

`пишеш feature ↓ пускаш го локално/dev ↓ тестваш ↓ автоматичните тестове минават ↓ code review / PR ↓ merge ↓ deploy ↓ production`

При по-големи системи често има и междинна среда:

`dev ↓ test ↓ staging ↓ production`

staging е почти копие на production, но без реалните production данни или с контролирани тестови данни.

Там проверяваш:

> „Ще работи ли тази версия в среда, максимално близка до production?“

Това е много полезно, защото локално може всичко да работи, а на production да има друга:

`DB OS environment variables network configuration permissions external services`

Затова правилото е:

> **Dev е мястото, където можеш да чупиш. Production е мястото, което пазиш стабилно.**

И много важно: не трябва случайно локалният ти Spring Boot да сочи към production базата. Това е една от причините профилите и отделните env variables да са толкова важни.

на този етап нямам нужда от такава база, но при пуснато в prod трябва да създам и науча profiles?

Да. На този етап не е нужно да усложняваш SFS с много среди, ако още разработваш локално и нямаш реални потребители.

Когато стигнеш до production, тогава вече е много добра идея да имаш поне:

`dev prod`

Например:

`application.properties application-dev.properties application-prod.properties`

Идеята е да държиш различни настройки за различните среди.

Примерно:

`# application-dev.properties spring.jpa.show-sql=true spring.jpa.hibernate.ddl-auto=update`

а:

`# application-prod.properties spring.jpa.show-sql=false spring.jpa.hibernate.ddl-auto=validate`

И двете могат да ползват:

`spring.datasource.url=${DB_URL} spring.datasource.username=${DB_USER} spring.datasource.password=${DB_PASSWORD}`

като самите DB_URL, DB_USER, DB_PASSWORD са различни на dev и production машината.

Така flow-ът става:

`Локално: SPRING_PROFILES_ACTIVE=dev DB_URL=dev база Production: SPRING_PROFILES_ACTIVE=prod DB_URL=production база`

Не е нужно още сега да учиш profiles в дълбочина. За production ще ти стигне първо да разбереш 4 неща:

- как се активира profile;
- как работят application-dev.properties и application-prod.properties;
- как Spring чете env variables;
- как да не държиш secrets в Git.

После при нужда можеш да добавиш test, staging и по-сложни конфигурации.

За твоя етап бих казал:

> **Сега продължи с функционалността. Когато започнем deployment-а на SFS, тогава profiles вече стават практическа нужда и ще ги направим като част от production setup-а.**




@PutMapping("/{id}")  
public ResponseEntity<Void> updatePendingProduct(  
@PathVariable Long id,  
@RequestBody UpdatePendingProductRequest request) {  
  
pendingProductService.updatePendingProduct(  
id,  
request  
);  
  
return ResponseEntity._noContent_().build();  
}  
защо тук е Put? Мислех, че Put е за една стойност.

Не — тук вероятно смесваш **`PUT` и `PATCH`**. Разликата не е „една стойност срещу много стойности“.

При теб:

```
@PutMapping("/{id}")
public ResponseEntity<Void> updatePendingProduct(
        @PathVariable Long id,
        @RequestBody UpdatePendingProductRequest request) {

    pendingProductService.updatePendingProduct(id, request);

    return ResponseEntity.noContent().build();
}
```

### Какво означава `PUT`

`PUT` по REST принцип означава:

> **„Ето новото състояние на този ресурс.“**

Например имаш:

```
PendingProduct #15

name = "Milk"
brand = "Vereya"
calories = 60
protein = 3.2
```

и изпращаш:

```
PUT /pending-products/15
```

с:

```
{
  "name": "Milk 3%",
  "brand": "Vereya",
  "calories": 61,
  "protein": 3.3
}
```

Идеята е, че актуализираш **ресурса `PendingProduct #15`**, не че променяш една стойност.

### А `PATCH`?

`PATCH` означава:

> **„Промени само тези части от ресурса.“**

Например искаш да смениш **само името**:

```
PATCH /pending-products/15
```

```
{
  "name": "Fresh Milk"
}
```

Останалите полета не ги закачаш.

Така че:

|Метод|Идея|
|---|---|
|`PUT`|Замени/задай цялото състояние на ресурса|
|`PATCH`|Промени част от състоянието|
|`POST`|Обикновено създай нов ресурс|
|`DELETE`|Изтрий ресурс|
|`GET`|Вземи ресурс|

И тук има важна подробност за **твоя SFS**. Ако `UpdatePendingProductRequest` позволява да изпратиш само някои полета и service-ът прави примерно:

```
if (request.name() != null) {
    pendingProduct.setName(request.name());
}

if (request.brand() != null) {
    pendingProduct.setBrand(request.brand());
}
```

тогава endpoint-ът **семантично прилича повече на `PATCH`**, независимо че си го написал с `@PutMapping`.

Ако frontend-ът винаги изпраща цялото текущо състояние на формата, както в твоя `handleSave()`, тогава `PUT` е по-логичен избор.

И още нещо: вероятно си запомнил обратното — **`PATCH` е този, който обикновено свързваме с частична промяна**, не `PUT`.
