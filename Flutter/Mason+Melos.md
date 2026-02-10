# Guia Baby Steps – Melos + Mason no Flutter

Este documento consolida **todo o passo a passo explicado na conversa**, organizado para estudo posterior. A ideia é ir do **zero absoluto** até uma integração funcional entre **Melos** (monorepo) e **Mason** (geração de código), de forma simples, incremental e segura.

---

## 1. Conceitos (antes de codar)

### Melos

- Ferramenta para **gerenciar monorepos Dart/Flutter**.
- Orquestra comandos (`pub get`, `test`, `analyze`, etc.) em múltiplos packages/apps.
- Hoje funciona **em cima de Pub Workspaces**.

### Mason

- Ferramenta de **code generation** baseada em _bricks_.
- Usada para gerar páginas, features, widgets, módulos, etc.
- Ajuda a padronizar código e acelerar desenvolvimento.

### Integração Melos + Mason

> Não existe integração “mágica”.

A integração acontece quando:

- O **Mason** é configurado no monorepo (geralmente na raiz)
- O **Melos executa comandos do Mason** via scripts

Melos = _orquestração_
Mason = _geração de código_

---

## 2. Pré-requisitos

### Instalar os CLIs

```bash
dart pub global activate melos
dart pub global activate mason_cli
```

> Se os comandos não funcionarem, verifique se o diretório do pub global está no seu PATH (ex: `~/.pub-cache/bin`).

---

## 3. Estrutura mínima do monorepo

Exemplo simples e funcional:

```
my_repo/
  apps/
    app_one/
  packages/
    core/
    design_system/
```

Criando rapidamente:

```bash
mkdir -p my_repo/apps my_repo/packages
cd my_repo

flutter create apps/app_one
flutter create --template=package packages/design_system
```

---

## 4. Configurar Pub Workspaces (obrigatório)

Crie o arquivo `pubspec.yaml` **na raiz do repositório**:

```yaml
name: my_repo_workspace
publish_to: "none"

environment:
  sdk: ">=3.0.0 <4.0.0"

workspace:
  - apps/*
  - packages/*
```

Isso habilita o monorepo para o Melos.

---

## 5. Configurar Melos no pubspec.yaml raiz

No **mesmo arquivo**, adicione a seção do Melos:

```yaml
melos:
  scripts:
    bootstrap:
      run: melos bootstrap
      description: "Resolve e linka dependências do workspace"

    get:
      run: melos exec -- flutter pub get
      description: "flutter pub get em todos os packages/apps"
```

Rodar:

```bash
melos bootstrap
```

---

## 6. Inicializar o Mason no monorepo

Na raiz do repositório:

```bash
mason init
```

Isso cria o arquivo:

```
mason.yaml
```

Esse arquivo será o **catálogo central de bricks** do monorepo.

---

## 7. Registrar um brick no Mason

Exemplo simples usando BrickHub (apenas para teste):

```yaml
bricks:
  hello: 0.1.0+2
```

Baixar os bricks:

```bash
mason get
```

---

## 8. Integrando Melos + Mason (parte principal)

A integração acontece via **scripts do Melos**.

No `pubspec.yaml` da raiz:

```yaml
melos:
  scripts:
    bootstrap:
      run: melos bootstrap

    mason:get:
      run: mason get
      description: "Baixa bricks do mason.yaml da raiz"

    mason:hello:app_one:
      run: melos exec --scope="app_one" -- mason make hello
      description: "Gera o brick hello dentro do app_one"
```

Executando:

```bash
melos run mason:get
melos run mason:hello:app_one
```

### O que está acontecendo aqui?

- `mason.yaml` fica **centralizado na raiz**
- `melos exec --scope="app_one"` executa o comando **dentro do app**
- O Mason gera código no contexto correto

---

## 9. Organização recomendada para times

### Estrutura de bricks

```
bricks/
  feature/
  page/
  widget/
```

No `mason.yaml`:

```yaml
bricks:
  feature:
    path: bricks/feature
  page:
    path: bricks/page
```

---

## 10. Scripts úteis no dia a dia

Exemplo de kit básico de scripts:

```yaml
melos:
  scripts:
    bootstrap: melos bootstrap
    get: melos exec -- flutter pub get

    mason:get: mason get

    gen:feature:core:
      run: melos exec --scope="core" -- mason make feature -o lib/src/features
```

Uso:

```bash
melos run gen:feature:core
```

---

## 11. Checklist de debug rápido

Se algo der errado, verifique:

- `melos bootstrap` funciona?
- `mason get` funciona na raiz?
- `melos exec --scope="app_one" -- mason --version` funciona?
- PATH do `mason` e `melos` está configurado?

---

## 12. Resumo mental (guarde isso)

- **Melos** organiza o monorepo
- **Mason** gera código
- **Integração = Melos chamando Mason**
- `mason.yaml` normalmente fica **na raiz**
- Scripts do Melos são o coração da automação

---

Se quiser, no próximo passo eu posso:

- Montar um **template de monorepo real de produção**
- Criar **bricks Mason padrão para Flutter (feature/page/widget)**
- Sugerir **convenções de nomes e pastas** para times grandes
