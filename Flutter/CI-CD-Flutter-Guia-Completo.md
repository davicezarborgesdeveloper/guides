📘 Guia Completo de CI/CD para Flutter (GitHub e GitLab)
1️⃣ O que é CI/CD?
🔹 O que é CI (Continuous Integration)?

Integração Contínua é a prática de integrar código frequentemente ao repositório principal, validando automaticamente cada alteração.

No contexto Flutter, CI normalmente executa:

Checkout do código

Instalação do Flutter SDK

Cache de dependências

flutter pub get

Verificação de formatação (dart format)

Análise estática (flutter analyze)

Testes unitários (flutter test)

(Opcional) Coverage

(Opcional) Build de validação

🎯 Objetivo da CI

Detectar erros cedo

Impedir que código quebrado chegue na main

Padronizar ambiente

Automatizar validações

🔹 O que é CD?

CD pode significar:

Continuous Delivery (Entrega Contínua)

O build é gerado automaticamente

Está pronto para publicar

Publicação pode ser manual

Continuous Deployment (Implantação Contínua)

Build é gerado

Publicado automaticamente (Play Store, TestFlight, etc.)

📱 Por que CI/CD é importante no Flutter?

Flutter depende de:

Versão específica do Flutter/Dart

JDK correta

Gradle

CocoaPods

Xcode (iOS)

Sem CI/CD:

Builds podem variar por ambiente

"Na minha máquina funciona" vira rotina

Processo de release vira manual e arriscado

2️⃣ Conceitos Fundamentais
🔹 Triggers (Quando o pipeline roda?)

Push

Pull Request / Merge Request

Tags (v1.0.0)

Schedule

Manual trigger

Estratégia recomendada:
Evento O que roda
PR/MR Lint + Test
Main Lint + Test + Build
Tag Build + Deploy
🔹 Stages, Jobs e Pipeline

Stage → Fase (ex: test, build, deploy)

Job → Tarefa executável

Pipeline → Conjunto organizado de stages

Exemplo:

stages:

- quality
- test
- build
- deploy
  🔹 Cache vs Artifacts
  Cache

Acelera builds reutilizando dependências.

Exemplos:

.pub-cache

.gradle

Pods

Artifacts

Arquivos gerados no build.

Exemplos:

.apk

.aab

.ipa

Relatórios de teste

🔹 Secrets

Nunca commitar:

Keystore Android

Senhas

Certificados iOS

Chaves API

Guardar em:

GitHub Secrets

GitLab CI Variables

3️⃣ Pipeline Mínimo Recomendado para Flutter

Checklist ideal:

Fixar versão do Flutter

flutter pub get

dart format --set-exit-if-changed .

flutter analyze

flutter test

Build Android

(Opcional) Build iOS

4️⃣ Implementação Step by Step — GitHub Actions
📌 Baby Step 1 — CI básico

Criar:

.github/workflows/ci.yml
name: CI

on:
pull_request:
push:
branches: [ "main" ]

jobs:
flutter-ci:
runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4


      - uses: actions/setup-java@v4
        with:
          distribution: "temurin"
          java-version: "17"


      - uses: subosito/flutter-action@v2
        with:
          flutter-version: "3.22.3"
          cache: true


      - run: flutter pub get
      - run: dart format --set-exit-if-changed .
      - run: flutter analyze
      - run: flutter test

📌 Baby Step 2 — Build Android Debug
build-android:
runs-on: ubuntu-latest
needs: flutter-ci

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: "temurin"
          java-version: "17"
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: "3.22.3"
          cache: true


      - run: flutter pub get
      - run: flutter build apk --debug


      - uses: actions/upload-artifact@v4
        with:
          name: app-debug-apk
          path: build/app/outputs/flutter-apk/app-debug.apk

📌 Baby Step 3 — Build Android Release Assinado
Criar Secrets:

ANDROID_KEYSTORE_BASE64

ANDROID_KEYSTORE_PASSWORD

ANDROID_KEY_PASSWORD

ANDROID_KEY_ALIAS

Pipeline: - name: Decode keystore
run: |
echo "${{ secrets.ANDROID_KEYSTORE_BASE64 }}" | base64 --decode > android/app/release.jks

      - name: Create key.properties
        run: |
          cat > android/key.properties <<EOF
          storePassword=${{ secrets.ANDROID_KEYSTORE_PASSWORD }}
          keyPassword=${{ secrets.ANDROID_KEY_PASSWORD }}
          keyAlias=${{ secrets.ANDROID_KEY_ALIAS }}
          storeFile=app/release.jks
          EOF


      - run: flutter build appbundle --release

📌 Baby Step 4 — Build iOS
build-ios:
runs-on: macos-latest
needs: flutter-ci

    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: "3.22.3"


      - run: flutter pub get
      - run: flutter build ios --release --no-codesign

5️⃣ Implementação Step by Step — GitLab CI

Criar:

.gitlab-ci.yml
📌 Baby Step 1 — CI básico
stages:

- quality
- test

variables:
FLUTTER_VERSION: "3.22.3"

quality:
stage: quality
image: ghcr.io/cirruslabs/flutter:${FLUTTER_VERSION}
script: - flutter pub get - dart format --set-exit-if-changed . - flutter analyze

test:
stage: test
image: ghcr.io/cirruslabs/flutter:${FLUTTER_VERSION}
  script:
    - flutter pub get
    - flutter test
📌 Baby Step 2 — Build Android Debug
build_android_debug:
  stage: build
  image: ghcr.io/cirruslabs/flutter:${FLUTTER_VERSION}
script: - flutter pub get - flutter build apk --debug
artifacts:
paths: - build/app/outputs/flutter-apk/app-debug.apk
📌 Baby Step 3 — Build Android Release Assinado

Criar variáveis no GitLab:

ANDROID_KEYSTORE_BASE64

ANDROID_KEYSTORE_PASSWORD

ANDROID_KEY_PASSWORD

ANDROID_KEY_ALIAS

build_android_release:
stage: build
image: ghcr.io/cirruslabs/flutter:${FLUTTER_VERSION}
  script:
    - echo "$ANDROID_KEYSTORE_BASE64" | base64 -d > android/app/release.jks - |
cat > android/key.properties <<EOF
storePassword=$ANDROID_KEYSTORE_PASSWORD
      keyPassword=$ANDROID_KEY_PASSWORD
keyAlias=$ANDROID_KEY_ALIAS
storeFile=app/release.jks
EOF - flutter pub get - flutter build appbundle --release
artifacts:
paths: - build/app/outputs/bundle/release/app-release.aab
6️⃣ Versionamento

Flutter usa:

version: 1.4.2+103

1.4.2 → versionName

103 → versionCode

Estratégias:

Versionamento manual

Versionamento por tag

Build number automático via CI

7️⃣ Assinatura Mobile
🔹 Android

Necessário:

.jks

Alias

Senhas

Recomendação:

Armazenar keystore em Base64 nos secrets

🔹 iOS

Necessário:

Certificado

Provisioning profile

Apple Developer

Recomendação:

Usar Fastlane Match

8️⃣ Estratégia Profissional Recomendada

Ordem ideal de evolução:

CI básico

Bloquear merge se CI falhar

Build debug automático

Build release assinado

Deploy interno (Firebase / TestFlight)

Deploy produção por tag + aprovação manual

9️⃣ Boas Práticas

Fixar versão do Flutter

Separar pipeline de PR e pipeline de Release

Usar artifacts

Nunca commitar secrets

Usar Fastlane para deploy

Criar pipeline por tags

🔟 Estrutura Final Ideal
CI (PR)
├── format
├── analyze
└── test

Build (main)
├── build android debug
├── build android release
└── build ios

Deploy (tag v\*)
├── publish android
└── publish ios
🚀 Conclusão

CI/CD em Flutter:

Garante qualidade

Automatiza builds

Padroniza ambiente

Remove dependência de máquina local

Profissionaliza releases
