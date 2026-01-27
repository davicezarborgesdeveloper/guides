# Guides

Repositório pessoal de guias, documentações e materiais de estudo para desenvolvimento de software.

## Estrutura

```
guides/
├── Backend/
│   └── Security.md          # Diretrizes de segurança para sistemas backend
├── Dev MD/
│   ├── claude.md            # Instruções para IA Claude
│   ├── deepseek.md          # Instruções para IA DeepSeek
│   └── gemini.md            # Instruções para IA Gemini
├── Flutter/
│   ├── Architeture.md       # Arquitetura Flutter (MVVM, Clean Architecture)
│   ├── Automated Testing.md # Testes automatizados no Flutter
│   ├── Flavors.md           # Configuração de flavors (dev/prod)
│   ├── Mason.md             # Geração de código com Mason
│   ├── Modular.md           # Monorepo e modularização enterprise
│   ├── Scripts.md           # Execução de scripts no Flutter
│   ├── Security.md          # Segurança em apps Flutter
│   └── Spotify.md           # Integração Flutter + Spotify SDK
└── Ideias/
    ├── Ideias apps.md       # Trilha de apps Flutter para portfólio
    └── ideias app first offline.md  # Produtos offline-first
```

## Conteúdo

### Flutter

| Guia | Descrição |
|------|-----------|
| **Architeture** | MVVM Google-style, Clean Architecture, Provider vs Riverpod, modularização por feature |
| **Modular** | Monorepo com/sem Melos, lazy loading manual, arquitetura enterprise |
| **Automated Testing** | Testes unitários, widget, integração, golden tests, Firebase Test Lab |
| **Mason** | Templates reutilizáveis (bricks), hooks, geração de features padronizadas |
| **Flavors** | Configuração de ambientes (dev/prod), Android productFlavors, iOS Schemes |
| **Scripts** | Automação de comandos com `rps`, organização de scripts no pubspec |
| **Security** | Armazenamento seguro, criptografia, validação, LGPD no Flutter |
| **Spotify** | Integração com Spotify SDK, leitura de QR, controle de reprodução |

### Backend

| Guia | Descrição |
|------|-----------|
| **Security** | Diretrizes completas para sistema ERP/CRM em Golang + Flutter com Clean Architecture, SOLID, offline-first e LGPD |

### Dev MD

Arquivos de instruções para diferentes IAs assistentes de código, contendo:
- Diretrizes de arquitetura
- Padrões de segurança
- Estrutura de pastas
- Estratégias de teste
- Compliance LGPD

### Ideias

| Guia | Descrição |
|------|-----------|
| **Ideias apps** | Trilha de 15 apps Flutter: 10 para prática e 5 para portfólio profissional |
| **ideias app first offline** | Produtos SaaS offline-first: FieldService, RetailNow, Delivera, MarketFlow |

## Tecnologias Abordadas

- **Mobile/Web**: Flutter, Dart
- **Backend**: Golang
- **Arquitetura**: Clean Architecture, MVVM, Modularização
- **Estado**: Provider, Riverpod, ChangeNotifier
- **Persistência**: Hive, SQLite, Drift
- **Testes**: flutter_test, integration_test, golden tests
- **CI/CD**: GitHub Actions, Firebase Test Lab
- **Segurança**: AES-256, JWT, LGPD

## Como Usar

1. Navegue até a pasta do tema desejado
2. Leia os arquivos `.md` como material de estudo
3. Use como referência durante desenvolvimento
4. Adapte os exemplos para seus projetos

## Licença

Uso pessoal e educacional.
