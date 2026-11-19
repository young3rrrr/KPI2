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
* У циклі loop для кожного товару DealService звертається до ProductRepository.findById(product_id), отримує дані Product і перевіряє що is_active = true.
* Для активного товару створюється запис DealItem, де в actual_price фіксується поточна ціна Product.base_price.
* DealService передає список позицій у PricingService.calculateTotal(items) та отримує загальну суму total_amount = 5000.
* DealService зберігає позиції через DealRepository.saveDealItems(deal_id, items) та оновлює угоду через DealRepository.updateDeal(deal_id, status='contract', total_amount=5000).
* DealService викликає EmailService.sendContractNotification(client_id, deal_id) для сповіщення клієнта, після чого контролер повертає менеджеру 200 OK.

2. SC-02: Альтернативний, успішний (Застосування персональної знижки).
* Менеджер подає запит у DealController на застосування знижки 10% для угоди в статусі negotiation на суму 10 000 грн (клієнт має статус VIP).
* DealController передає виклик у DealService.applyDiscount(deal_id, discount_percent).
* DealService отримує угоду через DealRepository.findById(deal_id).
* В умовному блоці alt система перевіряє ліміт знижки (discount_percent <= 15).
* Якщо ліміт в нормі, DealService викликає PricingService.calculateDiscountedTotal(total_amount, discount_percent) і отримує нову суму 9000 грн.
* DealService оновлює суму угоди через DealRepository.updateDealTotal(deal_id, 9000) та повертає успішний результат менеджеру.
* У гілці else (якщо знижка більше 15%) система відхиляє зміну і повертає помилку.

3. SC-03: Негативний (Порожня угода або неактивний товар).
* Менеджер намагається перевести угоду зі статусу new у contract без жодного товару або додає товар, у якого Product.is_active = false.
* DealController викликає DealService.confirmDealContract(deal_id, items).
* DealService перевіряє список позицій та звертається до ProductRepository.findById(product_id).
* В умовному блоці alt/else при порожньому списку або неактивному товарі DealService повертає помилку 400 Bad Request у DealController.
* Запис у DealRepository не відбувається, угода залишається в статусі new, а менеджер бачить повідомлення про помилку.

# 4. Критерії прийняття для Sequence діаграм (Acceptance Criteria)

1. **AC-01 (Окремі діаграми під кожен сценарій)**:
    * Для кожного сценарію (SC-01, SC-02, SC-03) має бути побудована окрема діаграма послідовності у форматі Mermaid (sequenceDiagram) в теці lab3/model/.
2. **AC-02 (Комбіновані фрагменти та повернення)**:
    * У кожній діаграмі має бути мінімум один комбінований фрагмент (loop, alt/else, opt), а кожен синхронний виклик має містити зворотнє return повідомлення.
3. **AC-03 (Смуги активації)**:
    * Для контролера, сервісів та репозиторіїв обовʼязково використовуються смуги активації (activate / deactivate), які починаються до умовного фрагмента і завершуються після нього. Актору SalesManager смуга активації не ставиться.
4. **AC-04 (Узгодженість із моделлю даних Лаб 1)**:
    * Всі виклики та параметри повинні використовувати точні назви сутностей і полів з першої лабораторної (Deal, DealItem, Product, deal_id, product_id, quantity, actual_price, base_price, is_active, total_amount, status).
