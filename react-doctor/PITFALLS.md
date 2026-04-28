# React Pitfalls Catalog

Reference for the `react-doctor` skill. Each entry: detection signal, fix recipe, minimal example.

Sources: [react.dev/learn/you-might-not-need-an-effect](https://react.dev/learn/you-might-not-need-an-effect) and [developerway.com/posts/react-re-renders-guide](https://www.developerway.com/posts/react-re-renders-guide), plus common pitfalls outside both.

Categories: Effects · State · Re-renders · Lists · Memoization · Context · Refs

---

## Effects

### 1. Derived state via useEffect [low]

**Detect:** `useState` for value `X`, plus `useEffect(() => setX(...))` deriving from other state/props.
**Fix:** Compute `X` during render. Delete the state and the effect.

```tsx
// Before
const [total, setTotal] = useState(0);
useEffect(() => setTotal(items.reduce((s, i) => s + i.price, 0)), [items]);

// After
const total = items.reduce((s, i) => s + i.price, 0);
```

### 2. Caching expensive calc via useEffect [low]

**Detect:** `useEffect` runs an expensive function, then `setState(result)`.
**Fix:** Replace with `useMemo`. Drop the state.

```tsx
// Before
const [filtered, setFiltered] = useState([]);
useEffect(() => setFiltered(filter(items, query)), [items, query]);

// After
const filtered = useMemo(() => filter(items, query), [items, query]);
```

### 3. Resetting all state on prop change [medium]

**Detect:** `useEffect(() => { setX(initial); setY(initial); }, [propId])`.
**Fix:** Pass `key={propId}` to the component. React re-mounts; state resets naturally.

```tsx
// Before
function Profile({ userId }) {
  const [tab, setTab] = useState('home');
  useEffect(() => setTab('home'), [userId]);
}
<Profile userId={userId} />

// After
function Profile({ userId }) {
  const [tab, setTab] = useState('home');
}
<Profile key={userId} userId={userId} />
```

### 4. Adjusting state on prop change [medium]

**Detect:** `useEffect` with `[prop]` deps that conditionally calls `setState`.
**Fix:** Compare against previous prop during render and update directly, or compute the value instead of storing it.

```tsx
// Before
const [selection, setSelection] = useState(null);
useEffect(() => setSelection(null), [items]);

// After
const [prevItems, setPrevItems] = useState(items);
const [selection, setSelection] = useState(null);
if (items !== prevItems) {
  setPrevItems(items);
  setSelection(null);
}
```

### 5. Event-handler logic in Effect [low]

**Detect:** `useEffect` body branches on a flag set by a handler (e.g., `if (jsonToSubmit) post(jsonToSubmit)`).
**Fix:** Move the logic into the event handler.

```tsx
// Before
const [jsonToSubmit, setJsonToSubmit] = useState(null);
useEffect(() => { if (jsonToSubmit) post(jsonToSubmit); }, [jsonToSubmit]);
function onClick() { setJsonToSubmit(buildJson()); }

// After
function onClick() { post(buildJson()); }
```

### 6. Chains of Effects [high]

**Detect:** Multiple `useEffect`s where one's deps include state set by another.
**Fix:** Collapse the chain: compute during render, or do all work in one handler.

```tsx
// Before — A sets B sets C
useEffect(() => setB(deriveB(a)), [a]);
useEffect(() => setC(deriveC(b)), [b]);

// After
const b = deriveB(a);
const c = deriveC(b);
```

### 7. App-init logic in `useEffect(..., [])` [medium]

**Detect:** Empty-dep effect calling `loadAuth/checkToken/init` in `App` or root component.
**Fix:** Run once at module top-level, or in the entry file before mount.

```tsx
// Before
function App() {
  useEffect(() => { loadAuthFromStorage(); }, []);
}

// After
loadAuthFromStorage();
function App() { /* ... */ }
```

### 8. Notifying parent via Effect [medium]

**Detect:** `useEffect(() => onChange(value), [value])`.
**Fix:** Either lift state to the parent, or call `onChange` in the handler alongside `setState`.

```tsx
// Before
const [open, setOpen] = useState(false);
useEffect(() => onToggle(open), [open]);

// After
function handleToggle(next) {
  setOpen(next);
  onToggle(next);
}
```

### 9. Passing fetched data up via Effect [high]

**Detect:** Child fetches data, then `useEffect(() => onFetched(data), [data])`.
**Fix:** Move the fetch up to the parent. Pass data down.

### 10. Fetching without cleanup (race condition) [medium]

**Detect:** `useEffect` with `fetch(...).then(setX)` and no `ignore` flag or `AbortController`.
**Fix:** Add an `ignore` cleanup, or adopt a data library (React Query, SWR).

```tsx
// Before
useEffect(() => { fetch(url).then(r => r.json()).then(setData); }, [url]);

// After
useEffect(() => {
  let ignore = false;
  fetch(url).then(r => r.json()).then(d => { if (!ignore) setData(d); });
  return () => { ignore = true; };
}, [url]);
```

### 11. Manual subscribe-via-Effect [medium]

**Detect:** `useEffect` adding `addEventListener` / store subscription + `setState`.
**Fix:** Use `useSyncExternalStore` for external store subscriptions.

```tsx
// Before
const [online, setOnline] = useState(navigator.onLine);
useEffect(() => {
  const onChange = () => setOnline(navigator.onLine);
  window.addEventListener('online', onChange);
  window.addEventListener('offline', onChange);
  return () => {
    window.removeEventListener('online', onChange);
    window.removeEventListener('offline', onChange);
  };
}, []);

// After
const online = useSyncExternalStore(
  cb => {
    window.addEventListener('online', cb);
    window.addEventListener('offline', cb);
    return () => {
      window.removeEventListener('online', cb);
      window.removeEventListener('offline', cb);
    };
  },
  () => navigator.onLine,
  () => true
);
```

### 12. Effect with no deps array [medium]

**Detect:** `useEffect` called with a single argument (no second array).
**Fix:** Add a dependency array. Mount-only → `[]`. Otherwise list every value the effect reads.

```tsx
// Before — runs after every render
useEffect(() => {
  document.title = `Hello ${name}`;
});

// After
useEffect(() => {
  document.title = `Hello ${name}`;
}, [name]);
```

### 13. Unstable literal in deps array [medium]

**Detect:** Dependency array contains an inline object/array literal, or a value (object, array, function) constructed in render scope without `useMemo`/`useCallback`.
**Fix:** Depend on primitives, or memoize the constructed value.

```tsx
// Before — `filter` is a new object every render → effect re-runs every render
const filter = { pageSize, sort };
useEffect(() => {
  fetchData(filter);
}, [filter]);

// After
useEffect(() => {
  fetchData({ pageSize, sort });
}, [pageSize, sort]);
```

---

## State

### 14. Direct state mutation [low]

**Detect:** `.push`/`.splice`/`.sort`/`.pop`/`.shift`/`.reverse`/`.unshift` called on a state value, or property assignment on a state object (`state.x = y`), followed by setter with the same reference.
**Fix:** Build a new array/object and pass that to the setter.

```tsx
// Before — same reference; React skips the re-render
items.push(newItem);
setItems(items);

// After
setItems([...items, newItem]);
```

---

## Re-renders

### 15. State placed too high in tree [medium]

**Detect:** `useState` in a large parent whose state is read by only one small subtree.
**Fix:** Extract the subtree into its own component; move the state inside it.

### 16. Stateful custom hook in heavy parent [medium]

**Detect:** `useModal`, `useResize`, `useScrollPosition`, etc., called in a component that renders many siblings.
**Fix:** Encapsulate the hook plus its consumers in a small dedicated component. Only that subtree re-renders on state change.

### 17. Component defined inside another component [medium]

**Detect:** `function Foo` or `const Foo = () => …` declared inside another component body.
**Fix:** Hoist to module scope. Pass any captured values as props.

```tsx
// Before
function Page({ items }) {
  function Item({ x }) { return <li>{x}</li>; }
  return <ul>{items.map(i => <Item x={i} key={i.id} />)}</ul>;
}

// After
function Item({ x }) { return <li>{x}</li>; }
function Page({ items }) {
  return <ul>{items.map(i => <Item x={i} key={i.id} />)}</ul>;
}
```

---

## Lists

For all list-key pitfalls below: if no stable id exists on the items, skip the fix and flag the data-shape issue instead of inventing a key.

### 18. Index as key in dynamic list [low]

**Detect:** `.map((x, i) => <X key={i} …>)` where the array can reorder, insert, or delete.
**Fix:** Use a stable id (`x.id`) as key.

### 19. Missing key on list items [low]

**Detect:** `.map(...)` returning JSX without a `key`.
**Fix:** Add a stable key.

### 20. Non-stable key in list [low]

**Detect:** `key={Math.random()}`, `key={uuid()}`, `key={Date.now()}`, or any expression that returns a fresh value on each render.
**Fix:** Use a stable identifier from the data (`item.id`). Generate ids once when items are created, not during render.

```tsx
// Before — every row re-mounts on every render
{items.map(item => <Row key={Math.random()} item={item} />)}

// After
{items.map(item => <Row key={item.id} item={item} />)}
```

---

## Memoization

### 21. useCallback/useMemo on props to non-memoized child [low]

**Detect:** `useCallback` or `useMemo` whose result is passed as a prop to a child not wrapped in `React.memo`.
**Fix:** Drop the memo (it does nothing) — or memoize the child too if its re-renders are actually expensive.

### 22. Spreading props through a memoized child [low]

**Detect:** `<ChildMemo {...props} />` — props forwarded by spread to a `React.memo`-wrapped child.
**Fix:** Pass explicit props. Spread doesn't break memo by itself (`React.memo` shallow-compares each prop after JSX expansion), but it hides which props are flowing through. If any one of them is an unstable callback, inline object, or array, memo silently breaks and the cause is invisible. Listing props makes the unstable one obvious.

### 23. `React.memo` with non-memoized children prop [medium]

**Detect:** `<ParentMemo><Child/></ParentMemo>` where the children JSX is created inline.
**Fix:** Either memoize the children element with `useMemo`, wrap `Child` in `React.memo`, or rethink the boundary.

### 24. Stale closure in `useCallback([])` [medium]

**Detect:** `useCallback` reading state or props with empty `[]` deps.
**Fix:** Add the missing dependency, or use a ref to read the latest value.

```tsx
// Before
const onClick = useCallback(() => save(value), []);

// After
const onClick = useCallback(() => save(value), [value]);
```

### 25. `useMemo` for cheap values [low]

**Detect:** `useMemo` wrapping primitive math, string concat, or small inline objects/arrays not used as memoized props or hook deps.
**Fix:** Drop the memo. The bookkeeping costs more than the recomputation.

---

## Context

### 26. Unstable context value object [low]

**Detect:** `<Ctx.Provider value={{ a, b }}>` with an inline object or array literal.
**Fix:** Wrap the value in `useMemo`.

```tsx
// Before
<Ctx.Provider value={{ user, setUser }}>

// After
const value = useMemo(() => ({ user, setUser }), [user]);
<Ctx.Provider value={value}>
```

### 27. Single context mixing data + actions [high]

**Detect:** One provider exposing both `state` and `setState`/actions; many consumers re-render whenever any state changes.
**Fix:** Split into two contexts — `DataContext` (changes often) and `ApiContext` (stable setters). Consumers subscribe to only what they need.

---

## Refs

### 28. Reading `ref.current` during render [medium]

**Detect:** `ref.current` referenced inside JSX/return body for displayed values.
**Fix:** Use `useState` for renderable values. Refs don't trigger re-renders.

### 29. Ref instead of state for input/displayed value [medium]

**Detect:** Input `onChange` writes to `ref.current` and a sibling reads it for display.
**Fix:** Use `useState`.
