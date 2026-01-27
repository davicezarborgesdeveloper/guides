# 📜 Especificação Técnica: Ecossistema ERP/CRM Fullstack

Este documento serve como guia mestre para o desenvolvimento do sistema, garantindo conformidade com arquitetura, segurança e tecnologias definidas.

---

## 🛠 1. Stack Tecnológica

### **Backend**

- **Linguagem:** Golang (Go)
- **Foco:** Performance, concorrência nativa e tipagem forte.

### **Frontend (Multiplataforma)**

- **Framework:** Flutter
- **Targets:** Web Portal, Desktop ERP e Mobile App.
- **Gerenciamento de Estado:** Provider (ChangeNotifier).
- **Persistência:** Offline-first com banco de dados local (Hive ou SQLite).
- **Proibições:** - 🚫 Proibido uso de `setState`.
  - 🚫 Proibido uso de `Streams` para gerência de estado de UI.
  - 🚫 Proibido uso da biblioteca `flutter_modular` (Modularização manual).

---

## 🏗 2. Arquitetura e Padrões

### **Clean Architecture**

O projeto deve ser dividido em camadas independentes:

1.  **Domain (Domínio):** Entidades, Casos de Uso (Use Cases) e Interfaces de Repositório. (Lógica de negócio pura).
2.  **Data (Dados):** Implementações de repositórios, DTOs, Data Sources (API/Local DB).
3.  **Presentation (Apresentação):** - **Go:** Handlers/Controllers.
    - **Flutter:** Widgets e ViewModels (Providers).

### **Princípios de Desenvolvimento**

- **SOLID:** Rigor na inversão de dependência e responsabilidade única.
- **Desenvolvimento Otimista:** A UI deve reagir instantaneamente às ações do usuário, sincronizando com o backend em segundo plano e revertendo em caso de falha.
- **Módulos Manuais:** Organização por pastas (ex: `lib/modules/auth`, `lib/modules/sales`) sem dependência de libs externas de modularização.

---

## 🔒 3. Segurança e Compliance (Diretrizes Obrigatórias)

O sistema deve ser construído "Secure by Design":

- **Criptografia:** - Dados sensíveis em repouso (AES-256).
  - Trânsito de dados obrigatoriamente via TLS/HTTPS.
- **LGPD:** - Implementação de logs de auditoria.
  - Gestão de Termos de Aceite e Política de Privacidade com versionamento no banco.
  - Possibilidade de anonimização de dados de clientes.
- **Prevenção de Ataques:**
  - **SQL Injection:** Uso obrigatório de Prepared Statements ou ORM seguro.
  - **Vazamento de Dados:** Sanitização de logs (não logar senhas ou tokens) e tratamento de erros genéricos para o usuário final.
  - **Auth:** Implementação de JWT com tempo de expiração curto e Refresh Tokens.

---

## 🧪 4. Estratégia de Testes

### **Backend (Go)**

- **Unitários:** Testar regras de negócio nos Use Cases.
- **Integração:** Testar a comunicação entre as rotas e o banco de dados.

### **Frontend (Flutter)**

- **Unitários:** Testar lógica das ViewModels (Providers) e Models.
- **Widget Tests:** Testar componentes de UI isolados.
- **Golden Tests:** Testar a integridade visual (renderização pixel-a-pixel).
- **Integration Tests:** Testar fluxos completos (Ex: Fluxo de venda do offline ao online).

---

## 📝 5. Passo a Passo para Implementação

### **Fase 1: Setup e Estrutura**

1. [ ] Inicializar workspace Go e projeto Flutter.
2. [ ] Criar estrutura de pastas seguindo Clean Architecture.
3. [ ] Configurar injeção de dependência manual (Go) e Provider (Flutter).

### **Fase 2: Backend Core**

1. [ ] Implementar Middlewares de segurança (CORS, Auth, Logger).
2. [ ] Criar camada de Domínio (Entidades de CRM/ERP).
3. [ ] Implementar Repositórios com validação de SQL Injection.

### **Fase 3: Frontend & Offline-First**

1. [ ] Configurar banco local para persistência offline.
2. [ ] Implementar lógica de "Sincronizador" (background sync).
3. [ ] Desenvolver UI com componentes reutilizáveis.
4. [ ] Aplicar lógica de Atualização Otimista nos Providers.

### **Fase 4: Qualidade e Documentação**

1. [ ] Cobrir camadas com testes unitários e de integração.
2. [ ] Executar Golden Tests para garantir consistência visual em Web e Mobile.
3. [ ] Revisar conformidade com LGPD (Termos de uso).

---

_Nota: Este documento deve ser atualizado conforme novas regras de negócio forem definidas._
