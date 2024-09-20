Basicamente são necessárias apenas duas linhas para que possa ser criada a arquitetura necessária para um CRUD.

O Laravel possui uma forma mais fácil de criar os métodos do Controller.

*Obs: Como já foi abordado antes, é importante criar o controller antes das rotas para não ter problema quando o artisan realizar a leitura do código.*

#### Criando o Controller

`php artisan make:controller Form/TestController --resource --model=User`

O parâmetro `--resource` cria todos os métodos necessários e o parâmetro `--model=User` faz com que seja feita toda a injeção de dependência nos métodos.

Também existe uma maneira mais otimizada para criação das rotas:

#### Criando as Rotas:

Vamos utilizar o `resource` para criar as rotas.

```php
Route::resource('usuarios', TestController::class);
```

Rodamos o comando `php artisan route:list`.

Resultado:

```
GET|HEAD        / .................................................................... 
  GET|HEAD        up ................................................................... 
  GET|HEAD        usuarios .................. usuarios.index › Form\TestController@index 
  POST            usuarios .................. usuarios.store › Form\TestController@store 
  GET|HEAD        usuarios/create ......... usuarios.create › Form\TestController@create 
  GET|HEAD        usuarios/{usuario} .......... usuarios.show › Form\TestController@show 
  PUT|PATCH       usuarios/{usuario} ...... usuarios.update › Form\TestController@update  
  DELETE          usuarios/{usuario} .... usuarios.destroy › Form\TestController@destroy  
  GET|HEAD        usuarios/{usuario}/edit ..... usuarios.edit › Form\TestController@edit  
```

Foram criadas rotas para os métodos de forma automatizada.
Vamos mudar os nomes das rotas de usuarios para user. O que precisa ficar em português é apenas a URI.

```php
Route::resource('usuarios', TestController::class)->names('user');
```

Resultado: 

```
  GET|HEAD        usuarios ...................... user.index › Form\TestController@index  
  POST            usuarios ...................... user.store › Form\TestController@store  
  GET|HEAD        usuarios/create ............. user.create › Form\TestController@create  
  GET|HEAD        usuarios/{usuario} .............. user.show › Form\TestController@show  
  PUT|PATCH       usuarios/{usuario} .......... user.update › Form\TestController@update  
  DELETE          usuarios/{usuario} ........ user.destroy › Form\TestController@destroy  
  GET|HEAD        usuarios/{usuario}/edit ......... user.edit › Form\TestController@edit  
```

Agora, temos os nomes das rotas em inglês como `user.create`.

Vamos mudar o parâmetro  `{usuario}` para `{user}`, pois precisamos deixá-lo exatamente igual ao parâmetro recebido dentro do método do controlador.

```php
Route::resource('usuarios', [TestController::class])->names('user')->parameters(['usuarios' => 'user']);
```

Resultado:

```
  GET|HEAD        usuarios ...................... user.index › Form\TestController@index  
  POST            usuarios ...................... user.store › Form\TestController@store  
  GET|HEAD        usuarios/create ............. user.create › Form\TestController@create  
  GET|HEAD        usuarios/{user} ................. user.show › Form\TestController@show  
  PUT|PATCH       usuarios/{user} ............. user.update › Form\TestController@update  
  DELETE          usuarios/{user} ........... user.destroy › Form\TestController@destroy  
  GET|HEAD        usuarios/{user}/edit ............ user.edit › Form\TestController@edit 
```

Porém, o nome da URL ainda está misturando português com inglês como por exemplo `usuarios/create` e queremos que tenha um padrão em português como `usuarios/novo`.

##### Parametrizar a URL para que fique toda em português:

- Acessar : `app > providers > AppServiceProvider.php`
- Dentro do método boot, vamos chamar o Route

```php

use Illuminate\Support\Facades\Route;

    public function boot(): void
    {
        Route::resourceVerbs([
                                'create' => 'novo',
                                'edit' => 'editar'
					        ]);
    }
```

==Obs: usar o `Illuminate\Support\Facades\Route`, pois o Laravel provavelmente vai importar o errado e dar erro!==

Agora, todas as rotas que forem criadas a partir desse ponto, sempre que tiver um `create` ou `edit`, vão obedecer aos nomes `novo` e `editar`. 

Resultado:

```
  GET|HEAD        usuarios ......................... user.index › Form\TestController@index  
  POST            usuarios ......................... user.store › Form\TestController@store  
  GET|HEAD        usuarios/novo .................. user.create › Form\TestController@create  
  GET|HEAD        usuarios/{user} .................... user.show › Form\TestController@show  
  PUT|PATCH       usuarios/{user} ................ user.update › Form\TestController@update  
  DELETE          usuarios/{user} .............. user.destroy › Form\TestController@destroy  
  GET|HEAD        usuarios/{user}/editar ............. user.edit › Form\TestController@edit
```

Essa parametrização só precisa ser feita uma vez. As demais parametrizações no arquivo de rotas precisam ser feitas em todas as rotas em outros momentos.

A partir de agora, fica tudo mais fácil. Agora que já temos tudo parametrizado, vamos supor que queremos fazer um CRUD de produtos, categorias ou artigos: vamos executar exatamente a linha que temos acima, duplicamos ela e simplesmente alteramos os parâmetros que são dinâmicos. Tudo estará funcionando e sendo interpretado pelo framework.

**Precisamos colocar na regra de negócios**
- Pegar as views, colocar o conteúdo necessário e fazer a persistência no banco de dados.

#### Index:

- Responsável pela listagem geral.

1.  Criar a view

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Lista dos Usuários</title>
</head>

<body>
    <table>
        <tr>
            <td>ID</td>
            <td>Nome</td>
            <td>E-mail</td>
            <td>Ações</td>
        </tr>

        <tr>
            <td></td>
            <td></td>
            <td></td>
            <td>
                <a href="">Ver Usuário</a>
                <form action="" method="post">
                    <input type="hidden" name="user" value="">
                    <input type="submit" value="Remover">
                </form>
            </td>
        </tr>
    </table>

</body>
</html>
```

2.  Retornar a view pelo Controller

```php
    public function index()
    {
        return view('listAllUsers');
    }
```

3.  Fazer uma pesquisa no banco de dados para popular a visão

```php
    public function index()
    {
        $users = User::all();
        return view('listAllUsers', [
                                        'users' => $users
                                    ]);
    }
```

4. Popular a view com os dados passados

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Lista dos Usuários</title>
</head>

<body>
    <table>
        <tr>
            <td>ID</td>
            <td>Nome</td>
            <td>E-mail</td>
            <td>Ações</td>
        </tr>
        
        @foreach ($users as $user)
        <tr>
            <td>{{ $user->id }}</td>
            <td>{{ $user->name }}</td>
            <td> {{ $user->email }}</td>
            <td>
                <a href="">Ver Usuário</a>
                <form action="" method="post">
                    <input type="hidden" name="user" value="">
                    <input type="submit" value="Remover">
                </form>
            </td>
        </tr>
        @endforeach
    </table>

</body>
</html>
```

#### Show:

- O show é utilizado para mostrar detalhes. No exemplo, ele corresponde ao "ver usuário".
-  Vamos passar a rota utilizando o helper do blade no link Ver usuário:

```html
<a href="{{ route('user.show', ['user' => $user->id]) }}">Ver Usuário</a>
```

1.  Criar a view

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Detalhes do Usuário</title>
</head>

<body>
    <h1></h1>
    <p></p>
    <p></p>
</body>

</html>
```

Como estamos mandando um usuário para dentro da view, precisamos fazer a leitura desse usuário para poder conversar com a visão.

2. Retornar a view pelo Controller:

```php
    public function show(User $user)
    {
        return view('listUser', [
                                    'user' => $user
                                ]);
    }
```

3. Popular a view com os dados 

```html
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
    <p> {{ $user->email }}</p>
    <p>{{ date('d/m/Y H:i', strtotime($user->created_at)) }}</p>
</body>

</html>
```

#### Create e Store

-  O `create` é responsável pelo formulário de criação e o `store` é responsável por pegar esses dados e persistí-los no banco de dados.

1. Criar a view

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

        <input type="submit" value="Cadastrar">
    </form>

</body>

</html>
```

2.  Retornar a view pelo Controller

```php
    public function create()
    {
        return view('newUser');
    }
```

3.  Preparar o formulário para enviar os dados

```html
    <form action="{{ route('user.store') }}" method="POST">
        @csrf
```

-  Precisamos colocar a rota responsável pela action e colocar o `@csrf` token
-  Para ver tudo que está chegando na requisição, vamos no método store e colocamos a variável `$request`(responsável por receber os dados) no `var_dump`.

```php
    public function store(Request $request)
    {
        var_dump($request);
    }
```

4. Persistir os dados no banco de dados

-  Instancia o modelo, seta as propriedades, salva os dados e redireciona o usuário para a listagem.

```php
    public function store(Request $request)
    {
        $user = new User();
        $user->name = $request->name;
        $user->email = $request->email;
        $user->password = Hash::make($request->password);
        $user->save();

        return redirect()->route('user.index');
    }
```


#### Edit e Update

-  O `edit` é responsável pelo formulário de edição e o `update` é responsável por persistir as alterações no banco de dados.

1.  Criar a view

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

    <form action="{{ route('user.update', ['user' => $user->id]) }}" method="POST">
        <label for="">Nome do usuário:</label>
        <input type="text" name="name" value="{{ $user->name }}">

        <label for="">E-mail do usuário:</label>
        <input type="email" name="email" value="{{ $user->email }}">

        <label for="">Senha do usuário:</label>
        <input type="password" name="password">

        <input type="submit" value="Salvar">
    </form>

</body>

</html>
```

-  O usuário é passado por injeção de dependência no método edit, então podemos usá-lo no formulário para preencher o `value` dos campos.
-  Passamos na action a rota `'user.update` e o id do usuário que será editado.

2.  Retornar a view pelo Controller

```php
    public function edit(User $user)
    {
        return view('editUser', [
                                    'user' => $user
                                ]);
    }
```

-  Passar a view `editUser` e passar o usuário através da injeção de dependência.

3.  Preparar o formulário para enviar os dados

```html
    <form action="{{ route('user.update', ['user' => $user->id]) }}" method="POST">
        @csrf
        @method('put')
```

-  Precisamos fazer o _form spoofing_, pois o navegador não suporta o método `put`.
-  Para testar, podemos utilizar um `var_dump` no método `update`.

```php
    public function update(Request $request, User $user)
    {
        var_dump($request, $user);
    }
```

4.  Fazer a atualização no Banco de Dados

```php
    public function update(Request $request, User $user)
    {
        $user->name = $request->name;
        $user->email = $request->email;
        
		// validação para verificar se passou uma senha
        if(!empty($request->password)) {
            $user->password = Hash::make($request->password);
        }

        $user->save();

        return redirect()->route('user.index');

    }
```


#### Delete

-  Temos uma `user.destroy` que utiliza o verbo delete. Agora, precisamos disparar a ação para ela, remover o registro do banco de dados e redirecionar o usuário para a listagem.

1. Disparar a ação 

```html
<form action="{{ route('user.destroy', ['user' => $user->id]) }}" method="post">

@csrf
@method('delete')

<input type="submit" value="Remover">

</form>
```

- Como estamos trabalhando com verbalização, precisamos criar um formulário, pois não é possível disparar um verbo através de um link.
- O verbo `delete` não é suportado pelo navegador. Logo, precisamos utilizar um _form spoofing_.

```php
    public function destroy(User $user)
    {
        dd($user);
    }
```

-  Testar se o usuário está sendo recebido.

2.  Remover da base de dados e redirecionar o usuário

```php
    public function destroy(User $user)
    {
        $user->delete();

        return redirect()->route('user.index');
    }
```


### Checklist ✅

✅ Criação do controlador
✅Criação das rotas (e tradução dos parâmetros)
✅ Alimentar as visões (HTML)
✅ Fazer os métodos
