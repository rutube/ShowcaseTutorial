# Руководство по разработке витрин на основе API Rutube

## Общая информация

Витрины Rutube - способ организации и представления видео-контента, используемый на портале [rutube.ru](http://rutube.ru). Одним из примеров контента, представленного с помощью витрин, может служить главная страница портала. Используя API партнеры Rutube имеют возможность разработать подобный тип представления видео-роликов на своей площадке.

## Техническая реализация

Технически, витрины состоят из вкладок, каждая из которых может содержать один или несколько источников карточек, например, один или несколько тегов или несколько телешоу. Каждый источник, являясь отдельной сущностью внутри платформы Rutube, требует отдельного запроса к API для получения контента (списка видео-роликов). Количество источников на одной вкладке не ограничено, также нет ограничений по их смешиванию. Например, на одной вкладке могут быть представлены и теги, и телешоу, и ролики пользователей. Карточки для каждого источника подгружаются постранично и могут смешиваться любым способом перед отрисовкой на странице.

Набор вкладок и источников для каждой витрины задается редакторами Rutube.

Кроме того, каждая вкладка имеет поле для задания внешней ссылки, и если оно не пустое, есть возможность при переходе на вкладку не показывать карточки, а делать переадресацию на указанный адрес.

Работа витрин построена на основе нескольких API, запрашиваемых в определенной последовательности.

Ответы API могут быть получены в форматах JSON, JSONP и XML.

Примеры запросов:

```
rutube.ru/api/feeds/tnt?format=json
rutube.ru/api/feeds/tnt?format=jsonp&callback=foobar
rutube.ru/api/feeds/tnt?format=xml
```

А также в удобном для чтения виде:

```
rutube.ru/api/feeds/tnt?format=api
```

## API витрины

Работа с витриной начинается с обращения к ее API. С помощью ответа этого API выстраивается меню вкладок и становятся доступны адреса источников, из которых в дальнейшем будут получены карточки.

Шаблон запроса API витрины:

```
rutube.ru/api/feeds/slug
```

где `slug` - уникальный идентификатор витрины.

Пример запроса:

```
rutube.ru/api/feeds/tnt
```

Основные поля, содержащиеся в ответе:

`id` - [ _number_ ] id витрины;

`slug` - [ _string_ ] идентификатор витрины;

`name` - [ _string_ ] название витрины;

`meta_description` - [ _string_ ] описание витрины;

`page_url` - [ _string_ ] адрес витрины на rutube.ru;

`tabs` - [ _array_ ] массив вкладок витрины.

Ключевым полем является `tabs`, в котором перечислены вкладки витрины.

Поля, которые содержат элементы в массиве `tabs`:

`id` - [ _number_ ] id вкладки;

`name` - [ _string_ ] название вкладки;

`sort` - [ _string_ ] сортировка карточек;

`order_number` - [ _number_ ] порядковый номер вкладки;

`slug` - [ _string_ || _null_ ] уникальное название вкладки (для hash- или history-навигации);

`resources` - [ _array_ ] массив источников карточек;

`link` - [ _string_ || _null_ ] внешняя ссылка (опционально).

Располагая массивом `tabs` из ответа API витрины, можно построить меню.

Каждый из элементов `tabs`, в свою очередь, содержит массив `resources` с перечислением источников карточек.

Поля элементов в массиве `resources`:

`id` - [ _number_ ] id источника;

`object_id` - [ _string_ ] служебный id источника внутри платформы Rutube;

`content_type` - [ _object_ ] объект с информацией об источнике;

`url` - [ _string_ ] адрес для получения карточек.

Поле `url` является адресом, по которому необходимо запрашивать список роликов источника.

_Совет! Рекомендуется рассматривать запросы к источникам одной вкладки как deferred-объекты и объединять их с помощью утилиты $.when() из jQuery. Это обеспечит равномерность работы интерфейса при загрузке и отрисовке карточек._

Возможные типы источников (объект `content_type`):

`tv` - ТВ-шоу;

`tvlastepisode` - последний эпизод конкретного ТВ-шоу;

`person` - персона, например, эстрадный исполнитель. Не путать с пользователем Rutube;

`tag` - тег;

`cardgroup` - группа карточек (ссылки на другие источники). На [rutube.ru](http://rutube.ru/) выглядит как карточка с кнопкой “Подписаться” и возможностью перехода на страницу этого источника);

`userchannel` - пользователь Rutube;

`promogroup`, `promofeed` - источники, содержащие промо-элементы (не ролики).

## API источников

При запросе источника карточек в ответ приходят следующие поля:

next - [ _string_ || _null_ ] адрес для запроса следующей страницы;

previous - [ _string_ || _null_ ] адрес для запроса предыдущей страницы;

page - [ _number_ ] номер запрошенной страницы с карточками;

has_next - [ _boolean_ ] наличие следующей страницы;

results - [ _array_ ] массив карточек.

Значение массива `results` и есть список видео-роликов источника.

Также стоит обратить внимание на поле `thumbnail_url` - картинка-превью. С помощью параметра `size` можно выбрать размер изображения, например:

```
pic.rutube.ru/video/d4/52/d452e0d567ce6ca5035d712279d3b9bb.jpg?size=m
```

Доступно три размера: `l`(arge), `m`(edium), `s`(mall).

## Сортировка карточек

В витринах имеются гибкие возможности для задания сортировки карточек, и, соответственно, возможности для отображения карточек на странице в определенном порядке. Например, ролики сериала могут быть показаны от большей серии к меньшей, или наоборот.

Возможности сортировки можно разделить на две составляющие: задание сортировки конкретным источникам на вкладке, и сортировка результатов, полученных из нескольких источников перед отображением на странице.

#### Задание сортировки источнику

Каждый тип источника может принимать определенные значения сортировки.

| Тип источника | Название параметра для сортировки | Возможные значения |
|---------------|-----------------------------------|--------------------|
| tv            | sort                              | series_a, series_d, created_a, created_d, publication_a, publication_d |
| tvlastepisode |                                   |                    |
| person        |                                   |                    |
| tag           | sort                              | tagged_a, tagged_d, created_a, created_d, publication_a, publication_d |
| cardgroup     |                                   |                    |
| userchannel   | ordering                          | publication_ts, created_ts, views, id
                                                      _* Можно также задавать обратные значения этих параметров, например,
                                                      -created_ts, а также группировать их через запятую, например, -created_ts,-id._ |

Эти значения необходимо подставлять к `url` при запросе источника. Например, для источника типа ТВ-шоу:

```
rutube.ru/api/metainfo/tv/28/video?sort=series_d
```

#### Сортировка результатов перед выводом на страницу

Сортировка результатов, полученных из нескольких источников может быть реализована на усмотрение разработчика. Однако, есть возможность принять во внимание мнение редакции по поводу того, как лучше всего сортировать карточки на конкретной вкладке. Эта рекомендация присутствует всегда и указана в поле `sort` для каждой вкладки (в массиве `tabs` в ответе API витрины).

Возможные значения поля `sort`:

`created_date` - смешать ролики и вывести их по дате в порядке убывания;

`original` - смешать ролики и вывести их по возрастанию индекса в массиве ответа;

`random` - смешать ролики и вывести в случайном порядке.

## Дополнительные параметры для источников

Помимо сортировки, источники могут принимать несколько других параметров.

`origin__type` - фильтр роликов конкретной платформы. Возможные значения: `rtb`, `ytb`, `vim`, `now`.

На Rutube могут присутствовать ролики других платформ, и для гарантированной отдачи роликов именно Rutube желательно подставлять этот параметр при каждом запросе источника, например:

```
rutube.ru/api/metainfo/tv/28/video?origin__type=rtb
```

`season`, `episode` - комбинируемые параметры, которые могут принимать источники типа ТВ-шоу, например:

```
rutube.ru/api/metainfo/tv/28/video?season=1&episode=2
```

В в результате этого запроса будет получена вторая серия первого сезона сериала “Интерны”.