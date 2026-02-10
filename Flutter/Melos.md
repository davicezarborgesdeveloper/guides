# Melos — Guia Completo para Estudo

## O que é o Melos?

**Melos** é uma ferramenta de linha de comando (CLI) voltada para projetos **Dart e Flutter** que utilizam a abordagem de **monorepo**, ou seja, múltiplos pacotes dentro de um único repositório.

Ele foi inspirado no **Lerna** (JavaScript) e é mantido pela equipe da **Invertase**.

### Principais objetivos:

- Gerenciar múltiplos pacotes de forma centralizada
- Automatizar tarefas repetitivas
- Facilitar versionamento e publicação
- Melhorar organização e produtividade em projetos grandes

---

## Quando usar Melos?

Use Melos quando:

- Seu repositório possui **mais de um package ou app**
- Você compartilha código entre projetos
- Precisa rodar testes, análises ou builds em vários pacotes
- Deseja automatizar versionamento e releases
- Trabalha em equipe com arquitetura modular

---

## Instalação

### Instalação global

```bash
dart pub global activate melos
```

### Como dependência de desenvolvimento

```bash
dart pub add melos --dev
```

---

## Estrutura de um Monorepo com Melos

Exemplo de estrutura:

```
my_project/
├── pubspec.yaml
├── apps/
│   └── app_flutter/
├── packages/
│   ├── shared_ui/
│   ├── utils/
│   └── api_client/
```

---

## Configuração do pubspec.yaml (raiz)

```yaml
name: my_project
publish_to: none

environment:
  sdk: ^3.9.0

workspace:
  - packages/shared_ui
  - packages/utils
  - packages/api_client

dev_dependencies:
  melos: ^7.0.0
```

---

## Configuração dos pacotes

Em cada pacote do workspace:

```yaml
name: shared_ui
resolution: workspace
```

Isso informa ao Dart/Melos que o pacote faz parte do workspace.

---

## Comandos Principais

### Bootstrap

Instala dependências e conecta os pacotes locais.

```bash
melos bootstrap
```

---

### Limpeza

Remove arquivos temporários gerados pelo Melos.

```bash
melos clean
```

---

### Execução de comandos em todos os pacotes

```bash
melos exec -- flutter test
```

Útil para:

- Testes
- Análise estática
- Build
- Geração de código

---

### Scripts personalizados

Você pode definir scripts no pubspec.yaml da raiz:

```yaml
scripts:
  analyze:
    run: melos exec -- dart analyze
```

Executar:

```bash
melos run analyze
```

---

## Versionamento

Melos suporta versionamento automático com base em **Conventional Commits**:

Exemplos:

- feat: nova funcionalidade
- fix: correção de bug
- chore: tarefa interna

```bash
melos version
```

Ele:

- Calcula novas versões
- Atualiza pubspec.yaml
- Gera changelog

---

## Publicação

Publica pacotes no pub.dev:

```bash
melos publish
```

Ideal para pipelines CI/CD.

---

## Benefícios do Melos

- Organização clara de monorepos
- Redução de esforço manual
- Padronização de processos
- Melhor escalabilidade
- Integração fácil com CI/CD

---

## Resumo Final

**Melos é ideal para:**

- Projetos Dart/Flutter grandes
- Times com múltiplos pacotes
- Arquitetura modular
- Automação de builds e releases

---

## Links Úteis

- https://pub.dev/packages/melos
- https://melos.invertase.dev

---

Bons estudos 🚀
