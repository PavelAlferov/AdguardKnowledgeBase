---
title: 'Как составлять свои фильтры'
taxonomy:
    category:
        - docs
visible: true
---

* [Введение](#Введение)
* [Базовые правила](#Базовые-правила)
  * [Синтаксис](#-1)
  * [Спецсимволы](#Спецсимволы-используемые-в-паттерне)
  * [Примеры](#Примеры)
  * [Модификаторы](#Модификаторы)
 	  * [Базовые модификаторы](#Базовые-модификаторы)
        * [$domain](#domain)
        * [$third-party](#third-party)
        * [$popup](#popup)
        * [$match-case](#match-case)
        * [$image](#image)
        * [$stylesheet](#stylesheet)
        * [$xmlhttrequest](#xmlhttprequest)
        * [$empty](#empty)
        * [$mp4](#mp4)
    * [Модификаторы правил-исключений](#Модификаторы-правил-исключений)
        * [$elemhide](#elemhide)
        * [$content](#content)
        * [$jsinject](#jsinject)
        * [$urlblock](#urlblock)
        * [$document](#document)
    * [Generic-правила](#generic-rules)
        * [$generichide](#generichide)
        * [$genericblock](#genericblock)
* [RegExp rules](#regexp-support)
* [Replace rules](#replace-rules)
* [Правила сокрытия элементов](#Правила-сокрытия-элемeнтов)
  * [Синтаксис](#Синтаксис-3)
  * [Примеры](#Примеры-1)
* [CSS-injections](#css-injections)
  * [Синтаксис](#Синтаксис-4)
  * [Примеры](#Примеры-2)
* [JS rules](#javascript-rules)
  * [Синтаксис](#Синтаксис-5)
  * [Примеры](#Примеры-3)
* [HTML filtering rules](#html-filtering-rules)
  * [Синтаксис](#Синтаксис-6)
  * [Примеры](#Примеры-4)
  * [Атрибуты](#attributes)
    * [wildcard](#wildcard)
    * [loaded-script](#loaded-script)
    * [max-length](#max-length)
    * [min-length](#min-length)
    * [parent-elements](#parent-elements)
    * [parent-search-level](#parent-search-level)

## Введение

Фильтр — это набор правил фильтрации рекламного контента (баннеров, всплывающих окон и тому подобного). Вместе с Adguard поставляется набор стандартных фильтров, создаваемых нами. Они постоянно дорабатываются и дополняются, и, как мы надеемся, удовлетворяют большинство пользователей.

Вместе с тем, Adguard позволяет создать ваш собственный пользовательский фильтр, используя те же самые правила составления фильтров, что используем мы в наших фильтрах.

Важное замечание: Синтаксис правил Adguard изначально основан на синтаксисе правил Adblock Plus, но расширяет его, добавляя новые типы правил для лучшей фильтрации. Часть информации в этой статье о типах правил, общих для ABP и Adguard, взята из [этой статьи](http://adblockplus.org/ru/filters).

Для описания синтаксиса правил мы используем [Augmented BNF for Syntax Specifications](https://tools.ietf.org/html/rfc5234), но мы не всегда строго следуем этой спецификации.

## Базовые правила

Так называемые _"Базовые правила"_ — самый простой вид правил. Эти правила предназначены для блокировки запросов на определенный URL. 
Основной принцип для этого типа правил достаточно прост: необходимо указать адрес, где необходимо убрать рекламу, и дополнительные параметры, которые ограничивают или расширяют область действия правила.

### Синтаксис

```
      rule = [@@] pattern [ "$" modifiers ]
  modifiers = [modifier0, modifier1[, ...[, modifierN]]]
```

* **`pattern`** — маска адреса. URL каждого запроса сопостовляется с этой маской. В шаблоне вы можете использовать некоторые специальные символы, описание которых будет дано [ниже](#Специальные-символы).
* **`@@`** — маркер, который используется для обозначения правил-исключений. С такого маркера должны начинаться правила, отключающие фильтрацию для запроса.
* **`modifiers`** — Параметры, используемые для "уточнения" базового правила. Некоторые параметры ограничивают область действия правила, а некоторые могут полностью изменить принцип его работы.

#### Специальные символы

* **`*`** — Wildcard-символ. Символ, обозначающий "произвольный набор символов". Это может быть как пустая строка, так и строка любой длины.
* **`||`** — Соответствие началу адреса. Этот специальный символ позволяет не указывать конкретный протокол и поддомен в маске адреса. То есть, `||` соответствует сразу `http://*.`, `https://*.`, `ws://*.`, `wss://*.`.
* **`^`** — указатель для разделительного символа. Разделителем может быть любой символ кроме буквы, цифры и следующих символов: `_` `-` `.` `%`. Например, в адресе `http:`**`//`**`example.com`**`/?`**`t=1`**`&`**`t2=t3` жирным выделены разделительные символы.
* **`|`** — указатель на начало или конец адреса. Значение зависит от того, находится этот символ в начале или конце маски. Например, правило `swf|` соответствует `http://example.com/annoyingflash.swf` , но не `http://example.com/swf/index.html`. `|http://example.org` соответствует `http://example.org`, но не `http://domain.com?url=http://example.org`.

#### Примеры

* `||example.com/ads/*` — простое правило, которое просто соотвествует адресам типа `http://example.com/ads/banner.jpg` и даже `http://subdomain.example.com/ads/otherbanner.jpg`.

* `||example.org^$third-party` — правило, которое блокирует сторонние запросы к домену `example.org` и его поддоменам.

* `@@||example.com$document` — наиболее общее правило-исключение. Такое правило полностью отключает фильтрацию на домене `example.com` и всех его поддоменах. Существует ряд параметров, которые также можно использовать в правилах-исключениях. Более подробно о правилах-исключениях и параметрах, которые могут в таких правилах использоваться, написано [ниже](#exception-rules-modifiers).

### Модификаторы

_Возможности, описанные в этом разделе, обычно используются опытными пользователями. Они расширяют возможности «Общих правил», но для их применения необходимо иметь начальное представление о работе браузера._

Вы можете изменить поведение «общего правила», используя дополнительные модификаторы. Список этих параметров располагается в конце правила за знаком доллара `$` и разделяется запятыми. 
Например:
```
||domain.com$popup,third-party
```


#### Базовые модификаторы

Приведенные ниже модификаторы являются наиболее простыми (для понимания) и часто применяемыми. Они не могут быть использованы в правилах-исключениях.

##### **`domain`**

Модификатор `domain` ограничивает действия правила списком доменов. 
Для того, чтобы указать множество доменов в одном правиле, можно использовать символ `|` как разделитель.
Чтобы правило _не применялось_ на определенных доменах, перед доменным именем необходимо добавить символ `~`.


_Примеры:_

* `||baddomain.com/ads/banners*^$domain=example.com|example.ru` - блокировка контента на доменах `example.com` и `example.ru` 
* `||baddomain.com/ads*^$domain=~example.ru` - контент будет блокироваться на всех доменах, кроме `example.ru`
* `||baddomain.com/ads*^$domain=example.com|~foo.example.com` - в данном примере контент будет блокирвоаться на домене `example.com`, но не будет блокироваться на `foo.example.com`

##### **`third-party`**

Ограничение на сторонние/собственные запросы. Если указан модификатор `third-party`, то правило применяется лишь к запросам ресурсов из внешних источников.
`~third-party` ограничивает правило запросами из того же источника, что и открытая страница.

_Примеры:_

* `||domain.com$third-party` - правило применяется на всех сайтах, кроме самого `domain.com`
* `||domain.com$~third-party` - такое правило уже будет применяться только на самом `domain.com`, но не на других сайтах

##### **`popup`**

Adguard будет пытаться закрыть браузерную вкладку для любого адреса, который подходит под правило с этим модификатором.

_Примеры:_

* `||domain.com*^$popup` - при попытке перехода на сайт `http:\\domain.com` с любой страницы в браузере, новая вкладка, в которой должен открыться указанный сайт, будет закрыта  

##### **`match-case`**

Определяет правило, которое применяется только к адресам с совпадением регистра букв. По умолчанию регистр символов не учитывается.

_Примеры:_

* `*/BannerAd.gif$match-case` - такео правило будет блокировать `http://example.com/BannerAd.gif`, но не `http://example.com/bannerad.gif`.

##### **`image`**

Блокирует все картинки на указанном домене.

_Примеры:_

* `||domain.com^$image` - данное правило заблокирует все картинки на домене `domain.com`

##### **`stylesheet`**

Блокирует загрузку CSS-стилей. При использовании `@@` наоборот, разблокирует.

_Примеры:_

* `||domain.com/stylesheets^*$stylesheet` - блокирует загрузку css-стилей на указанных URL.
* `@@||gooddomain.com^*$` - разблокирует все заблокированные стили на указанном домене.

##### **`xmlhttprequest`**

Блокирует xmlhttpsrequest-запрос (запрос, который браузер загружает без обновления страницы). 
Напрмер, такие запросы зачастую пользуются интернет-магазинами при применении выбранных вами фильтров.

_Примеры:_

* `||example.com/Banners/*$xmlhttrequest` - блокирует xmlhttp запросы на указзаном URL
* `@@||example.ru^$xmlhttprequest` - разблокирует xmlhttp запросы на указанном домене 

##### **`empty`**

TBD: текст

_Примеры:_

TBD: example

##### **`mp4`**

Заменяет видео на заглушку. Данный модификатор на текущий момент не поддерживается расширениями.

_Примеры:_

* `||example.com/videos/*.mp4$mp4,domain=best-videos.com` - блокирует загрузку видео с URL `http://example.com/videos` на домене `http://best-videos.com`

#### Модификаторы правил-исключений

Модификаторы, описанные ниже, имеет смысл использовать только в правилах-исключениях. 
Такие правила должны начинаться с символа `@@`.

##### **`elemhide`**

Запрещает правила сокрытия элементов на страницах, подходящих под правило.
О правилах сокрытия элементов речь пойдет [ниже](#elemhide).

_Примеры:_

* `@@||example.com^$elemhide` - отменяет все правила скорытия элементов для домена `example.com`

##### **`content`**

Запрещает правила фильтрации HTML-элементов на страницах, подходящих под правило.
О правилах фильтрации HTML-элементов речь пойдет [ниже](#html-filtering-rules).

_Примеры:_

* `@@||domain.com^$content` - отменяет правила html фильтрации на домене `domain.com` 

##### **`jsinject`**

Запрещает добавление javascript-кода на страницу. 
О правилах для вставки javascript-кода речь пойдет ниже.
Также, javascript-код добавляется на страницу для работы _«помощника Adguard»_.
То есть, используя исключающие javascript-код правила, вы отключаете еще и "Помощника" на странице.

_Примеры:_

* `@@||example.com^$jsinject` - отменяет все правила вставки javascript-кода для домена `example.com`

##### **`urlblock`**

Запрещает блокировку запросов, отправленных со страниц подходящих под это правило.

_Примеры:_

* `@@||example.com^$urlblock` - запросы, отправленные с этого домена, больше не будет блокироваться.

##### **`document`**

Полностью запрещает фильтрацию на страницах, подходящих под правило. 
Эквивалентно одновременному использованию модификаторов `elemhide`, `content`, `urlblock` и `jsinject`.

_Примеры:_

* `@@||example.com^$document` - Полностью отключает фильтрацию на домене `example.com`

#### Generic rules

Generic-праила - это правила, действие которых не ограничено конкретными доменами

_Примеры_:

Например, следующие правила являются generic-правилами:
```
###banner
~domain.com###banner
||domain.com^
||domain.com^$domain=~example.ru
```
А вот такие правила уже не является generic-правилом:
```
domain.com###banner
||domain.com^$domain=example.ru
```

##### **`generichide`**

Разблокирует элементы, заблокированные generic-[правилами сокрытия элементов](#Правила-сокрытия-элемeнтов).

_Примеры_:

* `@@||example.com^generichide` - разблокирует на домене `example.com` все элементы, скрытые generic-правилами.

##### **`genericblock`**

Разблокирует элементы, скрытые простыми URL generic-правилами.

_Примеры:_

* `@@||example.com^$genericblock` - разблокирует на домене `example.com` все элементы, скрытые URL generic-правилами.

___
## RegExp support

Если вы хотите еще большей гибкости при составлении правил, вы можете использовать [регулярные выражения](https://ru.wikipedia.org/wiki/%D0%A0%D0%B5%D0%B3%D1%83%D0%BB%D1%8F%D1%80%D0%BD%D1%8B%D0%B5_%D0%B2%D1%8B%D1%80%D0%B0%D0%B6%D0%B5%D0%BD%D0%B8%D1%8F).

Суть работы проста: найденные при помощи регекспа адреса, будут заблокированы. Так же, возможно использование параметров обычных URL-правил.

**_Важное замечание:_** с точки зрения производительности рекомендуется по возможности избегать использования регулярных выражений.

#### Синтаксис

```
rule = pattern [ $ modifiers ]
```
В случае RegExp правил, в `pattern` записывается само регулярное выражение.

_Exaple:_

* `/banner\d+/` - такое правило, например, подойдет для блокировки URL, модержащих `banner123` или `banner321`, но не будет работать для banners.
* `/banner\d+/$domain=example.com` - так же, как и правило из примера выше, будет блокировать адреса, подходящие под это правило, но уже только на домене `example.com`

___
## Replace rules

Данный вид правил предназначем для "подмены" контента. При помощи регулярных выражений указывается сначала, _что_ необходимо заменить, затем то, _на что_ будет произведена замена. 
При этом, не обязательно что-то вставлять на замену, можно просто заменить "ни на что".

#### Синтаксис

**TBD!!!!!!!**
```
rule = pattern [ $ replace ] [RegExp1][RegExp2] 
```

_Примеры:_

* `||domain.com/config.xml$replace=/(<Ad[\s\S]*?>)[\s\S]*<\/Ad>/\$1<\/Ad>/` - в данном примере тэг `Ad` будет очищен

```
<AdServingTemplate>
<Ad id="advert-1">
//ads config
</Ad>
</AdServingTemplate>
```


___
## Правила сокрытия элемeнтов

**_Важно:_** Для работы с правилами скрытия необходимы знания HTML и CSS. Фактически, правила скрытия — это просто CSS-селекторы. Adguard добавляет на страницу собственные стили, состоящие из правил скрытия. Ко всем CSS-селекторам применяется стиль {display:none!important}.

**Примечание:**
_Правила скрытия кардинально отличаются от обычных правил. Например, не поддерживаются привычные символы масок — они имеют другое значение и применение._

### Синтаксис

```
rule = [domains] "##" selector
```

**Domains**

Домен не обязателен для указания. Если составить правило примерно следующим образом: `##.textad`, то оно будет блокирвоать класс `textad` для всех доменов.
Указать можно как один, так и множетво доменов. Если доменов множетсво, их необходимо перечислить через запятую: `domain1,domain2,domain3##.ads`
Для того, чтоб правило не работало на определенном домене, перед самим доменом необходимо добавить символ `~`. 

**Selector**

После списка доменов `##` является признаком правила скрытия. Далее следует указать селектор, который будет определять скрываемый элемент.
Почитать про css-селекторы можно, например, [тут](https://www.w3.org/wiki/CSS/Selectors).

### Примеры

* `example.com##.textad` - заблокирует `div` с классом _`textad`_ `<div class=textad>` на домене `example.com`
* `example.com,example.ru,domain.com###adblock` - скроет элемент по его атрибуту _`id`_ `<div id="adblock">` на доменах `example.com, example.ru, domain.com`
* `~example.com##.textad` - заблокирует `div` с классом _`textad`_ `<div class=textad>` на всех доменах, кроме `example.com`

___
## CSS-injections

Иногда недостаточно просто скрыть какой-либо элемент, чтобы заблокировать рекламу. Например, блокировка рекламного элемента может просто сломать верстку сайта. Для таких случаев Adguard позволяет использовать гораздо более гибкие правила, чем обычные правила сокрытия.

С помощью правил вставки CSS-стилей, вы сможете добавить на любой сайт любой CSS-стиль.

### Синтаксис

```
rule = [ domains ] "#$#" selector { style }
```

**Domains** 

Домен не обязателен для указания, но _очень желательно_ все же его указать, иначе можно поломать верстку на всех страницах.
Правила указания доменов те же, что и для [правил сокрытия элементов](#Правила-сокрытия-элементов).

**Selector** 

После списка доменов `#$#`указывает на то, что далее будет CSS-правило. Далее следует указать селектор, который будет определять скрываемый элемент.
Почитать про css-селекторы можно, например, [тут](https://www.w3.org/wiki/CSS/Selectors).

**Options:**

Внутри `{}` следует указать непосредственно стиль, который необходимо добавить на страницу.
В конце стиля для надежности можно добавить `!important` для того, чтобы "перебить" существующий стиль.

### Примеры

```
example.com#$#body { background-color: #333!important; }
```
___
## Javascript rules

Adguard поддерживает специальный тип правил, позволяющий вставить любой javascript-код на страницы интернет-сайтов.

### Синтаксис

```
rule = [ domains ]  "#%#" js-code
```

* _pattern - здесь вы указываете домен (домены), на которых будет работать праило. Работа с доменами происходит по аналогии с [правилами сокрытия элементов](#Правила-сокрытия-элементов). Если не указать домен, то правило будет применяться для всех доменов.

### Примеры

* `example.com#%#$('#somebanner').remove()` - Это правило означает, что в код всех страниц сайта `example.com` будет добавлен javascript-код:
```
<script type="text/javascript">
    $('#somebanner').remove();
</script>
```
* `example.com#%#(function() { ***JS code*** })();` - выполнится сразу(по возможности) при открытии страницы
* `example.com#%#AG_onLoad(function() { ***JS code*** });` - выполнится после загрузки страницы(ориентируется на событие DOMContentLoaded)

___
## HTML filtering rules

В большинстве случаев достаточно перечисленных правил. Но иногда для фильтрации рекламы необходимо изменять HTML-код самой страницы. Для того, чтобы сделать это применяются правила фильтрации HTML-контента. Они позволяют указать, какие HTML-элементы необходимо вырезать из страницы перед тем как отдать ее браузеру.

### Синтаксис

```
rule = [ domain ] "$$" element attributes
attributes = [[ name1 = value1 ][, name2 = value2]...[, nameN = valueN ]]
```
* domain - включает в себя URL (необязательно). Работа с доменами происходит аналогично [правилам сокрытия элементов](#ПРавила-сокрытия-элементов).
* element - элемент на странице, в котором необходимо отфильровать контент.
* attributes - параметры, при помощи которых можно более точно выбирать изменяемый контент.

### Примеры

Рассмотрим следующий пример:

```
<script type="text/javascript">
    document.write('<div>Покупайте пельмешки <a href="http://pelmeshki.com">Здесь!</a></div>" />');
</script>
```
В данном случае рекламный слоган появляется на странице не сразу, а после загрузки страницы. Для вывода используется `Javascript`. Используя первые два вида правил мы ничего сделать не можем. Тут нам на помощь и приходят правила фильтрации HTML-контента. Для того, чтобы избавиться от рекламы, нам нужно вырезать из страницы весь элемент script. Но все элементы `script` вырезать нельзя — они могут быть нужны для работы сайта. Используем следующее правило:

```
$$script[type="text/javascript"][tag-content="pelmeshki.com"]
```
Это правило расшифровывается следующим образом: удаляются все элементы _`script`_, атрибут type которых равен _`text/javascript`_, а внутри элемента встречается строка _`pelmeshki.com`_.

### Attributes

##### `wildcard`

В общем виде этот атрибут может быть записан следующим образом:
`[wildcard="*part1*part2*part3*...*"]`
Он позволяет удалить тэг, содержащий указанные внутри него части кода или параметры.

_Примеры:_

* `example.com$$style[wildcard="*width: 660px;*min-height: 239px;*"]` - удалит стили, содержащие `width: 660px` и `min-height: 239px`

##### `loaded-script`

Позволяет применять правило не только к элементам `script` на странице, но также и к подгружаемым скриптам.
Иногда это необходимо, так как скрипты рекламных сетей хранят в закриптованном виде на самом сайте. 

**Важно:**
Важно, чтобы правило, применяемое к подгружаемым скриптам, не содержало никаких атрибутов кроме: `tag-content`, `loaded-script`, `max-length`, `min-length`.

_Примеры:_

* `$$script[tag-content="crypted script"][loaded-script="true"]`

##### `max-length`

Задает максимальную длину содержимого HTML-элемента.
Если этот параметр задан, и длина содержимого превышает заданное значение — правило не применяется к элементу.

_Примеры:_

* `example.com$$div[tag-content="advertise"][max-length="1500"]` 

##### `min-length`

Задает минимальную длину содержимого HTML-элемента.
Если этот параметр задан, и длина содержимого меньше заданного значения — правило не применяется к элементу.

_Примеры:_

* `example.com$$div[tag-content="advertise"][min-length="300"]`

##### `parent-elements`

Очень важный атрибут. Он сильно изменяет то, как работает это правило.
Суть в том, что вырезается теперь не тот элемент, который найден, а заданный родительский элемент.

_Примеры:_

```
<table style="background: url('http://domain.com/banner.gif')">
    <tr>
        <td>
            <a href="http://pelmeshki.com">Купил пельмешки БЫСТРО</a>
        </td>
    </tr>
</table>
```
Проблема этого HTML-кода в том, что нам недостаточно вырезать ссылку на рекламу. Сам баннер показывается с помощью родительской таблицы (как ее background). 
Тут нам и приходит на помощь атрибут _parent-elements_. Мы используем вот такое правило, чтобы заблокировать всю таблицу: 
`$$a[href="pelmeshki.com"][parent-elements="table"]`.
Когда Adguard найдет на странице ссылку на `pelmeshki.com`, вместо того, чтобы вырезать ее - он будет искать ближайший родительский элемент `table`, и, если найдет — вырежет его.
Вы можете указать несколько искомых родительских элементов через запятую. Заблокирован будет ближайший.

##### `parent-search-level`

Задает максимальную глубину поиска родительского элемента. По умолчанию максимальная глубина поиска равна 3. 
Это сделано для того, чтобы не вырезать лишнего, если HTML страницы поменяется. Не ставьте слишком большие значения для этого атрибута.
