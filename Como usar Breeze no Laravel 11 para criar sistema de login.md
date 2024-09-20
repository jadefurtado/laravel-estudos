
#### Instalar o Breeze

`composer require laravel/breeze --dev``

Caso dê algum erro, atualizar o composer
`composer update` 

#### Publicar a autenticação, rotas, controllers , entre outras funcionalidades:

`php artisan breeze:install`

Aparecerão algumas opções. Escolhi as seguintes:
`react, dark, 1(PHPUnit)`

#### Parametrizar o banco de dados no arquivo `.env` e depois executar as migrations:

`php artisan migrate`

#### Criar Seed:

`php artisan make:seeder UserSeeder`

#### Executar a Seed para criar um usuário teste no banco de dados:

`php artisan db:seed`

#### [[Criar uma factory]]:

`php artisan make:factory UserFactory --model=User
`
#### Definir a estrutura da factory para criação de um usuário:

```php

namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

class UserFactory extends Factory
{
    protected $model = User::class;

    public function definition()
    {
        return [
            'name' => $this->faker->name, // Gera um nome aleatório
            'email' => $this->faker->unique()->safeEmail, // Gera um e-mail único
            'password' => Hash::make('password'), // Gera uma senha criptografada com 'password'
        ];
    }
}

```

#### Usar a Factory:

```php
use Illuminate\Database\Seeder;
use App\Models\User;

class DatabaseSeeder extends Seeder
{
    public function run()
    {
        // Cria 10 usuários
        User::factory(10)->create();
           
		// criar o usuário administrador
        User::factory()->create([
            'name' => 'Admin User',
            'email' => 'admin@admin.com',
            'password' => 123,
        ]);
    }
}
```

No arquivo DatabaseSeeder, vamos usar o método `call()` e criar um array para passar as seeds que queremos usar:

```php
    public function run(): void
    {
        // array das seeds para executar
        $this->call([
                        UserSeeder::class,
                    ]);
    }
```

#### Executar a Seed:

`php artisan db:seed`

#### Instalar as dependências do Node.js

`npm install`

#### Executar as bibliotecas do Node.js

`npm run dev`

###  Como traduzir Breeze e Laravel 11 para português -Traduzir as mensagens de erro para português


https://github.com/lucascudo/laravel-pt-BR-localization

Para traduzir a página de login, vamos em
`js > Pages > Auth > Login.jsx`

Alterar os campos:

```js
<InputLabel htmlFor="email" value="E-mail" />
```

```js
<InputLabel htmlFor="password" value="Senha" />
```