# Ітерація 2: Вторинна генерація Sequence моделей та усунення розбіжностей.

## Промпт
> Ознайомся з оновленим файлом специфікації lab3/spec.md, архітектурним рішенням lab3/adr/0003-pricing-service-and-async-email.md та зауваженнями аудиту в lab3/ai/prompt-1.md. Згідно з новими уточненнями перегенеруй три діаграми послідовності у форматі Mermaid. Результат збережи поверх файлів в теці lab3/model/ та онови рендери щоб вони були достовірні до нових діаграм.

## Контекст
- Файл специфікації: `lab3/spec.md` (commit: `8234c67`)
- Архітектурне рішення: `lab3/adr/0003-pricing-service-and-async-email.md` (commit: `c73cfcc`)
- Аудит Ітерації 1: `lab3/ai/prompt-1.md` (commit: `047b755`)

## Очікуваний результат
- Оновлено декларативні файли діаграм у `lab3/model/`.
- Оновлено графічні рендери у `lab3/model/`.
- Повністю усунуто зауваження першої ітерації: пропорційне оновлення DealItem.actual_price при знижці, доменні помилки сервісу з блоком break у SC-03 та асинхронний сигнал для EmailService у SC-01.

## Аудит результатів
- **Статус**: Прийнято повністю
    - **Хеш коміту згенерованих артефактів**: `5ac8749`
    - **Згенеровані файли**:
        - `lab3/model/sc01-happy-path.mmd`
        - `lab3/model/sc01-happy-path.png`
        - `lab3/model/sc01-happy-path.svg`
        - `lab3/model/sc02-vip-discount.mmd`
        - `lab3/model/sc02-vip-discount.png`
        - `lab3/model/sc02-vip-discount.svg`
        - `lab3/model/sc03-negative-validation.mmd`
        - `lab3/model/sc03-negative-validation.png`
        - `lab3/model/sc03-negative-validation.svg`
    - **Огляд результатів**:
        - **Позитивно**:
        1. У сценарії SC-02 перед оновленням загальної суми угоди завантажуються позиції DealItem та пропорційно перераховуються їхні ціни actual_price через PricingService.
        2. У сценарії SC-03 сервісний шар повертає доменні помилки (EmptyDealItemsError, InactiveProductError), а HTTP статус 400 Bad Request формує DealController. Ранній вихід при помилках оформлено через блок break.
        3. У сценарії SC-01 додано початкову перевірку статусу угоди, а виклик EmailService винесено в блок opt як асинхронний неблокуючий сигнал (-)).
        4. Усі критерії прийнятності `AC-01` – `AC-05` виконано на 100%.
