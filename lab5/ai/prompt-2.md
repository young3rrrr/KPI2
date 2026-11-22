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
