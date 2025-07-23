# 📋 Documentação Backend: Sistema Order Bump, Upsell & Downsell

## 🎯 Visão Geral

Este documento especifica as APIs e funcionalidades backend necessárias para implementar o sistema completo de Order Bump, Upsell e Downsell no frontend.

## 🛒 Fluxo Completo

```
[ Checkout ] → [ Order Bump ] → [ Upsell ] → [ Downsell ] → [ Thank You ]
     ↓              ↓              ↓           ↓              ↓
   Pedido      Produto Adicional  Upgrade   Alternativa   Confirmação
   Principal   (mesmo checkout)   Premium   Mais Barata   Final
```

---

## 📡 APIs Necessárias

### 1. **Criação de Pedido Principal**

**Endpoint:** `POST /api/orders/create`

**Descrição:** Cria o pedido principal após checkout.

**Request Body:**
```json
{
  "productId": "string",
  "productSlug": "string", 
  "sellerId": "string",
  "customer": {
    "name": "string",
    "email": "string",
    "cpf": "string",
    "phone": "string"
  },
  "paymentMethod": "cartao|pix",
  "amount": "number",
  "orderBump": {
    "enabled": "boolean",
    "productId": "string",
    "added": "boolean"
  },
  "utmData": {},
  "metadata": {}
}
```

**Response:**
```json
{
  "id": "order_123456",
  "status": "confirmed|pending|failed",
  "customer": {},
  "items": [
    {
      "type": "main|order_bump",
      "productId": "string",
      "name": "string", 
      "price": "number",
      "quantity": "number"
    }
  ],
  "payment": {
    "method": "string",
    "status": "approved|pending|failed",
    "total": "number",
    "transactionId": "string"
  },
  "nextStep": "upsell|thank_you",
  "upsellUrl": "/upsell/order_123456",
  "createdAt": "datetime"
}
```

---

### 2. **Configuração de Order Bump**

**Endpoint:** `GET /api/products/{productId}/order-bump`

**Descrição:** Retorna configuração do order bump para um produto.

**Response:**
```json
{
  "enabled": true,
  "product": {
    "id": "bump_product_id",
    "name": "Garantia Estendida Premium",
    "description": "Cobertura total por 12 meses",
    "price": 29.90,
    "imageUrl": "/images/garantia.jpg"
  },
  "config": {
    "discountType": "percentage|fixed",
    "discountValue": 10,
    "title": "🛡️ Proteja seu investimento!",
    "description": "Adicione nossa garantia premium por apenas R$ 26,91",
    "badge": "OFERTA ESPECIAL",
    "benefits": [
      "Cobertura total por 12 meses",
      "Suporte prioritário 24/7",
      "Reposição gratuita em caso de defeito"
    ]
  }
}
```

---

### 3. **Adição de Order Bump**

**Endpoint:** `POST /api/orders/{orderId}/add-bump`

**Descrição:** Adiciona order bump ao pedido existente.

**Request Body:**
```json
{
  "bumpProductId": "string",
  "price": "number"
}
```

**Response:**
```json
{
  "success": true,
  "order": {
    "id": "order_123456",
    "items": [...],
    "payment": {
      "total": "number"
    }
  }
}
```

---

### 4. **Configuração de Upsell**

**Endpoint:** `GET /api/orders/{orderId}/upsell`

**Descrição:** Retorna dados do upsell baseado no pedido.

**Response:**
```json
{
  "available": true,
  "upsell": {
    "id": "upsell_1",
    "title": "Leve 2 Produtos + Frete Grátis!",
    "description": "Aproveite esta oferta exclusiva...",
    "imageUrl": "/images/upsell.jpg",
    "originalPrice": 197.00,
    "price": 147.00,
    "discount": 30,
    "benefits": [
      "Segundo produto idêntico GRÁTIS",
      "Frete grátis para todo o Brasil",
      "Suporte premium incluso"
    ],
    "testimonial": {
      "text": "Comprei o upsell e não me arrependo!",
      "author": "Maria Santos, São Paulo"
    },
    "timeLimit": 600
  }
}
```

---

### 5. **Adição de Upsell**

**Endpoint:** `POST /api/orders/{orderId}/add-upsell`

**Descrição:** Adiciona upsell ao pedido.

**Request Body:**
```json
{
  "upsellId": "string",
  "price": "number"
}
```

**Response:**
```json
{
  "success": true,
  "order": {
    "id": "order_123456",
    "items": [...],
    "payment": {
      "total": "number"
    }
  },
  "nextStep": "thank_you",
  "thankYouUrl": "/thank-you/order_123456"
}
```

---

### 6. **Configuração de Downsell**

**Endpoint:** `GET /api/orders/{orderId}/downsell`

**Descrição:** Retorna dados do downsell após recusa do upsell.

**Response:**
```json
{
  "available": true,
  "downsell": {
    "id": "downsell_1",
    "title": "Que tal só o essencial por menos?",
    "description": "Versão mais acessível...",
    "imageUrl": "/images/downsell.jpg",
    "originalPrice": 197.00,
    "price": 97.00,
    "discount": 50,
    "comparisonText": "Comparado à versão completa:",
    "problemSolved": "seu problema principal",
    "benefits": [
      "Conteúdo essencial do curso",
      "Acesso por 30 dias",
      "Suporte básico por email"
    ],
    "timeLimit": 300
  }
}
```

---

### 7. **Adição de Downsell**

**Endpoint:** `POST /api/orders/{orderId}/add-downsell`

**Descrição:** Adiciona downsell ao pedido.

**Request Body:**
```json
{
  "downsellId": "string", 
  "price": "number"
}
```

**Response:**
```json
{
  "success": true,
  "order": {
    "id": "order_123456",
    "items": [...],
    "payment": {
      "total": "number"
    }
  },
  "nextStep": "thank_you",
  "thankYouUrl": "/thank-you/order_123456"
}
```

---

### 8. **Detalhes Finais do Pedido**

**Endpoint:** `GET /api/orders/{orderId}/details`

**Descrição:** Retorna todos os detalhes do pedido para a página de obrigado.

**Response:**
```json
{
  "id": "order_123456",
  "status": "confirmed",
  "customer": {
    "name": "string",
    "email": "string"
  },
  "items": [
    {
      "type": "main|order_bump|upsell|downsell",
      "name": "string",
      "price": "number"
    }
  ],
  "payment": {
    "method": "cartao|pix",
    "status": "approved",
    "total": "number",
    "transactionId": "string"
  },
  "deliveryInfo": {
    "type": "digital|physical",
    "accessInfo": "string",
    "estimatedDelivery": "datetime"
  },
  "createdAt": "datetime"
}
```

---

## 🗄️ Estrutura do Banco de Dados

### Tabela: `orders`
```sql
CREATE TABLE orders (
  id VARCHAR(255) PRIMARY KEY,
  customer_id VARCHAR(255),
  product_id VARCHAR(255),
  seller_id VARCHAR(255),
  status ENUM('pending', 'confirmed', 'cancelled'),
  total_amount DECIMAL(10,2),
  payment_method ENUM('cartao', 'pix'),
  payment_status ENUM('pending', 'approved', 'failed'),
  transaction_id VARCHAR(255),
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);
```

### Tabela: `order_items`
```sql
CREATE TABLE order_items (
  id INT AUTO_INCREMENT PRIMARY KEY,
  order_id VARCHAR(255),
  product_id VARCHAR(255),
  item_type ENUM('main', 'order_bump', 'upsell', 'downsell'),
  name VARCHAR(255),
  price DECIMAL(10,2),
  quantity INT DEFAULT 1,
  FOREIGN KEY (order_id) REFERENCES orders(id)
);
```

### Tabela: `product_upsells`
```sql
CREATE TABLE product_upsells (
  id INT AUTO_INCREMENT PRIMARY KEY,
  product_id VARCHAR(255),
  upsell_type ENUM('order_bump', 'upsell', 'downsell'),
  title VARCHAR(255),
  description TEXT,
  image_url VARCHAR(500),
  original_price DECIMAL(10,2),
  price DECIMAL(10,2),
  discount_percentage INT,
  benefits JSON,
  enabled BOOLEAN DEFAULT true,
  created_at TIMESTAMP
);
```

---

## 🔄 Eventos e Webhooks

### Eventos para Analytics

**Evento:** `order.created`
```json
{
  "event": "order.created",
  "orderId": "order_123456",
  "customerId": "customer_789",
  "amount": 97.00,
  "timestamp": "datetime"
}
```

**Evento:** `order_bump.added`
```json
{
  "event": "order_bump.added", 
  "orderId": "order_123456",
  "bumpProductId": "bump_product_id",
  "addedAmount": 26.91,
  "timestamp": "datetime"
}
```

**Evento:** `upsell.accepted` / `upsell.declined`
```json
{
  "event": "upsell.accepted",
  "orderId": "order_123456", 
  "upsellId": "upsell_1",
  "addedAmount": 147.00,
  "timestamp": "datetime"
}
```

**Evento:** `downsell.accepted` / `downsell.declined`
```json
{
  "event": "downsell.accepted",
  "orderId": "order_123456",
  "downsellId": "downsell_1", 
  "addedAmount": 97.00,
  "timestamp": "datetime"
}
```

---

## 📊 Métricas e KPIs

### Dashboard de Conversão
- **Taxa de conversão Order Bump:** % de pedidos que adicionaram order bump
- **Taxa de conversão Upsell:** % de clientes que aceitaram upsell
- **Taxa de conversão Downsell:** % de clientes que aceitaram downsell após recusar upsell
- **Ticket médio:** Valor médio dos pedidos finais
- **Revenue por funil:** Receita adicional gerada por cada etapa

### Endpoint de Métricas
**Endpoint:** `GET /api/analytics/funnel-metrics`

**Query Params:**
- `startDate`: Data inicial (YYYY-MM-DD)
- `endDate`: Data final (YYYY-MM-DD)
- `productId`: Filtrar por produto (opcional)

**Response:**
```json
{
  "period": {
    "startDate": "2024-01-01",
    "endDate": "2024-01-31"
  },
  "totalOrders": 1000,
  "orderBump": {
    "conversionRate": 25.5,
    "totalAccepted": 255,
    "additionalRevenue": 6854.55
  },
  "upsell": {
    "conversionRate": 15.2,
    "totalAccepted": 152,
    "additionalRevenue": 22344.00
  },
  "downsell": {
    "conversionRate": 8.7,
    "totalAccepted": 87,
    "additionalRevenue": 8439.00
  },
  "averageOrderValue": {
    "beforeFunnel": 97.00,
    "afterFunnel": 134.23,
    "increase": 38.4
  }
}
```

---

## 🔐 Segurança e Validações

### Headers Obrigatórios
```
Content-Type: application/json
Authorization: Bearer {token}
X-Request-ID: {unique_request_id}
```

### Validações
1. **Verificar se order existe** antes de adicionar upsell/downsell
2. **Validar se customer é proprietário** do pedido
3. **Verificar timeout** das ofertas (upsell: 10min, downsell: 5min)
4. **Limitar tentativas** de adição por IP/session
5. **Validar se produto está ativo** e disponível

### Rate Limiting
- **Criação de pedidos:** 10 req/min por IP
- **Adição de ofertas:** 5 req/min por session
- **Consultas:** 60 req/min por usuário

---

## 🚀 Implementação Sugerida

### Fase 1: Backend Básico
1. ✅ APIs de criação e consulta de pedidos
2. ✅ Sistema de order bump
3. ✅ Estrutura básica do banco

### Fase 2: Funil Completo  
1. ✅ APIs de upsell e downsell
2. ✅ Sistema de timeouts e expiração
3. ✅ Webhook para eventos

### Fase 3: Analytics e Otimização
1. ✅ Dashboard de métricas
2. ✅ A/B testing de ofertas
3. ✅ Machine learning para ofertas personalizadas

---

## 📞 Próximos Passos

1. **Implementar APIs básicas** de criação de pedido
2. **Configurar estrutura do banco** com as tabelas especificadas  
3. **Implementar sistema de eventos** para tracking
4. **Criar dashboard** para acompanhar métricas
5. **Testar fluxo completo** com dados reais

---

## 🔧 Configurações de Desenvolvimento

### Variáveis de Ambiente
```env
# Banco de dados
DB_HOST=localhost
DB_PORT=5432
DB_NAME=checkout_system
DB_USER=postgres
DB_PASS=password

# APIs de pagamento
STRIPE_SECRET_KEY=sk_test_...
MERCADOPAGO_ACCESS_TOKEN=TEST-...

# URLs do frontend
FRONTEND_URL=http://localhost:3000
UPSELL_URL=http://localhost:3000/upsell
DOWNSELL_URL=http://localhost:3000/downsell
THANKYOU_URL=http://localhost:3000/thank-you

# Configurações de timeout
UPSELL_TIMEOUT_MINUTES=10
DOWNSELL_TIMEOUT_MINUTES=5
```

### Scripts de Migração
```sql
-- Migration: Create funnel tables
-- File: migrations/001_create_funnel_tables.sql

-- [Incluir aqui os scripts SQL completos das tabelas]
```

---

**📝 Nota:** Esta documentação serve como especificação completa para implementação do backend. Ajustar conforme necessidades específicas do projeto e stack tecnológica escolhida.
