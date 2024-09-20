
A principal função da camada Service é ==aglomerar regras de negócio== para poder reutilizá-las e ==evitar lógica de negócios nos controladores.==

### Por que usar o padrão Services?

1. **Separation of Concerns**: Mantém a lógica de negócios separada da lógica de apresentação (controladores) e persistência de dados (modelos).
2. **Reusabilidade**: Permite que a lógica de negócios seja reutilizada em diferentes partes da aplicação.
3. **Testabilidade**: Facilita a escrita de testes unitários, já que a lógica está isolada em serviços.
### Como usar o padrão Services no Laravel

#### 1. Crie um diretório para seus serviços

É uma boa prática criar um diretório para organizar seus serviços. No Laravel, você pode criar um diretório `Services` dentro do diretório `app`.
#### 2. Crie um serviço

Dentro do diretório `Services`, você pode criar uma classe de serviço. Por exemplo, vamos criar um serviço chamado `UserService`.

```php
<?php

namespace App\Services;

use App\Models\User;

class UserService
{
    public function getUserById($id)
    {
        return User::find($id);
    }

    public function createUser(array $data)
    {
        return User::create($data);
    }
}
```

#### 3. Utilize o serviço em um controlador

Para usar o serviço em um controlador, você deve injetá-lo através do construtor ou diretamente no método do controlador. Vamos usar o exemplo com injeção no construtor:

```php
<?php

namespace App\Http\Controllers;

use App\Services\UserService;
use Illuminate\Http\Request;

class UserController extends Controller
{
    protected $userService;

    public function __construct(UserService $userService)
    {
        $this->userService = $userService;
    }

    public function show($id)
    {
        $user = $this->userService->getUserById($id);
        return view('user.show', compact('user'));
    }

    public function store(Request $request)
    {
        $data = $request->all();
        $user = $this->userService->createUser($data);
        return redirect()->route('users.show', $user->id);
    }
}
```

### Resumo

- **Crie um diretório `Services`** para organizar seus serviços.
- **Defina classes de serviço** que encapsulam a lógica de negócios.
- **Injete e use o serviço** nos controladores ou outros componentes conforme necessário.

[[Exemplos de Service Layer]]