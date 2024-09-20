Os componentes em Laravel são blocos reutilizáveis de código e HTML que podem ser usados para criar partes da interface do usuário de maneira modular. Eles ajudam a organizar e reutilizar o código de forma eficiente, separando a lógica de apresentação em componentes independentes.

## Exemplo com Componente `Alert`

### **Passo a Passo para Criar um Componente `Alert`**

Vamos criar um componente `Alert` que será usado para exibir mensagens de alerta, como erros ou sucessos.

#### **1. Criar o Componente**

Para criar um componente, você usa o comando Artisan `make:component`. No terminal, execute:

`php artisan make:component Alert
`
Esse comando gera dois arquivos:

- **Classe do Componente:** `app/View/Components/Alert.php`
- **View do Componente:** `resources/views/components/alert.blade.php`

#### **2. Definir a Classe do Componente**

Abra o arquivo `app/View/Components/Alert.php` e defina a lógica e as propriedades do componente:

```php
namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    public $type;
    public $errors;

    /**
     * Create a new component instance.
     *
     * @param string $type
     * @param \Illuminate\Support\MessageBag|null $errors
     */
    public function __construct($type = 'danger', $errors = null)
    {
        $this->type = $type;
        $this->errors = $errors;
    }

    /**
     * Get the view / contents that represent the component.
     *
     * @return \Illuminate\View\View|string
     */
    public function render()
    {
        return view('components.alert');
    }
}
```

##### **Construtor da Classe**

- **`__construct($type = 'danger', $errors = null)`**:
    - **`$type`**: Define o tipo de alerta (ex.: `'danger'`, `'success'`). O valor padrão é `'danger'`.
    - **`$errors`**: Recebe uma coleção de erros. Pode ser `null` se não houver erros.

#### **3. Criar a View do Componente**

Edite o arquivo `resources/views/components/alert.blade.php` para definir a estrutura HTML do alerta:

```html
<div class="alert alert-{{ $type }}">
    @if ($errors && $errors->any())
        @foreach ($errors->all() as $error)
            <p>{{ $error }}</p>
        @endforeach
    @endif
</div>
```

Aqui, a view:

- **`alert alert-{{ $type }}`**: Adiciona a classe CSS com base no tipo de alerta.
- **`$errors && $errors->any()`**: Verifica se há erros a serem exibidos.
- **`$errors->all()`**: Itera sobre todos os erros e os exibe.

#### **4. Usar o Componente em uma View**

Agora, você pode usar o componente `Alert` em suas views Blade:
```html
<x-alert type="danger" :errors="$errors" />
```

- **`type="danger"`**: Define o tipo do alerta.
- **`:errors="$errors"`**: Passa a coleção de erros para o componente.

