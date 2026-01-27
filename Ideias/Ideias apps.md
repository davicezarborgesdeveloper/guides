# 🧭 Trilha Flutter Completa — do Zero ao Portfólio Profissional

> Objetivo: criar uma progressão clara de estudos em Flutter, saindo de apps simples de prática até apps de portfólio que impressionam recrutadores.
>
> Observação: **nenhum app envolve finanças pessoais**.

---

## 🔹 FASE 1 — Apps Iniciais (Prática e Fundamentos)

**Objetivo:** ganhar fluidez, segurança e repertório técnico.

Esses apps são de prática. Não precisam estar todos públicos no GitHub.

### 📱 Lista de Apps

### App 1 — To-Do App Simples

**Objetivo**: aprender estrutura básica, listas e estado.

**Funcionalidades**

- Criar tarefa
- Marcar como concluída
- Remover tarefa
- Listar pendentes e concluídas

**Modelo**

```dart
class Task {
  final String id;
  final String title;
  final bool isDone;
}
```

**Tecnologias**

- Flutter
- setState ou Provider

---

### App 2 — App de Notas Básico

**Objetivo**: CRUD + persistência local.

**Funcionalidades**

- Criar / editar / excluir notas
- Listagem de notas
- Persistência local

**Modelo**

```dart
class Note {
  final String id;
  final String title;
  final String content;
  final DateTime createdAt;
  final DateTime updatedAt;
}
```

**Persistência**

- Hive (recomendado)
- SharedPreferences (aceitável)

---

### App 3 — Contador de Hábitos Simples

**Objetivo**: lógica de domínio básica.

**Funcionalidades**

- Criar hábito
- Marcar como concluído no dia
- Reset diário

**Modelo**

```dart
class Habit {
  final String id;
  final String title;
  final bool isCompletedToday;
  final DateTime lastCompletedDate;
}
```

---

### App 4 — Lista de Compras

**Objetivo**: listas com múltiplos estados.

**Funcionalidades**

- Criar item
- Editar item
- Marcar como comprado
- Remover item

**Modelo**

```dart
class ShoppingItem {
  final String id;
  final String name;
  final int quantity;
  final bool isBought;
}
```

---

### App 5 — App de Clima (Simples)

**Objetivo**: consumo de API e estados assíncronos.

**Funcionalidades**

- Buscar clima por cidade
- Exibir temperatura e condição
- Loading e erro

**Modelo**

```dart
class Weather {
  final String city;
  final double temperature;
  final String condition;
  final double feelsLike;
}
```

---

### App 6 — App de Frases / Quotes

**Objetivo**: fluidez em UI e estado leve.

**Funcionalidades**

- Exibir frase
- Atualizar frase
- Favoritar (opcional)

---

### App 7 — Cronômetro / Timer

**Objetivo**: controle de tempo e ciclo de vida.

**Funcionalidades**

- Iniciar / pausar / resetar
- Exibir tempo formatado

---

### App 8 — Agenda Simples

**Objetivo**: trabalhar com datas e horários.

**Funcionalidades**

- Criar evento
- Definir data e hora
- Listar eventos do dia

**Modelo**

```dart
class Event {
  final String id;
  final String title;
  final DateTime dateTime;
}
```

---

### App 9 — Checklist de Viagem

**Objetivo**: listas categorizadas e UX prática.

**Funcionalidades**

- Criar checklist
- Marcar itens
- Editar / remover

---

### App 10 — App de Lembretes

**Objetivo**: consolidar datas + notificações.

**Funcionalidades**

- Criar lembrete
- Definir data e hora
- Notificação local

**Modelo**

```dart
class Reminder {
  final String id;
  final String title;
  final DateTime dateTime;
}
```

---

## 🔹 FASE 2 — Apps Estratégicos (Portfólio Real)

Esses apps **devem estar públicos**, bem documentados e polidos.

---

### App 11 — Notes Markdown App

**Foco**: arquitetura + UX.

**Funcionalidades**

- Editor Markdown
- Preview em tempo real
- Busca
- Favoritos
- Dark Mode

**Arquitetura**

- Clean Architecture ou MVVM
- Feature-first

---

### App 12 — Habit Tracker Gamificado

**Foco**: lógica de domínio + produto.

**Funcionalidades**

- Streak
- Níveis
- Badges
- Histórico
- Gráficos

---

### App 13 — App de Filmes & Séries

**Foco**: APIs e performance.

**Funcionalidades**

- Listagem via API
- Busca
- Paginação
- Favoritos
- Cache local

---

### App 14 — Planner Diário Offline-First

**Foco**: engenharia e confiabilidade.

**Funcionalidades**

- Planejamento diário
- Tarefas recorrentes
- Notificações
- Histórico

**Persistência**

- SQLite (Drift/Floor)

---

### App 15 — Pokédex / Enciclopédia Offline

**Foco**: showcase técnico e visual.

**Funcionalidades**

- Catálogo grande
- Busca e filtros
- Offline total após sync
- Hero animations
- Design System próprio

---

## 🧠 Regra de Ouro do Portfólio

> **10 apps te ensinam Flutter.**
> **5 apps te dão emprego.**

---

## 📌 Como estudar este material

- Estude **fase 1** para ganhar fluidez
- Construa **fase 2** com calma e capricho
- Sempre finalize um app antes de iniciar outro
- Documente decisões técnicas

---

## 🚀 Próximos usos deste material

- Base de estudo diário
- Planejamento de cronograma (60–90 dias)
- Criação de READMEs profissionais
- Preparação para entrevistas Flutter

---

Fim da trilha.
