# Flavors em Flutter — Guia Completo (Conceito + Implementação em Baby Steps)

## 1. O que são Flavors em Flutter

Flavors são variações do mesmo aplicativo Flutter, geradas a partir de um único código-base, mas com **configurações diferentes** por ambiente ou contexto, como:

- dev / hml / prod
- cliente A / cliente B (white label)
- builds internos vs builds de loja

Em Flutter, flavors não são um recurso nativo do Dart. Eles são a **combinação de configurações nativas (Android e iOS)** + uma forma de **informar o ambiente para o código Dart**.

---

## 2. Quando usar (e quando não usar)

### Use flavors quando:

- Você precisa de **applicationId / bundleId diferentes**
- Quer instalar **dev e prod no mesmo celular**
- Usa **Firebase separado por ambiente**
- Precisa de **ícones, nomes ou comportamentos diferentes**
- Trabalha com CI/CD e múltiplos ambientes

### Evite flavors quando:

- A diferença é apenas uma URL facilmente controlável via backend
- O comportamento deveria mudar via **feature flag dinâmica**
- Não há diferença nativa entre os builds

---

## 3. O que normalmente muda por flavor

### Nativo

- ApplicationId / Bundle Identifier
- Nome do app
- Ícone
- Firebase (Analytics, Crashlytics, etc.)
- URL Schemes, entitlements

### Dart / Flutter

- Base URL de APIs
- Flags (logs, mocks, debug menu)
- Chaves de integração (com cuidado)
- Comportamentos específicos por ambiente

---

## 4. Como o Flutter “descobre” o flavor

As abordagens mais comuns:

1. `--dart-define` (recomendado)
2. Múltiplos `main_*.dart`
3. Arquivos de configuração externos

Este guia usa **`--dart-define`**, por ser simples e compatível com CI/CD.

---

## 5. Baby Steps — Implementação

---

## Baby Step 0 — Preparação

- Garanta que o app roda:

```bash
flutter run

Identifique:

Android: applicationId

iOS: Bundle Identifier

Baby Step 1 — Conceito de ambiente no Dart
Criar lib/app_env.dart
enum AppEnv { dev, hml, prod }


class EnvConfig {
  final AppEnv env;
  final String baseUrl;
  final bool enableLogs;


  const EnvConfig({
    required this.env,
    required this.baseUrl,
    required this.enableLogs,
  });
}


AppEnv _parseEnv(String raw) {
  switch (raw.toLowerCase()) {
    case 'dev':
      return AppEnv.dev;
    case 'hml':
    case 'stg':
    case 'staging':
      return AppEnv.hml;
    default:
      return AppEnv.prod;
  }
}


class Env {
  static const _flavor =
      String.fromEnvironment('FLAVOR', defaultValue: 'prod');
  static const _baseUrl =
      String.fromEnvironment('BASE_URL', defaultValue: 'https://api.seuapp.com');


  static final config = EnvConfig(
    env: _parseEnv(_flavor),
    baseUrl: _baseUrl,
    enableLogs: _flavor != 'prod',
  );
}
Usar no main.dart
void main() {
  debugPrint('FLAVOR=${Env.config.env}');
  debugPrint('BASE_URL=${Env.config.baseUrl}');
  runApp(const MyApp());
}
Rodar com defines
flutter run \
  --dart-define=FLAVOR=dev \
  --dart-define=BASE_URL=https://dev-api.seuapp.com
Baby Step 2 — Android: Product Flavors
Em android/app/build.gradle
flavorDimensions "env"


productFlavors {
    dev {
        dimension "env"
        applicationIdSuffix ".dev"
        versionNameSuffix "-dev"
        resValue "string", "app_name", "SeuApp DEV"
    }
    hml {
        dimension "env"
        applicationIdSuffix ".hml"
        versionNameSuffix "-hml"
        resValue "string", "app_name", "SeuApp HML"
    }
    prod {
        dimension "env"
        resValue "string", "app_name", "SeuApp"
    }
}
Rodar
flutter run --flavor dev \
  --dart-define=FLAVOR=dev \
  --dart-define=BASE_URL=https://dev-api.seuapp.com
Baby Step 3 — Android: Nome e ícone por flavor
Nome do app

Em strings.xml:

<string name="app_name">${app_name}</string>
Ícones por flavor

Pastas:

android/app/src/dev/res/mipmap-*
android/app/src/hml/res/mipmap-*
android/app/src/prod/res/mipmap-*
Baby Step 4 — iOS: Schemes

Abrir ios/Runner.xcworkspace

Product → Scheme → Manage Schemes

Criar:

Runner-Dev

Runner-Hml

Runner-Prod

Baby Step 5 — iOS: Build Configurations + Bundle ID
Criar .xcconfig

Exemplo Debug-dev.xcconfig:

#include "Debug.xcconfig"
PRODUCT_BUNDLE_IDENTIFIER=com.suaempresa.seuapp.dev
APP_DISPLAY_NAME=SeuApp DEV

Criar também:

Debug-hml / Debug-prod

Release-dev / Release-hml / Release-prod

Criar Build Configurations

Debug-Dev / Debug-Hml / Debug-Prod

Release-Dev / Release-Hml / Release-Prod

Mapear Scheme → Config

Runner-Dev → Debug-Dev / Release-Dev

Runner-Hml → Debug-Hml / Release-Hml

Runner-Prod → Debug-Prod / Release-Prod

Baby Step 6 — iOS: Nome e ícone
Nome

No Info.plist:

Bundle display name = $(APP_DISPLAY_NAME)
Ícones

Criar AppIcon por ambiente e definir no .xcconfig:

ASSETCATALOG_COMPILER_APPICON_NAME=AppIconDev
Baby Step 7 — Padronizar comandos
Dev
flutter run --flavor dev \
  --dart-define=FLAVOR=dev \
  --dart-define=BASE_URL=https://dev-api.seuapp.com
Hml
flutter run --flavor hml \
  --dart-define=FLAVOR=hml \
  --dart-define=BASE_URL=https://hml-api.seuapp.com
Prod
flutter run --flavor prod \
  --dart-define=FLAVOR=prod \
  --dart-define=BASE_URL=https://api.seuapp.com
Baby Step 8 — Firebase por flavor (opcional, mas comum)
Android
android/app/src/dev/google-services.json
android/app/src/hml/google-services.json
android/app/src/prod/google-services.json
iOS

GoogleService-Info-Dev.plist

GoogleService-Info-Hml.plist

GoogleService-Info-Prod.plist
Associar por Build Configuration.

6. Boas práticas

Flavor ≠ segurança

Não confundir buildType (debug/release) com ambiente

Padronizar scripts

Usar flavors para identidade do app

Usar feature flags para comportamento dinâmico

7. Checklist final

 App dev e prod instalam juntos

 BundleId/ApplicationId diferentes

 URLs corretas por ambiente

 Firebase separado

 CI consegue buildar cada flavor

8. Modelo mental

Flavor → identidade do app (nome, ícone, bundle, Firebase)

dart-define → comportamento do app (URLs, flags, logs)
```
