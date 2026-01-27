# Flutter App Architecture – Guia Completo para Estudo

Este documento consolida **todo o conteúdo da conversa** em um material único, progressivo e didático, para estudo posterior.

---

## 1. O que o Google define oficialmente para arquitetura Flutter

O Google **não impõe um padrão fechado** (MVC, MVVM, Clean etc).

O que existe é uma **arquitetura orientada a princípios**, descrita na documentação oficial:  
[https://docs.flutter.dev/app-architecture](https://docs.flutter.dev/app-architecture)

### Princípios obrigatórios

- **Fluxo de Dados Unidirecional (UDF)**
- **Separação de responsabilidades**
- **Single Source of Truth**
- **UI declarativa e reativa**

O resultado prático é uma arquitetura que se parece com MVVM, mas sem rigidez acadêmica.

---

## 2. MVVM no Flutter (Google-style)

### Estrutura lógica

```
UI (Widgets)
↓
State Holder (ViewModel / Notifier / Bloc)
↓
Repository
↓
Data Source

```

### Camadas

- **UI Layer**: Widgets puros, sem lógica de negócio
- **State Holder**: mantém estado e coordena ações
- **Data Layer**: acesso a dados (API, DB, cache)
- **Domain (opcional)**: regras complexas e reutilizáveis

---

## 3. Mini App Base (Listagem de Usuários)

### Estrutura

```
lib/
 ├─ features/
 │   └─ users/
 │       ├─ domain/
 │       │   └─ user.dart
 │       ├─ data/
 │       │   ├─ users_repository.dart
 │       │   └─ fake_users_repository.dart
 │       ├─ state/
 │       │   └─ users_view_model.dart
 │       └─ ui/
 │           └─ users_page.dart

```

### Domain

```dart
class User {
  final String id;
  final String name;
  final String email;
}

```

### Repository (contrato)

```dart
abstract class UsersRepository {
  Future<List<User>> fetchUsers();
}

```

### Fake Data Source

```dart
class FakeUsersRepository implements UsersRepository {
  @override
  Future<List<User>> fetchUsers() async {
    await Future.delayed(Duration(seconds: 2));
    return [...];
  }
}

```

---

## 4. Provider vs Riverpod

### Provider (ChangeNotifier)

- Estado mutável
- notifyListeners manual
- Depende de BuildContext

```dart
class UsersViewModel extends ChangeNotifier { ... }

```

### Riverpod (Notifier)

- Estado imutável
- Reatividade automática
- Sem BuildContext
- Melhor para escala

```dart
class UsersViewModel extends Notifier<UsersState> { ... }

```

### Comparação direta

Critério

Provider

Riverpod

Simplicidade

Alta

Média

Escalabilidade

Média

Alta

Testes

Médio

Excelente

Estado

Mutável

Imutável

---

## 5. MVVM Flutter vs Clean Architecture

### MVVM Flutter

```
UI → ViewModel → Repository

```

- Menos boilerplate
- Ideal para 80% dos apps
- Rápido para evoluir

### Clean Architecture

```
UI → ViewModel → UseCase → Repository → DataSource

```

- Mais camadas
- Domínio isolado
- Ideal para regras complexas

### Comparação

Critério

MVVM

Clean

Complexidade

Baixa

Alta

Velocidade

Alta

Baixa

Escala

Média

Alta

---

## 6. Modularização sem flutter_modular

### Conceito

- Modularização por **feature**
- Dependências explícitas
- Sem service locator mágico

### Estrutura

```
features/
 └─ users/
     ├─ users_module.dart
     ├─ domain/
     ├─ data/
     ├─ state/
     └─ ui/

```

### users_module.dart (Riverpod)

```dart
final usersRepositoryProvider = Provider<UsersRepository>(...);
final usersViewModelProvider = NotifierProvider<UsersViewModel, UsersState>(...);

```

- Cada feature se monta sozinha
- App apenas consome páginas

---

## 7. go_router + módulos (conceito)

- Rotas declarativas
- Cada módulo expõe sua página
- App central apenas registra rotas

---

## 8. Regra de ouro final

> Arquitetura serve o produto, não o ego do dev.

- Comece simples (MVVM)
- Extraia domínio quando doer
- Modularize por feature
- Evite magia e frameworks desnecessários

---

## 9. Caminho recomendado de estudo

1.  MVVM Google-style
2.  Provider → Riverpod
3.  Modularização manual
4.  Testes de ViewModel
5.  Introdução pontual de UseCases

---

\*\*Este documento representa uma visão prática, realista e escalável de Flutter moderno, alinhada com o Google e com o mercado

### O que você tem agora

- 📘 **Guia único**, progressivo e coerente
- 🧠 Do **Google-style MVVM** até **Clean Architecture**
- 🔀 **Provider vs Riverpod** lado a lado
- 🧱 **Modularização real sem flutter_modular**
- 🗂 Estruturas de pastas, conceitos e regras práticas
- 🧭 Caminho de estudo sugerido

Você pode:

- Ler com calma, seção por seção
- Usar como **referência pessoal**
- Evoluir esse material (posso atualizar o documento)
- Exportar depois como base de curso, wiki ou notas

Se quiser, os próximos passos naturais seriam:

- 🔹 adicionar **testes unitários** no mesmo documento
- 🔹 incluir **go_router + módulos** com exemplo
- 🔹 transformar isso num **checklist arquitetural**
- 🔹 ou adaptar para **app real (auth + users + cache)**
