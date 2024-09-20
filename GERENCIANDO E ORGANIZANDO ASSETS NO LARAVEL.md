### Iniciando o projeto:

Em `views` vamos criar um diretório `site`, que ficará responsável por armazenar as visões que são públicas, ou seja, qualquer visitante que acessar o site poderá navegar por elas.

Obs: trata-se de um exemplo simples. O ideal é criar um `master` para que você possa estender as outras visões a partir dele.

É essencial que seu site tenha a folha de estilos (arquivo css) e também o arquivo de script para que você possa manipular o que está acontecendo dentro da sua página.
Esses arquivos precisam ser _web accessible_, ou seja, acessível pela web. No Laravel, temos apenas um diretório que é _web accessible_, o `public`.

Não vamos criar os arquivos css e js no diretório `public`, mas sim dentro do diretório do próprio tema, pois qualquer modificação que fizemos nesses arquivos, é possível pré-processar todas as atividades necessárias (concatenar todos eles em apenas um arquivo, minificar) e depois pegamos somente o resultado e jogamos para a `public`. Assim, o código fica muito mais enxuto e vai ficar em produção somente o necessário.  
 
 ==O código fonte dos arquivos não fica público.==
 
#### [[Laravel Vite]]:
 
O Vite é conhecido por sua experiência de desenvolvimento rápida e eficiente, oferecendo tempos de recarga instantâneos e uma configuração mais simples em comparação com o Webpack, que o Laravel Mix utiliza.

Dentro de `resources` criar  o arquivo `style.css` e o arquivo `script.js`.

#### Configurar o Vite para que a pasta `public` tenha acesso aos diretórios criados

Para configurar o Vite de forma que os arquivos dentro da pasta `resources` possam ser acessados e compilados para a pasta `public` em um projeto Laravel, você precisa ajustar algumas configurações.
##### Instalar as dependências: 
 `npm install
##### Configuração do Vite:
 O arquivo `vite.config.js` é onde você configura como o Vite deve processar e construir seus assets.
Você precisa configurar o Vite para processar os arquivos da pasta `resources` e gerar os arquivos compilados na pasta `public`. Crie ou edite o arquivo `vite.config.js` na raiz do seu projeto:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({

  plugins: [
    laravel({
      input: [
        'resources/css/style.css',
        'resources/js/script.js',
      ],
      refresh: true,
    }),
  ],
  build: {
    manifest: true,
    outDir: 'public/build',
  },
});
```


- **`outDir: 'public/build'`**: Define o diretório onde os arquivos compilados serão colocados. Neste caso, todos os arquivos gerados pelo Vite serão movidos para a pasta `public/build`.
	Obs: não é recomendado colocar direto na pasta public, mas sim criar uma pasta build, pois quando apontamos diretamente, alguns arquivos são perdidos.

- `refresh: true`: O Vite já vem com o _route reload_. É uma configuração relacionada ao comportamento do HMR *(hot module replacement)*. O HMR permite que o navegador atualize a interface do usuário de forma dinâmica quando há mudanças no código fonte, sem a necessidade de um recarregamento completo da página.

- **`input`**: Especifica os arquivos de entrada que o Vite deve processar. Aqui você está dizendo ao Vite para processar o arquivo `script.js` e `style.css` encontrados nas pastas `resources/js` e `resources/css`, respectivamente.

	Obs: não esquecer de importar o `laravel-vit-plugin`

**Minificação**: O Vite e o Rollup (a ferramenta de bundling usada pelo Vite) automaticamente minificam os arquivos CSS durante o build, garantindo que a saída seja otimizada.
##### Executar o comando de build para que o Vite processe os arquivos e gere a saída esperada:

`npm run build`


==Nunca importar o arquivo css dentro do js, pois o carregamento fica prejudicado.==
##### Execute o Build:
`npm run build`

No Laravel, quando você utiliza o Vite como seu bundler de assets, os comandos `npm run build` e `npm run dev` desempenham papéis diferentes e são usados em diferentes etapas do ciclo de desenvolvimento e implantação. Vamos explorar as diferenças entre esses dois comandos:

### `npm run dev`

- **Propósito**: O comando `npm run dev` é utilizado para iniciar um servidor de desenvolvimento. Ele faz uso do Vite para fornecer uma experiência de desenvolvimento com recarregamento rápido e atualizado em tempo real.
    
- **Função**:
    
    - **Hot Module Replacement (HMR)**: Permite que você veja as mudanças em tempo real sem precisar recarregar a página manualmente. Se você modificar um arquivo, o Vite aplica as mudanças dinamicamente, o que torna o processo de desenvolvimento mais ágil.
    - **Modo de Desenvolvimento**: Ele executa o Vite em modo de desenvolvimento, o que pode incluir a ativação de recursos como o HMR, que não são ideais para um ambiente de produção.
    - **Serviço Local**: Geralmente, isso também inicia um servidor local para servir seus arquivos e ativos durante o desenvolvimento.
	- **Uso**: Este comando é usado durante o desenvolvimento ativo do seu projeto. Você o executa para começar a trabalhar em seu projeto e visualizar as mudanças imediatamente.
    

### `npm run build`

- **Propósito**: O comando `npm run build` é utilizado para construir uma versão otimizada e pronta para produção dos seus arquivos e ativos.
    
- **Função**:
    
    - **Build para Produção**: Ele gera os arquivos finais e otimizados que serão usados em seu ambiente de produção. Isso inclui a minificação dos arquivos CSS e JavaScript, a criação de hashes para cache busting e a otimização geral dos recursos.
    - **Sem HMR**: O build não inclui recursos como o Hot Module Replacement. Em vez disso, ele se concentra em criar arquivos que podem ser entregues aos usuários finais de forma eficiente e segura.
    - **Diretório de Saída**: Os arquivos gerados geralmente são colocados em um diretório específico, como `public/build`, que é onde o Laravel espera que os arquivos compilados sejam encontrados.
	- **Uso**: Este comando é usado quando você está pronto para implantar seu aplicativo em um servidor de produção. Ele prepara os arquivos e ativos para serem servidos a partir de um ambiente de produção.
    

### Resumindo

- **`npm run dev`**: Usado durante o desenvolvimento. Fornece uma experiência interativa com atualizações em tempo real e HMR.
- **`npm run build`**: Usado para criar uma versão final do aplicativo, otimizada para produção. Gera arquivos prontos para serem servidos em um ambiente ao vivo.

Em um fluxo típico de trabalho, você utilizaria `npm run dev` enquanto desenvolve e testa o seu aplicativo, e `npm run build` quando estiver pronto para fazer o deploy do seu aplicativo em produção.


#### Usando os arquivos na view:

Após compilar os arquivos com `npm run build`, para utilizá-los usaremos a função `{{ assets() }}`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>

    <link rel="stylesheet" href="{{ asset('build/assets/reset-Cxr3N-1a.css') }}">
    <link rel="stylesheet" href="{{ asset('build/assets/style-DcBRhCul.css') }}">
</head>

<body>
    <h1>Testee</h1>
    <script src="{{ asset('build/assets/script-DO_9BPpY.js') }}"></script>
</body>
</html>
```