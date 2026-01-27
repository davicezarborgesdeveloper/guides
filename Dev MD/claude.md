# Sistema ERP/CRM - Diretrizes de Desenvolvimento

## Visão Geral do Sistema

Sistema ERP/CRM completo com:

- **Backend**: Golang

- **Frontend**: Flutter (Portal Web + Aplicativo Mobile)

- **Arquitetura**: Clean Architecture

- **Paradigma**: Offline-First com Desenvolvimento Otimista

---

## ÍNDICE

1. [Backend (Golang)](#1-backend-golang)

2. [Frontend (Flutter)](#2-frontend-flutter)

3. [Testes](#3-testes)

4. [Segurança](#4-segurança)

5. [Implantação](#5-implantação)

6. [Checklist de Implementação](#6-checklist-de-implementação)

---

## 1. BACKEND (Golang)

### 1.1 Estrutura de Pastas - Clean Architecture

```

backend/

├── cmd/

│ └── server/

│ └── main.go # Entry point

├── internal/

│ ├── domain/ # Entities

│ │ ├── entities/

│ │ ├── valueobjects/

│ │ └── errors/

│ ├── usecases/ # Business Logic

│ │ ├── interfaces/

│ │ ├── user/

│ │ ├── customer/

│ │ ├── product/

│ │ └── order/

│ ├── adapters/ # Interface Adapters

│ │ ├── http/

│ │ │ ├── handlers/

│ │ │ ├── middleware/

│ │ │ ├── dto/

│ │ │ └── router.go

│ │ └── websocket/

│ └── infrastructure/ # Frameworks & Drivers

│ ├── database/

│ ├── cache/

│ ├── security/

│ ├── logging/

│ └── storage/

├── pkg/ # Código compartilhado

├── tests/

│ ├── unit/

│ ├── integration/

│ └── e2e/

├── go.mod

└── README.md

```

### 1.2 Princípios SOLID

#### 1.2.1 Single Responsibility Principle (SRP)

```go

// ✅ CORRETO: Cada struct tem uma responsabilidade



// domain/entities/user.go

type  User  struct {

ID  string

Email  string

Password  string

Name  string

CreatedAt  time.Time

}



// usecases/user/create_user.go

type  CreateUserUseCase  struct {

userRepo  interfaces.UserRepository

encryptionSvc  interfaces.EncryptionService

}



func (uc *CreateUserUseCase) Execute(ctx  context.Context, input  CreateUserInput) (*User, error) {

// Apenas lógica de criação de usuário

}

```

#### 1.2.2 Open/Closed Principle (OCP)

```go

// ✅ CORRETO: Aberto para extensão



// usecases/interfaces/notification_service.go

type  NotificationService  interface {

Send(ctx  context.Context, message  Message) error

}



// Novas implementações sem modificar código existente

type  EmailNotification  struct{}

type  SMSNotification  struct{}

type  PushNotification  struct{}

```

#### 1.2.3 Liskov Substitution Principle (LSP)

```go

// ✅ CORRETO: Implementações substituíveis



type  Repository[T  any] interface {

Create(ctx  context.Context, entity  T) error

FindByID(ctx  context.Context, id  string) (*T, error)

}



type  PostgresRepository[T  any] struct{}

type  MongoRepository[T  any] struct{}

// Ambos implementam completamente a interface

```

#### 1.2.4 Interface Segregation Principle (ISP)

```go

// ✅ CORRETO: Interfaces segregadas



type  UserReader  interface {

FindByID(ctx  context.Context, id  string) (*User, error)

}



type  UserWriter  interface {

Create(ctx  context.Context, user *User) error

}



type  UserDeleter  interface {

Delete(ctx  context.Context, id  string) error

}

```

#### 1.2.5 Dependency Inversion Principle (DIP)

```go

// ✅ CORRETO: Dependa de abstrações



type  CreateUserUseCase  struct {

// Interfaces (abstrações)

userRepo  interfaces.UserRepository

encryptionSvc  interfaces.EncryptionService

}



// main.go - Injeção de dependências

func  main() {

db := postgres.NewConnection()

encryptor := encryption.NewAES(key)

userRepo := postgres.NewUserRepository(db)

createUserUC := user.NewCreateUserUseCase(userRepo, encryptor)

}

```

### 1.3 Segurança - Backend

#### 1.3.1 Criptografia

```go

// infrastructure/security/encryption/aes_encryption.go

type  AESEncryption  struct {

key []byte

}



func (e *AESEncryption) Encrypt(plaintext  string) (string, error) {

block, _ := aes.NewCipher(e.key)

gcm, _ := cipher.NewGCM(block)

nonce := make([]byte, gcm.NonceSize())

io.ReadFull(rand.Reader, nonce)

ciphertext := gcm.Seal(nonce, nonce, []byte(plaintext), nil)

return  base64.StdEncoding.EncodeToString(ciphertext), nil

}



// Hash de senhas com bcrypt

func  HashPassword(password  string) (string, error) {

bytes, err := bcrypt.GenerateFromPassword([]byte(password), 14)

return  string(bytes), err

}

```

#### 1.3.2 Prevenção SQL Injection

```go

// ✅ CORRETO: Prepared statements

func (r *UserRepository) FindByEmail(ctx  context.Context, email  string) (*User, error) {

query := `SELECT id, email, name FROM users WHERE email = $1`

var  user  User

err := r.db.QueryRowContext(ctx, query, email).Scan(&user.ID, &user.Email, &user.Name)

return &user, err

}



// ✅ CORRETO: Whitelist de campos

func (r *UserRepository) Search(filters  SearchFilters) ([]User, error) {

allowedSortFields := map[string]bool{"name": true, "email": true}

sortField := "created_at"

if  allowedSortFields[filters.SortBy] {

sortField = filters.SortBy

}

query := fmt.Sprintf(`SELECT * FROM users ORDER BY %s LIMIT $1`, sortField)

// ...

}

```

#### 1.3.3 Validação de Inputs

```go

// infrastructure/security/sanitization/input_validator.go

type  InputValidator  struct{}



func (v *InputValidator) SanitizeString(input  string) string {

input = strings.ReplaceAll(input, "\x00", "")

input = strings.Map(func(r  rune) rune {

if  unicode.IsControl(r) && r != '\n' && r != '\r' {

return -1

}

return  r

}, input)

return  strings.TrimSpace(input)

}



func (v *InputValidator) ValidateEmail(email  string) bool {

regex := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)

return  regex.MatchString(email)

}



func (v *InputValidator) ValidateCPF(cpf  string) bool {

cpf = regexp.MustCompile(`[^\d]`).ReplaceAllString(cpf, "")

if  len(cpf) != 11 { return  false }

// Validação completa dos dígitos verificadores

return  validateCPFDigits(cpf)

}

```

#### 1.3.4 LGPD - Gestão de Consentimento

```go

// infrastructure/security/lgpd/consent_manager.go

type  ConsentType  string



const (

ConsentDataProcessing  ConsentType = "data_processing"

ConsentMarketing  ConsentType = "marketing"

ConsentDataSharing  ConsentType = "data_sharing"

)



type  Consent  struct {

ID  string

UserID  string

Type  ConsentType

Granted  bool

GrantedAt *time.Time

RevokedAt *time.Time

IPAddress  string

Version  string

}



type  ConsentManager  struct {

repo  ConsentRepository

audit  AuditLogger

}



func (cm *ConsentManager) GrantConsent(ctx  context.Context, userID  string, consentType  ConsentType) error {

consent := &Consent{

UserID: userID,

Type: consentType,

Granted: true,

GrantedAt: timeNow(),

}

cm.repo.Save(ctx, consent)

cm.audit.Log(ctx, "consent_granted", userID)

return  nil

}



func (cm *ConsentManager) RevokeConsent(ctx  context.Context, userID  string, consentType  ConsentType) error {

// Revoga consentimento e registra auditoria

}

```

#### 1.3.5 Rate Limiting

```go

// adapters/http/middleware/rate_limiter.go

type  RateLimiter  struct {

limiters  map[string]*rate.Limiter

mu  sync.RWMutex

rate  rate.Limit

burst  int

}



func (rl *RateLimiter) Middleware(next  http.Handler) http.Handler {

return  http.HandlerFunc(func(w  http.ResponseWriter, r *http.Request) {

ip := getIP(r)

limiter := rl.getLimiter(ip)

if !limiter.Allow() {

http.Error(w, "Rate limit exceeded", 429)

return

}

next.ServeHTTP(w, r)

})

}

```

#### 1.3.6 Security Headers

```go

// adapters/http/middleware/security_headers.go

func  SecurityHeaders(next  http.Handler) http.Handler {

return  http.HandlerFunc(func(w  http.ResponseWriter, r *http.Request) {

w.Header().Set("X-Content-Type-Options", "nosniff")

w.Header().Set("X-Frame-Options", "DENY")

w.Header().Set("X-XSS-Protection", "1; mode=block")

w.Header().Set("Content-Security-Policy", "default-src 'self'")

w.Header().Set("Strict-Transport-Security", "max-age=31536000")

next.ServeHTTP(w, r)

})

}

```

#### 1.3.7 JWT Seguro

```go

// infrastructure/security/jwt/jwt_service.go

type  JWTService  struct {

accessSecret []byte

refreshSecret []byte

accessTTL  time.Duration

refreshTTL  time.Duration

}



func (s *JWTService) GenerateAccessToken(userID  string, roles []string) (string, error) {

claims := Claims{

UserID: userID,

Roles: roles,

RegisteredClaims: jwt.RegisteredClaims{

ExpiresAt: jwt.NewNumericDate(time.Now().Add(15 * time.Minute)),

IssuedAt: jwt.NewNumericDate(time.Now()),

},

}

token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)

return  token.SignedString(s.accessSecret)

}

```

### 1.4 Testes - Backend

#### 1.4.1 Testes de Unidade

```go

// tests/unit/usecases/user_usecase_test.go

func  TestCreateUserUseCase_Execute(t *testing.T) {

// Arrange

mockRepo := new(MockUserRepository)

mockEncryption := new(MockEncryptionService)

useCase := user.NewCreateUserUseCase(mockRepo, mockEncryption)

ctx := context.Background()

input := user.CreateUserInput{

Email: "test@example.com",

Password: "SecurePass123!",

}

mockRepo.On("FindByEmail", ctx, input.Email).Return(nil, errors.New("not found"))

mockEncryption.On("HashPassword", input.Password).Return("hashed", nil)

mockRepo.On("Create", ctx, mock.Anything).Return(nil)

// Act

result, err := useCase.Execute(ctx, input)

// Assert

assert.NoError(t, err)

assert.NotNil(t, result)

mockRepo.AssertExpectations(t)

}

```

#### 1.4.2 Testes de Integração

```go

// tests/integration/api/user_api_test.go

func  TestCreateUserAPI(t *testing.T) {

app := setupTestApp()

payload := map[string]string{

"email": "api@example.com",

"password": "SecurePass123!",

}

body, _ := json.Marshal(payload)

req := httptest.NewRequest("POST", "/api/v1/users", bytes.NewBuffer(body))

w := httptest.NewRecorder()

app.ServeHTTP(w, req)

assert.Equal(t, http.StatusCreated, w.Code)

}

```

---

## 2. FRONTEND (Flutter)

### 2.1 Estrutura de Pastas - Clean Architecture

```

frontend/

├── lib/

│ ├── main.dart

│ ├── app.dart

│ ├── core/

│ │ ├── constants/

│ │ ├── theme/

│ │ ├── errors/

│ │ ├── network/

│ │ ├── utils/

│ │ └── security/

│ ├── features/

│ │ ├── auth/

│ │ │ ├── domain/

│ │ │ │ ├── entities/

│ │ │ │ ├── repositories/

│ │ │ │ └── usecases/

│ │ │ ├── data/

│ │ │ │ ├── models/

│ │ │ │ ├── datasources/

│ │ │ │ └── repositories/

│ │ │ └── presentation/

│ │ │ ├── providers/

│ │ │ ├── pages/

│ │ │ └── widgets/

│ │ ├── customers/

│ │ ├── products/

│ │ └── orders/

│ └── shared/

│ ├── widgets/

│ └── providers/

├── test/

│ ├── unit/

│ ├── widget/

│ ├── integration/

│ └── golden/

└── pubspec.yaml

```

### 2.2 Provider para Estado (SEM Streams, SEM setState)

```dart

// presentation/providers/customer_list_provider.dart

class  CustomerListProvider  extends  ChangeNotifier {

final  GetCustomersUseCase getCustomersUseCase;

final  CreateCustomerUseCase createCustomerUseCase;

final  DeleteCustomerUseCase deleteCustomerUseCase;



CustomerListProvider({

required  this.getCustomersUseCase,

required  this.createCustomerUseCase,

required  this.deleteCustomerUseCase,

});



List<CustomerEntity> _customers = [];

bool _isLoading = false;

Failure? _failure;



List<CustomerEntity> get customers => _customers;

bool  get isLoading => _isLoading;

bool  get hasError => _failure != null;



Future<void> loadCustomers() async {

_isLoading = true;

notifyListeners(); // Ao invés de setState



final result = await  getCustomersUseCase();

result.fold(

(failure) {

_failure = failure;

_isLoading = false;

notifyListeners();

},

(customers) {

_customers = customers;

_isLoading = false;

notifyListeners();

},

);

}



// Desenvolvimento Otimista

Future<void> deleteCustomer(String id) async {

final index = _customers.indexWhere((c) => c.id == id);

final deletedCustomer = _customers.removeAt(index);

notifyListeners(); // Atualiza UI imediatamente



final result = await  deleteCustomerUseCase(id);

result.fold(

(failure) {

// Rollback em caso de erro

_customers.insert(index, deletedCustomer);

_failure = failure;

notifyListeners();

},

(_) {

// Sucesso - já foi removido

},

);

}

}



// presentation/pages/customer_list_page.dart

class  CustomerListPage  extends  StatelessWidget {

@override

Widget  build(BuildContext context) {

return  ChangeNotifierProvider(

create: (_) => GetIt.I<CustomerListProvider>()..loadCustomers(),

child: Scaffold(

body: Consumer<CustomerListProvider>(

builder: (context, provider, child) {

if (provider.isLoading) {

return  CircularProgressIndicator();

}

return  ListView.builder(

itemCount: provider.customers.length,

itemBuilder: (context, index) {

return  CustomerCard(customer: provider.customers[index]);

},

);

},

),

),

);

}

}

```

### 2.3 Offline-First com Desenvolvimento Otimista

```dart

// data/repositories/customer_repository_impl.dart

class  CustomerRepositoryImpl  implements  CustomerRepository {

final  CustomerRemoteDataSource remoteDataSource;

final  CustomerLocalDataSource localDataSource;

final  NetworkInfo networkInfo;

final  SyncQueue syncQueue;



@override

Future<Either<Failure, List<CustomerEntity>>> getCustomers() async {

try {

// SEMPRE retorna dados locais primeiro

final cachedCustomers = await localDataSource.getCachedCustomers();

final entities = cachedCustomers.map((m) => m.toEntity()).toList();



// Sincroniza em background se online

if (await networkInfo.isConnected) {

_syncInBackground();

}



return  Right(entities);

} catch (e) {

return  Left(CacheFailure());

}

}



@override

Future<Either<Failure, CustomerEntity>> createCustomer(params) async {

try {

// 1. ID temporário

final tempId = 'temp_${DateTime.now().millisecondsSinceEpoch}';

final tempCustomer = CustomerModel.fromParams(params, tempId);



// 2. Salva localmente IMEDIATAMENTE

await localDataSource.cacheCustomer(tempCustomer);



// 3. Adiciona à fila de sincronização

await syncQueue.enqueue(SyncOperation(

type: SyncOperationType.create,

entity: 'customer',

tempId: tempId,

data: tempCustomer.toJson(),

));



// 4. Retorna entity (usuário já vê)

final entity = tempCustomer.toEntity();



// 5. Sincroniza em background

if (await networkInfo.isConnected) {

_syncCreateInBackground(tempId, params);

}



return  Right(entity);

} catch (e) {

return  Left(CacheFailure());

}

}



Future<void> _syncCreateInBackground(tempId, params) async {

try {

final created = await remoteDataSource.createCustomer(params);

await localDataSource.replaceTempCustomer(tempId, created);

await syncQueue.removeByTempId(tempId);

} catch (e) {

// Mantém na fila para retry

}

}

}



// core/sync/sync_queue.dart

class  SyncQueue {

final  SyncQueueDataSource dataSource;



Future<void> processPendingOperations() async {

final operations = await dataSource.getAll();

for (final operation in operations) {

try {

await  _processOperation(operation);

await dataSource.remove(operation.id);

} catch (e) {

await dataSource.incrementRetryCount(operation.id);

}

}

}

}

```

### 2.4 Conceito de Módulos (sem modular)

```dart

// features/customers/customer_module.dart

class  CustomerModule {

static  void  register(GetIt getIt) {

// Data sources

getIt.registerLazySingleton<CustomerRemoteDataSource>(

() => CustomerRemoteDataSourceImpl(getIt()),

);

getIt.registerLazySingleton<CustomerLocalDataSource>(

() => CustomerLocalDataSourceImpl(getIt()),

);

// Repository

getIt.registerLazySingleton<CustomerRepository>(

() => CustomerRepositoryImpl(

remoteDataSource: getIt(),

localDataSource: getIt(),

networkInfo: getIt(),

),

);

// Use cases

getIt.registerLazySingleton(() => GetCustomersUseCase(getIt()));

getIt.registerLazySingleton(() => CreateCustomerUseCase(getIt()));

// Providers

getIt.registerFactory(() => CustomerListProvider(

getCustomersUseCase: getIt(),

createCustomerUseCase: getIt(),

));

}

static  Map<String, WidgetBuilder> routes() {

return {

'/customers': (context) => CustomerListPage(),

'/customers/detail': (context) => CustomerDetailPage(),

};

}

}



// main.dart

void  main() {

final getIt = GetIt.instance;

// Registrar módulos

CoreModule.register(getIt);

AuthModule.register(getIt);

CustomerModule.register(getIt);

ProductModule.register(getIt);

runApp(MyApp());

}

```

### 2.5 Segurança - Frontend

#### 2.5.1 Armazenamento Seguro

```dart

// core/security/secure_storage.dart

class  SecureStorage {

final  FlutterSecureStorage _storage;

final  EncryptionService _encryption;



Future<void> write(String key, String value) async {

final encrypted = await _encryption.encrypt(value);

await _storage.write(key: key, value: encrypted);

}



Future<String?> read(String key) async {

final encrypted = await _storage.read(key: key);

if (encrypted == null) return  null;

return  await _encryption.decrypt(encrypted);

}



Future<void> saveAccessToken(String token) async {

await  write(StorageKeys.accessToken, token);

}



Future<String?> getAccessToken() async {

return  await  read(StorageKeys.accessToken);

}

}



// core/security/encryption_service.dart

class  EncryptionService {

final encrypt.Key _key;

final encrypt.Encrypter _encrypter;



Future<String> encrypt(String plainText) async {

final encrypted = _encrypter.encrypt(plainText, iv: _iv);

return encrypted.base64;

}



Future<String> decrypt(String encryptedText) async {

final decrypted = _encrypter.decrypt64(encryptedText, iv: _iv);

return decrypted;

}

}

```

#### 2.5.2 Validação de Inputs

```dart

// core/utils/validators.dart

class  Validators {

static  String? email(String? value) {

if (value == null || value.isEmpty) {

return  'Email é obrigatório';

}

final regex = RegExp(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$');

if (!regex.hasMatch(value)) {

return  'Email inválido';

}

return  null;

}



static  String? cpf(String? value) {

if (value == null || value.isEmpty) {

return  'CPF é obrigatório';

}

final cpfNumbers = value.replaceAll(RegExp(r'[^\d]'), '');

if (cpfNumbers.length != 11) {

return  'CPF deve ter 11 dígitos';

}

if (!_validateCPFDigits(cpfNumbers)) {

return  'CPF inválido';

}

return  null;

}



static  String? password(String? value) {

if (value == null || value.isEmpty) {

return  'Senha é obrigatória';

}

if (value.length < 8) {

return  'Senha deve ter no mínimo 8 caracteres';

}

if (!value.contains(RegExp(r'[A-Z]'))) {

return  'Senha deve conter letra maiúscula';

}

if (!value.contains(RegExp(r'[0-9]'))) {

return  'Senha deve conter número';

}

if (!value.contains(RegExp(r'[!@#$%^&*(),.?":{}|<>]'))) {

return  'Senha deve conter caractere especial';

}

return  null;

}

}

```

#### 2.5.3 LGPD - Termo de Aceite

```dart

// features/auth/presentation/providers/terms_provider.dart

class  TermsProvider  extends  ChangeNotifier {

bool _dataProcessingConsent = false;

bool _marketingConsent = false;

bool _dataSharingConsent = false;



bool  get canAccept => _dataProcessingConsent;



void  setDataProcessingConsent(bool value) {

_dataProcessingConsent = value;

notifyListeners();

}



Future<void> acceptTerms() async {

final consents = {

ConsentType.dataProcessing: _dataProcessingConsent,

ConsentType.marketing: _marketingConsent,

ConsentType.dataSharing: _dataSharingConsent,

};

await consentManager.saveConsents(consents);

}

}



// features/auth/presentation/pages/terms_page.dart

class  TermsPage  extends  StatelessWidget {

@override

Widget  build(BuildContext context) {

return  ChangeNotifierProvider(

create: (_) => TermsProvider(),

child: Scaffold(

appBar: AppBar(title: Text('Termos e Condições')),

body: Consumer<TermsProvider>(

builder: (context, provider, child) {

return  Column(

children: [

Expanded(

child: SingleChildScrollView(

child: Column(

children: [

Text('Política de Privacidade...'),

ConsentCheckbox(

title: 'Processamento de Dados',

value: provider.dataProcessingConsent,

required: true,

onChanged: provider.setDataProcessingConsent,

),

ConsentCheckbox(

title: 'Marketing',

value: provider.marketingConsent,

onChanged: provider.setMarketingConsent,

),

],

),

),

),

ElevatedButton(

onPressed: provider.canAccept

? () => provider.acceptTerms()

: null,

child: Text('Aceitar e Continuar'),

),

],

);

},

),

),

);

}

}

```

### 2.6 Testes - Frontend

#### 2.6.1 Testes de Unidade

```dart

// test/unit/domain/usecases/login_usecase_test.dart

void  main() {

late  LoginUseCase useCase;

late  MockAuthRepository mockRepo;



setUp(() {

mockRepo = MockAuthRepository();

useCase = LoginUseCase(mockRepo);

});



test('deve retornar User quando login for bem-sucedido', () async {

// Arrange

final params = LoginParams(email: 'test@test.com', password: 'pass123');

final user = UserEntity(id: '1', email: 'test@test.com');

when(() => mockRepo.login(params))

.thenAnswer((_) async => Right(user));



// Act

final result = await  useCase(params);



// Assert

expect(result, Right(user));

verify(() => mockRepo.login(params)).called(1);

});

}

```

#### 2.6.2 Testes de Widget

```dart

// test/widget/login_page_test.dart

void  main() {

testWidgets('deve exibir campos de email e senha', (tester) async {

await tester.pumpWidget(

MaterialApp(home: LoginPage()),

);



expect(find.byType(TextField), findsNWidgets(2));

expect(find.text('Email'), findsOneWidget);

expect(find.text('Senha'), findsOneWidget);

expect(find.byType(ElevatedButton), findsOneWidget);

});



testWidgets('deve mostrar erro quando campos estão vazios', (tester) async {

await tester.pumpWidget(

MaterialApp(home: LoginPage()),

);



await tester.tap(find.byType(ElevatedButton));

await tester.pump();



expect(find.text('Email é obrigatório'), findsOneWidget);

expect(find.text('Senha é obrigatória'), findsOneWidget);

});

}

```

#### 2.6.3 Testes de Integração

```dart

// test/integration/auth_flow_test.dart

void  main() {

testWidgets('fluxo completo de autenticação', (tester) async {

await tester.pumpWidget(MyApp());



// Navega para login

await tester.tap(find.text('Login'));

await tester.pumpAndSettle();



// Preenche formulário

await tester.enterText(

find.byKey(Key('emailField')),

'test@example.com',

);

await tester.enterText(

find.byKey(Key('passwordField')),

'Password123!',

);



// Submete

await tester.tap(find.text('Entrar'));

await tester.pumpAndSettle();



// Verifica navegação para home

expect(find.text('Dashboard'), findsOneWidget);

});

}

```

#### 2.6.4 Golden Tests

```dart

// test/golden/login_page_golden_test.dart

void  main() {

testWidgets('Login page golden test', (tester) async {

await tester.pumpWidget(

MaterialApp(

home: LoginPage(),

),

);



await  expectLater(

find.byType(LoginPage),

matchesGoldenFile('goldens/login_page.png'),

);

});



testWidgets('Login page com erro golden test', (tester) async {

await tester.pumpWidget(

ChangeNotifierProvider(

create: (_) => LoginProvider()..setError('Credenciais inválidas'),

child: MaterialApp(home: LoginPage()),

),

);



await tester.pump();



await  expectLater(

find.byType(LoginPage),

matchesGoldenFile('goldens/login_page_error.png'),

);

});

}

```

---

## 3. TESTES

### 3.1 Estrutura de Testes

**Backend:**

- `tests/unit/` - Testes de unidade (entidades, use cases)

- `tests/integration/` - Testes de integração (repositórios, API)

- `tests/e2e/` - Testes end-to-end (fluxos completos)

**Frontend:**

- `test/unit/` - Testes de unidade (use cases, models)

- `test/widget/` - Testes de widgets

- `test/integration/` - Testes de integração (fluxos)

- `test/golden/` - Golden tests (screenshots)

### 3.2 Cobertura de Testes

**Objetivo:** Mínimo 80% de cobertura

**Backend (Golang):**

```bash

# Executar testes com cobertura

go  test  -v  -coverprofile=coverage.out  ./...

go  tool  cover  -html=coverage.out  -o  coverage.html



# Ver cobertura no terminal

go  test  -cover  ./...

```

**Frontend (Flutter):**

```bash

# Executar todos os testes

flutter  test



# Com cobertura

flutter  test  --coverage

genhtml  coverage/lcov.info  -o  coverage/html



# Golden tests

flutter  test  --update-goldens  # Atualizar goldens

flutter  test  # Rodar comparação

```

### 3.3 CI/CD - Testes Automatizados

```yaml

# .github/workflows/test.yml

name: Tests



on: [push, pull_request]



jobs:

backend-tests:

runs-on: ubuntu-latest

steps:

- uses: actions/checkout@v3

- uses: actions/setup-go@v4

with:

go-version: '1.21'

- name: Run tests

run: |

cd backend

go test -v -race -coverprofile=coverage.out ./...

go tool cover -func=coverage.out



frontend-tests:

runs-on: ubuntu-latest

steps:

- uses: actions/checkout@v3

- uses: subosito/flutter-action@v2

with:

flutter-version: '3.16.0'

- name: Run tests

run: |

cd frontend

flutter pub get

flutter test --coverage

flutter test --update-goldens

```

---

## 4. SEGURANÇA

### 4.1 Checklist de Segurança

#### Backend

- [ ] Criptografia AES-256 para dados sensíveis

- [ ] Bcrypt para hash de senhas (cost 14)

- [ ] Prepared statements (prevenir SQL injection)

- [ ] Validação e sanitização de todos os inputs

- [ ] Rate limiting por IP

- [ ] CORS configurado corretamente

- [ ] Security headers implementados

- [ ] JWT com refresh token

- [ ] Blacklist de tokens revogados

- [ ] HTTPS obrigatório

- [ ] Logs de auditoria (LGPD)

- [ ] Gestão de consentimentos

- [ ] Anonimização de dados

#### Frontend

- [ ] Flutter Secure Storage para tokens

- [ ] Criptografia local de dados sensíveis

- [ ] Validação de inputs em todos os formulários

- [ ] Sanitização antes de enviar ao backend

- [ ] Biometria para login (opcional)

- [ ] Certificate pinning (opcional)

- [ ] Ofuscação de código no build

- [ ] Termo de aceite LGPD

- [ ] Coleta de consentimentos

- [ ] Opção de deletar conta

### 4.2 LGPD - Requisitos

#### Consentimentos Obrigatórios

1.  **Processamento de Dados** (obrigatório)

2.  **Marketing** (opcional)

3.  **Compartilhamento com Terceiros** (opcional)

4.  **Perfilamento** (opcional)

#### Direitos do Titular

- Acesso aos dados pessoais

- Correção de dados

- Anonimização

- Portabilidade

- Eliminação

- Revogação de consentimento

- Informação sobre compartilhamento

#### Implementação

```go

// Backend - Endpoints LGPD

GET /api/v1/user/data # Exportar  dados (JSON)

POST /api/v1/user/consent/grant # Conceder  consentimento

POST /api/v1/user/consent/revoke # Revogar  consentimento

GET /api/v1/user/audit-trail # Histórico  de  acessos

DELETE /api/v1/user/account # Deletar  conta (anonimizar)

```

```dart

// Frontend - Tela de Privacidade

- Ver dados coletados

- Exportar dados (JSON)

- Gerenciar consentimentos

- Ver histórico de acessos

- Deletar conta

```

---

## 5. IMPLANTAÇÃO

### 5.1 Docker

**Backend:**

```dockerfile

# Dockerfile

FROM golang:1.21-alpine AS builder

WORKDIR /app

COPY go.* ./

RUN go mod download

COPY . .

RUN CGO_ENABLED=0 GOOS=linux go build -o server cmd/server/main.go



FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /root/

COPY --from=builder /app/server .

EXPOSE 8080

CMD ["./server"]

```

**Frontend (Web):**

```dockerfile

# Dockerfile

FROM nginx:alpine

COPY build/web /usr/share/nginx/html

COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

```

### 5.2 Docker Compose

```yaml

# docker-compose.yml

version: '3.8'



services:

postgres:

image: postgres:15-alpine

environment:

POSTGRES_DB: erp_crm

POSTGRES_USER: admin

POSTGRES_PASSWORD: ${DB_PASSWORD}

volumes:

- postgres_data:/var/lib/postgresql/data

ports:

- "5432:5432"



redis:

image: redis:7-alpine

ports:

- "6379:6379"



backend:

build: ./backend

environment:

DATABASE_URL: postgres://admin:${DB_PASSWORD}@postgres:5432/erp_crm

REDIS_URL: redis://redis:6379

JWT_SECRET: ${JWT_SECRET}

ports:

- "8080:8080"

depends_on:

- postgres

- redis



frontend:

build: ./frontend

ports:

- "80:80"

depends_on:

- backend



volumes:

postgres_data:

```

### 5.3 Variáveis de Ambiente

**Backend (.env):**

```env

# Database

DATABASE_URL=postgres://user:pass@localhost:5432/erp_crm

DATABASE_MAX_CONNECTIONS=25



# Redis

REDIS_URL=redis://localhost:6379



# Security

JWT_ACCESS_SECRET=your-super-secret-access-key-min-32-chars

JWT_REFRESH_SECRET=your-super-secret-refresh-key-min-32-chars

ENCRYPTION_KEY=your-encryption-key-32-characters



# API

API_PORT=8080

API_RATE_LIMIT=100



# LGPD

TERMS_VERSION=1.0.0

```

**Frontend (.env):**

```env

API_BASE_URL=https://api.example.com

ENCRYPTION_KEY=same-as-backend-32-chars

ENABLE_BIOMETRIC_AUTH=true

```

---

## 6. CHECKLIST DE IMPLEMENTAÇÃO

### 6.1 Backend - Checklist

#### Estrutura

- [ ] Estrutura de pastas Clean Architecture criada

- [ ] Camadas: domain, usecases, adapters, infrastructure

- [ ] Dependency injection configurada

#### Domain Layer

- [ ] Entities definidas (User, Customer, Product, Order, Invoice)

- [ ] Value Objects (Email, CPF, Money, Address)

- [ ] Domain errors

#### Use Cases Layer

- [ ] Interfaces de repositórios

- [ ] Interfaces de serviços

- [ ] Use cases CRUD para cada entidade

- [ ] Validações de negócio

#### Adapters Layer

- [ ] HTTP handlers

- [ ] DTOs

- [ ] Middlewares (auth, rate limit, security headers, CORS)

- [ ] Rotas configuradas

#### Infrastructure Layer

- [ ] Conexão com PostgreSQL

- [ ] Repositórios implementados

- [ ] Migrations

- [ ] Cache Redis

- [ ] Serviço de criptografia (AES-256)

- [ ] Serviço JWT

- [ ] Validação de inputs

- [ ] Prevenção SQL injection

#### Segurança

- [ ] Criptografia implementada

- [ ] Hash de senhas (bcrypt)

- [ ] JWT com refresh token

- [ ] Rate limiting

- [ ] Security headers

- [ ] CORS configurado

- [ ] Validação de inputs

- [ ] Sanitização de dados

- [ ] LGPD - Gestão de consentimentos

- [ ] LGPD - Audit trail

- [ ] LGPD - Anonimização de dados

#### Testes

- [ ] Testes de unidade (use cases)

- [ ] Testes de integração (repositórios)

- [ ] Testes de integração (API)

- [ ] Cobertura mínima 80%

### 6.2 Frontend - Checklist

#### Estrutura

- [ ] Estrutura de pastas Clean Architecture criada

- [ ] Módulos definidos (auth, customers, products, orders)

- [ ] Dependency injection (get_it)

#### Core

- [ ] Constants

- [ ] Theme

- [ ] Errors/Failures

- [ ] Network Info

- [ ] Utils (validators, formatters)

- [ ] Security (encryption, secure storage)

#### Features (para cada módulo)

- [ ] Entities

- [ ] Repositories (abstract)

- [ ] Use cases

- [ ] Models

- [ ] Data sources (remote + local)

- [ ] Repository implementation

- [ ] Providers (ChangeNotifier)

- [ ] Pages

- [ ] Widgets

#### Offline-First

- [ ] Local database (Hive/Drift)

- [ ] Cache de dados

- [ ] Sync Queue

- [ ] Network Info

- [ ] Desenvolvimento otimista implementado

#### Segurança

- [ ] Secure Storage implementado

- [ ] Criptografia local

- [ ] Validadores de input

- [ ] Sanitização de dados

- [ ] LGPD - Termo de aceite

- [ ] LGPD - Gestão de consentimentos

- [ ] LGPD - Tela de privacidade

#### Testes

- [ ] Testes de unidade (use cases, models)

- [ ] Testes de widget

- [ ] Testes de integração

- [ ] Golden tests

- [ ] Cobertura mínima 80%

### 6.3 DevOps - Checklist

- [ ] Dockerfile backend

- [ ] Dockerfile frontend

- [ ] docker-compose.yml

- [ ] CI/CD pipeline (GitHub Actions)

- [ ] Variáveis de ambiente

- [ ] Migrations automáticas

- [ ] Backup de banco de dados

- [ ] Monitoramento e logs

- [ ] SSL/TLS configurado

---

## 7. DEPENDÊNCIAS

### 7.1 Backend (Go)

```go

// go.mod

module  github.com/yourcompany/erp-crm-backend



go  1.21



require (

github.com/lib/pq  v1.10.9  // PostgreSQL

github.com/go-redis/redis/v8  v8.11.5  // Redis

github.com/golang-jwt/jwt/v5  v5.2.0  // JWT

github.com/gorilla/mux  v1.8.1  // Router

golang.org/x/crypto  v0.17.0  // Bcrypt

github.com/stretchr/testify  v1.8.4  // Testing

github.com/go-playground/validator/v10  v10.16.0  // Validation

golang.org/x/time  v0.5.0  // Rate limiting

)

```

### 7.2 Frontend (Flutter)

```yaml

# pubspec.yaml

dependencies:

flutter:

sdk: flutter

# State Management

provider: ^6.1.1

# Dependency Injection

get_it: ^7.6.4

# Network

dio: ^5.4.0

connectivity_plus: ^5.0.2

# Local Storage

hive: ^2.2.3

hive_flutter: ^1.1.0

flutter_secure_storage: ^9.0.0

# Security

encrypt: ^5.0.3

local_auth: ^2.1.8

# Functional Programming

dartz: ^0.10.1

# Utils

intl: ^0.18.1

dev_dependencies:

flutter_test:

sdk: flutter

# Testing

mockito: ^5.4.4

build_runner: ^2.4.7

# Code Quality

flutter_lints: ^3.0.1

# Golden Tests

golden_toolkit: ^0.15.0

```

---

## 8. RECURSOS ADICIONAIS

### 8.1 Documentação Recomendada

**Clean Architecture:**

- "Clean Architecture" - Robert C. Martin

- https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html

**SOLID:**

- "Agile Software Development" - Robert C. Martin

**Golang:**

- https://go.dev/doc/

- https://gobyexample.com/

**Flutter:**

- https://docs.flutter.dev/

- https://pub.dev/

**LGPD:**

- Lei nº 13.709/2018

- https://www.gov.br/cidadania/pt-br/acesso-a-informacao/lgpd

### 8.2 Ferramentas Úteis

**Backend:**

- Postman/Insomnia - Testes de API

- pgAdmin - Gerenciamento PostgreSQL

- Redis Commander - Gerenciamento Redis

**Frontend:**

- Flutter DevTools

- Android Studio / VSCode

- Figma - Design

**DevOps:**

- Docker Desktop

- GitHub Actions

- Sentry - Error tracking

---

## RESUMO EXECUTIVO

Este documento fornece diretrizes completas para construir um sistema ERP/CRM com:

✅ **Backend em Golang** com Clean Architecture e SOLID

✅ **Frontend em Flutter** com Provider, Offline-First e Desenvolvimento Otimista

✅ **Segurança robusta** (criptografia, validação, LGPD)

✅ **Testes completos** (unidade, integração, widget, golden)

✅ **Sem streams, sem setState** no Flutter

✅ **Conceito de módulos** sem lib modular

### Próximos Passos:

1. Criar estrutura de pastas conforme especificado

2. Implementar camadas seguindo exemplos de código

3. Aplicar princípios SOLID em todas as camadas

4. Implementar segurança (criptografia, validação, LGPD)

5. Escrever testes com cobertura mínima 80%

6. Configurar CI/CD

7. Deploy com Docker

---

**Versão:** 1.0.0

**Data:** 2025

**Autor:** Diretrizes para IA
