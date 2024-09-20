Vamos [[criar as rotas utilizando uma Closure]] para não precisar criar um controlador e métodos para simplesmente devolver uma visão.

```php
Route::get('/', function() {
    return view('welcome');
})->name('site.home');

Route::get('/cursos', function() {
    return view('welcome');
})->name('site.courses');

Route::get('/contato', function() {
    return view('welcome');
})->name('site.contact');
```

É interessante que a aplicação não conheça a URL, mas somente o nome das rotas, pois assim uma coisa fica desacoplada da outra e, se por qualquer motivo você precisar alterar o sistema de rotas, é possível alterar os caminhos que a aplicação continuará funcionando.

#### Configurando a rota `home` e criando uma view:

Vamos criar uma view no diretório `views`. Ela ficará dentro de uma pasta chamada `site`
	`home.blade.php`

Agora, vamos ajeitar a rota para `home`:

```php
Route::get('/', function () {
    return view('site.home');
})->name('site.home');
```

Quando utilizamos o helper view, automaticamente o laravel sabe que o diretório que ele precisa fazer a pesquisa para achar o arquivo é o `resources/views`.
O `site.` refere-se ao nome do diretório que criamos dentro da view.
O laravel também já compreende que ao invés de passar `/` nós podemos passar `. `, pois ele sabe que o arquivo `home` está dentro do diretório `site`.
Não precisamos colocar o `.blade.php`, pois automaticamente o motor do Laravel consegue reconhecer que o blade está sendo utilizado.

#### [[Incluir o Bootstrap]]

#### Pegando um site de exemplo

- Vamos usar o modelo Carousel do Bootstrap para agilizar e não precisar fazer o HTML.
-  Para isso, basta abrir o modelo, ver o código fonte e colar tudo que está dentro do `body`
-  No entanto, o css ficou desarrumado. O ideal é que ele fique em um outro arquivo dentro de uma pasta css. No momento, vamos copiar tudo que está em `<style>`.
-  Na pasta public, vamos criar um arquivo `style.css` e colar.
-  Dentro da view `home` vamos incluir o arquivo `style.css`

```html
<link rel="stylesheet" href="http://localhost/laravel-tips/public/style.css">
```

-  Dentro do código fonte da página, há um arquivo `carousel.css`. Vamos abrir, copiar tudo que está contido nele e depois colar em `style.css`.

### Usando o blade

#### [[Interpolação]]:

Vamos inserir no _footer_ a data de forma dinâmica. Para isso, precisamos usar a função `date()` do php. O blade facilita essa inserção utilizando o caractere de interpolação `{{ }}`

```html
    <footer class="container">
      <p class="float-end"><a href="#">Back to top</a></p>
      <p>&copy; {{date('Y')}} Company, Inc. &middot; <a href="#">Privacy</a> &middot; <a href="#">Terms</a></p>
    </footer>
```

O caractere de **interpolação** facilita quando temos uma visão com formulário com diversos campos. diversas informações que estão vindo do controlador e é necessário colocar os campos dinâmicos dentro do html. Basta colocar `{{ }}` e você pode colocar uma variável ou fazer uma condição.

#### [[Diretivas do Blade]]:

Vamos usar as diretivas do Blade no código. Antes disso, um exemplo de como seria o código sem utilizá-la, abrindo e fechando tag php:

Código para mostrar uma mensagem de Bom dia/Boa tarde/Boa noite de acordo com o horário:
```php
<?php

if(date('H') >= 0 && date('H') <= 12 {
	echo "<p>Bom dia!</p>";
} elseif (date('H') >= 13 && date('H') <= 18 {
	echo "<p>Boa tarde!</p>";
} else {
	echo "<p>Boa noite!</p>";
}

?>
```

No Laravel, o horário vem como padrão `UTC`. Vamos configurar para `America/Sao_Paulo`:

```php
'timezone' => 'America/Sao_Paulo',
```

Usando a diretiva Blade:

```php
@if(date('H') >= 0 && date('H') <= 12)
	<p>Bom dia!</p>
@elseif(date('H') >= 13 && date('H') <= 18)
	<p>Boa tarde!</p>
@else
	<p>Boa noite!</p>
@endif
```


### Criando as outras páginas:

Vamos alterar o código em `home` para colocar os links para página`cursos` e `contatos`

```html
  <header data-bs-theme="dark">
    <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
      <div class="container-fluid">
        <a class="navbar-brand" href="#">Up Cursos</a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarCollapse" aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav me-auto mb-2 mb-md-0">
            <li class="nav-item">
              <a class="nav-link active" aria-current="page" href="#">Home</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="#">Cursos</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="#">Contato</a>
            </li>
          </ul>
        </div>
      </div>
    </nav>
  </header>
```

Criar as views  `contact.blade.php` e `courses.blade.php`

Para montar a página de contatos, precisamos usar o menu e o rodapé.
Para isso, precisaremos _fatiar_ o código e trabalhar com conceito de **herança**.

### Herança:

Criar um _layout master_ e todas as páginas irão _estender_ esse layout.
Quando fazemos isso, conseguimos criar _blocos_ no site e alimentá-los dinamicamente.

Dentro da pasta `site` vamos criar um diretório `master`.
No diretório `master`, criar o arquivo `layout.blade.php`

Recortar todo o conteúdo de `home.blade.php` e colar em `layout.blade.php`

Em `home.blade.php` vamos usar a diretiva `@extends`:
```php
@extends('site.master.layout');
```

Copiar essa diretiva para as views `contact` e `courses`.

Alterar a url do menu usando o caractere de interpolação do blade:
```html
            <li class="nav-item">
              <a class="nav-link active" aria-current="page" href="{{ route('site.home') }}">Home</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="{{ route('site.courses') }}">Cursos</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="{{ route('site.contact') }}">Contato</a>
            </li>
```

### Yield e Section:

Em `layout.blade.php` vamos recortar tudo dentro do `main` e colar em `home`
Dentro da tag `main`, vamos usar a diretiva `yield`

```php
@yield('section')
```

Em `home` vamos colar todo o conteúdo dentro da diretiva `section`

```html
@section('content')
	<div>
	...
	</div>
@endsection
```

Para colocar algum conteúdo dentro das views `courses` e `contact`, usamos a diretiva `section`

```php
@extends('site.master.layout');
@section('content')
    <h1>Cursos</h1>
@endsection
```


Em `contact.blade.php` vamos criar uma `div.container`  para centralizar o conteúdo e colocar um form do bootstrap

```html
@extends('site.master.layout');

@section('content')
    <div class="jumbotron">
        <div class="container text-center">
            <h1 class="display-4">CONTATO</h1>
            <hr class="my-4">
            <p class="lead">Nossa equipe está sempre em disposição para te ajudar. Entre em contato e retornaremos o mais breve possível.</p>
        </div>
    </div>
    
    <div class="container">
        <form>
            <div class="mb-3">
              <label for="exampleInputEmail1" class="form-label">Email address</label>
              <input type="email" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp">
              <div id="emailHelp" class="form-text">We'll never share your email with anyone else.</div>
            </div>
            <div class="mb-3">
              <label for="exampleInputPassword1" class="form-label">Password</label>
              <input type="password" class="form-control" id="exampleInputPassword1">
            </div>
            <div class="mb-3 form-check">
              <input type="checkbox" class="form-check-input" id="exampleCheck1">
              <label class="form-check-label" for="exampleCheck1">Check me out</label>
            </div>
            <button type="submit" class="btn btn-primary">Submit</button>
          </form>
    </div>
@endsection
```

O mesmo será feito em `courses.blade.php`

```html
@extends('site.master.layout');

@section('content')
    <div class="container">
        <!-- START THE FEATURETTES -->
        <hr class="featurette-divider">
        <div class="row featurette">
        <div class="col-md-7">
            <h2 class="featurette-heading fw-normal lh-1">First featurette heading. <span class="text-body-secondary">It’ll blow your mind.</span></h2>
            <p class="lead">Some great placeholder content for the first featurette here. Imagine some exciting prose here.</p>
        </div>
        <div class="col-md-5">
            <svg class="bd-placeholder-img bd-placeholder-img-lg featurette-image img-fluid mx-auto" width="500" height="500" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Placeholder: 500x500" preserveAspectRatio="xMidYMid slice" focusable="false"><title>Placeholder</title><rect width="100%" height="100%" fill="var(--bs-secondary-bg)"/><text x="50%" y="50%" fill="var(--bs-secondary-color)" dy=".3em">500x500</text></svg>
        </div>
        </div>
        <hr class="featurette-divider">
        <div class="row featurette">
        <div class="col-md-7 order-md-2">
            <h2 class="featurette-heading fw-normal lh-1">Oh yeah, it’s that good. <span class="text-body-secondary">See for yourself.</span></h2>
            <p class="lead">Another featurette? Of course. More placeholder content here to give you an idea of how this layout would work with some actual real-world content in place.</p>
        </div>
        <div class="col-md-5 order-md-1">
            <svg class="bd-placeholder-img bd-placeholder-img-lg featurette-image img-fluid mx-auto" width="500" height="500" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Placeholder: 500x500" preserveAspectRatio="xMidYMid slice" focusable="false"><title>Placeholder</title><rect width="100%" height="100%" fill="var(--bs-secondary-bg)"/><text x="50%" y="50%" fill="var(--bs-secondary-color)" dy=".3em">500x500</text></svg>
        </div>
        </div>
        <hr class="featurette-divider">
        <div class="row featurette">
        <div class="col-md-7">
            <h2 class="featurette-heading fw-normal lh-1">And lastly, this one. <span class="text-body-secondary">Checkmate.</span></h2>
            <p class="lead">And yes, this is the last block of representative placeholder content. Again, not really intended to be actually read, simply here to give you a better view of what this would look like with some actual content. Your content.</p>
        </div>
        <div class="col-md-5">
            <svg class="bd-placeholder-img bd-placeholder-img-lg featurette-image img-fluid mx-auto" width="500" height="500" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Placeholder: 500x500" preserveAspectRatio="xMidYMid slice" focusable="false"><title>Placeholder</title><rect width="100%" height="100%" fill="var(--bs-secondary-bg)"/><text x="50%" y="50%" fill="var(--bs-secondary-color)" dy=".3em">500x500</text></svg>
        </div>
        </div>
        <hr class="featurette-divider">
        <!-- /END THE FEATURETTES -->
    </div>
@endsection
```

Dentro do menu `nav` temos uma classe chamada `active` que faz com que o menu atual fique aceso. Vamos ativar o menu de acordo com a rota:

```html
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav me-auto mb-2 mb-md-0">
            <li class="nav-item {{ (Route::current()->getName() === 'site.home' ? 'active' : '') }}">
              <a class="nav-link " aria-current="page" href="{{ route('site.home') }}">Home</a>
            </li>
            <li class="nav-item {{ (Route::current()->getName() === 'site.courses' ? 'active' : '') }}">
              <a class="nav-link " href="{{ route('site.courses') }}">Cursos</a>
            </li>
            <li class="nav-item {{ (Route::current()->getName() === 'site.contact' ? 'active' : '') }}">
              <a class="nav-link " href="{{ route('site.contact') }}">Contato</a>
            </li>
          </ul>
        </div>
```

`{{ (Route::current()->getName === 'site.home' ? 'active' : '') }}`

Se o nome da rota atual foi igual a site.home, coloca a classe `active`, caso o contrário, não faz nada.
