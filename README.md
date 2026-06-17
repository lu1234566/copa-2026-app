# Copa 2026 Replay — App Android (WebView)

App Android nativo que carrega **https://tabela2026.lovable.app** num WebView.
É uma casca leve: o site roda na nuvem, e toda vez que você atualizar o
deploy no Lovable, o app já mostra a versão nova — sem precisar gerar APK de novo.

- **Nenhuma dependência de npm/Capacitor/Cordova** — Android nativo puro.
- Botão "voltar" navega no histórico do site.
- Links pra fora do domínio abrem no navegador.
- Mantém sessão/cookies/localStorage (login do Supabase funciona).
- Ícone próprio (bola de futebol, fundo escuro), adaptativo + raster.

---

## ⭐ Jeito mais fácil: gerar o APK na nuvem (GitHub Actions)

Sem instalar **nada** no seu PC. O GitHub compila o APK pra você.

1. Crie um repositório novo no GitHub (pode ser privado).
2. Suba **todo o conteúdo desta pasta** pra esse repositório. Ou pelo site
   (botão "Add file → Upload files", arraste tudo), ou por linha de comando:
   ```bash
   git init
   git add .
   git commit -m "app android webview"
   git branch -M main
   git remote add origin https://github.com/SEU_USUARIO/SEU_REPO.git
   git push -u origin main
   ```
3. Assim que o push terminar, o build começa sozinho. Acompanhe na aba
   **Actions** do repositório.
4. Quando ficar verde (uns 2-4 min), entre no build → seção **Artifacts**
   → baixe **copa-2026-replay-apk**. Dentro do zip está o `app-debug.apk`.
5. Passe pro celular e instale (precisa liberar "instalar de fontes
   desconhecidas").

> Pra rodar de novo sem dar push: aba **Actions → Gerar APK → Run workflow**.

Esse APK é de **debug** — perfeito pra testar e distribuir informalmente,
mas não serve pra Play Store. Pra isso, veja a seção "APK de RELEASE".

---

## O que você precisa instalar (uma vez)  — só se NÃO usar o GitHub Actions

1. **JDK 17** (Temurin/Adoptium ou o que vem com o Android Studio).
2. **Android SDK** — a forma mais fácil é instalar o
   [Android Studio](https://developer.android.com/studio).

---

## Build — opção A: Android Studio (mais fácil)

1. Abra o Android Studio → **Open** → selecione esta pasta.
2. Espere o Gradle sincronizar (ele baixa o SDK que faltar sozinho).
3. Menu **Build → Build Bundle(s) / APK(s) → Build APK(s)**.
4. Quando terminar, clique em **locate** no aviso. O APK estará em:
   `app/build/outputs/apk/debug/app-debug.apk`

Esse `app-debug.apk` já instala em qualquer celular (com "fontes
desconhecidas" liberado). É o suficiente pra testar e distribuir
informalmente.

---

## Build — opção B: linha de comando

Com o Android SDK instalado e a variável `ANDROID_HOME` (ou
`ANDROID_SDK_ROOT`) apontando pra ele:

```bash
# na raiz do projeto
./gradlew assembleDebug
```

APK gerado em: `app/build/outputs/apk/debug/app-debug.apk`

> Se reclamar que falta o SDK, crie um arquivo `local.properties` na raiz
> com a linha:
> `sdk.dir=/caminho/para/o/Android/Sdk`
> (no Windows: `sdk.dir=C\:\\Users\\voce\\AppData\\Local\\Android\\Sdk`)

---

## APK de RELEASE (pra Play Store ou distribuição "de verdade")

O `app-debug.apk` é assinado com a chave de debug e não serve pra Play Store.
Pra um release assinado:

1. Gere uma keystore (uma vez):
   ```bash
   keytool -genkey -v -keystore copa2026.keystore \
     -alias copa2026 -keyalg RSA -keysize 2048 -validity 10000
   ```
2. No Android Studio: **Build → Generate Signed Bundle / APK**, escolha
   APK ou AAB (a Play Store prefere **AAB**), aponte a keystore e gere.

---

## Como mudar coisas depois

- **URL do site:** `app/src/main/java/app/tabela2026/copa/MainActivity.java`
  → constantes `START_URL` e `APP_HOST`.
- **Nome do app:** `app/src/main/res/values/strings.xml` → `app_name`.
- **Cor de fundo / status bar:** `app/src/main/res/values/colors.xml`.
- **Ícone:** substitua os PNGs em `app/src/main/res/mipmap-*/` e/ou o vetor
  em `app/src/main/res/drawable/ic_launcher_foreground.xml`.
- **Versão:** `app/build.gradle` → `versionCode` (inteiro, sobe a cada
  release) e `versionName` (texto, ex "1.1").

---

## Observação de segurança (importante)

No seu repositório o arquivo `.env` com as chaves do Supabase está
versionado e público. Mesmo sendo a chave *anon* (pública), o ideal é:
- adicionar `.env` ao `.gitignore`;
- conferir as **policies de Row Level Security** no painel do Supabase,
  pra garantir que ninguém leia/escreva o que não deve.
Isso não afeta este app — é sobre o repositório do site.
