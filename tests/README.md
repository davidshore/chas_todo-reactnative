# Automatiska tester för todo-appen

Testerna provar lägga till, tomma titlar, rätt detaljsida, statusändringar, två todos med samma titel, borttagning och Avbryt. De använder appens riktiga routes och delade state. Du väljer själv filnamn, komponenter och state-lösning.

## Installera i din app

1. Kopiera den här `tests`-mappen till roten av din Expo-app, bredvid `package.json`. Appen ska ligga i roten av sitt GitHub-repo.
2. Kör från appens projektrot:

```bash
npx expo install jest-expo jest @types/jest @testing-library/react-native@14.1.0 test-renderer@1.2.0 --dev
node tests/install.cjs
npm run test:assignment
```

Installationsskriptet lägger till `test:assignment` i `package.json` och kopierar workflow-filen till `.github/workflows/`. Övriga scripts behålls. Checka in dessa filer och din uppdaterade `package-lock.json`.

Testpaketet är verifierat med Expo SDK 57, React Native Testing Library 14.1 och test-renderer 1.2. Versionsvalet för renderern matchar React 19.2 i SDK 57. Se [Expos testguide](https://docs.expo.dev/develop/unit-testing/) och [Expo Router testing](https://docs.expo.dev/router/reference/testing/).

## Så hittar testerna gränssnittet

Lägg till följande `testID` på motsvarande element. Det ändrar inte deras synliga text eller utseende.

| `testID` | Element |
| --- | --- |
| `home-screen` | Home-skärmens innehåll |
| `add-screen` | Add-skärmens innehåll |
| `detail-screen` | Detail-skärmens innehåll |
| `add-todo` | Tryckbart element som öppnar Add |
| `todo-input` | Titelns `TextInput` |
| `save-todo` | Spara-knappen |
| `cancel-add` | Avbryt-knappen |
| `todo-row` | Varje tryckbar rad i listan, samma ID på alla rader |
| `todo-title` | `Text` med titeln på Detail |
| `todo-status` | `Text` med status på varje rad och på Detail |
| `toggle-todo` | Knappen som växlar status på Detail |
| `delete-todo` | Ta bort-knappen på Detail |

Status ska visas som **Ej genomförd** eller **Genomförd**. Varje rad ska visa sin titel. En ny todo börjar som ej genomförd. Home ska vara appens `/`-route. Testerna hittar automatiskt `src/app` eller `app`; använd bara en av dem.

Tom titel ska lämna formuläret öppet, och Avbryt ska stänga det. Spara och Ta bort ska återgå till Home. Ett valfritt native `Alert` för borttagning bekräftas av testerna om bekräftelseknappen har `style: 'destructive'`.

## Vad körs automatiskt

GitHub Actions kör testerna och `tsc --noEmit` vid push, pull request och manuell start. Resultatet finns under **Actions** i ditt app-repo. De här filerna är ett testpaket för studentappar; uppgiftsrepot innehåller ingen app att köra dem mot.

Tester körs utanför route-mappen och behöver ingen emulator. AsyncStorage mockas och töms mellan testerna om biblioteket är installerat; lagring efter omstart ingår inte i dessa grundtester. Modalens presentation, gester och tangentbord behöver fortfarande provas på Android eller iOS.

Vid bedömning kör läraren originaltesterna mot den commit som eleven lämnar in. En inlämning i lärplattformen startar inte automatiskt en GitHub-körning. Testerna är stöd för bedömningen och täcker inte hela VG-uppgiften.
