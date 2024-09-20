
Vamos executar um comando que dropa todas as tabelas do banco de dados e cria de acordo com as migrations que tem dentro do projeto:

`php artisan migrate:fresh`

Vamos criar uma tabela posts:

`php artisan make:migration create_posts_table`

Definir os campos da tabela

```php
    public function up(): void
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->string('slug');
            $table->string('subtitle');
            $table->text('content');
            $table->timestamps();
        });
    }
```

Executar a migration:

`php artisan migrate`

Vamos começar pelas rotas, para poder criar um formulário e poder persistir os dados dentro do banco de dados e depois voltamos a trabalhar com o banco para ver o que é a personalização que precisamos ter dentro do modelo. 

==Antes de criar as rotas, devemos criar o controlador==

`php artisan make:controller PostController`

Criando rota 

```php
Route::get('/', [PostController::class, 'showForm']);
```

O *showForm* vai devolver uma visão com um formulário para que possamos cadastrar um novo artigo dentro da  base de dados.

Após ter criado a rota, vamos criar o método:

```php
class PostController extends Controller
{
    public function showForm(){
    }
}
```

Criar a view *form.blade.php*

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>

    <form action="">
        @csrf
        <label for="">Título</label>
        <input type="text" name="title">
        <br>

        <label for="">Subtítulo</label>
        <input type="text" name="subtitle">
        <br>

        <label for="">Descrição</label>
        <textarea name="content" cols="30" rows="10"></textarea>
        <br>

        <input type="submit" value="Cadastrar Artigo">
    </form>

</body>
</html>
```

Vamos criar uma rota post `/debug` :

```php
Route::post('/debug', [PostController::class, 'debug']);
```

Criar o método debug e retornar a view form no método showForm:

```php
    public function showForm() {
        return view('form');
    }

    public function debug() {

    }
```

Precisamos passar na action a rota responsável por pegar os dados. Vamos dar um nome para a rota de debug

```php
Route::post('/debug', [PostController::class, 'debug'])->name('debug');
```

Passar a rota para a action no form:

```html
<form action="{{ route('debug') }}" method="POST">
```

Pegar os dados enviados utilizando o Request e testar se estão chegando:

```php
    public function debug(Request $request) {
        dd($request->all());
    }
```

Enfim, chegou o momento de persistir os dados no banco de dados

#### Criando o Modelo

`php artisan make:model Post`

Para saber se precisamos criar um modelo ou não, basta pensar se existe uma tabela no banco de dados para isso. ==Se tiver uma tabela, então _provavelmente_ precisará de um modelo.==

#### Personalizações do Modelo:

O Laravel já identifica qual é a tabela, porém, é recomendado indicarmos a tabela:

```php
protected $table = 'posts';
```

Quando olhamos a saída usando o `dd($request->all())` vimos que um token está sendo passado.
Não queremos persistir esse token no banco de dados, pois se mandarmos ele para dentro do modelo, não teremos um campo chamado `_token` para armazená-lo. Logo, precisamos removê-lo.

No controller, usamos um `except` para listar tudo, menos o `_token`: 

```php
    public function debug(Request $request) {
        dd($request->except(['_token']));
    }
```

Da mesma forma que temos o `except`, temos o `only` para informar apenas os campos que queremos receber.

Vamos tentar persistir os dados usando o `create`, passando um *array associativo* contendo 3 posições.

```php
    public function debug(Request $request) {

        $post = new Post();
        $post->create($request->except(['_token']));
    }
```

Ao tentar persistir os dados, retornará o seguinte erro:

`Add [title] to fillable property to allow mass assignment on [App\Models\Post].`

Trata-se de um mecanismo de segurança do Laravel, pois nada impede que o usuário troque o nome do input e passe um nome qualquer de outro campo que possa existir dentro da base de dados e tente persistir os dados no banco. 

==A propriedade `fillable` é um mecanismo de segurança fornecido pelo Laravel.== 
É necessário criar uma propriedade e informar quais os campos podem ser  passados através do array associativo.

Para contornar essa falha de segurança, podemos usar outra metodologia para persistir os dados ou setar as propriedades de `fillable` no modelo.

#### Setando o fillable:

Vamos criar uma propriedade chamada `$fillable`
Ela receberá um array com todas as posições que você pode passar através do _mass assignment_, que é a técnica de passar um _array associativo_. 

```php
    protected $fillable = [
                            'title',
                            'subtitle',
                            'content',
                        ];
```

Não precisamos passar o `slug`, pois ele não virá do formulário e sim através do método que iremos configurar.

Precisamos  também setar o campo slug, senão dará erro

Para isso, vamos pegar o conteúdo dentro de `title` e transformar em uma *url amigável* e persistir dentro de um outro campo.
Dentro do modelo, conseguimos segmentar tudo isso, ficando dentro de um mesmo arquivo. Logo, se fizermos qualquer tipo de alteração, não é necessário ficar vasculhando a aplicação para saber onde está.

Vamos criar o método `setTitleAttribute()` :

```php
    public function setTitleAttribute($value) {

    }
```

Esse método será disparado automaticamente pelo Laravel toda vez que tentarmos setar um valor em `title`

```php
use Illuminate\Support\Str;

    public function setTitleAttribute($value) {

        $this->attributes['title'] = $value;
        $this->attributes['slug'] = Str::slug($value);
    }
```

O `$this->attributes['title'] = $value;` é o comportamento padrão. É o que acontece toda vez que você seta um novo valor dentro de um atributo do seu modelo. 
Temos o slug que segue a mesma regra e que será persistido quando setarmos um novo `title`.

Vamos usar o `Str::slug` pegando do `Illuminate\Support\Str`.

No Laravel, a função `Str::slug` é usada para gerar uma versão amigável para URLs de uma string. Isso é particularmente útil para criar "*slugs*" para URLs, que são versões de texto limpo e legível de um título ou nome.
Por exemplo, se `$value` for `"Título do Artigo"`, `Str::slug($value)` retornará `"titulo-do-artigo"`.

 Quando chamarmos o `post->title` o método `setTitleAttribute` será disparado automaticamente.
 ==Obs: é importante manter o padrão `set + nome do atributo + Attribute` (obedecendo o case sensitive)==
 
Ao executar o método, ele vai setar as duas propriedades automaticamente e terminar a execução dela.

Com isso, podemos converter datas, moedas, verificar se algum parâmetro é aceito ou não.
Por exemplo, você tem uma coluna chamada sexo que aceita masculino e feminino. Se a pessoa informar qualquer valor diferente desses, você sabe que deu problema na aplicação, ou que ela está agindo de forma má intencionada. Nesse caso, você consegue barrar a execução ou não setar o parâmetro.
Se você tiver um campo que não aceita valor negativo, é possível colocar essa regra de negócio dentro desse atributo.

#### Passos para fazer a persistência no Banco de Dados

1.  Instanciar o modelo;
2.  Setar os atributos;
3.  Salvar.

#### Podemos persistir os dados no Banco de duas formas diferentes:

#####  Mass Assignment:

- Utiliza um array associativo dentro do método `create()`.
- Com isso, conseguimos persistir os dados com duas linhas: uma com a instância do modelo e a outra com o método create:

```php
    public function debug(Request $request) {

        $post = new Post();
        $post->create($request->except(['_token']));
    }
```

##### Instanciando os objetos e passando os atributos:

```php
    public function debug(Request $request) {
        $post = new Post();
        $post->title = $request->title;
        $post->subtitle = $request->subtitle;
        $post->content = $request->content;
        $post->save();
    }
```

Vai depender da situação. Caso você tenha um formulário com dezenas de campos, talvez seja melhor utilizar o _Mass Assignment_.
Se for um formulário mais simples com poucos campos, é aconselhável **instanciar o objeto e seus atributos**, pois fica muito mais legível e melhor para dar uma manutenção.
Quando você persiste pelo _Mass Assignment_, não é possível ver quais campos estão sendo informados. Logo, é interessante poder ver quais são os campos.

Aqui, podemos ver [[Formas de persistir dados em um Banco de dados em Laravel]]

#### Desabilitar o timestamps

```php
public $timestamps = false;
```


#### Automatizando a criação do Modelo com o Artisan

`php artisan make:model NomeDaModel -mcr`

`-m`: cria a migration
`-cr`: cria o Controller com os resources

Agora, só precisamos criar as rotas e as devidas ações dentro do controlador.



