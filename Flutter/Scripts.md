# Execução de Scripts no Flutter

Guia **didático, progressivo e aprofundado** para entender **como e por que rodar scripts em projetos Flutter**, inspirado no artigo:

> _Run scripts from pubspec with Flutter_

Este material foi feito para **estudo posterior**, indo do básico absoluto até o uso profissional.

---

## 1. O problema que os scripts resolvem

Flutter **não possui um sistema nativo de scripts**, diferente do Node.js (`npm run`).

Isso gera comandos longos e repetitivos:

```bash
flutter pub run build_runner build --delete-conflicting-outputs
flutter build apk --flavor dev -t lib/main_dev.dart

```

Com scripts, a ideia é simples:

```bash
rps gen
rps build dev

```

➡️ **Scripts são apelidos organizados para comandos de terminal.**

---

## 2. O que significa “rodar scripts” de verdade

Rodar um script significa:

- mapear um **nome curto** para um **comando longo**
- executar esse comando via CLI

Nada mágico acontece. É apenas automação e padronização.

---

## 3. Por que usar o `pubspec.yaml`

O `pubspec.yaml` já é:

- o centro do projeto Flutter
- versionado no Git
- compartilhado pelo time
- conhecido por qualquer dev Flutter

Adicionar scripts nele significa:

> _“Se isso faz parte do projeto, deve estar aqui.”_

📌 **Importante:** Flutter **não interpreta** a chave `scripts` nativamente.

---

## 4. Ferramentas que habilitam scripts

Como Flutter não tem isso nativo, usamos ferramentas externas.

### Exemplos

- `rps`
- `flutter_scripts`
- `script_runner`
- Makefile (alternativa clássica)

Neste guia, usamos o **rps** por ser simples e direto.

---

## 5. Instalando o rps (primeiro passo real)

```bash
dart pub global activate rps

```

Verifique se funciona:

```bash
rps --help

```

---

## 6. Estrutura básica de scripts

Dentro do `pubspec.yaml`:

```yaml
scripts:
  gen: flutter pub run build_runner build --delete-conflicting-outputs
```

Executando:

```bash
rps gen

```

➡️ O `rps` lê o YAML e executa o comando correspondente.

---

## 7. Scripts comuns em projetos Flutter

### 7.1 Code generation

```yaml
scripts:
  gen: flutter pub run build_runner build --delete-conflicting-outputs
```

---

### 7.2 Limpeza do projeto

```yaml
scripts:
  clean: flutter clean && flutter pub get
```

---

### 7.3 Análise estática

```yaml
scripts:
  lint: flutter analyze
```

---

### 7.4 Testes

```yaml
scripts:
  test: flutter test
```

---

## 8. Scripts hierárquicos (organização profissional)

```yaml
scripts:
  build:
    dev: flutter build apk --flavor dev -t lib/main_dev.dart
    prod: flutter build apk --flavor prod -t lib/main_prod.dart
```

Execução:

```bash
rps build dev

```

➡️ Excelente para **ambientes e flavors**.

---

## 9. Scripts NÃO precisam ser Flutter

Você pode rodar **qualquer comando do sistema**:

```yaml
scripts:
  format:
    dart: dart format .
    flutter: flutter format .
```

Ou ferramentas comuns:

```yaml
scripts:
  icons: flutter pub run flutter_launcher_icons
  splash: flutter pub run flutter_native_splash:create
```

---

## 10. Scripts como documentação viva

Quando você escreve:

```bash
rps gen

```

Você está dizendo:

> _“Atualize o código gerado do projeto”_

Não:

> _“Execute esse comando gigante”_

📌 Scripts representam **intenção**, não só execução.

---

## 11. Comparação com outras soluções

Solução

Quando usar

rps

Dev local, DX, padronização

Makefile

Build complexo

Bash scripts

Infra pesada

CI/CD

Automação remota

Na prática, projetos maduros usam **mais de uma abordagem**.

---

## 12. Quando NÃO usar scripts

❌ Projetos pequenos  
❌ POCs rápidas  
❌ Estudos iniciais

Porque scripts exigem:

- padronização
- disciplina
- manutenção

---

## 13. Insight final (o mais importante)

> **Scripts não são sobre terminal.**
>
> **São sobre clareza, padronização e intenção.**

Eles melhoram:

- onboarding
- DX (Developer Experience)
- consistência do time

---

## 14. Próximos aprofundamentos possíveis

- Scripts por ambiente (dev/stage/prod)
- Integração com CI/CD
- Comparação rps vs Makefile
- Scripts em arquitetura modular Flutter
- Padronização para times grandes

---

📚 _Este documento foi pensado para estudo contínuo._
Ele cobre:

- contexto e motivação (o _porquê_ dos scripts)
- funcionamento real por baixo dos panos
- exemplos práticos e organizados
- hierarquia de scripts
- comparação com outras abordagens
- insights de uso em times e projetos grandes

Daqui pra frente, a gente pode:

- **evoluir esse mesmo documento** (ex.: adicionar CI/CD, Makefile, arquitetura modular)
- criar um **capítulo só de exemplos reais de projetos**
- transformar isso num **guia pessoal seu de Flutter tooling**
- ou gerar uma **versão enxuta “cola de referência”**

É só me dizer como você quer continuar estudando a partir dele.
