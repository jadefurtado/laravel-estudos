
### Migrations:

- Permite ==versionar as modificações== feitas no banco de dados. Toda a parte de estrutura para modelar o banco de dados fica atrelada ao código.

Uma das primeiras preocupações após baixar o Laravel é fazer a integração com o banco de dados
Precisamos fazer as **parametrizações** no arquivo `.env`
[[APRENDENDO MVC E SISTEMA DE ROTAS]]

#### Onde ficam as migrations? 

-  Caminho: `database > migrations`

#### Up e Down:

- Quando estamos executando um comando no banco de dados e queremos que ele persista as alterações, o método que será executado é o método `up`. Quando queremos executar um _rollback_ ou desfazer as ações de uma migration, será executado o método `down`. 

- ==O método `down` tem que fazer exatamente o oposto do que é feito no método `up` para evitar problemas posteriores.==

#### Criando uma migration

`php artisan make:migration`

- No Laravel, temos uma metodologia de nomenclatura dos arquivos.
- Como vamos criar uma nova tabela de categoria de arquivos, vamos seguir o exemplo da nomenclatura dos arquivos padrão do Laravel
 
 `php artisan make:migration create_categories_table`

#### Estrutura do arquivo

`data + hash + nome`

É fundamental manter os prefixos, pois as migrations são rodadas em ordem, a partir da data mais antiga para a atual. Isso evita conflitos nos relacionamentos e versionamentos do banco.

==Se você criar o arquivo no mesmo padrão do Laravel, não é necessário informar os parâmetros no comando de criação da migration.==
#### --create

Criar tabela com o padrão de nome do laravel :
`php artisan make:migration teste --create=teste`

#### --table

Alterar os dados da tabela :
`php artisan make:migration teste2 --table=teste2`

#### Adicionar campos na tabela:

Ex:

```php
    public function up(): void
    {
        Schema::create('categories', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('title');
            $table->string('slug');
            $table->text('description')->nullable();
            $table->timestamps();
        });
    }
```

##### bigIncrements: 
Abstração do Laravel que cria um big int com autoincrement

**bigIncrements x id:**
A principal diferença é que `$table->id()` é uma forma mais curta e legível de criar uma coluna de chave primária `BIGINT` auto-incremental. Ele foi introduzido para tornar o código de migração mais limpo e menos repetitivo.
Ambos têm o mesmo comportamento subjacente, mas `$table->id()` é mais moderno e recomendado para as versões mais recentes do Laravel.

##### string: 
abstração do Laravel que cria um `varchar`.

O campo `slug` armazena o mesmo conteúdo contido em `title,` porém com uma url amigável (sem acento, cedilha, pontuação, espaços). 
Para a descrição, precisamos de um campo mais extenso e usamos o `text`

##### text: 
Trabalha com texto extensos. 

##### nullable( ): 
Esse método faz com que o campo possa ser nulo, ou seja, não é preciso informar um valor para ele. 

##### timestamps:
Gera dois campos dentro da base de dados: o `created_at`e o `updated_at`
O *Eloquent* faz a gestão desses campos e atualiza quando necessário. Ou seja, toda vez que você criar um registro dentro do campo de dados, o *Eloquent* salva dentro do create e toda vez que você atualizar, ele vai salvar dentro do update também.
Esse recurso é muito interessante para ter um **log das informações** para saber quando ela foi criada e atualizada. 
Caso você não queira esses campos, é necessário apagar o timestamps do método up. Depois, você precisa desabilitar o timestamps dentro do modelo e desabilitar o parâmetro timestamps.

#### Executar as migrates:

`php artisan serve migrate`

#### Entendendo as tabelas criadas:

- Por padrão, o Laravel cria três arquivos de tabelas. 
- Porém, no banco de dados aparecem 4 tabelas
- Essa tabela a mais é a tabela `migrations`, que faz parte do *framework*. É criada automaticamente pelo motor do Laravel. Basicamente, ela vai fazer a gestão das migrations: o que foi rodado, o que irá rodar, o que está pendente. Quando mandarmos as migrations para outra pessoa, ela não precisa executar as migrations que ela já fez, mas apenas as novas. ==É através da tabela migrations que o framework faz a gestão para que o ambiente esteja sempre sincronizado.==

#### Rollback:

Executa o método **down** de acordo com os passos que você informar:

`php artisan migrate:rollback --step=[quantidade de passos]`

Por exemplo, eu precisei fazer uma alteração grande no banco de dados e executei 5 migrations. Porém, isso acabou "quebrando" o banco de outra pessoa. Com isso, ela precisará desfazer essas 5 migrations para fazer algum ajuste, ver o que está acontecendo, debugar a aplicação. 

A pessoa pode executar um rollback com 5 steps:

`php artisan migrate:rollback --step=5`

Se quisermos omitir o parâmetro step, ele irá desfazer todas as alterações dentro do banco de dados.

#### É possível criar relacionamentos?

Sim. Porém, é preciso observar que, ==antes de criar um relacionamento, as duas tabelas que serão relacionadas precisam estar criadas.==
É possível criar um relacionamento dentro de uma mesma migration, mas é importante obedecer a ordem de hierarquia, pois não há como se relacionar com algo que não existe. Ao criar um relacionamento antes da tabela existir, resultará em problemas. 
É muito importante observar a cronologia das informações conforme elas vão chegando dentro do banco de dados, pois tudo para o banco é matemático.

Como exemplo, vamos criar uma migration para a tabela post

`php artisan make:migration create_posts_table`

Vamos adicionar alguns campos na tabela:

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

Vamos criar o relacionamento

Obs: o relacionamento pode ser criado em outra migration, mas como estamos criando a tabela posts, não tem problema colocar o relacionamento dentro dessa mesma migration.  

==Quando vamos criar um relacionamento, é necessário que os dois campos sejam exatamente do mesmo tipo.==
Precisamos analisar os campos da tabela, verificar se é assinado ou não-assinado.
Caso seja de tipos diferentes, consultar a documentação.

Definir a coluna que será a chave estrangeira:

```php
$table->unsignedBigInteger('author'); 
```

Definir o relacionamento

```php
$table->foreign('author')->references('id')->on('users')->onDelete('CASCADE');
```

- `foreign('author')`: Define que a coluna `author` será uma chave estrangeira.
- `references('id')`: Especifica que a chave estrangeira faz referência à coluna `id` na tabela `users`.
- `on('users')`: Indica a tabela de destino (`users`).
- `onDelete('cascade')`: Define o comportamento de exclusão em cascata, o que significa que se um registro na tabela `users` for excluído, todos os registros relacionados na tabela `posts` também serão excluídos.

Executar a migrate
`php artisan migrate`