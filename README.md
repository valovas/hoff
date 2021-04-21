# Документация по верстке массовых писем

## Добавление письма в exponea и начальная настройка
![image](/images/1.png)

Для создания ноды, перетащите блок Email в правую часть окна.

Название ноды берется из названия сценария вместе с датой и названием сегмента.

![image](/images/2.png)

Далее переходим на вкладку **Settings**

**Frequency policy** (глобальная политика частоты отправки писем) сейчас используется `smart_policy_test`

**Consent category** (тип подписки рассылки) бывает следующими
* Специальные предложения
  ```
  {% set customer_consents = catalogs.Consents.item_by_id(customer.email) %}
  {% if customer_consents['discount'] == "reject" %}
  {% abort: "Consent discount is rejected" %}
  {%endif%}
  ```
* Специальные предложения + Лучшее за неделю (Дайджест)
  ```
  {% set customer_consents = catalogs.Consents.item_by_id(customer.email) %}
  {% if (customer_consents['best_week'] == "reject") and
  (customer_consents['discount'] == "reject") %}
  {% abort: "Consents best_week and discount are rejected" %}
  {%endif%}
  ```
* Спецпредложения+Бонус (участники ПЛ)
  ```
  {% set customer_consents = catalogs.Consents.item_by_id(customer.email) %}
  {% if (customer_consents['loyalty'] == "reject") and (customer_consents['discount']
  == "reject") %}
  {% abort: "Consents loyalty and discount are rejected" %}
  {%endif%}
  ```
Если это не указано в задаче, то можно уточнить в блоке условий

![image](/images/3.png)

В блоке **Link transformation** переключаем в `Manual` и очищаем поля ЮТМ меток, так как они будут генирироваться в письме на основе
```
{% set utm_source = 'exponea' %}
{% set utm_medium = 'email' %}
{% set utm_campaign = 'rassylkaaktsii' %}
{% set utm_name = 'bestprice' %}
{% set utm_date = time | from_timestamp('%d%m%Y') %}
{% set utm_segment = 'detskaya' %}
{% set utm_region = catalogs.Region_names.item_by_id(customer.region_price) if customer.region_price else catalogs.Region_names.item_by_id('711') %}
{% set utm_test_name = '' %}
{% set utm_test_group = '' %}
{% set utm = 'utm_source='~utm_source~'&''utm_medium='~utm_medium~'&''utm_campaign='~utm_campaign~'_'~utm_name~'_'~utm_date~'_'~utm_segment~'_'~utm_region.utm~'_'~utm_test_name~'_'~utm_test_group %}
```
* utm_name = `'Название сценария'`
* utm_segment = `'Название сегмента'`
* utm_test_group = `'Если АБ тест'`

К каждой ссылке прописывается `?{{utm}}` или `&{{utm}}` если ссылка содержит символ `?`

## Вывод товара в цикле

### Товарная 1 в ряд
![image](/images/5.png)
```html
<table width="100%" cellpadding="0" cellspacing="0" border="0" style="border-collapse:collapse;">
  <tr>
    <td align="center" valign="top" style="padding:0;">
      <table align="center" cellpadding="0" cellspacing="0" border="0" style="border-collapse:collapse;">
        <tr>
          <td width="580" align="center" valign="top" style="padding:0;font-size:0;">
            <!--[if mso]><table align="center" cellpadding="0" cellspacing="0" border="0" style="border-collapse:collapse;"><tr><![endif]-->{% for product in products %}{% set productMin = productsMin[products.index(product)]%}{% if customer.region_price in productMin.available_region %}{% set oldPrice=productMin['price'~cp~'old']|int %}{% set newPrice=productMin['price'~cp]|int %}{% set discount=((1-newPrice/oldPrice)*100)|round(0,'floor')|int if oldPrice !=0 %}{% set count = count + 1 %}{%- if count < 9 -%}<!--[if mso]><td width="290" align="center" valign="top"><![endif]-->
            <div style="display:inline-block;vertical-align:top;max-width:290px;">
              <table align="center" cellpadding="0" cellspacing="0" border="0" style="border-collapse:collapse;">
                <tr>
                  <td width="290" align="center" valign="top" style="padding:0 0 40px 0;">
                    <div style="padding:0 10px 0 10px;">
                      <table align="center" cellpadding="0" cellspacing="0" border="0" style="border-collapse:collapse;">
                        <tr>
                          <td width="270" align="center" valign="top" style="padding:0 0 17px 0;"><a href="{{ product.url }}&{{utm}}" target="_blank" style="border:none;text-decoration:none;"><img src="{{ product.image }}" width="270" border="0" alt="{{ product.title }}" style="display:block;font-size:12px;width:100%;max-width:270px;" /></a></td>
                        </tr>
                        <tr>
                          <td width="270" align="left" valign="top" style="padding:0 0 5px 0;font-family:'Arial Black',Arial,Helvetica,Roboto,sans-serif;text-align:left;font-size:13px;font-weight:bold;color:#000000;line-height:18px;">
                            <a href="{{ product.url }}&{{utm}}" style="text-decoration:none;color:#000000;font-weight:bold;border:none;" target="_blank">{{ product.title | truncate(60) }}</a>
                          </td>
                        </tr>
                        {% if oldPrice != 0 %}{% if oldPrice > newPrice %}<tr>
                          <td width="270" align="left" valign="middle" style="padding:0;font-family:'Arial Black',Arial,Helvetica,Roboto,sans-serif;text-align:left;font-size:28px;font-weight:bold;color:#000000;line-height:34px;"><a href="{{ product.url }}&{{utm}}" style="text-decoration:none;color:#000000;font-weight:bold;border:none;" target="_blank">{{format_price(newPrice)}}&nbsp;р &nbsp;<span style="display:inline-block;padding:0 2px 0 0;font-size:17px;background-color:#ffdc00;border-radius:17px;">&nbsp;&ndash;{{discount}}%&nbsp;</span></a></td>
                        </tr>
                        <tr>
                          <td width="270" align="left" valign="middle" style="padding:0 0 10px 0;font-family:Arial,Helvetica,Roboto,sans-serif;text-align:left;font-size:13px;font-weight:normal;color:#000000;line-height:16px;text-decoration:line-through;"><a href="{{ product.url }}&{{utm}}" style="text-decoration:none;color:#000000;border:none;" target="_blank">{{format_price(oldPrice)}}&nbsp;р</a></td>
                        </tr>{%endif%}{%else%}<tr>
                          <td width="270" align="left" valign="middle" style="padding:0 0 10px 0;font-family:'Arial Black',Arial,Helvetica,Roboto,sans-serif;text-align:left;font-size:28px;font-weight:bold;color:#000000;line-height:34px;"><a href="{{ product.url }}&{{utm}}" style="text-decoration:none;color:#000000;font-weight:bold;border:none;" target="_blank">{{format_price(newPrice)}}&nbsp;р</a></td>
                        </tr>{%endif%}
                      </table>
                    </div>
                  </td>
                </tr>
              </table>
            </div><!--[if mso]></td><![endif]-->{% if (count is even) and (count != 8) %}<!--[if mso]></tr><tr><![endif]-->{%endif%}{%endif%}{%endif%}{%endfor%}<!--[if mso]></tr></table><![endif]-->
          </td>
        </tr>
      </table>
    </td>
  </tr>
</table>
```

### Товарная 2 в ряд
![image](/images/4.png)
```html
{% for item in goods %}{% set product = catalogs.Items.item_by_id(item) %}{% set productMin = catalogs.ItemsMin.item_by_id(item) %}{% if customer.region_price in productMin.available_region %}{% set oldPrice=productMin['price'~cp~'old']|int %}{% set newPrice=productMin['price'~cp]|int %}{% set discount=((1-newPrice/oldPrice)*100)|round(0,'floor')|int if oldPrice !=0 %}{% set count = count + 1 %}{%- if count < 7 -%}<table width="100%" cellpadding="0" cellspacing="0" border="0" style="border-collapse:collapse;">
  <tr>
    <td align="center" valign="middle" style="padding:0 0 20px 0;">
      <table align="center" cellpadding="0" cellspacing="0" border="0" style="border-collapse:collapse;">
        <tr>
          <td {% if count is even %}dir="rtl" {%endif%}width="560" align="center" valign="middle" style="padding:0;font-size:0;">
            <!--[if mso]><table align="center" cellpadding="0" cellspacing="0" border="0"><tr><td width="280" valign="middle" align="center"><![endif]-->
            <div style="display:inline-block;vertical-align:middle;max-width:280px;">
              <table align="center" cellpadding="0" cellspacing="0" border="0" style="border-collapse:collapse;">
                <tr>
                  <td dir="ltr" width="280" align="center" valign="top" style="padding:0 0 20px 0;">
                    <a href="{{ product.url }}&{{utm}}" target="_blank" style="text-decoration:none;border:none;"><img src="{{ product.image }}" width="280" border="0" alt="{{ product.title }}" style="display:block;font-size:12px;width:100%;max-width:280px;" /></a>
                  </td>
                </tr>
              </table>
            </div><!--[if mso]></td><td width="280" valign="middle" align="center"><![endif]--><div style="display:inline-block;vertical-align:middle;max-width:280px;">
              <table align="center" cellpadding="0" cellspacing="0" border="0" style="border-collapse:collapse;">
                <tr>
                  <td dir="ltr" width="280" align="center" valign="top" style="padding:0 0 20px 0;">
                    <div style="padding: 0 20px 0 20px;">
                      <table align="center" cellpadding="0" cellspacing="0" border="0" style="border-collapse:collapse;">
                        <tr>
                          <td width="240" align="left" valign="top" style="padding:0 0 5px 0;font-family:'Arial Black',Arial,Helvetica,Roboto,sans-serif;text-align:left;font-size:13px;font-weight:bold;color:#000000;line-height:18px;">
                            <a href="{{ product.url }}&{{utm}}" style="text-decoration:none;color:#000000;font-weight:bold;border:none;" target="_blank">{{ product.title }}</a>
                          </td>
                        </tr>
                        {% if oldPrice != 0 %}{% if oldPrice > newPrice %}<tr>
                          <td width="240" align="left" valign="middle" style="padding:0;font-family:'Arial Black',Arial,Helvetica,Roboto,sans-serif;text-align:left;font-size:28px;font-weight:bold;color:#000000;line-height:34px;">{{format_price(newPrice)}}&nbsp;р &nbsp;<span style="display:inline-block;padding:0 2px 0 0;font-size:17px;color:#000000;background-color:#ffdc00;border-radius:17px;">&nbsp;&ndash;{{discount}}%&nbsp;</span></td>
                        </tr>
                        <tr>
                          <td width="240" align="left" valign="middle" style="padding:0 0 10px 0;font-family:Arial,Helvetica,Roboto,sans-serif;text-align:left;font-size:13px;font-weight:normal;color:#000000;line-height:16px;text-decoration:line-through;">{{format_price(oldPrice)}}&nbsp;р</td>
                        </tr>{%endif%}{%else%}<tr>
                          <td width="240" align="left" valign="middle" style="padding:0 0 10px 0;font-family:'Arial Black',Arial,Helvetica,Roboto,sans-serif;text-align:left;font-size:28px;font-weight:bold;color:#000000;line-height:34px;">{{format_price(newPrice)}}&nbsp;р</td>
                        </tr>{%endif%}
                        <tr>
                          <td width="240" align="left" valign="top" style="padding:0;">
                            <table cellpadding="0" cellspacing="0" border="0" style="border-collapse:collapse;">
                              <tr>
                                <td width="180" valign="middle" align="center" style="padding:0;">
                                  <!--[if mso]><v:roundrect xmlns:v="urn:schemas-microsoft-com:vml" xmlns:w="urn:schemas-microsoft-com:office:word" href="{{ product.url }}&{{utm}}" style="height:36px;v-text-anchor:middle;width:180px;" arcsize="50%" stroke="f" fillcolor="#e60000">
                                    <w:anchorlock/>
                                      <center><![endif]-->
                                      <a href="{{ product.url }}&{{utm}}" target="_blank" style="background-color:#e60000;border-radius:18px;color:#ffffff;display:inline-block;font-family:'Arial Black',Arial,Helvetica,Roboto,sans-serif;font-size:14px;font-weight:bold;line-height:36px;text-align:center;text-decoration:none;border:none;width:180px;-webkit-text-size-adjust:none;">Смотреть</a>
                                      <!--[if mso]></center>
                                    </v:roundrect><![endif]-->
                                </td>
                              </tr>
                            </table>
                          </td>
                        </tr>
                      </table>
                    </div>
                  </td>
                </tr>
              </table>
            </div><!--[if mso]></td></tr></table><![endif]-->
          </td>
        </tr>
      </table>
    </td>
  </tr>
</table>{%endif%}{%endif%}{%endfor%}
```

* `{% if (count is even) and (count != 8) %}<!--[if mso]></tr><tr><![endif]-->{%endif%}`  
 Добавляет код для Outlook после четного товара и не последнего
* Или `{% if (outer_loop.index is even) and (not outer_loop.last) %}<!--[if mso]></tr><tr><![endif]-->{%endif%}`  
 если используется `{% set outer_loop = loop %}`
* `{%- if count < 9 -%}` Ограничевает количество выводимых товаров
* `{% if count is even %}dir="rtl" {%endif%}`  
 Переключает отображение товара в шахматном порядке 
* Или `{% if outer_loop.index is even %}dir="rtl" {%endif%}`  
 если используется `{% set outer_loop = loop %}`

Задаем список товаров  
`{% set goods = ['80322557', '80273537', '80332354', '80273539', '80359591', '80282159', '80310000', '80273555', '80373708', '80352851'] %}`  
`{% set products = catalogs.Items.items_by_id(goods) %}`  
`{% set productsMin = catalogs.ItemsMin.items_by_id(goods) %}`  
`{% set count = 0 %}`

## Вывод товаров через рекомендации

```
{% set items1 = recommendations('5f5311a7445d25f828f7271b', 20, fill_with_random=False,categoryNames=['Товары для уборки'],
catalog_filter = [
  ...
]
) | shuffle %}
```
`5f5311a7445d25f828f7271b`  Ид рекомендации  
`20` Количество товаров в запросе  
`categoryNames=['Товары для уборки']` Название категории на сайте  
Можно запрашивать сразу несколько категорий в одном выводе рекомендации.  
`categoryNames=['Офисные кресла', 'Компьютерные столы', 'Столовые сервизы']`  
`shuffle` Перемешивает товары  
`fill_with_random=False` условие, которое не позволяет рекомендации выводить товары рандомно, если вдруг не получается вывести требуемое количество товаров с учетом заданных фильтров.  


[Список категорий](https://docs.google.com/spreadsheets/d/1Bw4J2Jw3CnOpX7MyhpLinMI53rnvYTDM6frQ0RX76hY/edit#gid=138076730)

**Обязательный фильтр**  
```
{
"property": "available_region",
"constraint": {
"type": "string",
"operator": "contains",
"operands": [{"type": "constant", "value": customer.region_price }]
}
}
```
Он позволяет получать товары только доступные для покупки (имеют кнопку Купить в карточке)  

**Дополнительные фильтры**  
```
{
"property": "title",
"constraint": {
"type": "string",
"operator": "contains",
"operands": [{"type": "constant", "value": 'прихожая' }]
 }
}
```  
Будет выводить товары, у которых в названии присутствует слово `прихожая`  
Или наоборот не выводить товары если `contains` заменить на `does not contain`  

```
{
"property": "url",
"constraint": {
"type": "string",
"operator": "contains",
"operands": [{"type": "constant", "value": 'shkafi_raspashnie' }]
}
}
```
Выведет товары у которых в адресе есть `shkafi_raspashnie`

```
{
"property": "price",
"constraint": {
"type": "number",
"operator": "greater than",
"operands": [{"type": "constant", "value": 5000 }]
}
}
```
Выведет товары у которых цена больше `5000`

```
{
"property": customer.region_price,
"constraint": {
"type": "string",
"operator": "equals",
"operands": [{"type": "constant", "value": 'available' }]
}
}
```
Выведет товары если доступны в этом регионе 

### Виды рекомендаций 

**по просмотру**  
[category web](https://app-cis.exponea.com/p/hoff1/campaigns/recommendations/5ddcdcc257988b9af5501462?action=test)  
[category parent2 web](https://app-cis.exponea.com/p/hoff1/campaigns/recommendations/5ddce03349e68153a4739718?action=evaluate)  
[category parent3 web](https://app-cis.exponea.com/p/hoff1/campaigns/recommendations/5ddce0c5ccba0d84f38cea0f?action=evaluate) 

**по просмотру со скидками**  
[category web](https://app-cis.exponea.com/p/hoff1/campaigns/recommendations/5e1e47a1e5f2ec74c79e14cb?action=evaluate)  
[category parent2 web](https://app-cis.exponea.com/p/hoff1/campaigns/recommendations/5e1dd21b93df29ffec5f5fdc?action=evaluate)  
[category parent3 web](https://app-cis.exponea.com/p/hoff1/campaigns/recommendations/5e1dd3bda4905954ef5d99e1?action=evaluate) 

**по покупкам**  
[category web](https://app-cis.exponea.com/p/hoff1/campaigns/recommendations/5e84bca7be86ac1c5deea715?action=evaluate)  
[category parent2 web](https://app-cis.exponea.com/p/hoff1/campaigns/recommendations/5e84629916fe7fe472481219?action=evaluate)  
[category parent3 web](https://app-cis.exponea.com/p/hoff1/campaigns/recommendations/5e332b82459087f9a94a9b80?action=evaluate) 

**новая рекомендация**  
[all categories visit item](https://app-cis.exponea.com/p/hoff1/campaigns/recommendations/5f5311a7445d25f828f7271b?action=evaluate)  

## Использование блоков

Блоки удобно использовать при наличии одинаковых блоков в письмах.  
Блоки находятся [тут](https://app-cis.exponea.com/p/hoff1/data/assets/email-block-templates)  
Для того чтобы открыть панель, нужно нажать на иконку Плюс в правом нижнем углу в редакторе кода  
В триггеры вставляем через "copy reference", чтобы при изменении блока он автоматически обновлялся и в письме.  
![image](/images/6.png)


