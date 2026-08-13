/* מערכת אסף — Service Worker
   מאפשר טעינה של האפליקציה גם ללא אינטרנט.
   הנתונים (Firebase) תמיד עוברים דרך הרשת ואינם נשמרים במטמון. */
var CACHE = 'asaf-v1';
var SHELL = [ './', './index.html' ];

self.addEventListener('install', function(e){
  self.skipWaiting();
  e.waitUntil(caches.open(CACHE).then(function(c){ return c.addAll(SHELL); }));
});

self.addEventListener('activate', function(e){
  e.waitUntil(
    caches.keys().then(function(keys){
      return Promise.all(keys.map(function(k){ if(k !== CACHE) return caches.delete(k); }));
    }).then(function(){ return self.clients.claim(); })
  );
});

self.addEventListener('fetch', function(e){
  var req = e.request;
  if (req.method !== 'GET') return; // כתיבות (PUT/POST) — לרשת בלבד

  var url = req.url;
  // בקשות ל-Firebase — תמיד רשת (מידע חי, לא ממטמון)
  if (url.indexOf('firebasedatabase.app') >= 0) return;

  // ניווט לדף — קודם רשת, ואם אין חיבור — מהמטמון
  if (req.mode === 'navigate') {
    e.respondWith(
      fetch(req).then(function(res){
        var copy = res.clone();
        caches.open(CACHE).then(function(c){ c.put('./index.html', copy); });
        return res;
      }).catch(function(){ return caches.match('./index.html'); })
    );
    return;
  }

  // שאר המשאבים (גופנים, וכו') — קודם מטמון, אחרת רשת ושמירה
  e.respondWith(
    caches.match(req).then(function(hit){
      return hit || fetch(req).then(function(res){
        if (res && res.status === 200 && (res.type === 'basic' || res.type === 'cors')) {
          var copy = res.clone();
          caches.open(CACHE).then(function(c){ c.put(req, copy); });
        }
        return res;
      }).catch(function(){ return hit; });
    })
  );
});
