# 1. Обраний прецедент та контекст

1. **UC-01: Вести комерційну угоду(Manage Deal)** - Основний прецедент із Лабораторної 2.
    * Хто робить: Менеджер з продажу (SalesManager).
    * Суть: додавання товарних позицій (UC-05), застосування персональної знижки для VIP клієнтів (UC-06), перерахунок суми угоди та переведення її в статус contract.

# 2. Учасники взаємодії

1. **Менеджер з продажу(SalesManager)** - Основний актор, який працює з карткою угоди через інтерфейс.
2. **Контролер угод(DealController)** - Приймає вхідні HTTP запити від клієнта та повертає відповідь.
3. **Сервіс угод(DealService)** - Основний компонент бізнес логіки, керує статусами угоди (Deal) та координує роботу інших сервісів.
4. **Сервіс розрахунку цін(PricingService)** - Відповідає за підрахунок вартості позицій та розрахунок знижки.
5. **Репозиторій товарів(ProductRepository)** - Доступ до каталогу товарів (Product) для перевірки ціни та активності.
6. **Репозиторій угод(DealRepository)** - Збереження та читання даних угоди (Deal) і товарних позицій (DealItem).
7. **Поштовий сервіс(EmailService)** - Зовнішній сервіс для відправки повідомлення клієнту про сформований договір.

# 3. Кроки сценаріїв взаємодії.

1. SC-01: Основний, успішний (Формування позицій та переведення в договір).
* Менеджер передає в DealController запит на додавання товарів (2 шт. по 1500 грн та 1 шт. по 2000 грн) до відкритої угоди в статусі negotiation і переведення її в contract.
* DealController викликає метод DealService.confirmDealContract(deal_id, items).
* На початку DealService читає угоду через DealRepository.findById(deal_id) для перевірки що статус угоди не won і не lost.
* У циклі loop для кожного товару DealService звертається до ProductRepository.findById(product_id), отримує дані Product і перевіряє що is_active = true.
* Для активного товару створюється запис DealItem, де в actual_price фіксується поточна ціна Product.base_price.
* DealService передає список позицій у PricingService.calculateTotal(items) та отримує загальну суму total_amount = 5000.
* DealService зберігає позиції через DealRepository.saveDealItems(deal_id, items) та оновлює угоду через DealRepository.updateDeal(deal_id, status='contract', total_amount=5000).
* У блоці opt DealService надсилає асинхронний неблокуючий сигнал (-)) у EmailService.sendContractNotification(client_id, deal_id) без очікування відповіді, після чого контролер повертає менеджеру 200 OK.

2. SC-02: Альтернативний, успішний (Застосування персональної знижки).
* Менеджер подає запит у DealController на застосування знижки 10% для угоди в статусі negotiation на суму 10 000 грн (клієнт має статус VIP).
* DealController передає виклик у DealService.applyDiscount(deal_id, discount_percent).
* DealService отримує угоду через DealRepository.findById(deal_id) та завантажує її позиції через DealRepository.findItemsByDealId(deal_id).
* В умовному блоці alt система перевіряє умову VIP угоди та ліміт знижки (total_amount >= 10000 і discount_percent <= 15).
* Якщо ліміт в нормі, DealService викликає PricingService.applyDiscountToItems(items, discount_percent), який пропорційно зменшує ціну actual_price кожної позиції DealItem і повертає нову загальну суму 9000 грн.
* DealService зберігає оновлені ціни позицій через DealRepository.updateDealItems(deal_id, items) та оновлює суму угоди через DealRepository.updateDealTotal(deal_id, 9000), після чого повертає успішний результат менеджеру.
* У гілці else (якщо знижка більше 15%) DealService повертає доменну помилку ValidationError, а DealController повертає 400 Bad Request.

3. SC-03: Негативний (Порожня угода або неактивний товар).
* Менеджер намагається перевести угоду зі статусу new у contract без жодного товару або додає товар, у якого Product.is_active = false.
* DealController викликає DealService.confirmDealContract(deal_id, items).
* Якщо список позицій порожній, у блоці break сервіс DealService одразу перериває процес і повертає доменну помилку EmptyDealItemsError, яку DealController перетворює на 400 Bad Request.
* Якщо позиції є, у циклі loop перевіряється кожен товар через ProductRepository.findById(product_id); при виявленні Product.is_active = false спрацьовує блок break, який зупиняє цикл і повертає з DealService доменну помилку InactiveProductError (а контролер віддає 400 Bad Request).
* Запис у DealRepository не відбувається, угода залишається в статусі new.

# 4. Критерії прийняття для Sequence діаграм (Acceptance Criteria)

1. **AC-01 (Окремі діаграми під кожен сценарій)**:
    * Для кожного сценарію (SC-01, SC-02, SC-03) має бути побудована окрема діаграма послідовності у форматі Mermaid (sequenceDiagram) в теці lab3/model/.
2. **AC-02 (Комбіновані фрагменти та типи викликів)**:
    * У кожній діаграмі має бути мінімум один комбінований фрагмент (loop, alt/else, opt, break). Кожен синхронний виклик має містити зворотнє return повідомлення, а виклик EmailService у SC-01 має бути асинхронним сигналом (-)) без блокування потоку.
3. **AC-03 (Смуги активації)**:
    * Для контролера, сервісів та репозиторіїв обовʼязково використовуються смуги активації (activate / deactivate), які починаються до умовного фрагмента і завершуються після нього. Актору SalesManager смуга активації не ставиться.
4. **AC-04 (Узгодженість із моделлю даних Лаб 1 та збереження суми)**:
    * Всі виклики повинні використовувати точні назви сутностей і полів з першої лабораторної (Deal, DealItem, Product, deal_id, product_id, quantity, actual_price, base_price, is_active, total_amount, status). При застосуванні знижки у SC-02 обовʼязково оновлюються ціни DealItem.actual_price разом із Deal.total_amount.
5. **AC-05 (Розділення рівнів абстракції та ранній вихід break)**:
    * Сервісний шар (DealService) має повертати доменні помилки без HTTP кодів (HTTP статус 400 Bad Request повертає тільки DealController), а для переривання при порожній угоді чи неактивному товарі в SC-03 використовується оператор break.
