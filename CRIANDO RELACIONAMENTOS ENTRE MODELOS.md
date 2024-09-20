
Vamos criar 3 modelos automatizados com o artisan para fazer alguns relacionamentos

`php artisan make: model Address -mcr`
`php artisan make:model Category -mcr`
`php artisan make:model Post -mcr`

#### Relacionamento 1:1

1 usuário só terá 1 endereço

Migration `create_address_table`:

```php
    public function up(): void
    {
        Schema::create('addresses', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('user');
            $table->string('street');
            $table->string('number');
            $table->string('city');
            $table->string('state');
            $table->timestamps();

            $table->foreign('user')->references('id')->on('users')->onDelete('cascade');

        });
    }
```

Criamos 2 usuários teste no banco de dados

Agora, vamos disparar uma rota para que possa disparar uma ação para um controlador e cair dentro de um método e assim podermos ver essas informações na tela.

Antes de criar a rota, vamos criar um controlador para User

`php artisan make:controller UserController --resource`

Criar a rota:

```php
Route::get('/usuario/{id}', [UserController::class, 'show']);
```

Criamos uma rota com a URL `/usuario/{id}` que passa um parâmetro. O método será responsável por resgatar esse parâmetro e vamos colocar a saída na tela dentro do próprio controlador.

No método `show`, precisamos fazer a pesquisa dentro do banco de dados para retornar o usuário.

```php
    public function show(string $id)
    {
        $user = User::where('id', $id)->first();
        dd($user);
    }
```

Como estamos pegando apenas um objeto, vamos usar o método `first`

```php
    public function show(string $id)
    {
        $user = User::where('id', $id)->first();
        // Se conseguiu listar um usuário
        if($user) {
            echo "<h1> Dados do usuário</h1>";
            echo "<p>Nome: {$user->name} E-mail: {$user->email}</p>";
        }
    }
```

Model Address:

```php
class Address extends Model
{
    use HasFactory;
    
    protected $table = 'addresses';
}
```

Passar o nome da tabela
Se tivéssemos um formulário, poderíamos passar o parâmetro `fillable`, mas para agilizar o processo, vamos fazer o cadastro diretamente no banco de dados.

Vamos inserir um endereço de teste na tabela Adresses

#### Criando o relacionamento 1:1

##### Caminho de ida:

Vamos pegar a tabela de origem do relacionamento (no caso, a tabela users). O usuário possui o endereço. O endereço sozinho não faz sentido, pois não sabemos de quem é ou o que fazer com ele.

Model User:

```php
    public function address()
    {
        return $this->hasOne(Address::class, 'user', 'id');
    }
```

- **`$this->hasOne()`**: Este é um método fornecido pelo Eloquent para definir um relacionamento de "um-para-um" entre dois modelos. Isso significa que um registro no modelo atual está relacionado com exatamente um registro no modelo `Address`.

- **`Address::class`**: Refere-se à classe do modelo `Address`. Isso indica qual modelo está relacionado com o modelo atual.

- **`'user'`**: Este é o nome da coluna na tabela `addresses` (ou o nome do campo na tabela do banco de dados que está associado ao relacionamento). Esta coluna é usada para armazenar a referência ao modelo atual. Em outras palavras, é a chave estrangeira que estabelece a relação entre `Address` e o modelo atual.

- **`'id'`**: Este é o nome da coluna na tabela do modelo atual (o modelo onde este método está definido). É a chave primária da tabela do modelo atual e será comparada com a chave estrangeira para encontrar a relação correta.

#### Exibir o relacionamento

Apenas para fim de teste, vamos exibir o relacionamento criado na controller

```php
$address = $user->address()->first();

        if($address) {
            echo "<h1>Endereço:</h1>";
            echo "Endereço Completo: {$address->street}, {$address->number} {$address->city}/{$address->state}";
        }
```

##### Caminho de volta:

Criando a rota

```php
Route::get('/endereco/{address}', [AddressController::class, 'show']);
```

AddressController:

```php
    public function show(Address $address)
    {
        if($address) {
            echo "<h1>Endereço:</h1>";
            echo "Endereço Completo: {$address->street}, {$address->number} {$address->city}/{$address->state}";
        }
    }
```

Há uma injeção de dependência, logo quando passamos um parâmetro dentro da variável $address o Laravel já consegue fazer um bind dos parâmetros e devolver um modelo pronto
O objeto $address já está alimentado
Esse parâmetro tem que estar igual ao passado pela rota, então colocamos `/endereco/{address}`

Modelo Address:

```php
    public function user() {
        return $this->belongsTo(User::class, 'user', 'id');
    }
```

O caminho de volta se chama `belongsTo`.
Essa parte de nomenclatura desses métodos é simplesmente por **semântica**. Se você quisesse utilizar o `hasOne` também conseguiria fazer o relacionamento.

Vamos pensar o seguinte:

- Um usuário **tem** um endereço
- Um endereço **pertence** a um usuário

Porém, dizer que um usuário pertence a um endereço, não faz sentido algum. 

O primeiro parâmetro é o modelo que irá se relacionar: `User::class`;
O segundo parâmetro é a coluna que contém a chave estrangeira: `'user'`;
O terceiro parâmetro é a coluna que contém a chave local: `'id'`.

#### Testando o relacionamento:

AddressController:

```php
    public function show(Address $address)
    {
        if($address) {
            echo "<h1>Endereço:</h1>";
            echo "Endereço Completo: {$address->street}, {$address->number} {$address->city}/{$address->state}";
        }

        $user = $address->user()->first();

        if($user) {
            echo "<h1> Dados do usuário</h1>";
            echo "<p>Nome: {$user->name} E-mail: {$user->email}</p>";
        }
    }
```


#### Criando o relacionamento 1:N

Vamos adicionar algumas colunas na tabela `posts`:

`php artisan make:migration posts --table=posts`

```php
    public function up(): void
    {
        Schema::table('posts', function (Blueprint $table) {
            $table->unsignedBigInteger('user');
            $table->string('title');
            $table->string('slug');
            $table->string('content');
        });

    }

    /**

     * Reverse the migrations.

     */

    public function down(): void
    {
        Schema::table('posts', function (Blueprint $table) {
            $table->dropColumn(['user', 'title', 'slug', 'content']);
        });
    }
```

Vamos popular essa tabela com dados testes diretamente do banco de dados

##### Parametrizando os Modelos

Model User:

```php
    public function posts() {
        return $this->hasMany(Post::class, 'user', 'id');
    }
```


UserController:

```php
    public function show(string $id)
    {
        $posts = $user->posts()->get();

        if($posts) {
            echo "<h1>Artigos: </h1>";
            foreach($posts as $post) {
                echo "<p>#{$post->id} {$post->title}, {$post->content}</p>";
            }
        }
    }
```

`get()`: como são vários objetos, usamos o `get`.
Para pegar cada post, precisaremos de um `foreach`.

##### Caminho de volta:

Dentro do artigo, queremos saber quem é o dono dele.

Model Post

```php
class Post extends Model
{
    use HasFactory;

    protected $table = 'posts';

  
    public function author()
    {
        return $this->belongsTo(User::class, 'user', 'id');
    }
}
```

PostController

```php
    public function show(Post $post)
    {
        echo "<h1>Artigo:</h1>";
        echo "<p>#{$post->id} {$post->title}, {$post->content}</p>";
    }
```

Criando a rota

```php
Route::get('/artigo/{post}', [PostController::class, 'show']);
```

*Obs: importante lembrar que o nome do parâmetro enviado pela URL precisa ser o mesmo do parâmetro passado por injeção de dependência no método do controlador.*

Agora, vamos mostrar o usuário(autor) dono do post:

PostController:
```php
    public function show(Post $post)
    {
        echo "<h1>Artigos:</h1>";
        echo "<p>#{$post->id} {$post->title}, {$post->content}</p>";

        $user = $post->author()->first();

        if($user) {
            echo "<h1>Autor:</h1>";
            echo "<p>Nome: {$user->nome} E-mail {$user->email}</p>";
        }
    }
```


#### Relacionamento N:N

Vamos fazer o relacionamento de Posts e Categorias, ou seja, vários posts podem estar em várias categorias. O relacionamento é o mesmo nas duas pontas, diferente dos anteriores que possuem um relacionamento para a ida e outro para a volta.

Preciamos ter uma *tabela pivô* para fazer a união entre esses registros:
`artisan make:migration create_posts_categories_table`

Migration:

```php
    public function up(): void
    {
        Schema::create('posts_categories', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('post');
            $table->unsignedBigInteger('category');
            $table->timestamps();

            $table->foreign('post')->references('id')->on('posts');
            $table->foreign('category')->references('id')->on('categories');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('posts_categories');
    }
```


Vamos inserir campos na tabela e criar algumas categorias diretamente no banco de dados

`php artisan make:migration categories --table=categories`

```php
    public function up(): void
    {
        Schema::table('categories', function (Blueprint $table) {
            $table->string('title');
            $table->string('description');
        });
    }

    /**

     * Reverse the migrations.

     */

    public function down(): void

    {
        Schema::table('categories', function (Blueprint $table) {
            $table->dropColumn(['title', 'description']);
        });
    }
```

#### Criando o relacionamento:

Vamos popular alguns dados na _tabela pivô_ no banco.
Post 1 pertencendo a categoria 1; Post 2 pertencendo a categoria 2; Post 1 pertencendo a categoria 1; Post 3 pertencendo a categoria 2.

```php
class Post extends Model

{
    use HasFactory;

    protected $table = 'posts';


    public function categories() {
        return $this->belongsToMany(Category::class, 'posts_categories', 'post', 'category');
    }
}
```

`belongsToMany` o segundo parâmetro pede para passar a *tabela pivô*
O terceiro parâmetro pede para passar qual campo do modelo atual está sendo relacionado com a tabela pivô. No caso, Post é relacionado com o campo `post`.
o quarto parâmetro é o outro campo da tabela pivô. No caso, o campo `category`.

PostController

```php
    public function show(Post $post)
    {
        $categories = $post->categories()->get();

        if($categories) {
            echo "<h1>Categorias:</h1>";
            foreach($categories as $category) {
                echo "<p>#{$category->id} {$category->title}</p>";
            }
        }
    }
```


#### Caminho de volta N:N

Criando a rota 

```php
Route::get('/categoria/{category}', [CategoryController::class, 'show']);
```

CategoryController

```php
    public function show(Category $category)
    {
        echo "<h1>Categoria:</h1>";
        echo "<p>#{$category->id} {$category->title}</p>";
    }
```

Model Category

```php
class Category extends Model
{
    use HasFactory;

    protected $table = 'categories';

    public function posts() {
        return $this->belongsToMany(Post::class, 'posts_categories', 'category', 'post');
    }
}
```

É o mesmo código da ida. O que muda é a ordem dos parâmetros.

Mostrando todos os artigos no CategoryController:

```php
    public function show(Category $category)
    {
        $posts = $category->posts()->get();

        if($posts) {
            echo "<h1>Artigos:</h1>";
            foreach($posts as $post) {
                echo "<p>#{$post->id} {$post->title}, {$post->content}</p>";
            }
        }
    }
```

