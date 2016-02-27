## Расширения

Здесь описаны расширения протокола, необязательные к реализации.

#### Количество сообщений в эхоконференциях

Предназначен для отслеживания изменений в эхе и для отсеивания лишнего трафика. Обычное целое число.

###### Метод
`GET /x/с/<параметры>`

###### Параметры
Единственный параметр метода - список, разделенный '/'.

###### Возвращаемое значение
Словарь "эха":"количество".

###### Пример
`GET /x/c/test.14/im.100`

```
test.14:221
im.100:1500
```

#### Push (пуш), обратная синхронизация
Предназначен для получения сервером (нодой) сообщений от другого доверенного авторизованного сервера. Может быть полезным для транзитных гейтов-посредников или серверов с неработающим фетчингом. Хоть пуш и является частью **/u/**, он остаётся необязательным расширением.

Проще говоря, push - это бандл наоборот. Если фетчинг скачивает бандлы, то push их проталкивает на другой узел. Сначала у push-ноды запрашивается список сообщений, которые уже есть в эхе, а затем через /u/push загружаются недостающие. 

###### Метод
`POST /u/push`

###### Параметры в теле POST запроса:
* **nauth** - ключ для авторизации на сервере (пароль)
* **upush** - бандл сообщений
* **echoarea** - эхоконференция, в которую помещать сообщения

###### Возвращаемое значение
Может отличаться, в зависимости от сервера, либо отсутствовать. PHP-нода возвращает `message saved: ok` для каждого сообщения в бандле в случае успешного принятия, либо `error: <название ошибки>` в случае неправильного формата сообщений или `error: no auth`, если неправильный пароль.

#### Список эхоконференций
Нужен для автоматического получения данных о наличии эх на станции. Может пригодиться для автоподписки на клиентах.

###### Метод
`GET /list.txt`

###### Возвращаемое значение
Словарь "эха":"количество сообщений":"описание эхи".

###### Пример
`GET /list.txt`
```
test.14:35:Тестирование и проверки
im.100:270:Болталка, общение на любые темы
```

#### Чёрный список
Номера тех сообщений, которые нода не принимает во время обмена и не показывает в эхоконференциях. Используется для чистки станции от спама/некорректных сообщений.

###### Метод
`GET /blacklist.txt`

###### Возвращаемое значение
Список номеров сообщений на каждой строке по одному.

#### Сокращённый индекс (расширенный /u/e)
Вариант схемы **/u/e** с указанием смещения и количества отдаваемых msgid. Полезно для экономии трафика, чтобы не скачивать абсолютно все сообщения с эхи, а только нужные.

###### Метод
`/u/e/эха/эха/эха/.../<смещение>:<лимит>`
<Смещение> и <лимит> - целые числа. Смещение может быть отрицательным. Если параметры указаны ошибочно, выдаёт эхи целиком.

###### Возвращаемое значение
Список сообщений из заданных эхоконференций, с максимумом <лимит>.

###### Пример

`GET /u/e/ii.test.14/test.15/1:2`

```
ii.test.14
msgid
msgid
test.15
msgid
msgid
```

`GET /u/e/ii.test.14/test.15/-5:5`

```
ii.test.14
msgid
msgid
msgid
msgid
msgid
test.15
msgid
msgid
msgid
msgid
msgid
```

#### Список схем

Нужен, чтобы узнать, какие дополнительные возможности поддерживает станция.

###### Метод

`GET /x/features`

###### Пример

`GET /x/features`

```
x/c
list.txt
blacklist.txt
```

#### Размещение файлов

Сисоп складывает разные файлы себе на ноду, а поинты могут их скачивать.

###### Метод
`POST /x/file`

Параметры в теле POST-запроса: `pauth` и `filename`. `pauth` - строка авторизации, `filename` - имя скачиваемого файла. Если `pauth` неверный (нет такого поинта), то выдаёт ошибку `error: no auth`. Если `filename` отсутствует, то выдаёт список файлов в формате "Имя":<размер в байтах>:Описание.

###### Примеры
`POST /x/file`
`pauth=12345`
```
myfile.txt:421:Интересная информация
cats.jpg:63253:Картинка с котиками
```

`POST /x/file`
`pauth=12345;filename=myfile.txt`

`<Содержимое файла>`