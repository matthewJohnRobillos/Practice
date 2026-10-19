# React Hooks Practical Exam

CSIT340 Midterm Practical Exam. I built six small React apps, one per exam, using only React Hooks for state logic. I did not install any state management library.

## How to Run

```bash
npm install
npm run dev
npm test
```

I turned StrictMode off so the Exam 5 console logs show one entry per real render instead of doubled ones.

## Completed Requirements

| Exam | Hooks I Used | What I Built |
|------|--------------|--------------|
| 1. Smart Task Counter | `useState` | `count` and `target` state with Increment, Decrement, and Reset. The count never goes below 0. Plus two uses two queued functional updates. The status is derived (Not started, In progress, Target reached). Increment and Plus two are disabled at the target. |
| 2. Live Search and Timer | `useEffect`, `useState` | A delayed search effect that cancels the previous search, a one second timer with cleanup on unmount, and the result count shown in the document title. |
| 3. Accessible Form Toolkit | `useRef`, `useState` | Controlled name, email, and password fields with three input refs. Validation shows messages and focuses the first invalid field (name, then email, then password). The previous submitted email is kept in a ref. |
| 4. Application Preferences | `useContext`, `useState` | One context and provider holding theme, fontSize, and their update functions. Includes a light/dark toggle and font sizes 14/16/18. `usePreferences` throws an error when used outside the provider. |
| 5. Product Explorer | `useMemo`, `useCallback` | 600 products with search, category filter, and price sort. The list is memoized, the select callback is memoized, and `ProductList` uses `React.memo`. An unrelated counter proves it doesn't cause re-renders. All dependency arrays are complete, and analytics run inside `useEffect`. |
| 6. Issue Tracker | `useReducer`, custom Hook | Five reducer actions with a pure, immutable reducer. The default case returns state. A `usePersistentReducer` Hook saves to `localStorage`. All/Open/Done views show counts. Three reducer tests. |

## Known Limitations

- **Exam 1:** The target input ignores non-numeric input and cannot go below 1. The count is capped at the target, and lowering the target below the count lowers the count to match.
- **Exam 3:** The form does not send data anywhere.
- **Exam 6:** Storage errors are ignored and the app falls back to the initial state.

## Test Steps

**Exam 1**
1. Target 5 and count 0 shows Not started.
2. Click Increment and it shows In progress.
3. At count 4, click Plus two. The count becomes 5, it shows Target reached, and both buttons disable.
4. At 0, Decrement stays at 0.
5. Reset returns the count to 0 and re-enables the buttons.

**Exam 2**
1. Type quickly ("d", "da", "dan") and only the final result appears.
2. The seconds increase once per second.
3. Click Unmount timer, then Mount timer. It restarts at 0 with only one interval.
4. The tab title shows the result count.

**Exam 3**
1. Submit empty and focus goes to name, then email, then password as each one is fixed.
2. Submit a valid form and the previous email shows "none yet".
3. Submit another and the first email shows as the previous one.
4. The click counter updates on screen.

**Exam 4**
1. Toggle the theme and the Dashboard and ArticlePreview change together.
2. Choose 14, 16, or 18 px and the article text size changes.
3. Use `usePreferences` outside the provider to see the error.

**Exam 5**
1. Open the console.
2. Search, change category, or change sort and the console logs filtering and a list render.
3. The unrelated counter logs nothing from the list or filter.
4. Selecting a product re-renders the list.

**Exam 6**
1. Add, toggle, delete, filter, and Clear done.
2. Refresh the page and the issues remain.
3. Run `npm test` and three tests pass.

## Debugging Cases

### Exam 1
- **Problem:** Calling `setCount(count + 1)` twice reads the same captured `count` from one render, so both calls queue the same value and the result only rises by one.
- **Fix:** Use `setCount(c => c + 1)` twice. Each updater receives the latest queued value.

### Exam 2
- **Problem:** An effect that depends on the state it updates re-runs forever.
  **Fix:** Depend only on the real input (`[query]`).
- **Problem:** `setSeconds(seconds + 1)` in an effect with `[]` captures 0 and stays at one.
  **Fix:** Use `setSeconds(s => s + 1)`.
- **Cleanup:** `clearTimeout` stops an older search from writing results after a newer query. `clearInterval` prevents duplicate timers after unmount or remount.

### Exam 3
- **Problem:** Displaying `clicks.current` never updates, because changing a ref does not schedule a render.
- **Fix:** Use `useState` for anything shown on screen. I only read refs inside event handlers.

### Exam 4
- **Problem:** With no provider, `useContext` returns the default value (`null`).
  **Fix:** Wrap the tree in `PreferencesProvider`. `usePreferences` throws a clear error if it is missing.
- **Problem:** Two separate context objects never connect.
  **Fix:** Create the context once in `PreferencesContext.js` and import it everywhere.
- **Benefit:** Context removes intermediate prop passing, because the provider owns the state and any descendant reads it directly.

### Exam 5
- **Stale `useMemo`:** I added the missing `query`. Dependencies are now `[query, category, sortOrder]`.
- **Stale `useCallback`:** I added the missing `selectedId`. Dependencies are now `[selectedId]`.
- **Side effect in `useMemo`:** Analytics inside `useMemo` is a side effect, so I moved it to `useEffect`.

### Exam 6
- **Mutation:** `issue.done = !issue.done` and `push` mutate state. I replaced them with new state and new issue objects using spread, `map`, and `filter`.
- **Action shape:** The reducer reads `action.payload`, so dispatch sends `{ type, payload: { ... } }`, not `action.title`.
- **Default branch:** It returned `undefined`. I fixed it with `default: return state`.

## Memoization Not Needed

Examples: `useMemo(() => first + ' ' + last, [first, last])` or filtering a 10 item list. The memo overhead equals the cost of the calculation, and no memoized child depends on the reference, so memoizing gains nothing.

## Demonstration Bug

In Exam 1, Plus two raised the count by only one because both `setCount(count + 1)` calls used the same captured value. Changing both to functional updates fixed it.
