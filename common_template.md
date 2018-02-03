## Общий шаблон сайта

Одна из наиболее типичных архитектурных ошибок при проектировании веб-приложений - общий HTML-шаблон для всего сайта, примерно такого вида:

```php
<!DOCTYPE html>
<html>
    <head>
        <?php include('./path/to/header.phtml'); ?>
    </head>
    <body id="main">

        <div id="content">
            <div id="head">
                <!-- много HTML-кода -->
            </div>

            <!-- много HTML-кода -->
            <?php include('./path/to/' . $content_template_file); ?>
            <!-- много HTML-кода -->
        </div>

        <div id="footer">
            <?php include('./path/to/footer.phtml'); ?>
        </div>
    </body>
</html>
```

Наличие общих шаблонов для всего сайта оправдано, наверно, на совсем небольших сайтах-визитках. В остальных случаях более целесообразно идти по принципу один обработчик = один шаблон.

### Расширяемость проекта как основной аргумент

Любой проект рано или поздно расширяется. Появляется новый функционал, который неизбежно затрагивает и вёрстку. Допустим,  у нас имеется один общий шаблон сайта, который мы используем как для показа списка новостей, так и для показа отдельной новости с комментариями. Внезапно появляется задача - на странице показа отдельной новости выводить слева или справа вертикальный баннер, который должен идти чуть ли не от самого верхнего края страницы. Общий шаблон требует переверстки и он верстается в соответствии с новыми требованиями. Но возникает казус - а как быть со страницей со списком новостей - её трогать не было указаний. И тут возникает два пути решения конфликта:

1. Создать по одному общему шаблону как для страницы со списком новостей, так и для страницы отдельной новости.
2. Создать в общем шаблоне условие на PHP, которое в зависимости от наличия определенного флага "рисовало" бы дополнительный HTML-код под вертикальный баннер.

Если пойти по второму пути, то в конечном итоге мы получим такой общий шаблон:

```php
<!DOCTYPE html>
<html>
    <head>
        <?php include('./path/to/header.phtml'); ?>
    </head>
    <body id="main">

        <div id="content">
            <!-- Добавили в общий шаблон условие показа баннера на странице одной новости -->
            <? if ($view_banner_on_single_news_page): ?>
                 <div id="banner"><img src="..." /></div>
            <? endif; ?>

            <div id="head">
                <!-- много HTML-кода -->
            </div>

            <!-- много HTML-кода -->
            <?php include('./path/to/' . $content_template_file); ?>
            <!-- много HTML-кода -->
        </div>

        <div id="footer">
            <?php include('./path/to/footer.phtml'); ?>
        </div>
    </body>
</html>
```

B это в **лучшем** случае, т.к. в данном примере HTML кода присутствует лишь одно условие. Но что делать, если бы сам каркас требовал кардинального изменения верстки? Очевидно, что писать IF-подобные условия для рисования "на лету" каркаса в реальном проекте было бы просто делом опасным - и верстальщик и программист могли бы просто потерять со временем контроль над кодом, его стало бы очень тяжело поддерживать.

Модулей сайта может быть очень много, соответственно, рост подобного числа вставок на PHP будет расти. Эти вставки с условиями будут путать разработчиков, которые будут гадать, к какому модулю относится та или иная IF-конструкция. В конечном итоге это приведет к разрастанию файлов шаблонов, которые будет очень тяжело поддерживать и читать.

### Аргументы защитников одного основного шаблона

Основной аргумент - не надо плодить код. Действительно, плодить однотипный код - не очень хорошо. Например, подвал страницы, динамически генерирующееся меню и другие практически независимые от основного шаблона куски HTML-кода действительно лучше вынести в отдельный файл и подключать через include или SSI. Но на сам каркас страницы должно распространяться правило **"один обработчик = один шаблон"**. Почему - описано выше.

Но не все независимые на первый взгляд элементы верстки нужно выносить в отдельные подключаемые файлы шаблонов. Например, HTML код тега <head> нет смысла выносить в отдельный файл, т.к. на каждой странице конкретного модуля будут требоваться индивидуальные JavaScript и CSS включения, различные мета-теги и т.п. Использование общего head-файла шаблона может привести к тому, что на каждой странице сайта будет подтягиваться куча совершенно не нужных JavaScript и CSS файлов.

### Последний аргумент

Если кто-то ещё сомневается и склоняется к использованию общего шаблона, то предлагаю посмотреть реальный пример HTML-кода из очень большого проекта интернет-магазина. Это лишь 100 строк из 600-строчного общего файла шаблона header.tpl, который, по мнению разработчиков, должен был обеспечить удобство и абстрагировать их от дублирования кода. Насколько ожидания разработчиков оказались оправданными - судить вам.

```php
<title>{$ptitle|default:$GlobalDisplay.title}</title>
<meta http-equiv="Content-Type" content="text/html; charset=windows-1251" />
{if $GlobalDisplay.redirect}{$GlobalDisplay.redirect}{/if}
{if $GlobalDisplay.meta_description || $meta_description}
    <meta name="description" content="{$meta_description|default:$GlobalDisplay.meta_description|escape}" />
{/if}
{if $GlobalDisplay.meta_keywords || $meta_keywords}
    <meta name="keywords" content="{$meta_keywords|default:$GlobalDisplay.meta_keywords|escape}" />
{/if}

{if $GlobalDisplay.meta_noindex}
    <meta name="robots" content="noindex, nofollow" />
{else}
    <meta name="robots" content="index,follow" />
{/if}
 
<link rel="stylesheet" type="text/css" href="/css/business.css" />
<link rel="stylesheet" type="text/css" href="/css/yourcity.css" />
{if $is_cart}
    <link rel="stylesheet" href="/newdesign/css/cart.css?v={$GlobalDisplay.cssVersion}" type="text/css" />
{/if}
{if $GlobalDisplay.main==1}
    <link rel="stylesheet" href="http://{$GlobalDisplay.HttpHost}/css/main.css?v={$GlobalDisplay.cssVersion}" type="text/css" />
{/if}
<link rel="stylesheet" href="http://{$GlobalDisplay.HttpHost}/css/new_year_2011.css??v={$GlobalDisplay.cssVersion}" type="text/css" />
<link rel="stylesheet" href="http://{$GlobalDisplay.HttpHost}/css/dropselect.css?v={$GlobalDisplay.cssVersion}" type="text/css" />
{if $GlobalConfig.RegionID == 6}
    <link rel="stylesheet" href="http://{$GlobalDisplay.HttpHost}/newdesign/css/fon_spb.css?v={$GlobalDisplay.cssVersion}" type="text/css" />
{else}
{if $smarty.now|date_format:"%Y-%m-%d" == "2012-05-09"}
    <link rel="stylesheet" href="http://{$GlobalDisplay.HttpHost}/newdesign/css/fon_9_may.css?v={$GlobalDisplay.cssVersion}" type="text/css" />
{else}
    {if $GlobalDisplay.homeshop}
        <link rel="stylesheet" href="http://{$GlobalDisplay.HttpHost}/newdesign/css/fon.css?v={$GlobalDisplay.cssVersion}" type="text/css" />
    {elseif $GlobalConfig.RegionID == 1}
        <link rel="stylesheet" href="http://{$GlobalDisplay.HttpHost}/newdesign/css/fon_msk.css?v={$GlobalDisplay.cssVersion}" type="text/css" />
    {elseif $GlobalConfig.RegionID == 6}
        <link rel="stylesheet" href="http://{$GlobalDisplay.HttpHost}/newdesign/css/fon_spb.css?v={$GlobalDisplay.cssVersion}" type="text/css" />
    {else}
        <link rel="stylesheet" href="http://{$GlobalDisplay.HttpHost}/newdesign/css/fon.css?v={$GlobalDisplay.cssVersion}" type="text/css" />
    {/if}
{/if}

<link rel="stylesheet" href="http://{$GlobalDisplay.HttpHost}/newdesign/css/card.css?v={$GlobalDisplay.cssVersion}" type="text/css" />
<link rel="stylesheet" href="http://{$GlobalDisplay.HttpHost}/newdesign/css/numbers.css?v={$GlobalDisplay.cssVersion}" type="text/css" />
<link rel="stylesheet" href="http://{$GlobalDisplay.HttpHost}/cssandjs/catalog.css?v={$GlobalDisplay.cssVersion}" type="text/css" />

{if $GlobalDisplay.is_cabinet}
    <link rel="stylesheet" href="http://{$GlobalDisplay.HttpHost}/newdesign/css/cabinet.css?v={$GlobalDisplay.cssVersion}" type="text/css" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
{/if}
 
{if $listing==1}
    <link rel="stylesheet" href="/css/listing{if $new_listing}_new{/if}.css?v={$GlobalDisplay.cssVersion}" type="text/css"/>
{/if}
 
{if $social_image_src || $GlobalDisplay.social_image_src}
    <link rel="image_src" href="{$social_image_src|default:$GlobalDisplay.social_image_src}" />
    <meta property="og:image" content="{$social_image_src|default:$GlobalDisplay.social_image_src}" />
    <meta property="og:title" content="{$ptitle|default:$GlobalDisplay.title|escape}" />
    <meta property="og:description" content="{$meta_description|default:$GlobalDisplay.meta_description|escape}" /> 
{/if}

{if $GlobalDisplay.coffee_konkurs && $GlobalDisplay.img_id}    
    <link rel="image_src" href="http://{$GlobalDisplay.HttpHost}/upload/coffee-konkurs/{$GlobalDisplay.img_id}.jpg" />
    <meta property="og:image" content="http://{$GlobalDisplay.HttpHost}/upload/coffee-konkurs/{$GlobalDisplay.img_id}.jpg" />  
    <meta property="og:title" content="Моя работа в конкурсе КОФЕМАГИЯ" />
    <meta property="og:description" content="{$GlobalDisplay.meta_description|escape}" />
{/if}
    
{if $GlobalDisplay.work_info.id}    
    <link rel="image_src" href="http://{$GlobalDisplay.HttpHost}/upload/coffee-konkurs/{$GlobalDisplay.work_info.id}.jpg" />
    <meta property="og:image" content="http://{$GlobalDisplay.HttpHost}/upload/coffee-konkurs/{$GlobalDisplay.work_info.id}.jpg" />  
    <meta property="og:title" content="Мне нравится работа в конкурсе КОФЕМАГИЯ" />
    <meta property="og:description" content="{$GlobalDisplay.meta_description|escape}" />
{/if}
    
{if $GlobalDisplay.ware.warecode}<link rel="image_src" href="http://{$GlobalDisplay.HttpHost}/Pdb/{$GlobalDisplay.ware.warecode}.jpg" />{/if}
{if $GlobalDisplay.news_info.id || $GlobalDisplay.today}
    {if $GlobalDisplay.news_info.id}
        <link rel="image_src" href="http://{$GlobalDisplay.HttpHost}/new_imgs/news/{$GlobalDisplay.news_info.id}.jpg" />
        <meta property="og:image" content="http://{$GlobalDisplay.HttpHost}/new_imgs/news/{$GlobalDisplay.news_info.id}.jpg" />    
    {/if}
    {if $GlobalDisplay.today}
        <link rel="image_src" href="http://{$GlobalDisplay.HttpHost}/Pdb/{$GlobalDisplay.today.warecode}.jpg" />
        <meta property="og:image" content="http://{$GlobalDisplay.HttpHost}/Pdb/{$GlobalDisplay.today.warecode}.jpg" />    
    {/if}
    <meta property="og:title" content="{$GlobalDisplay.title|escape}" />
    <meta property="og:description" content="{$GlobalDisplay.meta_description|escape}" />    
{/if}

{if $GlobalConfig.iphone}<link rel="stylesheet" href="/new_css/iphone.css?v={$GlobalDisplay.cssVersion}" type="text/css">{/if}
{if $GlobalConfig.ipod}<link rel="stylesheet" href="/ipod.css?v={$GlobalDisplay.cssVersion}" type="text/css">{/if}
{if $GlobalConfig.application}<link rel="stylesheet" href="/app.css?v={$GlobalDisplay.cssVersion}" type="text/css">{/if}
```


