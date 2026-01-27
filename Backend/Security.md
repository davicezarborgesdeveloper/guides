Os princípios de segurança para APIs REST no backend são fundamentais para garantir que os dados e funcionalidades da aplicação não sejam comprometidos por acessos não autorizados, vazamentos ou ataques. Abaixo estão os principais pilares e práticas recomendadas para garantir a segurança:

## **🔐 1. Autenticação e Autorização**

### **Autenticação**

- **Token-based (JWT, OAuth2):** Verifica a identidade do usuário. Tokens devem ser assinados e ter tempo de expiração.

- **Sessões e Cookies (menos comum em REST):** Podem ser usados, mas exigem precauções como HttpOnly e Secure.

### **Autorização**

- Controle de acesso baseado em perfis/roles (RBAC).

- **Scopes** no OAuth2, para limitar o que o usuário pode acessar.

- Verificação de permissões em cada recurso e ação.

## **🧱 2. Validação de Entrada e Sanitização**

### **Validação**

- Verifique tipos, tamanhos e padrões dos dados recebidos.

- Rejeite campos inesperados.

### **Sanitização**

- Previne **Injeção de SQL, NoSQL, XSS** etc.

- Use ORM seguro (Hibernate, Sequelize, etc.) para evitar injeção direta.

## **🔒 3. Uso de HTTPS**

- **Obrigatório:** Nunca permita requisições HTTP em produção.

- Certifique-se de que redirecionamentos para HTTPS estejam ativos.

- Use **HSTS** para forçar comunicação segura nos navegadores.

## **🧾 4. Controle de Erros e Logs Seguros**

- **Não exponha stack traces** ou mensagens internas da aplicação.

- Mensagens de erro devem ser genéricas para o cliente.

- Logs devem ser monitorados, mas **nunca conter dados sensíveis** como senhas ou tokens.

## **🕵️ 5. Proteção Contra Ataques Comuns**

### **CSRF**

- APIs REST normalmente usam tokens e não são vulneráveis, mas:
  - Se usar cookies, habilite **SameSite** e CSRF Tokens.

### **XSS**

- Filtre qualquer saída que possa ser interpretada como HTML/JS.

- Use bibliotecas de escape.

### **Rate Limiting**

- Implemente limites de requisições por IP/token.

- Prevê ataques de força bruta e negação de serviço (DoS).

### **CORS**

- Configure corretamente os domínios permitidos no CORS.

- Nunca use Access-Control-Allow-Origin: \* em produção com autenticação.

## **🧮 6. Criptografia e Armazenamento Seguro**

- Senhas: use algoritmos de hash seguros como bcrypt, argon2.

- Dados sensíveis (ex: CPF, cartão): armazenar com criptografia simétrica (AES).

- Tokens: armazenar com escopo e tempo de vida definidos, e protegidos de vazamentos.

## **🗂️ 7. Versionamento e Controle de Recursos**

- Use versionamento (/v1/api/...) para manter retrocompatibilidade.

- Limite exposição de dados nos payloads (retorne apenas o necessário).

## **🧰 8. Headers de Segurança**

Configure headers como:

- X-Content-Type-Options: nosniff

- X-Frame-Options: DENY

- Content-Security-Policy

- X-XSS-Protection

## **🧪 9. Testes e Monitoramento de Segurança**

- Testes automatizados com **ZAP**, **OWASP Dependency Check**, etc.

- Realize **pentests regulares**.

- Ferramentas como **Snyk** para verificação de vulnerabilidades em dependências.

## **📜 10. Documentação e Segurança da Documentação**

- Documente a API (ex: Swagger), mas proteja endpoints sensíveis.

- Nunca exponha tokens ou segredos no Swagger/OpenAPI sem autenticação adequada.

# **🔐 1. Autenticação**

Autenticação é o **processo de verificar a identidade** de quem está tentando acessar a API. Antes de qualquer permissão ou acesso ser concedido, é fundamental saber: **Quem é você?**

## **🧭 Por que a autenticação é crítica em APIs REST?**

Em uma API, diferente de uma aplicação web tradicional com interface gráfica, a comunicação é geralmente feita por **aplicações cliente** (mobile, frontend web, outros serviços), então:

- A API **não mantém estado da sessão** entre chamadas.

- Cada requisição precisa ser **independente e segura**.

- Isso exige um mecanismo de autenticação robusto e eficiente.

## **🔐 Formas comuns de autenticação em APIs REST**

### **1.1. JWT (JSON Web Token)**

- O servidor gera um token (string com 3 partes codificadas: header, payload e assinatura).

- O cliente armazena esse token e o envia em cada requisição via cabeçalho Authorization: Bearer \<token\>.

- O servidor **valida o token** sem acessar o banco, usando uma chave secreta ou certificado público.

**Vantagens:**

- Stateless (a API não precisa guardar sessões).

- Rápido e ideal para arquiteturas escaláveis (como microserviços).

**Cuidados:**

- Tokens devem expirar em tempo curto.

- Nunca armazenar dados sensíveis no token (como senhas).

- Se revogação for necessária, requer uma estratégia complementar (ex: blacklist).

### **1.2. OAuth2 (com ou sem OpenID Connect)**

- Muito utilizado para autenticação federada e delegada (ex: login com Google).

- Um terceiro provedor de identidade (Identity Provider) autentica o usuário.

- O cliente recebe um access_token e opcionalmente um refresh_token.

**Vantagens:**

- Permite login com contas externas.

- Recomendado em cenários B2B e APIs públicas com múltiplos consumidores.

**Cuidados:**

- Exige mais configuração e compreensão do fluxo (Authorization Code, Client Credentials, etc.).

- Tokens de acesso devem ter escopos bem definidos.

### **1.3. API Keys**

- Um token simples (ex: 12345-abcde-67890) enviado geralmente no cabeçalho ou query params.

- Usado para identificar **a aplicação cliente** (não o usuário).

**Vantagens:**

- Simples de usar e implementar.

- Útil em serviços machine-to-machine ou de uso interno.

**Cuidados:**

- Sem autenticação do usuário.

- Pode ser facilmente vazado se usado incorretamente (ex: em URL).

- Deve ser rotacionada e expirar periodicamente.

### **1.4. Basic Auth**

- Nome de usuário e senha codificados em Base64 no cabeçalho Authorization: Basic.

- Deve ser usada **apenas em conexões HTTPS** e com usuários bem controlados.

**Desvantagens:**

- Menos seguro, pois as credenciais são expostas em cada requisição (ainda que codificadas).

- Não escalável nem apropriado para usuários finais.

### **1.5. Sessões com Cookies**

- Mais comum em apps web tradicionais.

- O servidor cria uma sessão e associa a um cookie que é enviado ao cliente.

- Em APIs REST, essa abordagem quebra o princípio stateless.

**Por isso, não é recomendada** para APIs puras REST — salvo se você estiver lidando com aplicações híbridas (ex: Single Page App com backend unificado).

## **🛡️ Boas práticas de autenticação**

- \*\*Exigir HTTPS sempre.

  > \*\*

- \*\*Tokens devem expirar.

  > \*\*

- \*\*Nunca armazenar tokens ou credenciais no front sem criptografia.

  > \*\*

- **Evitar tokens na URL** — sempre prefira cabeçalhos HTTP.

- \*\*Auditar tentativas de login e uso de tokens.

  > \*\*

- **Aplicar limitações (rate limit, brute force protection).**

# **🧱 2. Validação de Entrada e Sanitização**

Esse princípio foca em **proteger a API contra dados maliciosos ou malformados**, garantindo que tudo o que entra seja seguro, coerente e controlado.

## **🧭 Por que isso é crítico?**

APIs expõem entradas ao público (formulários, requisições de apps, chamadas externas). Sem validação ou sanitização adequada, um invasor pode:

- Injetar código malicioso (SQL, NoSQL, scripts).

- Corromper dados.

- Derrubar ou manipular o comportamento do backend.

- Escalar privilégios.

## **🧰 Validação de Entrada – Garanta que o dado é o que deveria ser**

A validação consiste em **rejeitar qualquer entrada inválida**, mesmo antes de processá-la.

### **Tipos de validação:**

#### **✅ Tipo de dado**

- Números devem ser números.

- Datas devem seguir formato esperado.

- Booleans devem ser true ou false.

#### **✅ Formato**

- E-mails, CNPJs, telefones devem seguir regex específicos.

- Campos como CEP, códigos ou tokens devem seguir padrões definidos.

#### **✅ Tamanho**

- Limitar tamanho de strings (ex: nomes, senhas).

- Restringir número de itens em listas.

#### **✅ Valores obrigatórios**

- Campos essenciais não podem estar ausentes ou nulos.

#### **✅ Valores permitidos (whitelisting)**

- Só aceitar valores predefinidos para campos controlados (ex: status = ativo, inativo).

#### **✅ Estrutura**

- Objetos complexos devem seguir uma estrutura clara e validável (ex: schemas JSON).

## **🧴 Sanitização – Elimine o que pode ser perigoso**

Sanitização é o processo de **limpar ou neutralizar dados** que possam conter comandos ou scripts maliciosos antes de usá-los.

### **Exemplos de sanitização:**

#### **🧨 Prevenção de SQL Injection**

- Nunca concatenar strings diretamente em queries.

- Mesmo com ORM, evite interpolação direta.

#### **💻 Evitar Cross-Site Scripting (XSS)**

- Limpar ou escapar campos que serão exibidos em UI ou e-mails (ex: \<script\>, onload=, etc.).

#### **🔍 Prevenir Command Injection**

- Evitar passar entradas do usuário como comandos do sistema operacional.

#### **📦 Sanitizar arquivos de upload**

- Verificar extensão, tamanho, tipo MIME, assinaturas.

- Impedir upload de scripts ou binários disfarçados.

## **🚧 Consequências de ignorar essa etapa**

| **Tipo de ataque** | **Exemplo**                      | **Consequência**                     |
| ------------------ | -------------------------------- | ------------------------------------ |
| SQL Injection      | email=1'or'1'='1                 | Acesso ou exclusão de dados do banco |
| NoSQL Injection    | {"user": {"$ne": null}}          | Bypass de autenticação               |
| XSS                | \<script\>alert('x')\</script\>  | Roubo de sessão, phishing            |
| Command Injection  | rm -rf /                         | Execução de comandos no servidor     |
| Buffer Overflow    | input com milhares de caracteres | Travamento da aplicação              |

## **📋 Boas práticas de validação e sanitização**

- \*\*Valide tudo o que vem do cliente, sem exceção.

  > \*\*

- **Nunca confie no frontend** — sempre valide no backend.

- Use **schemas de validação** (JSON Schema, DTOs com validação anotada, etc).

- **Limite o tamanho** de payloads e campos.

- **Evite campos extras**: ignore ou rejeite atributos não esperados.

- **Escape ou sanitize dados antes de exibir ou usar em templates.**

# **🔒 3. Uso de HTTPS**

O uso do **HTTPS (Hypertext Transfer Protocol Secure)** é a base da **segurança na comunicação** entre clientes e servidores em APIs REST. Ele protege os dados que trafegam entre as partes, impedindo que sejam interceptados, lidos ou alterados por terceiros.

## **🧭 Por que HTTPS é indispensável para APIs REST?**

Uma API lida frequentemente com:

- Credenciais de login.

- Tokens de autenticação.

- Dados sensíveis (CPF, dados bancários, informações pessoais).

- Comandos que alteram o estado do sistema (exclusões, atualizações).

**Se essas informações trafegam em HTTP (sem o "S")**, qualquer pessoa na rede pode:

- Espionar os dados (ataques de **sniffing**).

- Interceptar e alterar a comunicação (ataques de **MITM – Man-in-the-Middle**).

- Roubar tokens e se passar por usuários legítimos (**session hijacking**).

## **🔐 O que o HTTPS protege?**

| **Proteção**          | **O que significa?**                                               |
| --------------------- | ------------------------------------------------------------------ |
| **Confidencialidade** | Ninguém além do servidor e cliente pode ver os dados transmitidos. |
| **Integridade**       | Os dados não podem ser alterados durante o trajeto.                |
| **Autenticidade**     | O cliente tem certeza de que está se conectando ao servidor certo. |

## **⚙️ Como o HTTPS funciona (de forma simplificada)?**

1.  O servidor tem um **certificado digital SSL/TLS**, emitido por uma autoridade confiável (CA).

2.  Quando o cliente acessa a API, ocorre um **handshake criptográfico**.

3.  Eles trocam chaves seguras para criptografar e descriptografar dados.

4.  Toda comunicação a partir daí é **criptografada ponta a ponta**.

## **🚨 O que pode dar errado sem HTTPS?**

| **Cenário inseguro**             | **Risco real**                                     |
| -------------------------------- | -------------------------------------------------- |
| Login via HTTP                   | Credenciais vazadas em rede pública ou corporativa |
| API pública sem HTTPS            | Dados sensíveis expostos em trânsito               |
| Envio de token por HTTP          | Roubo de sessão (session hijacking)                |
| Redirecionamento mal configurado | Interceptação antes do HTTPS                       |

## **✅ Boas práticas com HTTPS**

- **Obrigue o uso de HTTPS**: rejeite chamadas HTTP no servidor (status 301 ou 403).

- **Use HSTS (HTTP Strict Transport Security)**:
  - Força navegadores a acessarem apenas via HTTPS.

  - Protege contra downgrades para HTTP.

- **Renove seus certificados antes de vencerem**.
  - Ferramentas como **Let's Encrypt** podem automatizar isso.

- **Evite certificados autoassinados em produção**.
  - Só devem ser usados em ambientes de desenvolvimento controlado.

- **Revogue certificados comprometidos** o mais rápido possível.

- **Monitore a validade dos certificados** via ferramentas externas ou scripts internos.

## **📌 Observação importante:**

**HTTPS não substitui outros mecanismos de segurança.** Ele é apenas a fundação. Se os dados forem mal validados ou os tokens forem armazenados indevidamente no frontend, o uso de HTTPS por si só **não impede ataques.**

# **🧾 4. Controle de Erros e Logs Seguros**

Erros acontecem — sejam por falhas de código, entrada inválida ou problemas externos. O que diferencia uma API segura de uma vulnerável é **como ela lida com esses erros** e **como registra os eventos relevantes**, especialmente em produção.

## **⚠️ Por que o tratamento de erros é uma questão de segurança?**

Mensagens de erro **mal projetadas** podem revelar:

- Detalhes da estrutura interna da API (banco, queries, stack traces).

- Lógica de negócio.

- Endpoints ocultos.

- Nome de tabelas, campos ou tecnologias usadas.

**Exemplo real:**

json

{

"error": "SQLSyntaxErrorException: column 'senha' not found in table 'usuarios'"

}

Esse tipo de mensagem pode:

- Ajudar um invasor a preparar um ataque.

- Facilitar engenharia reversa da lógica da API.

- Revelar vulnerabilidades técnicas específicas.

## **✅ Boas práticas no tratamento de erros**

### **🔒 1. Erros devem ser genéricos para o cliente**

- Exemplo:
  - ✅ **Correto:** {"error": "Erro interno no servidor"}

  - ❌ **Inseguro:** {"error": "NullPointerException at linha 42 do arquivo AuthService.java"}

**Regra de ouro:**

> O cliente precisa saber que **houve um erro**, não **por que exatamente ele aconteceu** (isso fica nos logs).

### **🧼 2. Nunca exponha stack traces**

- Stack trace em produção = **mapa do backend** para atacantes.

- Configure o framework para não incluir stack trace nas respostas (ou oculte em modo produção).

### **🧑‍💻 3. Categorize os erros corretamente**

Utilize códigos HTTP adequados:

| **Código** | **Significado**       | **Quando usar**                       |
| ---------- | --------------------- | ------------------------------------- |
| 400        | Bad Request           | Entrada inválida, parâmetros faltando |
| 401        | Unauthorized          | Falta de autenticação                 |
| 403        | Forbidden             | Autenticado, mas sem permissão        |
| 404        | Not Found             | Recurso inexistente ou acesso negado  |
| 500        | Internal Server Error | Erro inesperado no backend            |
| 422        | Unprocessable Entity  | Erro de validação semântica           |

## **📝 Logs Seguros – Visibilidade sem comprometer a privacidade**

### **🎯 Objetivo dos logs**

- Permitir **monitoramento e rastreamento** de ações.

- Identificar falhas, tentativas de ataque ou comportamento suspeito.

- Ajudar na **auditoria e investigação** de incidentes.

### **🧱 Boas práticas em logging seguro**

#### **🔐 1. Nunca registre dados sensíveis**

- Evite logar:
  - Senhas.

  - Tokens de acesso.

  - Dados bancários ou pessoais (CPF, cartão, endereço).

#### **🔍 2. Registre ações críticas**

- Logins (sucesso e falha).

- Alterações em dados sensíveis.

- Tentativas de acesso negadas.

- Erros internos e exceções tratadas.

#### **🔗 3. Inclua metadados úteis**

- IP de origem.

- ID do usuário (se autenticado).

- Data/hora com timezone.

- Endpoint acessado.

- Parâmetros limpos (sanitizados).

#### **⏳ 4. Mantenha logs com retenção segura**

- Logs devem ter tempo de retenção definido.

- Acesso aos logs deve ser **restrito e monitorado**.

- Use ferramentas de centralização (ex: ELK Stack, Grafana Loki, Graylog).

## **🧨 O que evitar a todo custo**

| **Ação insegura**                   | **Risco imediato**                |
| ----------------------------------- | --------------------------------- |
| Logar senhas ou tokens              | Vazamento de credenciais          |
| Stack trace em ambiente produtivo   | Exposição de código e estrutura   |
| Mensagens de erro com SQL ou código | Base para ataques de injeção      |
| Logs acessíveis publicamente        | Vazamento de dados e rastreamento |

# **🕵️ 5. Proteção Contra Ataques Comuns**

APIs públicas (ou mesmo privadas, se mal protegidas) estão expostas a diversos vetores de ataque. Muitos deles exploram falhas comuns como:

- Dados mal validados.

- Falta de controle de acesso.

- Mau uso de autenticação.

- Falta de proteção contra automação.

## **🧨 1. SQL Injection / NoSQL Injection**

### **Como funciona:**

- Um invasor injeta comandos maliciosos nos campos da requisição (como filtros ou parâmetros).

- Se a API concatena strings diretamente em consultas, ela pode ser manipulada para retornar, alterar ou deletar dados.

### **Prevenção:**

- \*\*Nunca construir queries com concatenação.

  > \*\*

- Use **ORMs seguros** (Hibernate, Sequelize, TypeORM, etc.).

- Sempre **parametrize** suas queries.

- **Valide e sanitize entradas**, como mostrado anteriormente.

## **💻 2. XSS (Cross-Site Scripting)**

### **Como funciona:**

- Injetar scripts maliciosos nos campos da API que depois são exibidos em interfaces frontend (dashboards, e-mails, etc.).

- Pode permitir **roubo de sessão**, **phishing**, ou \*\*execução de código no navegador do usuário.
  > \*\*

### **Prevenção:**

- Escape ou sanitize **qualquer conteúdo que será exibido em HTML**.

- Use bibliotecas de segurança para saída (ex: DOMPurify).

- Nunca confie no conteúdo salvo no banco.

## **🔁 3. CSRF (Cross-Site Request Forgery)**

### **Como funciona:**

- Um site malicioso induz o navegador do usuário a fazer requisições autenticadas contra sua API (por exemplo: POST de exclusão).

- Isso \*\*só acontece se a API estiver usando cookies de autenticação.
  > \*\*

### **Prevenção:**

- APIs REST normalmente usam **tokens no cabeçalho** (Bearer), o que **já mitiga o CSRF**.

- Se for usar cookies, **ative o atributo SameSite=Strict ou Lax**.

- Implemente **tokens CSRF** em formulários web quando necessário.

## **🚫 4. Brute Force e Credential Stuffing**

### **Como funciona:**

- O atacante tenta inúmeras combinações de login até acertar.

- No caso de stuffing, ele usa listas reais de logins/senhas vazadas de outros serviços.

### **Prevenção:**

- Implemente **rate limiting** por IP, por endpoint e por token.

- Use **proteções com CAPTCHA** após falhas seguidas.

- **Bloqueio temporário ou progressivo** após tentativas repetidas.

## **🌐 5. CORS (Cross-Origin Resource Sharing)**

### **Como funciona:**

- APIs abertas a qualquer origem (Access-Control-Allow-Origin: \*) permitem que **qualquer site acesse seus dados**, inclusive com o token do usuário autenticado (se mal configurado).

### **Prevenção:**

- Defina explicitamente as origens permitidas (ex: https://meusite.com).

- Não use \* em produção, especialmente se a API exige autenticação.

- Configure corretamente os cabeçalhos Access-Control-Allow-Headers e Allow-Credentials.

## **🌊 6. DoS / DDoS (Negação de Serviço)**

### **Como funciona:**

- O sistema é sobrecarregado com milhares de requisições, esgotando recursos.

### **Prevenção:**

- **Rate limiting** com regras inteligentes (por IP, por token).

- **Cache de respostas** em endpoints públicos e repetitivos.

- Usar ferramentas de proteção como **Cloudflare**, **AWS WAF**, **API Gateway com throttling**.

## **📦 7. Upload de Arquivos Maliciosos**

### **Como funciona:**

- Uploads de arquivos que escondem scripts, vírus, ou que exploram falhas no parser.

### **Prevenção:**

- Verifique:
  - Tipo MIME.

  - Extensão permitida.

  - Tamanho máximo.

  - Assinatura do arquivo (ex: magic number).

- **Armazene os arquivos fora da pasta pública**.

- **Nunca execute nem renderize diretamente os arquivos recebidos**.

## **📋 Boas práticas gerais de proteção**

- \*\*Rate limiting e monitoramento contínuo.

  > \*\*

- **Requisições com tempo máximo (timeout)** para evitar travamentos.

- **Respostas padronizadas e controladas**, sem revelar estrutura da API.

- **Firewalls de aplicação (WAF)** para prevenir padrões conhecidos de ataque.

- Use **bibliotecas de segurança atualizadas**.

- **Teste constantemente com ferramentas automatizadas (ex: OWASP ZAP, Burp Suite)**.

# **🧮 6. Criptografia e Armazenamento Seguro**

Este princípio trata da **proteção dos dados em repouso** (armazenados) e da aplicação de **métodos criptográficos** para garantir que, mesmo que haja acesso indevido ao banco ou ao sistema de arquivos, os dados continuem **ilegíveis e protegidos**.

## **🔐 Por que criptografar dados?**

Mesmo com todos os controles de acesso, falhas podem ocorrer:

- Um banco de dados pode ser exposto.

- Um backup pode vazar.

- Um atacante pode explorar uma brecha interna.

Se os dados estiverem **criptografados ou fortemente hasheados**, eles se tornam **inúteis para quem os rouba**.

## **📂 Tipos de dados que devem ser protegidos**

| **Tipo de dado**               | **Recomendação**                      |
| ------------------------------ | ------------------------------------- |
| Senhas de usuários             | Hash com salt e algoritmo forte       |
| Tokens de autenticação/sessão  | Criptografia ou expiração curta       |
| CPF, CNPJ, dados bancários     | Criptografia com chave controlada     |
| Informações de saúde, educação | Criptografia e controle de acesso     |
| Dados financeiros e transações | Criptografia e registros de auditoria |

## **🔄 Hash vs Criptografia – Qual a diferença?**

| **Conceito**  | **Hash**                           | **Criptografia**           |
| ------------- | ---------------------------------- | -------------------------- |
| É reversível? | ❌ Não                             | ✅ Sim (com chave certa)   |
| Usado para    | Verificação (ex: senhas)           | Proteção e sigilo de dados |
| Exemplos      | bcrypt, argon2, SHA-256 (com salt) | AES, RSA, TLS              |

## **🧾 Proteção de senhas – Hash seguro**

Senhas **nunca devem ser armazenadas como texto puro (plaintext)**, nem mesmo criptografadas de forma reversível.

### **✅ Algoritmos recomendados:**

- **bcrypt** (mais usado em APIs web)

- **argon2** (moderno e resistente a ataques de GPU)

- **scrypt** (resistente a brute-force)

### **⚠️ Regras de ouro:**

- Sempre aplicar **salt** (valor aleatório por senha).

- Não usar SHA-256 puro (muito rápido para senhas).

- Evite MD5 ou SHA1 – obsoletos e inseguros.

## **🔐 Criptografia de dados sensíveis**

Para dados como CPF, número de cartão, nome completo, etc., o ideal é **usar criptografia simétrica (como AES)** com uma chave mantida em local seguro.

### **🔒 Requisitos:**

- Use **AES-256** com modo seguro (GCM, CBC com IV aleatório).

- Armazene a **chave criptográfica em um cofre seguro** (ex: AWS KMS, Azure Key Vault, HashiCorp Vault).

- Nunca hardcode a chave no código-fonte.

- Rotacione chaves periodicamente, se possível.

## **🧩 Tokens e identificadores**

- **Tokens JWT** devem ser **assinados (com chave privada)** e, se necessário, criptografados.

- **Refresh tokens** ou identificadores de sessão devem ser armazenados com criptografia se salvos no banco.

- Nunca exponha IDs internos diretamente (ex: ID sequencial de usuário) — use **UUIDs** ou tokens opacos.

## **🛡️ Proteção de arquivos e backups**

- Arquivos enviados ou gerados pela API devem ser **armazenados criptografados**, especialmente se forem sensíveis.

- **Backups de banco de dados** devem ser criptografados em repouso e no trânsito.

- Nunca exponha buckets de armazenamento diretamente sem autenticação.

## **🔍 Auditoria e integridade**

- Mantenha \*\*logs de acesso a dados criptografados.

  > \*\*

- Verifique a **integridade dos dados** com checksums ou HMACs.

- Aplique criptografia de ponta a ponta se possível (ex: entre microserviços ou no frontend).

## **✅ Boas práticas de armazenamento seguro**

- Minimize o volume de dados sensíveis salvos.

- Só armazene o que for **estritamente necessário**.

- Aplique **princípio do menor privilégio**: acesso restrito por função e finalidade.

- Implemente **camadas de defesa**: criptografia + autenticação + logs + monitoramento.

# **🗂️ 7. Versionamento e Controle de Recursos**

Este princípio trata de **como organizar os endpoints da API de forma segura, previsível e sustentável**, evitando:

- Quebras acidentais em clientes que já usam a API.

- Exposição indevida de dados.

- Ambiguidades nos contratos de comunicação entre cliente e servidor.

## **📌 Por que versionar uma API REST?**

APIs evoluem com o tempo: novos recursos, mudanças em estruturas de dados, exclusão de campos ou funcionalidades antigas.

Se não houver versionamento, uma mudança pode:

- **Quebrar aplicações em produção** que consomem a versão anterior.

- **Gerar vulnerabilidades** por dependência de comportamento não documentado.

- Comprometer a experiência de outros consumidores da API.

## **✅ Boas práticas de versionamento**

### **🔢 1. Incluir a versão na URL da API**

A forma mais clara, comum e recomendada:

http

GET /api/v1/usuarios

POST /api/v2/relatorios

**Vantagens:**

- Simples de entender.

- Suporte simultâneo a múltiplas versões.

- Fácil de controlar e documentar.

**Nota:** A versão **faz parte do contrato da API**, e mudanças incompatíveis devem levar a uma nova versão.

### **🧠 2. Versão deve refletir mudança de contrato**

Você **só precisa subir uma nova versão** da API quando:

- Campos são **removidos ou renomeados**.

- A semântica de um campo muda.

- Um recurso muda sua estrutura de retorno ou comportamento.

**Exemplo que exige nova versão:**

- /usuarios retorna nome completo no v1 e passa a retornar nome e sobrenome separados no v2.

### **🧪 3. Controle de versões no backend**

- Mantenha cada versão da API isolada (por controladores, pacotes, módulos).

- Evite duplicação, mas também **não cruze lógica entre versões** incompatíveis.

## **📦 Controle de Recursos — Retorne só o necessário**

### **🔐 1. Evite exposição de dados além do necessário**

Cada recurso deve retornar:

- Apenas os **atributos relevantes ao contexto**.

- Nada de campos internos, técnicos ou de depuração.

**Exemplo de erro comum:**

json

{

"id": 42,

"email": "admin@empresa.com",

"senhaHash": "$2a$10$...",

"ultimaAtualizacao": "2025-08-01"

}

> Nesse caso, o campo senhaHash **nunca deveria ser exposto** em um retorno público.

### **🧾 2. Filtros, paginação e projeções**

- Use parâmetros para limitar a resposta:
  - GET /usuarios?page=1&limit=20

  - GET /relatorios?mes=07&ano=2025

- Permita **retornar somente os campos necessários**, quando aplicável:
  - GET /produtos?fields=id,nome,preco

Isso evita:

- Respostas gigantes.

- Exposição de dados desnecessários.

- Processamento excessivo no cliente.

### **🔎 3. URLs RESTful bem definidas**

Padronize os endpoints:

http

GET /clientes → lista clientes

GET /clientes/{id} → detalhes de um cliente

POST /clientes → cria um cliente

PUT /clientes/{id} → atualiza um cliente

DELETE /clientes/{id} → remove um cliente

**Benefícios:**

- Previsibilidade.

- Facilidade de documentação.

- Menor risco de endpoints escondidos ou desorganizados.

### **🛡️ 4. Evite endpoints genéricos e ambíguos**

Endpoints como /api/processaTudo ou /api/run/{query}:

- São difíceis de auditar.

- Dão margem a abusos ou comportamentos não documentados.

- Podem esconder lógica sensível mal protegida.

## **🚧 Riscos se esse princípio for ignorado**

| **Problema**           | **Consequência**                      |
| ---------------------- | ------------------------------------- |
| Falta de versionamento | Quebra de apps clientes, regressões   |
| Retorno excessivo      | Vazamento de dados, uso de banda      |
| URLs mal definidas     | Ambiguidade, endpoints mal protegidos |
| Sem paginação          | Excesso de carga e lentidão           |

Esse princípio garante que a API possa crescer de forma controlada, **sem sacrificar segurança ou previsibilidade**.

# **🧰 8. Headers de Segurança**

Os **HTTP Security Headers** são **camadas adicionais de proteção** que atuam no nível da comunicação entre cliente e servidor. Eles ajudam a:

- Prevenir ataques como XSS, clickjacking e content sniffing.

- Restringir o comportamento do navegador.

- Garantir que a API seja consumida de forma segura.

Mesmo sendo mais comuns em aplicações web, **muitos desses headers também se aplicam a APIs REST**, especialmente quando integradas a frontends, apps ou terceiros.

## **📦 Principais headers de segurança**

### **🧼 1. X-Content-Type-Options: nosniff**

### **❓ O que faz:**

Evita que o navegador tente **adivinhar o tipo de conteúdo** (MIME sniffing) quando ele não corresponde ao declarado.

### **✅ Por que é útil:**

- Impede que scripts sejam executados como outro tipo de mídia.

- Reduz o risco de execução não intencional de conteúdo malicioso.

### **🧱 2. X-Frame-Options: DENY ou SAMEORIGIN**

### **❓ O que faz:**

Controla se a API ou aplicação pode ser embutida em um \<iframe\>.

### **✅ Por que é útil:**

- Protege contra **clickjacking** — quando um site malicioso tenta enganar o usuário para clicar em elementos ocultos.

- Use DENY para bloquear totalmente ou SAMEORIGIN se houver uso interno controlado.

### **💻 3. Content-Security-Policy (CSP)**

### **❓ O que faz:**

Define regras de quais recursos externos podem ser carregados (scripts, fontes, imagens, etc.).

### **✅ Por que é útil:**

- Reduz **drasticamente o risco de XSS**, injeções de script e outras vulnerabilidades client-side.

- Para APIs puras (sem UI), o uso é limitado, mas **essencial em aplicações híbridas** (API com Swagger UI, por exemplo).

### **🔒 4. Strict-Transport-Security (HSTS)**

### **❓ O que faz:**

Informa ao navegador que o site **deve ser acessado apenas via HTTPS**, mesmo que o usuário tente acessar via HTTP.

http

Strict-Transport-Security: max-age=31536000; includeSubDomains

### **✅ Por que é útil:**

- Evita ataques de downgrade (ex: SSL stripping).

- \*\*Obrigatório para APIs seguras com HTTPS.
  > \*\*

### **🚫 5. Access-Control-Allow-Origin (CORS)**

### **❓ O que faz:**

Controla **quais domínios têm permissão para fazer requisições cross-origin** para a API.

### **✅ Por que é útil:**

- Impede que **qualquer site externo acesse sua API** com o token do usuário.

- \*\*Nunca usar \* se a API exigir autenticação.

  > \*\*

- Configure de forma restrita e dinâmica, se necessário:

http

Access-Control-Allow-Origin: https://meusite.com

### **🧪 6. Referrer-Policy**

### **❓ O que faz:**

Controla quais informações de origem (Referer) são enviadas em requisições para outros domínios.

### **✅ Por que é útil:**

- Evita **exposição de tokens na URL**, dados sensíveis ou rotas privadas.

- Valor recomendado: no-referrer ou strict-origin-when-cross-origin.

## **📋 Exemplo de headers seguros em uma resposta de API**

http

HTTP/1.1 200 OK

Strict-Transport-Security: max-age=31536000; includeSubDomains

X-Content-Type-Options: nosniff

X-Frame-Options: DENY

Content-Security-Policy: default-src 'none'

Referrer-Policy: no-referrer

Access-Control-Allow-Origin: https://meusite.com

## **🚨 Cuidados ao configurar**

- \*\*Evite configurações genéricas ou permissivas.

  > \*\*

- \*\*Testar localmente e em staging antes de aplicar em produção.

  > \*\*

- Cuidado com o uso de CSP em APIs puras — pode interferir no funcionamento de UIs embutidas (como Swagger ou Painéis Admin).

## **✅ Boas práticas**

- Sempre use X-Content-Type-Options e X-Frame-Options, mesmo em APIs.

- Configure HSTS corretamente e **monitore o comportamento após ativar**.

- Ajuste o CORS com base no uso real da API.

- Considere ferramentas como **Helmet.js** (Node), **Spring Security Headers** (Java), ou configurações nativas em nginx/apache para aplicar esses headers de forma automática.

# **🧪 9. Testes e Monitoramento de Segurança**

Não basta implementar segurança — é preciso **verificar continuamente se ela está de fato funcionando**, e monitorar **tentativas de violação, falhas de configuração ou novos riscos**.

Esse princípio garante que sua API esteja:

- Livre de vulnerabilidades conhecidas.

- Resiliente a ataques automatizados.

- Sendo **observada ativamente** para comportamentos suspeitos.

## **🔍 Testes de Segurança**

### **🧪 1. Testes Automatizados (Segurança Estática e Dinâmica)**

#### **✅ SAST (Static Application Security Testing)**

- Análise de código fonte em busca de vulnerabilidades.

- Detecta más práticas, uso de funções inseguras, falhas de validação.

**Ferramentas:**

- SonarQube

- Semgrep

- Checkmarx

- Fortify

#### **✅ DAST (Dynamic Application Security Testing)**

- Testa a API rodando, simulando ataques e fuzzing.

- Ideal para detectar problemas como:
  - Injeções (SQL/NoSQL).

  - Quebra de autenticação.

  - Erros de configuração.

**Ferramentas:**

- OWASP ZAP (Zed Attack Proxy)

- Burp Suite

- Postman Security Scanner

### **🧪 2. Testes Manuais / Pentests**

- Realizados por analistas especializados ou red teams.

- Simulam ataques reais, exploram falhas de lógica e autorização.

- Avaliam mais do que o código: \*\*design, arquitetura e comportamento da API.
  > \*\*

**Recomendado em:**

- APIs públicas ou críticas.

- Lançamentos de novas versões.

- Integrações com terceiros.

### **✅ 3. Testes em dependências (SCA)**

- APIs modernas usam muitas bibliotecas.

- **Vulnerabilidades conhecidas em pacotes de terceiros** podem expor sua aplicação.

**Ferramentas:**

- Snyk

- OWASP Dependency-Check

- NPM Audit / Composer Audit / Maven Plugin

## **📡 Monitoramento de Segurança**

### **🎯 1. Logging e rastreamento de atividades**

- Registre tentativas de login, ações sensíveis, falhas de autenticação.

- Inclua:
  - IP, timestamp, ID do usuário, endpoint acessado.

- Utilize serviços centralizados:
  - ELK Stack (Elasticsearch, Logstash, Kibana)

  - Grafana + Loki

  - Datadog, Splunk, New Relic

### **🚦 2. Alertas e detecção de anomalias**

- Configure alertas para eventos suspeitos:
  - Muitas falhas de login em pouco tempo.

  - Tentativas de acesso a recursos inexistentes.

  - Ataques de força bruta ou fuzzing.

- Integre com sistemas de notificação (ex: Slack, PagerDuty, e-mail).

### **🛡️ 3. WAF (Web Application Firewall)**

- Atua como filtro de tráfego HTTP.

- Detecta e bloqueia padrões de ataque antes de chegarem à API.

**Exemplos:**

- AWS WAF

- Cloudflare

- Azure Application Gateway

- ModSecurity (para nginx/apache)

### **🔐 4. Monitoramento de integridade**

- Verifica se arquivos, banco ou configurações foram alterados sem autorização.

- Importante para detectar **acessos indevidos ou backdoors**.

## **🗓️ Ciclo contínuo de segurança**

A segurança não é evento único, é um **processo cíclico**:

text

Planejar → Implementar → Testar → Corrigir → Monitorar → Melhorar

- Adote uma rotina de **testes regulares (mensal/trimestral)**.

- **Corrija vulnerabilidades rapidamente** com base em relatórios.

- Mantenha todos os componentes e bibliotecas atualizados.

## **✅ Boas práticas finais**

- **Inclua testes de segurança no CI/CD**, como etapa obrigatória.

- **Acompanhe CVEs e vulnerabilidades emergentes**.

- Mantenha a equipe atualizada com as melhores práticas.

- Integre o monitoramento com a **resposta a incidentes**.

Esse princípio garante que a segurança da sua API **não se degrade com o tempo**, mesmo após novas versões ou integrações.

# **📜 10. Documentação e Segurança da Documentação**

Ter uma API bem documentada é essencial para **adoção, uso correto e manutenção**. No entanto, **a documentação também pode expor riscos de segurança** se for mal gerenciada.

## **🧭 Por que a documentação pode ser uma ameaça?**

Se a documentação:

- Expõe **endpoints não utilizados ou internos**,

- Revela **detalhes sensíveis de headers, tokens ou respostas**,

- Está **publicamente acessível** sem proteção,

então ela pode se tornar um **mapa de ataque** para qualquer pessoa mal-intencionada.

## **📚 Formas comuns de documentar APIs**

- \*\*Swagger / OpenAPI (ex: Swagger UI)

  > \*\* Interface visual interativa para testar endpoints.

- \*\*Postman Collections

  > \*\* Conjunto de requisições organizadas com exemplos.

- \*\*Markdown em repositórios

  > \*\* Documentos escritos com exemplos e instruções.

- \*\*Portais públicos ou privados de desenvolvedores (API Gateway, DevPortals)
  > \*\*

## **🔐 Riscos de segurança na documentação**

| **Risco**                                 | **Consequência**                                   |
| ----------------------------------------- | -------------------------------------------------- |
| Tokens reais nos exemplos                 | Vazamento de credenciais                           |
| Endpoints internos documentados           | Exploração indevida por terceiros                  |
| Falta de autenticação no Swagger UI       | Qualquer um pode testar e acessar                  |
| Documentação exposta publicamente         | Ataques automatizados baseados na estrutura da API |
| Descrição excessiva de regras de negócios | Engenharia reversa facilitada                      |

## **✅ Boas práticas para proteger a documentação**

### **🛡️ 1. Proteja o acesso à documentação interativa**

- **Swagger UI e Postman devem exigir autenticação** se a API for protegida.

- Utilize ambientes diferentes (ex: documentação apenas em staging).

- Remova ou oculte endpoints sensíveis ou administrativos.

### **🔐 2. Nunca inclua tokens, senhas ou dados reais**

Substitua por placeholders:

json

"Authorization": "Bearer {seu_token_aqui}"

-

- Não documente **segredos ou variáveis sensíveis** nos exemplos.

### **📑 3. Mantenha a documentação alinhada com a política de acesso**

- Se um endpoint exige token de admin, isso deve estar **claro e visível**.

- Campos retornados devem refletir exatamente o que será entregue — **sem campos internos ocultos**.

### **🧼 4. Revise e higienize periodicamente**

- Documentação é viva: deve ser mantida em sincronia com o código.

- Remova:
  - Endpoints depreciados ou desabilitados.

  - Comentários de testes ou instruções antigas.

  - Dados de debug.

### **🌐 5. Controle onde a documentação é hospedada**

- Se for pública:
  - Exponha apenas versões seguras e estáveis.

  - Retire detalhes técnicos avançados.

  - Use URLs com controle de escopo (ex: /docs/v1, não /admin/internal/docs).

- Se for privada:
  - Proteja com autenticação.

  - Monitore acessos e alterações.

### **📋 6. Não exponha detalhes excessivos do backend**

Evite mostrar:

- Estrutura de banco de dados.

- Stack traces.

- Frameworks ou tecnologias internas.

## **🛠️ Ferramentas que ajudam**

| **Ferramenta**    | **Função**                                                   |
| ----------------- | ------------------------------------------------------------ |
| Swagger / OpenAPI | Geração e visualização da documentação interativa            |
| Redoc             | Visualização estática e elegante de OpenAPI                  |
| Postman           | Criação e compartilhamento de coleções com ambientes seguros |
| Stoplight         | Portal de API com controle de acesso e publicação            |

## **✅ Checklist de segurança da documentação**

- Swagger UI exige autenticação?

- Não há tokens ou dados reais nos exemplos?

- Endpoints internos estão ocultos?

- Campos sensíveis estão fora das respostas?

- A documentação está separada por versão?

- Está hospedada em ambiente seguro ou protegido?

Documentação segura garante que os consumidores da API tenham uma boa experiência, **sem comprometer a segurança da aplicação**.

# **🔐 1. Autenticação e Autorização em API REST com PHP + MySQL**

## **📌 Objetivo**

- Implementar autenticação via **e-mail e senha**.

- Gerar e verificar **tokens JWT assinados**.

- **Autorizar acesso** com base no token.

- Validar permissões do usuário (autorização por papel).

## **🧱 Estrutura geral do projeto**

kotlin

/api

├── db.php → conexão com MySQL

├── login.php → autenticação e geração do token JWT

├── protected.php → rota protegida, só acessível com token válido

├── auth.php → middleware de validação do token

└── utils.php → funções auxiliares (JWT, base64url, etc)

## **🧩 Tabela de exemplo no MySQL**

sql

CREATE TABLE usuarios (

id INT AUTO_INCREMENT PRIMARY KEY,

email VARCHAR(255) NOT NULL UNIQUE,

senha_hash VARCHAR(255) NOT NULL,

papel ENUM('admin', 'usuario') DEFAULT 'usuario'

);

## **🔌 db.php – Conexão segura com o MySQL**

php

\<?php

function conectarBD() {

return new PDO('mysql:host=localhost;dbname=api_seguranca;charset=utf8mb4', 'seu_usuario', 'sua_senha', \[

PDO::ATTR_ERRMODE =\> PDO::ERRMODE_EXCEPTION,

PDO::ATTR_DEFAULT_FETCH_MODE =\> PDO::FETCH_ASSOC

\]);

}

## **🛠️ utils.php – Funções JWT**

php

\<?php

$chaveSecreta = 'sua-chave-secreta-segura';

// Base64 URL Safe

function base64url_encode($data) {

return rtrim(strtr(base64_encode($data), '+/', '-\_'), '=');

}

function base64url_decode($data) {

return base64_decode(strtr($data, '-\_', '+/'));

}

// Geração do token

function gerarJWT($payload, $chave) {

$header = base64url_encode(json_encode(\['alg' =\> 'HS256', 'typ' =\> 'JWT'\]));

$payload = base64url_encode(json_encode($payload));

$assinatura = hash_hmac('sha256', "$header.$payload", $chave, true);

$assinatura = base64url_encode($assinatura);

return "$header.$payload.$assinatura";

}

// Validação do token

function verificarJWT($token, $chave) {

if (substr_count($token, '.') !== 2) return false;

list($headerB64, $payloadB64, $sigB64) = explode('.', $token);

$verifAss = base64url_encode(hash_hmac('sha256', "$headerB64.$payloadB64", $chave, true));

if (!hash_equals($verifAss, $sigB64)) return false;

$payload = json_decode(base64url_decode($payloadB64), true);

if (!$payload \|\| time() \> $payload\['exp'\]) return false;

return $payload;

}

## **🔑 login.php – Autenticação e geração do JWT**

php

\<?php

require 'db.php';

require 'utils.php';

$dados = json_decode(file_get_contents('php://input'), true);

$email = $dados\['email'\] ?? '';

$senha = $dados\['senha'\] ?? '';

if (!$email \|\| !$senha) {

http_response_code(400);

exit(json_encode(\['erro' =\> 'Email e senha obrigatórios'\]));

}

$pdo = conectarBD();

$stmt = $pdo-\>prepare("SELECT \* FROM usuarios WHERE email = ?");

$stmt-\>execute(\[$email\]);

$usuario = $stmt-\>fetch();

if (!$usuario \|\| !password_verify($senha, $usuario\['senha_hash'\])) {

http_response_code(401);

exit(json_encode(\['erro' =\> 'Credenciais inválidas'\]));

}

$payload = \[

'sub' =\> $usuario\['id'\],

'email' =\> $usuario\['email'\],

'papel' =\> $usuario\['papel'\],

'iat' =\> time(),

'exp' =\> time() + 3600

\];

$token = gerarJWT($payload, $chaveSecreta);

echo json_encode(\['token' =\> $token\]);

## **🔒 auth.php – Middleware de autorização via token**

php

\<?php

require 'utils.php';

$headers = getallheaders();

$authHeader = $headers\['Authorization'\] ?? '';

$token = str_replace('Bearer ', '', $authHeader);

$usuario = verificarJWT($token, $chaveSecreta);

if (!$usuario) {

http_response_code(401);

exit(json_encode(\['erro' =\> 'Token inválido ou expirado'\]));

}

// Você agora tem: $usuario\['sub'\], $usuario\['email'\], $usuario\['papel'\]

## **🔐 protected.php – Endpoint protegido com verificação de papel**

php

\<?php

require 'auth.php';

// Exemplo: permitir apenas admins

if ($usuario\['papel'\] !== 'admin') {

http_response_code(403);

exit(json_encode(\['erro' =\> 'Acesso não autorizado'\]));

}

echo json_encode(\[

'mensagem' =\> 'Bem-vindo, administrador!',

'usuario_id' =\> $usuario\['sub'\]

\]);

## **✅ Resumo do que você tem até aqui**

- Login com e-mail/senha via MySQL.

- Hash seguro com password_hash() / password_verify().

- Geração de JWT manual, com verificação de expiração e assinatura.

- Proteção de endpoints via auth.php.

- Autorização por papel (admin, usuario, etc.).

# **🧱 2. Validação de Entrada e Sanitização**

> Objetivo: garantir que **nenhum dado malicioso ou inválido** entre no seu sistema ou banco de dados.

## **🧨 Por que isso é crítico?**

Sem validação, sua API pode sofrer:

- \*\*SQL Injection

  > \*\*

- \*\*XSS (Cross-Site Scripting)

  > \*\*

- \*\*Armazenamento de dados inconsistentes

  > \*\*

- \*\*Quebras inesperadas na aplicação
  > \*\*

## **✅ Etapas principais**

1.  **Validação** → confirmar se o dado tem o formato esperado.

2.  **Sanitização** → limpar/remover elementos perigosos ou desnecessários.

## **🔒 Exemplos de ataques comuns**

### **🔥 SQL Injection**

sql

email=admin@teste.com' OR 1=1 --

### **🔥 XSS**

html

\<script\>alert('roubo de sessão')\</script\>

## **📌 Contexto de exemplo**

Suponha um endpoint para **cadastro de usuários** com os campos:

- email

- senha

- nome

## **🛠️ Exemplo com validação e sanitização**

php

\<?php

require 'db.php';

// Recebe JSON da requisição

$dados = json_decode(file_get_contents('php://input'), true);

// 1. Validação (campos obrigatórios e formato)

if (

empty($dados\['email'\]) \|\|

empty($dados\['senha'\]) \|\|

empty($dados\['nome'\]) \|\|

!filter_var($dados\['email'\], FILTER_VALIDATE_EMAIL)

) {

http_response_code(400);

exit(json_encode(\['erro' =\> 'Campos obrigatórios inválidos'\]));

}

// 2. Sanitização (limpar caracteres maliciosos)

$email = filter_var($dados\['email'\], FILTER_SANITIZE_EMAIL);

$nome = htmlspecialchars(trim($dados\['nome'\]), ENT_QUOTES, 'UTF-8');

$senha = trim($dados\['senha'\]);

// 3. Regras de negócios

if (strlen($senha) \< 6) {

http_response_code(400);

exit(json_encode(\['erro' =\> 'A senha deve ter no mínimo 6 caracteres'\]));

}

// 4. Inserção segura no banco (uso de prepared statements)

$pdo = conectarBD();

$stmt = $pdo-\>prepare("INSERT INTO usuarios (email, senha_hash, papel) VALUES (?, ?, ?)");

try {

$stmt-\>execute(\[

$email,

password_hash($senha, PASSWORD_DEFAULT),

'usuario'

\]);

echo json_encode(\['mensagem' =\> 'Usuário cadastrado com sucesso'\]);

} catch (PDOException $e) {

http_response_code(400);

echo json_encode(\['erro' =\> 'Erro ao inserir: ' . $e-\>getMessage()\]);

}

## **✅ Explicações das proteções**

| **Etapa**                              | **Proteção**                                      |
| -------------------------------------- | ------------------------------------------------- |
| filter_var(..., FILTER_VALIDATE_EMAIL) | Garante formato válido de e-mail                  |
| htmlspecialchars()                     | Protege contra XSS                                |
| trim()                                 | Remove espaços escondidos (evita entradas falsas) |
| password_hash()                        | Hash seguro da senha                              |
| PDO::prepare + execute                 | Previne SQL Injection                             |
| try/catch                              | Controla exceções seguras                         |

## **⚠️ O que não fazer**

- ❌ Concatenar strings em SQL ("... WHERE email = '$email'")

- ❌ Aceitar HTML sem sanitização

- ❌ Confiar nos dados do frontend

- ❌ Ignorar campos inesperados no JSON

## **🔁 Extras (opcional)**

**Valide tamanho máximo dos campos**:

php

if (strlen($nome) \> 100) {

exit(json_encode(\['erro' =\> 'Nome muito longo'\]));

}

-

**Valide tipo de dados esperados (inteiros, datas, booleans)**:

php

if (!is_numeric($idade)) { ... }

-

## **✅ Resumo**

- Validar é **confirmar que o dado é aceitável**.

- Sanitizar é **limpar o que não deveria estar ali**.

- Sempre use filter_var(), htmlspecialchars() e **prepared statements** com PDO.

- **Nunca confie no cliente**. Toda validação é feita **no backend**.

# **🔒 3. Uso de HTTPS em APIs REST com PHP + MySQL**

## **📌 O que é HTTPS?**

- HTTPS = HTTP + \*\*SSL/TLS

  > \*\*

- Garante que a comunicação entre cliente e servidor seja:
  - \*\*Criptografada

    > \*\*

  - \*\*Íntegra

    > \*\*

  - \*\*Autenticada
    > \*\*

## **🧭 Por que é obrigatório?**

### **Sem HTTPS:**

- Qualquer pessoa na rede pode ver:
  - Tokens JWT

  - E-mails, senhas

  - Dados sensíveis

- Vulnerável a ataques como:
  - \*\*MITM (Man-in-the-Middle)

    > \*\*

  - \*\*Sniffing de pacotes

    > \*\*

  - \*\*Session hijacking
    > \*\*

### **Com HTTPS:**

- Os dados viajam **protegidos contra espionagem e alteração**.

- Segurança básica para qualquer API real.

## **✅ Como aplicar HTTPS em ambiente PHP**

Você deve **configurar o HTTPS no servidor web**, e o PHP só deve **exigir** e **verificar** essa conexão.

## **🛠️ Etapa 1: Configurar HTTPS no Apache ou Nginx**

### **🔐 A. Obtenha um certificado SSL**

- **Gratuito**: Let's Encrypt

- **Pago**: Com empresas como GoDaddy, Sectigo etc.

### **🔧 B. Configuração básica (exemplo com Apache)**

apache

\<VirtualHost \*:443\>

ServerName api.exemplo.com

DocumentRoot /var/www/api

SSLEngine on

SSLCertificateFile /etc/letsencrypt/live/api.exemplo.com/fullchain.pem

SSLCertificateKeyFile /etc/letsencrypt/live/api.exemplo.com/privkey.pem

\<Directory /var/www/api\>

Options -Indexes

AllowOverride All

Require all granted

\</Directory\>

\</VirtualHost\>

\# Redirecionar HTTP para HTTPS

\<VirtualHost \*:80\>

ServerName api.exemplo.com

Redirect permanent / https://api.exemplo.com/

\</VirtualHost\>

## **🛡️ Etapa 2: Forçar HTTPS no PHP (proteção extra)**

### **🚫 Bloqueia requisições sem HTTPS**

php

\<?php

if (empty($\_SERVER\['HTTPS'\]) \|\| $\_SERVER\['HTTPS'\] === 'off') {

http_response_code(403);

exit(json_encode(\['erro' =\> 'Conexão insegura. Use HTTPS.'\]));

}

**Coloque esse trecho no início de endpoints sensíveis**, como login, cadastro, troca de senha, etc.

## **🧪 Etapa 3: Testar HTTPS corretamente**

- Acesse sua API via https:// e não http://

- Teste redirecionamento automático

- Use ferramentas como:
  - SSL Labs Test

  - Postman com https://...

## **🔐 Etapa 4: HSTS (HTTP Strict Transport Security)**

> Evita que o navegador volte a usar HTTP.

### **Apache:**

apache

Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"

## **⚠️ Atenção com desenvolvimento local**

Durante o desenvolvimento:

- Use localhost sem HTTPS apenas em ambiente controlado.

- Pode usar **certificados autoassinados** para simular HTTPS.

## **✅ Resumo**

| **Ação**                              | **Motivo**                                |
| ------------------------------------- | ----------------------------------------- |
| Ativar HTTPS                          | Criptografia do tráfego                   |
| Redirecionar HTTP → HTTPS             | Evita conexões inseguras                  |
| Verificar $\_SERVER\['HTTPS'\] no PHP | Bloqueia requisições inseguras            |
| Usar HSTS                             | Garante que navegadores sempre usem HTTPS |
| Nunca aceite tokens por HTTP          | Evita roubo de autenticação               |

# **🧾 4. Controle de Erros e Logs Seguros em PHP + MySQL**

## **🎯 Objetivo**

- **Proteger informações internas** da aplicação.

- \*\*Evitar vazamentos de exceções, SQLs ou estruturas internas.

  > \*\*

- **Manter rastreabilidade** segura dos eventos importantes no sistema.

## **⚠️ Por que isso é importante?**

Se um erro não for tratado corretamente, o PHP pode exibir:

- Stack traces

- Nomes de arquivos e classes

- Queries SQL internas

- Variáveis de ambiente

> Um atacante pode usar isso para **mapear o sistema e preparar invasões.**

## **💣 Exemplo de risco (sem proteção)**

php

\<?php

$pdo = new PDO('mysql:host=localhost;dbname=api', 'user', 'senha');

$stmt = $pdo-\>query("SELECT \* FROM usuarios WHERE id = " . $\_GET\['id'\]);

### **Erro exposto:**

nginx

Fatal error: Uncaught PDOException: SQLSTATE\[42000\]: Syntax error

## **✅ Boas práticas em PHP**

### **🔧 1. Desative exibição de erros em produção**

No php.ini:

ini

display_errors = Off

log_errors = On

error_log = /var/log/php_errors.log

Ou diretamente no seu código:

php

ini_set('display_errors', 0);

ini_set('log_errors', 1);

ini_set('error_log', \_\_DIR\_\_ . '/logs/php_errors.log');

> Em ambiente de produção, **nunca mostre mensagens de erro para o cliente.**

### **🧾 2. Retorne erros genéricos para a API**

#### **✅ Exemplo correto:**

php

\<?php

try {

$pdo = conectarBD();

// código seguro...

} catch (PDOException $e) {

http_response_code(500);

echo json_encode(\['erro' =\> 'Erro interno no servidor'\]);

error_log("\[DB ERROR\] " . $e-\>getMessage());

}

- O cliente recebe:

json

{"erro": "Erro interno no servidor"}

- O log armazena a exceção com mais detalhes **somente no servidor**.

### **🔐 3. Nunca logar dados sensíveis**

Evite logar:

- Senhas

- Tokens de autenticação

- Dados bancários, CPF, etc.

#### **❌ Errado:**

php

error_log("Login falhou para {$email} com senha {$senha}");

#### **✅ Certo:**

php

error_log("Login falhou para o e-mail {$email}");

### **📥 4. Logs seguros e organizados**

- Use arquivos dedicados, como auth.log, api.log, etc.

- Inclua:
  - IP do usuário ($\_SERVER\['REMOTE_ADDR'\])

  - Data e hora

  - Endpoint acessado ($\_SERVER\['REQUEST_URI'\])

  - ID do usuário autenticado (se disponível)

#### **Exemplo:**

php

$usuarioId = $payload\['sub'\] ?? 'não autenticado';

$ip = $\_SERVER\['REMOTE_ADDR'\] ?? 'ip desconhecido';

$rota = $\_SERVER\['REQUEST_URI'\];

$data = date('Y-m-d H:i:s');

error_log("\[$data\] \[$ip\] \[$rota\] usuário {$usuarioId} tentou acesso sem permissão", 3, \_\_DIR\_\_ . '/logs/acesso.log');

### **📊 5. Categorize os erros por tipo**

| **Tipo de erro**              | **Código HTTP** |
| ----------------------------- | --------------- |
| Requisição malformada         | 400             |
| Não autenticado               | 401             |
| Acesso negado (sem permissão) | 403             |
| Recurso não encontrado        | 404             |
| Erro interno do servidor      | 500             |

#### **Exemplo:**

php

http_response_code(403);

echo json_encode(\['erro' =\> 'Acesso negado'\]);

## **🚫 Evite**

- Stack traces no frontend.

- Print de exceções sem try/catch.

- Uso de var_dump() ou die() em produção.

- Logs em arquivos públicos acessíveis por URL (como /logs/erros.log).

## **✅ Checklist final**

| **Item**                                  | **Status** |
| ----------------------------------------- | ---------- |
| Exibir erros desativado em produção       | ✅         |
| Logs ativados com log_errors = On         | ✅         |
| Mensagens genéricas no json_encode()      | ✅         |
| Detalhes dos erros salvos só em log local | ✅         |
| Dados sensíveis **nunca logados**         | ✅         |

# **🕵️ 5. Proteção Contra Ataques Comuns em PHP**

## **🔥 1. SQL Injection**

### **❗O que é**

Manipulação de queries SQL via campos como id, email, etc.

### **❌ Exemplo inseguro:**

php

$id = $\_GET\['id'\];

$resultado = $pdo-\>query("SELECT \* FROM usuarios WHERE id = $id");

### **✅ Solução: Use prepared statements**

php

$stmt = $pdo-\>prepare("SELECT \* FROM usuarios WHERE id = ?");

$stmt-\>execute(\[$id\]);

> Nunca **concatene strings** em queries SQL — **sempre use prepare() com placeholders**.

## **💉 2. XSS (Cross-Site Scripting)**

### **❗O que é**

Entrada maliciosa armazenada no banco ou enviada na resposta, que executa **scripts no navegador da vítima**.

### **Exemplo de entrada:**

html

\<script\>alert('pwned')\</script\>

### **✅ Solução: Escape/limpe saída com htmlspecialchars()**

#### **Ao armazenar ou exibir dados em HTML:**

php

$nomeSeguro = htmlspecialchars($usuario\['nome'\], ENT_QUOTES, 'UTF-8');

echo "\<h1\>$nomeSeguro\</h1\>";

> Mesmo que sua API seja apenas backend, **proteja os dados** antes de entregar para UIs (React, Vue, etc.).

## **🔁 3. CSRF (Cross-Site Request Forgery)**

### **❗O que é**

Ataques via navegação automática que exploram **cookies persistentes** para enganar o usuário e forçar ações (ex: deletar conta).

### **Situação: mais comum em apps que usam cookies**

### **✅ Proteção em APIs REST:**

- Se usar **JWT no header**, **não precisa** de CSRF token.

- Se usar **cookies**, use:
  - SameSite=Strict ou Lax

  - HttpOnly e Secure

#### **Exemplo de configuração em setcookie:**

php

setcookie('token', $jwt, \[

'expires' =\> time() + 3600,

'path' =\> '/',

'secure' =\> true,

'httponly' =\> true,

'samesite' =\> 'Strict'

\]);

> Para APIs REST, **evite autenticação via cookies** sempre que possível — prefira Authorization: Bearer.

## **🚫 4. Brute Force (força bruta)**

### **❗O que é**

Tentativas repetidas de login com várias senhas até acertar.

### **✅ Soluções:**

#### **A. Limite tentativas por IP/usuário**

php

$tentativas = contarTentativasRecentes($email, $ip);

if ($tentativas \>= 5) {

http_response_code(429);

exit(json_encode(\['erro' =\> 'Muitas tentativas. Tente mais tarde.'\]));

}

#### **B. Atraso progressivo**

php

sleep(pow(2, $tentativas)); // Exponencial: 2, 4, 8, etc

#### **C. Log de falhas**

php

logarTentativaFalha($email, $\_SERVER\['REMOTE_ADDR'\]);

## **🌐 5. CORS mal configurado**

### **❗Risco**

Permitir que qualquer site consuma sua API:

http

Access-Control-Allow-Origin: \*

### **✅ Solução: defina origens específicas**

php

$origem = $\_SERVER\['HTTP_ORIGIN'\] ?? '';

if (in_array($origem, \['https://app.empresa.com', 'https://admin.empresa.com'\])) {

header("Access-Control-Allow-Origin: $origem");

header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");

header("Access-Control-Allow-Headers: Authorization, Content-Type");

}

## **📦 6. Upload de arquivos maliciosos**

### **❗Risco**

Upload de .php, .js, .exe disfarçado de imagem.

### **✅ Soluções:**

#### **A. Verifique tipo MIME real**

php

$finfo = finfo_open(FILEINFO_MIME_TYPE);

$tipo = finfo_file($finfo, $\_FILES\['arquivo'\]\['tmp_name'\]);

if (!in_array($tipo, \['image/jpeg', 'image/png', 'application/pdf'\])) {

exit(json_encode(\['erro' =\> 'Tipo de arquivo não permitido'\]));

}

#### **B. Renomeie arquivos e remova extensão original**

php

$novoNome = uniqid() . '.jpg'; // mesmo que tenha vindo como .php

#### **C. Armazene fora da pasta pública**

php

move_uploaded_file($\_FILES\['arquivo'\]\['tmp_name'\], '/uploads_privados/' . $novoNome);

## **🛑 Resumo dos principais ataques e defesas**

| **Ataque**       | **Solução em PHP**                   |
| ---------------- | ------------------------------------ |
| SQL Injection    | prepare() e execute() com PDO        |
| XSS              | htmlspecialchars() na saída          |
| CSRF             | Headers seguros (SameSite, HttpOnly) |
| Brute Force      | Limite de tentativas + delay         |
| CORS             | Verificar HTTP_ORIGIN                |
| Upload malicioso | Verificação MIME + pasta segura      |

# **🧮 6. Criptografia e Armazenamento Seguro**

> Implementado em PHP + MySQL, com **exemplos práticos** para proteger senhas, dados sensíveis e arquivos.

## **🎯 Objetivo**

- **Proteger dados em repouso**: tudo que está salvo no banco ou no sistema de arquivos.

- Garantir que **mesmo que haja vazamento**, os dados sejam **inúteis sem a chave**.

## **📦 Tipos de proteção**

| **Tipo de dado** | **Proteção recomendada**                    |
| ---------------- | ------------------------------------------- |
| Senhas           | Hash com salt (bcrypt, Argon2)              |
| Dados pessoais   | Criptografia simétrica (AES)                |
| Tokens / chaves  | Criptografia com tempo de expiração         |
| Arquivos         | Restrição de acesso + criptografia opcional |

## **🔐 1. Armazenamento seguro de senhas (com hash)**

### **❗Nunca armazene senha como texto puro ou criptografada de forma reversível.**

### **✅ Use password_hash() e password_verify()**

php

$senha = $\_POST\['senha'\];

$hash = password_hash($senha, PASSWORD_DEFAULT);

// Armazene $hash no banco

#### **Para verificar:**

php

$senha = $\_POST\['senha'\];

$hashDoBanco = $usuario\['senha_hash'\];

if (password_verify($senha, $hashDoBanco)) {

echo 'Login OK';

}

> **Seguro, salgado e lento de propósito** → protege contra ataques de força bruta.

## **🔐 2. Criptografia de dados sensíveis (ex: CPF, chave PIX)**

### **✅ Use AES-256-CBC com openssl_encrypt()**

#### **Exemplo de criptografia:**

php

$chave = 'minha-chave-256bits-super-segura'; // 32 caracteres

$iv = openssl_random_pseudo_bytes(16); // 128 bits

$cpf = '123.456.789-00';

$cifrado = openssl_encrypt($cpf, 'AES-256-CBC', $chave, 0, $iv);

// Armazene $cifrado e base64_encode($iv) no banco

#### **Descriptografar:**

php

$iv = base64_decode($registro\['iv'\]);

$cpf = openssl_decrypt($registro\['cpf_criptografado'\], 'AES-256-CBC', $chave, 0, $iv);

### **🛑 Atenção**

- Armazene o **IV separado**, em base64, para cada campo criptografado.

- Nunca reutilize o mesmo IV.

- **Nunca armazene a chave de criptografia no código-fonte**.
  - Use variáveis de ambiente (getenv('CHAVE_CRIPTO')) ou arquivos fora do webroot.

## **🔑 3. Proteção de tokens, UUIDs e identificadores**

Geração:

php

$token = bin2hex(random_bytes(32)); // 64 chars

-

- Para tokens JWT:
  - Assine com HMAC SHA256 (como já vimos).

  - Armazene apenas se forem revogáveis (refresh tokens, blacklist, etc).

## **🧾 4. Proteção de arquivos enviados**

- Armazene **fora do /public ou /htdocs**.

Gere nomes únicos (evita acesso direto):

php

$nomeSeg = hash('sha256', uniqid('', true)) . '.pdf';

move_uploaded_file($\_FILES\['doc'\]\['tmp_name'\], \_\_DIR\_\_ . '/uploads_privados/' . $nomeSeg);

-

Se for necessário armazenar criptografado:

php

file_put_contents('/uploads_privados/' . $nomeSeg, openssl_encrypt(file_get_contents($\_FILES\['doc'\]\['tmp_name'\]), 'AES-256-CBC', $chave, 0, $iv));

-

## **🔍 5. Proteção no banco de dados (MySQL)**

- Use tipos adequados:
  - VARCHAR(255) para hashes

  - TEXT para dados criptografados

- Não use colunas de texto puro para senhas, chaves, CPF, etc.

- Configure o MySQL para usar disco criptografado se possível (InnoDB Encryption).

## **✅ Checklist de criptografia segura**

| **Item**                        | **Está certo?** |
| ------------------------------- | --------------- |
| Senhas com password_hash()      | ✅              |
| Nada de MD5 ou SHA1 para senhas | ✅              |
| AES-256 para dados sensíveis    | ✅              |
| IV aleatório para cada operação | ✅              |
| Chave fora do repositório       | ✅              |
| Arquivos fora do webroot        | ✅              |

## **🧨 O que não fazer**

- ❌ md5($senha)

- ❌ base64_encode() como se fosse criptografia

- ❌ Guardar tokens em arquivos públicos

- ❌ Armazenar senhas criptografadas de forma reversível

# **🗂️ 7. Versionamento e Controle de Recursos em APIs REST com PHP**

## **🎯 Objetivo**

- **Evitar quebra de clientes existentes** quando a API evolui.

- **Organizar recursos REST** de forma clara, previsível e segura.

- Controlar exatamente o que é exposto — e como.

## **📦 1. Versionamento da API**

### **❗Por que é necessário?**

Imagine que sua API POST /usuarios muda de comportamento ou estrutura de resposta.  
Clientes antigos que consomem a versão anterior **quebram** se não houver controle de versão.

### **✅ Prática recomendada: versão na URL**

http

GET /api/v1/usuarios

POST /api/v1/autenticacao

GET /api/v2/relatorios

- Fácil de entender, documentar e manter.

- Permite manter **múltiplas versões ativas**.

### **📁 Estrutura de diretórios com PHP**

bash

/api

├── v1

│ ├── usuarios.php

│ └── login.php

└── v2

└── relatorios.php

### **✨ Roteador exemplo:**

php

$uri = $\_SERVER\['REQUEST_URI'\];

if (strpos($uri, '/api/v1/') === 0) {

require \_\_DIR\_\_ . '/v1' . str_replace('/api/v1', '', $uri);

} elseif (strpos($uri, '/api/v2/') === 0) {

require \_\_DIR\_\_ . '/v2' . str_replace('/api/v2', '', $uri);

} else {

http_response_code(404);

echo json_encode(\['erro' =\> 'Endpoint não encontrado'\]);

}

### **🔁 Quando criar nova versão?**

- Mudanças **quebram compatibilidade**:
  - Campos renomeados ou removidos.

  - Semântica da resposta alterada.

  - Regras de negócios diferentes.

> Se só **adicionar campos** ou **melhorar performance**, não é necessário mudar a versão.

## **🧾 2. Controle de Recursos RESTful**

### **✅ Estruture seus endpoints de forma previsível e segura:**

http

GET /api/v1/usuarios → lista usuários

GET /api/v1/usuarios/10 → dados do usuário 10

POST /api/v1/usuarios → criar usuário

PUT /api/v1/usuarios/10 → atualizar usuário 10

DELETE /api/v1/usuarios/10 → deletar usuário 10

## **🔐 3. Limite o que sua API retorna**

### **❗Risco: retornar dados sensíveis por engano**

php

// NÃO FAÇA

echo json_encode($usuario); // contém senha_hash, etc.

### **✅ Faça manualmente ou com DTOs:**

php

echo json_encode(\[

'id' =\> $usuario\['id'\],

'nome' =\> $usuario\['nome'\],

'email' =\> $usuario\['email'\]

\]);

## **📑 4. Paginação e filtros**

- **Nunca retorne todos os dados de uma vez**, especialmente listas grandes.

http

GET /api/v1/usuarios?page=2&limit=10

### **Exemplo:**

php

$page = max(1, (int)($\_GET\['page'\] ?? 1));

$limit = min(100, (int)($\_GET\['limit'\] ?? 10));

$offset = ($page - 1) \* $limit;

$stmt = $pdo-\>prepare("SELECT \* FROM usuarios LIMIT ? OFFSET ?");

$stmt-\>execute(\[$limit, $offset\]);

## **🧼 5. Evite recursos ambíguos ou perigosos**

### **❌ Não use:**

http

/api/runComando?sql=SELECT+\*+FROM+usuarios

/api/processarTudo

Esses endpoints são **inseguros, imprevisíveis e difíceis de manter**.

## **📋 6. Checklist de boas práticas**

| **Prática**                            | **Status** |
| -------------------------------------- | ---------- |
| Versionamento por URL (/api/v1/...)    | ✅         |
| Recursos RESTful (GET/POST/PUT/DELETE) | ✅         |
| Campos sensíveis ocultos               | ✅         |
| Paginação, filtros e ordenação         | ✅         |
| Documentação separada por versão       | ✅         |
| Nenhum endpoint genérico (ex: /exec)   | ✅         |

## **✅ Resumo**

- Mantenha **versões claras e separadas** de sua API.

- Estruture seus endpoints de forma **RESTful e previsível**.

- **Controle o que retorna** — evite vazar senhas, tokens, ou campos internos.

- Implemente **paginação e filtros** para não sobrecarregar o sistema.

# **🧰 8. Headers de Segurança em APIs REST com PHP**

## **🎯 Objetivo**

Aplicar cabeçalhos HTTP que:

- \*\*Impedem execução de código malicioso

  > \*\*

- \*\*Evitam que a API seja embutida em iframes

  > \*\*

- \*\*Bloqueiam adivinhação de tipo de conteúdo

  > \*\*

- \*\*Limitam compartilhamento entre domínios
  > \*\*

Esses headers **não substituem outras camadas de segurança**, mas são **defesas adicionais fundamentais**.

## **🔐 Como aplicar os headers em PHP**

Você pode aplicar headers:

- Diretamente nos arquivos .php com header()

- Centralmente via .htaccess (Apache) ou nginx.conf

## **✅ 1. X-Content-Type-Options: nosniff**

### **🔒 O que faz:**

Evita que navegadores **tentem adivinhar o tipo de conteúdo** (sniffing).

### **📦 PHP:**

php

header("X-Content-Type-Options: nosniff");

## **✅ 2. X-Frame-Options: DENY**

### **🔒 O que faz:**

Impede que sua API (ou painel) seja **embutido em iframes**, protegendo contra **clickjacking**.

### **📦 PHP:**

php

header("X-Frame-Options: DENY"); // ou SAMEORIGIN

## **✅ 3. Strict-Transport-Security**

### **🔒 O que faz:**

Informa ao navegador que o site **só pode ser acessado por HTTPS**.

### **📦 PHP:**

php

header("Strict-Transport-Security: max-age=31536000; includeSubDomains");

> Só use se **HTTPS já estiver 100% ativado**, ou usuários ficarão sem acesso.

## **✅ 4. Content-Security-Policy (opcional para APIs REST)**

### **🔒 O que faz:**

Limita quais recursos (scripts, imagens, etc.) podem ser carregados.

### **Em APIs REST, o uso é mínimo, mas essencial se você expõe dashboards, Swagger UI, etc.**

### **📦 PHP:**

php

header("Content-Security-Policy: default-src 'none'; frame-ancestors 'none';");

## **✅ 5. Referrer-Policy**

### **🔒 O que faz:**

Controla o que o navegador envia no cabeçalho Referer.

### **📦 PHP:**

php

header("Referrer-Policy: no-referrer");

## **✅ 6. Access-Control-Allow-Origin (CORS)**

### **🔒 O que faz:**

Controla **quais domínios podem acessar sua API** via browser (cross-origin).

### **📦 Exemplo seguro:**

php

$origensPermitidas = \['https://app.meusistema.com'\];

$origem = $\_SERVER\['HTTP_ORIGIN'\] ?? '';

if (in_array($origem, $origensPermitidas)) {

header("Access-Control-Allow-Origin: $origem");

header("Access-Control-Allow-Headers: Authorization, Content-Type");

header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS");

header("Access-Control-Allow-Credentials: true");

}

## **📁 Exemplo completo de headers em PHP**

php

\<?php

header("Content-Type: application/json");

header("X-Content-Type-Options: nosniff");

header("X-Frame-Options: DENY");

header("Strict-Transport-Security: max-age=31536000; includeSubDomains");

header("Referrer-Policy: no-referrer");

header("Access-Control-Allow-Origin: https://app.meusistema.com");

header("Access-Control-Allow-Headers: Authorization, Content-Type");

header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");

## **🌐 Como aplicar via .htaccess (Apache)**

apache

Header always set X-Content-Type-Options "nosniff"

Header always set X-Frame-Options "DENY"

Header always set Referrer-Policy "no-referrer"

Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"

## **✅ Checklist dos headers essenciais**

| **Header**                  | **Proteção**                               | **Aplicar?** |
| --------------------------- | ------------------------------------------ | ------------ |
| X-Content-Type-Options      | Bloqueia sniffing de conteúdo              | ✅           |
| X-Frame-Options             | Impede clickjacking                        | ✅           |
| Strict-Transport-Security   | Força uso de HTTPS                         | ✅           |
| Referrer-Policy             | Evita vazamento de URLs                    | ✅           |
| Content-Security-Policy     | Bloqueia carregamento de recursos externos | ⚠️ se UI     |
| Access-Control-Allow-Origin | Controla consumo da API                    | ✅           |

# **🧪 9. Testes e Monitoramento de Segurança em APIs REST com PHP**

## **🎯 Objetivo**

- **Identificar vulnerabilidades** antes que sejam exploradas.

- **Rastrear atividades suspeitas** em tempo real.

- Garantir que a API continue segura \*\*após atualizações, integrações ou mudanças de comportamento.
  > \*\*

## **🔍 1. Testes de Segurança**

### **✅ A. Testes Automatizados (SAST e DAST)**

#### **SAST – Static Application Security Testing (Código-fonte)**

Ferramentas que analisam seu código PHP e apontam falhas como:

- SQL injection por mysqli_query

- Uso inseguro de eval(), $\_GET, $\_POST, etc.

**Ferramentas recomendadas:**

| **Ferramenta** | **Tipo** | **Descrição**                                |
| -------------- | -------- | -------------------------------------------- |
| **Psalm**      | SAST     | Análise estática para PHP                    |
| **PHPStan**    | SAST     | Detecta bugs e problemas de segurança        |
| **SonarQube**  | SAST     | Auditoria de segurança e qualidade de código |
| **Semgrep**    | SAST     | Detecta padrões de código perigosos          |

#### **DAST – Dynamic Application Security Testing (Aplicação rodando)**

**Simula ataques reais** contra sua API (sem ver o código):

| **Ferramenta**               | **Tipo** | **Descrição**                              |
| ---------------------------- | -------- | ------------------------------------------ |
| **OWASP ZAP**                | DAST     | Teste automático contra XSS, SQLi, etc.    |
| **Burp Suite**               | DAST     | Intercepta requisições para testes manuais |
| **Postman Security Scanner** | DAST     | Testes básicos de segurança                |

### **✅ B. Testes manuais (Pentest)**

- Simula um invasor real.

- Usa ataques como:
  - Injeção de comandos SQL

  - Escalada de privilégios

  - Bypass de autenticação

  - Descoberta de endpoints ocultos

> Pode ser feito internamente ou por **consultores de segurança especializados**.

## **📈 2. Monitoramento e Logs Seguros**

### **✅ A. Logging de eventos de segurança**

**Eventos que devem ser logados:**

- Tentativas de login (falhas e sucessos)

- Acesso negado a endpoints

- Mudança de senha

- Criação ou exclusão de dados sensíveis

- Requisições com erros 4xx/5xx

- Atividades fora do padrão (ex: muitos acessos em segundos)

#### **Exemplo em PHP:**

php

function logEvento($mensagem, $arquivo = 'logs/seguranca.log') {

$ip = $\_SERVER\['REMOTE_ADDR'\] ?? 'IP-desconhecido';

$rota = $\_SERVER\['REQUEST_URI'\];

$data = date('Y-m-d H:i:s');

file_put_contents($arquivo, "\[$data\] \[$ip\] \[$rota\] $mensagem\\n", FILE_APPEND);

}

### **✅ B. Rastreamento por usuário**

Sempre que possível, registre o ID do usuário autenticado:

php

logEvento("Usuário {$usuario\['id'\]} tentou acessar recurso não autorizado");

## **🚨 3. Alertas e Limites**

### **✅ A. Bloqueio automático por tentativas**

php

$falhas = contarFalhasRecentes($email, $ip);

if ($falhas \>= 5) {

http_response_code(429);

exit(json_encode(\['erro' =\> 'Acesso bloqueado temporariamente'\]));

}

### **✅ B. Notificações e monitoramento**

Ferramentas para isso:

| **Ferramenta**        | **Função**                      |
| --------------------- | ------------------------------- |
| Grafana + Loki        | Visualização e alerta de logs   |
| ELK Stack (Elastic)   | Centralização de logs           |
| Sentry                | Erros e exceções em tempo real  |
| UptimeRobot, Cronitor | Verifica disponibilidade da API |

## **🔁 4. Auditoria e Retenção**

- \*\*Guarde logs por no mínimo 6 meses

  > \*\*

- Aplique criptografia ou proteção de permissões (chmod)

- Evite armazenar:
  - Senhas

  - Tokens

  - Dados bancários

## **✅ Checklist de Testes e Monitoramento**

| **Item**                                   | **Status** |
| ------------------------------------------ | ---------- |
| Verificação automática de código (SAST)    | ✅         |
| Testes dinâmicos com OWASP ZAP ou Burp     | ✅         |
| Logs de login, erros, acessos negados      | ✅         |
| Identificação de usuário e IP nos logs     | ✅         |
| Limite de tentativas e bloqueio automático | ✅         |
| Armazenamento seguro e auditável de logs   | ✅         |
| Alertas visuais/sonoros (Grafana, Sentry)  | ✅         |

# **📜 10. Documentação e Segurança da Documentação**

## **🎯 Objetivo**

- Fornecer **documentação útil e acessível** para desenvolvedores legítimos.

- **Evitar exposição acidental** de informações internas ou sensíveis.

- **Proteger sua API** contra engenharia reversa e ataques automatizados.

## **⚠️ Por que isso é um risco?**

Uma documentação mal gerenciada pode:

- Expôr endpoints ocultos ou perigosos.

- Revelar dados sensíveis ou tokens de exemplo reais.

- Descrever estruturas internas (banco, queries, lógica).

- Ajudar um invasor a planejar um ataque mais eficaz.

## **🧾 1. Ferramentas comuns de documentação**

| **Ferramenta**      | **Utilidade**                     |
| ------------------- | --------------------------------- |
| Swagger/OpenAPI     | Interface interativa de endpoints |
| Postman Collections | Requisições prontas com exemplos  |
| Markdown + Git      | Documentação técnica versionada   |

## **🧰 2. Riscos mais comuns**

| **Risco**                          | **Consequência**                 |
| ---------------------------------- | -------------------------------- |
| Tokens reais nos exemplos          | Vazamento de credenciais         |
| Endpoints internos documentados    | Acesso a funções restritas       |
| Documentação pública sem controle  | Qualquer pessoa consome a API    |
| Stack traces ou erros nos exemplos | Exposição de detalhes do backend |

## **✅ 3. Boas práticas para segurança da documentação**

### **🔐 A. Proteja o acesso à documentação interativa (Swagger, Redoc, etc.)**

php

// Em um index de Swagger

session_start();

if (!isset($\_SESSION\['usuario_autenticado'\]) \|\| $\_SESSION\['papel'\] !== 'admin') {

http_response_code(403);

exit('Acesso restrito');

}

> Ou proteja com **autenticação HTTP básica**, se for ambiente controlado.

### **🧼 B. Nunca use dados reais nos exemplos**

#### **❌ Errado:**

json

{

"email": "admin@empresa.com",

"token": "eyJhbGciOiJIUzI1NiJ9..."

}

#### **✅ Certo:**

json

{

"email": "usuario@example.com",

"token": "Bearer {seu_token_aqui}"

}

### **📁 C. Separe documentação por ambiente e versão**

- Documente /api/v1, /api/v2 separadamente.

- Não exponha:
  - Endpoints internos (ex: /admin/exec)

  - Estruturas do banco de dados

  - Diretórios ou rotas de debug

### **🔏 D. Documente permissões e escopos**

Para cada endpoint, informe claramente:

| **Endpoint** | **Método** | **Requer token?** | **Papel necessário** |
| ------------ | ---------- | ----------------- | -------------------- |
| /usuarios    | GET        | Sim               | admin                |
| /relatorios  | POST       | Sim               | usuario              |
| /login       | POST       | Não               | N/A                  |

### **🛑 E. Nunca expor documentação com ambiente ativo sem autenticação**

Se você está usando Swagger em produção:

- Proteja com token.

- Ou desative em produção:

php

if ($ambiente === 'producao') {

exit('Documentação desabilitada.');

}

## **🧰 4. Postman – boas práticas**

- Use **variáveis de ambiente** ({{url}}, {{token}}).

- Evite salvar tokens de produção em coleções.

- Compartilhe apenas coleções **filtradas**, com os endpoints públicos e controlados.

## **📋 5. Checklist de segurança da documentação**

| **Item**                                         | **Status** |
| ------------------------------------------------ | ---------- |
| Sem tokens reais                                 | ✅         |
| Sem exposição de endpoints internos              | ✅         |
| Swagger / Redoc protegido com login ou bloqueado | ✅         |
| Versão da API documentada                        | ✅         |
| Campos e permissões bem definidos                | ✅         |
| Documentação não pública (se for API privada)    | ✅         |

## **✅ Resumo**

- Documentar é essencial, mas deve ser feito com **cuidado extremo**.

- **Proteger a documentação** é tão importante quanto proteger a API.

- O que estiver documentado está **potencialmente visível para um atacante**.
