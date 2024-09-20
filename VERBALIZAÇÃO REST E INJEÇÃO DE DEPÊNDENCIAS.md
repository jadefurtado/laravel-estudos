Seguir as verbalizações mantém a aplicação íntegra, pois você sabe tudo que está acontecendo dentro dela e, por mais que você esteja "travando" a aplicação em determinado verbo, é possível evitar que qualquer pessoa possa mandar qualquer verbo para a sua URL e faça ataques maliciosos.

###### No controlador `UserController`, você pode ter métodos como:

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    public function index() { /* Exibir lista de usuários */ }
    public function create() { /* Mostrar formulário para criar um novo usuário */ }
    public function store(Request $request) { /* Salvar novo usuário */ }
    public function show($id) { /* Exibir um usuário específico */ }
    public function edit($id) { /* Mostrar formulário para editar um usuário */ }
    public function update(Request $request, $id) { /* Atualizar um usuário */ }
    public function destroy($id) { /* Excluir um usuário */ }
}

```

==OBS: Quando trabalhamos com o artisan, ele faz uma leitura do código para saber se está tudo ok e se é possível fazer a criação. Caso tenha alguma rota inválida, utilizando um controlador ou método que não existe, pode ser que o artisan não consiga efetuar a leitura que ele precisa e acaba acusando um erro.==

Para evitar erros, vamos **primeiro criar o controlador** antes de criar a rota:

`php artisan make:controller Form/TestController`

Como passamos o \Form antes, ao acessar o diretório app > http > controllers, um diretório Form é criado

##### Criando a rota: 

```php
Route::get('usuarios', [TestController::class, 'listAllUsers'])->name('users.listAll');
```

No TestController, vamos criar o método listAllUsers

```php
    public function listAllUsers()
    {
        $users = User::all();

        dd($users);
    }
```

Todos os usuários serão listados. Para testar, vamos utilizar um *dump and die* passando a varíavel $users.

*Obs: o dump and die é semelhante ao vardump, porém é possível recolher os nós para que não fique tanta saída no navegador.*

Agora que verificamos que todos os usuários estão sendo retornados, precisamos criar uma visão para que possam ir para uma listagem e posteriormente possa ter a ação de visualizar, editar e excluir um usuário.

Criando uma view simples:

```html
<!DOCTYPE html>

<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Listagem de Usuários</title>
</head>
<body>

    <table>
        <tr>
            <td>ID</td>
            <td>Nome:</td>
            <td>E-mail:</td>
            <td>Ações</td>
        </tr>

        <tr>
            <td></td>
            <td></td>
            <td></td>
            <td>
                <a href="">Ver Usuário</a>
                <form action="" method="POST">
                    <input type="hidden" name="user" value="">
                    <input type="submit" value="Remover">
                </form>
            </td>
        </tr>
    </table>
    
</body>
</html>
```

Vamos passar para a view o conteúdo da variável $users com o nome de users

```php
    public function listAllUsers()
    {
        $users = User::all(); 
        return view('listAllUsers', [
								        'users' => $users
							        ]);

    }
```

Na view:
- Vamos testar usando um `@foreach` do *Blade*.
- Basicamente, o @ do *Blade* serve para que você não tenha que ficar abrindo e fechando tag do php e o código acabe ficando mais organizado.

*Obs: isso só pode ser utilizado dentro da template engine, ou seja, apenas para montar o layout do site. Dentro da programação, dentro das classes, você vai utilizar o for each normal.*

```html
    <table>
        <tr>
            <td>ID</td>
            <td>Nome:</td>
            <td>E-mail:</td>
            <td>Ações</td>
        </tr>
        @foreach ($users as $user)
            <tr>
                <td>{{ $user->id }}</td>
                <td>{{ $user->name }}</td>
                <td>{{ $user->email }}</td>
                <td>
                    <a href="">Ver Usuário</a>
                    <form action="" method="POST">
                        <input type="hidden" name="user" value="">
                        <input type="submit" value="Remover">
                    </form>
                </td>
            </tr>
        @endforeach
    </table>
```

##### Ver Usuário:

Precisamos de uma rota, pois estaremos disparando uma nova ação. É necessário informar para a rota qual é o usuário que desejamos ter maiores informações e uma visão.

Ao passar um parâmetro na URL usando chaves, conseguimos recebê-lo através de uma variável dentro do método que estiver desenvolvendo. Logo, como parâmetro do método, teremos uma variável e através dela conseguimos resgatar o que estará sendo informado pela URL. Baseado nisso, conseguimos fazer uma pesquisa no banco de dados para popular a visão de acordo com o que precisamos.

```php
Route::get('usuarios/{user}', [TestController::class, 'listUser'])->name('users.list');
```

Precisamos resgatar o parâmetro {users}

Vamos trabalhar com **injeção de dependência**

Ler sobre injeção de dependência:
https://www.treinaweb.com.br/blog/entendendo-injecao-de-dependencia
  
Vamos passar a variável $user (mesmo nome que está na rota) como parâmetro do método listUser
```php
    public function listUser($user)
    {
        var_dump($user);
    }
```

Acessar a URL referente ao ID do usuário cadastrado:
`http://127.0.0.1:8000/usuarios/1`

Resultado:
`string(1) "1"`

Logo, a string referente ao ID está sendo passada.

No momento, se informarmos qualquer coisa na URL, algo será retornado e precisamos travar o sistema para que ele obedeça a somente uma regra.
O campo que queremos passar é chave primária do modelo. Logo, podemos fazer uma **injeção de dependência** informando diretamente o modelo.

```php
    public function listUser(User $user)
    {
        dd($user);
    }
```

Vamos criar a view e alimentá-la com o usuário que foi devolvido

```php
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Detalhes do Usuário</title>
</head>

<body>
    <h1>{{ $user->name }}</h1>
    <p>{{ $user->email }}</p>
    <p>{{ date('d/m/Y H:i', strtotime($user->created_at)) }}</p>
</body>

</html>
```

*Obs: O campo created at é criado automaticamente pelas migrações* 

##### Cadastrar um usuário:

Precisaremos de duas rotas: 
- Uma rota será responsável por **exibir** o formulário de cadastro do usuário; 
- A outra rota trabalhará com a verbalização POST para que possa disparar as ações. Essa rota chamará o controlador, que vai chamar um método que vai **persistir esses dados** no banco de dados.

Rota para **exibir** o formulário de cadastro do usuário:

```php
Route::get('usuarios/novo', [TestController::class, 'formAddUser'])->name('users.formAddUser');
```

Criar a view para o formulário de cadastro do usuário:

```html
<!DOCTYPE html>

<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Cadastro de Usuário</title>
</head>

<body>

    <form action="" method="POST">
        <label for="">Nome do usuário:</label>
        <input type="text" name="name">

        <label for="">E-mail do usuário:</label>
        <input type="email" name="email">

        <label for="">Senha do usuário</label>
        <input type="password" name="password">

        <input type="submit" value="Cadastrar usuário">
    </form>

</body>
</html>
```

Se tentarmos acessar a rota usuarios/novo, aparecerá um erro 404 not found. Isso acontece porque no arquivo de rotas existe um parâmetro dinâmico `'usuarios/{user}'` e quando cai dentro desse parâmetro, estamos "travando" a partir da model que só pode receber um inteiro e o `/novo` é uma string. Logo, as duas rotas acabam entrando em conflito.
Para resolver este problema, ==todas as rotas que tiverem parâmetros com palavras fixas ficarão acima das rotas que possuem parâmetros dinâmicos.==

Arquivo de rotas:

```php
Route::get('usuarios', [TestController::class, 'listAllUsers'])->name('users.listAll');

Route::get('usuarios/novo', [TestController::class, 'formAddUser'])->name('users.formAddUser');
Route::get('usuarios/{user}', [TestController::class, 'listUser'])->name('users.list');
```

==É muito importante ficar ciente com a maneira na qual você define as rotas.==

Depois que o formulário for preenchido, precisamos disparar para outra rota e agora vamos trabalhar com o verbo _post_.

```php
Route::post('usuarios/store', [TestController::class, 'storeUser'])->name('users.store');
```

Dentro da visão addUser temos um *action* onde iremos passar a rota *post*.
Usamos um *helper* chamado _route_ e informamos o _name_ da rota

```html
<form action="{{ route('users.store') }}" method="POST">
```

*Obs: Quando utilizamos o caminho da URL no action, ele fica fixo no código e caso você precise alterar esse endereço, você precisará repassar por todos os seus arquivos verificando se você não tem URL fixa. Quando você utiliza o **nome das rotas**, o sistema fica totalmente **independente** desses caminhos. O sistema não precisa conhecer o caminho específico da rota, mas sim o nome, pois o nome nunca vamos alterar.*

O Laravel possui um mecanismo de proteção para que nenhuma pessoa externa possa disparar uma ação para dentro da URL (que qualquer um consegue ter acesso) e, juntamente com o verbo *post,* disparar uma ação para fazer um cadastro.
O Laravel implementa um mecanismo de segurança chamado `csrf`. Ou seja, quando você for disparar uma ação para essa rota, você precisa informar um campo oculto que está dentro do formulário e o motor do Laravel vai fazer uma verificação se este (que será um hash criptografado) é compatível com o arquivo que ele estará esperando dentro do servidor. Se for compatível, a ação pode passar. Caso o contrário, a ação resultará em um erro.
O **erro 419** é gerado pela falta do ***csrf token***

Existe um helper do blade que implementa o csrf
Vamos colocá-lo dentro do form

```html
 <form action="{{ route('users.store') }}" method="POST">
     @csrf
```

Um input hidden é gerado com um token
```html
<input type="hidden" name="_token" value="XOTriowedPMgrpD6xXLwx3vSB6uPBd3sg1smFKMZ" autocomplete="off">
```

###### Request:
- O objeto *Request* é usado para ==resgatar informações== 

Dentro do método *store*, vamos passar o objeto Request por **injeção de dependência** para recuperar as informações do formulário.

```php
    public function storeUser(Request $request)
    {
        $user = new User();
        $user->name = $request->name;
        $user->email = $request->email;
        $user->password = Hash::make($request->password);
        $user->save();

        return redirect()->route('users.listAll');

    }
```

Estamos instanciando um novo usuário. O $user (modelo) na propriedade name vai receber o request (que é o formulário da página anterior) também na posição name. O $user->save( ) é usado para persistir as informações no banco de dados e o cadastro estará pronto.
Agora, vamos redirecionar o usuário para a página de listagem usando um redirect e um route para indicar a rota para qual o usuário será redirecionado.

##### Edição:

-  Assemelha-se com a parte de cadastro, porém temos que passar um parâmetro a mais que precisa ser informado através de uma rota.
- Entraremos em uma outra verbalização com os verbos *put/patch* que são usados para ==atualizar um recurso==.
-  Quando precisamos atualizar um recurso, precisamos saber qual o registro que será atualizado. Logo, precisamos de um parâmetro na URL **passando o ID** do objeto.

Assim como na criação, na edição precisamos criar duas rotas: uma para o formulário e a outra para enviar os dados deste formulário.

```php
Route::get('usuarios/editar/{user}', [TestController::class, 'formEditUser'])->name('users.formEditUser');

Route::put('usuarios/edit/{user}', [TestController::class, 'edit'])->name('users.edit');
```

Criamos os métodos formEditUser e edit no controlador:

```php
    public function formEditUser()
    {

    }

    public function edit()
    {

    }
```

A view do formulário de edição é diferente, pois precisa apresentar os dados já persistidos dentro do formulário.
O formulário de edição é bastante semelhante ao de cadastro. A diferença é que teremos o atributo *value* nos campos para recuperar os dados persistidos.

```php
    public function formEditUser(User $user)
    {
        return view('editUser', [
                                    'user' => $user
                                ]);
    }
```

O formEditUser recebe dados por parâmetro informados na rota `{user}`. Passamos esses dados por **injeção de dependência** e os enviamos para a view através de um vetor.

*Resumindo: você cria uma rota passando um parâmetro; cria um controlador e cria um método dentro dele que faz injeção de dependêcias; chama uma visão e passa um parâmetro para ela.*

Na view, vamos recuperar esses dados para preencher os campos:

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Editar Usuário</title>
</head>

<body>

    <form action="{{ route('users.edit', ['user' => $user->id]) }}" method="post">
	    @csrf
	    @method('PUT')
        <label for="">Nome do usuário:</label>
        <input type="text" name="name" value="{{ $user->name }}">

        <label for="">E-mail do usuário:</label>
        <input type="email" name="email" value="{{ $user->email }}">

        <label for="">Senha do usuário</label>
        <input type="password" name="password">

        <input type="submit" value="Editar usuário">
    </form>

</body>
</html>
```

No *action*, precisamos passar os dados para a rota `users.edit` e passar o id. 
Obs: A posição `'user'` tem que ser exatamente igual ao parâmetro passado na rota `{user}`

Dentro da tag html só temos os métodos get e post.
No Laravel, temos uma métodologia chamada _form spoofing_. Basicamente, o verbo é falsificado quando o Laravel receber essa requisição.
Para fazer isso, temos um helper chamado `@method` onde informamos o `PUT`.
Novamente, isso gerará um `input hidden` que terá o nome de método e o valor dele será put.

```html
<input type="hidden" name="_method" value="PUT">
```

Agora, precisamos capturar essas informações dentro do método edit que é responsável por persistir as informações dentro do banco de dados.

```php
    public function edit(User $user, Request $request)
    {
        $user->name = $request->name;
        
        if(filter_var($request->email, FILTER_VALIDATE_EMAIL)) {
            $user->email = $request->email;
        }

        if(!empty($request->password)) {
            $user->password = Hash::make($request->password);
        }

        $user->save();

        return redirect()->route('users.listAll');
    }
```


Não precisamos instanciar com `new User( )` pois o construtor do método já está fazendo isso. Utilizamos algumas validações para e-mail e senha, salvamos os dados e redirecionamos para a rota que lista tudo.

##### Remoção:

Criando a rota para o delete:

```php
Route::delete('usuarios/destroy/{user}', [TestController::class, 'destroy'])->name('user.destroy');
```

Método para remoção:

```php
    public function destroy(User $user) {
        var_dump($user);
    }
```

Resgata o parâmetro através da injeção de dependências e var_dump para testar se os dados estão sendo recebidos.

Para que possamos executar a ação de remoção, vamos na listagem geral de usuários 

*Obs: como estamos trabalhando com a verbalização, precisamos disparar um formulário para que a ação de remoção possa ser executada. Se tentarmos através do link, vai chegar como verbo `get` e não funcionará.*

==Sempre que trabalharmos com verbalização e utilizarmos um verbo delete, precisamos disparar um formulário==

```php
 <form action="{{ route('user.destroy', ['user' => $user->id]) }}" method="POST">
                        @csrf
                        @method('delete')
                        <input type="hidden" name="user" value="{{ $user->id }}">
                        <input type="submit" value="Remover">
                    </form>
```

Vamos testar clicando no botão `Remover`. Se mostrar `var_dump`, está tudo certo.

Implementando a remoção no método destroy

```php
    public function destroy(User $user) {
        $user->delete();
        
        return redirect()->route('users.listAll');
    }
```

##### Agrupamento de rotas 

Lista todas as rotas que foram parametrizadas:

`php artisan route:list `

A partir do Laravel 8, o uso de namespaces para rotas foi simplificado e tornou-se menos necessário, pois o Laravel agora usa o conceito de "Controller Actions" diretamente. O agrupamento de rotas no Laravel 11 é feito de forma semelhante, mas sem a necessidade de especificar namespaces, já que o Laravel considera que a estrutura padrão de controllers já reflete a estrutura de pastas.

Para Laravel 11, o código para o agrupamento de rotas pode ser assim:

```php
<?php

use App\Http\Controllers\Form\TestController;
use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('welcome');
});

Route::prefix('usuarios')->group(function () {
 /**
 * VERBO GET
 */
Route::get('/', [TestController::class, 'listAllUsers'])->name('users.listAll');
Route::get('novo', [TestController::class, 'formAddUser'])->name('users.formAddUser');
Route::get('editar/{user}', [TestController::class, 'formEditUser'])->name('users.formEditUser');
Route::get('{user}', [TestController::class, 'listUser'])->name('users.list');

/**
 * VERBO POST
 */
Route::post('store', [TestController::class, 'storeUser'])->name('users.store');

/**
 * VERBO PUT/PATCH
 */
Route::put('edit/{user}', [TestController::class, 'edit'])->name('users.edit');
/**
 * VERBO DELETE
 *
 */
Route::delete('destroy/{user}', [TestController::class, 'destroy'])->name('user.destroy');
});
```

##### Principais Pontos:

1. **Namespaces**: Em versões anteriores, você frequentemente usava o namespace para agrupar rotas. A partir do Laravel 8, o uso de namespaces foi reduzido, e o Laravel agora favorece a resolução de controladores através de "Controller Actions" com referências de classe.
    
2. **Prefixos e Middleware**: O agrupamento de rotas ainda é muito útil para aplicar prefixos de URL e middleware a um grupo de rotas. No exemplo acima, `Route::prefix` aplica um prefixo de URL a todas as rotas dentro do grupo.
    
3. **Importação de Controladores**: Em versões mais recentes, você pode importar diretamente a classe do controlador usando `use` no início do arquivo de rotas. Isso ajuda a tornar o código mais limpo e menos propenso a erros.