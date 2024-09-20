Crie ou edite o arquivo `resources/sass/app.scss` e importe o Bootstrap. Você pode então adicionar suas customizações nesse arquivo.

```css
// Importa o Bootstrap
@import "bootstrap/scss/bootstrap";

// Suas customizações
$primary: #ff5733; // Exemplo: Mudando a cor primária
$font-family-base: 'Arial', sans-serif; // Exemplo: Mudando a fonte base

// Reimporta o Bootstrap para aplicar as customizações
@import "bootstrap/scss/bootstrap";

```
