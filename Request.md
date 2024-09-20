No Laravel, o uso de Requests é uma maneira elegante de validar e manipular dados de entrada em suas rotas ou controladores.

### 1. **Criando uma Request**

Para criar uma nova classe de Request no Laravel, você usa o Artisan CLI. No terminal, execute o seguinte comando:

`php artisan make:request StoreUserRequest`

Esse comando cria uma nova classe de Request na pasta `app/Http/Requests`.

### 2. **Editando a Classe de Request**

A classe criada terá um aspecto básico como este:

```php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreUserRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        // Aqui você pode colocar a lógica para determinar se o usuário está autorizado.
        // Para simplificar, vamos retornar true.
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users,email',
            'password' => 'required|string|min:8|confirmed',
        ];
    }
}

```

#### Explicação:

- **`authorize`**: Verifica se o usuário tem permissão para fazer essa requisição. Geralmente, você retornará `true` ou usará a lógica para decidir.
- **`rules`**: Define as regras de validação para os dados da requisição. Cada campo é validado de acordo com as regras especificadas.

### 3. **Usando a Request no Controlador**

Para usar a Request criada em um controlador, você precisa injetar a classe de Request no método do controlador.

```php
namespace App\Http\Controllers;

use App\Http\Requests\StoreUserRequest;
use Illuminate\Http\Request;
use App\Models\User;

class UserController extends Controller
{
    public function store(StoreUserRequest $request)
    {
        // Aqui, os dados da requisição já foram validados.
        $validatedData = $request->validated();

        // Cria um novo usuário usando os dados validados.
        $user = User::create([
            'name' => $validatedData['name'],
            'email' => $validatedData['email'],
            'password' => bcrypt($validatedData['password']),
        ]);

        return response()->json($user, 201);
    }
}
```

#### Explicação:

- Ao injetar `StoreUserRequest` no método `store`, o Laravel automaticamente valida a requisição com base nas regras definidas.
- O método `validated()` retorna apenas os dados que passaram pela validação.
- Se a validação falhar, o Laravel redirecionará automaticamente de volta com erros e os dados da requisição.

### 4. **Customizando Mensagens de Erro**

Você pode personalizar as mensagens de erro adicionando o método `messages` na classe de Request:

```php
public function messages()
{
    return [
        'name.required' => 'O nome é obrigatório.',
        'email.required' => 'O e-mail é obrigatório.',
        'email.email' => 'O e-mail deve ser válido.',
        'password.required' => 'A senha é obrigatória.',
        'password.confirmed' => 'A confirmação da senha não confere.',
    ];
}
```


### **Exibindo Mensagens de Erro em uma View Blade**

Se você estiver usando Blade, o sistema de templates do Laravel, você pode exibir mensagens de erro de validação em suas views facilmente. Aqui estão algumas maneiras comuns de fazer isso:

#### Exibindo Mensagens de Erro para um Campo Específico

```html
<!-- Formulário com campos para validar -->
<form action="{{ route('user.store') }}" method="POST">
    @csrf
    
    <div>
        <label for="name">Nome:</label>
        <input type="text" id="name" name="name" value="{{ old('name') }}">
        @error('name')
            <span class="text-red-500">{{ $message }}</span>
        @enderror
    </div>

    <div>
        <label for="email">E-mail:</label>
        <input type="email" id="email" name="email" value="{{ old('email') }}">
        @error('email')
            <span class="text-red-500">{{ $message }}</span>
        @enderror
    </div>

    <div>
        <label for="password">Senha:</label>
        <input type="password" id="password" name="password">
        @error('password')
            <span class="text-red-500">{{ $message }}</span>
        @enderror
    </div>

    <button type="submit">Registrar</button>
</form>
```

#### Exibindo Todas as Mensagens de Erro de Uma Só Vez

Se você deseja exibir todas as mensagens de erro de uma vez, pode fazer isso no topo da sua view:

```html
<!-- Verificar e exibir todas as mensagens de erro -->
@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```

### 5. **Definindo Regras Condicionais**

Se precisar aplicar regras de validação condicionalmente, você pode usar a função `sometimes` dentro da classe de Request:

```php
public function rules()
{
    return [
        'name' => 'required|string|max:255',
        'email' => 'required|email|unique:users,email',
        'password' => 'required|string|min:8|confirmed',
        'age' => 'nullable|integer|min:18', // Regra condicional para idade
    ];
}
```

Aqui, o campo `age` é opcional (`nullable`), mas se fornecido, deve ser um número inteiro e maior ou igual a 18.
