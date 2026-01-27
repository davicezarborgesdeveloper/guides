# Guia Completo — Mason + Flutter

Este documento consolida **tudo o que foi construído até agora** sobre o uso do **Mason no Flutter**, desde o básico até um **setup profissional com arquitetura, CI e Git hooks**. A ideia é servir como **material de estudo e referência**.

---

## 1. O que é Mason

**Mason** é uma ferramenta CLI em Dart usada para **gerar código a partir de templates reutilizáveis (bricks)**.

Ele resolve problemas clássicos de projetos Flutter:

- Copy/paste excessivo
- Inconsistência de arquitetura
- Nomes errados de arquivos e classes
- Features criadas de formas diferentes por cada dev

Com Mason:

- Código nasce padronizado
- Arquitetura é imposta
- Times escalam sem virar caos

---

## 2. Conceitos Fundamentais

Conceito

Descrição

mason_cli

CLI que executa os comandos

brick

Template reutilizável

brick.yaml

Configuração do brick

mason.yaml

Registro de bricks no projeto

**brick**

Pasta com os arquivos template

Mustache

Sintaxe de variáveis ({{ }})

Transformações comuns:

- camelCase
- pascalCase
- snakeCase
- kebabCase

---

## 3. Setup Inicial

### Instalação

```bash
dart pub global activate mason_cli

```

### Inicializar Mason no projeto Flutter

```bash
mason init

```

Cria o arquivo `mason.yaml`.

---

## 4. Brick de Feature MVVM (base)

### Objetivo

Gerar automaticamente:

```
lib/features/login/
├── view/
│   └── login_page.dart
├── viewmodel/
│   └── login_viewmodel.dart
├── model/
│   └── login_state.dart
└── login_module.dart

```

### brick.yaml

```yaml
name: feature_mvvm
description: Cria uma feature Flutter usando MVVM

vars:
  feature_name:
    type: string
    prompt: Feature name?
```

### Responsabilidades

- View: UI pura
- ViewModel: lógica + estado
- Model: estado imutável
- Module: ponto de entrada da feature

---

## 5. Arquitetura Real de Projeto (Clean Architecture pragmática)

### Estrutura global

```
lib/
├── app/
├── core/
├── features/
└── main.dart

```

### Regra de ouro

> Nenhuma feature conhece outra feature.

Código compartilhado vai para `core/`.

---

## 6. Estrutura de uma Feature (Clean)

```
features/login/
├── data/
│   ├── datasources/
│   ├── models/
│   └── repositories/
│
├── domain/
│   ├── entities/
│   ├── repositories/
│   └── usecases/
│
└── presentation/
    ├── pages/
    ├── viewmodel/
    └── widgets/

```

### Domain

- Não conhece Flutter
- Não conhece HTTP
- Apenas regras de negócio

### Data

- Implementa repositories
- Converte DTO → Entity
- Conhece API, JSON, Firebase etc.

### Presentation

- UI
- ViewModel
- Consome UseCases

---

## 7. Dependency Injection (manual e simples)

`app/di.dart`

- Monta datasources
- Monta repositories
- Monta usecases
- Entrega ViewModels prontos

Benefício:

- Fácil de testar
- Fácil de trocar implementação

---

## 8. Mason como Regra do Projeto

### Convenção de time

```md
Toda feature deve ser criada via Mason.
Criar feature manualmente é proibido.
```

Isso garante:

- Estrutura consistente
- Menos bugs
- Menos review de organização

---

## 9. Hooks do Mason

### Pre-gen (validação)

Usado para:

- Validar nome da feature
- Impedir snake_case inválido
- Bloquear geração errada

### Post-gen

Usado para:

- Rodar dart format
- Ajustar arquivos
- Executar scripts

Hooks = primeira linha de defesa.

---

## 10. Git Hooks

### pre-commit

Executado antes do commit:

- flutter analyze
- flutter test

Benefício:

- Código quebrado não entra no repo
- Erros são resolvidos localmente

---

## 11. CI com GitHub Actions

Pipeline básico:

- checkout
- flutter pub get
- mason get
- flutter analyze
- flutter test

CI garante:

- Qualidade contínua
- Padrão validado no servidor
- PR só entra se estiver correto

---

## 12. Proteção de Arquitetura no CI

Scripts simples podem:

- Bloquear imports errados
- Impedir camada presentation acessando data

Isso transforma arquitetura em **regra técnica**, não opinião.

---

## 13. Papel de Cada Camada de Controle

Camada

Função

Mason

Gera estrutura correta

Hooks

Validação local

Git Hooks

Bloqueio antes do commit

CI

Qualidade contínua

Review

Regra de negócio

---

## 14. Resultado Final

Você passa a ter:

- Arquitetura previsível
- Código consistente
- Onboarding rápido
- Escalabilidade real
- Menos bugs estruturais

Esse setup é usado em **times grandes e projetos de longa duração**.

---

## 15. Próximos Estudos (opcional)

- Bricks com testes automáticos
- Bricks com GoRouter
- Versionamento de bricks em repo separado
- Monorepo Flutter + Mason

---

Fim do guia.

# Flutter Flavors – Guia Completo (Step by Step + Babysteps)

Este documento explica **de forma exaustiva** o que são **Flavors em Flutter**, por que usar, e como implementar **passo a passo**, cobrindo **Dart, Android e iOS**, com exemplos práticos.

---

## 1. O que são Flavors em Flutter

**Flavors** permitem criar **múltiplas variações do mesmo aplicativo**, usando **um único código-base**.

Cada flavor pode ter:

- Ambiente diferente (dev / hml / prod)
- Base URL diferente
- Nome do app diferente
- Ícone diferente
- Firebase diferente
- BundleId / ApplicationId diferente

> Pense em flavors como "personalidades" do app.

---

## 2. As duas camadas dos Flavors

### 2.1 Camada Flutter (Dart)

Controla comportamento em **runtime**:

- URLs
- Flags
- Features
- Logs

### 2.2 Camada Nativa

Controla identidade do app:

- Android: `productFlavors` (Gradle)
- iOS: `Schemes` + `Build Configurations`

---

## 3. Estrutura final esperada

```text
lib/
 ├─ env/
 │   ├─ app_env.dart
 │   └─ env.dart
 ├─ main_dev.dart
 ├─ main_prod.dart
 └─ app.dart
```

---

## 4. Implementação no Flutter (Dart)

### 4.1 Criar enum de Flavor

```dart
enum Flavor { dev, prod }
```

---

### 4.2 Criar classe de ambiente

```dart
class AppEnv {
  final Flavor flavor;
  final String appName;
  final String baseUrl;

  const AppEnv({
    required this.flavor,
    required this.appName,
    required this.baseUrl,
  });
}
```

---

### 4.3 Criar holder global

```dart
class Env {
  static late AppEnv current;
}
```

---

### 4.4 Criar entrypoint DEV

```dart
void main() {
  Env.current = const AppEnv(
    flavor: Flavor.dev,
    appName: 'MeuApp (Dev)',
    baseUrl: 'https://api-dev.exemplo.com',
  );

  runApp(const MyApp());
}
```

---

### 4.5 Criar entrypoint PROD

```dart
void main() {
  Env.current = const AppEnv(
    flavor: Flavor.prod,
    appName: 'MeuApp',
    baseUrl: 'https://api.exemplo.com',
  );

  runApp(const MyApp());
}
```

---

### 4.6 Usar no app

```dart
MaterialApp(
  title: Env.current.appName,
)
```

---

## 5. Android – Configurando productFlavors

### 5.1 build.gradle

```gradle
flavorDimensions "env"

productFlavors {
    dev {
        dimension "env"
        applicationIdSuffix ".dev"
        versionNameSuffix "-dev"
        resValue "string", "app_name", "MeuApp (Dev)"
    }

    prod {
        dimension "env"
        resValue "string", "app_name", "MeuApp"
    }
}
```

---

### 5.2 AndroidManifest.xml

```xml
<application
    android:label="@string/app_name" />
```

---

### 5.3 Rodar Android

```bash
flutter run --flavor dev -t lib/main_dev.dart
flutter run --flavor prod -t lib/main_prod.dart
```

---

## 6. iOS – Schemes e Configurations

### 6.1 Criar Configurations

- Debug-Dev
- Debug-Prod
- Release-Dev
- Release-Prod

---

### 6.2 Criar Schemes

- dev → Debug-Dev / Release-Dev
- prod → Debug-Prod / Release-Prod

---

### 6.3 Bundle Identifier por flavor

#### Debug-Dev.xcconfig

```xcconfig
#include "Debug.xcconfig"
PRODUCT_BUNDLE_IDENTIFIER=com.exemplo.meuapp.dev
```

#### Debug-Prod.xcconfig

```xcconfig
#include "Debug.xcconfig"
PRODUCT_BUNDLE_IDENTIFIER=com.exemplo.meuapp
```

---

## 7. Firebase por Flavor

### Android

```text
android/app/src/dev/google-services.json
android/app/src/prod/google-services.json
```

### iOS

```text
ios/Runner/GoogleService-Info-Dev.plist
ios/Runner/GoogleService-Info-Prod.plist
```

---

## 8. Ícones por Flavor

### Android

```text
android/app/src/dev/res/mipmap-*
android/app/src/prod/res/mipmap-*
```

### iOS

- AppIcon diferente por configuração

---

## 9. Alternativa: --dart-define

```bash
flutter run --dart-define=FLAVOR=dev --dart-define=BASE_URL=https://api-dev...
```

```dart
const flavor = String.fromEnvironment('FLAVOR');
```

---

## 10. Comandos úteis

### Build APK

```bash
flutter build apk --flavor dev -t lib/main_dev.dart
flutter build apk --flavor prod -t lib/main_prod.dart
```

### Build iOS

```bash
flutter build ios --flavor prod -t lib/main_prod.dart
```

---

## 11. Erros comuns

| Erro                       | Causa                      |
| -------------------------- | -------------------------- |
| assembleDevDebug not found | Flavor mal configurado     |
| App sobrescreve outro      | Falta applicationIdSuffix  |
| API errada                 | Esqueceu -t ou dart-define |

---

## 12. Estratégia recomendada

- **Dev local**: flavor dev
- **Homologação**: flavor hml
- **Produção**: flavor prod

Combine:

- `--flavor` → identidade nativa
- `--dart-define` → comportamento

---

## 13. Próximos passos sugeridos

- CI/CD com flavors
- Feature flags
- Injeção de dependência
- Modularização por ambiente

---

✅ **Este markdown pode ser usado como material de estudo, documentação interna ou base para onboarding de time Flutter.**
