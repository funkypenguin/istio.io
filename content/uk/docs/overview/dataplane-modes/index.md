---
title: Sidecar чи Ambient?
description: Дізнайтеся про два режими даних Istio та про те, який з них варто використовувати.
weight: 30
keywords: [sidecar, ambient]
owner: istio/wg-docs-maintainers-english
test: n/a
---

Сервісна мережа Istio логічно поділена на панель даних та панель управління.

{{< gloss "панель даних">}}Панель даних{{< /gloss >}} — це набір проксі-серверів, які контролюють усю мережеву комунікацію між мікросервісами. Вони також збирають і звітують про телеметрію всього трафіку мережі.

{{< gloss "панель управління">}}Панель управління{{< /gloss >}} керує та налаштовує проксі-сервери на рівні даних.

Istio підтримує два основних {{< gloss "режим панелі даних" >}}режими панелі даних{{< /gloss >}}:

* **Режим sidecar**, який розгортає проксі-сервер Envoy разом з кожним Podʼом, який ви запускаєте у вашому кластері, або працює поряд з сервісами, що працюють на віртуальних машинах.
* **Режим ambient**, який використовує проксі-сервер на рівні 4 для кожного вузла та, за потреби, проксі-сервер Envoy на рівні 7 для функцій на рівні 7.

Ви можете вибрати певні простори імен або навантаження для кожного режиму.

## Режим Sidecar {#sidecar-mode}

Istio була побудована на патерні sidecar з першого релізу у 2017 році. Режим sidecar добре зрозумілий і ретельно перевірений, але має витрати на ресурси та операційні накладні витрати.

* Кожне застосування, яке ви розгортаєте, має проксі-сервер Envoy {{< gloss "інʼєкція" >}}вставлений{{< /gloss >}} як sidecar.
* Всі проксі-сервери можуть обробляти як рівень 4, так і рівень 7.

## Режим Ambient {#ambient-mode}

Запущений у 2022 році, режим ambient був створений для усунення недоліків, про які повідомляли користувачі режиму sidecar. В Istio 1.22, він готовий до використання в операційних середовищах для випадків використання в одному кластері.

* Увесь трафік пропускається через проксі-сервер на рівні 4.
* Застосування можуть вибрати маршрутизацію через проксі-сервер Envoy для отримання функцій рівня 7.

## Вибір між Sidecar і Ambient {#choosing-between-sidecar-and-ambient}

Користувачі часто розгортають мережу для забезпечення рівня нульової довіри як перший крок, а потім вибірково активують можливості L7 за потреби. Ambient-мережа дозволяє цим користувачам уникнути витрат на обробку L7, коли це не потрібно.

<table>
  <thead>
    <tr>
      <td style="border-width: 0px"></td>
      <th><strong>Sidecar</strong></th>
      <th><strong>Ambient</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Управління трафіком</th>
      <td>Повний набір функцій Istio</td>
      <td>Повний набір функцій Istio (потребує використання waypoint)</td>
    </tr>
    <tr>
      <th>Безпека</th>
      <td>Повний набір функцій Istio</td>
      <td>Повний набір функцій Istio: шифрування та авторизація L4 в режимі ambient. Потребує waypoint для авторизації L7.</td>
    </tr>
    <tr>
      <th>Спостережуваність</th>
      <td>Повний набір функцій Istio</td>
      <td>Повний набір функцій Istio: телеметрія L4 в режимі ambient; спостережуваність L7 при використанні waypoint</td>
    </tr>
    <tr>
      <th>Розширюваність</th>
      <td>Повний набір функцій Istio</td>
      <td>За допомогою <a href="/docs/ambient/usage/extend-waypoint-wasm">втулків WebAssembly</a> (вимагає використання waypoint)<br> API EnvoyFilter не підтримується.</td>
    </tr>
    <tr>
      <th>Додавання навантажень до мережі</th>
      <td>Встановіть мітку на простір імен і перезапустіть усі Podʼи, щоб додати sidecar</td>
      <td>Встановіть мітку на простір імен — перезапуск Podʼів не потрібен</td>
    </tr>
    <tr>
      <th>Поетапне розгортання</th>
      <td>Бінарний: sidecar доданий або ні</td>
      <td>Поступове: L4 завжди ввімкнено, L7 може бути додано за допомогою конфігурації</td>
    </tr>
    <tr>
      <th>Управління життєвим циклом</th>
      <td>Проксі-сервери управляються розробником застосунків</td>
      <td>Адміністратор платформи</td>
    </tr>
    <tr>
      <th>Використання ресурсів</th>
      <td>Неекономно; ресурси ЦП і памʼяті повинні бути виділені для найгіршого випадку використання кожного окремого Podʼа</td>
      <td>Проксі-сервери waypoint можуть автоматично масштабуватися як будь-яке інше розгортання Kubernetes.<br>Завантаження з великою кількістю реплік може використовувати один waypoint, на відміну від кожного з них, що має свій sidecar.
      </td>
    </tr>
    <tr>
      <th>Середня вартість ресурсів</th>
      <td>Велика</td>
      <td>Маленька</td>
    </tr>
    <tr>
      <th>Середня затримка (p90/p99)</th>
      <td>0.63мс-0.88мс</td>
      <td>Ambient: 0.16мс-0.20мс<br />Waypoint: 0.40мс-0.50мс</td>
    </tr>
    <tr>
      <th>Кроки обробки L7</th>
      <td>2 (source і destination sidecar)</td>
      <td>1 (destination waypoint)</td>
    </tr>
    <tr>
      <th>Конфігурація в масштабі</th>
      <td>Потребує <a href="/docs/ops/configuration/mesh/configuration-scoping/">конфігурації області кожного sidecar</a>, щоб зменшити конфігурацію</td>
      <td>Працює без спеціальної конфігурації</td>
    </tr>
    <tr>
      <th>Підтримка протоколів "server-first"</th>
      <td><a href="/docs/ops/deployment/application-requirements/#server-first-protocols">Потребує конфігурації</a></td>
      <td>Так</td>
    </tr>
    <tr>
      <th>Підтримка завдань (Jobs) Kubernetes</th>
      <td>Ускладнена тривалим життям sidecar</td>
      <td>Прозора</td>
    </tr>
    <tr>
      <th>Модель безпеки</th>
      <td>Найсильніша: кожен робочий процес має свої власні ключі</td>
      <td>Сильна: кожен агент вузла має лише ключі для навантажень на цьому вузлі</td>
    </tr>
    <tr>
      <th>Скомпрометований Pod застосунку<br>дає доступ до ключів мережі</th>
      <td>Так</td>
      <td>Ні</td>
    </tr>
    <tr>
      <th>Підтримка</th>
      <td>Стабільна, включаючи багатокластерні</td>
      <td>Стабільна, тільки одиночний кластер</td>
    </tr>
    <tr>
      <th>Платформи, що підтримуються</th>
      <td>Kubernetes (будь-який CNI)<br />Віртуальні машини</td>
      <td>Kubernetes (будь-який CNI)</td>
    </tr>
  </tbody>
</table>

## Функції рівня 4 проти рівня 7 {#layer-4-vs-layer-7-features}

Навантаження для обробки протоколів на рівні 7 суттєво вищі, ніж для обробки мережевих пакетів на рівні 4. Для конкретного сервісу, якщо ваші вимоги можуть бути виконані на L4, сервісна мережа може бути реалізована за суттєво меншу вартість.

### Безпека {#security}

<table>
  <thead>
    <tr>
      <td style="border-width: 0px" width="20%"></td>
      <th width="40%">L4</th>
      <th width="40%">L7</th>
    </tr>
   </thead>
   <tbody>
    <tr>
      <th>Шифрування</th>
      <td>Увесь трафік між Podʼами шифрується за допомогою {{< gloss "Взаємна TLS автентифікація" >}}mTLS{{< /gloss >}}.</td>
      <td>Н/Д&mdash;службова ідентичність в Istio базується на TLS.</td>
    </tr>
    <tr>
      <th>Автентифікація між службами</th>
      <td>{{< gloss >}}SPIFFE{{< /gloss >}}, через сертифікати mTLS. Istio видає короткоживучий X.509 сертифікат, який кодує ідентичність службового облікового запису Podʼа.</td>
      <td>Н/Д&mdash;службова ідентичність в Istio базується на TLS.</td>
    </tr>
    <tr>
      <th>Авторизація між службами</th>
      <td>Мережева авторизація, плюс політика на основі ідентичності, наприклад:
        <ul>
          <li>A може приймати вхідні виклики лише від "10.2.0.0/16";</li>
          <li>A може викликати B.</li>
        </ul>
      </td>
      <td>Повна політика, наприклад:
        <ul>
          <li>A може GET /foo на B тільки з дійсними обліковими даними кінцевого користувача, які містять READ область.</li>
        </ul>
      </td>
    </tr>
    <tr>
      <th>Автентифікація кінцевого користувача</th>
      <td>Н/Д&mdash;не можна застосувати налаштування для кожного користувача.</td>
      <td>Локальна автентифікація JWT, підтримка віддаленої автентифікації через OAuth та OIDC потоки.</td>
    </tr>
    <tr>
      <th>Авторизація кінцевого користувача</th>
      <td>Н/Д&mdash;див. вище.</td>
      <td>Політики між службами можуть бути розширені для вимоги <a href="/docs/reference/config/security/conditions/">облікових даних кінцевого користувача з певними областями, видавцями, принципалами, аудиторіями тощо</a><br />Повний доступ користувача до ресурсу може бути реалізований за допомогою зовнішньої авторизації, що дозволяє політику на кожний запит з рішеннями з зовнішньої служби, наприклад, OPA.</td>
    </tr>
  </tbody>
</table>

### Спостережуваність {#observability}

<table>
  <thead>
    <tr>
      <td style="border-width: 0px" width="20%"></td>
      <th width="40%">L4</th>
      <th width="40%">L7</th>
    </tr>
   </thead>
   <tbody>
    <tr>
      <th>Логування</th>
      <td>Основна мережева інформація: мережевий 5-кортеж, байти надіслані/отримані тощо. <a href="https://www.envoyproxy.io/docs/envoy/latest/configuration/observability/access_log/usage#command-operators">Див. документацію Envoy</a>.</td>
      <td><a href="https://www.envoyproxy.io/docs/envoy/latest/configuration/observability/access_log/usage#command-operators">Повні метадані запиту</a>, окрім основної мережевої інформації.</td>
    </tr>
    <tr>
      <th>Трейсинг</th>
      <td>На даний момент ні; можливий в майбутньому з HBONE.</td>
      <td>Envoy бере участь у розподіленому трейсі. <a href="/docs/tasks/observability/distributed-tracing/overview/">Див. огляд Istio по трейсу</a>.</td>
    </tr>
    <tr>
      <th>Метрики</th>
      <td>TCP тільки (байти надіслані/отримані, кількість пакетів тощо).</td>
      <td>Метрики L7 RED: кількість запитів, кількість помилок, тривалість запиту (затримка).</td>
    </tr>
  </tbody>
</table>

### Управління трафіком {#traffic-management}

<table>
  <thead>
    <tr>
      <td style="border-width: 0px" width="20%"></td>
      <th width="40%">L4</th>
      <th width="40%">L7</th>
    </tr>
   </thead>
   <tbody>
    <tr>
      <th>Балансування навантаження</th>
      <td>Тільки на рівні зʼєднання. <a href="/docs/tasks/traffic-management/tcp-traffic-shifting/">Див. завдання зміщення TCP трафіку</a>.</td>
      <td>На запит, що дозволяє, наприклад, розгортання canary, трафік gRPC тощо. <a href="/docs/tasks/traffic-management/traffic-shifting/">Див. завдання зміщення HTTP трафіку</a>.</td>
    </tr>
    <tr>
      <th>Запобіжник (Circuit breaking)</th>
      <td><a href="/docs/reference/config/networking/destination-rule/#ConnectionPoolSettings-TCPSettings">TCP тільки</a>.</td>
      <td><a href="/docs/reference/config/networking/destination-rule/#ConnectionPoolSettings-HTTPSettings">HTTP налаштування</a> на додачу до TCP.</td>
    </tr>
    <tr>
      <th>Виявлення аномалій</th>
      <td>При встановленні/збої зʼєднання.</td>
      <td>При успіху/збої запиту.</td>
    </tr>
    <tr>
      <th>Обмеження швидкості</th>
      <td><a href="https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/network_filters/rate_limit_filter#config-network-filters-rate-limit">Обмеження швидкості на даних зʼєднання L4 тільки, при встановленні зʼєднання</a>, з глобальними та локальними опціями обмеження швидкості.</td>
      <td><a href="https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/rate_limit_filter#config-http-filters-rate-limit">Обмеження швидкості на метаданих запиту L7</a>, на кожний запит.</td>
    </tr>
    <tr>
      <th>Тайм-аути</th>
      <td>Тільки встановлення зʼєднання (продовження зʼєднання конфігуроване через налаштування розриву зʼєднання).</td>
      <td>На кожний запит.</td>
    </tr>
    <tr>
      <th>Повторні спроби</th>
      <td>Повторна спроба встановлення зʼєднання</td>
      <td>Повторна спроба кожного запиту.</td>
    </tr>
    <tr>
      <th>Інʼєкція збоїв</th>
      <td>Н/Д&mdash;інʼєкція збоїв не може бути налаштована на TCP зʼєднаннях.</td>
      <td>Повні помилки рівня застосунка та зʼєднання (<a href="/docs/tasks/traffic-management/fault-injection/">тайм-аути, затримки, конкретні коди відповідей</a>).</td>
    </tr>
    <tr>
      <th>Моніторинг трафіку</th>
      <td>Н/Д&mdash;тільки HTTP</td>
      <td><a href="/docs/tasks/traffic-management/mirroring/">Моніторинг запитів на кілька бекендів за відсотком</a>.</td>
    </tr>
  </tbody>
</table>

## Функції, що не підтримуються {#unsupported-features}

Наступні функції доступні в режимі sidecar, але ще не реалізовані в режимі ambient:

* Взаємодія з sidecar-to-waypoint
* Мультикластерні установки
* Підтримка мультимереж
* Підтримка ВМ
