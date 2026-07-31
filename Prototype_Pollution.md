# Prototype Pollution — Complete Explanation

## 0. What is prototype pollution?

**Definition:** Prototype pollution is a JavaScript vulnerability where an attacker adds or overwrites a property on `Object.prototype` — the one shared "template" object that every plain object in the program automatically inherits from. Once polluted, that new property silently appears on every object in the running app, including objects the attacker never directly touched.

**How it occurs (the recipe):**
1. The app takes user-controlled input (usually JSON from a request body).
2. Somewhere in the code, that input gets merged, copied, or assigned into an existing object — often through a recursive "deep merge," `Object.assign`, or a loop like `for (let key in source)`.
3. That merge logic does **not** filter out the special key names `__proto__`, `constructor`, or `prototype`.
4. The attacker sends a payload containing one of those special keys (e.g. `{"__proto__": {"isAdmin": true}}`).
5. Because those key names have special meaning in JavaScript (they reach the shared prototype chain instead of behaving like a normal property name), the merge ends up writing onto the shared template object instead of the intended target object.
6. Any code anywhere in the app that later checks a property which doesn't exist on its own object (like `req.user.isAdmin`) now falls through to the polluted shared template — and gets the attacker's value instead of `undefined`.

**How to check for the vulnerability (as a defender/tester):**
- **Read the code:** search for recursive merge/clone/extend functions, `Object.assign(target, userInput)`, JSON deserialization into objects, or any loop using `for...in` / `Object.keys()` over user-supplied data. Check whether `__proto__`, `constructor`, and `prototype` are excluded.
- **Test it live:** send a request with a body like `{"__proto__": {"polluted": true}}` to any endpoint that accepts and stores/merges JSON (profile updates, settings, config endpoints are common targets). Then check if a brand-new, unrelated object anywhere in the app now has `.polluted === true`.
- **Quick console test** (for local/Node code you can run yourself):
  ```javascript
  const testObj = {};
  console.log(testObj.polluted); // should be undefined before the payload
  deepMerge(testObj, JSON.parse('{"__proto__":{"polluted":true}}'));
  const freshObj = {};
  console.log(freshObj.polluted); // if this prints true, the function is vulnerable
  ```
- **Automated tools:** static analysis linters (e.g. ESLint security plugins), Snyk, and dependency scanners flag known-vulnerable merge libraries. For live apps, tools like Burp Suite have prototype pollution scanning extensions.

**Examples of objects, before and after pollution:**

```javascript
// Two completely ordinary, unrelated objects
const user   = { username: "gojo", role: "farmer" };
const config = { theme: "dark" };

// Before pollution — neither has isAdmin, both correctly fall through to undefined
console.log(user.isAdmin);   // undefined
console.log(config.isAdmin); // undefined

// Attacker sends this as a JSON body to a vulnerable merge endpoint:
// { "__proto__": { "isAdmin": true } }

// After pollution — NEITHER object was directly modified,
// but both now report isAdmin as true, because the shared
// Object.prototype they both silently fall back to was changed
console.log(user.isAdmin);   // true  ⚠
console.log(config.isAdmin); // true  ⚠

// Even objects created AFTER the attack are affected
const brandNewLater = {};
console.log(brandNewLater.isAdmin); // true  ⚠
```

---

## 0.5 The shared template, walked through again (with the diagram's insight)

Every object in JavaScript is secretly linked to a shared "template" object — this is what `Object.prototype` is. Normally when you write `myObj.__proto__`, you're not creating a new property called `__proto__` on your object — you're reaching through your object and touching that shared template that every object in the whole program links back to. If a merge function blindly copies a key called `__proto__` onto a target object, it's not setting a normal property — it's editing the shared template itself. Since every object (including ones created later, in totally unrelated parts of the app, like the object representing "is this user an admin") inherits from that same template, your one malicious request corrupts a value that every object in the running server can now see.

![Prototype pollution flow diagram](./pollution-diagram.png)

**The diagram's flow:**
```
Your request body
  contains "__proto__" key
        │
        ▼
target[key] = value        (no key filtering)
        │
        ▼
Object.prototype           (shared template — isAdmin now set to true)
        │
   ┌────┼──────────────────────┐
   ▼                            ▼                          ▼
req.user (your session)   any new {} object          other users' objects
inherits isAdmin: true    inherits isAdmin: true      inherit isAdmin: true too
```
One malicious key corrupts the template every object in the server shares.

**The key insight the diagram is showing:** your request never had permission to touch anything beyond your own profile object. But because `deepMerge` blindly copies whatever key names you send, and `__proto__` is a special name that reaches through to the shared template every object inherits from, one crafted request corrupts something global — not just your own data.

That's exactly why the fix is so small: you don't need to redesign the merge logic, you just need to refuse those three special key names (`__proto__`, `constructor`, `prototype`) before the copy happens. Block the door, and the rest of the function works exactly as it did before — legitimate keys like `favoriteCrop` or `experienceLevel` still merge normally.

**What you got right:** Yes — `__proto__` doesn't create a new property on your object. It's a special name that acts like a pointer to something else, not a normal key like `favoriteCrop`.

**The correction:** It's not that "admin is an object" that gets updated. It's simpler than that — `isAdmin` is just a property name (a boolean flag), and what gets polluted is the shared template object itself (`Object.prototype`), by adding a new property called `isAdmin: true` directly onto it.

Here's the precise chain:

1. Every plain object you create in JS (`{}`) is secretly linked to one single shared object called `Object.prototype`. Think of it as: every object has an invisible thread connecting it back to this one shared object.
2. `__proto__` is the name of that thread. When you write `something.__proto__`, you're not making a property called `__proto__` — you're grabbing hold of that thread and reaching the shared object it points to.
3. So when your payload does `target.__proto__.isAdmin = true`, JavaScript reads it as: "follow the thread from `target` to the shared object, then set a new property called `isAdmin` to `true` directly on that shared object."
4. Now here's the important part: that shared object never had an `isAdmin` property before — you just added one. And because every object's thread points back to that same shared object, when the code later checks `someUser.isAdmin`, JavaScript looks at `someUser` itself first (no `isAdmin` there), then automatically follows the thread to the shared object — and finds the `isAdmin: true` you just planted.

---

## 1. The vulnerable function (deepMerge)

```javascript
function deepMerge(target, source) {
  for (let key in source) {
    if (source[key] && typeof source[key] === 'object' && !Array.isArray(source[key])) {
      if (!target[key]) target[key] = {};
      deepMerge(target[key], source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}
```

**What it does, in plain words:**
`deepMerge` takes two objects — your existing profile (`target`) and the new data you sent (`source`) — and copies every property from `source` into `target`. It loops over every key name in `source` using `for...in`, with no checks on what those key names actually are. It just trusts whatever the client sends.

That's the whole bug: it never asks "is this key name safe to copy?"

---

## 2. The patched function

```javascript
function deepMerge(target, source) {
  const DANGEROUS_KEYS = ['__proto__', 'constructor', 'prototype'];
  for (let key in source) {
    if (DANGEROUS_KEYS.includes(key)) continue;
    if (source[key] && typeof source[key] === 'object' && !Array.isArray(source[key])) {
      if (!target[key]) target[key] = {};
      deepMerge(target[key], source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}
```

**What changed:** one line — `if (DANGEROUS_KEYS.includes(key)) continue;`. Before copying any key, it checks whether that key's name is one of the three dangerous names: `__proto__`, `constructor`, or `prototype`. If it is, that key is skipped entirely. Everything else — legitimate keys like `favoriteCrop`, `experienceLevel` — merges exactly as before. The fix doesn't change the logic, it just refuses to open the door for those three specific names.

---

## 3. How every object connects to one shared object

```javascript
const user = {};
```

The instant this line runs, **two things happen, not one**:
1. A brand new, empty object is created in memory.
2. JavaScript **automatically** sets that object's hidden `__proto__` link to point at `Object.prototype`.

You never wrote any code for step 2 — it's automatic, built into the language, for every single object literal (`{}`) you ever create.

**Proof it's automatic:**
```javascript
const user = {};
console.log(user.__proto__ === Object.prototype); // true
```

**Proof it's shared (not copied):**
```javascript
const user1 = {};
const user2 = {};
const settings = {};

console.log(user1.__proto__ === user2.__proto__);   // true
console.log(user2.__proto__ === settings.__proto__); // true
```

All three completely unrelated objects — created at different times, for different purposes — point their `__proto__` link at the **exact same object in memory**. It's not three copies of a template. It's one single shared object everyone points to — like three phones all calling the same person.

---

## 4. The normal property lookup process

When you write `someObject.someProperty`, JavaScript checks in order:

1. **First** — does `someObject` itself have its own property called `someProperty`?
2. **If not found** — follow the hidden `__proto__` link to `Object.prototype` and check there.
3. **If still not found** — return `undefined` (not an error, just "doesn't exist anywhere in the chain").

**Before any attack:**
```
someUser.isAdmin
  → not found on someUser
  → check Object.prototype → not found there either
  → returns undefined (falsy) → "not admin" ✓ correct
```

**After the attack** (once `__proto__.isAdmin = true` is injected):
```
someUser.isAdmin
  → not found on someUser (still true — nothing changed here!)
  → check Object.prototype → FOUND: isAdmin = true
  → returns true → "is admin" ✗ wrong, but the code believes it
```

**Key insight:** your own user object is never touched or modified. What changes is step 2 — the fallback destination now has a property it never had before. Every object that reaches that fallback now incorrectly inherits `true` instead of correctly getting `undefined`.

---

## 5. The pollution moment, explicitly

```javascript
const user = {};
user.__proto__.isAdmin = true;
```

Line 2 doesn't touch `user` at all. It follows `user`'s link to the shared object, and adds a new property directly onto **that shared object**.

```javascript
const brandNewObject = {}; // created AFTER the pollution, totally unrelated
console.log(brandNewObject.isAdmin); // true !!
```

Even though `brandNewObject` was never near the malicious code, it gets `__proto__` wired to the same shared object automatically at creation — and that shared object is now permanently polluted (until the server restarts and rebuilds from scratch).

---

## 6. Full attack chain in the AgriWeb challenge

```
Your request body
  contains "__proto__" key
        │
        ▼
target[key] = value      (deepMerge, no key filtering)
        │
        ▼
Object.prototype         (shared template — isAdmin now set to true)
        │
   ┌────┼────────────────┐
   ▼                      ▼                      ▼
req.user             any new {} object      other users' objects
(your session)        inherits isAdmin        inherit isAdmin
inherits isAdmin: true      : true                  : true too
```

One malicious key corrupts the template that **every object in the running server shares** — not just your own profile.

---

## 7. One-sentence summary

You're not "finding and modifying an admin object." You're planting a brand-new property on the **one shared object every other object silently checks as a fallback** — and because the admin-check code (`req.user.isAdmin`) relies on that same fallback, your unprivileged session suddenly passes the check, without your own account ever actually being granted admin rights in the database.

---

## 8. Your question, answered directly

> "means js create obj and these obj shared one single obj that is proto in every js file — and if it not exist proto object then what the thing occur"

**Answer:** If `isAdmin` doesn't exist anywhere — not on your object, not on the shared `Object.prototype` — JavaScript doesn't error out. It simply returns `undefined`, which acts as `false` in a check. That's the safe, un-polluted default: nobody is admin unless explicitly set. The vulnerability exists precisely because an attacker can add `isAdmin: true` onto that normally-empty shared fallback, making the "nobody is admin by default" safety net disappear.

> "hte confusion is in my mind is how we create obje in js and how they ref or shred to protr object"

**Answer:** Every time you write `{}`, JavaScript automatically wires that new object's hidden link to one single shared object (`Object.prototype`) — you never do this manually, it happens at creation, every time, for every object, without exception. That single shared object is what gets corrupted by prototype pollution.

---

## 9. How to trigger the vulnerability

Concretely, in an app like AgriWeb's profile-merge endpoint:

1. **Register and log in** as a normal, unprivileged user — this gives you a valid session cookie tied to a low-privilege account.
2. **Find the vulnerable endpoint** — any route that takes JSON input and merges it into an existing stored object (a profile update, settings update, config update, etc.).
3. **Send a request with a polluting payload**, e.g.:
   ```json
   {
     "favoriteCrop": "wheat",
     "__proto__": { "isAdmin": true }
   }
   ```
   as the body of a `POST` to that endpoint, with your session cookie attached.
4. **If the backend's merge function doesn't filter `__proto__`/`constructor`/`prototype`**, this write reaches `Object.prototype` instead of your own profile object.
5. **Immediately re-use your same session** to hit a privileged route (e.g. `/admin`). If the authorization check reads a property like `req.user.isAdmin` that was never explicitly set on your account, it now falls through to the polluted shared template and incorrectly returns `true`.

That's the entire trigger: one JSON body with a special key name, sent to an endpoint that merges input unsafely.

---

## 10. How to check for the vulnerability

**A. Static code review (fastest, no live testing needed):**
- Search the codebase for merge/clone/deep-copy functions — grep for `for...in`, `Object.assign(`, `...spread` merges, or custom `deepMerge`/`extend`/`merge` helpers.
- Check whether each one explicitly excludes `__proto__`, `constructor`, and `prototype` before assigning keys.
- If none of those three names are filtered, the function is vulnerable.

**B. Dynamic / black-box testing (when you only have the running app):**
1. Register a low-privilege test account.
2. Send a harmless polluting payload to any input-accepting endpoint, e.g.:
   ```bash
   curl -X POST http://target/api/profile \
     -H "Content-Type: application/json" \
     -H "Cookie: auth_token=YOUR_TOKEN" \
     -d '{"__proto__":{"pollutionTest":"yes"}}'
   ```
3. Then check a completely unrelated, freshly-created object somewhere else in the app for that same property — for example, load any page/route that creates a new object and see if `pollutionTest` unexpectedly shows up, or directly test with a follow-up request to a route whose behavior would change if a boolean flag like `isAdmin` were polluted.
4. If a property you never explicitly granted your account suddenly affects app behavior (e.g. unlocking an admin route), the merge function is confirmed vulnerable.

**C. Automated tooling:**
- Dependency/static scanners (Snyk, npm audit, ESLint security plugins) flag known-vulnerable merge libraries and unsafe `for...in` patterns.
- Burp Suite has prototype-pollution-focused extensions that automate payload injection and response-diffing across endpoints.

**The one-line fix, once confirmed:** filter `__proto__`, `constructor`, and `prototype` out of every key loop in every merge function that touches user-controlled input.
