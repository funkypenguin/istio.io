---
title: Посібник по стилю
description: Пояснює стилістичні конвенції, що використовуються в документації Istio.
weight: 11
aliases:
    - /uk/docs/welcome/contribute/style-guide.html
    - /uk/docs/reference/contribute/style-guide.html
    - /uk/about/contribute/style-guide
    - /latest/uk/about/contribute/style-guide
keywords: [contribute]
owner: istio/wg-docs-maintainers
test: n/a
---

Всі матеріали, прийняті в документацію Istio, повинні бути **чіткими** і
**зрозумілими**. Стандарт, який ми використовуємо для цього визначення, базується на [ключових моментах](https://developers.google.com/style/highlights) та [загальних принципах](https://developers.google.com/style/tone) стилістичного посібника Google для розробників. Перегляньте нашу [сторінку огляду](/docs/releases/contribute/review) для деталей щодо того, як перевіряються та приймаються ваші внески в контент.

Тут ви можете знайти всі сценарії, де проєкт Istio вирішив дотримуватися іншої практики та базового стилістичного керівництва з прикладами,специфічними для Istio.

## Використовуйте стиль речення для заголовків {#use-sentence-case-for-headings}

Використовуйте стиль речення для заголовків у вашому документі. Великими літерами слід писати лише перше слово заголовка, за винятком власних імен або абревіатур.

| Зробіть                 | Не робіть
|-------------------------|----------
| Налаштування обмежень швидкості | Налаштування Обмежень Швидкості
| Використання Envoy для ingress | Використання envoy для ingress
| Використання HTTPS     | Використання https

## Використовуйте стиль заголовка для значення поля `title:` у front matter {#use-title-case-for-the-value-of-the-title-field-of-the-front-matter}

Текст для поля `title:` у front matter повинен використовувати стиль заголовка. Великими літерами слід писати першу літеру кожного слова, за винятком сполучників і прийменників.

## Використовуйте теперішній час {#use-present-tense}

| Зробіть                     | Не робіть
|-----------------------------|----------
| Ця команда запускає проксі. | Ця команда запустить проксі.

Виключення: Використовуйте майбутній або минулий час, якщо це необхідно для передачі правильного значення. Це виключення надзвичайно рідкісне і його варто уникати.

## Використовуйте активний голос {#use-active-voice}

| Зробіть                                    | Не робіть
|--------------------------------------------|----------
| Ви можете дослідити API за допомогою оглядача. | API може бути досліджене за допомогою оглядача.
| YAML файл визначає кількість реплік.      | Кількість реплік визначена в YAML файлі.

## Використовуйте просту і зрозумілу мову {#use-simple-and-direct-language}

Використовуйте просту і зрозумілу мову. Уникайте зайвих фраз, таких як "будь ласка."

| Зробіть                             | Не робіть
|-------------------------------------|----------
| Щоб створити `ReplicaSet`, ...      | Для того щоб створити `ReplicaSet`, ...
| Перегляньте конфігураційний файл.  | Будь ласка, перегляньте конфігураційний файл.
| Перегляньте Pods.                   | За допомогою наступної команди, ми переглянемо Pods.

## Звертайтеся до читача на "ви" {#address-the-reader-as-you}

| Зробіть                              | Не робіть
|--------------------------------------|----------
| Ви можете створити `Deployment`, ... | Ми створимо `Deployment`, ...
| У попередньому виводі ви можете побачити... | У попередньому виводі ми можемо побачити...

## Створюйте корисні посилання {#create-useful-links}

Є хороші та погані гіперпосилання. Загальноприйнята практика використовувати посилання *тут* або *натисніть тут* є прикладом поганих гіперпосилань. Перегляньте [цю чудову статтю](https://medium.com/@heyoka/dont-use-click-here-f32f445d1021), що пояснює, що робить гіперпосилання корисним, і намагайтеся дотримуватися цих керівництв при створенні або перегляді контенту сайту.

## Уникайте використання "ми" {#avoid-using-we}

Використання "ми" у реченні може бути заплутаним, оскільки читач може не знати, чи є він частиною "ми", яке ви описуєте.

| Зробіть                                 | Не робіть
|-----------------------------------------|----------
| Версія 1.4 включає ...                  | У версії 1.4, ми додали ...
| Istio надає нову функцію для ...        | Ми надаємо нову функцію ...
| Ця сторінка навчає вас використовувати pods. | На цій сторінці ми будемо дізнаватись про pods.

## Уникайте жаргону та ідіом {#avoid-jargon-and-idioms}

Деякі читачі говорять англійською як другою мовою. Уникайте жаргону та ідіом, щоб полегшити їх розуміння.

| Зробіть                            | Не робіть
|------------------------------------|----------
|Internally, ...       | Under the hood, ...
|Create a new cluster. | Turn up a new cluster.
|Initially, ...        | Out of the box, ...

## Уникайте заяв про майбутнє {#avoid-statements-about-the-future}

Уникайте обіцянок або натяків про майбутнє. Якщо вам потрібно говорити про функцію в розробці, додайте стандартний текст під front matter, що ідентифікує інформацію відповідно.

### Уникайте заяв, які швидко стануть застарілими {#avoid-statements-that-will-soon-be-out-of-date}

Уникайте використання слів, що швидко стають застарілими, таких як "в даний час" і "новий". Функція, яка є новою сьогодні, не залишиться новою надовго.

| Зробіть                                | Не робіть
|----------------------------------------|----------
| У версії 1.4, ...                     | У поточній версії, ...
| Функція Federation надає ...          | Нова функція Federation надає ...
