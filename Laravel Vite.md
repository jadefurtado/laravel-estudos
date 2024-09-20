
### Pré-requisito:

- Node.js
##### Instalar as dependências do Node.js:

`npm install`

O Vite é configurado por meio de um arquivo `vite.config.js` na raiz do seu projeto.

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
});
```

A partir de agora, você sempre criará seus arquivos `css` e `js` dentro da pasta `resources`.

`refresh: true`: O Vite já vem com o _route reload_. É uma configuração relacionada ao comportamento do HMR *(hot module replacement)*. O HMR permite que o navegador atualize a interface do usuário de forma dinâmica quando há mudanças no código fonte, sem a necessidade de um recarregamento completo da página.

==Obs: Não é recomendado importar o arquivo css no js, pois o carregamento fica lento.==

### Rodando o Vite

Existem duas maneiras de executar o Vite. Você pode executar o servidor de desenvolvimento por meio do comando dev, que é útil durante o desenvolvimento local. O servidor de desenvolvimento detectará automaticamente alterações em seus arquivos e as refletirá instantaneamente em qualquer janela aberta do navegador.

Ou executar o comando build irá versionar e agrupar os ativos do seu aplicativo e prepará-los para implantação na produção:

```
# Executa o servidor de desenvolvimento Vite...
npm run dev

# Cria e versiona os ativos para produção...
npm run build
```

Se você não executar o servidor com `npm run dev`, vai aparecer um erro `Vite manifest not found`

### Carregando os Scripts e Styles

Com so Vite configurado, agora você pode referenciá-los em uma diretiva Blade `@vite()` que você adiciona ao `<head>` do modelo raiz do seu aplicativo:

```html
<!DOCTYPE html>
<head>
@vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
```

### Asset( )

Após rodar o comando build, vamos colocar na view o arquivo compilado


```html
<head>
	<link rel="stylesheet" href="{{ asset('build/assets/style-DcBRhCul.css') }}">
</head>

...
<body>
	<script src="{{ asset('build/assets/script-DO_9BPpY.js') }}"></script>
</body>
```

#### Documentação:

https://laravel.com/docs/11.x/vite#installation