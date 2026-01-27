# Mason no Flutter – Guia Completo, Exaustivo e Didático

> Este documento consolida **todo o conteúdo explicado na conversa** sobre Mason no ecossistema Flutter/Dart, organizado para estudo posterior, do nível iniciante absoluto até uso avançado em times.

---

## 1. O que é Mason?

O **Mason** é uma ferramenta de **geração de código baseada em templates**. Ele permite criar estruturas padronizadas de arquivos e pastas (features, widgets, camadas, testes, etc.) a partir de **bricks** (templates reutilizáveis).

Ele resolve problemas comuns como:

- Copiar e colar código repetitivo
- Inconsistência de nomes e estrutura
- Esquecimento de arquivos obrigatórios
- Dificuldade de padronização em times

Com Mason, você **define um padrão uma única vez** e passa a gerá-lo automaticamente.

---

## 2. Componentes do Ecossistema Mason

### 2.1 Mason (`mason`)

- Biblioteca base (engine)
- Responsável por interpretar templates, variáveis e gerar arquivos
- Normalmente usada internamente pelo CLI

### 2.2 Mason CLI (`mason_cli`)

- Ferramenta de linha de comando
- É o que você usa no dia a dia
- Comandos principais:
  - `mason init`
  - `mason add`
  - `mason get`
  - `mason make`
  - `mason new`
  - `mason bundle`
  - `mason publish`

### 2.3 BrickHub

- Repositório público de bricks
- Permite publicar e consumir templates prontos

---

## 3. Instalação (Baby Steps)

### 3.1 Pré-requisitos

- Dart ou Flutter instalado

### 3.2 Instalar o Mason CLI

```bash
dart pub global activate mason_cli
```

Ou via Homebrew:

```bash
brew tap felangel/mason
brew install mason
```

### 3.3 Verificar instalação

```bash
mason --version
```

---

## 4. Inicializando Mason em um Projeto Flutter

No root do projeto:

```bash
mason init
```

Isso cria o arquivo:

```yaml
mason.yaml
```

Esse arquivo registra todos os bricks usados no projeto.

---

## 5. O Conceito de Brick

Um **brick** é um template reutilizável.

Estrutura básica:

```
my_brick/
 ├─ brick.yaml
 └─ __brick__/
```

- `brick.yaml`: definição do brick
- `__brick__/`: arquivos e pastas que serão gerados

Criar um brick novo:

```bash
mason new my_brick
```

Com hooks:

```bash
mason new my_brick --hooks
```

---

## 6. Estrutura do `brick.yaml`

Exemplo:

```yaml
name: widget_template
description: Template de Widget Flutter
vars:
  widget_name:
    type: string
    description: Nome do widget
    default: MyWidget
    prompt: Qual o nome do widget?
```

Cada variável:

- Aparece como prompt
- Pode ter valor padrão
- Pode ser usada nos templates

---

## 7. Templates com Mustache

O Mason usa **Mustache syntax**.

### 7.1 Variáveis no código

```dart
class {{widget_name.pascalCase()}} extends StatelessWidget {
```

### 7.2 Variáveis no nome do arquivo

```
{{widget_name.snakeCase()}}.dart
```

### 7.3 Casings mais comuns

- `snakeCase()` → `my_widget`
- `pascalCase()` → `MyWidget`
- `camelCase()` → `myWidget`

---

## 8. Condicionais (Geração Dinâmica)

Você pode ligar/desligar partes do código.

### Exemplo no `brick.yaml`

```yaml
is_stateful:
  type: boolean
  default: false
```

### Exemplo no template

```mustache
{{#is_stateful}}
class {{widget_name.pascalCase()}} extends StatefulWidget {
{{/is_stateful}}

{{^is_stateful}}
class {{widget_name.pascalCase()}} extends StatelessWidget {
{{/is_stateful}}
```

---

## 9. Hooks (Automação Avançada)

Hooks são scripts Dart executados:

- Antes da geração (`pre_gen.dart`)
- Depois da geração (`post_gen.dart`)

Estrutura:

```
hooks/
 ├─ pre_gen.dart
 └─ post_gen.dart
```

### Casos de uso comuns

- Validar input
- Criar pastas adicionais
- Rodar `dart format`
- Rodar `dart fix --apply`

---

## 10. Registrando Bricks no Projeto

No `mason.yaml` do root:

```yaml
bricks:
  widget_template:
    path: bricks/widget_template
```

Depois:

```bash
mason get
```

---

## 11. Gerando Código

```bash
mason make widget_template
```

Com diretório de saída:

```bash
mason make widget_template -o lib/widgets
```

---

## 12. Organização Recomendada em Times

```
/bricks
 ├─ feature_clean
 ├─ widget
 └─ repository

mason.yaml
```

Benefícios:

- Padronização
- Versionamento
- Onboarding rápido

---

## 13. Bricks Locais vs Globais

### Local

- Versionado com o projeto
- Ideal para times

### Global

- Templates pessoais
- Reutilização entre projetos

```bash
mason add -g meu_brick --path ./meu_brick
```

---

## 14. Bundle e Unbundle

### Bundle

```bash
mason bundle ./brick -o ./bundle
```

### Unbundle

```bash
mason unbundle ./bundle -o ./brick
```

Usado para:

- Distribuição
- CLIs customizados

---

## 15. Publicar Bricks (BrickHub)

```bash
mason login
mason publish
```

⚠️ Não é possível remover um brick publicado.

---

## 16. Roteiro Prático – Do Zero

1. `mason init`
2. `mason new widget_template --hooks`
3. Definir variáveis no `brick.yaml`
4. Criar templates em `__brick__/`
5. Registrar no `mason.yaml`
6. `mason get`
7. `mason make widget_template`

---

## 17. Erros Comuns

- Brick não aparece → esqueceu `mason get`
- Nome errado → usar casing correto
- Código não formatado → usar hooks

---

## 18. Próximo Nível

- Gerar **features completas** (UI + State + Domain + Data)
- Integrar com Clean Architecture
- Criar bricks específicos por contexto de negócio
- Automatizar testes e mocks

---

📌 **Este material pode ser usado como referência contínua.**
Se quiser, posso evoluir este guia com exemplos de **Clean Architecture**, **Bloc**, **Riverpod**, **MVVM**, ou criar **bricks prontos para seu time**.
