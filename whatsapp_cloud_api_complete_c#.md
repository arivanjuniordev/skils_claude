---
title: WhatsApp Cloud API
description: >
  API REST hospedada pela Meta para envio e recebimento de mensagens WhatsApp
  via HTTP. Permite integração de chatbots, notificações, atendimento ao cliente
  e automações sem infraestrutura própria.
source: https://developers.facebook.com/docs/whatsapp/cloud-api
version: v22.0
updated: 2026-02
tags: [whatsapp, messaging, rest-api, webhooks, meta]
---

# WhatsApp Cloud API — Documentação Completa

## Índice

1. [Visão Geral e Arquitetura](#1-visão-geral-e-arquitetura)
2. [Configuração Inicial](#2-configuração-inicial)
3. [Autenticação](#3-autenticação)
4. [Endpoints de Mensagens](#4-endpoints-de-mensagens)
5. [Tipos de Mensagem (Outbound)](#5-tipos-de-mensagem-outbound)
   - 5.1 Texto
   - 5.2 Imagem
   - 5.3 Vídeo
   - 5.4 Áudio
   - 5.5 Documento
   - 5.6 Localização
   - 5.7 Template
   - 5.8 Interativa com Botões (Reply Buttons)
   - 5.9 Interativa com Lista (List Message)
   - 5.10 Reação
   - 5.11 Contato
6. [Upload de Mídia](#6-upload-de-mídia)
7. [Webhooks — Configuração](#7-webhooks--configuração)
8. [Webhooks — Payloads de Entrada](#8-webhooks--payloads-de-entrada)
   - 8.1 Texto recebido
   - 8.2 Mídia recebida (image/video/audio/sticker)
   - 8.3 Reação recebida
   - 8.4 Localização recebida
   - 8.5 Contato recebido
   - 8.6 Botão de template clicado
   - 8.7 List Reply (lista interativa)
   - 8.8 Button Reply (botão interativo)
   - 8.9 Click-to-WhatsApp Ad
   - 8.10 Pedido (Order)
   - 8.11 Mensagem desconhecida/não suportada
   - 8.12 Status de entrega
9. [Modelos C# para Deserialização](#9-modelos-c-para-deserialização)
10. [Regras de Negócio Críticas](#10-regras-de-negócio-críticas)
11. [Rate Limits](#11-rate-limits)
12. [Códigos de Erro](#12-códigos-de-erro)
13. [Verificação do Webhook (GET)](#13-verificação-do-webhook-get)
14. [Registro de Número de Telefone](#14-registro-de-número-de-telefone)

---

## 1. Visão Geral e Arquitetura

```
Seu Backend (ASP.NET Core)
       │
       ├── POST /v22.0/{PHONE_NUMBER_ID}/messages  ──▶  Meta Cloud API
       │                                                      │
       └── GET|POST /webhook  ◀──────────────────────────────┘
                                  (notificações em tempo real)
```

**Base URL:**
```
https://graph.facebook.com/v22.0
```

**Permissões necessárias no App:**
- `whatsapp_business_messaging` — envio e recebimento de mensagens
- `whatsapp_business_management` — gerenciamento de números e templates

---

## 2. Configuração Inicial

### Passo a passo

1. Acesse [developers.facebook.com](https://developers.facebook.com) e crie ou selecione um app
2. Adicione o produto **WhatsApp** ao app
3. Vincule (ou crie) uma **Meta Business Account (MBA)**
4. No **API Setup**, adicione até 5 números de destinatários de teste
5. Copie o **Phone Number ID** e o **Access Token** temporário
6. Para produção: crie um **System User** no Business Manager e gere um token permanente

---

## 3. Autenticação

Todas as requisições usam **Bearer Token** no header HTTP.

```http
Authorization: Bearer {ACCESS_TOKEN}
Content-Type: application/json
```

| Tipo de Token       | Validade     | Uso                            |
|---------------------|--------------|--------------------------------|
| Temporary Token     | ~24 horas    | Testes no App Dashboard        |
| System User Token   | Permanente   | Produção via Business Manager  |

### Exemplo de header C# (HttpClient)

```csharp
_httpClient.DefaultRequestHeaders.Authorization =
    new AuthenticationHeaderValue("Bearer", _options.AccessToken);
_httpClient.DefaultRequestHeaders.Accept
    .Add(new MediaTypeWithQualityHeaderValue("application/json"));
```

---

## 4. Endpoints de Mensagens

### Enviar mensagem

```
POST https://graph.facebook.com/v22.0/{PHONE_NUMBER_ID}/messages
```

### Campos obrigatórios (todos os tipos)

| Campo               | Tipo   | Descrição                                                  |
|---------------------|--------|------------------------------------------------------------|
| `messaging_product` | string | Sempre `"whatsapp"`                                        |
| `to`                | string | Número com código do país, sem `+` (ex: `"5584999999999"`) |
| `type`              | string | Tipo da mensagem (ver seção 5)                             |
| `recipient_type`    | string | Opcional. Padrão: `"individual"`                           |

### Resposta de sucesso

```json
{
  "messaging_product": "whatsapp",
  "contacts": [{ "input": "5584999999999", "wa_id": "5584999999999" }],
  "messages": [{ "id": "wamid.HBgN..." }]
}
```

---

## 5. Tipos de Mensagem (Outbound)

> ⚠️ Mensagens de texto, mídia e interativas só podem ser enviadas dentro da **janela de 24h** após a última mensagem do usuário. Fora dela, use apenas **templates**.

---

### 5.1 Texto

```json
{
  "messaging_product": "whatsapp",
  "to": "5584999999999",
  "type": "text",
  "text": {
    "preview_url": true,
    "body": "Olá! Bem-vindo ao nosso serviço. Acesse: https://exemplo.com"
  }
}
```

| Campo         | Tipo    | Obrigatório | Descrição                              |
|---------------|---------|-------------|----------------------------------------|
| `body`        | string  | ✅           | Texto da mensagem (máx. 4096 chars)    |
| `preview_url` | boolean | ❌           | Habilita preview de URL no WhatsApp    |

---

### 5.2 Imagem

```json
{
  "messaging_product": "whatsapp",
  "to": "5584999999999",
  "type": "image",
  "image": {
    "link": "https://exemplo.com/imagem.jpg",
    "caption": "Legenda opcional"
  }
}
```

> Use `"id": "MEDIA_ID"` no lugar de `"link"` para mídias previamente enviadas via upload.

---

### 5.3 Vídeo

```json
{
  "messaging_product": "whatsapp",
  "to": "5584999999999",
  "type": "video",
  "video": {
    "link": "https://exemplo.com/video.mp4",
    "caption": "Assista nosso tutorial"
  }
}
```

---

### 5.4 Áudio

```json
{
  "messaging_product": "whatsapp",
  "to": "5584999999999",
  "type": "audio",
  "audio": {
    "link": "https://exemplo.com/audio.mp3"
  }
}
```

---

### 5.5 Documento

```json
{
  "messaging_product": "whatsapp",
  "to": "5584999999999",
  "type": "document",
  "document": {
    "link": "https://exemplo.com/contrato.pdf",
    "caption": "Contrato de serviço",
    "filename": "contrato.pdf"
  }
}
```

---

### 5.6 Localização

```json
{
  "messaging_product": "whatsapp",
  "to": "5584999999999",
  "type": "location",
  "location": {
    "longitude": -35.9787,
    "latitude": -5.7945,
    "name": "Bom Jardim, RN",
    "address": "Bom Jardim, Rio Grande do Norte, Brasil"
  }
}
```

---

### 5.7 Template

> Podem ser enviados **a qualquer momento**, independente da janela de 24h.

```json
{
  "messaging_product": "whatsapp",
  "to": "5584999999999",
  "type": "template",
  "template": {
    "name": "nome_do_template",
    "language": { "code": "pt_BR" },
    "components": [
      {
        "type": "header",
        "parameters": [
          { "type": "image", "image": { "link": "https://exemplo.com/banner.jpg" } }
        ]
      },
      {
        "type": "body",
        "parameters": [
          { "type": "text", "text": "Arivan" },
          { "type": "text", "text": "10/03/2026" }
        ]
      },
      {
        "type": "button",
        "sub_type": "quick_reply",
        "index": "0",
        "parameters": [{ "type": "payload", "payload": "CONFIRMAR_PEDIDO" }]
      }
    ]
  }
}
```

**Categorias de template:**

| Categoria     | Uso                                          |
|---------------|----------------------------------------------|
| `MARKETING`   | Promoções, ofertas, novidades                |
| `UTILITY`     | Confirmações, atualizações de pedido, alertas|
| `AUTHENTICATION` | OTPs, verificação de identidade           |

---

### 5.8 Interativa com Botões (Reply Buttons)

> Máx. 3 botões. Apenas dentro da janela de 24h.

```json
{
  "messaging_product": "whatsapp",
  "to": "5584999999999",
  "type": "interactive",
  "interactive": {
    "type": "button",
    "header": {
      "type": "text",
      "text": "Confirmação de agendamento"
    },
    "body": {
      "text": "Deseja confirmar sua consulta para amanhã às 14h?"
    },
    "footer": {
      "text": "Responda abaixo"
    },
    "action": {
      "buttons": [
        { "type": "reply", "reply": { "id": "btn_sim", "title": "Sim, confirmar" } },
        { "type": "reply", "reply": { "id": "btn_nao", "title": "Não, cancelar" } },
        { "type": "reply", "reply": { "id": "btn_remarcar", "title": "Remarcar" } }
      ]
    }
  }
}
```

---

### 5.9 Interativa com Lista (List Message)

> Máx. 10 seções, 10 itens por seção. Apenas dentro da janela de 24h.

```json
{
  "messaging_product": "whatsapp",
  "to": "5584999999999",
  "type": "interactive",
  "interactive": {
    "type": "list",
    "header": { "type": "text", "text": "Nossos serviços" },
    "body": { "text": "Escolha uma opção:" },
    "footer": { "text": "Atendimento 24/7" },
    "action": {
      "button": "Ver opções",
      "sections": [
        {
          "title": "Suporte",
          "rows": [
            { "id": "row_1", "title": "Abrir chamado", "description": "Registrar novo problema" },
            { "id": "row_2", "title": "Status do chamado", "description": "Verificar chamados abertos" }
          ]
        },
        {
          "title": "Vendas",
          "rows": [
            { "id": "row_3", "title": "Solicitar orçamento", "description": "Receba uma proposta" }
          ]
        }
      ]
    }
  }
}
```

---

### 5.10 Reação

```json
{
  "messaging_product": "whatsapp",
  "to": "5584999999999",
  "type": "reaction",
  "reaction": {
    "message_id": "wamid.HBgN...",
    "emoji": "👍"
  }
}
```

---

### 5.11 Contato

```json
{
  "messaging_product": "whatsapp",
  "to": "5584999999999",
  "type": "contacts",
  "contacts": [
    {
      "name": { "formatted_name": "João Silva", "first_name": "João", "last_name": "Silva" },
      "phones": [{ "phone": "+5584999999999", "type": "CELL", "wa_id": "5584999999999" }],
      "emails": [{ "email": "joao@exemplo.com", "type": "WORK" }],
      "org": { "company": "Minha Empresa", "title": "Desenvolvedor" }
    }
  ]
}
```

---

## 6. Upload de Mídia

### Upload

```
POST https://graph.facebook.com/v22.0/{PHONE_NUMBER_ID}/media
Content-Type: multipart/form-data
Authorization: Bearer {ACCESS_TOKEN}
```

```bash
curl -X POST \
  "https://graph.facebook.com/v22.0/{PHONE_NUMBER_ID}/media" \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  -F "file=@/caminho/do/arquivo.pdf;type=application/pdf" \
  -F "messaging_product=whatsapp"
```

**Resposta:**
```json
{ "id": "1234567890" }
```

### Recuperar URL de mídia

```
GET https://graph.facebook.com/v22.0/{MEDIA_ID}
Authorization: Bearer {ACCESS_TOKEN}
```

**Resposta:**
```json
{
  "messaging_product": "whatsapp",
  "url": "https://lookaside.fbsbx.com/...",
  "mime_type": "application/pdf",
  "sha256": "HASH",
  "file_size": "123456",
  "id": "1234567890"
}
```

> ⚠️ A URL retornada expira em **5 minutos**. Faça download imediato.

### Deletar mídia

```
DELETE https://graph.facebook.com/v22.0/{MEDIA_ID}
Authorization: Bearer {ACCESS_TOKEN}
```

### Tipos de mídia suportados

| Tipo       | MIME Types aceitos                                    | Tamanho máx. |
|------------|-------------------------------------------------------|--------------|
| image      | image/jpeg, image/png                                 | 5 MB         |
| audio      | audio/aac, audio/mp4, audio/mpeg, audio/ogg           | 16 MB        |
| video      | video/mp4, video/3gp                                  | 16 MB        |
| document   | application/pdf, text/plain, application/msword, etc. | 100 MB       |
| sticker    | image/webp                                            | 500 KB       |

---

## 7. Webhooks — Configuração

### Setup no App Dashboard

1. Vá em **WhatsApp** → **Configuration** → **Webhook**
2. Defina a **Callback URL** (HTTPS obrigatório, sem self-signed)
3. Defina o **Verify Token** (string qualquer, ex: `"meu-verify-token-secreto"`)
4. Clique em **Verify and Save**
5. Em **Webhook Fields**, ative o campo `messages`

### Verificação inicial (GET)

A Meta envia um GET para validar a URL:

```
GET {CALLBACK_URL}?hub.mode=subscribe&hub.verify_token={SEU_TOKEN}&hub.challenge={VALOR}
```

Responda com o valor de `hub.challenge` e status **200**.

```csharp
// ASP.NET Core — Program.cs ou Controller
app.MapGet("/webhook", (HttpContext ctx) =>
{
    var mode = ctx.Request.Query["hub.mode"].ToString();
    var token = ctx.Request.Query["hub.verify_token"].ToString();
    var challenge = ctx.Request.Query["hub.challenge"].ToString();

    if (mode == "subscribe" && token == configuration["WhatsApp:VerifyToken"])
        return Results.Ok(challenge);

    return Results.Forbid();
});
```

### Recebimento de notificações (POST)

```csharp
app.MapPost("/webhook", async (HttpContext ctx) =>
{
    var body = await new StreamReader(ctx.Request.Body).ReadToEndAsync();
    var payload = JsonConvert.DeserializeObject<WhatsAppWebhookPayload>(body);

    // processar mensagem...

    return Results.Ok(); // sempre retornar 200 imediatamente
});
```

> ⚠️ **Retorne 200 imediatamente.** Se demorar mais de 20s, a Meta reenviará o evento. Use filas (ex: BackgroundService, RabbitMQ) para processamento assíncrono.

---

## 8. Webhooks — Payloads de Entrada

### Estrutura raiz (comum a todos)

```json
{
  "object": "whatsapp_business_account",
  "entry": [{
    "id": "<WABA_ID>",
    "changes": [{
      "value": {
        "messaging_product": "whatsapp",
        "metadata": {
          "display_phone_number": "5584999999999",
          "phone_number_id": "<PHONE_NUMBER_ID>"
        },
        "contacts": [{ "profile": { "name": "Nome Usuário" }, "wa_id": "5584999999999" }],
        "messages": [ /* ou "statuses": [] */ ]
      },
      "field": "messages"
    }]
  }]
}
```

---

### 8.1 Texto recebido

```json
{
  "messages": [{
    "from": "5584999999999",
    "id": "wamid.ID",
    "timestamp": "1234567890",
    "type": "text",
    "text": { "body": "Olá, preciso de ajuda!" }
  }]
}
```

**Deserialização C#:**
```csharp
var msg = payload.Entry[0].Changes[0].Value.Messages[0];
var senderName = payload.Entry[0].Changes[0].Value.Contacts[0].Profile.Name;
var senderNumber = msg.From;
var text = msg.Text.Body;
```

---

### 8.2 Mídia recebida (image/video/audio/sticker)

```json
{
  "messages": [{
    "from": "5584999999999",
    "id": "wamid.ID",
    "timestamp": "1234567890",
    "type": "image",
    "image": {
      "caption": "Legenda se houver",
      "mime_type": "image/jpeg",
      "sha256": "HASH",
      "id": "MEDIA_ID"
    }
  }]
}
```

**Deserialização C# (buscar URL da mídia):**
```csharp
var mediaId = msg.Image?.Id ?? msg.Video?.Id ?? msg.Audio?.Id ?? msg.Document?.Id;
var mediaResponse = await httpClient.GetAsync(
    $"https://graph.facebook.com/v22.0/{mediaId}",
    bearerToken
);
// mediaResponse.url expira em 5 minutos — faça download imediato
```

---

### 8.3 Reação recebida

```json
{
  "messages": [{
    "from": "5584999999999",
    "id": "wamid.ID",
    "timestamp": "1234567890",
    "type": "reaction",
    "reaction": {
      "message_id": "wamid.MENSAGEM_ORIGINAL",
      "emoji": "❤️"
    }
  }]
}
```

> Não recebido para mensagens com mais de 30 dias.

---

### 8.4 Localização recebida

```json
{
  "messages": [{
    "type": "location",
    "location": {
      "latitude": -5.7945,
      "longitude": -35.9787,
      "name": "Bom Jardim, RN",
      "address": "Bom Jardim, Rio Grande do Norte"
    }
  }]
}
```

---

### 8.5 Contato recebido

```json
{
  "messages": [{
    "type": "contacts",
    "contacts": [{
      "name": { "formatted_name": "Maria Silva", "first_name": "Maria" },
      "phones": [{ "phone": "+5584988887777", "type": "CELL" }],
      "emails": [{ "email": "maria@exemplo.com", "type": "HOME" }]
    }]
  }]
}
```

---

### 8.6 Botão de template clicado (Quick Reply)

```json
{
  "messages": [{
    "type": "button",
    "button": {
      "text": "Sim, confirmar",
      "payload": "CONFIRMAR_PEDIDO"
    },
    "context": {
      "from": "5584999999999",
      "id": "wamid.MENSAGEM_ORIGINAL"
    }
  }]
}
```

**C#:**
```csharp
var buttonPayload = msg.Button.Payload; // "CONFIRMAR_PEDIDO"
var buttonText = msg.Button.Text;
```

---

### 8.7 List Reply (seleção em lista interativa)

```json
{
  "messages": [{
    "type": "interactive",
    "interactive": {
      "type": "list_reply",
      "list_reply": {
        "id": "row_1",
        "title": "Abrir chamado",
        "description": "Registrar novo problema"
      }
    }
  }]
}
```

**C#:**
```csharp
var listReply = msg.Interactive.ListReply;
var selectedId = listReply.Id;       // "row_1"
var selectedTitle = listReply.Title; // "Abrir chamado"
```

---

### 8.8 Button Reply (clique em botão interativo)

```json
{
  "messages": [{
    "type": "interactive",
    "interactive": {
      "type": "button_reply",
      "button_reply": {
        "id": "btn_sim",
        "title": "Sim, confirmar"
      }
    }
  }]
}
```

**C#:**
```csharp
var buttonReply = msg.Interactive.ButtonReply;
var btnId = buttonReply.Id;       // "btn_sim"
var btnTitle = buttonReply.Title; // "Sim, confirmar"
```

---

### 8.9 Click-to-WhatsApp Ad (referral)

```json
{
  "messages": [{
    "type": "text",
    "text": { "body": "Texto enviado pelo usuário" },
    "referral": {
      "source_url": "https://fb.com/ad/123",
      "source_id": "AD_ID",
      "source_type": "ad",
      "headline": "Título do anúncio",
      "body": "Descrição do anúncio",
      "media_type": "image",
      "image_url": "https://exemplo.com/imagem.jpg",
      "ctwa_clid": "CTWA_CLID"
    }
  }]
}
```

---

### 8.10 Pedido (Order)

```json
{
  "messages": [{
    "type": "order",
    "order": {
      "catalog_id": "CATALOG_ID",
      "text": "Quero esses produtos",
      "product_items": [
        {
          "product_retailer_id": "SKU-001",
          "quantity": "2",
          "item_price": "49.90",
          "currency": "BRL"
        }
      ]
    }
  }]
}
```

---

### 8.11 Mensagem desconhecida/não suportada

```json
{
  "messages": [{
    "type": "unknown",
    "errors": [{
      "code": 131051,
      "title": "Unsupported message type",
      "details": "Message type is not currently supported"
    }]
  }]
}
```

---

### 8.12 Status de entrega

O campo `statuses` substitui `messages` quando o evento é de status.

```json
{
  "statuses": [{
    "id": "wamid.HBgN...",
    "status": "delivered",
    "timestamp": "1234567890",
    "recipient_id": "5584999999999",
    "conversation": {
      "id": "CONVERSATION_ID",
      "origin": { "type": "utility" }
    },
    "pricing": {
      "billable": true,
      "pricing_model": "CBP",
      "category": "utility"
    }
  }]
}
```

| status      | Descrição                                      |
|-------------|------------------------------------------------|
| `sent`      | Aceita pelo servidor da Meta                   |
| `delivered` | Entregue ao dispositivo do destinatário        |
| `read`      | Lida pelo destinatário                         |
| `failed`    | Falha no envio — verifique o campo `errors`    |
| `deleted`   | Mensagem deletada pelo usuário                 |

---

## 9. Modelos C# para Deserialização

```csharp
using Newtonsoft.Json;
using System.Collections.Generic;

public class WhatsAppWebhookPayload
{
    [JsonProperty("object")] public string Object { get; set; }
    [JsonProperty("entry")] public List<Entry> Entry { get; set; }
}

public class Entry
{
    [JsonProperty("id")] public string Id { get; set; }
    [JsonProperty("changes")] public List<Change> Changes { get; set; }
}

public class Change
{
    [JsonProperty("value")] public Value Value { get; set; }
    [JsonProperty("field")] public string Field { get; set; }
}

public class Value
{
    [JsonProperty("messaging_product")] public string MessagingProduct { get; set; }
    [JsonProperty("metadata")] public Metadata Metadata { get; set; }
    [JsonProperty("contacts")] public List<Contact> Contacts { get; set; }
    [JsonProperty("messages")] public List<Message> Messages { get; set; }
    [JsonProperty("statuses")] public List<MessageStatus> Statuses { get; set; }
}

public class Metadata
{
    [JsonProperty("display_phone_number")] public string DisplayPhoneNumber { get; set; }
    [JsonProperty("phone_number_id")] public string PhoneNumberId { get; set; }
}

public class Contact
{
    [JsonProperty("profile")] public Profile Profile { get; set; }
    [JsonProperty("wa_id")] public string WaId { get; set; }
}

public class Profile
{
    [JsonProperty("name")] public string Name { get; set; }
}

public class Message
{
    [JsonProperty("from")] public string From { get; set; }
    [JsonProperty("id")] public string Id { get; set; }
    [JsonProperty("timestamp")] public string Timestamp { get; set; }
    [JsonProperty("type")] public string Type { get; set; }
    [JsonProperty("text")] public TextContent Text { get; set; }
    [JsonProperty("image")] public MediaContent Image { get; set; }
    [JsonProperty("video")] public MediaContent Video { get; set; }
    [JsonProperty("audio")] public MediaContent Audio { get; set; }
    [JsonProperty("document")] public MediaContent Document { get; set; }
    [JsonProperty("sticker")] public StickerContent Sticker { get; set; }
    [JsonProperty("reaction")] public Reaction Reaction { get; set; }
    [JsonProperty("location")] public LocationContent Location { get; set; }
    [JsonProperty("contacts")] public List<MessageContact> Contacts { get; set; }
    [JsonProperty("button")] public ButtonContent Button { get; set; }
    [JsonProperty("interactive")] public InteractiveContent Interactive { get; set; }
    [JsonProperty("referral")] public ReferralContent Referral { get; set; }
    [JsonProperty("order")] public OrderContent Order { get; set; }
    [JsonProperty("context")] public Context Context { get; set; }
    [JsonProperty("errors")] public List<ErrorContent> Errors { get; set; }
}

public class TextContent { [JsonProperty("body")] public string Body { get; set; } }
public class MediaContent {
    [JsonProperty("id")] public string Id { get; set; }
    [JsonProperty("mime_type")] public string MimeType { get; set; }
    [JsonProperty("sha256")] public string Sha256 { get; set; }
    [JsonProperty("caption")] public string Caption { get; set; }
    [JsonProperty("filename")] public string Filename { get; set; }
}
public class StickerContent {
    [JsonProperty("id")] public string Id { get; set; }
    [JsonProperty("mime_type")] public string MimeType { get; set; }
    [JsonProperty("sha256")] public string Sha256 { get; set; }
    [JsonProperty("animated")] public bool Animated { get; set; }
}
public class Reaction {
    [JsonProperty("message_id")] public string MessageId { get; set; }
    [JsonProperty("emoji")] public string Emoji { get; set; }
}
public class LocationContent {
    [JsonProperty("latitude")] public double? Latitude { get; set; }
    [JsonProperty("longitude")] public double? Longitude { get; set; }
    [JsonProperty("name")] public string Name { get; set; }
    [JsonProperty("address")] public string Address { get; set; }
}
public class ButtonContent {
    [JsonProperty("text")] public string Text { get; set; }
    [JsonProperty("payload")] public string Payload { get; set; }
}
public class InteractiveContent {
    [JsonProperty("type")] public string Type { get; set; }
    [JsonProperty("list_reply")] public ListReply ListReply { get; set; }
    [JsonProperty("button_reply")] public ButtonReply ButtonReply { get; set; }
}
public class ListReply {
    [JsonProperty("id")] public string Id { get; set; }
    [JsonProperty("title")] public string Title { get; set; }
    [JsonProperty("description")] public string Description { get; set; }
}
public class ButtonReply {
    [JsonProperty("id")] public string Id { get; set; }
    [JsonProperty("title")] public string Title { get; set; }
}
public class ReferralContent {
    [JsonProperty("source_url")] public string SourceUrl { get; set; }
    [JsonProperty("source_id")] public string SourceId { get; set; }
    [JsonProperty("source_type")] public string SourceType { get; set; }
    [JsonProperty("headline")] public string Headline { get; set; }
    [JsonProperty("body")] public string Body { get; set; }
    [JsonProperty("media_type")] public string MediaType { get; set; }
    [JsonProperty("image_url")] public string ImageUrl { get; set; }
    [JsonProperty("ctwa_clid")] public string CtwaClid { get; set; }
}
public class OrderContent {
    [JsonProperty("catalog_id")] public string CatalogId { get; set; }
    [JsonProperty("text")] public string Text { get; set; }
    [JsonProperty("product_items")] public List<ProductItem> ProductItems { get; set; }
}
public class ProductItem {
    [JsonProperty("product_retailer_id")] public string ProductRetailerId { get; set; }
    [JsonProperty("quantity")] public string Quantity { get; set; }
    [JsonProperty("item_price")] public string ItemPrice { get; set; }
    [JsonProperty("currency")] public string Currency { get; set; }
}
public class Context {
    [JsonProperty("from")] public string From { get; set; }
    [JsonProperty("id")] public string Id { get; set; }
    [JsonProperty("referred_product")] public ReferredProduct ReferredProduct { get; set; }
}
public class ReferredProduct {
    [JsonProperty("catalog_id")] public string CatalogId { get; set; }
    [JsonProperty("product_retailer_id")] public string ProductRetailerId { get; set; }
}
public class ErrorContent {
    [JsonProperty("code")] public int Code { get; set; }
    [JsonProperty("title")] public string Title { get; set; }
    [JsonProperty("details")] public string Details { get; set; }
}
public class MessageContact {
    [JsonProperty("addresses")] public List<Address> Addresses { get; set; }
    [JsonProperty("birthday")] public string Birthday { get; set; }
    [JsonProperty("emails")] public List<ContactEmail> Emails { get; set; }
    [JsonProperty("name")] public ContactName Name { get; set; }
    [JsonProperty("org")] public Organization Org { get; set; }
    [JsonProperty("phones")] public List<Phone> Phones { get; set; }
    [JsonProperty("urls")] public List<ContactUrl> Urls { get; set; }
}
public class Address {
    [JsonProperty("city")] public string City { get; set; }
    [JsonProperty("country")] public string Country { get; set; }
    [JsonProperty("country_code")] public string CountryCode { get; set; }
    [JsonProperty("state")] public string State { get; set; }
    [JsonProperty("street")] public string Street { get; set; }
    [JsonProperty("type")] public string Type { get; set; }
    [JsonProperty("zip")] public string Zip { get; set; }
}
public class ContactEmail {
    [JsonProperty("email")] public string Email { get; set; }
    [JsonProperty("type")] public string Type { get; set; }
}
public class ContactName {
    [JsonProperty("formatted_name")] public string FormattedName { get; set; }
    [JsonProperty("first_name")] public string FirstName { get; set; }
    [JsonProperty("last_name")] public string LastName { get; set; }
}
public class Organization {
    [JsonProperty("company")] public string Company { get; set; }
    [JsonProperty("department")] public string Department { get; set; }
    [JsonProperty("title")] public string Title { get; set; }
}
public class Phone {
    [JsonProperty("phone")] public string PhoneNumber { get; set; }
    [JsonProperty("wa_id")] public string WaId { get; set; }
    [JsonProperty("type")] public string Type { get; set; }
}
public class ContactUrl {
    [JsonProperty("url")] public string Url { get; set; }
    [JsonProperty("type")] public string Type { get; set; }
}
public class MessageStatus {
    [JsonProperty("id")] public string Id { get; set; }
    [JsonProperty("status")] public string Status { get; set; }
    [JsonProperty("timestamp")] public string Timestamp { get; set; }
    [JsonProperty("recipient_id")] public string RecipientId { get; set; }
    [JsonProperty("errors")] public List<ErrorContent> Errors { get; set; }
}
```

---

## 10. Regras de Negócio Críticas

### Janela de 24 horas (Customer Service Window)

| Situação                             | Tipos permitidos                             |
|--------------------------------------|----------------------------------------------|
| Usuário enviou mensagem < 24h atrás  | Texto, mídia, interativa, template           |
| Usuário enviou mensagem > 24h atrás  | **Apenas templates** (marketing/utility/auth)|
| Usuário nunca enviou mensagem        | **Apenas templates**                         |

### Cache de mídia

> A API faz cache de links de mídia por **10 minutos**. Se atualizar a imagem no servidor mantendo a mesma URL, o usuário receberá a versão antiga. Use query string para forçar refresh: `?v=2`.

### Time-To-Live (TTL)

- Padrão: **30 dias**. Se o dispositivo do usuário estiver offline por mais de 30 dias, a mensagem é descartada.
- Customize com `"ttl": { "value": 3600 }` para mensagens sensíveis ao tempo (ex: OTP).

### Quality Rating

- Se usuários bloquearem sua conta ou reportarem spam, o **quality rating** cai.
- Rating baixo reduz automaticamente os rate limits.
- Mantenha taxa de opt-out abaixo de 3%.

---

## 11. Rate Limits

| Limite                              | Valor                              |
|-------------------------------------|------------------------------------|
| Mensagens por segundo (padrão)      | 80 mps por número registrado       |
| Mensagens por segundo (upgradeable) | 1.000 mps (elegível automaticamente)|
| API Management calls (sem WABA ativo)| 200 calls/hora por app por WABA   |
| API Management calls (com WABA ativo)| 5.000 calls/hora por app por WABA |
| Máx. tentativas de registro de número| 10 por número em 72 horas         |
| Destinatários únicos por template   | Sem limite definido (cuidado com spam)|

---

## 12. Códigos de Erro

### Estrutura de resposta de erro

```json
{
  "error": {
    "message": "(#130429) Rate limit hit",
    "type": "OAuthException",
    "code": 130429,
    "error_subcode": 2494055,
    "fbtrace_id": "AbCdEfGh"
  }
}
```

### Tabela de erros comuns

| Código   | Nome                            | Causa & Solução                                                              |
|----------|---------------------------------|------------------------------------------------------------------------------|
| `3`      | API Method                      | Token sem permissão `whatsapp_business_messaging`. Verificar escopo.         |
| `4`      | API Too Many Calls              | Limite de management API. Aguardar ou usar cache local.                      |
| `10`     | Permission Denied               | App sem permissão. Verifique App Review e permissões.                        |
| `100`    | Invalid Parameter               | Parâmetro inválido no payload. Checar documentação do tipo de mensagem.      |
| `130429` | Rate Limit Hit                  | Throughput excedido. Implementar backoff exponencial.                        |
| `130472` | User's number is part of experiment | Número em teste A/B da Meta. Tentar novamente mais tarde.              |
| `131000` | Something Went Wrong            | Erro genérico. Checar payload e retry.                                       |
| `131005` | Access Denied                   | Sem permissão para enviar para este número.                                  |
| `131008` | Required Parameter Missing      | Campo obrigatório ausente no payload.                                        |
| `131009` | Parameter Value Not Allowed     | Valor inválido em algum campo.                                               |
| `131021` | Recipient Not In Allowed List   | Em modo de desenvolvimento, destinatário não adicionado na lista de teste.   |
| `131026` | Message Undeliverable           | Mensagem não entregável. Usuário pode ter bloqueado o número.                |
| `131047` | Re-engagement Message           | Tentou enviar mensagem fora da janela 24h sem usar template.                 |
| `131048` | Spam Rate Limit Hit             | Volume considerado spam. Reduzir frequência.                                 |
| `131051` | Unsupported Message Type        | Tipo de mensagem não suportado para este destinatário.                       |
| `131056` | Pair Rate Limit Hit             | Muitas mensagens para o mesmo número em pouco tempo.                         |
| `132000` | Template Count Limit Reached    | Limite de templates atingido.                                                |
| `132001` | Template Does Not Exist         | Template com esse nome/idioma não existe.                                    |
| `132005` | Template Hydration Error        | Parâmetros do template não correspondem ao aprovado.                         |
| `133000` | Incomplete Deregistration       | Número não completou o processo de registro.                                 |
| `133016` | Register Rate Limit             | Máx. 10 tentativas de registro por 72 horas.                                 |

---

## 13. Verificação do Webhook (GET)

```csharp
// Exemplo completo ASP.NET Core Minimal API
app.MapGet("/webhook", (
    [FromQuery(Name = "hub.mode")] string mode,
    [FromQuery(Name = "hub.verify_token")] string token,
    [FromQuery(Name = "hub.challenge")] string challenge,
    IConfiguration config) =>
{
    if (mode == "subscribe" && token == config["WhatsApp:VerifyToken"])
        return Results.Ok(challenge);

    return Results.Forbid();
});
```

---

## 14. Registro de Número de Telefone

### Listar números da WABA

```
GET https://graph.facebook.com/v22.0/{WABA_ID}/phone_numbers
Authorization: Bearer {ACCESS_TOKEN}
```

### Registrar número

```bash
curl -X POST "https://graph.facebook.com/v22.0/{PHONE_NUMBER_ID}/register" \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "pin": "SEU_PIN_6_DIGITOS"
  }'
```

### Desregistrar número

```bash
curl -X POST "https://graph.facebook.com/v22.0/{PHONE_NUMBER_ID}/deregister" \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  -d '{"messaging_product": "whatsapp"}'
```

---

## Referências

- [Cloud API Overview](https://developers.facebook.com/docs/whatsapp/cloud-api)
- [Get Started](https://developers.facebook.com/docs/whatsapp/cloud-api/get-started)
- [Send Messages Reference](https://developers.facebook.com/documentation/business-messaging/whatsapp/messages/send-messages)
- [Webhooks Reference](https://developers.facebook.com/documentation/business-messaging/whatsapp/webhooks/reference/messages/)
- [Error Codes](https://developers.facebook.com/documentation/business-messaging/whatsapp/support/error-codes)
- [Media Reference](https://developers.facebook.com/documentation/business-messaging/whatsapp/business-phone-numbers/media/)
- [Phone Numbers Reference](https://developers.facebook.com/documentation/business-messaging/whatsapp/business-phone-numbers/phone-numbers)
