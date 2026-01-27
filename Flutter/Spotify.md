# Flutter + Spotify Premium — Guia passo a passo

> Objetivo: abrir a tela de conexão (imagem 1), ler um QR (imagem 2), localizar a faixa numa playlist e tocar via Spotify (imagens 3 e 4).

> Stack: **spotify_sdk** (controle de reprodução + token), **mobile_scanner** (QR), **http** (Web API do Spotify), **url_launcher** (fallback para instalar app).

---

## 0) Pré‑requisitos

- **Conta Spotify Premium** no dispositivo.

- **App do Spotify** instalado e logado.

- **Flutter 3.x**.

- Android **minSdk 24+** e iOS 13+.

Instale as dependências:

```bash

flutter  pub  add  spotify_sdk  mobile_scanner  http  url_launcher

```

---

## 1) Spotify for Developers — configurações necessárias

1. Acesse <https://developer.spotify.com/dashboard> e crie um **App**.

2. Anote o **Client ID** e o **Client Secret** (use o secret apenas no backend, se optar por Authorization Code).

3. Em **Redirect URIs**, adicione o esquema do seu app (exemplo):

```

gs3flutter://callback

```

4. Habilite os **escopos** (scopes) necessários:

- `user-read-email` (opcional)

- `playlist-read-private` (se a playlist não for pública)

- `user-modify-playback-state`

- `user-read-playback-state`

- `app-remote-control`

5. Salve as alterações.

> Produção: o ideal é Authorization Code + **refresh token** via backend para uso da Web API sem expirar. Para validar o fluxo mobile rapidamente, este guia usa o `spotify_sdk` para obter um **access token** temporário (≈1h).

---

## 2) Conexão Flutter ↔ Spotify (tela “Connect with Spotify”, imagem 1)

### Android

`android/app/build.gradle`

```gradle

android {
	defaultConfig {
		minSdkVersion 24
	}
}
```

`android/app/src/main/AndroidManifest.xml`

```xml

<manifest ...>
	<application ...>
	<!-- Deep link do Redirect URI para o login via SDK -->
		<activity  android:name="com.spotify.sdk.android.authentication.LoginActivity"
		android:exported="true">
			<intent-filter>
				<action  android:name="android.intent.action.VIEW"  />
				<category  android:name="android.intent.category.DEFAULT"  />
				<category  android:name="android.intent.category.BROWSABLE"  />
				<data  android:scheme="gs3flutter"  android:host="callback"  />
			</intent-filter>
		</activity>
	</application>
</manifest>
```

### iOS

`ios/Runner/Info.plist`

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
	<string>spotify</string>
	<string>spotify-action</string>
</array>
<key>CFBundleURLTypes</key>
<array>
<dict>
<key>CFBundleURLSchemes</key>
<array>
<string>gs3flutter</string>
</array>
</dict>
</array>
```

### Serviço de conexão/token

`lib/services/spotify_service.dart`

```dart

import 'dart:async';
import 'package:spotify_sdk/spotify_sdk.dart';

class SpotifyService {
final String clientId;
final String redirectUrl;
final String scopes;

SpotifyService({
  required this.clientId,
  required this.redirectUrl,
  this.scopes =
      'app-remote-control,user-read-playback-state,user-modify-playback-state,playlist-read-private',
});

Future<bool> connect({String? accessToken}) async {
  final connected = await SpotifySdk.connectToSpotifyRemote(
    clientId: clientId,
    redirectUrl: redirectUrl,
    scope: scopes,
    accessToken: accessToken,
  );
  return connected;
}

/// Token temporário para Web API
Future<String> getAccessToken() async {
  final token = await SpotifySdk.getAccessToken(
    clientId: clientId,
    redirectUrl: redirectUrl,
    scope: scopes,
  );
  return token;
}

Future<bool> authorizeAndConnect() async {
  final token = await getAccessToken();
  return connect(accessToken: token);
}

Future<bool> disconnect() => SpotifySdk.disconnect();
}


```

Uso:

```dart
final spotify = SpotifyService(
  clientId: 'SEU_CLIENT_ID',
  redirectUrl: 'gs3flutter://callback',
);

final connected = await spotify.connect();
final token = await spotify.getAccessToken();
```

---

## 3) Ler QR code e buscar a música da playlist (imagem 2)

### Scanner

`lib/features/qr/qr_scanner_page.dart`

```dart
import 'package:flutter/material.dart';
import 'package:mobile_scanner/mobile_scanner.dart';
class QrScannerPage extends StatelessWidget {
  final void Function(String code) onCode;
  const QrScannerPage({super.key, required this.onCode});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Stack(
        children: [
          MobileScanner(
            onDetect: (capture) {
              final barcodes = capture.barcodes;
              if (barcodes.isNotEmpty) {
                final raw = barcodes.first.rawValue ?? '';
                Navigator.of(context).pop();
                onCode(raw);
              }
            },
          ),
          // Aqui você pode desenhar um alvo quadrado como no mock (imagem 2)
        ],
      ),
    );
  }
}

```

### Normalização do QR

Aceite formatos:

- `spotify:track:<ID>`

- `https://open.spotify.com/track/<ID>`

- texto livre (nome/artista) → pesquisa

`lib/features/qr/qr_utils.dart`

```dart
String? parseTrackIdFrom(String qr) {
  final uriLike = RegExp(r'spotify:track:([A-Za-z0-9]+)').firstMatch(qr);
  if (uriLike != null) return uriLike.group(1);

  final urlLike = RegExp(
    r'open\.spotify\.com/track/([A-Za-z0-9]+)',
  ).firstMatch(qr);
  if (urlLike != null) return urlLike.group(1);

  return null; // cairá no modo "search"
}


```

### Web API (playlist e search)

`lib/services/spotify_webapi.dart`

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class SpotifyWebApi {
  final String accessToken;

  SpotifyWebApi(this.accessToken);

  Map<String, String> get _h => {
    'Authorization': 'Bearer $accessToken',
    'Content-Type': 'application/json',
  };

  Future<List<Map<String, dynamic>>> getPlaylistTracks(
    String playlistId,
  ) async {
    final out = <Map<String, dynamic>>[];
    var url = Uri.parse(
      'https://api.spotify.com/v1/playlists/$playlistId/tracks'
      '?limit=100&fields=items(track(uri,id,name,artists(name))),next',
    );

    while (true) {
      final r = await http.get(url, headers: _h);
      if (r.statusCode != 200) {
        throw Exception('Playlist GET ${r.statusCode}');
      }
      final j = json.decode(r.body) as Map<String, dynamic>;
      final items = (j['items'] as List)
          .map<Map<String, dynamic>>((e) => e['track'])
          .where((t) => t != null)
          .cast<Map<String, dynamic>>()
          .toList();
      out.addAll(items);
      final next = j['next'];
      if (next == null) break;
      url = Uri.parse(next);
    }
    return out;
  }

  Future<Map<String, dynamic>?> searchTrack(String query) async {
    final url = Uri.parse(
      'https://api.spotify.com/v1/search'
      '?type=track&limit=1&q=${Uri.encodeQueryComponent(query)}',
    );
    final r = await http.get(url, headers: _h);
    if (r.statusCode != 200) return null;
    final j = json.decode(r.body);
    final items = j['tracks']?['items'] as List? ?? [];
    if (items.isEmpty) return null;
    return items.first as Map<String, dynamic>;
  }
}

```

### Resolver o QR para uma URI tocável, validando na playlist

`lib/features/qr/resolve_qr.dart`

```dart
import '../services/spotify_webapi.dart';
import 'qr_utils.dart';

Future<String?> findTrackUriFromQr({
  required String qr,
  required String accessToken,
  required String playlistId,
}) async {
  final api = SpotifyWebApi(accessToken);
  final id = parseTrackIdFrom(qr);
  if (id != null) {
    final tracks = await api.getPlaylistTracks(playlistId);
    final hit = tracks.firstWhere((t) => t['id'] == id, orElse: () => {});
    if (hit.isNotEmpty) return hit['uri'] as String; // spotify:track:XYZ
    return null;
  }

  // Sem ID → pesquisa por texto
  final guess = await api.searchTrack(qr);
  if (guess == null) return null;
  // Opcional: garantir que está na playlist
  final tracks = await api.getPlaylistTracks(playlistId);
  final ok = tracks.any((t) => t['id'] == guess['id']);
  return ok ? (guess['uri'] as String) : null;
}


```

---

## 4) Tocar a música no app (imagens 3 e 4)

### Controle de player (App Remote)

`lib/features/player/player_controller.dart`

```dart

import 'package:spotify_sdk/spotify_sdk.dart';

class PlayerController {
  Future<void> playUri(String spotifyUri) async {
    await SpotifySdk.play(spotifyUri: spotifyUri);
  }

  Future<void> pause() => SpotifySdk.pause();
  Future<void> resume() => SpotifySdk.resume();

  Stream<PlayerState> get playerStateStream =>
      SpotifySdk.subscribePlayerState();
}

```

### UI minimalista (círculo Play/Pause)

`lib/features/player/player_screen.dart`

```dart
import 'package:flutter/material.dart';

import 'package:spotify_sdk/models/player_state.dart';

import 'player_controller.dart';

class PlayerScreen extends StatefulWidget {
  final String initialUri;

  const PlayerScreen({super.key, required this.initialUri});

  @override
  State<PlayerScreen> createState() => _PlayerScreenState();
}

class _PlayerScreenState extends State<PlayerScreen> {
  final ctrl = PlayerController();

  @override
  void initState() {
    super.initState();

    ctrl.playUri(widget.initialUri);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: StreamBuilder<PlayerState>(
          stream: ctrl.playerStateStream,

          builder: (context, snap) {
            final playing = snap.data?.isPaused == false;

            return GestureDetector(
              onTap: () => playing ? ctrl.pause() : ctrl.resume(),

              child: Container(
                width: 220,

                height: 220,

                decoration: BoxDecoration(
                  shape: BoxShape.circle,

                  gradient: const LinearGradient(
                    colors: [Color(0xFF7BE495), Color(0xFF5EC5FF)],

                    begin: Alignment.topRight,

                    end: Alignment.bottomLeft,
                  ),

                  boxShadow: const [
                    BoxShadow(blurRadius: 24, color: Colors.black26),
                  ],
                ),

                child: Icon(
                  playing ? Icons.pause : Icons.play_arrow,

                  size: 120,

                  color: Colors.white,
                ),
              ),
            );
          },
        ),
      ),
    );
  }
}


```

### Orquestração do fluxo

`lib/main_flow.dart`

```dart

import  'package:flutter/material.dart';

import  'features/qr/qr_scanner_page.dart';

import  'features/qr/resolve_qr.dart';

import  'features/player/player_screen.dart';

import  'services/spotify_service.dart';



class  MainFlow  extends  StatefulWidget {

const  MainFlow({super.key});

@override

State<MainFlow> createState() => _MainFlowState();

}



class  _MainFlowState  extends  State<MainFlow> {

late  final  SpotifyService spotify;



@override

void  initState() {

super.initState();

spotify = SpotifyService(

clientId: 'SEU_CLIENT_ID',

redirectUrl: 'gs3flutter://callback',

);

}



Future<void> _start() async {

final connected = await spotify.connect();

if (!connected) {

ScaffoldMessenger.of(context).showSnackBar(

const  SnackBar(content: Text('Não foi possível conectar ao Spotify.')),

);

return;

}

final token = await spotify.getAccessToken();



// 1) Scanner

final qr = await  Navigator.push<String?>(

context,

MaterialPageRoute(builder: (_) => QrScannerPage(onCode: (c) => Navigator.pop(_, c))),

);



if (qr == null || qr.isEmpty) return;



// 2) Resolver QR → trackUri

const playlistId = 'SUA_PLAYLIST_ID';

final trackUri = await  findTrackUriFromQr(

qr: qr,

accessToken: token,

playlistId: playlistId,

);



if (trackUri == null) {

ScaffoldMessenger.of(context).showSnackBar(

const  SnackBar(content: Text('Não encontrei essa música na playlist.')),

);

return;

}



// 3) Tocar

if (context.mounted) {

Navigator.push(context, MaterialPageRoute(

builder: (_) => PlayerScreen(initialUri: trackUri),

));

}

}



@override

Widget  build(BuildContext context) {

return  Scaffold(

appBar: AppBar(title: const  Text('Demo Spotify Premium + QR')),

body: Center(

child: ElevatedButton(

onPressed: _start,

child: const  Text('Conectar e iniciar'),

),

),

);

}

}

```

---

## 5) Tratamento de erros úteis

- **Premium obrigatório**: se `play()` falhar com erro de autorização, informe claramente que é preciso **Spotify Premium**.

- **App do Spotify não instalado**: trate o erro de `connectToSpotifyRemote` e abra a loja com `url_launcher`:

```dart

import 'package:url_launcher/url_launcher.dart';

launchUrl(Uri.parse('market://details?id=com.spotify.music')); // Android

// fallback web: https://play.google.com/store/apps/details?id=com.spotify.music

```

- **Token expirado (~1h)**: renove com `getAuthenticationToken()` antes de operações longas de Web API.

- **Ordem recomendada**: conectar ao App Remote **antes** de chamar `play()`.

---

## 6) Extras

- Backend com Authorization Code + **refresh token** para operações robustas de Web API.

- Loop de jogo: após tocar, volte automaticamente ao scanner para o próximo QR.

- Garantir que o **dispositivo ativo** de reprodução é o próprio telefone (o App Remote toca no device atual do Spotify).

---

## 7) Checklist

- [ ] App criado no Dashboard com `gs3flutter://callback`.

- [ ] `clientId`/`redirectUrl` configurados no código.

- [ ] Manifest (Android) e Info.plist (iOS) com o esquema.

- [ ] Conexão exibindo a UI da imagem 1.

- [ ] Scanner lendo e normalizando o QR (imagem 2).

- [ ] Faixa validada na playlist via Web API.

- [ ] Reprodução com `SpotifySdk.play()` e UI de play/pause (imagens 3–4).

---

### Observações

- Este guia assume **Spotify Premium** por requisito do App Remote.

- Ajuste nomes de pastas/arquivos à sua estrutura.
