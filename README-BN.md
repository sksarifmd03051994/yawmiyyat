# يَوْمِيَّات — Android Installable PWA

## বর্তমান প্রোটোটাইপে যা আছে
- বাংলা UI, Hind Siliguri font, Islamic green theme, dark/light mode
- প্রথমবার ৪–৮ সংখ্যার PIN তৈরি, পরবর্তীবার PIN unlock
- Daily checklist, weekly checklist, monthly checklist
- জামাতে/একাকী নামাজ সংখ্যা, আয়-ব্যয়, ডায়েরি ও গুরুত্বপূর্ণ ঘটনা (৫০০ শব্দের নির্দেশিত সীমা)
- IndexedDB-তে এই ডিভাইসে রেকর্ড রাখা
- তারিখভিত্তিক রেকর্ড, মাসের সারাংশ, JSON backup/restore
- A4 portrait PDF print layout: দৈনিক ২ পৃষ্ঠা + সাপ্তাহিক/মাসিক ১ পৃষ্ঠা
- PWA manifest ও offline service worker

## Google Cloud Backup চালু করার জন্য আবশ্যক
এই ZIP-এ Firebase project বা Google OAuth credentials নেই। Cloud backup এখনো সক্রিয় নয়; নিরাপদভাবে চালু করতে নিজের Firebase project প্রয়োজন।

1. https://console.firebase.google.com/ এ একটি Firebase project তৈরি করুন।
2. Authentication > Sign-in method থেকে Google enable করুন।
3. Firestore Database তৈরি করুন।
4. Project settings > Your apps > Web app থেকে Firebase config কপি করুন।
5. `index.html`-এ `</head>`-এর আগে নিচের config যোগ করুন (নিজের প্রকৃত মান বসাবেন):
```html
<script>
window.YAWMIYYAT_FIREBASE_CONFIG = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  appId: "YOUR_APP_ID"
};
</script>
```
6. Firestore rules-এ user নিজের `users/{uid}/records/{date}` path-ই read/write করতে পারবে—এমন rule দিন:
```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{uid}/records/{recordId} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```
7. HTTPS hosting-এ deploy করুন (Firebase Hosting / অন্য static hosting)। তারপর Android Chrome-এ URL খুলে menu > Add to Home screen / Install app চাপুন।

## নিরাপত্তা ও সীমাবদ্ধতা
- PIN lock এই প্রোটোটাইপে একই browser/device-এ app screen lock; এটি encrypted vault বা server-side account password নয়।
- IndexedDB browser data clear করলে local copy হারাতে পারে। তাই Google Sign-In ও cloud sync config/deploy না করা পর্যন্ত uninstall/phone-loss recovery নিশ্চিত নয়।
- Google password কখনো এই app-এ টাইপ করবেন না; Google-এর নিজস্ব Sign-In popup ব্যবহার হবে।
- Cloud backup-এর জন্য Firebase project-এর usage free tier-এর বাইরে গেলে খরচ হতে পারে।
- Cloud restore/sync ও offline conflict handling production ব্যবহারের আগে আরও পরীক্ষা এবং security hardening প্রয়োজন।
