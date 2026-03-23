# Натягиваем шаблон на CMS SHADOW
Пишем на php, поэтому при натяжке шаблонов используем шаблонизатор blade, можно смотреть документацию по конкретным компонентам в его документации https://laravel.su/docs/12.x/blade

Советую при натяжке опираться на пример furniture-2.vitmp.ru

## ВАЖНО

Добавлять `eval(response.reachgoal);` В успешной обработке формы, для проставления цели

## Описание методов

Например: `@include(get_view('head'))` 

`@include()` -- встроенная blade директива, котороая вставляет компонент в разметку. 

`get_view()` -- наш метод, который забирает разметку из созданных представлений (пример представления: https://furniture-2.vitmp.ru/dashboard/views/2/edit) по полю `slug` 

`setting('site.victory.call', 'false')` -- берет значение из раздела настроек

`{!! !!}` -- c помощью такой конструкции можено выводить блоки кода записанные в переменные (html/js например)

## Зарезервированные имена представлений
`template-main` -- основная обертка всех страниц

`template-blog` -- страница статьи, в которую уже прокинута сама статья их бд 

### template-main
Создаем представление под названием "template-main" - основная обертка всех страниц. На всех шаблонах он выглядит примерно одинаково, отличается в основном только базовая html разметка. Смотрим пример на furniture-2.vitmp.ru.

```html
<!DOCTYPE html>
<html lang="en">
	<head>
        @include(get_view('head'))
        @include('metrica.head')
	</head>
	<body>
   	    @include('metrica.body')
		<div class="page">
			@include(get_view('header'), [
          'indexPage' => $indexPage ?? true
      ])
			@yield('content')
            @include(get_view('footer'))
		</div>
        @include(get_view('scripts'))
        @include(get_view('popups'))
		<script>
		    @if(setting('site.victory.call', 'false') == 'false')
            window.verification = false;
        @else 
            window.verification = true;
        @endif
        </script>
		@yield('scripts')
		@include(get_view('cookie'))
		@include('metrica.footer')
	</body>
</html>
```
#### @include(get_view('head'))
Подключение тега `<head><head/>`, который создается отдельным представлением

#### @include('metrica.head'),  @include('metrica.body')
Все компоненты, у которых не вызывается метод `get_view()` зашиты в саму cms, это сервисные компоненты, которые как в примере нужны для работы яндекс метрики

## Инфоблоки
Все контентные элементы создаем в инфоблоках

### Создание инфоблока
1.https://furniture-2.vitmp.ru/dashboard/collections -> добавить

2.Указываем название

3.Указываем идентификатор (ниже в примере `galleryText`)

4.Добавляем нужные поля

Я создаю текстовую часть (заголовки, подзаголовки) в инфоблоках, хотя также их можно хранить в константах

`get_collections` -- метод возвращающий значение инфоблоков. Аргументы: ключ инфоблока; ключ кеширования инфоблока; массив значений, которые хотим получить

```php
$galleryText = get_collections('galleryText', 'galleryText', [
        'tag',
        'title',
        'text'
    ]);
```

#### Пример
```html
@php
    $galleryText = get_collections('galleryText', 'galleryText', [
        'tag',
        'title',
        'text'
    ]);
    
    $gallery = get_collections('gallery', 'gallery' , ['image', 'name']);
@endphp;

@if(count($gallery))
    <section class="section gallery">
        <div class="container">
            <div class="section__header">
                <span class="section__tag">{{ $galleryText[0]['tag'] }}</span>
                <div class="section__text">
                    <h2 class="section__title h2">{{ $galleryText[0]['title'] }}</h2>
                    <span>{{ $galleryText[0]['text'] }}</span>
                </div>
            </div>
        
            <div class="gallery__container">
                <div class="slider-button-container">
                    <button class="slider-button slider-button--prev gallery__button--prev" ></button>
                    <button class="slider-button slider-button--next gallery__button--next" ></button>
                </div>
                <div class="gallery__slider swiper">
                    <div class="swiper-wrapper">
                        @foreach($gallery as $galleryItem)
                            <div class="gallery__slide swiper-slide">
                                <img src="{{ $galleryItem['image'] }}" alt="{{ $galleryItem['name'] }}" />
                            </div>
                        @endforeach
                    </div>
                </div>
            </div>
        </div>
    </section>
@endif
```

## Переменные
Берем через метод `{{ get_variable('address') }}`

Создаются в разделе константы

## Формы

на тег `form` накидываем класс `js-form-validator` а так-же все параметры которые указаны ниже, для всех форм они едины.

добавляем `@csrf` внутрь формы

на инпуты телефона у нас `name="telephone"`, `class="js-phone-mask"`
на инпут политики у нас `name="agreement"`

по ним валидирует валидатор


пример

```html
<form class="modal__form js-form-validator" data-form data-method="POST" data-action="{{ route('form.send.ajax', ['view_id' => $view['id']]) }}">
    @csrf
  <div class="form__fields form__fields-row">
    <div class="form__field">
      <span>Имя</span>
      <input type="text" name="name" placeholder="Иван" />
    </div>
    <div class="form__field">
      <span>Телефон</span>
      <input type="tel" name="telephone" class="js-phone-mask" placeholder="+7 (000) 000-00-00"/>
    </div>
    <button class="form__button button button--primary button--arrow" type="submit">
      <span>Отправить</span>
    </button>
  </div>
  @include(get_view('agreement'))
</form>
```

## Шаблоны для копирования

### Хлебные крошки

#### Использование
```html
	@php
		$items = [
			['title' => 'Блог', 'url' => '/blog'],
			['title' => $article->title],
		];
	@endphp

	@include(get_view('breadcrumbs'), ['items' => $items])
```
#### Пример
```html
	@php 
	    array_unshift($items, [ 'title' => 'Главная', 'url' => '/' ]); 
	
	    $host = request()->getSchemeAndHttpHost();
	
	    $breadcrumbsJson = [
	        "@context" => "https://schema.org",
	        "@type" => "BreadcrumbList",
	        "itemListElement" => []
	    ];
	
	    $navigationGraph = [];
	    $addedUrls = [];
	
	    foreach($items as $index => $item) {
	        // Хлебные крошки
	        $breadcrumbItem = [
	            "@type" => "ListItem",
	            "position" => $index + 1,
	            "name" => $item['title']
	        ];
	
	        if (!empty($item['url'])) {
	            $fullUrl = $host . $item['url'];
	            $breadcrumbItem['item'] = $fullUrl;
	
	            // Навигация
	            if (!in_array($item['url'], $addedUrls)) {
	                $addedUrls[] = $item['url'];
	                $navigationGraph[] = [
	                    "@type" => "SiteNavigationElement",
	                    "name" => $item['title'],
	                    "url" => $fullUrl,
	                ];
	            }
	        }
	
	        $breadcrumbsJson["itemListElement"][] = $breadcrumbItem;
	    }
	
	    $navigationJson = [
	        "@context" => "https://schema.org",
	        "@graph" => $navigationGraph,
	    ];
	@endphp
	
	@if(isset($breadcrumbsJson))
	    @section('json-ld-breadcrumbs')
	        <script type="application/ld+json">@json($breadcrumbsJson)</script>
	        <script type="application/ld+json">@json($navigationJson)</script>
	    @endsection
	@endif
	
	<div class="breadcrumbs @if(isset($class)) {{ $class }} @endif">
	    @foreach($items as $item)
	        @if(isset($item['url']))
	            <a href="{{ url($item['url']) }}" class="breadcrumbs__link">{{ $item['title'] }}</a>
	        @else
	            <span class="breadcrumbs__link current">{{ $item['title'] }}</span>
	        @endif
	    @endforeach
	</div>
```

### HEAD

#### Использование
```html
	@include(get_view('head')
```
#### Пример
```html
	<meta charset="utf-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0" />
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<meta name="referrer" content="origin">
	<meta name="HandheldFriendly" content="true">
	
	<meta name="csrf-token" content="{{ csrf_token() }}">
	<meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">
	
	@hasSection('og')
		<meta property="og:type" content="website">
		<meta property="og:site_name" content="{{ request()->getHost() }}">
		<meta property="og:title" content="@yield('og.title')">
		<meta property="og:description" content="@yield('og.description')">
		<meta property="og:url" content="@yield('og.url')">
		<meta property="og:image" content="@yield('og.image')">
	@endif
	
	@php
	    $sections = View::getSections();
	
	    $title = isset($sections['title']) ? $sections['title'] : '';
	    $description = isset($sections['description']) ? $sections['description'] : '';
	    $noindex = isset($sections['no-index']) ? $sections['no-index'] : false;
	
	    $name =  get_variable('name_dealer');
	    $domain = request()->getHost();
	
	    $title = str_replace('[name]', $name, $title);
	    $title = str_replace('[domain]', $domain, $title);
	
	    $description = str_replace('[name]', $name, $description);
	    $description = str_replace('[domain]', $domain, $description);
	    
	    $urlSlug = '@' . url_slug( request()->getHost() );
	@endphp
	
	@hasSection('twitter')
	    <meta name="twitter:card" content="summary_large_image">
	    <meta name="twitter:site" content="{{ $urlSlug }}">
	    <meta name="twitter:title" content="@yield('twitter.title')">
	    <meta name="twitter:description" content="@yield('twitter.description')">
	    <meta name="twitter:image" content="@yield('twitter.image')">
	    <meta name="twitter:url" content="@yield('twitter.url')">
	@endif
	
	<title>{{ $title }}</title>
	<meta name="description" content="{{ $description }}">
	
	@if($noindex)
	    <meta name="robots" content="index, follow"/>
	@endif
	
	@if(!request()->has('page') or (request()->has('page') and request()->get('page') == '1'))
	    <link rel="canonical" href="{{ url()->current() }}">
	@else
	    <link rel="canonical" href="{{ url()->full() }}">
	@endif
	
	<link rel="shortcut icon" type="image/jpg" href="{{ get_variable('favicon') }}"/>
	<link rel="stylesheet" href="{{ assets_url() }}/css/main-CjGgNvcF.css">
	@include(get_view('custom_styles'))
	
	{!! get_variable('style') !!}
	
	@yield('json-ld-breadcrumbs')
	@yield('json-ld')
```

### Пагинация

#### Пример
```html
	@if ($data->lastPage() > 1)
    @section('pagination-links')
        @if (!$data->onFirstPage())
            @php
                $prevUrl = $data->previousPageUrl();
                $prevUrl = preg_replace('/[?&]page=1(&|$)/', '', $prevUrl);
                $prevUrl = rtrim($prevUrl, '?&');
            @endphp
            <link rel="prev" href="{{ $prevUrl }}">
        @endif
        
        @if($data->hasMorePages())
            <link rel="next" href="{{ $data->nextPageUrl() }}">
        @endif
    @endsection
    <div class="pagination">
        @if(!$data->onFirstPage())
            <a href="{{ $data->previousPageUrl() == $data->url(1) ? $data->url(1) : $data->previousPageUrl() }}" class="pagination__link prev"></a>
        @endif

        <div class="pagination__links">
            @if($data->currentPage() == 1)
                <a href="javascript:;" class="pagination__link number current">1</a>
            @else
                <a href="{{ $data->url(1) }}" class="pagination__link number">1</a>
            @endif

            @if($data->currentPage() > 3)
                <span class="pagination__link number">...</span>
            @endif

            @php
                $start = max(2, $data->currentPage() - 2);
                $end   = min($data->lastPage() - 1, $data->currentPage() + 2);
            @endphp
            
            @for($i = $start; $i <= $end; $i++)
                @php
                    $extraClass = '';
                    // крайний левый скрываем только если между текущей и start >= 2
                    if ($i == $start && $data->currentPage() - $start == 2) {
                        $extraClass = ' hide-mobil';
                    }
                    // крайний правый скрываем только если между end и текущей >= 2
                    if ($i == $end && $end - $data->currentPage() == 2) {
                        $extraClass = ' hide-mobil';
                    }
                @endphp
            
                @if ($i == $data->currentPage())
                    <a href="javascript:;" class="pagination__link number current">{{ $i }}</a>
                @else
                    <a href="{{ $data->url($i) }}" class="pagination__link number">{{ $i }}</a>
                @endif
            @endfor


            @if($data->currentPage() < $data->lastPage() - 2)
                <span class="pagination__link number">...</span>
            @endif

            @if($data->lastPage() > 1)
                @if($data->currentPage() == $data->lastPage())
                    <a href="javascript:;" class="pagination__link number current">{{ $data->lastPage() }}</a>
                @else
                    <a href="{{ $data->url($data->lastPage()) }}" class="pagination__link number">{{ $data->lastPage() }}</a>
                @endif
            @endif
        </div>

        @if($data->hasMorePages())
            <a href="{{ $data->nextPageUrl() }}" class="pagination__link next"></a>
        @endif
    </div>
@endif
```
#### Использование
```html
	@include(get_view('pagination'), [
		'data' => $articles->appends(request()->query())
	])
```


