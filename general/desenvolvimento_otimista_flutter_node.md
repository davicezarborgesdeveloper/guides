# Desenvolvimento Otimista (Flutter + Node.js) — Resumo Completo

Este material resume e organiza tudo o que conversamos sobre **desenvolvimento otimista** aplicado a **Flutter (frontend)** e **Node.js (backend)**, com exemplos práticos e fluxo completo.

---

## 1) O que é “desenvolvimento otimista” em software?

Em software, “desenvolvimento otimista” é uma abordagem onde o sistema:

- Assume que **conflitos e falhas são raros**
- Evita travar/segurar recursos “por precaução”
- Mantém o fluxo rápido e fluido
- **Detecta e resolve problemas depois**, quando realmente acontecem

É o oposto da abordagem pessimista, que assume conflito como regra e tenta evitar tudo com travas.

---

## 2) Onde o “otimismo” aparece na prática?

Existem 2 formas muito comuns e importantes:

### 2.1) Optimistic UI (no app / frontend)
- A interface **se atualiza imediatamente**
- A requisição para o servidor ocorre em paralelo
- Se der erro, a UI desfaz (rollback)

### 2.2) Optimistic Locking (no backend / banco)
- O backend garante que ninguém sobrescreva dados de outra pessoa
- Usa `version` (ou `updatedAt`) para detectar conflito
- Se detectar, retorna erro **409 Conflict**

---

## 3) Flutter: Optimistic UI (o mais usado)

### 🎯 Ideia
Quando o usuário executa uma ação (curtir, confirmar presença, favoritar, aprovar):

1. Você atualiza a UI **na hora**
2. Dispara a requisição pro backend
3. Se falhar, faz rollback e avisa o usuário

---

### 3.1) Exemplo: “Curtir” um evento (optimistic update)

#### Modelo simples
```dart
class Event {
  final String id;
  final String title;
  bool liked;
  int likesCount;

  Event({
    required this.id,
    required this.title,
    required this.liked,
    required this.likesCount,
  });
}
```

#### Função com optimistic UI
```dart
Future<void> toggleLike(Event event) async {
  final oldLiked = event.liked;
  final oldLikesCount = event.likesCount;

  // 1) Atualiza UI imediatamente
  event.liked = !event.liked;
  event.likesCount += event.liked ? 1 : -1;

  notifyListeners(); // ou setState / Riverpod etc.

  try {
    // 2) Chama backend
    await api.toggleLike(event.id);
  } catch (e) {
    // 3) Rollback se falhar
    event.liked = oldLiked;
    event.likesCount = oldLikesCount;

    notifyListeners();

    // feedback pro usuário
    showToast("Falha ao curtir. Tente novamente.");
  }
}
```

---

### 3.2) Exemplo: “Confirmar presença” (mesmo padrão)
- Usuário toca “Confirmar presença”
- UI muda imediatamente
- Backend confirma
- Se falhar, volta o estado

Esse padrão é comum em:
- Instagram/Twitter (curtir)
- WhatsApp (mensagem aparece “enviando”)
- iFood/Uber (status muda e depois confirma)

---

## 4) Flutter: fila local + retry (otimismo avançado)

Esse é o “nível acima”.

### 🎯 Ideia
Mesmo com internet ruim ou offline:

- O usuário faz a ação
- O app registra a ação localmente (Hive/SQLite)
- O app tenta sincronizar automaticamente depois

### Benefícios
- O app parece “indestrutível”
- Usuário não perde ações
- Experiência muito mais premium

### Onde isso é útil
- Confirmar presença
- Favoritar
- Curtir
- Reportar evento
- Enviar feedback
- Alterações pequenas e repetíveis

---

## 5) Node.js Backend: Optimistic Locking (segurança real)

Optimistic UI é “experiência”.  
Quem garante consistência de verdade é o backend.

### Problema real
Dois curadores editando o mesmo evento ao mesmo tempo:

- Curador A muda o endereço
- Curador B muda o horário
- Se salvar sem controle: um sobrescreve o outro

---

### Solução: campo `version`

Exemplo no banco:
```sql
ALTER TABLE events ADD COLUMN version INT NOT NULL DEFAULT 1;
```

---

## 6) Fluxo correto (Flutter ↔ Backend)

### 6.1) O_attach: o Flutter faz GET
O backend retorna o evento com `version`:

```json
{
  "id": "evt_123",
  "title": "Culto Jovem",
  "date": "2026-02-10",
  "version": 7
}
```

### 6.2) O Flutter envia update com a versão
```json
{
  "title": "Culto Jovem - Especial",
  "date": "2026-02-10",
  "version": 7
}
```

### 6.3) O backend só salva se `version` ainda for 7
Se alguém salvou antes e virou `version=8`, o update falha.

---

## 7) Node.js com SQL (Postgres): exemplo prático

Exemplo usando `node-postgres`:

```js
async function updateEvent(req, res) {
  const { id } = req.params;
  const { title, date, version } = req.body;

  const result = await db.query(
    `
    UPDATE events
    SET title = $1,
        date = $2,
        version = version + 1,
        updated_at = NOW()
    WHERE id = $3 AND version = $4
    RETURNING *
    `,
    [title, date, id, version]
  );

  if (result.rowCount === 0) {
    return res.status(409).json({
      message: "Conflito: esse evento foi alterado por outra pessoa.",
    });
  }

  return res.json(result.rows[0]);
}
```

### Por que usar status 409?
Porque **409 Conflict** é o status HTTP padrão para indicar:

> “Sua atualização entrou em conflito com uma atualização feita por outra pessoa.”

---

## 8) Node.js com Prisma: exemplo comum

### Model (Prisma)
```prisma
model Event {
  id        String   @id @default(uuid())
  title     String
  date      DateTime
  version   Int      @default(1)
  updatedAt DateTime @updatedAt
}
```

### Update com versão
```js
const updated = await prisma.event.updateMany({
  where: {
    id,
    version, // só atualiza se versão bater
  },
  data: {
    title,
    date,
    version: { increment: 1 },
  },
});

if (updated.count === 0) {
  return res.status(409).json({
    message: "Conflito: evento já foi alterado por outra pessoa.",
  });
}
```

> Observação: usamos `updateMany` porque ele permite detectar “0 atualizações” de forma limpa.

---

## 9) O que o Flutter faz ao receber 409?

Quando o backend retorna 409, o Flutter tem 3 caminhos:

### Opção 1 (mais simples)
- Mostrar mensagem:  
  **“Esse evento foi atualizado. Recarregue e tente novamente.”**

### Opção 2 (boa)
- Recarrega automaticamente
- Mostra o que mudou
- Oferece botão “Aplicar minhas alterações novamente”

### Opção 3 (top, mas complexa)
- Merge automático campo a campo
- Recomendado apenas se o app tiver necessidade real

---

## 10) Onde aplicar isso no seu app de eventos gospel

Você disse que terá:

- agregador de eventos gospel
- curadoria humana
- scraping automatizado

### No Flutter (Optimistic UI)
Use para:
- curtir evento
- favoritar
- confirmar presença
- aprovar/rejeitar evento
- reportar erro em evento
- pequenas alterações

### No Backend (Optimistic Locking)
Use para:
- edição manual do evento (curadoria)
- mudança de status (aprovado/rejeitado)
- atribuição de responsável por revisão
- correções feitas por moderadores

---

## 11) Benefícios e riscos

### Desenvolvimento otimista (vantagens)
- UI rápida e fluida
- Escala melhor (menos locks)
- Menos gargalo no banco
- Melhor experiência do usuário

### Desenvolvimento otimista (riscos)
- Você precisa lidar com conflitos
- Você precisa tratar rollback
- Precisa UX para conflitos (ex.: mensagem clara)

---

## 12) Resumo final (pra fixar)

### Flutter → Optimistic UI
- Atualiza UI primeiro
- Backend confirma depois
- Se falhar, rollback

### Backend → Optimistic Locking
- Usa `version` ou `updatedAt`
- Salva somente se versão bater
- Se não bater → 409 Conflict

---

# Checklist prático (para implementar)

## Flutter
- [ ] Guardar o estado antigo antes de mudar
- [ ] Atualizar UI imediatamente
- [ ] Chamar backend
- [ ] Se falhar → rollback + mensagem
- [ ] (Opcional) fila offline + retry

## Node.js / Backend
- [ ] Adicionar coluna `version`
- [ ] Retornar `version` no GET
- [ ] Exigir `version` no PUT/PATCH
- [ ] Atualizar com `WHERE version = X`
- [ ] Se 0 rows → retornar 409

---

Se você quiser, eu posso gerar também:
- Um template de endpoints REST (Express)
- Um exemplo completo com NestJS
- Um exemplo Flutter com Riverpod (StateNotifier) ou Bloc
