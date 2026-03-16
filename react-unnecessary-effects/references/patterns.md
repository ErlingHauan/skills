# Unnecessary Effect Patterns — Full Reference

This file contains detailed before/after examples for each anti-pattern. Read the relevant section when you encounter a specific pattern in user code.

## Table of Contents

1. [Derived state via Effect](#1-derived-state-via-effect)
2. [Caching expensive calculations](#2-caching-expensive-calculations)
3. [Resetting all state on prop change](#3-resetting-all-state-on-prop-change)
4. [Adjusting some state on prop change](#4-adjusting-some-state-on-prop-change)
5. [Sharing logic between event handlers](#5-sharing-logic-between-event-handlers)
6. [POST requests in Effects](#6-post-requests-in-effects)
7. [Chains of Effects](#7-chains-of-effects)
8. [App initialization](#8-app-initialization)
9. [Notifying parent of state changes](#9-notifying-parent-of-state-changes)
10. [Passing data up to parent](#10-passing-data-up-to-parent)
11. [External store subscriptions](#11-external-store-subscriptions)
12. [Data fetching race conditions](#12-data-fetching-race-conditions)

---

## 1. Derived state via Effect

The value is computable from existing props or state. There is no reason for it to live in `useState`.

**Before:**
```jsx
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  const [fullName, setFullName] = useState('');

  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
}
```

**After:**
```jsx
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  const fullName = firstName + ' ' + lastName;
}
```

**Why:** The Effect version renders twice — first with the stale `fullName`, then again after the Effect updates it. The direct computation is faster, simpler, and keeps values in sync.

---

## 2. Caching expensive calculations

When a computation is genuinely slow, use `useMemo` — not an Effect that writes to state.

**Before:**
```jsx
function TodoList({ todos, filter }) {
  const [visibleTodos, setVisibleTodos] = useState([]);

  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);
}
```

**After:**
```jsx
function TodoList({ todos, filter }) {
  const visibleTodos = useMemo(
    () => getFilteredTodos(todos, filter),
    [todos, filter]
  );
}
```

**When to reach for `useMemo`:** Measure first. Unless you're creating or looping through thousands of objects, it's probably not expensive. Use `console.time` / `console.timeEnd` around the computation — if it's under 1ms, skip the memo.

**Note:** If you are not certain the computation is expensive, just compute it directly without `useMemo`. Only add memoization when you have evidence it matters:
```jsx
const visibleTodos = getFilteredTodos(todos, filter);
```

---

## 3. Resetting all state on prop change

When a component should fully reset its internal state because it now represents a different entity (different user, different conversation, etc.), use `key`.

**Before:**
```jsx
function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  useEffect(() => {
    setComment('');
  }, [userId]);
}
```

**After:**
```jsx
function ProfilePage({ userId }) {
  return <Profile userId={userId} key={userId} />;
}

function Profile({ userId }) {
  const [comment, setComment] = useState('');
  // State resets automatically when key changes
}
```

**Why:** Passing a different `key` tells React this is a conceptually different component. React destroys and recreates it, resetting all state — including nested children. The Effect approach requires manually clearing every piece of state and misses nested components.

---

## 4. Adjusting some state on prop change

When only *part* of the state should change in response to a prop change, try to derive the value during render instead of "adjusting" it with an Effect.

**Before:**
```jsx
function List({ items }) {
  const [selection, setSelection] = useState(null);

  useEffect(() => {
    setSelection(null);
  }, [items]);
}
```

**After:**
```jsx
function List({ items }) {
  const [selectedId, setSelectedId] = useState(null);
  const selection = items.find(item => item.id === selectedId) ?? null;
}
```

**Why:** Instead of resetting selection when items change, store the *ID* and derive whether it's still valid. If the selected item is still in the list, it stays selected. If not, `selection` becomes `null` — no Effect needed, and the behavior is actually better (preserves selection across partial list changes).

---

## 5. Sharing logic between event handlers

When two event handlers need to do the same thing, extract a shared function — don't route through an Effect.

**Before:**
```jsx
function ProductPage({ product, addToCart }) {
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
}
```

**After:**
```jsx
function ProductPage({ product, addToCart }) {
  function buyProduct() {
    addToCart(product);
    showNotification(`Added ${product.name} to the shopping cart!`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
}
```

**Why:** The notification should fire because the user *clicked a button*, not because the component rendered. The Effect version shows the notification on page reload if the cart is persisted.

---

## 6. POST requests in Effects

Form submissions belong in event handlers. Analytics tracking belongs in Effects.

**Before:**
```jsx
function Form() {
  const [jsonToSubmit, setJsonToSubmit] = useState(null);

  useEffect(() => {
    if (jsonToSubmit !== null) {
      post('/api/register', jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
}
```

**After:**
```jsx
function Form() {
  // Analytics: runs because component was displayed — Effect is correct
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  // Submission: runs because user pressed submit — belongs in handler
  function handleSubmit(e) {
    e.preventDefault();
    post('/api/register', { firstName, lastName });
  }
}
```

**The deciding question:** Is the reason for the request that the user *did something*, or that the component *appeared on screen*?

---

## 7. Chains of Effects

Multiple Effects that each set state in response to another piece of state changing create cascading renders.

**Before:**
```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

  useEffect(() => {
    if (card !== null && card.gold) {
      setGoldCardCount(c => c + 1);
    }
  }, [card]);

  useEffect(() => {
    if (goldCardCount > 3) {
      setRound(r => r + 1);
      setGoldCardCount(0);
    }
  }, [goldCardCount]);

  useEffect(() => {
    if (round > 5) {
      setIsGameOver(true);
    }
  }, [round]);
}
```

**After:**
```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // Derived — no need for state
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) throw Error('Game already ended.');

    setCard(nextCard);
    if (nextCard.gold) {
      if (goldCardCount < 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        setRound(round + 1);
        if (round === 5) alert('Good game!');
      }
    }
  }
}
```

**Why:** The chain version causes up to 3 unnecessary re-renders. Computing all next state in the event handler results in a single render and makes the logic easier to follow.

**Exception:** Chains of Effects *are* appropriate when each Effect synchronizes with a different external system (e.g., cascading dropdowns where each one fetches from the network based on the previous selection).

---

## 8. App initialization

Code that should run once per app load — not once per component mount.

**Before:**
```jsx
function App() {
  useEffect(() => {
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []);
}
```

**Problem:** In development Strict Mode, this runs twice — which can cause issues like invalidating auth tokens.

**After (option A — guard variable):**
```jsx
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
}
```

**After (option B — module-level init):**
```jsx
if (typeof window !== 'undefined') {
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

---

## 9. Notifying parent of state changes

When a child component changes its own state and needs to tell the parent, call the parent callback in the same event handler — not in an Effect.

**Before:**
```jsx
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  useEffect(() => {
    onChange(isOn);
  }, [isOn, onChange]);

  function handleClick() {
    setIsOn(!isOn);
  }
}
```

**After (option A — call in handler):**
```jsx
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  function updateToggle(nextIsOn) {
    setIsOn(nextIsOn);
    onChange(nextIsOn);
  }

  function handleClick() {
    updateToggle(!isOn);
  }
}
```

**After (option B — fully controlled):**
```jsx
function Toggle({ isOn, onChange }) {
  function handleClick() {
    onChange(!isOn);
  }
}
```

**Why:** The Effect version causes two render passes — first the child renders with new state, then the parent re-renders after the Effect fires. Calling both in the event handler lets React batch them into one render.

Option B (lifting state up) is often the cleanest solution.

---

## 10. Passing data up to parent

If a child fetches data and passes it to the parent via an Effect, the data flow becomes hard to trace.

**Before:**
```jsx
function Parent() {
  const [data, setData] = useState(null);
  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useSomeAPI();
  useEffect(() => {
    if (data) onFetched(data);
  }, [onFetched, data]);
}
```

**After:**
```jsx
function Parent() {
  const data = useSomeAPI();
  return <Child data={data} />;
}
```

**Why:** React's data flow is top-down. Fetching in the parent and passing down makes the flow predictable and debuggable.

---

## 11. External store subscriptions

Subscribing to browser APIs or external stores is a legitimate synchronization need, but `useSyncExternalStore` is cleaner than a manual Effect.

**Before:**
```jsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);

  useEffect(() => {
    function updateState() {
      setIsOnline(navigator.onLine);
    }
    updateState();
    window.addEventListener('online', updateState);
    window.addEventListener('offline', updateState);
    return () => {
      window.removeEventListener('online', updateState);
      window.removeEventListener('offline', updateState);
    };
  }, []);

  return isOnline;
}
```

**After:**
```jsx
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {
  return useSyncExternalStore(
    subscribe,
    () => navigator.onLine,      // client value
    () => true                    // server value
  );
}
```

---

## 12. Data fetching race conditions

Fetching data in an Effect is often appropriate, but you must handle race conditions when the inputs change faster than the network responds.

**Before (buggy):**
```jsx
useEffect(() => {
  fetchResults(query, page).then(json => {
    setResults(json);
  });
}, [query, page]);
```

**After (with cleanup):**
```jsx
useEffect(() => {
  let ignore = false;

  fetchResults(query, page).then(json => {
    if (!ignore) {
      setResults(json);
    }
  });

  return () => {
    ignore = true;
  };
}, [query, page]);
```

**Why:** Without cleanup, if the user types "hello" quickly, requests for "h", "he", "hel", "hell", and "hello" all fire. Responses may arrive out of order, and whichever arrives last wins — which might not be "hello". The `ignore` flag ensures only the latest request's response is used.

**Better yet:** Extract into a custom hook like `useData(url)` that encapsulates the cleanup pattern, so every call site gets race condition handling for free.
