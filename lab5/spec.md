# 1. Внутрішні компоненти системи OpenCRM

1. **Клієнтський веб-інтерфейс(Web UI - React / Ant Design)** - Фронтенд частина системи для менеджерів з продажу (SalesManager) та адміністратора (Admin).
    * Призначення: відображення воронки угод (Kanban), карток лідів і клієнтів, каталогу товарів та списку активностей.
2. **Вхідний шар API(API Gateway & Controllers - Fastify)** - Шар обробки HTTP запитів по HTTPS (TLS 1.3).
    * Auth & RateLimit Middleware: перевірка JWT токенів (15 хв / 7 днів), розмежування ролей RBAC (SalesManager / Admin за NFR-04) та обмеження частоти запитів понад 80 RPS зі статусом HTTP 429 (NFR-02).
    * Контролери: DealController (маршрути угод і знижок), LeadClientController (робота з лідами та клієнтами), CatalogController (керування товарами для Admin), ActivityController (задачі менеджера).
3. **Сервісний шар бізнес-логіки(Business Services)** - Основне ядро правил системи.
    * DealService: управління життєвим циклом угоди (UC-01, UC-05), перевірка статусів та координація суміжних сервісів.
    * PricingService: ізольований обчислювальний сервіс для розрахунку вартості позицій DealItem та пропорційного перерахунку персональної знижки (UC-06) без прямого доступу до БД.
    * CatalogService: керування каталогом товарів (UC-03), зміна цін base_price та прапорця активності is_active.
    * LeadClientService: кваліфікація та конвертація ліда в клієнта 1:1 (UC-02).
    * ActivityService: створення та контроль виконання завдань менеджера (UC-04).
    * EmailService: внутрішній модуль формування сповіщень про договори.
4. **Шар доступу до даних(Data Access Layer - Drizzle ORM)** - Репозиторії для роботи з базою даних.
    * DealRepository (таблиці Deal та DealItem), ProductRepository (таблиця Product), ClientLeadRepository (таблиці Lead та Client), ActivityUserRepository (таблиці Activity та User).
5. **База даних(PostgreSQL 16+)** - Реляційне сховище даних для всіх 7 сутностей із Лабораторної 1.

# 2. Межі системи та зовнішні інтеграції

1. **Межа системи(OpenCRM System Boundary)** - Охоплює Web UI, Fastify API з Middleware безпеки, сервісний шар, репозиторії та локальну БД PostgreSQL у контейнері (до 512 МБ RAM за NFR-05).
2. **Зовнішній поштовий шлюз(External SMTP Gateway)** - Єдина зовнішня система за межами OpenCRM, через яку EmailService відправляє клієнтам листи з договорами. Інтеграції онлайн-еквайрингу, телефонії та курсів валют виключені згідно з розділом Out of Scope Лабораторної 2.

# 3. Інтерфейси та залежності між компонентами.

1. Взаємодія клієнта та API.
* Web UI звертається до вхідного шару Fastify через інтерфейс REST API (HTTPS / TLS 1.3), де запити спочатку проходять через Auth & RateLimit Middleware, а потім потрапляють у контролери.

2. Сувора ізоляція між контролерами та сервісами.
* Контролери звертаються виключно до відповідних сервісів бізнес-логіки через їхні provided інтерфейси: DealController викликає тільки IDealService, CatalogController викликає ICatalogService, LeadClientController викликає ILeadClientService, ActivityController викликає IActivityService.
* Жоден контролер не може звертатися напряму до репозиторіїв Data Access Layer або до PricingService.

3. Взаємодія всередині сервісного шару, репозиторіїв та зовнішньої пошти.
* DealService координує оформлення угоди (як на Sequence-діаграмах Лаб 3 та Activity-діаграмі Лаб 4): використовує IPricingService для фінансових розрахунків, IActivityService для створення контрольної задачі дзвінка та IEmailService для асинхронного сповіщення.
* PricingService працює як чистий доменний обчислювач і не має залежностей від репозиторіїв БД.
* До репозиторіїв звертаються лише сервіси володіння даними: DealService (IDealRepository, IProductRepository), CatalogService (IProductRepository), LeadClientService (IClientLeadRepository), ActivityService (IActivityUserRepository).
* EmailService передає листи до External SMTP Gateway через зовнішній інтерфейс ISMTP.

# 4. Критерії прийняття для компонентної діаграми (Acceptance Criteria)

1. **AC-01 (Нотація та валідність)**:
    * Архітектура системи має бути описана декларативним кодом PlantUML (component diagram) у файлі lab5/model/component-diagram.puml без помилок рендерингу.
2. **AC-02 (Повнота шарів, NFR та межі системи)**:
    * На діаграмі мають бути чітко виділені межі системи OpenCRM, всі 5 внутрішніх шарів (включно з Auth & RateLimit Middleware та CatalogService) і зовнішній поштовий шлюз поза межами системи.
3. **AC-03 (Provided/Required інтерфейси та сувора шарова ізоляція)**:
    * Між компонентами повинні бути показані provided та required інтерфейси (ball-and-socket). Контролери не повинні мати прямих звʼязків із репозиторіями чи PricingService, а PricingService не повинен мати прямого звʼязку з БД або DealRepository.
4. **AC-04 (Трасування до Лабораторних 1–4)**:
    * Всі компоненти та їхні звʼязки повинні відповідати 7 сутностям з Лабораторної 1, функціональним вимогам і NFR з Лабораторної 2, викликам Sequence-діаграм з Лабораторної 3 та крокам бізнес-процесу з Лабораторної 4.
