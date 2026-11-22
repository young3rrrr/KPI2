# Ітерація 2: Вторинна генерація Component моделі та усунення розбіжностей.

## Промпт
> Ознайомся з оновленим файлом специфікації lab5/spec.md, архітектурним рішенням lab5/adr/0005-layered-component-isolation.md та зауваженнями аудиту в lab5/ai/prompt-1.md. Згідно з новими уточненнями перегенеруй компонентну діаграму мовою PlantUML. Результат збережи поверх файлів в теці lab5/model/ та онови рендери щоб вони були достовірні до нової діаграми.

## Контекст
- Файл специфікації: `lab5/spec.md` (commit: `677c7fe`)
- Архітектурне рішення: `lab5/adr/0005-layered-component-isolation.md` (commit: `d02e03d`)
- Аудит Ітерації 1: `lab5/ai/prompt-1.md` (commit: `789c138`)

## Очікуваний результат
- Оновлено декларативний файл `lab5/model/component-diagram.puml`.
- Оновлено графічні рендери у `lab5/model/`.
- Повністю усунуто зауваження першої ітерації: прибрано прямі звʼязки DealController -> PricingService та PricingService -> DealRepository, додано CatalogService та Auth & RateLimit Middleware.

## Аудит результатів
- **Статус**: Прийнято повністю
    - **Хеш коміту згенерованих артефактів**: `ccc0532`
    - **Згенеровані файли**:
        - `lab5/model/component-diagram.puml`
        - `lab5/model/component-diagram.png`
        - `lab5/model/component-diagram.svg`
    - **Огляд результатів**:
        - **Позитивно**:
        1. Забезпечено сувору ізоляцію PricingService: він викликається виключно з боку DealService і не має прямих звʼязків із репозиторіями БД (повна відповідність Лабораторній 3).
        2. Усунено обхід сервісного шару: між CatalogController та ProductRepository додано компонент CatalogService з інтерфейсом ICatalogService.
        3. У вхідний шар Fastify додано Auth & RateLimit Middleware, що покриває вимоги NFR-02 (Rate Limiting > 80 RPS) та NFR-04 (JWT + RBAC).
        4. Усі критерії прийнятності `AC-01` – `AC-04` виконано на 100%.
