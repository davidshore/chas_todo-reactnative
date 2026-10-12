# Todo-app i React Native med Expo Router

I dagens uppgift övar vi på navigering och delat state i React Native. Bygg en todo-app med tre skärmar: **Home**, **Detail** och **Add**. Add ska öppnas som en modal.

Vi använder **Expo Router**, som är [Expos rekommenderade val för nya Expo-projekt](https://docs.expo.dev/develop/app-navigation/). Routes skapas från filer, och projektets standardmall innehåller Expo Router och TypeScript.

![Exempel på todo-appens gränssnitt](todo-app.png)

## Sätt upp projektet

Du behöver Node.js i en aktuell LTS-version och en mobil med Expo Go eller en Android-emulator/iOS-simulator.

```bash
npx create-expo-app@latest todo-app
cd todo-app
```

Använd standardmallen med Expo Router och TypeScript. Följ [Expos aktuella startguide](https://docs.expo.dev/tutorial/create-your-first-app/) om verktyget frågar efter SDK-version eller om din version av Expo Go kräver en viss SDK.

Innan du börjar skriva egen kod kan du köra följande i det nyskapade projektet för att ta bort mallens exempelskärmar:

```bash
npm run reset-project
npx expo start
```

Öppna appen med Expo Go eller starta emulatorn/simulatorn. Du kan även använda webben under utvecklingen, men kontrollera navigation och modal på Android eller iOS inför inlämningen.

## Hur du klarar uppgiften

### 1. Skapa tre routes och en stack

Använd följande struktur. Varje skärmfil ska exportera sin komponent med `export default`.

```text
src/
  app/
    _layout.tsx         # Gemensam layout och stack
    index.tsx           # Home: /
    add.tsx             # Add: /add
    todos/
      [id].tsx          # Detail: /todos/:id
  context/
    TodosContext.tsx    # Delat state och funktioner
  components/           # Återanvändbara UI-komponenter
```

Om din mall använder `app/` direkt i projektroten kan du använda samma struktur där. Välj en plats för routes; skapa inte både `app/` och `src/app/`. Context, hjälpfunktioner och vanliga komponenter ska ligga utanför route-mappen.

Konfigurera stacken i `_layout.tsx`. När du har skapat din `TodosProvider` i nästa steg kan layouten se ut så här:

```tsx
import { Stack } from 'expo-router';
import { TodosProvider } from '../context/TodosContext';

export default function RootLayout() {
  return (
    <TodosProvider>
      <Stack>
        <Stack.Screen name="index" options={{ title: 'Mina todos' }} />
        <Stack.Screen name="todos/[id]" options={{ title: 'Detaljer' }} />
        <Stack.Screen
          name="add"
          options={{ title: 'Lägg till todo', presentation: 'modal' }}
        />
      </Stack>
    </TodosProvider>
  );
}
```

Läs mer om [layouts](https://docs.expo.dev/router/basics/navigation-layouts/) och [modaler i Expo Router](https://docs.expo.dev/router/advanced/modals/).

### 2. Dela todo-state mellan skärmarna

Skapa en `TodosProvider` med React Context och `useState`, samt en egen hook, till exempel `useTodos()`. Providern ska ligga runt stacken så att alla tre skärmar använder samma todo-lista.

Varje todo ska ha ett stabilt, unikt ID, en titel och en status:

```ts
type Todo = {
  id: string;
  title: string;
  done: boolean;
};
```

Låt ditt context erbjuda listan och funktioner som `addTodo(title)`, `toggleTodo(id)` och `deleteTodo(id)`. Uppdatera state utan att mutera arrayen eller objekten. Använd funktionella uppdateringar, till exempel `setTodos(previous => previous.filter(todo => todo.id !== id))` för borttagning.

Todo-listan ska behållas när du byter skärm. I grunduppgiften räcker det att lagra den i minnet; den får återställas när appen startas om.

Läs om [delat state](https://react.dev/learn/sharing-state-between-components), [Context](https://react.dev/learn/passing-data-deeply-with-context) och [arrayuppdateringar i React](https://react.dev/learn/updating-arrays-in-state).

### 3. Visa listan på Home

- Läs todos från ditt context och visa dem med `FlatList`.
- Använd todo-ID som nyckel: `keyExtractor={item => item.id}`.
- Visa ett tydligt meddelande när listan är tom.
- Använd `Pressable` för varje rad och visa återkoppling när användaren trycker. Ge knappar en tydlig text och lämplig `accessibilityRole`.
- Lägg till en knapp som öppnar `/add`.

### 4. Öppna Detail med ett ID

Navigera från en rad på Home till dess detaljsida, till exempel i radens `onPress`:

```tsx
router.push({ pathname: '/todos/[id]', params: { id: item.id } });
```

Importera `router` från `expo-router`. I `todos/[id].tsx` läser du ID med `useLocalSearchParams` från samma paket:

```tsx
const { id } = useLocalSearchParams<{ id: string }>();
const { todos } = useTodos();
const todo = todos.find(item => item.id === id);
```

Importera även din egen `useTodos`-hook. Visa titeln och statusen, och lägg till knappar för att växla mellan genomförd/ej genomförd och ta bort todon. Anropa funktionerna i ditt context. Efter borttagning ska användaren komma tillbaka till Home.

**Skicka bara ID i route-parametrar.** Skicka inte hela todo-objekt, `setTodos` eller andra funktioner. Detaljsidan ska läsa aktuell data från delat state. Det undviker gamla kopior av data och håller parametrarna möjliga att representera i en URL.

Hantera också ett ID som inte finns: visa exempelvis ”Todon finns inte” och en väg tillbaka till Home. Om ingen föregående skärm finns kan du använda `router.replace('/')`; annars kan du använda `router.back()`.

Läs mer om [navigering](https://docs.expo.dev/router/basics/navigation/) och [route-parametrar](https://docs.expo.dev/router/reference/url-parameters/).

### 5. Lägg till en todo i en modal

- Öppna `/add` från Home med `Link` eller `router.push('/add')`. Stackens `presentation: 'modal'` styr hur skärmen visas.
- Skapa ett formulär med `TextInput` och en Spara-knapp. Formulärets text kan ligga i lokalt state på Add.
- Trimma titeln och visa ett begripligt felmeddelande om den är tom. Spara inte tomma titlar.
- Lägg till todon via ditt context och stäng sedan modalen så att den nya todon syns på Home.
- Lägg till en Avbryt-knapp som stänger utan att spara. Hantera även fallet där `/add` öppnas direkt och det saknas en föregående skärm.
- Se till att tangentbordet inte täcker fältet eller Spara-knappen, och att innehållet inte hamnar bakom systemets kanter. Använd vid behov `KeyboardAvoidingView` och `react-native-safe-area-context`.

## Kontrollera innan du lämnar in

- Du kan skapa flera todos, även med samma titel, och varje todo har ett eget ID.
- En tom eller enbart blankstegsfylld titel går inte att spara.
- Varje rad öppnar rätt detaljsida.
- Statusändringar syns på både Detail och Home utan att appen laddas om.
- Borttagning tar bort rätt todo och återgår till Home.
- Add öppnas som modal på Android eller iOS. Spara och Avbryt fungerar, liksom plattformens vanliga sätt att gå tillbaka/stänga modalen.
- Tom lista och okänt todo-ID ger ett begripligt gränssnitt.
- TypeScript-kontrollen går igenom: `npx tsc --noEmit`.

## Hur du lämnar in

1. Skapa ett tomt repo på GitHub för din app och ladda upp projektet dit.
2. Lägg till en README med startinstruktioner, vilken plattform du har provat på och eventuella kända begränsningar.
3. Klicka på uppgiften i Canvas och ange länken till ditt repo.

Om du behöver sätta upp Git och koppla projektet till ditt nya repo:

```bash
git init
git add .
git commit -m "Build todo app with Expo Router"
git branch -M main
git remote add origin <adressen-till-ditt-repo>
git push -u origin main
```

`create-expo-app` kan redan ha skapat Git-repot. Om `origin` redan finns behöver du inte lägga till det igen. Behåll projektets `.gitignore` så att exempelvis `node_modules` inte checkas in.

## Efter uppgiften

Du ska kunna skapa filbaserade routes, konfigurera en stack och en modal, läsa route-parametrar och dela state mellan skärmar utan att skicka funktioner genom navigationen.

## Extrauppgifter

- Lägg till en Genomfört-sektion under todo-listan. Byt ut `FlatList` mot `SectionList`.
- Spara todos lokalt så att de finns kvar efter omstart. Använd exempelvis AsyncStorage, installera med `npx expo install @react-native-async-storage/async-storage` och hantera inläsning innan du skriver tillbaka data.
- Lägg till möjlighet att redigera en todos titel.
