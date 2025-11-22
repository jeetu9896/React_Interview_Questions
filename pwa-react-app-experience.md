# Progressive Web Apps (PWAs): 100 Questions with Full Explanations and Code

---

## Fundamentals (Q1–20)

### 1. What is a Progressive Web App (PWA)?
A PWA is a web application that uses modern web capabilities—service workers, HTTPS, and a web app manifest—to deliver an app-like experience. PWAs are:
- Reliable (work offline or on flaky networks),
- Fast (optimized loading and responsiveness),
- Engaging (installable, push-enabled, standalone UI).

Key features: installability, offline caching, background sync, push notifications, responsive UI.

---

### 2. How do PWAs differ from traditional web applications?
- PWAs: installable, offline-ready, can send push notifications, run in standalone mode, use service workers and caching.
- Traditional web apps: require constant connectivity, not installable, limited engagement mechanisms, run in browser tab only.

PWAs blend the reach of the web with native-like capabilities.

---

### 3. Core pillars of a PWA
- Reliability: App shell cached; graceful offline fallbacks.
- Performance: Optimize main-thread work, bundle/minify, lazy load.
- Engagement: Install prompts, push notifications, background sync.

These map to Lighthouse categories and Web Vitals.

---

### 4. Advantages over native apps
- Single codebase across platforms; no app store friction.
- Lower dev/maintenance cost; instant web deployments.
- Linkable/sharable URLs; SEO benefits.
- Progressive enhancement ensures broad compatibility.

---

### 5. Role of the service worker in a PWA
A service worker is a background script acting as a programmable network proxy:
- Intercepts fetch requests to cache or modify responses.
- Enables offline capability and app shell pattern.
- Handles push notifications and background sync.
- Runs independently from the main thread, event-driven.

```js
// sw.js
self.addEventListener('install', (event) => {
  event.waitUntil(caches.open('static-v1').then((cache) =>
    cache.addAll(['/', '/index.html', '/styles.css', '/app.js', '/offline.html'])
  ));
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((cached) => cached || fetch(event.request))
  );
});
```
---

### 6. How do you make a web app installable on a `user's` home screen?

- Serve over HTTPS (secure context).

- Provide a valid manifest.json with name, icons, start_url, display.

- Register a service worker.

- Meet browser heuristics (user engagement and page criteria).

- Use beforeinstallprompt to surface a custom install UX.

```js
// sw registration
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js');
}

// install prompt UX
let deferredPrompt;
window.addEventListener('beforeinstallprompt', (e) => {
  e.preventDefault();
  deferredPrompt = e;
  installButton.addEventListener('click', async () => {
    const outcome = await deferredPrompt.prompt();
    deferredPrompt = null;
    console.log('Install outcome:', outcome.outcome);
  });
});
```
---

### 7. What is a manifest file and what is its significance in PWAs?
manifest.json describes app metadata for installability and appearance:

- name, short_name

- icons (various sizes, maskable)

- start_url, scope

- display (standalone, fullscreen, browser)

- theme_color, background_color 

It enables OS-level integration (icon, splash screen, title) when installed.

```json
{
  "name": "Awesome PWA",
  "short_name": "Awesome",
  "start_url": "/?src=a2hs",
  "scope": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#0d9488",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" },
    { "src": "/icons/icon-maskable.png", "sizes": "512x512", "type": "image/png", "purpose": "any maskable" }
  ]
}
```
---

### 8. What is the purpose of the ‘Add to Home Screen’ feature?
It lets users install the PWA to their device launcher with an icon and standalone experience, increasing engagement, repeat usage, and performance perception (fast startup).

---

### 9. How can you detect if your web app has been added to the user's home screen?
- Listen to appinstalled event.

- Detect standalone with matchMedia('(display-mode: standalone)').

- On iOS Safari, check navigator.standalone.

```js
window.addEventListener('appinstalled', () => console.log('App installed'));
const isStandalone = window.matchMedia('(display-mode: standalone)').matches || window.navigator.standalone;
```
---

### 10. Explain how PWAs achieve offline functionality.
Service workers cache resources (app shell + dynamic data) and intercept requests to serve cached content or fallbacks when offline. IndexedDB stores structured data for offline reads/writes.

```js
self.addEventListener('fetch', (event) => {
  event.respondWith((async () => {
    const cache = await caches.open('dynamic-v1');
    const cached = await cache.match(event.request);
    if (cached) return cached;
    try {
      const res = await fetch(event.request);
      cache.put(event.request, res.clone());
      return res;
    } catch {
      return caches.match('/offline.html');
    }
  })());
});
```
---

### 11. What are the security requirements for a PWA?
- HTTPS is mandatory for service workers and modern APIs.

- Apply Content Security Policy (CSP) to prevent XSS.

- Handle sensitive data carefully; avoid storing secrets in localStorage.

- Proper CORS headers for cross-origin requests.

- Validate inputs and sanitize outputs.

---

### 12. Can you delineate between App Shell and content in a PWA context?
- App Shell: Minimal static assets required to render skeleton UI (HTML/CSS/JS), cached on install for instant load.

- Content: Dynamic data loaded from APIs or local storage; updated independently.

App shell ensures quick startup even offline; content fills in later.

---

### 13. How does a PWA function on a low-bandwidth or offline network?
- Serve cached app shell instantly.

- Use adaptive strategies: smaller images (WebP/AVIF), reduced payloads, pagination, timeouts + retries.

- Degrade gracefully: show cached content, skeletons, or offline messages.

```js
async function fetchWithTimeout(url, timeout = 4000) {
  const controller = new AbortController();
  const t = setTimeout(() => controller.abort(), timeout);
  try { return await fetch(url, { signal: controller.signal }); }
  finally { clearTimeout(t); }
}
```
---

### 14. What are push notifications in the context of PWAs?
Push notifications are messages delivered via service workers from a server through web push services, even when the app isn’t open. They require user permission and a push subscription.

```js
const reg = await navigator.serviceWorker.ready;
const sub = await reg.pushManager.subscribe({
  userVisibleOnly: true,
  applicationServerKey: '<BASE64_VAPID_PUBLIC_KEY>'
});
```
---

### 15. Can you explain the concept of background sync in PWAs?
Background Sync defers actions (e.g., submitting forms) until the device has stable connectivity. It improves reliability and reduces failures.

```js
// In page
navigator.serviceWorker.ready.then((reg) => reg.sync.register('sync-outbox'));

// In SW
self.addEventListener('sync', (event) => {
  if (event.tag === 'sync-outbox') event.waitUntil(processOutbox());
});
```
---


### 16. What is a service worker lifecycle?
- Install: cache assets.

- Waiting: new SW waits until existing SW is not controlling clients.

- Activate: cleanup old caches; claim clients.

- Running: handles fetch, push, sync; can be terminated when idle.

```js
self.addEventListener('install', () => self.skipWaiting());
self.addEventListener('activate', (e) => e.waitUntil(self.clients.claim()));
```
---

### 17. How do you handle compatibility issues across different browsers for PWAs?
- Feature detection ('serviceWorker' in navigator, 'PushManager' in window).

- Progressive enhancement: baseline works everywhere; advanced features if supported.

- Polyfills for missing APIs where appropriate.

- Conditional code paths; graceful fallbacks.

---

### 18. How would you test a PWA?
- Lighthouse audits for performance, accessibility, best practices, PWA.

- DevTools Application tab: SW, cache, manifest verification.

- Network throttling and offline simulation.

- Cross-browser/device testing; remote debugging.

---

### 19. What is the role of HTTPS in PWAs?
Enables service workers and secure APIs (push, geolocation, etc.), protects integrity/confidentiality, and is required by browsers for sensitive capabilities.

----

### 20. What are the limitations of PWAs?
- API gaps vs native (background tasks, certain sensors).

- iOS restrictions (historically limited push/sync/storage).

- App store presence requires packaging/wrappers.

- Browser inconsistencies necessitate testing and fallbacks.
