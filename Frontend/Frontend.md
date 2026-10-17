---
handwriting-page-id: b5bf86db-e722-470a-91bd-4d96fdeebc57
---
Добре, имам функция, която може да вземе продуктите. Но **кога** React трябва да я извика и **къде** да запази получения array?

Тов се решава от две React концепции - `useEffect()` и `useState()`

Какво е `useState([])`?
Този компенент трябва да пази една стойност между когато се рендира. `[]` е началната стойност. 

Защо синтаксиса има две имена?
```js
const [pendingProducts, setPendingProducts] = useState([]);
```

`pendingProducts` е текущата стойност
`setPendingProducts` е функция с която се променя state-а

Например `setPendingProducts(products);` означава запази тези продукти като новия state

```js
const [pendingProducts, setPendingProducts] = useState([]);
```

```
setPendingProducts(...)
        ↓
state се променя
        ↓
React разбира за промяната
        ↓
component се render-ира отново
        ↓
UI показва новите данни
```

Не се ползва 
```js
let pendingProducts = [];
```
защото React няма как автоматично да знае когато тази променлива се промени да обнови интерфейса.

Кога се зареждат данните?
```
Admin отваря страницата
        ↓
AdminPendingProductsPage се появява
        ↓
зареди pending продуктите
```
Тоест заявката трябва да се случи като **страничен ефект от появяването на страницата**.

За решение на този проблем се ползва `useEffect`.
Добавя се 
```js
import { useEffect, useState } from "react";
```

Какво преставлява `useEffect`?
Основната форма е 
```js
useEffect(() => {

    // действие

}, []);
```

Може да се мисли така:
```
React render-ира component-а
        ↓
component-ът вече е показан
        ↓
React изпълнява effect-а
```

```
AdminPendingProductsPage render
        ↓
<h1>Pending Products</h1>
        ↓
useEffect
        ↓
loadPendingProducts()
        ↓
pendingProductService.getPendingForReview()
        ↓
GET /admin/pending-products
```

```
useState
    → КЪДЕ пазим данните

useEffect
    → КОГА извършваме страничното действие
```

Какво е `() => {}`?
```js
useEffect(() => {

}, []);
```

```js
() => {
}
```
е **arrow** function.
То е друг JavaScript синтаксис за функция.
Например:
```js
function sayHello() {
    console.log("Hello");
}
```
и
```js
const sayHello = () => {
    console.log("Hello");
};
```
са два начина за създаване на функция.

В `useEffect` подаваме функция на React:
```js
useEffect(
    функция,
    dependencies
);
```
Тоест не казваме:
„Изпълни веднага този код.“
А подаваме на React функция, която React да изпълни като effect.

А какво е `[]` накрая?
```js
useEffect(() => { }, []);
```
Това се нарича **dependency array**.

`[]` означава, че effect-ът няма dependencies, които следим за промяна.
Практически го използваме тук с намерението:
Зареди първоначалните данни, когато страницата се появи, а не при всяко следващо render-иране.

Защото вътре правим:

```
setPendingProducts(products);
```

Това предизвиква нов render.

Ако безконтролно правехме заявката при всеки render, бихме могли да стигнем до:

```
render
 ↓
GET
 ↓
setState
 ↓
render
 ↓
GET
 ↓
setState
 ↓
render
 ...
```

Dependency array-ят контролира кога effect-ът трябва отново да бъде изпълнен.

Първи render
↓
`useEffect` се изпълнява
↓
`setPendingProducts(...)`
↓
Нов render
↓
React проверява dependencies
↓
[] няма какво да се е променило
↓
`useEffect` НЕ се изпълнява пак

след `setPendingProducts(products);`
се случва това:
```
useState се променя
        ↓
React казва:
"Този компонент трябва да се render-ира отново"
        ↓
AdminPendingProductsPage() се изпълнява пак
        ↓
React получава нов JSX от return
        ↓
Сравнява го с предишния UI
        ↓
Променя само необходимото на страницата
```

**React компонентът е функция**:

```
function AdminPendingProductsPage() {
    ...
}
```

Тази функция връща JSX:

```
return (
    <div>
        <h1>Pending Products</h1>
    </div>
);
```

React извиква функцията, взима JSX-а и по него показва UI.

Тоест можеш да го мислиш така:

```
React component = JavaScript функция,
която връща описание на UI
```

При промяна на state React извиква тази функция пак, за да види как трябва да изглежда UI-ът с новите данни.

Важно: компонентът не е просто „обикновена функция“ в смисъл на произволна функция — React я третира специално, защото я използва като компонент и вътре можеш да използваш hooks като `useState` и `useEffect`.

Първоначално влизане в страницата.
Да — при първоначалното показване можеш да го мислиш точно така:

1. React извиква `AdminPendingProductsPage()`.
2. Изпълнява се:

```
const [pendingProducts, setPendingProducts] = useState([]);
```

В този момент `pendingProducts` е `[]`.

3. React стига до `useEffect(...)`, но **не изпълнява веднага функцията вътре**. Само регистрира effect-а: „това трябва да се изпълни след render-а“.
    
4. После стига до:
    

```
return (
    <div>
        <h1>Pending Products</h1>
    </div>
);
```

Това е първият render.

5. **След като този render приключи**, React изпълнява `useEffect`:

```
loadPendingProducts();
```

6. Прави се заявката:

```
await pendingProductService.getPendingForReview();
```

7. Когато backend-ът върне продуктите:

```
setPendingProducts(products);
```

8. `setPendingProducts()` променя state-а → React извиква `AdminPendingProductsPage()` **отново** → прави втори render, вече с реалния array.

Може да го визуализираш така:

```
AdminPendingProductsPage()
        ↓
useState([])          pendingProducts = []
        ↓
useEffect(...)        регистрира се, не се изпълнява още
        ↓
return JSX
        ↓
ПЪРВИ RENDER
        ↓
useEffect се изпълнява
        ↓
GET /pending-products/...
        ↓
backend връща array
        ↓
setPendingProducts(products)
        ↓
state се променя
        ↓
AdminPendingProductsPage()
        ↓
ВТОРИ RENDER
```

И има една много важна подробност: при втория render React пак стига до `useEffect`, но понеже имаш:

```
}, []);
```

празният dependency array казва:

> „Изпълни този effect само след първоначалното монтиране на компонента.“

Затова GET заявката не започва отново след всеки `setPendingProducts`.

Точно затова този модел е толкова често срещан:

```
const [data, setData] = useState([]);

useEffect(() => {
    async function loadData() {
        const result = await getData();
        setData(result);
    }

    loadData();
}, []);
```

Логиката е: **първо render с началните данни → после зареждане → после нов render с истинските данни.**


# Проследи целия flow

При отваряне на:

```
/admin/pending-products
```

става:

```
React Router
      ↓
<AdminPendingProductsPage />
      ↓
първи render
      ↓
pendingProducts = []
      ↓
<h1>Pending Products</h1>
      ↓
useEffect се изпълнява
      ↓
loadPendingProducts()
      ↓
pendingProductService.getPendingForReview()
      ↓
apiClient.get("/admin/pending-products")
      ↓
JWT interceptor
      ↓
HTTP GET
────────────────────────────────────────
                BACKEND
────────────────────────────────────────
      ↓
AdminPendingProductController
      ↓
getPendingForReview()
      ↓
PendingProductService
      ↓
Repository / Database
      ↓
List<AdminPendingProductResponse>
────────────────────────────────────────
                FRONTEND
────────────────────────────────────────
      ↓
Axios response
      ↓
response.data
      ↓
products
      ↓
setPendingProducts(products)
      ↓
React state се променя
      ↓
нов render
      ↓
pendingProducts вече съдържа продуктите
```

Това е основният **data fetching pattern**, който ще срещаш постоянно във frontend приложения.

След тази стъпка данните вече са в React, но **още не ги показваме**.

Следващият естествен проблем вече е:

> Имаме `pendingProducts`, който съдържа например 5 обекта. Как от един array React създава 5 визуални елемента?

Точно тогава има причина да научим `.map()` и JSX `{}` и да направим първата реална част от твоята чернова:

```
Оставащи: 5

#43
#44
#45
...
```

В резултат 
```
AdminPendingProductsPage
        ↓
useEffect
        ↓
loadPendingProducts()
        ↓
pendingProductService
        ↓
apiClient + JWT
        ↓
GET /admin/pending-products
        ↓
BACKEND
        ↓
JSON response
        ↓
setPendingProducts(products)
        ↓
pendingProducts съдържа array-а
```


```js
<p>Оставащи: {pendingProducts.length}</p>
```
Защо `{}`?
С `{}` казваме на JSX. Това не е текст. Направи JavaScript expression-а и сложи резултата.
Следващия проблем е следния. Има array с обекти в JS. Как да превърнем всеки обект във визуален елемент?
Ползва се `.map()`. 
`map()` е JavaScript array функция. Минава през всеки елемент.
```js
const numbers = [1, 2, 3];
```

```js
const doubledNumbers = numbers.map((number) => {
    return number * 2;
});
```

```js
[2, 4, 6]
```

Тоест `map()` минава през всеки елемент:

```
1 → функция → 2
2 → функция → 4
3 → функция → 6
```

и създава **нов array от резултатите**.

Вместо product -> число, product -> JSX
Например:

```js
pendingProducts.map((product) => {
    return (
        <div>
            <p>#{product.id}</p>
        </div>
    );
})
```

Ако имаме:

```js
[
    { id: 43 },
    { id: 44 },
    { id: 45 }
]
```

можеш концептуално да си го представиш като:

```js
{id: 43}
    ↓
<div><p>#43</p></div>

{id: 44}
    ↓
<div><p>#44</p></div>

{id: 45}
    ↓
<div><p>#45</p></div>
```

React получава array от JSX елементи и ги render-ира.

коментарът в JSX се пише:
```js
{/* коментар */}
```

```jsx
{pendingProducts.map((pendingProduct) => {

                return (

                    <div key={pendingProduct.id}>

                    <p>Номер: #{pendingProduct.id}</p>

                    <p>Име: {pendingProduct.name}</p>

                    <p>Статус: {pendingProduct.status}</p>

                </div>

                );

            })}
```

Имаме:
```jsx
<div key={pendingProduct.id}>
```

`key` е специална информация за React.

Представи си:

```
#43
#44
#45
```

После approve-ваме #44:

```
#43
#45
```

React трябва да разбере:
„#44 е изчезнал. #43 и #45 са същите елементи като преди.“

Какво вече се случва автоматично?

Това е хубавата част.

Първоначално:

```
const [pendingProducts, setPendingProducts] = useState([]);
```

следователно:

```
pendingProducts.length
```

е:

```
0
```

а:

```
[].map(...)
```

няма нито един елемент за обработване.

Първият render практически е:

```
Pending Products

Оставащи: 0
```

След това:

```
useEffect
 ↓
GET
 ↓
backend връща 5
 ↓
setPendingProducts(products)
```

React render-ира отново.

Сега:

```
pendingProducts.length
```

е `5`.

И:

```
pendingProducts.map(...)
```

създава 5 JSX елемента.

Получаваме:

```
Pending Products

Оставащи: 5

Номер: #43
Име: ...
Статус: REVIEW

Номер: #44
Име: ...
Статус: REVIEW

...
```

Без ние ръчно да казваме:

> „Прерисувай страницата.“

Това е една от основните идеи на React:

```
STATE
  ↓
определя
  ↓
UI
```

Променяш state-а → React отново изчислява как трябва да изглежда UI.


Не ни трябва `selectedPendingProduct`

Можехме да направим:

```
const [selectedPendingProduct, setSelectedPendingProduct] = useState(null);
```

Но тогава ще пазим една и съща информация на две места:

```
pendingProducts
    ↓
[product A, product B, product C]

selectedPendingProduct
    ↓
product A
```

И после трябва да внимаваме двата state-а винаги да са синхронизирани.

Нямаме нужда от това, защото бизнес правилото вече казва:

> Текущият винаги е първият.

Следователно:

```
const currentPendingProduct = pendingProducts[0];
```

Това е обикновена JavaScript променлива, не state.

Добави я **след `useEffect` и преди `return`**:

```
// Backend-ът връща review queue-а по FIFO ред.
// Затова не пазим отделен selected product state:
// първият елемент винаги е продуктът, който admin трябва да обработи.
const currentPendingProduct = pendingProducts[0];
```
## Но има проблем при първия render

Помниш, че началният state е:

```
[]
```

Следователно при първия render:

```
pendingProducts[0]
```

е:

```
undefined
```

Това е нормално — GET заявката още не е приключила.

Затова не можем веднага да направим:

```
<p>{currentPendingProduct.submittedByEmail}</p>
```

защото първоначално все едно казваме:

```
undefined.submittedByEmail
```

и ще получим грешка.

Това ни води до следващия React/JS проблем:

 **Как показваме част от UI-а само когато вече имаме текущ продукт?**

Това се нарича **conditional rendering**.

## Нека първо покажем само review header-а

Махни временния `.map()`, защото вече решихме, че admin няма да гледа всички submissions наведнъж.

`return` може да стане:

```jsx
return (
    <div>
        <h1>Pending Products</h1>

        {/* Броят показва размера на целия review queue, докато самият
            panel по-долу работи само с първия продукт от FIFO реда. */}
        <p>Оставащи: {pendingProducts.length}</p>

        {/* При първия render GET заявката още може да не е приключила.
            Затова показваме review информацията само когато вече има
            текущ PendingProduct. */}
        {currentPendingProduct && (
            <div>
                <p>Изпратен от: {currentPendingProduct.submittedByEmail}</p>
            </div>
        )}
    </div>
);
```

### Как работи това?

```
{currentPendingProduct && (
    <div>...</div>
)}
```

`&&` тук може да се мисли като:

```
Има ли currentPendingProduct?
        │
       ДА
        ↓
покажи този JSX

       НЕ
        ↓
не показвай този JSX
```

При първия render:

```jsx
pendingProducts = []
        ↓
currentPendingProduct = undefined
        ↓
review блокът не се показва
```

След GET:

```
setPendingProducts(products)
        ↓
React render отново
        ↓
currentPendingProduct = pendingProducts[0]
        ↓
вече има object
        ↓
показва submittedByEmail
```

Да, точно си разбрал flow-а. Само една малка корекция в терминологията: при `undefined && (...)` JavaScript не го приема като **грешка**. `undefined` е _falsy_ стойност, затова `&&` спира и JSX вдясно не се изпълнява.

Тоест:

```
Първи render
↓
pendingProducts = []
↓
currentPendingProduct = undefined
↓
undefined && (...) → не изпълнява дясната част
↓
няма опит за undefined.submittedByEmail
↓
няма грешка
```

После GET приключва:

```
setPendingProducts(products)
↓
state се променя
↓
React извиква component функцията отново
↓
useState вече дава новия array
↓
currentPendingProduct = pendingProducts[0]
↓
вече е object
↓
object && (...) → JSX се изпълнява
↓
React го render-ира
```

И правилно: `useEffect` **не се стартира нормално след този state update**, защото dependency array-ят е празен:

```
}, []);
```

Това означава, че не следим стойност като `pendingProducts`, чиято промяна да задейства effect-а отново. В development React `StrictMode` може умишлено да изпълни effect-а допълнително при mount, но това е development поведение, не причинено от `setPendingProducts`.

Следващият проблем е прост:

> Вече знаем кой е текущият PendingProduct. Admin трябва да вижда неговите front/back снимки, за да може след това да въвежда barcode и продуктовите данни.

Backend вече ни дава:

```
String frontImageUrl,
String backImageUrl
```

Следователно няма нужда от нов state. Това са данни на:

```
currentPendingProduct
```

Промени conditional блока така:

```jsx
{/* При първия render currentPendingProduct още може да е undefined.
    Review съдържанието се показва едва след като GET заявката зареди
    първия продукт от FIFO queue-а. */}
{currentPendingProduct && (
    <div>
        <p>
            Изпратен от: {currentPendingProduct.submittedByEmail}
        </p>

        {/* Снимките принадлежат на текущия PendingProduct и вече са част
            от получения backend response, затова не са нужни отделни state-ове. */}
        <div>
            <img
                src={currentPendingProduct.frontImageUrl}
                alt="Front side of submitted product"
            />

            <img
                src={currentPendingProduct.backImageUrl}
                alt="Back side of submitted product"
            />
        </div>
    </div>
)}
```

### Какво става със `src={...}`?

При нормален HTML можеш да имаш:

```jsx
<img src="http://localhost:8080/uploads/image.jpg">
```

Но URL-ът при нас е динамичен — различен е за всеки PendingProduct.

Затова JSX:

```jsx
src={currentPendingProduct.frontImageUrl}
```

означава:

> Вземи JavaScript стойността на `frontImageUrl` и я използвай като `src` на `<img>`.

Отново `{}` превключва от JSX/HTML към JavaScript expression.

Същият принцип вече видяхме тук:

```jsx
{pendingProducts.length}
```

и тук:

```jsx
{currentPendingProduct.submittedByEmail}
```

Нашето правило засега може да бъде:

> **Не създаваме state просто защото ще използваме някаква стойност в JSX. State ни трябва, когато компонентът трябва да пази променящо се състояние, което не можем просто да изведем от вече съществуващия state.**

В момента имаме две места, които трябва да знаят адреса на backend-а:

```
apiClient
   ↓
HTTP API заявки

снимките
   ↓
HTTP заявки за файлове

И двете използват:
http://localhost:8080
```

Не искаме този адрес да е написан на различни места. Затова го изнасяме в **environment variable**.

## Стъпка 1 — `.env`

В root директорията на React/Vite проекта — там, където е `package.json` — създай:

```
.env
```

Вътре:

```
VITE_API_URL=http://localhost:8080
```

### Какво е environment variable?

Това е конфигурационна стойност, която може да бъде различна според средата.

Например development:

```
VITE_API_URL=http://localhost:8080
```

а някой ден production може да бъде:

```
VITE_API_URL=https://api.smartfoodsystem.com
```

Кодът не се променя. Променя се конфигурацията.

При Vite frontend environment variables, които трябва да са достъпни от JavaScript кода, започват с:

```
VITE_
```

И се четат чрез:

```
import.meta.env.VITE_API_URL
```

Важно: такива frontend environment variables **не са тайни**. Потребителят може да ги види в крайния frontend bundle. Затова API URL е подходящ за `.env`, но парола или secret key не би била.

## Стъпка 3 — правим helper за URL на файловете

Тук вече имаме друг проблем.

Backend връща:

```
/uploads/pending-products/abc.jpg
```

а браузърът трябва да получи:

```
http://localhost:8080/uploads/pending-products/abc.jpg
```

Не искаме `AdminPendingProductsPage` да се занимава с това преобразуване.

Направи например:

```
src/
├── api/
│   └── apiClient.js
├── services/
│   └── pendingProductService.js
├── utils/
│   └── fileUrl.js
└── pages/
```

`fileUrl.js`:

```
export function getFileUrl(filePath) {

    // Backend-ът пази и връща относителен HTTP path като "/uploads/...".
    // Frontend-ът добавя адреса на API сървъра, за да получи URL,
    // който браузърът може реално да зареди.
    return `${import.meta.env.VITE_API_URL}${filePath}`;
}
```

### Защо е функция?

Можехме навсякъде да пишем:

```
`${import.meta.env.VITE_API_URL}${currentPendingProduct.frontImageUrl}`
```

Но тогава страниците трябва да знаят **как се конструира file URL**.

С helper-а:

```
getFileUrl("/uploads/a.jpg")
```

получаваме:

```
http://localhost:8080/uploads/a.jpg
```

Страницата просто казва:

> Дай ми URL за този файл.


## Стъпка 4 — използваме helper-а

В `AdminPendingProductsPage.jsx`:

```
import { getFileUrl } from "../../utils/fileUrl";
```

И при изображенията:

```
<div>
    {/* PendingProduct пази относителния /uploads/... path.
        getFileUrl() го превръща в пълен backend URL,
        който браузърът може да поиска през HTTP. */}
    <img
        src={getFileUrl(currentPendingProduct.frontImageUrl)}
        alt="Front side of submitted product"
    />

    <img
        src={getFileUrl(currentPendingProduct.backImageUrl)}
        alt="Back side of submitted product"
    />
</div>
```

Сега целият flow става:

```
PostgreSQL
    │
    │ "/uploads/pending-products/abc.jpg"
    ▼
Spring DTO
    │
    ▼
GET /admin/pending-products
    │
    ▼
React
currentPendingProduct.frontImageUrl
    │
    │ getFileUrl(...)
    ▼
http://localhost:8080/uploads/pending-products/abc.jpg
    │
    ▼
Browser прави GET
    │
    ▼
Spring FileResourceConfiguration
    │
    │ /uploads/** → uploadDirectory
    ▼
физическият abc.jpg
    │
    ▼
Browser показва снимката
```

 Една важна подробност

След като създадеш/промениш `.env`, **рестартирай Vite dev server-а**, защото environment конфигурацията се зарежда при стартирането му.


Продължаваме само с **barcode input-а**, без PATCH засега. Целта е да разбереш как React управлява поле, в което потребителят пише.

Какъв е проблемът?

Имаме:

```
currentPendingProduct
├── images
├── submittedByEmail
└── ...
```

Това са данни, дошли от backend-а.

Но сега admin трябва да може да напише:

```
Barcode: [ 3800123456789 ]
```

Докато пише:

```
"3"
↓
"38"
↓
"380"
↓
"3800"
...
```

React трябва да пази **текущата стойност на полето**.

Затова тук вече има смисъл от state.

## Добавяме barcode state

В началото на компонента, до `pendingProducts`:

```jsx
const [pendingProducts, setPendingProducts] = useState([]);

// Barcode-ът се въвежда от администратора и се променя независимо
// от данните, които сме получили първоначално от backend-а.
// Затова текущата стойност на input-а има собствен state.
const [barcode, setBarcode] = useState("");
```

Началната стойност е:

```
""
```

защото първоначално input-ът е празен.

Имаме:

```
barcode
    ↓
текущата стойност

setBarcode(...)
    ↓
променя стойността
```

Същият принцип като:

```jsx
pendingProducts
setPendingProducts
```

Само че единият state е array, а другият е string.

# Добавяме input

Под снимките:

```jsx
<div>
    <label htmlFor="barcode">
        Barcode
    </label>

    <input
        id="barcode"
        type="text"
        value={barcode}
        onChange={(event) => setBarcode(event.target.value)}
    />
</div>
```

Тук има няколко нови неща и си струва да ги разгледаме едно по едно.

### `value={barcode}`

Това казва:

> Стойността, която `<input>` показва, идва от React state `barcode`.

В началото:

```
barcode = ""
```

следователно:

```
[              ]
```

След като state стане:

```
barcode = "3800"
```

input-ът показва:

```
[ 3800         ]
```


## Но как state се променя, когато пишем?

Тук:

```
onChange={(event) => setBarcode(event.target.value)}
```

`onChange` казва на React:

> Когато стойността на това поле се промени, изпълни тази функция.

Например натискаш `3`.

React извиква функцията:

```jsx
(event) => setBarcode(event.target.value)
```

`event` съдържа информация за случилото се.

А:

```jsx
event.target
```

е елементът, върху който е станало събитието — нашият:

```jsx
<input>
```

Следователно:

```jsx
event.target.value
```

е това, което в момента пише вътре.

Например:

```
Admin пише 380
       ↓
onChange event
       ↓
event.target
       ↓
<input>
       ↓
event.target.value
       ↓
"380"
       ↓
setBarcode("380")
       ↓
state се променя
       ↓
component функцията се изпълнява отново
       ↓
barcode === "380"
       ↓
value={barcode}
       ↓
input показва "380"
```

Това е много типичен React flow.

# Защо изобщо `value={barcode}`?

Може да си кажеш:

> HTML input и без React може да пази написаното.

Да.

Но тогава браузърът управлява стойността, а React не я управлява директно.

Ние искаме React state да бъде **source of truth**:

```
             barcode state
                  │
                  ▼
             value={barcode}
                  │
                  ▼
                input
                  │
             admin пише
                  │
                  ▼
               onChange
                  │
                  ▼
             setBarcode(...)
                  │
                  └──────────► barcode state
```

Това се нарича **controlled input** — React контролира стойността на полето чрез state.

И след малко това ще ни позволи лесно да изпратим:

```json
{
    barcode: barcode
}
```

към backend-а.

## Защо `type="text"`, а не `number`?

Barcode изглежда като число:

```
3800123456789
```

но концептуално не го използваме за математика.

Няма да правим:

```
barcode + 5
barcode / 2
```

Той е **идентификатор**.

Освен това може да има водещи нули:

```
0012345678905
```

Затова:

```
type="text"
```

и JavaScript `string` са по-подходящи.

Същият принцип важи за телефонни номера, някои кодове, номера на документи и т.н. — това, че съдържат цифри, не ги прави непременно числа.

Сега свързваме barcode полето с вече готовия backend endpoint. Тук ще направим цялата верига, но **още няма да правим Merge/New Product UI**.

Целта на тази стъпка е:

```
Admin пише barcode
        ↓
barcode state
        ↓
натиска "Check barcode"
        ↓
React handler
        ↓
frontend service
        ↓
PATCH /admin/pending-products/{id}/barcode
        ↓
Spring
        ↓
CheckBarcodeResponse
        ↓
React
```

## 1. Първо добавяме операцията в `pendingProductService`

Вече имаме:

```jex
async getPendingForReview() {
    const response = await apiClient.get("/admin/pending-products");
    return response.data;
}
```

Добавяме в същия object:

```jsx
async checkBarcode(pendingProductId, barcode) {

    // Barcode проверката принадлежи към конкретен PendingProduct,
    // затова неговото id участва в URL-а. Самият barcode се изпраща
    // в request body според backend CheckBarcodeRequest contract-а.
    const response = await apiClient.patch(
        `/admin/pending-products/${pendingProductId}/barcode`,
        {
            barcode: barcode
        }
    );

    return response.data;
}
```

Обърни внимание на:

```
apiClient.patch(
    URL,
    BODY
);
```

При нас:

```http
URL
/admin/pending-products/43/barcode

BODY
{
    "barcode": "3800123456789"
}
```

Това съвпада с backend-а:

```http
@PatchMapping("/{id}/barcode")
public CheckBarcodeResponse checkBarcode(
        @PathVariable Long id,
        @Valid @RequestBody CheckBarcodeRequest request)
```

Тоест:

```
${pendingProductId}
        ↓
@PathVariable Long id

{ barcode: barcode }
        ↓
@RequestBody CheckBarcodeRequest
```

Това е API contract-ът в действие.

---

## 2. Малко JavaScript съкращение

Написах нарочно:

```json
{
    barcode: barcode
}
```

за да се вижда логиката.

Когато property името и името на променливата са еднакви, JavaScript позволява:

```json
{
    barcode
}
```

Тези двете са еквивалентни:

```json
{ barcode: barcode }
```

```json
{ barcode }
```

Можеш засега да оставиш първата версия, ако ти е по-ясна.

---

# 3. Сега страницата трябва да извика service-а

Имаме:

```
barcode
```

и:

```
currentPendingProduct
```

Следователно разполагаме с двете неща, необходими за заявката:

```
currentPendingProduct.id
        +
barcode
```

Добави функция **след `currentPendingProduct` и преди `return`**:

```jsx
async function handleCheckBarcode() {

    // Проверката винаги е за продукта, който в момента е първи
    // във FIFO review queue-а.
    const result = await pendingProductService.checkBarcode(
        currentPendingProduct.id,
        barcode
    );

    console.log(result);
}
```

Засега нарочно използваме:

```jsx
console.log(result);
```

защото първо искаме да проверим:

> Frontend → backend комуникацията работи ли?

Преди да започнем да строим UI според резултата.

---

# 4. Защо функцията е `async`?

Защото вътре имаме:

```
await pendingProductService.checkBarcode(...)
```

PATCH заявката не приключва моментално.

Тя трябва да мине:

```
Browser
  ↓
HTTP
  ↓
Spring Security
  ↓
Controller
  ↓
Service
  ↓
Database
  ↓
Response
  ↓
Browser
```

Затова:

```
await
```

означава приблизително:

> Изчакай Promise-а от тази операция да приключи и след това сложи получения резултат в `result`.

---

# 5. Добавяме бутона

До input-а:

```jsx
<div>
    <label htmlFor="barcode">
        Barcode
    </label>

    <input
        id="barcode"
        type="text"
        value={barcode}
        onChange={(event) => setBarcode(event.target.value)}
    />

    {/* Button-ът не изпраща заявката сам.
        Той стартира handler функцията, която управлява barcode check flow-а. */}
    <button
        type="button"
        onClick={handleCheckBarcode}
    >
        Check barcode
    </button>
</div>
```

Тук:

```jsx
onClick={handleCheckBarcode}
```

е важно.

Не пишем:

```jsx
onClick={handleCheckBarcode()}
```

Защото това би означавало:

> Изпълни функцията по време на render.

Ние искаме:

```jsx
onClick={handleCheckBarcode}
```

> Дай функцията на React и я извикай, когато човекът натисне бутона.

Същата идея като callback функциите, които вече разглеждахме.

---

# 6. Какво трябва да видиш

Например въвеждаш:

```
3800123456789
```

натискаш:

```
Check barcode
```

Frontend изпраща:

```http
PATCH /admin/pending-products/43/barcode
```

с:

```json
{
    "barcode": "3800123456789"
}
```

А backend може да върне два основни резултата.

Ако няма такъв Product:

```json
{
    "exists": false,
    "product": null
}
```

Ако вече има:

```json
{
    "exists": true,
    "product": {
        "id": 15,
        "name": "Milk",
        "brand": "...",
        "barcode": "3800123456789",
        "calories": 60,
        "protein": 3.2,
        "carbohydrates": 4.7,
        "fat": 3.5,
        "ingredients": "..."
    }
}
```

Засега резултатът трябва просто да се появи в Console чрез:

```
console.log(result);
```

## Какво ще направим веднага след това

Тогава ще възникне нов React въпрос:

> Получихме `result`, но `result` е само локална променлива във функцията. Как да го запазим така, че UI-ът да реагира на него?

Отговорът отново ще бъде state:

```
barcodeCheckResult
       ↓
exists?
   ↙        ↘
false       true
 ↓           ↓
нова         съществуващ
форма        Product
```

Точно там ще започнем да изграждаме двете разклонения на реалния admin review flow.

Първо тествай PATCH-а и виж какво излиза в Console за barcode, който **не съществува**, и ако имаш такъв — за barcode, който **съществува**.

Супер. Вече имаме работеща верига **input → state → click → PATCH → backend → response**. Сега махаме временния `console.log` като крайно решение и караме React да използва резултата.

## Следващият проблем

В момента:

```
async function handleCheckBarcode() {
    const result = await pendingProductService.checkBarcode(
        currentPendingProduct.id,
        barcode
    );

    console.log(result);
}
```

`result` съществува само докато се изпълнява `handleCheckBarcode()`.

А ние искаме след приключване на функцията UI-ът да знае:

```
exists = false
→ този barcode е нов
→ трябва да попълним данните за нов Product

exists = true
→ вече има Product
→ трябва да покажем съществуващия Product и Merge
```

Затова резултатът вече е **състояние на страницата** → `useState`.

### 1. Добавяме state

До `barcode`:

```
const [barcode, setBarcode] = useState("");

// Резултатът от barcode проверката определя кой review flow трябва
// да покаже страницата: създаване на нов Product или merge със съществуващ.
const [barcodeCheckResult, setBarcodeCheckResult] = useState(null);
```

Защо начално `null`, а не `{}`?

Защото имаме три различни състояния:

```
null
→ barcode още НЕ е проверен

{ exists: false, product: null }
→ проверен е и продукт НЕ съществува

{ exists: true, product: {...} }
→ проверен е и продукт съществува
```

Това разграничение е важно.

---

## 2. Записваме response-а

Промени handler-а:

```
async function handleCheckBarcode() {

    const result = await pendingProductService.checkBarcode(
        currentPendingProduct.id,
        barcode
    );

    // Barcode response-ът влияе върху следващата част от UI-а,
    // затова го пазим в state вместо само да го използваме локално.
    setBarcodeCheckResult(result);
}
```

Какво ще стане?

```
PATCH приключва
      ↓
result = response.data
      ↓
setBarcodeCheckResult(result)
      ↓
state се променя
      ↓
React изпълнява компонента отново
      ↓
barcodeCheckResult вече не е null
      ↓
можем да render-ираме различен UI
```

Това е същият механизъм, който вече видя с:

```
setPendingProducts(products);
```

---

# 3. Първо показваме само резултата

Все още не правим цялата форма.

Под barcode input/button добави:

```
{/* След barcode проверката backend-ът определя дали продължаваме
    към създаване на нов Product или към merge със съществуващ. */}
{barcodeCheckResult && (
    <div>
        {barcodeCheckResult.exists ? (
            <p>Намерен е съществуващ продукт.</p>
        ) : (
            <p>Няма съществуващ продукт с този barcode.</p>
        )}
    </div>
)}
```

Тук срещаме още един много използван JavaScript/React оператор:

```
condition ? valueIfTrue : valueIfFalse
```

Това е **ternary operator**.

При нас:

```
barcodeCheckResult.exists ? (
    <p>Намерен е съществуващ продукт.</p>
) : (
    <p>Няма съществуващ продукт с този barcode.</p>
)
```

е еквивалентно като идея на:

```
if (barcodeCheckResult.exists) {
    // покажи "Намерен..."
} else {
    // покажи "Няма..."
}
```

Само че ternary може директно да произведе JSX.

---

## Получаваме вече истински branching UI

Преди проверката:

```
Barcode: [____________]
         [Check barcode]
```

При `exists: false`:

```
Barcode: [380000000000]
         [Check barcode]

Няма съществуващ продукт с този barcode.
```

При `exists: true`:

```
Barcode: [3800123456789]
         [Check barcode]

Намерен е съществуващ продукт.
```

И забележи нещо важно: **React не отваря нова страница**.

Променя се state:

```
barcodeCheckResult
```

и от него React изчислява как трябва да изглежда страницата в момента.

---

### Има един малък bug, който скоро трябва да решим

Представи си:

1. проверяваш barcode A;
2. получаваме `exists=true`;
3. резултатът се показва;
4. започваш да пишеш barcode B.

Докато още не си натиснал `Check barcode` за B, UI все още съдържа резултата от **A**.

Това вече е въпрос за синхронизация на state-овете.

Когато стигнем до него, ще направим така, че промяната на barcode да обезсилва стария резултат.

Но първо направи тази стъпка и провери дали при двата backend response-а React правилно показва съответното съобщение. След това ще оправим този stale-result проблем и ще започнем **`exists=true` → показване на ExistingProduct данните + Merge**.

защо при попълване на barcode има възможност втори път да се попълни и смени състоянието? Тоест, ако се обърка пи записване да може пак да се прати заявка?

Да, точно това е идеята.

barcode е отделен state, защото администраторът може да го променя колкото пъти трябва:

```
const [barcode, setBarcode] = useState("");
```

При всяко писане:

```
onChange={handleBarcodeChange}
```

се обновява:

```
setBarcode(newBarcode);
```

Ако администраторът е написал грешен barcode, може да го поправи и пак да натисне:

```
<button onClick={handleCheckBarcode}>
    Check barcode
</button>
```

Тогава се праща **нова заявка** с новата текуща стойност:

```
pendingProductService.checkBarcode(
    currentPendingProduct.id,
    barcode
);
```

Тоест flow-ът е:

`пише barcode 111 → Check → заявка за 111 → получава резултат осъзнава, че е сгрешил → променя на 222 → старият barcodeCheckResult става null → Check → нова заявка за 222 → нов резултат`

Това е полезно именно защото Check barcode не е финално действие. То е просто **проверка**.

По-късно финалните действия ще са неща като:

`Approve Merge Reject`

Те вече ще променят реалното състояние на PendingProduct-а в backend-а.

Така че да — възможността barcode да се редактира и проверява многократно е нарочно, за да може admin-ът да коригира грешка преди финалното решение.

Добре. Преди да показваме Existing Product, нека оправим малкия логически проблем, който видяхме. Това е добър пример защо при frontend не мислим само „какво да се покаже“, а и **дали различните state-ове все още описват една и съща ситуация**.

## 1. Проблемът със стария резултат

Да кажем:

```
barcode = "111"
```

натискаш Check и backend връща:

```
barcodeCheckResult = {
    exists: true,
    product: {...}
}
```

После изтриваш barcode и пишеш:

```
barcode = "222"
```

Но все още имаме:

```
barcodeCheckResult = резултатът за "111"
```

Получава се несъответствие:

```
barcode state
"222"

barcodeCheckResult state
резултат за "111"
```

UI може да твърди, че `"222"` съществува, въпреки че изобщо не сме го проверили.

## 2. При промяна на barcode обезсилваме резултата

В момента имаш:

```
onChange={(event) => setBarcode(event.target.value)}
```

Вече трябва да направим две операции, затова е по-четимо да ги изнесем във функция.

Добави:

```
function handleBarcodeChange(event) {

    const newBarcode = event.target.value;

    setBarcode(newBarcode);

    // Предишният check result принадлежи на стария barcode.
    // След промяна на input-а вече не можем да го използваме като
    // валиден резултат за текущата стойност.
    setBarcodeCheckResult(null);
}
```

И input:

```
<input
    id="barcode"
    type="text"
    value={barcode}
    onChange={handleBarcodeChange}
/>
```

Така:

```
проверяваме A
↓
barcodeCheckResult = result за A
↓
пишем B
↓
handleBarcodeChange()
├── barcode = B
└── barcodeCheckResult = null
↓
старият резултат изчезва
```

Това е по-правилно състояние на системата.

---

# 3. Сега `exists=true`

Backend вече ни връща не само:

```
exists: true
```

а:

```
product: {
    id,
    name,
    brand,
    barcode,
    calories,
    protein,
    carbohydrates,
    fat,
    ingredients
}
```

Тоест при:

```
barcodeCheckResult.exists === true
```

вече имаме всички данни, необходими да покажем съществуващия глобален Product.

Промени сегашния блок:

```
{barcodeCheckResult && (
    <div>
        {barcodeCheckResult.exists ? (
            <p>Намерен е съществуващ продукт.</p>
        ) : (
            <p>Няма съществуващ продукт с този barcode.</p>
        )}
    </div>
)}
```

на:

```
{barcodeCheckResult && (
    <div>
        {barcodeCheckResult.exists ? (
            <div>
                <h2>Existing Product</h2>

                {/* При exists=true backend contract-ът гарантира, че product
                    съдържа глобалния Product със същия barcode. Засега само
                    показваме данните; admin не трябва да ги редактира тук. */}
                <p>Name: {barcodeCheckResult.product.name}</p>
                <p>Brand: {barcodeCheckResult.product.brand}</p>
                <p>Barcode: {barcodeCheckResult.product.barcode}</p>
                <p>Calories: {barcodeCheckResult.product.calories}</p>
                <p>Protein: {barcodeCheckResult.product.protein}</p>
                <p>
                    Carbohydrates: {barcodeCheckResult.product.carbohydrates}
                </p>
                <p>Fat: {barcodeCheckResult.product.fat}</p>
                <p>Ingredients: {barcodeCheckResult.product.ingredients}</p>
            </div>
        ) : (
            <p>Няма съществуващ продукт с този barcode.</p>
        )}
    </div>
)}
```

Тук не ни трябва нов state като:

```
const [existingProduct, setExistingProduct] = useState(...)
```

защото отново бихме дублирали информация.

Вече имаме:

```
barcodeCheckResult
        │
        ├── exists
        │
        └── product
              ├── id
              ├── name
              ├── brand
              └── ...
```

Следователно можем директно да използваме:

```
barcodeCheckResult.product
```

---

## 4. Това подготвя Merge

Вече frontend разполага с двете ID-та, необходими за твоя backend endpoint:

```
@PostMapping("/{pendingProductId}/merge/{productId}")
```

Имаме:

```
currentPendingProduct.id
```

например:

```
43
```

и:

```
barcodeCheckResult.product.id
```

например:

```
12
```

Следователно бъдещата заявка ще стане:

```
POST /admin/pending-products/43/merge/12
                              ↑        ↑
                         pending     existing
                         product      product
```

И това е хубав пример защо **не показваме непременно ID-тата на администратора**, но frontend-ът ги пази и ги използва.

Направи засега тези две промени: `handleBarcodeChange()` и показването на Existing Product. Ако работят, следващата стъпка ще бъде сравнително малка: **Merge button → service → POST → успешен merge → преминаване автоматично към следващия PendingProduct от FIFO queue-а.**

работи. Мисля си, че после когато се изпълни един pending product трябва да се махне старото състояние в системата, а не когато се напише нов barcode. Но това предполагам и после може да се мисли.

Да — и тук всъщност има **две различни причини за изчистване на state**, които не трябва да смесваме.

Първата е тази, която вече направихме: ако admin промени barcode-а от 111 на 222, резултатът за 111 вече е невалиден. Затова е правилно още при промяната да правим:

```
setBarcodeCheckResult(null);
```

Това предотвратява показването на **stale резултат**.

Но това, което ти казваш, също е задължително и е по-големият reset: когато приключим целия PendingProduct чрез:

`Approve Reject Merge`

трябва да преминем към следващия продукт с **чист review state**.

Например:

`PendingProduct #10 barcode = "123" barcodeCheckResult = existing Product някакви попълнени полета... ↓ Merge ↓ PendingProduct #10 приключва ↓ PendingProduct #11 става current ↓ barcode = "" barcodeCheckResult = null form data = initial state errors = clear ...`

Иначе ще получим много по-опасен bug:

`PendingProduct #11 + state, останал от #10`

Това не трябва да се случва.

Така че мисленето ти е правилно: **state-ът принадлежи не просто на страницата, а част от него принадлежи на конкретния review, който admin обработва. Когато review-ът приключи, този state трябва да приключи заедно с него.**

Засега оставяме и:

```
function handleBarcodeChange(event) {
    const newBarcode = event.target.value;

    setBarcode(newBarcode);

    // Резултатът вече не отговаря на текущия barcode.
    setBarcodeCheckResult(null);
}
```

А когато направим Merge след малко, ще добавим и reset при преминаването към следващия PendingProduct.

## 8.1. Добавяме `merge()` в service-а

В `pendingProductService.js`, до `getPendingForReview()` и `checkBarcode()`, добави:

```
async merge(pendingProductId, productId) {

    // Merge свързва текущия PendingProduct с вече съществуващия
    // глобален Product. Backend endpoint-ът очаква и двете id-та в URL-а.
    await apiClient.post(
        `/admin/pending-products/${pendingProductId}/merge/${productId}`
    );
}
```

Тук не правим:

```
return response.data;
```

защото backend endpoint-ът ти е:

```
@PostMapping("/{pendingProductId}/merge/{productId}")
public ResponseEntity<Void> merge(...) {

    pendingProductService.merge(pendingProductId, productId);

    return ResponseEntity.noContent().build();
}
```

Тоест при успех frontend се интересува основно от това, че:

```
POST завърши успешно
```

а не от response body.

---

## 8.2. Какво трябва да стане след успешен Merge?

Имаме три неща:

```
Merge успешно
     ↓
1. старият barcode state вече не принадлежи
   на новия PendingProduct

2. старият barcodeCheckResult също не принадлежи
   на новия PendingProduct

3. трябва да поискаме актуалното състояние
   на queue-а от backend
```

Следователно handler-ът е:

```
async function handleMerge() {

    await pendingProductService.merge(
        currentPendingProduct.id,
        barcodeCheckResult.product.id
    );

    // Barcode state-ът принадлежи на току-що приключения review.
    // Изчистваме го, за да не се покаже върху следващия PendingProduct.
    setBarcode("");
    setBarcodeCheckResult(null);

    // Не премахваме продукта локално. Backend/DB е source of truth,
    // затова взимаме отново актуалния FIFO продукт и remaining count.
    await loadPendingProducts();
}
```

Тук двете ID-та идват от различни места:

```
currentPendingProduct.id
        ↓
PendingProduct, който разглеждаме
например #43


barcodeCheckResult.product.id
        ↓
вече съществуващият глобален Product
например #7
```

И заявката става:

```
POST /admin/pending-products/43/merge/7
```

Тоест:

```
PendingProduct #43
       ↓ merge
Product #7
```

---

## 8.3. Защо reset-ът е след `await merge()`?

Това е важно.

Не правим:

```
setBarcode("");
setBarcodeCheckResult(null);

await pendingProductService.merge(...);
```

Защото ако backend върне грешка:

```
Merge
↓
500 / 409 / друга грешка
```

вече сме изтрили локалното състояние на формата.

При нашия вариант:

```
await pendingProductService.merge(...);

// изпълнява се само ако Promise е завършил успешно
setBarcode("");
setBarcodeCheckResult(null);

await loadPendingProducts();
```

Ако `await merge()` хвърли exception, JavaScript няма да продължи към следващите редове.

По-късно ще добавим и нормално error handling.

---

## 8.4. Добавяме Merge бутона

В `exists=true` частта вече показваш `Existing Product`.

Под данните му добави:

```
<button
    type="button"
    onClick={handleMerge}
>
    Merge
</button>
```

Той трябва да е именно вътре в клона, в който:

```
barcodeCheckResult.exists
```

е `true`, защото само тогава имаме:

```
barcodeCheckResult.product.id
```

---

Целият flow вече става:

```
Admin въвежда barcode
        ↓
Check barcode
        ↓
exists = true
        ↓
показваме ExistingProduct
        ↓
[Merge]
        ↓
handleMerge()
        ↓
POST /merge
        ↓
Backend:
PendingProduct → MERGED
        ↓
POST приключва успешно
        ↓
reset barcode state
        ↓
GET /admin/pending-products
        ↓
{
  remainingCount: новият брой,
  pendingProduct: следващият FIFO
}
        ↓
setCurrentPendingProduct(...)
setRemainingCount(...)
        ↓
React rerender
        ↓
Admin автоматично вижда следващия
```

Значи вече имаме завършен **Merge flow от frontend до DB и обратно до следващия review**:

```
Check barcode
→ existing Product
→ Merge
→ DB се променя
→ GET review queue
→ актуален remainingCount
→ следващият FIFO PendingProduct
→ reset на стария barcode state
```

Следващият логичен етап е случаят **`exists = false`**.

Тогава flow-ът ще бъде:

```
Check barcode
        ↓
exists = false
        ↓
показваме editable полетата
        ↓
Admin попълва данните
        ↓
Save
        ↓
PUT /admin/pending-products/{id}
        ↓
данните се пазят в PendingProduct

по-късно:
        ↓
Approve
        ↓
създава се глобален Product
        ↓
GET review state
        ↓
следващият PendingProduct
```

Тук обаче не бих започнал веднага с input-ите. Първо трябва да решим **как да пазим form state-а в React**, защото имаме 7 свързани полета:

```
name
brand
calories
protein
carbohydrates
fat
ingredients
```

и освен това трябва при зареждане на вече Save-нат `PendingProduct` тези стойности да се възстановят във формата.

Следващата стъпка ще е да сравним:

```
const [name, setName] = useState("");
const [brand, setBrand] = useState("");
...
```

с един общ:

```
const [productForm, setProductForm] = useState({
    name: "",
    brand: "",
    calories: "",
    protein: "",
    carbohydrates: "",
    fat: "",
    ingredients: ""
});
```

Добре. Първо само **JavaScript object + промяната му**. Още не правим формата.

## 1. Какво е object

В Java си свикнал обектът да идва от class:

```
Product product = new Product();
product.setName("Milk");
product.setBrand("Vereya");
```

В JavaScript можем направо да създадем object:

```
const product = {
    name: "Milk",
    brand: "Vereya",
    calories: 60
};
```

Имаме:

```
product
│
├── name      → "Milk"
├── brand     → "Vereya"
└── calories  → 60
```

Лявата страна:

```
name
brand
calories
```

се нарича **property** (свойство).

Дясната е стойността на property-то.

За да вземем стойност:

```
product.name
```

→ `"Milk"`

```
product.calories
```

→ `60`

---

## 2. `const` не означава, че object-ът не може да се променя

Това е малко подвеждащо:

```
const product = {
    name: "Milk"
};
```

JavaScript позволява:

```
product.name = "Cheese";
```

и вече:

```
product.name
```

е:

```
"Cheese"
```

`const` означава, че не можем да дадем **друг object на самата променлива**:

```
product = {
    name: "Bread"
};
```

Това ще даде грешка.

Но properties вътре в съществуващия object могат да бъдат променяни.

---

# 3. При React state обаче не искаме това

Да кажем:

```
const [productForm, setProductForm] = useState({
    name: "",
    brand: "",
    calories: ""
});
```

Не трябва да правим:

```
productForm.name = "Milk";
```

Причината е важна.

React работи добре с идеята:

```
стар state
   ↓
създаваме НОВ state
   ↓
setProductForm(новState)
   ↓
React разбира за промяната
   ↓
rerender
```

Затова вместо да променяме стария object, създаваме **нов object**.

---

# 4. Най-лесният начин

Имаме:

```
const productForm = {
    name: "",
    brand: "",
    calories: ""
};
```

И искаме `name = "Milk"`.

Можем да създадем:

```
const newProductForm = {
    name: "Milk",
    brand: productForm.brand,
    calories: productForm.calories
};
```

Получаваме:

```
СТАР OBJECT                  НОВ OBJECT

productForm                  newProductForm

name: ""          ───────→   name: "Milk"
brand: ""         ───────→   brand: ""
calories: ""      ───────→   calories: ""
```

След това React:

```
setProductForm(newProductForm);
```

Това е правилната идея.

Но представи си нашите 7 полета. Всеки път ще трябва:

```
const newProductForm = {
    name: "Milk",
    brand: productForm.brand,
    calories: productForm.calories,
    protein: productForm.protein,
    carbohydrates: productForm.carbohydrates,
    fat: productForm.fat,
    ingredients: productForm.ingredients
};
```

Само за да променим `name`.

Затова JavaScript има удобен синтаксис.

---

# 5. Spread syntax `...`

Можем да напишем:

```
const newProductForm = {
    ...productForm,
    name: "Milk"
};
```

`...productForm` тук означава приблизително:

> Вземи properties на `productForm` и ги постави в този нов object.

Ако:

```
productForm = {
    name: "",
    brand: "Vereya",
    calories: "60"
};
```

тогава:

```
{
    ...productForm
}
```

създава нов object със същите properties:

```
{
    name: "",
    brand: "Vereya",
    calories: "60"
}
```

А ние след това пишем:

```
{
    ...productForm,
    name: "Milk"
}
```

Получава се:

```
{
    name: "",
    brand: "Vereya",
    calories: "60",

    name: "Milk"
}
```

Понеже `name` се задава **след** spread-а, последната стойност печели.

Крайният object е:

```
{
    name: "Milk",
    brand: "Vereya",
    calories: "60"
}
```

Старият `productForm` не е променен.

---

# 6. В React това става

Вместо:

```
productForm.name = "Milk";
```

правим:

```
setProductForm({
    ...productForm,
    name: "Milk"
});
```

Мисли за него така:

```
Вземи стария productForm
        ↓
копирай всичките му properties
        ↓
замени name с новата стойност
        ↓
получи нов object
        ↓
дай новия object на React
```

Например:

```
ПРЕДИ

productForm
{
    name: "",
    brand: "Vereya",
    calories: "60"
}

        ↓

setProductForm({
    ...productForm,
    name: "Milk"
});

        ↓

СЛЕД RERENDER

productForm
{
    name: "Milk",
    brand: "Vereya",
    calories: "60"
}
```

Това е основата, която ни трябва.

## 7. Защо един object е удобен точно за SFS

Нашите:

```
name
brand
calories
protein
carbohydrates
fat
ingredients
```

са части на **една форма**.

Затова вместо 7 отделни state-а:

```
const [name, setName] = useState("");
const [brand, setBrand] = useState("");
// ...
```

можем да имаме един:

```
const [productForm, setProductForm] = useState({
    name: "",
    brand: "",
    calories: "",
    protein: "",
    carbohydrates: "",
    fat: "",
    ingredients: ""
});
```

Така по-късно Save логиката естествено ще може да вземе целия:

```
productForm
```

и от него да построи request-а към твоя:

```
UpdatePendingProductRequest
```

Сега свързваме object state-а с реалните `<input>` полета. Тук има един JavaScript синтаксис, който е важно да разбереш, вместо да го запомняш.

## 1. Започваме с един input

Имаме state:

```
const [productForm, setProductForm] = useState({
    name: "",
    brand: "",
    calories: "",
    protein: "",
    carbohydrates: "",
    fat: "",
    ingredients: ""
});
```

За `name` можем да направим:

```
<input
    type="text"
    value={productForm.name}
    onChange={handleNameChange}
/>
```

и:

```
function handleNameChange(event) {

    setProductForm({
        ...productForm,
        name: event.target.value
    });
}
```

Ако admin напише `Milk`:

```
input
 ↓
onChange
 ↓
event.target.value = "Milk"
 ↓
нов object
 ↓
{
   ...старите стойности,
   name: "Milk"
}
 ↓
setProductForm(...)
 ↓
rerender
```

Това работи.

Но после ще имаме:

```
handleNameChange()
handleBrandChange()
handleCaloriesChange()
handleProteinChange()
...
```

Всички ще правят почти едно и също.

---

# 2. Даваме `name` на input-а

HTML input може да има:

```
<input
    name="name"
    type="text"
    value={productForm.name}
/>
```

Това `name` не е името на продукта.

То е **името на самото поле**.

За Brand:

```
<input
    name="brand"
    type="text"
    value={productForm.brand}
/>
```

За calories:

```
<input
    name="calories"
    type="number"
    value={productForm.calories}
/>
```

Забележи съвпадението:

```
INPUT                     OBJECT

name="name"       ↔       productForm.name
name="brand"      ↔       productForm.brand
name="calories"   ↔       productForm.calories
```

Това ще ни позволи една функция да разбере **кое property трябва да промени**.

---

# 3. `event.target.name`

Когато се промени:

```
<input name="brand" ... />
```

React ни дава `event`.

В него:

```
event.target
```

е input-ът, който е предизвикал събитието.

Затова можем да вземем:

```
event.target.name
```

→

```
"brand"
```

и:

```
event.target.value
```

→ например:

```
"Vereya"
```

Можем да ги запазим:

```
function handleInputChange(event) {

    const fieldName = event.target.name;
    const fieldValue = event.target.value;
}
```

Ако сме писали в Brand:

```
fieldName  = "brand"
fieldValue = "Vereya"
```

Ако сме писали в Name:

```
fieldName  = "name"
fieldValue = "Milk"
```

Една и съща функция вече знае **кое поле се променя и каква е новата му стойност**.

---

# 4. Но как променяме динамично property?

Тук идва нов JS синтаксис.

Ако знаем предварително property-то:

```
{
    ...productForm,
    brand: "Vereya"
}
```

Но сега имаме:

```
const fieldName = "brand";
```

Не можем да напишем:

```
{
    ...productForm,
    fieldName: "Vereya"
}
```

защото това буквално ще създаде property:

```
{
    fieldName: "Vereya"
}
```

А ние искаме JavaScript да използва **стойността вътре във `fieldName`**.

Затова пишем:

```
{
    ...productForm,
    [fieldName]: "Vereya"
}
```

`[fieldName]` означава:

> Изчисли каква е стойността на `fieldName` и използвай нея като име на property.

Ако:

```
fieldName = "brand";
```

това:

```
[fieldName]: "Vereya"
```

на практика става:

```
brand: "Vereya"
```

Това се нарича **computed property name**.

---

# 5. Получаваме един общ handler

Вече можем:

```
function handleInputChange(event) {

    const fieldName = event.target.name;
    const fieldValue = event.target.value;

    setProductForm({
        ...productForm,
        [fieldName]: fieldValue
    });
}
```

Например admin пише:

```
60
```

в:

```
<input
    name="calories"
    type="number"
    value={productForm.calories}
    onChange={handleInputChange}
/>
```

Получаваме:

```
event.target.name
→ "calories"

event.target.value
→ "60"
```

следователно:

```
setProductForm({
    ...productForm,
    ["calories"]: "60"
});
```

което е все едно:

```
setProductForm({
    ...productForm,
    calories: "60"
});
```

---

## 6. И същият handler работи за всички inputs

```
<input
    name="name"
    type="text"
    value={productForm.name}
    onChange={handleInputChange}
/>

<input
    name="brand"
    type="text"
    value={productForm.brand}
    onChange={handleInputChange}
/>

<input
    name="calories"
    type="number"
    value={productForm.calories}
    onChange={handleInputChange}
/>
```

Всички извикват:

```
handleInputChange
```

но всеки казва кой е чрез:

```
name="name"
name="brand"
name="calories"
```

Затова не ни трябват 7 handler функции.

---

## 7. Една малка React подобрена версия

На практика предпочитам да напишем handler-а така:

```
function handleInputChange(event) {

    const fieldName = event.target.name;
    const fieldValue = event.target.value;

    setProductForm(previousForm => ({
        ...previousForm,
        [fieldName]: fieldValue
    }));
}
```

Тук се появява:

```
previousForm => (...)
```

и може би изглежда по-сложно.

Причината е, че новият state **зависи от стария state**:

```
вземи предишната форма
+
промени едно поле
=
нова форма
```

React подава актуалната предишна стойност като `previousForm`.

Можеш да го мислиш приблизително като:

```
React:
"Ето ти най-актуалния productForm."

        ↓

previousForm

        ↓

копирай го
и промени конкретното поле

        ↓

нов productForm
```

Това избягва проблеми, ако няколко state updates се случат близо един до друг.

Не е някаква специална форма на object — това:

```
previousForm => (...)
```

е **arrow function**. Ще я разгледаме отделно, ако не ти е ясна.

---

### Засега не добавяй още inputs

Добави само state-а:

```
const [productForm, setProductForm] = useState({
    name: "",
    brand: "",
    calories: "",
    protein: "",
    carbohydrates: "",
    fat: "",
    ingredients: ""
});
```

и handler-а:

```
function handleInputChange(event) {

    const fieldName = event.target.name;
    const fieldValue = event.target.value;

    setProductForm(previousForm => ({
        ...previousForm,
        [fieldName]: fieldValue
    }));
}
```

Това създава механизма. Следващата стъпка ще е да го свържем с полетата **само когато barcode проверката върне `exists: false`**, и ще видиш целия процес реално в React.

Вече имаме механизма:

```
productForm
+
handleInputChange()
```

Сега започваме да изграждаме реалната форма, но първо само **Name и Brand**, за да видиш целия цикъл, преди да добавим останалите 5 полета.

## 1. Кога трябва да се показва формата?

Имаме три състояния:

```
barcodeCheckResult === null
→ още не сме проверили barcode
→ не показваме форма

exists === true
→ има Product
→ показваме Existing Product + Merge

exists === false
→ няма Product
→ показваме формата за нов Product
```

В момента вероятно имаш нещо подобно:

```
{barcodeCheckResult && (
    <div>
        {barcodeCheckResult.exists ? (
            <div>
                {/* Existing Product */}
            </div>
        ) : (
            <p>Няма съществуващ продукт с този barcode.</p>
        )}
    </div>
)}
```

Ще заменим само:

```
<p>Няма съществуващ продукт с този barcode.</p>
```

с формата.

---

## 2. Добавяме първо Name

```
{barcodeCheckResult && (
    <div>
        {barcodeCheckResult.exists ? (
            <div>
                {/* Тук остава сегашният Existing Product + Merge UI. */}
            </div>
        ) : (
            <div>
                <p>Няма съществуващ продукт с този barcode.</p>

                <label htmlFor="name">
                    Name
                </label>

                <input
                    id="name"
                    name="name"
                    type="text"
                    value={productForm.name}
                    onChange={handleInputChange}
                />
            </div>
        )}
    </div>
)}
```

Тук има няколко различни неща.

### `id="name"`

```
id="name"
```

се свързва с:

```
<label htmlFor="name">
```

Така browser знае:

```
този label
   ↓
принадлежи на
   ↓
този input
```

### `name="name"`

Това е важно за нашия handler:

```
name="name"
```

Когато пишеш `Milk`:

```
event.target.name
```

ще бъде:

```
"name"
```

а:

```
event.target.value
```

ще бъде:

```
"Milk"
```

Следователно handler-ът стига до:

```
[fieldName]: fieldValue
```

което става:

```
["name"]: "Milk"
```

и практически:

```
name: "Milk"
```

---

## 3. `value={productForm.name}`

Това е другата половина на връзката.

```
value={productForm.name}
```

означава:

> Стойността, която input-ът показва, идва от React state.

Целият цикъл е:

```
productForm.name = ""
        ↓
<input value="">
        ↓

Admin пише M

        ↓
onChange
        ↓
event.target.value = "M"
        ↓
handleInputChange()
        ↓
setProductForm(...)
        ↓
productForm.name = "M"
        ↓
React rerender
        ↓
<input value="M">
```

После пишеш `i`:

```
"Mi"
↓
state
↓
rerender
↓
input показва "Mi"
```

И така докато стане `"Milk"`.

Това е **controlled input** — React state контролира стойността на input-а.

---

# 4. Добавяме Brand

Сега виж предимството на общия handler.

Не пишем:

```
handleBrandChange()
```

Добавяме само:

```
<div>
    <label htmlFor="brand">
        Brand
    </label>

    <input
        id="brand"
        name="brand"
        type="text"
        value={productForm.brand}
        onChange={handleInputChange}
    />
</div>
```

Така `exists=false` частта става:

```
<div>
    <p>Няма съществуващ продукт с този barcode.</p>

    <div>
        <label htmlFor="name">
            Name
        </label>

        <input
            id="name"
            name="name"
            type="text"
            value={productForm.name}
            onChange={handleInputChange}
        />
    </div>

    <div>
        <label htmlFor="brand">
            Brand
        </label>

        <input
            id="brand"
            name="brand"
            type="text"
            value={productForm.brand}
            onChange={handleInputChange}
        />
    </div>
</div>
```

Не слагаме коментар върху всеки input, защото кодът вече е очевиден. Полезните коментари трябва да са около неочевидната логика като `handleInputChange()`.

---

## 5. Един handler управлява и двете

Ако пишеш:

```
Name → Milk
```

input-ът има:

```
name="name"
```

следователно:

```
fieldName = "name"
fieldValue = "Milk"

        ↓

productForm

{
    name: "Milk",
    brand: "",
    ...
}
```

После пишеш:

```
Brand → Vereya
```

този input има:

```
name="brand"
```

получаваме:

```
fieldName = "brand"
fieldValue = "Vereya"
```

и:

```
setProductForm(previousForm => ({
    ...previousForm,
    [fieldName]: fieldValue
}));
```

React взима предишния:

```
{
    name: "Milk",
    brand: "",
    ...
}
```

копира го:

```
...previousForm
```

и заменя само:

```
brand: "Vereya"
```

Получава се нов object:

```
{
    name: "Milk",
    brand: "Vereya",
    calories: "",
    protein: "",
    carbohydrates: "",
    fat: "",
    ingredients: ""
}
```

`name` не се губи именно заради:

```
...previousForm
```

---

Добави засега само **Name + Brand** и ги тествай: напиши различни стойности и виж дали остават независимо една от друга. След това добавяме `calories`, `protein`, `carbohydrates` и `fat`, където има един нов важен детайл: дори при `<input type="number">`, `event.target.value` пристига като **string**, а backend DTO очаква `Double`.

Продължаваме с останалите полета. Понеже вече имаме `productForm` и общия `handleInputChange`, тук не ни трябва нова логика — само свързваме още inputs към същия object.

Под `Brand` добави:

```
<div>
    <label htmlFor="calories">
        Calories
    </label>

    <input
        id="calories"
        name="calories"
        type="number"
        value={productForm.calories}
        onChange={handleInputChange}
    />
</div>

<div>
    <label htmlFor="protein">
        Protein
    </label>

    <input
        id="protein"
        name="protein"
        type="number"
        value={productForm.protein}
        onChange={handleInputChange}
    />
</div>

<div>
    <label htmlFor="carbohydrates">
        Carbohydrates
    </label>

    <input
        id="carbohydrates"
        name="carbohydrates"
        type="number"
        value={productForm.carbohydrates}
        onChange={handleInputChange}
    />
</div>

<div>
    <label htmlFor="fat">
        Fat
    </label>

    <input
        id="fat"
        name="fat"
        type="number"
        value={productForm.fat}
        onChange={handleInputChange}
    />
</div>

<div>
    <label htmlFor="ingredients">
        Ingredients
    </label>

    <textarea
        id="ingredients"
        name="ingredients"
        value={productForm.ingredients}
        onChange={handleInputChange}
    />
</div>
```

### Защо един handler работи за всичко?

Например пишеш `3.5` в Protein.

Този input има:

```
name="protein"
```

Browser/React създава event и нашият handler получава:

```
fieldName = event.target.name;
// "protein"

fieldValue = event.target.value;
// "3.5"
```

След това:

```
setProductForm(previousForm => ({
    ...previousForm,
    [fieldName]: fieldValue
}));
```

е все едно да сме написали:

```
setProductForm(previousForm => ({
    ...previousForm,
    protein: "3.5"
}));
```

Така object-ът постепенно става например:

```
{
    name: "Milk",
    brand: "Vereya",
    calories: "60",
    protein: "3.5",
    carbohydrates: "4.7",
    fat: "3",
    ingredients: "Milk"
}
```

## Но защо числата са `"60"`, а не `60`?

Това е важна особеност на HTML input.

Дори:

```
<input type="number" />
```

при:

```
event.target.value
```

получаваме **string**.

Тоест ако напишеш:

```
60
```

получаваме:

```
"60"
```

а не:

```
60
```

`type="number"` основно казва на browser-а какъв тип въвеждане да предостави; не превръща автоматично `event.target.value` в JavaScript `number`.

Това засега е удобно. Например admin може да изтрие цялата стойност и да имаме:

```
calories: ""
```

Ако веднага насилствено го превръщахме:

```
Number("")
```

резултатът е:

```
0
```

Но:

```
няма въведени calories
```

и:

```
calories = 0
```

са две различни неща.

Затова стратегията ни ще е:

```
Докато admin редактира:

"60"
"3.5"
""
        ↓
React productForm пази strings
        ↓
admin натиска Save
        ↓
ТОГАВА преобразуваме
        ↓
60
3.5
null
        ↓
PUT request
        ↓
Java Double
```

Това разделя две отговорности:

**Form state** → удобно редактиране в browser-а.

**API request** → правилните типове, които backend contract-ът очаква.

След като тези полета работят, вече сме готови за по-интересната част: **Save**. Там за първи път ще вземем целия `productForm`, ще го преобразуваме в request object и ще го изпратим с `PUT` към твоя `UpdatePendingProductRequest`.

Добре. Сега правим **Save**, но първо логиката, после кода.

В backend вече имаш:

```
@PutMapping("/{id}")
public ResponseEntity<Void> updatePendingProduct(
        @PathVariable Long id,
        @RequestBody UpdatePendingProductRequest request) {

    pendingProductService.updatePendingProduct(id, request);

    return ResponseEntity.noContent().build();
}
```

и request:

```
public record UpdatePendingProductRequest(
        String name,
        String brand,
        Double calories,
        Double protein,
        Double carbohydrates,
        Double fat,
        String ingredients
) {}
```

Следователно frontend трябва да изпрати:

```
PUT /admin/pending-products/{id}

{
    "name": "Milk",
    "brand": "Vereya",
    "calories": 60,
    "protein": 3.5,
    "carbohydrates": 4.7,
    "fat": 3,
    "ingredients": "Milk"
}
```

## 1. Първо добавяме service функцията

В `pendingProductService.js`, след `merge`, добави:

```
async updatePendingProduct(pendingProductId, productData) {

    // Save запазва текущата работа на admin-а в PendingProduct,
    // без да приключва review процеса и без да създава глобален Product.
    await apiClient.put(
        `/admin/pending-products/${pendingProductId}`,
        productData
    );
}
```

Защо няма:

```
return response.data;
```

както при barcode?

Защото backend връща:

```
ResponseEntity<Void>
```

и:

```
ResponseEntity.noContent().build();
```

Тоест успешният response е:

```
204 No Content
```

Няма response body, който да ни трябва.

---

# 2. Не можем директно да изпратим `productForm`

В момента примерно имаме:

```
productForm = {
    name: "Milk",
    brand: "Vereya",
    calories: "60",
    protein: "3.5",
    carbohydrates: "4.7",
    fat: "3",
    ingredients: "Milk"
};
```

Забележи:

```
calories: "60"
```

е string.

Backend очаква:

```
Double calories
```

Затова при **границата между UI и API** ще преобразуваме стойностите.

---

## 3. Малка helper функция

В компонента добави:

```
function toNullableNumber(value) {

    // Празното поле означава "няма стойност", а не числото 0.
    // Непразните стойности от HTML number input идват като string
    // и ги преобразуваме към JavaScript number преди API заявката.
    return value === "" ? null : Number(value);
}
```

Ако:

```
value = "60"
```

получаваме:

```
Number("60")
```

→

```
60
```

Ако:

```
value = "3.5"
```

→

```
3.5
```

Ако:

```
value = ""
```

→

```
null
```

Това съвпада много по-добре с:

```
Double calories
```

защото `Double` може да бъде `null`.

---

# 4. Правим Save handler

Добави:

```
async function handleSave() {

    // Form state-ът е оптимизиран за HTML input-и, затова number
    // полетата са strings. Тук изграждаме отделен API request object
    // с типовете, които backend contract-ът очаква.
    const request = {
        name: productForm.name,
        brand: productForm.brand,
        calories: toNullableNumber(productForm.calories),
        protein: toNullableNumber(productForm.protein),
        carbohydrates: toNullableNumber(productForm.carbohydrates),
        fat: toNullableNumber(productForm.fat),
        ingredients: productForm.ingredients
    };

    await pendingProductService.updatePendingProduct(
        currentPendingProduct.id,
        request
    );
}
```

Тук има важна архитектурна идея.

Имаме **два object-а с различна роля**:

```
productForm
    │
    │ object за UI
    ↓
{
    calories: "60",
    protein: "3.5"
}

       ↓ handleSave()

request
    │
    │ object за API
    ↓
{
    calories: 60,
    protein: 3.5
}

       ↓ Axios

JSON
       ↓
Spring
       ↓
UpdatePendingProductRequest
```

Не е задължително UI моделът и API моделът да са абсолютно еднакви.

Това е подобно на идеята, която вече използваш в backend-а с **Entity срещу DTO**.

---

# 5. Добавяме Save button

Под формата:

```
<button
    type="button"
    onClick={handleSave}
>
    Save
</button>
```

Кликването вече ще направи целия flow:

```
Admin попълва input-и
        ↓
handleInputChange()
        ↓
productForm state
        ↓
натиска Save
        ↓
handleSave()
        ↓
правим request object
        ↓
pendingProductService.updatePendingProduct(...)
        ↓
apiClient.put(...)
        ↓
PUT /admin/pending-products/{id}
        ↓
Spring Controller
        ↓
UpdatePendingProductRequest
        ↓
Service
        ↓
PendingProduct се обновява в DB
        ↓
204 No Content
```

## Как да го провериш

Попълни например:

```
Name: Test Milk
Brand: Test Brand
Calories: 60
Protein: 3.5
Carbohydrates: 4.8
Fat: 3.2
Ingredients: Milk
```

Натисни **Save**.

Супер. Значи вече имаме завършен **Save flow**:

```
Admin попълва полетата
        ↓
handleInputChange(event)
        ↓
productForm state
        ↓
Save
        ↓
handleSave()
        ↓
request object
        ↓
pendingProductService.updatePendingProduct()
        ↓
PUT /admin/pending-products/{id}
        ↓
UpdatePendingProductRequest
        ↓
PendingProduct се обновява в DB
        ↓
204 No Content
```

Това е важна граница: **Save не приключва review-а**. Той само запазва текущата работа.

Но сега имаме един реален проблем, който трябва да оправим **преди Approve**:

> Ако запазиш половината форма, затвориш страницата и после я отвориш отново, backend ще върне запазените `name`, `brand`, `calories` и т.н., но нашият `productForm` започва от празни стойности.

Тоест DB може да има:

```
name = Milk
brand = Vereya
calories = 60
```

а React след ново отваряне пак започва с:

```
productForm = {
    name: "",
    brand: "",
    calories: "",
    ...
}
```

А нали точно **затова запазихме Save** — admin да може да прекъсне и после да продължи.

Следващата стъпка следователно не е още Approve. Трябва да направим:

```
GET current PendingProduct
        ↓
currentPendingProduct
        ↓
вземаме вече запазените му полета
        ↓
попълваме productForm
        ↓
admin продължава откъдето е спрял
```

Тук ще научиш още една важна употреба на `useEffect`: не само „изпълни нещо при първото отваряне“, а **„изпълни нещо, когато определен state се промени“**.

Да. Сега ще направим така, че **запазените данни от DB да се върнат обратно във формата**, когато заредим PendingProduct.

Вече имаме първия `useEffect`:

```
useEffect(() => {
    loadPendingProducts();
}, []);
```

`[]` означава:

> Изпълни effect-а при първоначалното показване на компонента.

Сега ни трябва втори effect с друга задача.

## 1. Какъв е проблемът?

GET връща например:

```
currentPendingProduct = {
    id: 15,
    name: "Milk",
    brand: "Vereya",
    barcode: "123456",
    calories: 60,
    protein: 3.5,
    carbohydrates: 4.7,
    fat: 3,
    ingredients: "Milk"
};
```

Но `productForm` си остава:

```
{
    name: "",
    brand: "",
    calories: "",
    protein: "",
    carbohydrates: "",
    fat: "",
    ingredients: ""
}
```

Трябва да кажем:

> Когато `currentPendingProduct` се промени, вземи неговите запазени данни и ги сложи във формата.

---

## 2. Добавяме втори `useEffect`

След сегашния `useEffect` добави:

```
useEffect(() => {

    // Всеки PendingProduct може да има вече запазена admin работа.
    // Когато backend зареди текущия продукт, прехвърляме тези данни
    // във form state-а, за да може admin да продължи редакцията.
    if (currentPendingProduct) {
        setProductForm({
            name: currentPendingProduct.name ?? "",
            brand: currentPendingProduct.brand ?? "",
            calories: currentPendingProduct.calories ?? "",
            protein: currentPendingProduct.protein ?? "",
            carbohydrates: currentPendingProduct.carbohydrates ?? "",
            fat: currentPendingProduct.fat ?? "",
            ingredients: currentPendingProduct.ingredients ?? ""
        });
    }

}, [currentPendingProduct]);
```

Тук има две нови важни неща.

### `[currentPendingProduct]`

Преди имахме:

```
[]
```

Сега:

```
[currentPendingProduct]
```

Това е **dependency array**.

Казваме на React:

> Изпълни този effect, когато `currentPendingProduct` се промени.

Flow-ът става:

```
първи render

currentPendingProduct = null
        ↓
първият useEffect
        ↓
GET backend
        ↓
setCurrentPendingProduct(...)
        ↓
React променя currentPendingProduct
        ↓
rerender
        ↓
вторият useEffect вижда промяната
        ↓
setProductForm(...)
        ↓
формата получава запазените данни
```

Това е различно от първия effect.

```
useEffect(() => {
    loadPendingProducts();
}, []);
```

има смисъл:

> Зареди queue-а при стартиране.

Докато:

```
useEffect(() => {
    // populate form
}, [currentPendingProduct]);
```

има смисъл:

> Когато текущият продукт се смени, синхронизирай формата с него.

Това ще е полезно и след Approve/Reject/Merge, защото тогава зареждаме следващия PendingProduct.

---

## 3. Какво е `?? ""`?

Имаме например:

```
currentPendingProduct.name ?? ""
```

Операторът:

```
??
```

се нарича **nullish coalescing operator**.

Казва приблизително:

> Ако лявата стойност е `null` или `undefined`, използвай дясната.

Например:

```
"Milk" ?? ""
```

→

```
"Milk"
```

Но:

```
null ?? ""
```

→

```
""
```

Това ни трябва, защото backend може да върне:

```
{
    "name": null,
    "brand": null,
    "calories": null
}
```

ако admin още не ги е попълнил.

А нашите controlled inputs предпочитаме да получат:

```
value=""
```

вместо:

```
value={null}
```

Следователно преобразуваме:

```
Backend                 Form

"Milk"       →          "Milk"
null         →          ""
60           →          60
null         →          ""
```

---

## 4. А barcode?

И той е част от запазеното състояние.

Можем да добавим в същия effect:

```
useEffect(() => {

    // Всеки PendingProduct може да има вече запазена admin работа.
    // При смяна на текущия продукт възстановяваме review state-а
    // от последното състояние, записано в backend-а.
    if (currentPendingProduct) {
        setProductForm({
            name: currentPendingProduct.name ?? "",
            brand: currentPendingProduct.brand ?? "",
            calories: currentPendingProduct.calories ?? "",
            protein: currentPendingProduct.protein ?? "",
            carbohydrates: currentPendingProduct.carbohydrates ?? "",
            fat: currentPendingProduct.fat ?? "",
            ingredients: currentPendingProduct.ingredients ?? ""
        });

        setBarcode(currentPendingProduct.barcode ?? "");

        // Резултатът от стара barcode проверка не трябва да се пренася
        // към новозареден продукт. Barcode-ът може да се провери отново.
        setBarcodeCheckResult(null);
    }

}, [currentPendingProduct]);
```

Това е по-добрият вариант.

Но забележи нещо важно:

```
barcode = "123456"
```

можем да възстановим.

Но:

```
barcodeCheckResult
```

**не можем** да възстановим от GET-а, защото backend response-ът не съдържа:

```
exists=true/false
existingProduct=...
```

И не трябва да си измисляме резултата.

Затова след refresh може да видиш:

```
Barcode: 123456
[Check barcode]
```

и admin го проверява отново.

Това всъщност е полезно, защото междувременно глобалният каталог може да се е променил.

---

### Тествай точно този сценарий

Попълни продукт → `Save` → refresh на browser-а.

След GET трябва автоматично да получиш:

```
Barcode:        предишният barcode
Name:           предишното име
Brand:          предишният brand
Calories:       предишните calories
Protein:        ...
...
```

Тогава вече **Save изпълнява първоначалната си бизнес цел**: можеш да прекъснеш review-а и после да продължиш от запазеното състояние.

След като това работи, вече има смисъл да минем към **Approve**, където има важен въпрос: как да гарантираме, че ако admin е променил полетата и веднага натисне Approve без нов Save, няма да одобрим старите стойности от DB.

обясни пак пълният процес в кода след това не виждам след попълване на barcode къде се запазва, няма такова поле.

Да. Тук всъщност имаме **две различни групи данни** и затова изглежда сякаш barcode няма къде да се запази:

1. barcode има **отделен React state**.
2. Останалите полета са събрани в productForm.
3. Barcode се записва в backend **още при Check barcode**, а не при Save.

Нека проследим целия код от началото до DB.

## 1. Страницата се отваря

Първоначално React създава state-овете:

```
const [currentPendingProduct, setCurrentPendingProduct] = useState(null);
const [remainingCount, setRemainingCount] = useState(0);

const [barcode, setBarcode] = useState("");
const [barcodeCheckResult, setBarcodeCheckResult] = useState(null);

const [productForm, setProductForm] = useState({
    name: "",
    brand: "",
    calories: "",
    protein: "",
    carbohydrates: "",
    fat: "",
    ingredients: ""
});
```

В този момент имаме:

`currentPendingProduct = null barcode = "" barcodeCheckResult = null productForm = { name: "", brand: "", ... }`

---

# 2. Първият useEffect зарежда PendingProduct

Имаме:

```
useEffect(() => {
    loadPendingProducts();
}, []);
```

[] казва:

> Изпълни това при първоначалното показване на компонента.

Извиква се:

```
loadPendingProducts();
```

а тя:

```
async function loadPendingProducts() {

    const review =
        await pendingProductService.getPendingForReview();

    setCurrentPendingProduct(review.pendingProduct);
    setRemainingCount(review.remainingCount);
}
```

Service:

```
async getPendingForReview() {
    const response =
        await apiClient.get("/admin/pending-products");

    return response.data;
}
```

Получаваме например:

```
{
    "remainingCount": 3,
    "pendingProduct": {
        "id": 15,
        "name": null,
        "brand": null,
        "barcode": null,
        "calories": null,
        ...
    }
}
```

После:

```
setCurrentPendingProduct(review.pendingProduct);
```

и React state става:

`currentPendingProduct ↓ PendingProduct #15`

---

# 3. Вторият useEffect взима данните от PendingProduct

Това е effect-ът, който сега добавяме:

```
useEffect(() => {

    if (currentPendingProduct) {
        setProductForm({
            name: currentPendingProduct.name ?? "",
            brand: currentPendingProduct.brand ?? "",
            calories: currentPendingProduct.calories ?? "",
            protein: currentPendingProduct.protein ?? "",
            carbohydrates: currentPendingProduct.carbohydrates ?? "",
            fat: currentPendingProduct.fat ?? "",
            ingredients: currentPendingProduct.ingredients ?? ""
        });

        setBarcode(currentPendingProduct.barcode ?? "");
        setBarcodeCheckResult(null);
    }

}, [currentPendingProduct]);
```

Тоест:

`currentPendingProduct │ ├── barcode ─────────→ barcode state │ ├── name ──────┐ ├── brand ─────┤ ├── calories ──┤ ├── protein ───┼────→ productForm state ├── carbs ─────┤ ├── fat ───────┤ └── ingredients┘`

Това е важната разлика.

**Barcode не е в productForm.**

---

# 4. Admin пише barcode

Имаш:

```
<input
    id="barcode"
    type="text"
    value={barcode}
    onChange={handleBarcodeChange}
/>
```

Например пишеш:

`123456`

Извиква се:

```
function handleBarcodeChange(event) {

    const newBarcode = event.target.value;

    setBarcode(newBarcode);
    setBarcodeCheckResult(null);
}
```

Следователно:

`barcode = "123456"`

Но тук си напълно прав:

> На този етап barcode още НЕ е записан в DB.

Той е само в React state.

---

# 5. Къде тогава се записва barcode?

Когато натиснеш:

`Check barcode`

се извиква:

```
async function handleCheckBarcode() {

    const result = await pendingProductService.checkBarcode(
        currentPendingProduct.id,
        barcode
    );

    setBarcodeCheckResult(result);
}
```

Например:

`currentPendingProduct.id = 15 barcode = "123456"`

Извикваме:

```
pendingProductService.checkBarcode(
    15,
    "123456"
);
```

---

## 6. Service-ът изпраща barcode към backend

Там имаме:

```
async checkBarcode(pendingProductId, barcode) {

    const response = await apiClient.patch(
        `/admin/pending-products/${pendingProductId}/barcode`,
        {
            barcode: barcode
        }
    );

    return response.data;
}
```

Следователно HTTP заявката е:

`PATCH /admin/pending-products/15/barcode`

с body:

```
{
    "barcode": "123456"
}
```

Ето **къде barcode напуска React**.

---

# 7. Backend го получава

Controller:

```
@PatchMapping("/{id}/barcode")
public CheckBarcodeResponse checkBarcode(
        @PathVariable Long id,
        @Valid @RequestBody CheckBarcodeRequest request) {

    return pendingProductService.checkBarcode(
            id,
            request.barcode()
    );
}
```

Spring превръща JSON:

```
{
    "barcode": "123456"
}
```

в:

```
CheckBarcodeRequest
```

и:

```
request.barcode()
```

е:

`"123456"`

После:

```
pendingProductService.checkBarcode(id, request.barcode());
```

---

# 8. Backend service записва barcode

Това беше важната част от backend логиката ни:

`checkBarcode() ↓ намира PendingProduct #15 ↓ записва barcode в PendingProduct ↓ проверява Product таблицата за същия barcode ↓ връща exists true/false`

Точно затова endpoint-ът е:

`PATCH /{id}/barcode`

а не просто:

`GET /products?barcode=...`

Нашата операция прави **две неща**:

`1. записва barcode в PendingProduct 2. проверява дали съществува Product с този barcode`

Това беше и коментарът в backend controller-а:

```
// Service-ът едновременно записва barcode-а в PendingProduct
// и проверява дали вече има глобален Product със същия barcode.
return pendingProductService.checkBarcode(id, request.barcode());
```

Следователно след Check barcode DB вече има:

`PendingProduct #15 barcode = "123456"`

---

# 9. Backend връща резултат

Ако няма Product:

```
{
    "exists": false,
    "product": null
}
```

Frontend прави:

```
setBarcodeCheckResult(result);
```

и получаваме:

`barcodeCheckResult { exists: false, product: null }`

React rerender-ва.

Условието:

```
barcodeCheckResult.exists ? (
    // Existing product
) : (
    // Form
)
```

избира формата.

---

# 10. Admin попълва формата

Например:

`Name: Milk Brand: Vereya Calories: 60`

handleInputChange() постепенно променя:

```
productForm
```

докато получим:

```
{
    name: "Milk",
    brand: "Vereya",
    calories: "60",
    protein: "3.5",
    carbohydrates: "4.7",
    fat: "3",
    ingredients: "Milk"
}
```

Това още е само frontend state.

---

# 11. Натискаме Save

Тогава:

```
async function handleSave() {

    const request = {
        name: productForm.name,
        brand: productForm.brand,
        calories: toNullableNumber(productForm.calories),
        protein: toNullableNumber(productForm.protein),
        carbohydrates: toNullableNumber(productForm.carbohydrates),
        fat: toNullableNumber(productForm.fat),
        ingredients: productForm.ingredients
    };

    await pendingProductService.updatePendingProduct(
        currentPendingProduct.id,
        request
    );
}
```

И тук забележи:

```
const request = {
    name: ...,
    brand: ...,
    ...
};
```

**няма barcode.**

Това не е грешка.

Barcode вече беше записан от:

`PATCH /{id}/barcode`

---

# Имаме две отделни операции

Това е ключът:

`ADMIN REVIEW │ въвежда barcode ↓ barcode React state ↓ Check barcode ↓ PATCH /{id}/barcode ↓ ┌───────────┴───────────┐ ↓ ↓ записва barcode проверява Product в PendingProduct със същия barcode ↓ exists true/false ↓ ┌─────────────────┴──────────────┐ ↓ ↓ exists=true exists=false ↓ ↓ Existing Product productForm ↓ ↓ Merge admin попълва ↓ Save ↓ PUT /{id} ↓ записва name, brand, nutrition, ingredients`

## Защо сме ги разделили?

Защото barcode има специална бизнес роля.

Той не е просто още едно поле като name.

Когато admin въведе barcode, системата трябва **преди останалата работа** да реши:

`Този реален продукт вече съществува ли?`

Ако да:

`няма смисъл да създаваме нов Product → Merge`

Ако не:

`попълваме данните → Save → по-късно Approve`

Затова barcode има собствен flow:

`barcode → PATCH + check`

а останалата форма:

`productForm → PUT + save`

И точно затова имаме отделно:

```
const [barcode, setBarcode] = useState("");
```

и:

```
const [productForm, setProductForm] = useState({...});
```

а не barcode вътре в productForm.

След refresh новият GET връща **и двете групи от DB**, а вторият useEffect ги разпределя обратно: currentPendingProduct.barcode → barcode state, а запазените editable полета → productForm. Това затваря целия цикъл **DB → React → редакция → DB → React**.

преди да продължим как човек се сеща да направи всичко това. Като го видя изглежда логично, но дори тогава не разбирам всичко и не мисля, че ще успея сам да го направя. Уча се сега, но пак изглежда доста. Не знам какъв mindset ми трябва. Може би преди е ставало по-бавно за сравнение с ai сега.

Това, което усещаш, е нормалният проблем при преминаването от **„разбирам кода, когато го видя“** към **„мога сам да проектирам решението“**. Това са различни умения.

При SFS например изглежда сякаш човек трябва изведнъж да се сети за useState, object, useEffect, PATCH, PUT, DTO, refetch и т.н. В действителност добрият подход е почти обратният: **не започваш от технологиите. Започваш от поведението, което системата трябва да има.**

### Как бих мислил без да знам предварително кода

Вземаме точно сегашната функция. Не мисля първо:

> „Тук трябва useEffect и object state.“

Мисля:

> „Какво трябва да може да направи администраторът?“

Получаваме:

`1. Отваря review 2. Вижда най-стария продукт 3. Въвежда barcode 4. Системата проверява дали продуктът съществува 5. Ако съществува → Merge 6. Ако не → попълва данните 7. Може да ги Save-не 8. Ако се върне утре → трябва да продължи оттам 9. Накрая → Approve или Reject`

Това вече е огромна част от „програмирането“, въпреки че няма един ред код.

После вземаш **само една стъпка**:

> „Admin въвежда Name. Къде трябва да стои написаното?“

Във frontend-а трябва някак да го пазим → state.

После:

> „Имам 7 свързани полета.“

Може 7 state-а, но по-удобно е един object → productForm.

После:

> „Как променям само name, без да загубя останалите?“

Търсиш/знаеш начина в React → spread + state setter.

После:

> „Добре, данните са във frontend-а. Как да останат след refresh?“

Frontend state изчезва при refresh → трябва да са в DB → PUT.

После:

> „А като отворя страницата пак?“

DB ги има, но input-ите не → GET трябва да ги върне → state трябва да се попълни с тях.

Тогава стигаш до useEffect.

Виж разликата:

`❌ Трябва да се сетя: useEffect + useState + object + spread + PUT + DTO... ✅ Имам проблем: След refresh искам написаното да остане. Какво ми трябва, за да стане? → DB трябва да го пази → GET трябва да го върне → React трябва да го сложи във формата → как реагирам на заредения currentPendingProduct? → useEffect`

Технологията идва **след проблема**.

---

## Mindset-ът, който според мен трябва да развиваш

Не:

> „Трябва да мога сам да напиша цялото това нещо от празен файл.“

А:

> **„Трябва да мога да определя какво трябва да се случи следващо и да намеря начин да го реализирам.“**

Например утре може да не помниш синтаксиса:

```
setProductForm(previousForm => ({
    ...previousForm,
    [fieldName]: fieldValue
}));
```

Това не е голям проблем.

Много по-важно е да можеш да кажеш:

> „Имам object с цялата форма. Искам да променя едно property, без да загубя другите, и понеже е React state, не трябва просто да го mutate-на.“

После синтаксисът може да се провери за 20 секунди.

Това е съвсем различно от човек, който знае синтаксиса наизуст, но не знае **защо му трябва object state**.

### Има три нива

`Ниво 1 „Виждам решението и горе-долу го разбирам.“ ↓ практика Ниво 2 „Виждам проблема и се сещам какъв тип решение ми трябва, но проверявам синтаксис / документация / AI.“ ↓ много решени проблеми Ниво 3 „Виждам проблема и вече съм срещал подобен модел. Мога бързо да построя решение и да проверя детайлите.“`

В момента естествено си между първото и второто за React. За backend вече на места си по-напред, защото си решавал повече такива проблеми.

---

## AI променя какво си струва да учиш

Да, преди подобна работа често е ставала по-бавно. Трябвало е повече да търсиш документация, Stack Overflow, примери, да дебъгваш сам и т.н. AI може драматично да съкрати това.

Но това **не премахва необходимостта да разбираш системата**.

Напротив, според мен ценната комбинация става:

`Човек: Какъв е проблемът? Как трябва да работи системата? Какви са edge cases? Добро ли е предложеното решение? Как да разделим задачата? Работи ли реално? + AI: Какъв е синтаксисът? Как се използва API-то? Дай варианти. Напиши boilerplate. Обясни грешката. Предложи implementation.`

Ако AI ти генерира целия Admin Panel, без да разбираш какво става, при първата промяна:

> „Искам Save да работи различно.“

ще се изгубиш.

Но ако ти разбираш:

`barcode state productForm state ↓ API requests ↓ PendingProduct ↓ DB`

можеш да кажеш на AI какво искаш, да разгледаш предложението и да го промениш.

---

## И точно затова начинът, по който правим SFS, има смисъл

Не се опитваме първо да научиш:

> „React от глава до пети.“

Имаме реален проблем:

> „Admin трябва да review-ва продукт.“

И докато го решаваме, срещаме:

`Трябва да пазим нещо в UI → useState Има много свързани стойности → object Променяме едно поле → spread Нещо трябва да стане след зареждане/промяна → useEffect Frontend трябва да говори с backend → Axios/service Трябва да запазим частична работа → PUT Трябва да променим само barcode → PATCH След terminal action искаме актуалното DB състояние → refetch`

Така понятията вече имат **причина да съществуват**.

И след като направиш 5–10 различни features по този начин, ще започнеш сам да разпознаваш моделите. При следваща форма вероятно няма да питаш „как да направя това?“, а ще си кажеш:

> „Това са няколко свързани input-а → вероятно object state.“

Това е развитието, което търсим.

Не си поставяй критерий **„мога ли днес да построя целия SFS без помощ?“**. По-добрият критерий е:

> **„Когато срещна следващия проблем, мога ли да го разбия на по-малки въпроси и да разбера решението, което намирам с инструментите си?“**

А с времето все по-голяма част от отговорите на тези малки въпроси вече ще са ти познати. Това е много по-близо до реалното разработване на софтуер, отколкото запаметяването на огромно количество код.

Започваме само с първия проблем: **защо запазената форма е скрита и как да решим кога да я показваме.**

## 1. Как работи условието ни в момента

В JSX логиката е приблизително:

```
{barcodeCheckResult && (
    barcodeCheckResult.exists ? (
        // показваме намерения Product + Merge
    ) : (
        // показваме productForm
    )
)}
```

Нека го прочетем като нормален език:

```
Има ли barcodeCheckResult?
        │
        ├── НЕ → не показвай нищо
        │
        └── ДА
             │
             ├── exists = true
             │      → Existing Product
             │
             └── exists = false
                    → productForm
```

Това е било правилно в началото, когато flow-ът беше:

```
въвеждаш barcode
→ Check
→ чак тогава решаваме какво да покажем
```

Но след като добавихме **Save**, вече имаме още едно възможно състояние.

---

## 2. Новото състояние

Admin вчера е направил:

```
barcode = 123
name = Milk
brand = Vereya
```

и е натиснал Save.

Днес GET връща:

```
currentPendingProduct = {
    barcode: "123",
    name: "Milk",
    brand: "Vereya",
    ...
};
```

Вторият `useEffect` прави:

```
currentPendingProduct.barcode
        ↓
barcode

currentPendingProduct.name/brand/...
        ↓
productForm
```

Но също правим:

```
setBarcodeCheckResult(null);
```

Следователно:

```
barcode = "123"

productForm = {
    name: "Milk",
    brand: "Vereya",
    ...
}

barcodeCheckResult = null
```

И JSX пита:

```
barcodeCheckResult има ли?
```

Отговорът е:

```
НЕ
```

и крие формата.

Значи проблемът **не е в зареждането на данните**. Проблемът е правилото за това **кога формата е видима**.

---

# 3. Какво трябва да бъде новото правило?

Вече имаме две причини да покажем формата:

```
Причина 1:
Току-що проверихме barcode
и Product НЕ съществува.

ИЛИ

Причина 2:
PendingProduct вече има запазена admin работа.
```

Тоест концептуално:

```
showForm =
    barcodeCheckResult казва "не съществува"
    ИЛИ
    имаме вече запазена форма
```

Тук обаче възниква следващият въпрос:

> Как React ще знае дали има „вече запазена форма“?

Не искам още да пишем условие като:

```
currentPendingProduct.name !== null
```

защото това започва да смесва **данните** с **състоянието на review flow-а**.

Например какво ако admin е Save-нал само:

```
ingredients
```

а `name` още е `null`?

Или само `brand`?

Тогава проверката само на `name` би била грешна.

---

## 4. Имаме два начина

**Вариант A:** извеждаме дали има запазена работа от самите данни:

```
const hasSavedProductData =
    currentPendingProduct?.name != null ||
    currentPendingProduct?.brand != null ||
    currentPendingProduct?.calories != null ||
    currentPendingProduct?.protein != null ||
    currentPendingProduct?.carbohydrates != null ||
    currentPendingProduct?.fat != null ||
    currentPendingProduct?.ingredients != null;
```

Това означава:

> Ако поне едно от editable полетата вече е записано, имаме започната форма.

Тогава:

```
hasSavedProductData = true
→ можем да покажем формата
```

**Вариант B:** backend изрично да ни каже състоянието, например `hasSavedReviewData`.

За сегашната система това би било излишно. Backend вече ни връща самите данни и можем лесно да определим това във frontend-а.

Затова бих избрал **A**.

---

# 5. Но първо една нова концепция: derived value

Не е нужно да правим:

```
const [hasSavedProductData, setHasSavedProductData] = useState(false);
```

Защо?

Защото вече имаме информацията в:

```
currentPendingProduct
```

Можем просто да **изчислим** нова стойност от нея:

```
const hasSavedProductData =
    currentPendingProduct?.name != null ||
    currentPendingProduct?.brand != null ||
    currentPendingProduct?.calories != null ||
    currentPendingProduct?.protein != null ||
    currentPendingProduct?.carbohydrates != null ||
    currentPendingProduct?.fat != null ||
    currentPendingProduct?.ingredients != null;
```

Това се нарича **derived value** — производна/изчислена стойност.

Имаме:

```
currentPendingProduct
        ↓
изчисляваме
        ↓
hasSavedProductData
```

Не създаваме втори state, който трябва постоянно да синхронизираме.

---

## 6. Какво е `?.`

Тук:

```
currentPendingProduct?.name
```

`?.` е **optional chaining**.

Първоначално имаме:

```
currentPendingProduct = null;
```

Ако напишем:

```
currentPendingProduct.name
```

JavaScript ще се опита да вземе `name` от `null` → грешка.

С:

```
currentPendingProduct?.name
```

казваме:

> Ако `currentPendingProduct` съществува, вземи `name`. Ако е `null`/`undefined`, просто върни `undefined`.

---

# 7. Какво е `!= null` тук?

Умишлено използваме:

```
value != null
```

Тук loose comparison има полезно конкретно значение — хваща и:

```
null
```

и:

```
undefined
```

Но например:

```
0
```

не се счита за липсваща стойност.

Това е важно за nutrition полетата, защото:

```
fat = 0
```

може да е реална записана стойност.

Не искаме да правим:

```
if (currentPendingProduct.fat)
```

защото JavaScript третира `0` като falsy.

---

# Какво да добавиш сега

Вътре в компонента, след state declarations, добави:

```
// Запазената admin работа може да съдържа само част от полетата.
// Ако поне едно editable поле вече има стойност от backend-а,
// считаме review формата за започната и можем да я покажем отново.
const hasSavedProductData =
    currentPendingProduct?.name != null ||
    currentPendingProduct?.brand != null ||
    currentPendingProduct?.calories != null ||
    currentPendingProduct?.protein != null ||
    currentPendingProduct?.carbohydrates != null ||
    currentPendingProduct?.fat != null ||
    currentPendingProduct?.ingredients != null;
```

**Засега не променяй JSX условието.**

Първо искам да разделим работата така:

```
Стъпка 1 — готова:
Можем ли да определим дали DB съдържа започната форма?
→ hasSavedProductData

Стъпка 2:
Как да комбинираме това с barcodeCheckResult,
за да решим какво точно да се показва?
```

Така ще видиш как изграждаме решението от бизнес правилото към React кода, вместо да сложим едно голямо условие и после да се чудим защо работи.

тоест функциите дали са async или не няма значение за callstack-а, а за вътрешността на функцията, че може да се ползва await и връща обект Promise от резултата, който съдържа данни или грешка нещо като Optional?

Да — разсъждаваш почти правилно.

async не прави магически функцията „отделна нишка“ и не означава, че тя изчезва от call stack-а. Когато я извикаш, тя пак влиза в call stack-а и започва да се изпълнява нормално.

Пример:

```
async function test() {
    console.log("A");
    console.log("B");
}
```

Това се изпълнява синхронно до края:

`A B`

Нищо специално не става само защото има async.

Специалното е:

`async function → позволява await → винаги връща Promise`

Например:

```
async function getNumber() {
    return 5;
}
```

не връща директно:

```
5
```

а концептуално:

```
Promise.resolve(5)
```

Тоест:

```
const x = getNumber();

console.log(x);
```

x е Promise.

А:

```
const x = await getNumber();
```

тогава x е:

```
5
```

### А за сравнението с Optional

Има прилика, но Promise и Java Optional решават различни проблеми.

Optional<T> казва:

> „Може да има стойност, може и да няма.“

Пример:

```
Optional<User>
```

е концептуално:

`има User или няма User`

Promise<T> казва:

> „Тази операция ще приключи по-късно.“

И има основно състояния:

`pending ↓ ├── fulfilled → има успешен резултат │ └── rejected → има грешка`

Например:

```
Promise<Product>
```

можеш грубо да си го представиш като:

> „Обещание, че в бъдеще или ще получа Product, или операцията ще се провали с грешка.“

Така че по-близка аналогия е:

`Optional<T> → има / няма стойност Promise<T> → още чака / успех с T / грешка`

И за call stack-а най-важният момент идва при await:

```
async function test() {
    console.log("A");

    const data = await getData();

    console.log("B");
}
```

Става приблизително:

`test влиза в call stack ↓ печатаме A ↓ стигаме await ↓ getData още не е готово ↓ test се прекъсва временно и освобождава stack-а ↓ друг JavaScript може да се изпълнява ↓ Promise приключва ↓ продължението на test се планира ↓ test продължава от след await ↓ печатаме B`

Така че твоето изречение бих го коригирал съвсем малко:

> async не означава отделно изпълнение в call stack-а. То позволява await и кара функцията да връща Promise. Promise не е като Optional, а представлява бъдещ резултат: чакащ, успешен или неуспешен.

как js преобразува каквото е получил като обект в променлива като няма тип. Наприме Promise се присвоява към const a.

Тук ключовото е: **JavaScript не преобразува Promise в променливата.** Просто променливата сочи към стойността/обекта, който е върнат.

Например:

```
const a = getData();
```

ако getData() върне Promise, става концептуално:

`getData() ↓ Promise object ↓ const a ───────► Promise object`

В Java си свикнал типът да е записан при променливата:

```
String name = "Ivan";
int age = 22;
```

и name може да съдържа само String.

В JavaScript:

```
let a = "Ivan";
a = 22;
a = true;
a = {};
```

Това е позволено, защото JavaScript е **динамично типизиран**.

По-точно:

> В JavaScript **стойността има тип**, а променливата няма фиксиран тип.

Например:

```
const a = Promise.resolve(5);
```

a е име, което сочи към Promise object.

Самият Promise знае какъв вид обект е на runtime:

```
console.log(a instanceof Promise); // true
```

Няма нужда да пишеш:

```
Promise a = ...
```

както би очаквал от Java.

### А какво прави const?

const не означава „типът е константен“.

Означава:

> След като a е свързано с тази стойност, не можеш да присвоиш друга стойност на a.

Например:

```
const a = Promise.resolve(5);

a = 10; // ❌ Error
```

Но това няма общо с типа Promise.

При let:

```
let a = Promise.resolve(5);

a = 10;       // ✅
a = "hello";  // ✅
```

Променливата може да започне с Promise и после да сочи към number или string.

### А какво става с await?

Тук става интересното:

```
const a = getData();
```

Ако getData() връща:

`Promise<Product>`

тогава:

`a ↓ Promise`

Но:

```
const a = await getData();
```

await изчаква Promise-а и взима **стойността, с която Promise-ът е fulfilled**.

Тоест:

`getData() ↓ Promise ↓ чакаме ↓ Promise fulfilled с Product ↓ await взима Product ↓ const a ─────► Product`

Пример:

```
async function getNumber() {
    return 5;
}
```

Без await:

```
const a = getNumber();

console.log(a);
// Promise
```

С await:

```
const a = await getNumber();

console.log(a);
// 5
```

Самата a не се „преобразува“. Просто в двата случая ѝ присвояваш **различна стойност**:

`без await: a → Promise с await: a → резултата вътре в Promise-а`

Това е една от най-големите разлики спрямо Java:

`Java: променливата има предварително определен тип Product a JavaScript: променливата е име/връзка към стойност const a ↓ самата стойност носи информация какъв тип е`

И затова JavaScript може на runtime да провери:

```
typeof a
```

или за обекти:

```
a instanceof Promise
```

без ти предварително да си написал типа.

stack и heap като при java ли работят? кога знам, че пиша на typescript?

Да, идеята за **stack и heap е сходна с Java**, но не е добре да я мислиш като 1:1 една и съща реализация.

В Java обикновено учиш:

`stack → локални променливи → stack frames за методите → references към обекти heap → самите обекти → масиви`

В JavaScript концептуално пак имаш:

`call stack → кои функции в момента се изпълняват → локални execution contexts heap → обекти → масиви → функции → Promise-и → други динамично създадени структури`

Пример:

```
function test() {
    const user = {
        name: "Ivan"
    };
}
```

Концептуално можеш да го мислиш така:

`Call Stack ---------------- test() user ───────────────┐ │ ▼ Heap ---------------- { name: "Ivan" }`

Тоест user съдържа reference към обекта.

Но при JavaScript има важна подробност: спецификацията на езика не казва буквално „това задължително трябва да е физически на stack, това задължително на heap“. Това е работа на engine-а, например V8.

Затова за учене е напълно ОК да използваш Java модела:

> primitive стойности и execution frames — мисли ги около stack-а  
> object-и — мисли ги в heap-а

но знай, че engine-ът може да оптимизира нещата.

За Promise например:

```
const a = getData();
```

можеш да си го представиш:

`stack: a ────────────────┐ │ heap: ▼ Promise object`

---

За втория въпрос: **знаеш, че пишеш TypeScript основно по файловете и по синтаксиса.**

JavaScript файлове:

`.js .jsx`

TypeScript файлове:

`.ts .tsx`

За React:

`Component.jsx → React + JavaScript Component.tsx → React + TypeScript`

При твоя проект, ако имаш например:

`AdminPendingProductsPage.jsx`

това е **JavaScript + JSX**, не TypeScript.

TypeScript започва да личи и по това, че имаш типове в кода.

JavaScript:

```
function add(a, b) {
    return a + b;
}
```

TypeScript:

```
function add(a: number, b: number): number {
    return a + b;
}
```

Тук:

```
a: number
b: number
): number
```

са TypeScript типове.

Или:

```
interface Product {
    id: number;
    name: string;
    barcode: string;
}
```

после:

```
const product: Product = {
    id: 1,
    name: "Milk",
    barcode: "123"
};
```

Това вече е ясно TypeScript.

Най-просто:

`.jsx → React с JavaScript .tsx → React с TypeScript`

И още нещо важно: TypeScript не е отделен runtime като Java/JVM.

Обикновено става:

`TypeScript ↓ проверка на типовете ↓ преобразуване до JavaScript ↓ browser изпълнява JavaScript`

Тоест браузърът в крайна сметка не изпълнява TypeScript директно, а JavaScript.

Ако искаш, следващото мога да ти обясня точно **как reference към object работи в JS спрямо Java**, защото там приликата е много голяма и ще ти стане още по-ясно.

сега проекта, защо се прави на js react?

За твоя проект React е на JavaScript, защото **React по принцип е JavaScript библиотека за изграждане на UI**, а ти си започнал frontend-а с .jsx файлове.

Тоест при теб структурата е:

`Spring Boot → backend → Java React → frontend → JavaScript + JSX`

Пример:

`AdminPendingProductsPage.jsx`

.jsx означава:

> JavaScript файл, в който може да има JSX синтаксис.

Пример:

```
function Page() {
    return (
        <div>Hello</div>
    );
}
```

Това изглежда като HTML, но е JSX и е част от React кода.

Защо изобщо React е направен с JavaScript?

Защото браузърът естествено работи с:

`HTML CSS JavaScript`

JavaScript е езикът, който управлява логиката в браузъра:

`натискане на бутон ↓ промяна на state ↓ request към backend ↓ получаване на данни ↓ обновяване на UI`

React просто дава по-удобен начин да организираш тази JavaScript логика.

Вместо ръчно:

```
document.getElementById(...)
element.innerHTML = ...
```

ти работиш със:

```
const [product, setProduct] = useState(null);
```

и React сам обновява интерфейса според state-а.

Твоят проект обаче **не е задължително** да бъде на JavaScript. Можеше още от началото да бъде:

`React + TypeScript`

тогава файловете щяха да са:

`.jsx → .tsx .js → .ts`

и вместо:

```
const [product, setProduct] = useState(null);
```

можеше например:

```
const [product, setProduct] =
    useState<Product | null>(null);
```

Така TypeScript щеше да проверява повече грешки още преди приложението да тръгне.

За твоя SFS JavaScript е напълно нормален избор. Даже за учене React + JS е по-лесен старт, защото първо разбираш:

`components props state useEffect events async/await requests`

без едновременно да се бориш и с TypeScript типовете.

След като ти станат ясни React и JavaScript, преминаването към TypeScript после е сравнително естествено.

Накратко:

`защо React + JS? → защото React е JS библиотека → браузърът изпълнява JavaScript → проектът е създаден с .jsx/.js → JavaScript е по-прост старт`

А не защото React **трябва** задължително да е JavaScript — React спокойно може да се пише и с TypeScript.

React е библиотека на js и дава инструменти да се изпълнява js, но автоматично го прави с по-абстрактни инструменти? TS е продължение на js, но как е направен взет е js и има типизация? Тоест пак е js, но е модифициран и не се тегли допълнително език, реално няма как да изтеглиш език, а преводач за него и каквото напишеш се превежда. За това може да имаш грешки. От js пранаписването на код на ts лесно ли става?

Да, почти си го разбрал правилно. Само ще коригирам няколко неща.

React не е точно „инструменти да се изпълнява JS“. По-скоро е:

> **JavaScript библиотека, която ти дава по-високо ниво на абстракция за изграждане и обновяване на UI.**

Вместо ръчно да правиш:

```
document.querySelector(...)
element.textContent = ...
```

ти описваш:

```
const [count, setCount] = useState(0);

return <div>{count}</div>;
```

и React се грижи кога и как DOM-ът да се обнови.

Тоест:

`JavaScript ↓ можеш ръчно да управляваш страницата React ↓ дава ти по-удобен модел: components state props effects rendering`

Относно TypeScript: да, той е по същество **надстройка върху JavaScript**.

Много важно правило:

> Всеки валиден JavaScript код е почти винаги валиден TypeScript код.

TypeScript добавя неща като:

```
let age: number = 22;
```

или:

```
interface Product {
    id: number;
    name: string;
}
```

Но браузърът не използва тези типове.

Преди изпълнение TypeScript кодът се преобразува до JavaScript.

Пример:

```
function add(a: number, b: number): number {
    return a + b;
}
```

след обработката става приблизително:

```
function add(a, b) {
    return a + b;
}
```

Типовете изчезват.

Точно затова TypeScript може да хваща грешки **преди runtime**.

Пример:

```
let age: number = "hello";
```

TypeScript ще каже:

`Type 'string' is not assignable to type 'number'`

дори преди браузърът да изпълни кода.

И да — правилно мислиш и за „езика“. Обикновено не „сваляш езика“, а инсталираш инструменти, които разбират езика.

Например при TypeScript инсталираш компилатора:

```
npm install typescript
```

Това не е „самият език“ като някакъв файл с езика вътре. Това е софтуер, който:

`чете TypeScript ↓ проверява типовете ↓ преобразува до JavaScript`

При React + Vite често build tool-ът върши голяма част от това вместо теб.

От JS към TS обикновено се минава сравнително лесно, особено ако кодът е подреден.

Например това JS:

```
function greet(name) {
    return "Hello " + name;
}
```

може да стане:

```
function greet(name: string): string {
    return "Hello " + name;
}
```

React компонент:

```
function ProductCard({ product }) {
    return <div>{product.name}</div>;
}
```

може да стане:

```
type Product = {
    id: number;
    name: string;
};

type ProductCardProps = {
    product: Product;
};

function ProductCard({ product }: ProductCardProps) {
    return <div>{product.name}</div>;
}
```

При по-голям проект миграцията обикновено става постепенно:

`.js → .ts .jsx → .tsx`

не е нужно всичко наведнъж.

За твоя SFS бих казал, че преминаването по-късно към TypeScript няма да е огромен проблем, защото вече имаш ясни структури като:

`Product PendingProduct InventoryItem Recipe`

Точно такива модели TypeScript описва много добре.

Най-кратко:

`JavaScript = езикът, който се изпълнява React = библиотека върху JS за UI TypeScript = JavaScript + система за типове + проверки TS ↓ преобразува се до JS ↓ браузърът изпълнява JS`

И да: от JS към TS обикновено се преминава доста по-лесно, отколкото от един напълно различен език към друг.

в java същи при компилация се гледат типовете преди runtime, което е когато кода се изпълнява, докато не стигне до състояние да се изключи?

Да, точно.

При Java имаш два основни етапа:

`1. compile time 2. runtime`

При compile time компилаторът проверява неща като:

```
int age = "hello"; // ❌
```

Това ще даде грешка още преди програмата да тръгне, защото типовете не съвпадат.

След успешна компилация Java кодът става bytecode:

`.java ↓ javac ↓ .class bytecode`

После идва runtime:

`JVM стартира програмата ↓ изпълнява bytecode-а ↓ програмата работи ↓ докато приключи нормално, се затвори, crash-не, или бъде спряна`

Тоест да — runtime е периодът, в който програмата реално се изпълнява.

Пример:

```
public static void main(String[] args) {
    System.out.println("Start");

    int x = 5 / 0;

    System.out.println("End");
}
```

Типовете са напълно валидни:

`5 → int 0 → int`

Затова compile time минава.

Но при runtime:

`5 / 0`

хвърля:

`ArithmeticException`

и програмата може да приключи.

Това е разликата:

`compile-time error → кодът не може да бъде компилиран runtime error → кодът е валиден за компилация, но проблем възниква докато се изпълнява`

При Java примерно:

```
String name = 5;
```

е compile-time problem.

Но:

```
String text = null;
System.out.println(text.length());
```

може да мине компилацията, а после при runtime да получиш:

`NullPointerException`

И да — runtime продължава от момента, в който JVM стартира програмата, до момента, в който процесът приключи.

При Spring Boot например:

`стартираш приложението ↓ JVM работи ↓ Spring Boot server работи ↓ приема requests ↓ runtime продължава часове/дни ↓ спираш server-а ↓ runtime приключва`

Точно така трябва да го мислиш.




защо се вика async функция с await?

Защото async функцията **винаги връща Promise**.

Пример:

```
async function getNumber() {
    return 5;
}
```

Когато я извикаш така:

```
const a = getNumber();
```

a не е 5, а:

`Promise`

Концептуално:

`getNumber() ↓ Promise<5> ↓ a = Promise`

Ако искаш директно стойността вътре в Promise-а:

```
const a = await getNumber();
```

тогава:

`getNumber() ↓ Promise ↓ await чака ↓ Promise приключва успешно ↓ a = 5`

Затова често виждаш:

```
const response = await apiClient.get(...);
```

apiClient.get() връща Promise, а ти не искаш самия Promise. Искаш response-а, който ще дойде по-късно.

Но не е задължително всяка async функция да я викаш с await.

Може:

```
getNumber();
console.log("Продължавам");
```

Тогава стартираш async функцията, но **не я чакаш**.

Или:

```
const promise = getNumber();
```

и запазваш самия Promise.

Най-кратко:

`async function → връща Promise извикване без await → получаваш Promise извикване с await → чакаш Promise-а и получаваш резултата му`

И затова await се слага **при извикването**, когато следващият ти код зависи от резултата.

Пример:

```
const product = await loadProduct();

console.log(product.name);
```

Тук трябва да чакаш, защото иначе product би бил Promise, а не самият продукт.

тоест, защото е asynch дори и да няма вътре await пак се вика с await>

Да — **ако функцията е async, тя винаги връща Promise**, дори вътре да няма нито един await.

Пример:

```
async function getNumber() {
    return 5;
}
```

Тук няма await, но:

```
const a = getNumber();
```

a е:

`Promise`

Ако искаш самото 5:

```
const a = await getNumber();
```

тогава:

`a = 5`

Тоест:

`async функция → винаги връща Promise`

независимо дали вътре има await.

Но важната част е: **не си длъжен винаги да я извикваш с await**.

Само когато ти трябва резултатът преди да продължиш:

```
const value = await getNumber();
console.log(value);
```

Ако не ти трябва да чакаш:

```
getNumber();
console.log("Продължавам");
```

Така че:

> async определя какво връща функцията — Promise.  
> await определя дали caller-ът ще изчака този Promise.

И за да се получи резултата трябва да се напише await, което чака да се изпълни функцията преди да продължи. Ако не се напише продължава се напред, докато се изпълнява и връща Promise обект в който е резултата, а не самия резултат?

Точно така.

```
async function getNumber() {
    return 5;
}
```

Без await:

```
const a = getNumber();

console.log(a);
```

a е:

`Promise`

а не директно 5.

Функцията се стартира, връща Promise веднага и кодът продължава нататък.

С await:

```
const a = await getNumber();

console.log(a);
```

тогава тази async функция чака Promise-ът да приключи и чак тогава:

`a = 5`

Най-кратко:

`без await: asyncFunction() → Promise → кодът продължава с await: await asyncFunction() → чака Promise-а → взима резултата → после кодът продължава`

Само една малка техническа поправка: резултатът не е буквално „вътре в Promise-а като поле“, а Promise-ът представлява бъдещ резултат. Когато завърши успешно, казваме, че е **fulfilled с дадена стойност**.

Например:

`Promise pending ↓ fulfilled with 5`

и await ти дава това 5.

не разбрах когато се прати заявка без await и е async автоматично променливата сочи към обект Promise още преди да е получил стойност? Как се достъпва после? Ако се ичака се дава реалната стойност? Но променливата пак има поле? Тоест?

Да — точно тук е тънкият момент.

Ако извикаш async функция **без await**:

```
const a = getData();
```

и getData() е async, тогава **още веднага** a получава Promise обекта.

Пример:

```
async function getData() {
    return 5;
}

const a = getData();
```

Можеш да го мислиш така:

`a ───────► Promise Promise в началото: pending по-късно: fulfilled with 5`

Тоест a не се променя после от Promise към 5.

Това е важно.

a си остава:

`Promise object`

Просто състоянието на този Promise се променя:

`pending ↓ fulfilled with 5`

За да достъпиш резултата по-късно, имаш два основни начина.

Първият е await:

```
const a = getData();

const value = await a;

console.log(value); // 5
```

Тук:

`a → Promise value → 5`

Вторият е .then():

```
const a = getData();

a.then(value => {
    console.log(value); // 5
});
```

.then() означава приблизително:

> „Когато този Promise приключи успешно, изпълни тази функция и ми дай резултата.“

Сега вариантът с директен await:

```
const a = await getData();
```

е различен.

Тук JavaScript първо прави концептуално:

`getData() ↓ Promise ↓ await чака Promise-а ↓ Promise fulfilled with 5 ↓ a = 5`

Затова a вече **не е Promise**.

То е директно:

```
5
```

И няма някакво специално поле като:

```
a.value
```

Не. Ако:

```
const a = await getData();
```

и резултатът е object:

```
{
    id: 10,
    name: "Milk"
}
```

тогава a е точно този object:

```
console.log(a.id);   // 10
console.log(a.name); // Milk
```

Но ако напишеш:

```
const a = getData();
```

тогава:

`a → Promise`

и не можеш директно:

```
a.id // ❌ това не е Product
```

трябва първо:

```
const product = await a;

console.log(product.id);
```

Най-ясно е с този пример:

```
async function getProduct() {
    return {
        id: 10,
        name: "Milk"
    };
}
```

Без await:

```
const a = getProduct();
```

имаш:

`a ↓ Promise ↓ pending... ↓ fulfilled with Product`

После:

```
const product = await a;
```

и вече:

`product ↓ { id: 10, name: "Milk" }`

Докато директно:

```
const product = await getProduct();
```

правиш двете стъпки наведнъж.

Така че можеш да го запомниш:

`const a = asyncFunction(); a = Promise const a = await asyncFunction(); a = резултатът на Promise-а`

И await не „отваря поле от Promise-а“. Той чака Promise-а да приключи и **връща стойността, с която Promise-ът е завършил успешно**.


преди това. Какво става, ако се натинсе бутонът два пъти? Как може друг потребител да достъпи същия сайт и да въведе през това време данни правейки конфликт? Оптимизацията е когато системата се бави и учиш как да я забързаш? Това са въпроси, които се задавам дори и сега да не ми се налага да ги решавам.

Да — и точно тези въпроси започват да те водят от „кодът работи“ към **„системата работи надеждно“**. Те са различни проблеми и е полезно да ги разделяш.

### 1. Какво става, ако натиснеш Approve два пъти?

В сегашния frontend бутонът остава активен по време на заявките. Значи теоретично можеш:

`Click #1 → handleApprove() Click #2 → handleApprove()`

и вече имаш **две едновременно изпълняващи се функции**:

`Click #1: PUT ──→ POST approve ──→ GET Click #2: PUT ──→ POST approve ──→ GET`

Backend защитата ти помага. Първият approve() променя:

`PENDING_REVIEW → APPROVED`

а вторият би трябвало да стигне до:

```
validatePendingReviewStatus(...)
```

и да бъде отказан, защото продуктът вече не е PENDING_REVIEW.

Но frontend-ът пак е изпратил излишни заявки и потребителят може да види грешка.

Затова по-късно правим например:

```
const [isApproving, setIsApproving] = useState(false);
```

и:

```
<button disabled={isApproving}>
    {isApproving ? "Approving..." : "Approve"}
</button>
```

Това е **frontend защита за UX**, но backend проверката пак трябва да остане. Не трябва да разчитаме на disabled бутона за сигурността.

---

### 2. А ако двама администратори отворят един PendingProduct?

Това е по-интересният проблем.

Представи си:

`DB: PendingProduct #5 PENDING_REVIEW Admin A Admin B ↓ ↓ GET GET ↓ ↓ получава #5 получава #5 пише Name = "Milk" пише Name = "Whole Milk" PUT #5 PUT #5 ↓ ↓ DB = "Milk" DB = "Whole Milk"`

Последният запис може просто да **презапише работата на първия**.

Още по-лош сценарий:

`Admin A Admin B GET #5 GET #5 ↓ ↓ редактира редактира ↓ ↓ APPROVE #5 ↓ APPROVED ↓ опитва SAVE/APPROVE`

Тук backend трябва да реши какво е позволено.

Това е цяла категория проблеми, наречена **concurrency** — множество операции работят с едни и същи данни по едно и също време.

Има различни решения. Например можем в бъдеще да направим **claim/lock**:

`Admin A отваря #5 ↓ #5 се claim-ва от Admin A Admin B прави GET ↓ получава #6`

Или optimistic locking:

```
@Version
private Long version;
```

И тогава приблизително:

`Admin A чете version=3 Admin B чете version=3 Admin A записва → version става 4 Admin B опитва да запише version=3 → конфликт → "Product was modified by another admin"`

**Не бих го добавял още**, но е много добре, че го забелязваш. Когато SFS има един admin, практически няма конфликт. Когато има няколко едновременно работещи admin-и, вече става реален архитектурен проблем.

---

### 3. Това ли е оптимизацията — системата се бави и търсиш как да я ускориш?

Да, но **оптимизация** е по-широко понятие.

Имаш някаква характеристика:

`текущо състояние ↓ намирам причината за ненужен разход ↓ променям дизайна/кода ↓ по-добър резултат`

Performance optimization е например:

`Approve отнема 1.2 s ↓ измерваме PUT 300 ms POST 600 ms GET 300 ms ↓ разбираме къде е bottleneck ↓ оптимизираме конкретната причина`

Но оптимизация може да бъде и за:

- по-малко RAM;
- по-малко DB queries;
- по-малко network traffic;
- по-малко CPU;
- по-нисък cloud разход;
- по-бързо зареждане;
- по-добра работа при 100 000 потребители.

Има важен принцип: **първо измерваш, после оптимизираш**.

Не:

`"Това сигурно е бавно" → усложнявам архитектурата`

а:

`"Усеща се бавно" ↓ колко точно? ↓ къде се губи времето? ↓ защо? ↓ коя промяна ще помогне? ↓ измервам отново`

Иначе лесно стигаш до **premature optimization** — усложняваш система, която всъщност няма performance проблем.

При сегашния SFS например аз **не бих обединявал трите заявки само защото виждаме delay**. Първо бихме измерили дали проблемът е network, Supabase DB, createFromPending(), development environment или нещо друго.

Това всъщност са три различни начина да мислиш за една feature:

`Функционалност: "Работи ли Approve?" Надеждност: "Какво става при double click, двама admins, failed request?" Performance: "Колко време отнема и как се държи при натоварване?"`

Тези въпроси не е необходимо винаги веднага да ги решаваш. По-важното е да започнеш да ги **разпознаваш**, да прецениш дали са реален проблем на текущия етап и чак тогава да инвестираш време в решение. Това е много близко до начина, по който се вземат архитектурни решения в реален проект.



const langBtn = document.getElementById("lang-btn");

Този ред:

```
const langBtn = document.getElementById("lang-btn");
```

означава:

> намери HTML елемента, който има id="lang-btn", и запази reference към него в променливата langBtn.

Примерно ако в HTML имаш:

```
<button id="lang-btn">English</button>
```

JavaScript търси елемента с:

```
document.getElementById("lang-btn")
```

и резултатът е самият DOM element.

Тоест:

`HTML: <button id="lang-btn">English</button> ↑ │ document.getElementById("lang-btn") ↓ langBtn ─────► button element`

После можеш да го използваш:

```
langBtn.textContent = "Български";
```

или:

```
langBtn.addEventListener("click", () => {
    console.log("Натиснат е бутонът");
});
```

document представлява текущата HTML страница като DOM структура.

getElementById(...) казва:

> намери ми елемента с този id.

А:

```
const langBtn = ...
```

просто пази reference към намерения element, за да не го търсиш отново всеки път.

Ако няма елемент с:

```
id="lang-btn"
```

тогава:

```
document.getElementById("lang-btn")
```

връща:

```
null
```

Това е класическият начин при обикновен JavaScript. В React обикновено не управляваш DOM елементите така директно, защото React го прави вместо теб чрез state и rendering.



const elements = document.querySelectorAll("[data-key]"); DOM е дърво с html елементи, които имат и css и могат да се променят от действия с js. Тази операция връща някакъв вид лист с който после се итерира през всеки елемент?

Да, точно така.

```
const elements = document.querySelectorAll("[data-key]");
```

прави следното:

`DOM дървото ↓ намира всички HTML елементи, които имат атрибут data-key ↓ връща колекция от тези елементи ↓ elements`

Например ако HTML е:

```
<h1 data-key="welcome">Welcome</h1>

<p data-key="description">
    Description
</p>

<button data-key="btnText">
    English
</button>
```

тогава:

```
document.querySelectorAll("[data-key]")
```

намира и трите.

Резултатът не е точно JavaScript Array, а:

`NodeList`

Тоест нещо подобно на списък:

`elements │ ├── [0] → <h1 ...> ├── [1] → <p ...> └── [2] → <button ...>`

И да — много често после итерираш:

```
elements.forEach(element => {
    console.log(element);
});
```

Това означава:

`вземи първия DOM element ↓ изпълни функцията вземи втория ↓ изпълни функцията вземи третия ↓ изпълни функцията`

### Какво значи "[data-key]"

Това е **CSS selector**.

Същият тип selector можеш да използваш в CSS:

```
[data-key] {
    ...
}
```

означава:

> „Всички елементи, които имат атрибут data-key, независимо каква е стойността му.“

Ако имаш:

```
<h1 data-key="welcome">
<p data-key="description">
<div>
```

ще намери първите два, но няма да намери <div>, защото той няма data-key.

---

И правилно разбираш DOM.

HTML:

```
<body>
    <div>
        <h1>Hello</h1>
        <button>Click</button>
    </div>
</body>
```

браузърът го представя концептуално като дърво:

`document └── html └── body └── div ├── h1 └── button`

JavaScript може да намери тези nodes:

```
document.querySelector(...)
document.querySelectorAll(...)
document.getElementById(...)
```

и да ги променя.

Например:

```
element.textContent = "Нов текст";
```

променя текста в DOM.

Или:

```
element.classList.add("active");
```

добавя CSS class и така индиректно може да промени как изглежда елементът.

Така че твоето разбиране е:

> **HTML създава структурата → браузърът я представя като DOM дърво → CSS определя визуализацията → JavaScript може да намира DOM елементи и да променя съдържание, attributes, classes, styles и поведение.**

А querySelectorAll() е просто инструмент за:

> **„Дай ми всички DOM елементи, които отговарят на този CSS selector.“**




елементите обекти ли са или друго

Да, DOM елементите в JavaScript са обекти.

Например:

```
const btn = document.getElementById("lang-btn");
```

btn сочи към DOM object.

Концептуално:

`btn ↓ HTMLButtonElement object`

Този object има свойства и методи, например:

```
btn.textContent
btn.id
btn.className
btn.style
btn.addEventListener(...)
btn.remove()
```

Тоест можеш да го мислиш почти като Java object:

```
Button btn;
```

с полета и методи.

Пример в HTML:

```
<button id="lang-btn">English</button>
```

В JavaScript браузърът го представя като обект:

`HTMLButtonElement ├── id = "lang-btn" ├── textContent = "English" ├── style = ... ├── classList = ... └── methods: ├── addEventListener() ├── remove() └── ...`

И не всички DOM обекти са един и същи тип.

Например:

```
<div>
<button>
<input>
```

могат да се представят като:

`HTMLDivElement HTMLButtonElement HTMLInputElement`

Те наследяват общи DOM типове нагоре по йерархията.

А при:

```
const elements = document.querySelectorAll("[data-key]");
```

получаваш NodeList, който съдържа references към тези DOM objects.

Тоест:

`NodeList ├── [0] → DOM object ├── [1] → DOM object └── [2] → DOM object`

Затова после:

```
elements.forEach(element => {
    element.textContent = "Hello";
});
```

element на всяка итерация е DOM object и ти използваш неговото свойство textContent.



когат се ползва function, когат asynch function и кога обект в който има async approvePendingProduct(pendingProductId) { , което е метод?

Точно така — тук има **3 отделни идеи**, които е важно да не смесваме:

### 1. function — обикновена функция

В твоя код например:

```
function handleBarcodeChange(event) {
    const newBarcode = event.target.value;
    setBarcode(newBarcode);
}
```

Това е нормална функция. Използваш я, когато вътре **няма нужда да чакаш Promise** с await.

Друг пример от твоя код е toNullableNumber() — тя просто преобразува стойност и веднага връща резултат. 

---

### 2. async function — асинхронна функция

При теб:

```
async function handleApprove() {
    await handleSave();

    await pendingProductService.approvePendingProduct(
        currentPendingProduct.id
    );

    await loadPendingProducts();
}
```

Това е функция, която съдържа асинхронни операции.

async ти позволява да използваш await:

```
await handleSave();
```

Смисълът е:

> „Изчакай handleSave() да приключи и чак тогава продължи надолу.“

При твоя flow това е много важно:

`handleApprove() ↓ чака handleSave() ↓ чака approvePendingProduct() ↓ чака loadPendingProducts() ↓ готово`

И точно това правиш в handleApprove. 

---

### 3. Метод в обект

Това:

```
const pendingProductService = {

    async approvePendingProduct(pendingProductId) {
        ...
    }

};
```

означава:

`pendingProductService │ └── approvePendingProduct()`

approvePendingProduct е **метод на обекта pendingProductService**.

Можеш да мислиш за него подобно на Java:

```
class PendingProductService {

    void approvePendingProduct(Long id) {
        ...
    }
}
```

и после:

```
pendingProductService.approvePendingProduct(id);
```

В JavaScript няма нужда непременно да правиш class. Можеш просто да имаш обект:

```
const pendingProductService = {
    async approvePendingProduct(id) {
        ...
    },

    async rejectPendingProduct(id) {
        ...
    },

    async merge(id) {
        ...
    }
};
```

и после:

```
pendingProductService.approvePendingProduct(id);
```

Точно такова извикване имаш и в страницата си. 

### Най-важното разграничение

async **няма общо с това дали нещо е функция или метод**.

Можеш да имаш:

```
// обикновена функция
function calculate() {
}

// async функция
async function loadData() {
}

// обикновен метод
const service = {
    calculate() {
    }
};

// async метод
const service = {
    async loadData() {
    }
};
```

Тоест:

**функция vs метод** → къде се намира функцията.

**normal vs async** → дали функцията работи асинхронно и можеш да използваш await.

За твоя SFS най-лесното правило засега е:

`Правя проста логика? → function Трябва да await-вам API/Promise? → async function Функцията принадлежи на service object? → method Service методът прави API request? → обикновено async method`

Например handleBarcodeChange() е нормална функция, а handleMerge() е async, защото чака backend заявката и после новото зареждане. 

И една малка забележка по твоя код: toNullableString() при теб е async, но **няма причина да бъде** — тя само връща стойност и не чака нищо. Същото важи за await toNullableNumber(...) в handleSave(). Можем да разгледаме защо това работи въпреки че е излишно.



къде се създадва обекта, защото изгелжда, че само се ползва референция към него

Да — и това е много важно наблюдение. В твоя AdminPendingProductsPage.jsx обектът **не се създава**. Там наистина само получаваш референция към вече създадения обект.

В началото имаш:

```
import pendingProductService from "../../services/pendingProductService";
```

Това означава:

> „Вземи стойността, която pendingProductService.js е export-нал, и тук я наричай pendingProductService.“

Самият обект най-вероятно е създаден в:

`services/pendingProductService.js`

и вероятно изглежда приблизително така:

```
const pendingProductService = {

    async approvePendingProduct(pendingProductId) {
        // ...
    },

    async rejectPendingProduct(pendingProductId, reason) {
        // ...
    }
};

export default pendingProductService;
```

Точно тук:

```
const pendingProductService = {
    ...
};
```

се **създава JavaScript обектът** {}.

После:

```
export default pendingProductService;
```

export-ва стойността — на практика референцията към този обект.

А другият файл:

```
import pendingProductService from "../../services/pendingProductService";
```

получава достъп до същия обект.

Мисли за него така:

`pendingProductService.js ОБЕКТ В ПАМЕТТА ┌─────────────────────────┐ │ approvePendingProduct() │ │ rejectPendingProduct() │ │ merge() │ └─────────────────────────┘ ↑ │ reference │ pendingProductService │ export default │ ↓ ──────────────────────────────────── AdminPendingProductsPage.jsx │ import │ ↓ pendingProductService │ │ reference ↓ СЪЩИЯТ ОБЕКТ В ПАМЕТТА`

Затова когато при теб пише:

```
await pendingProductService.approvePendingProduct(
    currentPendingProduct.id
);
```

можеш да го четеш:

> „Отиди до обекта, към който сочи pendingProductService, намери неговия метод approvePendingProduct и го извикай.“

Това е доста близко до нещо, което вече знаеш от Java. **Разликата е, че тук най-вероятно няма new PendingProductService()**. Самият { ... } създава обекта директно.

Ако ми покажеш pendingProductService.js, можем да минем **ред по ред точно къде се създава обектът → как се export-ва → как import-ът получава референцията**.

в ApplicationContext вид ли е направен при стартиране на програмата

Не. Тук е важната разлика между **Spring/Java** и **React/JavaScript**.

В Spring Boot си свикнал с:

```
@Service
public class PendingProductService {
}
```

Spring при стартиране вижда @Service, създава обект и го държи като **bean в ApplicationContext**:

`Spring Boot стартира ↓ сканира @Service ↓ new PendingProductService() ↓ ApplicationContext ↓ PendingProductService bean`

При твоя JavaScript frontend **няма Spring ApplicationContext**.

Ако pendingProductService.js е:

```
const pendingProductService = {
    async approvePendingProduct(id) {
        ...
    }
};

export default pendingProductService;
```

обектът се създава от този израз:

```
{
    async approvePendingProduct(id) {
        ...
    }
}
```

и референцията се записва в:

```
const pendingProductService = ...
```

Когато модулът бъде зареден от JavaScript module system-а заради:

```
import pendingProductService from "../../services/pendingProductService";
```

кодът на модула се изпълнява и обектът се създава.

Тоест можеш **концептуално** да ги сравниш:

`SPRING JAVASCRIPT @Service const service = {...} ↓ ↓ Spring създава bean JS изпълнява модула ↓ ↓ ApplicationContext service object ↓ ↓ @Autowired / DI import ↓ ↓ ползваш bean ползваш референцията`

Но механизмите са различни.

И има една интересна прилика: ако 5 различни JS файла import-нат същия pendingProductService модул, обикновено модулът **не се изпълнява наново 5 пъти**. Всички получават същата export-ната стойност/референция.

Затова на пръв поглед може да ти напомня на singleton Spring bean, но **не е Spring bean и няма ApplicationContext**.

тоест не е обект от който се достъпва методи, а са методи, които са присвоени към някакъв адрес абстрактно наречено обект или това е едно и също

Почти, но по-точно е да кажем: **има реален обект и методите са част от този обект**.

При:

```
const pendingProductService = {
    async approvePendingProduct(id) {
        // ...
    },

    async rejectPendingProduct(id) {
        // ...
    }
};
```

можеш мислено да си представиш паметта така:

`pendingProductService │ │ референция ▼ ┌──────────────────────────────┐ │ OBJECT │ │ │ │ approvePendingProduct ───────┼──► функция │ │ │ rejectPendingProduct ────────┼──► функция └──────────────────────────────┘`

Тоест pendingProductService **не е самият обект в смисъл на променливата** — променливата държи референция към обекта.

А обектът има properties. Стойностите на тези properties могат да бъдат функции. Когато функция е property на обект и я извикваме през обекта, обикновено я наричаме **метод**:

```
pendingProductService.approvePendingProduct(5);
```

Това може да се мисли като:

`pendingProductService ↓ намери обекта ↓ намери property "approvePendingProduct" ↓ стойността му е функция ↓ извикай функцията`

И всъщност това:

```
const service = {
    approve() {
        console.log("approve");
    }
};
```

е кратък синтаксис за нещо концептуално много близко до:

```
const service = {
    approve: function () {
        console.log("approve");
    }
};
```

Затова твоето първо разбиране **„обект, от който достъпвам методи“ е правилното**.

Само ако слезем едно ниво по-надолу, можем да кажем: **обектът съдържа properties → някои properties имат функции като стойности → тези функции наричаме методи.**

Това е много близко и до Java концепцията за обект, макар реализацията и създаването му да са различни.

тоест pendingProductService е обекта или не

Да. **В практичен JavaScript смисъл pendingProductService е обектът.**

Когато имаш:

```
const pendingProductService = {
    async approvePendingProduct(id) {
        // ...
    }
};
```

се случва:

`{ ... } ← създава се ОБЕКТ ↑ │ референция към него │ pendingProductService ← променливата държи референцията`

Затова казваме:

> **„pendingProductService е обект“**

макар че технически по-прецизно е:

> **„pendingProductService е променлива, чиято стойност е референция към обект.“**

Същото е и в Java:

```
Product product = new Product();
```

Обикновено казваме „product е обект“, въпреки че по-точно product е **референция към обекта**.

И после:

```
pendingProductService.approvePendingProduct(5);
```

означава:

`pendingProductService ↓ обект ↓ approvePendingProduct ↓ функция`

Така че за учене спокойно го мисли като **pendingProductService = обект с методи**. Техническата подробност за референцията става важна, когато започнем да говорим за копиране на обекти, ===, мутации и памет.




Promise се връща при async функции? Защото не се изисква Promise в функцията от преди малко, значи може и без await, защото резултата не ни интересува, иначе щеше да вземе Promise, а не стойността като е без await?

Точно така. Само да го формулираме прецизно, защото си хванал основната идея.

В JavaScript **всяка async функция винаги връща Promise**, дори ти никъде изрично да не си написал new Promise(...).

Например:

```
async function getNumber() {
    return 5;
}
```

Ако направиш:

```
const result = getNumber();
```

result **не е 5**, а Promise, който в бъдеще ще се изпълни със стойност 5:

`result → Promise ↓ fulfilled 5`

Ако искаш самата стойност:

```
const result = await getNumber();

console.log(result); // 5
```

Тоест:

```
const a = getNumber();       // Promise
const b = await getNumber(); // 5
```

### Това се връзва точно с твоя handleReject()

Имаш приблизително:

```
async function handleReject() {
    await handleSave();
    await pendingProductService.rejectPendingProduct(...);
    await loadPendingProducts();
}
```

Понеже функцията е async:

```
const x = handleReject();
```

x е Promise.

Но в предишната ти функция ти **не искаше никаква стойност от handleReject()** и нямаше следваща операция, която трябва да чака:

```
function checkRejectInputField() {
    if (...) {
        handleReject();
        return;
    }
}
```

Това е допустимо.

Стартираш handleReject() и казваш приблизително:

> „Не ми трябва резултатът му тук и няма какво да правя след него.“

Затова не е задължително:

```
await handleReject();
```

### Но има важна подробност

await не е само за **получаване на стойност**. Използва се и когато просто искаш да знаеш, че операцията е **приключила**.

Например:

```
await handleReject();
console.log("Reject приключи");
```

Може handleReject() изобщо да не връща полезна стойност. Пак има смисъл от await, защото искаме правилна последователност:

`Reject приключва ↓ console.log`

Без await:

```
handleReject();
console.log("Reject приключи");
```

може да стане:

`handleReject започва ↓ стига до await HTTP request ↓ console.log("Reject приключи") ← всъщност още НЕ е приключил ↓ HTTP приключва`

Затова можеш да си запомниш:

> **async функция → винаги връща Promise.**
> 
> **await → чака Promise да приключи и ако Promise има резултат, получава резултата му.**

Тоест await има две много близки употреби:

```
// Интересува ме стойността
const product = await getProduct();

// Не ме интересува стойността,
// но трябва да изчакам операцията
await saveProduct();
approveProduct();
```

А ако **нито стойността, нито приключването са ти необходими за следващата логика**:

```
someAsyncFunction();
```

може и без await.

Само имай предвид още нещо, което ще разгледаме при error handling: ако стартираш Promise без await, трябва да помислиш **кой ще обработи евентуална грешка**. Това е една от причините на практика да не игнорираме Promise-и безразборно.



първото го оправих. Да обсъдим 2. Защо не трябва два пъти да се натиска. Когато се натисне веднъж започва проверка за празно поле и се праща заявка към backend-а. Ако се пусне втора стига няма да може да се промени, защото backend-а има проверка за PENDING_REVIEW. Тогава защо

Да — разсъждението ти е правилно: **backend проверката вече пази данните**. Ако вторият Reject/Approve стигне след като първият е приключил транзакцията, проверката за PENDING_REVIEW ще го отхвърли.

Затова isProcessing не го добавяме основно за да „спасим базата“. Добавяме го най-вече за **frontend UX и за да не изпращаме излишни конкурентни заявки**.

Да разгледаме двойния Reject. При много бърз двоен клик може да стане:

`CLICK 1 CLICK 2 │ │ validate ✓ validate ✓ │ │ PUT Save #1 ───────────────► PUT Save #2 ───────────────► │ │ POST Reject #1 ────────────► POST Reject #2 ────────────►`

Важно е, че вторият click не чака първия handleReject() да приключи. Това са **две отделни извиквания на функцията**, които могат да вървят едновременно.

Тогава backend защитата прави нещо подобно:

`Reject #1 → PendingProduct е PENDING_REVIEW → REJECTED ✓ Reject #2 → PendingProduct вече е REJECTED → "не може да бъде rejected" ✗`

Това е чудесно като **последна линия на защита**. Backend трябва да я има независимо от frontend-а.

Проблемът е какво вижда потребителят. Първата операция е успешна, но втората може да върне 400/409. Така можеш да получиш грешка в UI, въпреки че действието всъщност е успешно. Освен това сме изпратили излишни Save + Reject заявки и сме натоварили backend-а без причина.

И има още по-интересен race condition. Двете извиквания могат да се преплетат по различен начин:

`Reject handler A ↓ Save A ↓ Reject A ↓ loadPendingProducts() ↓ currentPendingProduct = #6 Reject handler B ↓ ...`

В зависимост от това **кога точно всяка заявка е започнала и какви стойности е уловило конкретното извикване**, започваме да имаме concurrency, която трябва да разсъждаваме внимателно. Backend validation значително ни защитава, но frontend изобщо няма причина да допуска второто действие.

Затова разделяме отговорностите така:

`Frontend: „Admin вече натисна Reject. Не му позволявай да го натисне пак, докато чакаме.“ + Backend: „Нямам доверие на frontend-а. Ще проверя независимо дали този PendingProduct наистина е PENDING_REVIEW.“`

Това е много важен принцип:

**Frontend защита ≠ backend защита.**

Frontend:

`добър UX предотвратяване на случайни действия по-малко ненужни requests`

Backend:

`валидност на данните business rules защита при concurrency защита дори ако заявката не идва от нашия React frontend`

### А защо backend защитата не е достатъчна?

Защото frontend може изобщо да не съществува.

Някой може да изпрати:

`POST /admin/pending-products/5/reject`

през Postman, друг frontend, script или две browser tabs.

Затова никога не бихме премахнали:

```
validatePendingReviewStatus(...)
```

само защото бутонът е disabled.

Обратното също е вярно: няма смисъл да казваме „backend така или иначе ще върне грешка, нека потребителят натиска 15 пъти“.

### Следователно имаме две различни нива

Това всъщност е много добър пример от SFS:

`REJECT │ ┌──────────────┴──────────────┐ │ │ FRONTEND BACKEND │ │ isProcessing status validation │ │ не позволява гарантира правилност втори click на business state │ │ UX integrity`

И ти правилно забеляза нещо важно: **ако backend-ът ни е написан правилно, двойният click не би трябвало да доведе до два успешни Reject-а.** Това означава, че вече сме решили по-сериозния проблем.

Сега isProcessing е следващото ниво: да направим интерфейса да се държи добре, докато асинхронната операция тече.

И тук идва интересният React въпрос, който бих разгледал преди да пишем код: **как isProcessing всъщност ще спре втория click и защо само setIsProcessing(true) вътре в handler-а има една малка concurrency особеност.**

обясни

Да. Нека първо разберем **какво точно е isProcessing**, преди да го пишем.

### 1. Проблемът без него

Имаме:

```
<button onClick={handleReject}>
    Submit
</button>
```

При един click React извиква:

```
handleReject()
```

Но handleReject е async:

```
async function handleReject() {
    await handleSave();
    await rejectPendingProduct();
    await loadPendingProducts();
}
```

Когато стигне първия await, JavaScript не блокира целия интерфейс. Браузърът продължава да приема действия.

Следователно:

`click ↓ handleReject #1 ↓ await HTTP... ← бутонът все още може да се натиска ↓ click ↓ handleReject #2`

Имаме две отделни изпълнения на една и съща функция.

---

## 2. Искаме React да знае „в момента работя“

За това създаваме state:

```
const [isProcessing, setIsProcessing] = useState(false);
```

Можеш да го мислиш като:

`false → в момента няма операция true → Save/Approve/Reject/Merge се обработва`

Например при Reject:

```
async function handleReject() {

    if (!validateRejectReason()) {
        alert("Reason cannot be empty.");
        return;
    }

    setIsProcessing(true);

    await handleSave();
    await pendingProductService.rejectPendingProduct(...);
    await loadPendingProducts();

    setIsProcessing(false);
}
```

Когато извикаме:

```
setIsProcessing(true);
```

React ще направи нов render.

---

## 3. Как state-ът спира бутона?

HTML бутонът има свойство disabled.

Можем да свържем React state с него:

```
<button
    onClick={handleReject}
    disabled={isProcessing}
>
    Submit
</button>
```

Първоначално:

`isProcessing = false ↓ disabled={false} ↓ [ Submit ] ← може да се натиска`

Admin натиска:

```
setIsProcessing(true);
```

React render-ира отново:

`isProcessing = true ↓ disabled={true} ↓ [ Submit ] ← disabled`

Когато операцията приключи:

```
setIsProcessing(false);
```

и React отново render-ира:

`disabled={false}`

Това е хубав пример защо използваме **state**, а не обикновена променлива — промяната трябва да промени UI.

---

# 4. Но тук има една тънкост

setIsProcessing(true) **не променя веднага isProcessing в текущото изпълнение**.

Вече си го срещал с barcode state.

Ако имаме:

```
console.log(isProcessing); // false

setIsProcessing(true);

console.log(isProcessing); // пак false
```

текущият handleReject работи със state стойността от render-а, в който е създаден.

setIsProcessing(true) казва на React:

> При следващия render искам стойността да бъде true.

Следователно процесът е приблизително:

`Render #1 isProcessing = false │ ↓ CLICK │ ↓ setIsProcessing(true) │ ├── React планира update │ ↓ handleReject продължава ... Render #2 isProcessing = true │ ↓ button disabled`

### Може ли потребителят да натисне два пъти преди Render #2?

Теоретично това е причината да не разглеждаме самия React state като някаква абсолютна concurrency lock система.

Но при нормални отделни browser click events React обработва state update-а и render-а преди следващото потребителско събитие, така че:

```
disabled={isProcessing}
```

е нормалният и правилен начин за предотвратяване на обикновен double click в UI.

И пак backend остава последната защита.

---

# 5. Можем да имаме и проверка вътре в handler-а

Например:

```
async function handleReject() {

    if (isProcessing) {
        return;
    }

    if (!validateRejectReason()) {
        alert("Reason cannot be empty.");
        return;
    }

    setIsProcessing(true);

    // ...
}
```

Това се нарича **guard clause**.

Вече използваш същата идея тук:

```
if (!validateRejectReason()) {
    alert(...);
    return;
}
```

Просто добавяме още едно условие:

`handleReject() │ ├── вече processing? → STOP │ ├── reason invalid? → STOP │ └── всичко е наред ↓ започни операцията`

Но пак не трябва да си мислиш за това като за backend lock. React state е UI механизъм.

---

# 6. Има по-важен проблем: как връщаме false?

Да кажем:

```
setIsProcessing(true);

await handleSave();

await pendingProductService.rejectPendingProduct(...);

setIsProcessing(false);
```

Какво ще стане, ако backend Reject хвърли грешка?

Например:

`setIsProcessing(true) ↓ Save ✓ ↓ Reject request ↓ 500 ERROR ❌ ↓ exception`

JavaScript прекратява нормалното изпълнение на функцията.

Следователно:

```
setIsProcessing(false);
```

може никога да не бъде достигнато.

И бутонът остава:

`isProcessing = true Submit → disabled завинаги`

Ето защо loading state естествено ни води към:

```
try {
    ...
} finally {
    ...
}
```

---

# 7. finally решава точно този проблем

Идеята е:

```
setIsProcessing(true);

try {

    await handleSave();
    await reject(...);
    await loadPendingProducts();

} finally {

    setIsProcessing(false);
}
```

finally означава:

> Изпълни това накрая независимо дали операцията е успешна или е хвърлена грешка.

Тоест:

`setIsProcessing(true) ↓ try ┌─────┴─────┐ ↓ ↓ SUCCESS ERROR ↓ ↓ └─────┬─────┘ ↓ finally ↓ setIsProcessing(false)`

Това е една от реалните причини try/finally да съществува.

---

## А catch?

Това вече е следващата част:

```
try {
    ...
} catch (error) {
    // какво показваме на admin?
} finally {
    setIsProcessing(false);
}
```

Разликата е:

`try → опитай операцията catch → какво да направя, ако има грешка? finally → какво трябва да направя ВИНАГИ накрая?`

За нашия SFS:

`try → Save + Reject + GET next catch → покажи грешката finally → махни loading/processing състоянието`

И точно затова бих направил №2 и №3 заедно. **Loading state без правилно error handling лесно може да остави UI заключен.**

Преди да пишем кода обаче има архитектурно решение: дали искаме **един общ isProcessing за цялата review страница**, който блокира всички действия, или отделни isApproving, isRejecting, isMerging, isSaving. За сегашния FIFO flow един общ state вероятно е по-подходящ и мога да ти обясня защо.


Нали е еднонишков js, тогава как изпълнява няколко операции едновременно? ако се load-не следващия PendingProduct не е ли по-добре да се зададе isProcessing true преди него за да покаже готово и тогава да го направи като изчака няколко секунди да се види или по-добре да се даде новата заявка и изпише отдолу еди кой си номер е reject-ната, като това ще се отнася и за другите, ако е approve или merged.? isProcessing е за всички операции, защото, ако една е изпълнена става true и всички останали са заключени? Всеки потребител има своя изолирана инстанция на приложението, което значи и свои states. isProcessing пази от това сам той да опита някак няколко операции approve, reject и merge едновременно? isProcessing така заключва всички операции? Да продължим

Да — почти всичко, което описа, е правилно. И въпросът за еднонишковия JS е точно важният, за да разбереш защо изобщо може да има double-click проблем.

## 1. Как JS е еднонишков, но имаме няколко операции „едновременно“?

JavaScript кодът в основния browser thread изпълнява **една JS задача в даден момент**.

Но когато направиш:

```
await apiClient.post(...);
```

самият JavaScript не стои и не чака мрежата.

Опростено:

`JavaScript │ │ стартира HTTP request ▼ Browser/network ───────────────► Backend │ │ JS thread е свободен │ ├── може да обработи click ├── може да изпълни друг handler └── може да върши друга JS работа`

Когато HTTP request приключи, браузърът сигнализира:

> Promise-ът е готов.

И продължението след await се планира да бъде изпълнено от JS.

Затова при:

```
async function handleReject() {
    console.log("A");

    await reject();

    console.log("B");
}
```

мисли за него като:

`JS thread: "A" │ ├── стартирай request │ └── освободи JS thread HTTP request работи извън JS ──────────► междувременно: JS може да обработи друг click ◄──────── HTTP приключва JS: "B"`

Така **JS кодът не се изпълнява паралелно на основната нишка**, но различни външни операции могат да са в ход едновременно.

Това се нарича **concurrency**, не непременно parallel execution на JavaScript.

Затова може да имаш:

`JS: Reject #1 → HTTP ─────────────────► Reject #2 → HTTP ─────────────────► Backend обработва заявките`

без JavaScript да е станал многонишков.

---

## 2. За показването на „готово“ — не бих чакал изкуствено няколко секунди

Ти предложи две идеи.

### Вариант A

`Reject successful ✓ чакаме 2-3 секунди зареждаме следващия продукт`

Не бих го направил. Забавяме администратора нарочно.

Ако има примерно 100 продукта:

`100 × 2 секунди = 200 секунди`

над 3 минути изкуствено чакане само заради съобщенията.

### Вариант B — по-добър

Точно това, което предложи второ:

`Reject #15 ↓ успешно ↓ веднага GET next ↓ показваме #16 и съобщение: "Pending product #15 was rejected successfully."`

Така admin вече може да работи по #16, но все пак получава feedback за #15.

Същият механизъм може да работи за всички:

`Product #15 approved successfully. Product #16 rejected successfully. Product #17 merged successfully.`

Това по-късно можем да направим като notification/toast.

И забележи защо трябва да запомним старото ID. След:

```
await loadPendingProducts();
```

currentPendingProduct скоро вече ще е **следващият продукт**.

Затова може да ни трябва нещо като:

```
const processedProductId = currentPendingProduct.id;
```

преди операцията.

Това е още един хубав пример защо трябва да мислим кога state се променя.

---

# 3. Разбрал си правилно общия isProcessing

Ако имаме:

```
const [isProcessing, setIsProcessing] = useState(false);
```

и използваме същия state:

```
<button disabled={isProcessing}>
    Approve
</button>

<button disabled={isProcessing}>
    Reject
</button>

<button disabled={isProcessing}>
    Merge
</button>

<button disabled={isProcessing}>
    Save
</button>
```

тогава:

`isProcessing │ ┌────────────┼────────────┐ ↓ ↓ ↓ Approve Reject Merge disabled disabled disabled`

При Reject:

```
setIsProcessing(true);
```

React render:

`Approve 🔒 Reject 🔒 Merge 🔒 Save 🔒`

Точно това искаме.

Не само:

> „не натискай Reject два пъти“

а:

> „докато се извършва terminal operation върху този PendingProduct, не започвай друга конфликтна операция“.

Представи си:

`Reject #15 ───────────► backend преди да приключи: Approve #15 ──────────► backend`

Backend пак трябва да се защити, но няма никаква UX причина frontend да позволява това.

---

## 4. И да — всеки потребител има собствен React state

Това също си го разбрал правилно.

Admin A има браузър:

`Browser A React app isProcessing = true`

Admin B:

`Browser B React app isProcessing = false`

State-ът **не е глобален за всички потребители**.

Дори един и същи admin да отвори две tabs:

`TAB 1 TAB 2 React instance React instance isProcessing = true isProcessing = false`

Това са две отделни frontend инстанции.

Затова isProcessing не решава проблема:

> Admin A и Admin B едновременно обработват Product #15.

За това някой ден ще трябва backend механизъм като claim/locking/versioning, ако имаме няколко admin-а.

Така че можем да разделим защитите много ясно:

`isProcessing ↓ пази конкретната frontend инстанция от случайни конкурентни действия Backend status validation ↓ пази business state-а независимо от кой клиент идват requests`

---

# 5. Да го реализираме

Започни с най-малката стъпка.

До другите state-ове добави:

```
const [isProcessing, setIsProcessing] = useState(false);
```

Засега **не променяй handler-ите**.

След това избери например Approve бутона и го направи:

```
<button
    type="button"
    onClick={handleApprove}
    disabled={isProcessing}
>
    Approve
</button>
```

Тук няма нова магия:

`isProcessing = false ↓ disabled={false} ↓ бутонът работи isProcessing = true ↓ disabled={true} ↓ бутонът е заключен`

След като го направиш, следващата стъпка ще бъде по-интересната: **къде точно слагаме setIsProcessing(true) и setIsProcessing(false)**, така че да работи едновременно при success и при backend error.

И тогава естествено ще влезем в try / catch / finally, вместо просто да ти дам готов код.



кога може да се ползва {} за да изпълни js? в момента го ползвам в return

В React/JSX {} вътре в return се използва, когато искаш да вмъкнеш **JavaScript expression** в JSX.

Пример:

```
function App() {
    const name = "Ivan";

    return (
        <h1>Hello {name}</h1>
    );
}
```

Тук:

```
{name}
```

казва:

> „Тук спри JSX и оцени този JavaScript expression.“

Резултатът ще е:

```
<h1>Hello Ivan</h1>
```

Можеш вътре да имаш не само променлива, а почти всеки expression:

```
<p>{2 + 3}</p>
```

→ 5

```
<p>{user.name}</p>
```

```
<p>{isAdmin ? "Admin" : "User"}</p>
```

```
{showInput && <input />}
```

```
{products.map(product => (
    <div>{product.name}</div>
))}
```

Но има важно ограничение: вътре в JSX {} очакваш **expression**, тоест нещо, което произвежда стойност.

Затова това не става:

```
return (
    <div>
        {
            if (isAdmin) {   // ❌
                ...
            }
        }
    </div>
);
```

Защото if е statement, не expression.

Вместо това:

```
{isAdmin && <AdminPanel />}
```

или:

```
{isAdmin ? <AdminPanel /> : <UserPanel />}
```

Ако искаш нормален if, го пишеш преди return:

```
function App() {
    let message;

    if (isAdmin) {
        message = "Admin";
    } else {
        message = "User";
    }

    return (
        <div>{message}</div>
    );
}
```

И тук внимавай да не смесиш различните значения на {}.

В JavaScript:

```
const user = {
    name: "Ivan"
};
```

{} създава object.

При:

```
if (true) {
    console.log("hi");
}
```

{} е code block.

А в JSX:

```
<h1>{name}</h1>
```

{} означава:

> „вмъкни JavaScript тук“.

Най-кратко:

`JSX текст: <h1>Hello</h1> JS вътре в JSX: <h1>{name}</h1> JavaScript преди return: if (...) { ... } const x = ... function ...`

Тоест {} в JSX не е нещо специфично само за return; ползва се **навсякъде, където пишеш JSX и искаш вътре да вмъкнеш JavaScript expression**.



кога може да се ползва {} за да изпълни js? в момента го ползвам в return

В React/JSX {} вътре в return се използва, когато искаш да вмъкнеш **JavaScript expression** в JSX.

Пример:

```
function App() {
    const name = "Ivan";

    return (
        <h1>Hello {name}</h1>
    );
}
```

Тук:

```
{name}
```

казва:

> „Тук спри JSX и оцени този JavaScript expression.“

Резултатът ще е:

```
<h1>Hello Ivan</h1>
```

Можеш вътре да имаш не само променлива, а почти всеки expression:

```
<p>{2 + 3}</p>
```

→ 5

```
<p>{user.name}</p>
```

```
<p>{isAdmin ? "Admin" : "User"}</p>
```

```
{showInput && <input />}
```

```
{products.map(product => (
    <div>{product.name}</div>
))}
```

Но има важно ограничение: вътре в JSX {} очакваш **expression**, тоест нещо, което произвежда стойност.

Затова това не става:

```
return (
    <div>
        {
            if (isAdmin) {   // ❌
                ...
            }
        }
    </div>
);
```

Защото if е statement, не expression.

Вместо това:

```
{isAdmin && <AdminPanel />}
```

или:

```
{isAdmin ? <AdminPanel /> : <UserPanel />}
```

Ако искаш нормален if, го пишеш преди return:

```
function App() {
    let message;

    if (isAdmin) {
        message = "Admin";
    } else {
        message = "User";
    }

    return (
        <div>{message}</div>
    );
}
```

И тук внимавай да не смесиш различните значения на {}.

В JavaScript:

```
const user = {
    name: "Ivan"
};
```

{} създава object.

При:

```
if (true) {
    console.log("hi");
}
```

{} е code block.

А в JSX:

```
<h1>{name}</h1>
```

{} означава:

> „вмъкни JavaScript тук“.

Най-кратко:

`JSX текст: <h1>Hello</h1> JS вътре в JSX: <h1>{name}</h1> JavaScript преди return: if (...) { ... } const x = ... function ...`

Тоест {} в JSX не е нещо специфично само за return; ползва се **навсякъде, където пишеш JSX и искаш вътре да вмъкнеш JavaScript expression**.

как се означава една функция, че може дас върне такъв тип формат? Какво е JSX? HTML с възжмоност за ползване на js?

Да — най-просто можеш да мислиш JSX като:

> **синтаксис, който прилича на HTML, но се пише вътре в JavaScript и позволява да вмъкваш JavaScript expressions.**

Например:

```
function Welcome() {
    const name = "Ivan";

    return (
        <h1>Hello {name}</h1>
    );
}
```

Това **не е истински HTML файл**. Това е JSX.

### Как функцията „знае“, че може да върне JSX?

В JavaScript не я означаваш със специален тип:

```
function Welcome() {
    return <h1>Hello</h1>;
}
```

Няма нещо като в Java:

```
JSX Welcome() // няма такъв синтаксис
```

Функцията просто връща резултата от JSX expression-а.

Причината е, че този код:

```
<h1>Hello</h1>
```

преди да бъде изпълнен в браузъра, се преобразува от build tool-а до JavaScript.

Концептуално нещо като:

```
React.createElement("h1", null, "Hello");
```

Тоест JSX:

```
<h1>Hello</h1>
```

в крайна сметка е **JavaScript expression, който създава описание на UI елемент**.

Затова функцията може просто да го return-не.

### JSX не е точно „HTML + JavaScript“

Това е добра начална представа:

`HTML-подобен синтаксис + възможност за JS с {}`

но по-точно:

> JSX е extension на JavaScript syntax, предназначен да описва UI като дървовидна структура.

Например:

```
const element = (
    <div>
        <h1>Hello</h1>
        <button>Click</button>
    </div>
);
```

Това е JavaScript expression и затова можеш да го запазиш дори в променлива:

```
const element = <h1>Hello</h1>;
```

Това е важният момент — JSX не съществува само в return.

Можеш:

```
const title = <h1>Hello</h1>;

function App() {
    return title;
}
```

### А {}?

В JSX:

```
<h1>{name}</h1>
```

казва:

> „На това място оцени JavaScript expression.“

Например:

```
<p>{2 + 3}</p>
```

```
<p>{user.name}</p>
```

```
{isAdmin && <AdminPanel />}
```

### А React component?

Когато имаш:

```
function App() {
    return (
        <div>
            <h1>Hello</h1>
        </div>
    );
}
```

React третира тази функция като **component**, защото я използваш като:

```
<App />
```

Можеш да си представиш:

`App() ↓ връща JSX ↓ React получава описание на UI ↓ React решава какво трябва да има в DOM ↓ браузърът показва елементите`

Това е съществената разлика:

`HTML → директно описва документа JSX → JavaScript синтаксис, с който описваш какъв UI искаш → React използва това описание → обновява DOM`

Затова JSX прилича много на HTML, но има разлики. Например:

```
className="button"
```

вместо HTML:

```
class="button"
```

и можеш да правиш:

```
<input disabled={isLoading} />
```

което вече комбинира UI структура с JavaScript стойност.

Най-добрата ти ментална картина засега е:

`JavaScript ↓ React библиотека ↓ JSX = удобен синтаксис за описване на UI ↓ React го превръща/обработва ↓ DOM ↓ видимата страница`

И функцията не се маркира специално като „връщаща JSX“ — JSX просто е стойността/expression-ът, който тя връща.

това се преобразува до html?

Не директно.

JSX първо се преобразува до обикновен JavaScript, а React после използва този JavaScript, за да създава и обновява DOM елементи.

Пример:

```
const element = <h1>Hello</h1>;
```

концептуално става нещо като:

```
const element = React.createElement("h1", null, "Hello");
```

После React използва това описание и създава реален DOM елемент:

```
<h1>Hello</h1>
```

Тоест flow-ът е:

`JSX ↓ JavaScript ↓ React ↓ DOM ↓ видим HTML елемент в браузъра`

Затова е по-точно да кажеш:

> JSX не се компилира просто до HTML. То се преобразува до JavaScript, а React чрез него управлява DOM-а.

И когато state се промени, React не генерира наново цял HTML файл. То сравнява какъв UI трябва да има и обновява само нужните части от DOM-а.




