// CalXvoice Service Worker — Stale-While-Revalidate strategy
// Serves cached assets immediately for instant startup, then
// updates the cache in the background so the next launch is fresh.
// Currency API calls are explicitly excluded from caching since
// stale exchange rates would silently produce wrong answers.

const CACHE_NAME = "calvoice-v3";
const PRECACHE = [
  "./",
  "./index.html",
  "./manifest.json",
  "./icon-192.png",
  "./icon-512.png"
];

// Install: precache the app shell immediately
self.addEventListener("install", event => {
  event.waitUntil(
    caches.open(CACHE_NAME).then(cache => cache.addAll(PRECACHE))
  );
  self.skipWaiting(); // activate right away, don't wait for old SW to die
});

// Activate: delete stale caches from previous installs
self.addEventListener("activate", event => {
  event.waitUntil(
    caches.keys().then(keys =>
      Promise.all(keys.filter(k => k !== CACHE_NAME).map(k => caches.delete(k)))
    )
  );
  self.clients.claim();
});

// Fetch: stale-while-revalidate for app assets; network-only for currency API
self.addEventListener("fetch", event => {
  const url = new URL(event.request.url);

  // Never cache live currency rate fetches — stale rates = wrong answers
  if (url.hostname.includes("open.er-api.com") || url.hostname.includes("frankfurter")){
    event.respondWith(fetch(event.request).catch(() => new Response("{}", { status: 503 })));
    return;
  }

  // Stale-while-revalidate: respond from cache instantly, refresh in background
  event.respondWith(
    caches.open(CACHE_NAME).then(async cache => {
      const cached = await cache.match(event.request);
      const networkFetch = fetch(event.request).then(response => {
        if (response.ok) cache.put(event.request, response.clone());
        return response;
      }).catch(() => null);

      // Return cached immediately if available; otherwise wait for network
      return cached || await networkFetch;
    })
  );
});
