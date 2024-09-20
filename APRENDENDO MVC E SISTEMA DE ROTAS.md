
#### Diretório vendor:
- Não podemos mexer, pois tudo que está dentro deste diretório é uma dependência do projeto e o código que está nele pertence a parte de componentes da nossa aplicação. Portanto, se sair uma nova versão do componente e você editar algum arquivo dentro dele, essas alterações serão perdidas.
- Dentro dele há diversos diretórios onde cada um deles resolve determinada etapa da aplicação.

#### FUNCIONAMENTO DO FRAMEWORK:

- Precisamos entender como funciona o sistema de rotas e o MVC

##### **Fluxo de informação dentro do laravel:**
 
A pessoa está acessando o navegador e vai disparar uma ação para uma determinada url (home, /contato, /sobre). Essa ação será capturada pelo arquivo web.php, que está dentro do diretório routes, e dele será disparado para um controlador, caindo dentro de uma camada MVC. 

##### MVC:
Model, View e Controler. Cada um tem a sua devida responsabilidade dentro de uma aplicação.
##### -  Controller:
- tem a responsabilidade de receber as informações vindas da rota. Ele é o único que consegue conversar tanto com o Modelo quanto com a Visão. Basicamente, o Controller faz o "meio de campo" entre as outras duas camadas.
##### - Modelo: 
- é a camada responsável pelo acesso direto ao banco de dados. O modelo, dentro do Laravel, é feito com PDO, então ele compreende vários bancos de dados. Possui um componente chamado **Eloquent** que ajuda no processo de criar um registro, deletar, editar, fazer relacionamentos.

Quando a informação volta do modelo para o controlador , este chama uma **VISÃO**.

##### - Visão:  
- No laravel, temos a template engine chamada *Blade* (ver [[Diretivas do Blade]]), que permite que você use variáveis, loops, instruções condicionais e outros recursos PHP diretamente em seu código HTML. ==Um dos principais benefícios do Blade é a organização modular do código==. O Blade ajuda a organizar seu código em módulos reutilizáveis que você pode adicionar, remover ou atualizar facilmente sem afetar o restante do aplicativo.
- Assim que temos a visão renderizada, ela é devolvida para o navegador.

##### Onde fica o MVC no Laravel?

- **Controller:** app > Controller
- **Model** - app > models
- **View** - resources > views
A visão fica em um diretório a parte porque tudo que está contido nela **precisa estar público** para que os visitantes da página consigam acessar.

##### Criando um Controller:

``php artisan make:controller NomeDaController`` 

A ferramenta *artisan* nos auxilia em vários processos conforme precisamos criar determinados recursos dentro do laravel, como criar controller, model, migration. ==É interessante usar o artisan para criar recursos, pois ele já fornece uma estrutura.== Por exemplo, ao criar um controller, ele já vem com o namespace definido, use do *Request*, *extends* da classe Controller e o arquivo é colocado dentro do diretório correto. 

Ex: criando um controller chamado UserController

`php artisan make:controller UserController`

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    public function listUser()
    {
        echo "<h1>Listagem de Usuário</h1>";
    }

}
```

Agora que temos um controlador e um método, precisamos fazer com que a rota chegue até ele:

```php
Route::get('listagem-usuario', [UserController::class, 'listUser']);
```

#### Sintaxe básica para criação de rotas:

`Route::verbo('url', [Controller:class, 'nome_do_método'])->name('nome_da_rota');`

*OBS: As rotas nomeadas permitem a geração conveniente de URLs ou redirecionamentos para rotas específicas.*

Quando alguém digitar listagem-usuario dentro da URL do projeto, vai para UserController dentro de um método que se chama listUser e nele, temos simplesmente um `echo` dentro de uma tag `<h1>`.

Precisamos **parametrizar** o laravel para que ele tenha acesso ao Banco de dados.

- O arquivo `.env` é responsável pelas configurações do laravel:

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel-tips
DB_USERNAME=root
DB_PASSWORD=
```

Em DB_CONNECTION definimos qual é o driver e em DB_DATABASE o nome do banco. Após criar o banco, vamos rodar o comando migrate:

`php artisan migrate`

Ele irá rodar algumas migrações, fará alguns processos no banco de dados para que ele possa criar determinadas tabelas.

O laravel já vem com uma Model User, que por padrão define a tabela com algumas colunas para formar uma tabela de criação de usuário.

Agora, o controlador precisará conversar com a camada do Modelo

```php
    public function listUser()
    {
        $user = new User();
        $user->name = 'Example';
        $user->email = 'example@example.com.br';
        $user->password = Hash::make('123');
        $user->save();

        echo "<h1>Listagem de Usuário</h1>";
    }
```

Criamos o objeto User e populamos com dados.

Vamos criar uma view para listUser com o nome de listUser.blade.php

```php
<h1>Listagem de Usuário</h1>

<p>O nome do usuário é: {{ $user->name }} e ele tem o e-mail {{ $user->email}}</p>
```

Para que a variável $user esteja disponível, precisamos que o controlador mande essa variável para a view:

```php
    public function listUser()
    {
        $user = User::where('id', 1)->first();
        dd($user);
        return view('listUser');
    }
```

Fazemos uma consulta pelo usuário com o id = 1 e queremos pegar a primeira ocorrência. O dd (*dump and die)* mostra o objeto.

```php
    public function listUser()
    {
        $user = User::where('id', 1)->first();
        return view('listUser', [
                                    'user' => $user
                                ]);
    }
```

Para mandar o objeto para a visão, na hora de retornar a view passamos através de um vetor, pois um vetor permite mandar inúmeras informações. No vetor, mandamos a posição 'user' e como valor dela a variável $user.
A variável $user é referente ao
`$user = User::where('id', 1)->first();`
==E o nome 'user' será o nome da variável que será utilizada dentro da visão==

Resultado:

![[Pasted image 20240819115007.png]]

##### Arquivos que foram modificados e o que foi preciso fazer:

- Arquivo web.php (rota), UserController.php (controller) e o listUser.blade.php (view) 
- Na saída, temos uma saída simples HTML;
- Dentro do controlador, temos um método simples onde definimos o usuário e utilizamos; algumas ferramentas que o laravel já tem;
- Dentro do arquivo web, definimos a rota.