
SDK - это библиотека, созданная компанией Xsolla, которая помогает реализовать функционал магазина на основе  Paystation API.

SDK была создана, чтобы: 
* упростить использование платежного решения Xsolla для разработчиков альтернативных платформ;
* снизить порог вхождения при использовании продукта на конкретной платформе.

# Оглавление 
* [Термины и сокращения](#термины-и-сокращения)
* [Цели SDK](#цели-sdk)
* [Функциональное назначение](#функциональное-назначение)
* [Взаимодействие с пользователем](#взаимодействие-с-пользователем)


# Термины и сокращения 

| Термин   |      Определение                                   |
|----------|:-------------:|
| ПО       |Программное обеспечение, подлежащее сопровождению | 
| SDK |Набор средств разработки, позволяющий специалистам по программному обеспечению создавать приложения для определённого пакета программ |
|API|Описание способов (набор классов, процедур, функций, структур или констант), которыми одна компьютерная программа может взаимодействовать с другой программой|
| Токен       |Специальная шифрованная строка, необходимая для взаимодействия с Xsolla | 
| Header |Верхняя часть окна магазина; включает в себя всё до пунктов навигационного меню включительно   |
| Footer |Нижняя часть окна магазина |
|Серверная интеграция|Интеграция и взаимодействие с Xsolla, происходящая посредством специально настроенного сервера|
| Упрощённая интеграция       |Интеграция и взаимодействие с Xsolla, при котором сервер игры отсутствует, все взаимодействия происходят через клиента | 
| JSON |Текстовый формат представления данных в нотации объекта JavaScript, предназначен для обмена данными   |
| Режим Сандбокс |Специальный режим, при котором пользователь может ознакомиться с возможностями приложения |


# Цели SDK 
Цели создания данной библиотеки:
- предоставить удобный инструмент для разработчиков, который позволит снизить трудозатраты по внедрению сервисов автоматизации решения компании Xsolla;
- улучшить качество опыта взаимодействия конечного пользователя. Инструмент:
  - становится понятным и доступным, 
  - требует меньше (или исключает) дополнительных переходов на сторонние окна.

# Функциональное назначение

Данная библиотека используется разработчиками для получения полного функционала электронного магазина (модули виртуальных валют, модули виртуальных товаров, подписоки, поддержки платёжных методов и тд.) при помощи одной функции.

Посредством взаимодействия с API реализуется создание магазина со всеми внутренними взаимодействиями, включающие в себя следующие экраны:  
- экран виртуальных товаров;
- экран виртуальной валюты;
- экран подписок;
- экраны выбора платёжных методов; 
- экран оплаты;
- экран статуса;
- экран ошибки.

# Взаимодействие с пользователем 
SDK позволяет реализовать два варианта взаимодействия пользователя с [Paystation API](https://secure.xsolla.com/paystation2/api), посредством **POST** или **GET** запросов:
1. [Серверная интеграции](#серверная-интеграции)
1. [Упрощённой интеграции](#упрощённой-интеграции)
 
### Серверная интеграции 

Авторизация пользователя происходит посредством передачи строки токена. 

### Упрощённой интеграции 

Авторизация пользователя происходит посредством передачи JSON. Программный код реализации смотреть в Приложение 1. 

```JSON
{
"user":{
"id":{
"value":0
},
"name":"",
"email":"",
"country":{
"value":"",
"allow_modify":false
}
},
"settings":{
"project_id":0,
"language":"",
"currency":"",
"mode":"sandbox",
"secretKey":""
}
}
```

 Для реализации упрощённой интеграции с JSON необходимо:
- подготовить специальный параметризированный объект, чтобы у разработчика не возникла необходимости самостоятельно генерировать его. 
- добавить возможность взаимодействия с xsolla API в сандбокс режиме. В поле url перед secure добавить “sandbox-”. Соответственно, пользователь должен иметь возможность определить, в каком режиме проводятся платежи.
# Ожидаемый результат выполнения услуги
Функция либо объект, выполненный заказчиком должен иметь следующий вид: 
 CreateShop (
			string, // token or json
			bool, // is Sandbox
			OkCallback, // возможность обработать успешный платёж
			ErrorCallbck // возможность обработать платёж с ошибкой
)
где OkCallback, ErrorCallbck могут быть реализованы любым удобным способом и не  обязательно должны передаваться как параметры.
После вызова данной функции управление процессом работы приложения со стороны разработчика-пользователя заканчивается и переходит под управление библиотеки. Контроль над работой приложения разработчик получает вновь лишь в момент успешного или неуспешного завершения платежа. 

Порядок запросов и переходов описан в разделе Общая схема работы, построение интерфейса и обработка запросов описаны в разделе Обработка запросов, описание дизайна находится в разделе Дизайн. 
Необходимо дописать два раздела (Дизайн и Обработка запросов)
# Схема работы 
## Общая схема работы
Общая схема взаимодействия (рис. 1) с Paystation API включает в себя 5 основных шагов:
1. инициализирующий запрос;
1. выбор покупки;
1. выбор платёжной системы;
1. проведение платежа;
1. статус покупки.

Часть из этих шагов (№2, №3) не являются обязательными и при необходимости могут быть пропущены. Вся информация, необходимая для определения схемы работы, может быть получена из первого шага (инициализирующего запроса). Далее рассмотрим каждый из этих шагов в отдельности, описание будет состоять из сопровождающего текста и диаграммам, наглядно иллюстрирующих процесс работы сервиса.
(рис 1. Общая схема работы с Paystation API) 
Рис.1 Общая схема работы с Paystation API
### Инициализирующий запрос
Инициализирующим запросом в данном случае является запрос Utils. В нём содержится вся необходимая информация (информация о пользователе, информация о проекте, информация о покупке, настройки и переводы). Если был получен корректный ответ от сервера, то в первую очередь будет просматриваться информация о покупке. С помощью неё необходимо определить передана ли покупка("purchase"). Если данное действие совершено, то необходимо проверить наличие товара ("virtual_currency", "virtual_items", "subscription" или "subscription") и наличие платёжной системы("payment_system"). Если в полученном ответе поле для товара не пустое, то шаг №2 пропускается (рис.1). Если в полученном ответе поле для платёжной системы не пустое, то шаг №3 пропускается (рис.1). Схема запроса представлена на рисунке 2. 
На данном этапе реализации начинается построение интерфейса (header и footer окна), так как для этого действия уже получены вся необходимая информация.

 
Рис.2 Шаг №1 «Инициализирующий запрос»

### Выбор покупки
Выбор покупки является вторым шагом, если в шаге первом (инициализирующий запрос) не было заполнено поле для товара ("virtual_currency", "virtual_items", "subscription" или "subscription"). На данном шаге необходимо сконструировать магазин, в котором пользователь сможет выбрать понравившийся ему товар и реализовать свою покупку. В первую очередь будет задействована информация, полученная из шага первого (инициализирующий запрос), а именно из поля объекта настроек ("settings"): "goods_at_first", "pricepoints_at_first", "subscriptions_at_first". Данное поле может принимать значения 0 или 1. В зависимости от того, что было получено, будут формироваться следующие пункты меню соответственно (рис.3): 
- товары;
- виртуальная валюта;
- подписки.
В зависимости от полученных данных необходимо загрузить одну из категорий и отобразить её для пользователя (рис.4). Та же логика работает и при переключении пунктов меню. Далее пользователь может выбрать какой-либо из товаров и ему предоставляется два способа оплаты покупки: за реальную валюту или за виртуальную валюту.
 
Рис. 3 Шаг №2 открытие магазина
 
Рис. 4 Шаг №2 разделы магазина
#### Покупка за реальную валюту
В случае покупки за реальную валюту будет произведен переход к проверке пользовательского баланса. Если денежных средств на балансе пользователя достаточно, оплата продолжается через Xsolla Balance, в противном случае произойдёт открытие списка платёжных систем.
#### Покупка за виртуальную валюту
При произведении покупок за виртуальную валюту схема работы отлична. Сперва выполняется запрос «Summary», в котором будут получены содержимое покупки пользователя и метка, которая говорит о возможности пропуска подтверждение от пользователя. Если данный запрос выполнен успешно, то происходит переход к следующему запросу «Proceed». Данный запрос определяет корректность производимой денежной операции. Если он выполняется успешно, то платёж продолжается через Xsolla Balance, в противном случае он выдаст ошибку. Если была получена ошибка или необходимо подтверждение от пользователя, то мы попадаем на экран подтверждения платежа, на котором пользователю показана его покупка, чекбокс с надписью: “Спрашивать ли подтверждение в следующий раз” и сама ошибка, если был совершен переход после запроса «Proceed». Подробную схему реализации покупок можно увидеть на рисунке 5.
 
Рис. 5 Шаг №3 реализация покупки товара


 
