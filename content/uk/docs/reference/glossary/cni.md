---
title: CNI
test: n/a
---

[Container Network Interface (CNI)](https://www.cni.dev/) — це стандарт, який використовується Kubernetes для конфігурації мережі кластерів. Він реалізується за допомогою *втулків*, які поділяються на два типи:

* *втулки інтерфейсу*, які створюють мережевий інтерфейс, і надаються оператором кластера
* *ланцюгові втулки*, які можуть конфігурувати створений інтерфейс, і можуть надаватися програмним забезпеченням, встановленим у кластері

Istio працює з усіма реалізаціями CNI, які відповідають стандарту CNI, як у режимі sidecar, так і в ambient режимі.

Для конфігурації перенаправлення трафіку в mesh, Istio включає [вузловий агент CNI](/docs/setup/additional-setup/cni/). Цей агент встановлює ланцюговий CNI втулок, який виконується після всіх налаштованих втулків CNI інтерфейсу.

CNI вузловий агент є необовʼязковим для режиму {{< gloss >}}sidecar{{< /gloss >}} і обовʼязковим для режиму {{< gloss >}}ambient{{< /gloss >}}.
