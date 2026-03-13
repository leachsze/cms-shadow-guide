# Натягиваем шаблон на CMS SHADOW
Пишем на php, поэтому при натяжке шаблонов используем шаблонизатор blade, можно смотреть документацию по конкретным компонентам в его документации https://laravel.su/docs/12.x/blade

Советую при натяжке опираться на пример furniture-2.vitmp.ru

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

