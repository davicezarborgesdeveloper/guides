# Princípios Fundamentais de Teste Automatizado

Aqui estão os princípios fundamentais de teste automatizado que orientam a criação e a manutenção de testes eficazes.

## 🧭 1. Confiabilidade

**Objetivo:** um teste automatizado deve produzir o mesmo resultado sempre que for executado em condições iguais.

**Problemas comuns:** testes que falham aleatoriamente (_flaky tests_) minam a confiança e dificultam a detecção de problemas reais.

**Boas práticas:** evite dependência de dados externos, horários ou ambientes instáveis.

## 🔄 2. Repetibilidade

Um teste deve ser executável quantas vezes for necessário sem necessidade de intervenção manual.

Isso permite rodar os testes em pipelines de integração contínua, em diferentes ambientes ou após cada alteração no código.

## ⚡ 3. Rapidez

Testes automatizados devem ser rápidos para incentivar sua execução frequente.

**Dica:** divida os testes em níveis (ex.: testes de unidade rápidos, testes de integração um pouco mais lentos, testes de ponta a ponta mais pesados).

## 🧪 4. Isolamento

Cada teste deve rodar de forma independente, sem interferir ou depender de outros testes.

Isso evita falhas em cascata e facilita a depuração.

## 🔍 5. Determinismo

Dado o mesmo código, entrada e ambiente, o resultado do teste deve ser sempre o mesmo.

**Evite:** uso de dados aleatórios não controlados, dependência de horário atual, ou chamadas externas imprevisíveis.

## 📚 6. Legibilidade

Testes devem ser fáceis de entender. Um bom teste é uma documentação viva do comportamento esperado.

**Dica:** nomeie bem os testes e use estruturas claras (ex.: _Given, When, Then_).

## ⚙️ 7. Manutenibilidade

Um teste deve ser fácil de atualizar conforme o sistema evolui.

Evite testes altamente acoplados à implementação interna (testes frágeis).

## 🧩 8. Cobertura significativa

Os testes devem cobrir os casos relevantes de negócio, não apenas os caminhos felizes.

Mas evite perseguir **100%** de cobertura de código como fim em si mesmo — priorize valor e risco.

## 🛠️ 9. Automação

A automação não se limita à execução: inclua também a geração de dados, _setup_ e _teardown_.

Use ferramentas para rodar testes automaticamente após cada commit ou pull request (CI/CD).

## 🧯 10. Feedback rápido

Os testes devem fornecer respostas claras e imediatas quando algo falha, indicando onde e por quê.

Mensagens de erro devem ser úteis e informativas.

---

# Testes Automatizados no Flutter — Princípios e Boas Práticas

Os princípios de testes automatizados no Flutter seguem muitos dos fundamentos gerais de testes de software, mas também incorporam particularidades da plataforma. Abaixo estão os princípios específicos e boas práticas de testes automatizados no Flutter, divididos em categorias.

## 🧱 1. Separação por Nível de Teste

Flutter oferece três tipos principais de testes automatizados, cada um com seu propósito e abordagem.

### a) Testes de Unidade (_Unit Tests_)

**Foco:** testar funções, classes e lógica de negócio de forma isolada.

**Princípios:**

- Isolamento total (sem dependências de framework ou UI).

- Alta velocidade de execução.

- Ideal para lógica pura de Dart (ex.: validações, cálculos, manipuladores de estado).

### b) Testes de Widget

**Foco:** verificar a aparência e comportamento de widgets individuais.

**Princípios:**

- Utiliza `WidgetTester` para renderizar widgets em ambiente simulado.

- Deve manter testes rápidos.

- Testa interação, layout e estado visual.

### c) Testes de Integração (_Integration Tests_ ou E2E)

**Foco:** simular o uso do app completo, interagindo com a UI como um usuário real.

**Princípios:**

- Usa `integration_test` ou ferramentas como `flutter_driver`.

- Pode envolver dependências externas (ex.: rede, banco local).

- Ideal para jornadas completas, mas mais lentos.

## 🧪 2. Princípios Fundamentais de Qualidade

- **Confiabilidade:** os testes devem passar ou falhar consistentemente. Evite lógica assíncrona mal sincronizada (`pumpAndSettle` ajuda a esperar animações ou renders).

- **Rapidez:** testes devem ser rápidos para rodarem com frequência (especialmente os de unidade e widget). Limite o número de testes E2E no pipeline principal de CI.

- **Repetibilidade:** testes devem poder ser rodados quantas vezes for necessário com o mesmo resultado.

- **Isolamento:** use _mocks_ e _fakes_ para evitar dependências reais (ex.: Firebase, APIs). Bibliotecas úteis: `mockito`, `fake_async`, `mocktail`.

- **Clareza:** nomeie bem os testes e explique o que está sendo verificado. Estruture os testes com o padrão **Given → When → Then**.

## 🛠️ 3. Boas Práticas Específicas do Flutter

- **Use `setUp` e `tearDown`:** inicialize variáveis e limpe o estado para garantir isolamento.

- **Teste estados com gerenciadores (ex.: Bloc, Provider, Riverpod):** teste a emissão de estados, transições e reações. Ex.: usando `bloc_test` para verificar sequências de estados emitidos.

- **Modularize seu código:** separe lógica da UI para facilitar testes de unidade e widget (ex.: mover regras de negócio para serviços ou controllers).

- **Utilidades úteis:** `testWidgets`, `expect`, `find`, `tap`, `pump`, `pumpWidget`, `pumpAndSettle`.

- **Golden tests:** compare visualmente widgets (útil para regressão visual).

## ⚙️ 4. Automação e Integração Contínua

- Configure CI/CD para rodar testes automaticamente (GitHub Actions, Bitrise, Codemagic).

- Defina critérios mínimos de cobertura.

- Use testes rápidos para feedback imediato em PRs, e testes E2E em ciclos mais longos.

---

# Exemplos Práticos no Flutter

Abaixo seguem exemplos práticos e comentados para cada tipo de teste automatizado no Flutter: unitário, widget e integração. O cenário é uma aplicação de tarefas (_to-do_) com um campo de texto e botão para adicionar itens à lista.

## ✅ 1) Teste de Unidade (_Unit Test_)

**Objetivo:** testar a lógica de uma classe que gerencia uma lista de tarefas (`TaskManager`), sem nenhuma dependência de UI.

### Código da classe

```dart

// lib/task_manager.dart

class  TaskManager {

final  List<String> _tasks = [];



List<String> get tasks => List.unmodifiable(_tasks);



void  addTask(String task) {

if (task.trim().isEmpty) throw  ArgumentError('Task cannot be empty');

_tasks.add(task.trim());

}



void  clear() {

_tasks.clear();

}

}

```

### Teste

```dart

// test/task_manager_test.dart

import  'package:flutter_test/flutter_test.dart';

import  'package:my_app/task_manager.dart';



void  main() {

group('TaskManager', () {

late  TaskManager manager;



setUp(() {

manager = TaskManager();

});



test('adiciona uma tarefa corretamente', () {

manager.addTask('Estudar Flutter');

expect(manager.tasks, contains('Estudar Flutter'));

});



test('lança erro ao adicionar tarefa vazia', () {

expect(() => manager.addTask(' '), throwsArgumentError);

});



test('limpa todas as tarefas', () {

manager.addTask('Item 1');

manager.clear();

expect(manager.tasks.isEmpty, isTrue);

});

});

}

```

## 🧱 2) Teste de Widget

**Objetivo:** testar a renderização e comportamento de um widget `AddTaskWidget`.

### Widget a ser testado

```dart

// lib/add_task_widget.dart

import  'package:flutter/material.dart';



class  AddTaskWidget  extends  StatefulWidget {

final  void  Function(String) onAdd;



AddTaskWidget({required  this.onAdd});



@override

_AddTaskWidgetState  createState() => _AddTaskWidgetState();

}



class  _AddTaskWidgetState  extends  State<AddTaskWidget> {

final _controller = TextEditingController();



void  _submit() {

final text = _controller.text;

if (text.isNotEmpty) {

widget.onAdd(text);

_controller.clear();

}

}



@override

Widget  build(BuildContext context) {

return  Column(

children: [

TextField(key: Key('taskField'), controller: _controller),

ElevatedButton(

key: Key('addButton'),

onPressed: _submit,

child: Text('Adicionar'),

),

],

);

}

}

```

### Teste de widget

```dart

// test/add_task_widget_test.dart

import  'package:flutter/material.dart';

import  'package:flutter_test/flutter_test.dart';

import  'package:my_app/add_task_widget.dart';



void  main() {

testWidgets('adiciona tarefa ao pressionar botão', (WidgetTester tester) async {

String? result;



await tester.pumpWidget(

MaterialApp(

home: Scaffold(

body: AddTaskWidget(onAdd: (value) => result = value),

),

),

);



await tester.enterText(find.byKey(Key('taskField')), 'Nova Tarefa');

await tester.tap(find.byKey(Key('addButton')));

await tester.pump(); // re-renderiza o frame após o clique



expect(result, equals('Nova Tarefa'));

});

}

```

## 📱 3) Teste de Integração (End-to-End)

**Objetivo:** testar a aplicação completa simulando um usuário interagindo com a UI.

### Instalação no `pubspec.yaml`

```yaml
dev_dependencies:

integration_test:

flutter_test:

flutter:
```

### Arquivo de teste

```dart

// integration_test/app_test.dart

import  'package:flutter_test/flutter_test.dart';

import  'package:integration_test/integration_test.dart';

import  'package:my_app/main.dart'  as app;



void  main() {

IntegrationTestWidgetsFlutterBinding.ensureInitialized();



testWidgets('usuário adiciona tarefa e vê na lista', (WidgetTester tester) async {

app.main(); // inicia o app real

await tester.pumpAndSettle();



final taskField = find.byKey(Key('taskField'));

final addButton = find.byKey(Key('addButton'));



await tester.enterText(taskField, 'Tarefa Integração');

await tester.tap(addButton);

await tester.pumpAndSettle();



expect(find.text('Tarefa Integração'), findsOneWidget);

});

}

```

### Para rodar

```bash

flutter  test  integration_test/app_test.dart

```

---

# Golden Tests no Flutter

Golden tests no Flutter comparam visualmente se um widget renderiza exatamente como esperado. Eles são muito úteis para detectar regressões visuais.

## 1) Configuração no `pubspec.yaml`

No Flutter, os golden tests usam imagens como referência.

Você precisa adicionar uma pasta para armazenar essas imagens (por padrão, algo como `test/goldens`).

```yaml
dev_dependencies:

flutter_test:

sdk: flutter

flutter:

assets:
  - test/goldens/
```

## 2) Widget para teste visual

Vamos criar um widget simples que sempre deve ter a mesma aparência:

```dart

// lib/task_card.dart

import  'package:flutter/material.dart';



class  TaskCard  extends  StatelessWidget {

final  String title;

final  bool completed;



const  TaskCard({super.key, required  this.title, required  this.completed});



@override

Widget  build(BuildContext context) {

return  Card(

color: completed ? Colors.green[100] : Colors.white,

child: ListTile(

leading: Icon(

completed ? Icons.check_circle : Icons.radio_button_unchecked,

color: completed ? Colors.green : Colors.grey,

),

title: Text(title),

),

);

}

}

```

## 3) Golden test

O teste renderiza o widget e tira uma “foto” para comparar com a imagem esperada.

```dart

// test/task_card_golden_test.dart

import  'package:flutter/material.dart';

import  'package:flutter_test/flutter_test.dart';

import  'package:my_app/task_card.dart';



void  main() {

testWidgets('TaskCard - visual padrão', (WidgetTester tester) async {

await tester.pumpWidget(

MaterialApp(

home: Center(

child: TaskCard(title: 'Estudar Flutter', completed: false),

),

),

);



await  expectLater(

find.byType(TaskCard),

matchesGoldenFile('goldens/task_card_incompleto.png'),

);

});



testWidgets('TaskCard - visual concluído', (WidgetTester tester) async {

await tester.pumpWidget(

MaterialApp(

home: Center(

child: TaskCard(title: 'Estudar Flutter', completed: true),

),

),

);



await  expectLater(

find.byType(TaskCard),

matchesGoldenFile('goldens/task_card_completo.png'),

);

});

}

```

## 4) Gerando as imagens iniciais

Na primeira execução, você provavelmente não terá as imagens esperadas. Então, rode:

```bash

flutter  test  --update-goldens

```

Isso gera ou atualiza as imagens dentro da pasta `test/goldens/`.

Depois, para validar:

```bash

flutter  test

```

Se o widget mudar, o teste falhará mostrando as diferenças visuais.

## 5) Dicas para golden tests no Flutter

- **Ambiente fixo:** defina tamanho de tela e tema para consistência:

```dart

await tester.binding.setSurfaceSize(Size(300, 100));

```

- Evite conteúdo dinâmico (datas, textos aleatórios).

- Organize _goldens_ por componente/página para fácil manutenção.

- Integre no CI/CD para evitar que alterações visuais passem despercebidas.

---

# Executando seus testes do Flutter no Firebase Test Lab

Se você quiser rodar seus testes automatizados do Flutter — incluindo unitários, widget, integração e até golden tests — na nuvem, o **Firebase Test Lab** é uma ótima opção. Ele executa seus testes em dispositivos Android/iOS reais e virtuais, hospedados pelo Google.

## ☁️ 1) O que é o Firebase Test Lab

Plataforma de testes na nuvem do Google.

- Permite rodar testes automatizados (instrumentados e de unidade) em dispositivos reais e emuladores.

- Suporte para:

- **Android** (nativo, Flutter, Unity…)

- **iOS** (apenas com plano pago, usando Xcode e dispositivos reais)

- Integração com Google Cloud CLI ou CI/CD (GitHub Actions, Bitrise, Codemagic…).

## 🛠 2) Pré-requisitos

- Conta no Google Cloud.

- Ative o Firebase Test Lab no seu projeto do Google Cloud.

- Ative a **Cloud Testing API** e **Cloud Tool Results API**.

- Google Cloud SDK instalado:

```bash

curl https://sdk.cloud.google.com | bash

exec -l $SHELL

gcloud init

```

- Flutter SDK configurado.

## 📦 3) Configurando o projeto Flutter para testes no Test Lab

Para rodar testes de integração (ou golden tests via instrumentação), o Firebase Test Lab precisa de um APK instrumentado.

### 3.1 Dependências

Adicione no `pubspec.yaml`:

```yaml

dev_dependencies:

flutter_test:

sdk: flutter

integration_test:

sdk: flutter

```

### 3.2 Estrutura do teste de integração

Exemplo básico:

```dart

// integration_test/app_test.dart

import  'package:flutter_test/flutter_test.dart';

import  'package:integration_test/integration_test.dart';

import  'package:my_app/main.dart'  as app;



void  main() {

IntegrationTestWidgetsFlutterBinding.ensureInitialized();



testWidgets('fluxo principal do app', (tester) async {

app.main();

await tester.pumpAndSettle();



expect(find.text('Bem-vindo'), findsOneWidget);

});

}

```

## 📦 4) Gerando APK para o Test Lab

O Test Lab precisa de dois APKs:

- APK do app (build normal).

- APK de teste (instrumentado para rodar integração).

Comandos:

```bash

flutter  build  apk  --debug

flutter  build  apk  --debug  --target  integration_test/app_test.dart

```

Isso gera algo como:

```text

build/app/outputs/flutter-apk/app-debug.apk

build/app/outputs/apk/debug/app-debug-androidTest.apk

```

## ☁️ 5) Executando no Firebase Test Lab

Com os dois APKs prontos:

```bash

gcloud  firebase  test  android  run  --type  instrumentation  --app  build/app/outputs/flutter-apk/app-debug.apk  --test  build/app/outputs/apk/debug/app-debug-androidTest.apk  --device  model=Pixel2,version=30,locale=en,orientation=portrait

```

**Opções úteis:**

- `--device`: especifica modelo, versão, idioma, orientação.

- `--timeout`: tempo máximo por teste (ex.: `--timeout 5m`).

- `--results-bucket`: define _bucket_ no Google Cloud Storage para salvar resultados.

## 🖥 6) Integração com CI/CD (GitHub Actions)

```yaml

name: Firebase Test Lab



on: [push, pull_request]



jobs:

test:

runs-on: ubuntu-latest

steps:

- uses: actions/checkout@v3

- uses: subosito/flutter-action@v2

with:

flutter-version: '3.24.0'

- run: flutter pub get

- run: flutter build apk --debug

- run: flutter build apk --debug --target integration_test/app_test.dart

- uses: google-github-actions/auth@v1

with:

credentials_json: ${{ secrets.GCP_CREDENTIALS }}

- run: gcloud firebase test android run --type instrumentation --app build/app/outputs/flutter-apk/app-debug.apk --test build/app/outputs/apk/debug/app-debug-androidTest.apk --device model=Pixel2,version=30,locale=en,orientation=portrait

```

## 📊 7) Relatórios e Resultados

O Test Lab gera:

- Vídeo da execução

- Capturas de tela

- Logs detalhados

- Saída do teste (JUnit, JSON)

Tudo fica no Google Cloud Storage e no console do Firebase.

---

**FIM**
