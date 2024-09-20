
### Ir na Documentação do Laravel:

#### Instalar o Node caso não tenha

#### Instalar o Vite:
	
	`npm install`

#### Configurar o arquivo do vite:

`vite.config.js`

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
});
```

### Ir na documentação do Bootstrap

### Instalar o Bootstrap:

	`npm i --save bootstrap @popperjs/core`

#### Instalar o SASS:

	`npm i --save-dev sass`

#### Rodar o Node:

	`npm run dev`

#### Importar o arquivo do Bootstrap:

	`resources > app.js`
	
```js
import './bootstrap';
```

Agora, no arquivo bootstrap, importar o framework:

```js
import 'bootstrap';

import axios from 'axios';

window.axios = axios;
window.axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';
```

#### Incluir o SCSS:

Dentro de resources, criar um diretório chamado `sass` e dentro dele um arquivo `app.scss`.
Importar o framework bootstrap

==Atenção: o nome do diretório dentro do framework bootstrap é `scss` e o que criamos  é `sass`!!!! Quando for importar, lembrar disso.

```css
@import 'bootstrap/scss/bootstrap';
```


#### Indicar para o Vite o carregamento do diretório SASS:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/sass/app.scss',
                    'resources/js/app.js'],
            refresh: true,
        }),
    ],
});
```

#### Incluir na view:

```js
@vite(['resources/sass/app.scss', 'resources/js/app.js'])
```

### [[Customizando o Bootstrap]]