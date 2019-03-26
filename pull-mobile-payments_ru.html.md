---
title: Mobile payments API 2.1 Мобильная коммерция

search: true

metatitle: Mobile payments API 2.1 Мобильная коммерция

metadescription: Mobile payments API Мобильная коммерция открывает доступ к операциям с балансом вашемо счета у сотового оператора из вашего приложения. Поддерживаются операции инициирования оплаты счета с мобильного телефона, а также проверки статуса выполнения операций.

language_tabs:
  - shell
  - php

services:
 - <a href='#'>Swagger</a>  |  <a href='#'>Qiwi Demo</a>


toc_footers:
 - <a href='/'>На главную</a>

includes:
  - pull-payments/notification_ru

---

# Платежи с баланса мобильного оператора {#introduction}

###### Последнее обновление: 2017-10-05 | [Редактировать на GitHub](https://github.com/QIWI-API/pull-payments-docs/blob/master/pull-mobile-payments_ru.html.md)

Mobile Payments API открывает доступ к операциям сплатежами из вашего сервиса. Поддерживаются следующие операции:

* Инициация платежа
* Проверка статуса оплаты счета
* Уведомления об обплате


## Процесс оплаты заказа {#steps}

Процесс оплаты выглядит следующим образом:

<img id='mobile' src="/images/mobile.png" class="image" />

1. Пользователь формирует заказ на сайте провайдера.
2. Мерчант выполняет запрос [Инициировать платеж](#invoice_rest) с параметрами авторизации.
3. Сразу после инициирования платежа пользователь получает СМС от своего мобильного оператора с информацией о счете и подтверждает оплату ответным СМС. Если пользователь отказался от оплаты, то счет автоматически отклоняется.

   ![Mobile SMS](/images/pull_mobile.jpg){:height="45%" width="45%"}

4. Если провайдер включил отправку [уведомлений на сервер провайдера](#notification_ru), то после проведения платежа система высылает уведомление на сервер провайдера об оплате данного счета. Уведомления содержат параметры авторизации, которые необходимо проверять на сервере провайдера.

   В отсутствие уведомлений провайдер может [запросить текущий статус платежа](#invoice-status).

5. После подтверждения оплаты счета провайдер исполняет заказ пользователя.

**Данное API можно использовать только после [регистрации и подключения](https://kassa.qiwi.com).**


## Авторизация {#auth_param}

Запросы мерчанта к Pull API авторизуются посредством HTTP basic-авторизации. Для авторизации используются [API ID и API password](#auth_param). Заголовок представляет собой параметр Authorization, значение которого представлено как: Basic Base64(API_ID:API_PASSWORD)


~~~shell
user@server:~$ curl "адрес сервера"
  --header "Authorization: Basic MjMyNDQxMjM6NDUzRmRnZDQ0Mw=="
~~~

<ul class="nestedList params">
    <li><h3>Авторизация и работа с формами</h3><span>Данные могут быть получены на сайте <a href="https://kassa.qiwi.com">kassa.qiwi.com</a></span>
    </li>
</ul>

### Служебные данные {#auth_param}

Параметр|Описание|Тип|Обяз.
 ---------|--------|---|------
 API_ID | Идентификатор для авторизации провайдера в API | Integer| +
 API_PASSWORD | Пароль для авторизации в API| String | +
 ID проекта | Числовой идентификатор провайдера (идентификатор проекта или PRV_ID) | Integer | +

<aside class="notice">
Получить служебные данные можно на партнерском сайте <a href='http://kassa.qiwi.com'>kassa.qiwi.com</a> в разделе "Протоколы - REST-протокол - Аутентификационные данные".

</aside>

## Инициация платежа и оплата {#invoice_rest}

Запрос инициирует новый платеж на указанный номер телефона. Тип запроса - HTTP PUT.

Для подтверждения оплаты, на указанный номер телефона будет отправлен SMS код  оператором сотовой связи пользователя.

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/BILL-1"
  -X PUT --header "Accept: text/json"
  --header "Authorization: Basic ***"
  -d 'user=tel%3A%2B79161111111&amount=10.00&ccy=RUB&comment=test&pay_source=mobile&lifetime=2016-09-25T15:00:00'


HTTP/1.1 200 OK
Content-Type: text/json
{
  "response": {
     "result_code": 0,
     "bill": {
        "bill_id": "BILL-1",
        "amount": "10.00",
        "ccy": "RUB",
        "status": "waiting",
        "error": 0,
        "user": "tel:+79161111111",
        "comment": "test"
     }
  }
}
~~~

<h3 class="request method">Запрос → PUT</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a></span></h3>
        <ul>
        <strong>В path params PUT-запроса используются два параметра счета:</strong>
             <li><strong>prv_id</strong> - числовой идентификатор провайдера (идентификатор проекта на партнерском сайте kassa.qiwi.com)</li>
             <li><strong>bill_id</strong> - уникальный идентификатор счета в системе провайдера.</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json - формат ответа JSON</li>
             <li>Content-type: application/x-www-form-urlencoded; charset=utf-8</li>
             <li>Authorization: Basic ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры передаются в теле запроса как url-encoded formdata</span>
    </li>
</ul>

Параметр|Описание|Тип|Обяз.
---------|--------|---|------
user | Идентификатор номера QIWI Wallet, на который выставляется счет (в международном формате), с префиксом `tel:` | String(20)|+
amount | Сумма, на которую выставляется счет. Округляется в меньшую сторону до двух десятичных знаков | Number(6.2)|+
ccy | Идентификатор валюты (Alpha-3 ISO 4217 код). Может использоваться любая валюта, предусмотренная договором с КИВИ | String(3)|+
comment | Комментарий к счету | String(255)|+
lifetime | Дата, до которой счет будет доступен для оплаты. Если счет не будет оплачен до этой даты, ему присваивается финальный статус и последующая оплата станет невозможна.<br> **Внимание! По истечении 28 суток от даты выставления счет автоматически будет переведен в финальный статус.**|dateTime|+
pay_source |`mobile` - оплата счета будет производиться с баланса мобильного телефона пользователя |String|+

<h3 class="request">Ответ ←</h3>

~~~shell

HTTP/1.1 200 OK
Content-Type: text/json;charset=utf-8
{
  "response": {
     "result_code": 0,
     "bill": {
        "bill_id": "BILL-1",
        "amount": "10.00",
        "originAmount": "10.00"
        "ccy": "RUB",
        "originCcy": "RUB",
        "status": "waiting",
        "error": 0,
        "user": "tel:+79031234567",
        "comment": "test"
     }
  }
}


HTTP/1.1 500
Content-Type: text/json;charset=utf-8
{
 "response": {
  "result_code": 150,
  "description": "Authorization failed"
  }
}
~~~


Формат ответа в исходном запросе:

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json - формат ответа JSON</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><a href="#parameters"><h3>Параметры</h3></a>
    </li>
</ul>

Параметр|Тип|Описание
--------|---|--------
result_code|Integer|[Код результата](#errors)
description|String|Описание ошибки. Передается в случае ошибки
bill|Object|Описание счета
bill.bill_id|String|Копия параметра `bill_id` из исходного запроса
bill.amount|String|Сумма счета. Округляется в меньшую сторону до двух десятичных знаков.
bill.originAmount|String|Сумма счета в исходной валюте счета (см. параметр `originCcy`). Округляется в меньшую сторону до двух десятичных знаков.
bill.ccy |String|Идентификатор валюты (Alpha-3 ISO 4217 код)
bill.originCcy|String|Идентификатор валюты выставленного счета (Alpha-3 ISO 4217 код)
bill.status  |String|Текущий [статус счета](#status)
bill.error |Integer|Константа, всегда `0`
bill.user|String|Копия параметра `user` из исходного запроса
bill.comment|String|Копия параметра `comment` из исходного запроса

~~~php

<?php
//Пример реализации запроса на PHP
//Идентификатор магазина из вкладки "Данные магазина"
$SHOP_ID = "21379721";
//API ID из вкладки "Данные магазина"
$REST_ID = "62573819";
//API пароль из вкладки "Данные магазина"
$PWD = "**********";
//ID счета
$BILL_ID = "99111-ABCD-1-2-1";
$PHONE = "79191234567";

$data = array(
    "user" => "tel:+" . $PHONE,
    "amount" => "1000.00",
    "ccy" => "RUB",
    "comment" => "Все очень хорошо",
    "lifetime" => "2015-01-30T15:35:00",
    "pay_source" => "mobile",
    "prv_name" => "Хороший магазин"
);

$ch = curl_init('https://api.qiwi.com/api/v2/prv/'.$SHOP_ID.'/bills/'.$BILL_ID);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'PUT');
curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($data));
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
curl_setopt($ch, CURLOPT_HTTPAUTH, CURLAUTH_BASIC);
curl_setopt($ch, CURLOPT_USERPWD, $REST_ID.":".$PWD);
curl_setopt($ch, CURLOPT_HTTPHEADER,array (
    "Accept: application/json"
));
$results = curl_exec ($ch) or die(curl_error($ch));
echo $results;
echo curl_error($ch);
curl_close ($ch);
?>
~~~

## Проверка статуса платежа {#invoice-status}

Запрос позволяет проверить текущий статус оплаты клиентом.

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl "https://api.qiwi.com/api/v2/prv/373712/bills/BILL-1"
  --header "Authorization: Basic ***"
  --header "Accept: text/json"

~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/api/v2/prv/<a>prv_id</a>/bills/<a>bill_id</a></span></h3>
        <ul>
        <strong>Параметры передаются в path params.</strong>
             <li><strong>prv_id</strong> - числовой идентификатор провайдера (идентификатор проекта на партнерском сайте kassa.qiwi.com)</li>
             <li><strong>bill_id</strong> - уникальный идентификатор счета в системе провайдера.</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Basic ***</li>
             <li>Accept: text/json или Accept: application/json - формат ответа JSON</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

~~~shell

HTTP/1.1 200 OK
Content-Type: text/json
{
  "response": {
     "result_code": 0,
     "bill": {
        "bill_id": "BILL-1",
        "amount": "10.00",
        "originAmount": "10.00"
        "ccy": "RUB",
        "originCcy": "RUB",
        "status": "paid",
        "error": 0,
        "user": "tel:+79031234567",
        "comment": "test"
     }
  }
}


HTTP/1.1 500
Content-Type: text/json
{
 "response": {
  "result_code": 150,
  "description": "Authorization failed"
  }
}
~~~

Формат ответа в исходном запросе:

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: text/json - формат ответа JSON</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3>
    </li>
</ul>

Параметр|Тип|Описание
--------|---|--------
result_code|Integer|[Код результата](#errors)
description|String|Описание ошибки. Передается в случае ошибки
bill|Object|Описание счета
bill.bill_id|String|Уникальный идентификатор счета в системе провайдера
bill.amount|String|Сумма счета, округленная до 2 или 3 знаков после запятой. Способ округления зависит от валюты.
bill.originAmount|String|Сумма счета в исходной валюте счета (см. параметр `originCcy`), округленная до 2 или 3 знаков после запятой. Способ округления зависит от валюты.
bill.ccy |String|Идентификатор валюты (Alpha-3 ISO 4217 код)
bill.originCcy|String|Идентификатор валюты выставленного счета (Alpha-3 ISO 4217 код)
bill.status  |String|Текущий [статус счета](#status)
bill.error   |Integer|Константа, `0`
bill.user|String|Идентификатор кошелька пользователя, которому выставлен счет (номер телефона в международном формате с префиксом `tel:`)
bill.comment|String|Комментарий к счету


## Статусы операций {#status}

### Статусы платежей {#status_bills}

Статус|Описание|Финальный
------|--------|---------
waiting | Платеж инициирован, ожидает оплаты| -
paid|оплачен|+
rejected|отклонен|+
unpaid|Ошибка при проведении оплаты. Не оплачен|+
expired |Время жизни платежа истекло. Не оплачен|+

## Список ошибок {#errors}

Код| Описание |[Fatal](#fatal)
---|----------|------
0|Успех |
5|Неверные данные в параметрах запроса|+
13|Сервер занят, повторите запрос позже|-
78|Недопустимая операция|+
150|Ошибка авторизации|+
152|Не подключен или отключен протокол|-
155|Данный идентификатор провайдера ([API ID](#auth_param)) заблокирован|+
210|Платеж не найден|+
215|Платеж с таким bill_id уже существует|+
241|Сумма слишком мала|+
300|Техническая ошибка|-
303|Неверный номер телефона|+
316|Попытка авторизации заблокированным провайдером|-
319|Нет прав на данную операцию|-
339|Ваш IP-адрес или массив адресов заблокирован|+
341|Обязательный параметр указан неверно или отсутствует в запросе|+
700|Превышен месячный лимит на операции|+
774|Счет клиента в системе QIWI временно заблокирован|-
1001|Запрещенная валюта для провайдера|+
1003|Не удалось получить курс конвертации для данной пары валют|-
1019|Не удалось определить сотового оператора для мобильной коммерции|+
1419|Нельзя изменить данные счета – он уже оплачивается или оплачен|+

<aside class="notice"><a name="fatal">Признак</a> означает, что при повторном запросе результат не изменится (ошибка не временная, проанализируйте данные запроса или обратитесь в Support).</aside>
