Resumidamente, para fazer um upload precisamos:
- Criar uma `rota`
- Criar uma visão com um `formulário`
- O formulário precisa ser disparado através de um post e com algumas particularidades: ter um `csrf`, um `enctype = multipart/form-data `(para poder mandar um arquivo)
- Resgatar essas informações dentro de um `controlador`

### Criando a visão

Vamos criar uma pasta `upload` dentro da `views` um arquivo `form.blade.php`

```php
<form action="" method="post" enctype="multipart/form-data">
    @csrf
    <input type="file" name="arquivo">
    <button type="submit">Enviar arquivo</button>
</form>
```

### Criando a rota:

```php
Route::view('form', 'upload.form');
```

O método `Route::view` é uma maneira prática e rápida de definir rotas que retornam uma visualização específica (view) sem a necessidade de um controlador. Isso é útil para rotas que apenas servem um arquivo de visualização e não requerem lógica adicional.

### Criando o controlador:

`php artisan make:controller UploadController`

```php
class UploadController extends Controller
{
    public function upload() {
    }
}
```

### Rota POST 

```php
Route::post('upload', [UploadController::class, 'upload'])->name('upload');
```

Colocar o `action` no `form` para disparar a rota `upload`

```html
<form action="{{ route('upload') }}" method="post" enctype="multipart/form-data">
```

### Checklist da parte da visão

- Rota para a view ✅
- Rota para o post  ✅
- Token alimentado com csrf ✅
- Form com enctype ✅
- Form com input do tipo file com um nome ✅

Ainda não temos pronto **quem está recebendo as informações**, pois precisamos poder _debugar_ os dados. 

### Debug - Método upload no Controller

```php
    public function upload(Request $request) {
        dd($request->all());
    }
```

Vamos enviar um arquivo para ver se está funcionando normalmente

`"arquivo" => Illuminate\Http\UploadedFile`

O arquivo está chegando como `UploadedFile`. Uma vez que o Laravel sabe que o objeto é um arquivo, ele permite que você tenha algumas propriedades adicionais.

Conseguimos fazer testes para saber se o arquivo:
 - é de um tamanho específico
 - é de uma extensão específica
 - se conseguimos movê-lo para um diretório ou não
 - se temos permissão de leitura ou escrita
 - entre outras propriedades

A posição `arquivo` (name) é a posição `input` do formulário.
Podemos acessar essa posição diretamente do controlador. 

```php
class UploadController extends Controller
{
    public function upload(Request $request) {
        dd($request->file('arquivo'), $request->all());
    }
}
```

Para fazer upload do arquivo, que até o momento encontra-se dentro de uma pasta temporária, para dentro do projeto, basta invocar um método chamado `store()`.

```php
class UploadController extends Controller
{
    public function upload(Request $request) {
        $request->file('arquivo')->store('teste');
        dd($request->file('arquivo'), $request->all());
    }
}
```

No método `store` passamos o `path` para onde o arquivo irá. 

### Diretório Storage

Quando mandamos um arquivo via FTP, o Laravel salva no diretório `storage`.
Esse diretório é comandado pelo arquivo de configuração `filesystems.php`

### filesystem.php

Localização: `config > filesystems.php`

Dentro desse arquivo, temos a chave `default`:
	`'default' => env('FILESYSTEM_DISK', 'local'),`

Se ela não existir, pegará a `local`
Para saber se a chave existe ou não, copiamos o `FILESYSTEM_DISK` e procuramos no arquivo `.env`

Como é a chave local que está sendo utilizada, temos as seguintes configurações:

```php
    'disks' => [  
        'local' => [
            'driver' => 'local',
            'root' => storage_path('app'),
            'throw' => false,
        ],
```
Por padrão, quando fazemos a instalação do Laravel, utiliza-se essas configurações.

No entanto, o `local` não resolve o problema, pois quando está `local`, encontra-se apenas em uma máquina. 
Precisamos que seja **público**. Assim, quando fizermos o _upload_ de uma mídia, se tiver hospedado dentro do nosso servidor, todos terão acesso.

Para isso, vamos no arquivo `.env`
	`FILESYSTEM_DISK=public`
Quando fazemos isso, o Laravel é capaz de identificar que agora a chave existe e o valor dela é público. Logo, os dados que ela tem que pegar são os que estão em:

```php
        'public' => [
            'driver' => 'local',
            'root' => storage_path('app/public'),
            'url' => env('APP_URL').'/storage',
            'visibility' => 'public',
            'throw' => false,
        ],
```

`'root' => storage_path('app/public'),` agora, os arquivos vão para dentro do diretório `storage > app > public`

No controller, definimos que o arquivo irá para o diretório `store`:
```php
 $request->file('arquivo')->store('teste');
```

Logo, o arquivo será criado em `storage > app > public > test`.

Terminamos o passo-a-passo do upload de apenas um arquivo. Agora, veremos como fazer o **upload de múltiplos arquivos**.

## Upload de múltiplos arquivos:


### Configurando o DB no .env:

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel-tips
DB_USERNAME=root
DB_PASSWORD=
```

### Criando as migrations

Vamos criar duas migrations: 

`php artisan make:migration create_products_table`
```php
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->timestamps();
        });
    }
```

`php artisan make:migration create_products_images_table`

```php
    public function up(): void
    {
        Schema::create('products_images', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('product');
            $table->string('path');

            $table->foreign('product')->references('id')->on('products');
        });
    }
```

`php artisan migrate`

Criar as models e os relacionamentos:

`php artisan make:model Product`
```php
class Product extends Model
{
    use HasFactory;
	protected $table = 'products';

    public function images()
    {
        return $this->hasMany(ProductImage::class, 'product', 'id');
    }
}
```

`php artisan make:model ProductImage`
```php
class ProductImage extends Model

{
    use HasFactory;
    
    protected $table = 'products_images';
    public $timestamps = false;
    protected $fillable = ['product', 'path'];

    public function product() {
        return $this->belongsTo(Product::class, 'product', 'id');
    }
}
```


#### Criando ProductController:

`php artisan make:controller ProductController --resource`

ProductController:

```php
    public function create()
    {
        return view('product.create');
    }
```

Vamos criar duas views para `product`: 
Dentro do diretório product, vamos criar `create.blade.php` e `show.blade.php`

### Route::resource:

No Laravel, a função `Route::resource` é uma maneira prática de criar um conjunto completo de rotas para um controlador de recursos com base nos métodos padrões de um controlador RESTful.

```php
Route::resource('produtos', ProductController::class)->names('products')->parameters(['produtos' => 'products']);
```


A rota definida com `Route::resource` neste exemplo faz o seguinte:

- Cria rotas RESTful para o controlador `ProductController` com a base da URL sendo `'produtos'`
- Personaliza o prefixo dos nomes das rotas para `'products'` (por exemplo, `'products.index'`).
- Altera o nome do parâmetro de rota que representa o identificador do recurso de `'produtos'` para `'products'` (por exemplo, `/produtos/{products}`).

Vamos executar o comando `php artisan route:list` para verificar se as rotas foram criadas com êxito.
Saída:

``` 
  GET|HEAD        form .................................................................... 
  GET|HEAD        produtos ....................... products.index › ProductController@index 
  POST            produtos ....................... products.store › ProductController@store 
  GET|HEAD        produtos/create .............. products.create › ProductController@create 
  GET|HEAD        produtos/{products} .............. products.show › ProductController@show 
  PUT|PATCH       produtos/{products} .......... products.update › ProductController@update 
  DELETE          produtos/{products} ........ products.destroy › ProductController@destroy 
  GET|HEAD        produtos/{products}/edit ......... products.edit › ProductController@edit 
  GET|HEAD        up ...................................................................... 
  POST            upload ................................. upload › UploadController@upload 
```

Passando o `names` e o `parameters` o parâmetro da URL fica em inglês (`{products}`), o nome da rota fica em inglês  (`products.index`) e os métodos já estão normalizados (`index, store, create, etc`);

### Criando as views

Vamos criar o diretório `products` e dentro dele criar as views `create.blade.php` e `show.blade.php`

`create.blade.php`
```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    @vite(['resources/sass/app.scss', 'resources/js/app.js'])
</head>

<body>
    <div class="container mt-5">
        <form action="{{ route('products.store') }}" class="p-5 bg-white rounded" enctype="multipart/form-data" method="POST">
            @csrf
            <div class="form-group">
                <label for="title">Título do Produto</label>
                <input type="text" class="form-control" name="title" id="title">
            </div>

            <div class="custom-file mb-3">
                <input type="file" class="custom-file-input" name="images[]" id="customFileLangHTML" multiple>
                <label class="custom-file-label" for="customFileLangHTML" data-browse="Enviar Imagens"></label>
            </div>

            <button type="submit" class="btn btn-primary">Cadastrar Produto</button>
        </form>
    </div>
</body>

</html>
```

### Validar o método Store:

```php
    public function store(Request $request)
    {
        dd($request->all());
    }
```

Vamos tentar cadastrar um produto e ver se ele está chegando pelo `request`.

Observe que na saída temos o parâmetro `images`. No formulário, o `name` do `input` é `images[]`. Passar o array no nome é importantíssimo para que possamos fazer upload de múltiplas mídias. Afinal, o `[ ]` ao final do `name` fará com que seja criado um vetor a cada novo arquivo que seja enviado.
Também é importante que coloque a tag `multiple` para poder selecionar mais de um arquivo.


### Persistência no Banco de Dados

A tabela produto só contém o campo `title`. Vamos instanciar um produto, pegar o `title` pelo request e salvar os dados.

```php
    public function store(Request $request)
    {
        dd($request->all());

        $product = new Product();
        $product->title = $request->title;
        $product->save();
    }
```

Agora, vamos para a parte de **mídias**.
Quando upload das mídias for realizado, precisamos fazer um laço de repetição para que possamos saber quantas mídias existem e fazer com que o laço percorra a mesma quantidade de vezes correspondente ao número de mídias. Depois, criar um objeto, persistí-lo no banco de dados, movê-lo para dentro do diretório e limpá-lo no final.

```php
    public function store(Request $request)
    {
        dd($request->all());

        $product = new Product();
        $product->title = $request->title;
        $product->save();

        for($i = 0; $i < count($request->allFiles()['images']); $i++) {
            dd($i);
        }
    }
```

Existem diversas técnicas para persistir uma mídia: você pode instanciar vários objetos, usar _lazy collection_, etc.

```php
    public function store(Request $request)
    {
        $product = new Product();
        $product->title = $request->title;
        $product->save();

        for($i = 0; $i < count($request->allFiles()['images']); $i++) {
            $file = $request->allFiles()['images'][$i];

            $productImage = new ProductImage();
            $productImage->product = $product->id;
            $productImage->path = $file->store('produtos');
            $productImage->save();
            unset($productImage);
        }
    }
```

`$file = $request->allFiles()['images'][$i];`:  pega o arquivo no índice atual que será armazenado em `store`.
`$productImage->product = $product->id;`: usa o id da instância de produto que foi feita anteriormente.
`unset`: limpa o objeto, pois o `for` será executado novamente e é importante não ter nada dentro da variável.

Para fins de organização, vamos colocar uma pasta `id` para cada produto e dentro dele suas respectivas mídias.
Em `store`, vamos concatenar o `$product->id`.

```php
$productImage->path = $file->store('produtos/' . $product->id);
```

### Implementando a página de visualização (show):

Para fins de _debug_, vamos utilizar o _dump and die_ em `product`

```php
    public function show(Product $product)
    {
        dd($product);
        return view('products.show');
    }
```

Acessar a url para testar `http://127.0.0.1:8000/produtos/4`

Agora, passar o objeto `$product` para a view:

```php
    public function show($id)

    {
        // Buscar o produto pelo ID
        $product = Product::find($id);

        // Verificar se o produto foi encontrado
        if (!$product) {
            abort(404, 'Produto não encontrado');
        }
        
        // Retornar a view com o produto
        return view('products.show', ['product' => $product]);
    }
```

### Exibindo as imagens

Como fizemos o relacionamento na Model, podemos utilizar o `images` após ter enviado `product` para a view

```html
<body class="bg-light">
    <div class="container mt-5">
        <h2>{{ $product->title }}</h2>
        @foreach ($product->images as $image)
            {{ $image->path }}
        @endforeach
    </div>
</body>
```

`{{ $image->path }}`: imprime o path da imagem no html apenas para fins de _debug_.

Agora, dentro do `.env` vamos colocar o caminho da aplicação em `APP_URL`
`APP_URL=http://localhost/laravel-tips/public`

Vamos colocar uma tag `<img>` dentro do `foreach` e passar o helper `env()` para montar o caminho da aplicação + diretório das mídias + caminho da mídia que queremos exibir em `src`

```php
<body class="bg-light">
    <div class="container mt-5">
        <h2>{{ $product->title }}</h2>
        @foreach ($product->images as $image)
        <img src="{{ env('APP_URL') }}/storage/{{ $image->path }}" alt="">
        @endforeach
    </div>
</body>
```

As imagens ainda não serão exibidas, pois a pasta `storage` só existe dentro da aplicação, não existindo na `public`. Para isso, vamos utilizar o comando a seguir:

	`php artisan storage:link`

Esse comando cria um link simbólico, que nada mais é do que um atalho para o diretório `storage` em `public`
==Obs: sempre que você migrar de uma máquina para a outra, você precisa refazer esse link.==