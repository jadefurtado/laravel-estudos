### O que é Middleware?

Middleware é um mecanismo que permite interceptar e manipular solicitações HTTP antes que elas cheguem ao seu controlador ou depois que uma resposta é enviada. É uma forma de adicionar funcionalidades a todas as rotas ou a um grupo específico delas, sem precisar duplicar o código.

### Como Funciona?

Middleware é ==uma camada entre a requisição do usuário e a resposta do servidor==. Ele pode ser usado para diversas finalidades, como:

- **Autenticação**: Verificar se o usuário está autenticado.
- **Log**: Registrar informações sobre a requisição.
- **CORS**: Gerenciar a política de compartilhamento de recursos.
- **Modificação de requisições/respostas**: Alterar dados antes de passar para o controlador ou após a execução dele.

### Criando Middleware

Para criar um middleware em Laravel, você pode usar o comando Artisan:
`php artisan make:middleware nomeDoMiddleware`

Isso criará um arquivo em `app/Http/Middleware/NomeDoMiddleware.php`. Dentro desse arquivo, você verá um método `handle` que recebe a requisição, o próximo middleware e pode manipular a resposta.

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Support\Facades\Auth;

class CheckAuthenticated
{
    public function handle($request, Closure $next)
    {
        if (!Auth::check()) {
            return redirect('login'); // Redireciona para a página de login
        }

        return $next($request); // Continua para o próximo middleware ou controlador
    }
}
```

### Registrando Middleware

Após criar o middleware, você precisa registrá-lo. Isso pode ser feito em `app/Http/Kernel.php`. Você pode adicioná-lo ao grupo de middleware existente ou criar um novo.

#### Exemplo de registro:

```php
protected $routeMiddleware = [
    // Outros middlewares...
    'auth.check' => \App\Http\Middleware\CheckAuthenticated::class,
];
```

### Usando Middleware nas Rotas

Você pode aplicar o middleware diretamente nas rotas no arquivo de rotas (`routes/web.php` ou `routes/api.php`):

```php
Route::get('/dashboard', [DashboardController::class, 'index'])->middleware('auth.check');
```

Ou aplicar a um grupo de rotas:

```php
Route::middleware(['auth.check'])->group(function () {
    Route::get('/profile', [ProfileController::class, 'show']);
    Route::get('/settings', [SettingsController::class, 'edit']);
});
```

### Middleware Global

Se você quiser que um middleware seja aplicado a todas as requisições, pode registrá-lo no array `$middleware` em `app/Http/Kernel.php`.

### Conclusão

Middleware é uma ferramenta poderosa em Laravel que permite organizar e reutilizar lógica comum para suas aplicações. Usando middleware, você pode manter seu código limpo e modular, facilitando a manutenção e a adição de novas funcionalidades.