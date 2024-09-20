### Cenário: Aplicação de Descontos em Pedidos

Suponha que você tenha um controlador que lida com pedidos, e você precisa aplicar um desconto baseado em um código de cupom. A lógica de aplicação de descontos pode ser complexa e mudar com o tempo, então é uma boa prática isolar essa lógica em um serviço.

#### 1. **Lógica no Controlador (Sem Serviços)**

Sem o padrão de serviços, o controlador pode ficar sobrecarregado com a lógica de negócios.

**Exemplo de Controlador Sem Serviços:**

```php
<?php

namespace App\Http\Controllers;

use App\Models\Order;
use App\Models\Coupon;
use Illuminate\Http\Request;

class OrderController extends Controller
{
    public function applyCoupon(Request $request, $orderId)
    {
        // Validação
        $request->validate([
            'coupon_code' => 'required|string',
        ]);

        // Encontrar o pedido
        $order = Order::findOrFail($orderId);

        // Encontrar o cupom
        $coupon = Coupon::where('code', $request->input('coupon_code'))->first();

        if ($coupon) {
            // Aplicar o desconto
            $discount = $this->calculateDiscount($coupon, $order);
            $order->total -= $discount;
            $order->save();

            return response()->json(['message' => 'Cupom aplicado com sucesso.', 'discount' => $discount]);
        } else {
            return response()->json(['message' => 'Cupom inválido.'], 400);
        }
    }

    private function calculateDiscount($coupon, $order)
    {
        // Exemplo simplificado de cálculo de desconto
        $discount = 0;
        if ($coupon->type === 'percentage') {
            $discount = ($order->total * ($coupon->value / 100));
        } elseif ($coupon->type === 'fixed') {
            $discount = $coupon->value;
        }
        return min($discount, $order->total);
    }
}
```

Neste exemplo, o controlador lida com a validação da requisição, busca o pedido e o cupom, e contém a lógica de cálculo do desconto. O controlador está agora sobrecarregado com responsabilidades.

#### 2. **Separando a Lógica no Serviço**

Vamos criar um serviço `DiscountService` que encapsula a lógica de aplicação de descontos. O controlador será mais limpo e o serviço ficará responsável apenas pela lógica de negócios.

**Serviço `DiscountService`:**

```php
<?php

namespace App\Services;

use App\Models\Coupon;
use App\Models\Order;

class DiscountService
{
    public function applyCoupon(Order $order, $couponCode)
    {
        $coupon = Coupon::where('code', $couponCode)->first();

        if ($coupon) {
            $discount = $this->calculateDiscount($coupon, $order);
            $order->total -= $discount;
            $order->save();

            return $discount;
        }

        return 0;
    }

    private function calculateDiscount(Coupon $coupon, Order $order)
    {
        $discount = 0;
        if ($coupon->type === 'percentage') {
            $discount = ($order->total * ($coupon->value / 100));
        } elseif ($coupon->type === 'fixed') {
            $discount = $coupon->value;
        }
        return min($discount, $order->total);
    }
}
```

Controlador Usando o Serviço:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Order;
use App\Services\DiscountService;
use Illuminate\Http\Request;

class OrderController extends Controller
{
    protected $discountService;

    public function __construct(DiscountService $discountService)
    {
        $this->discountService = $discountService;
    }

    public function applyCoupon(Request $request, $orderId)
    {
        // Validação
        $request->validate([
            'coupon_code' => 'required|string',
        ]);

        // Encontrar o pedido
        $order = Order::findOrFail($orderId);

        // Usar o serviço para aplicar o cupom
        $discount = $this->discountService->applyCoupon($order, $request->input('coupon_code'));

        if ($discount > 0) {
            return response()->json(['message' => 'Cupom aplicado com sucesso.', 'discount' => $discount]);
        } else {
            return response()->json(['message' => 'Cupom inválido.'], 400);
        }
    }
}
```

### Passo a Passo

1. **Criar o Serviço**:
    
    - **`DiscountService`** encapsula a lógica de negócios para aplicar um desconto com base em um cupom. Ele lida com a validação do cupom, o cálculo do desconto e a aplicação do desconto no pedido.

2. **Atualizar o Controlador**:
    
    - O controlador **`OrderController`** agora se concentra apenas em lidar com a requisição HTTP, validando os dados e usando o serviço para aplicar o cupom.
    - O controlador chama o método `applyCoupon` do serviço `DiscountService` para aplicar o desconto, simplificando a responsabilidade do controlador.

### Benefícios da Separação

1. **Responsabilidade Única**: O controlador agora é responsável apenas por gerenciar requisições e respostas, enquanto o serviço lida com a lógica de negócios.
2. **Facilidade de Manutenção**: Alterar a lógica de cálculo de desconto ou adicionar novos tipos de cupons se torna mais fácil, já que toda a lógica está centralizada no serviço.
3. **Testabilidade**: O `DiscountService` pode ser testado isoladamente sem depender do controlador. Você pode criar testes unitários específicos para a lógica de aplicação de descontos.

Essa abordagem ajuda a manter o código mais modular, limpo e fácil de gerenciar, seguindo o princípio de responsabilidade única e facilitando a manutenção e a evolução da aplicação.