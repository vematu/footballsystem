# Create the project again with simpler steps (no heavy libraries), then zip it.
import os, json, zipfile, shutil, base64
from pathlib import Path

root = Path("/mnt/data/clubpos-starter")
if root.exists():
    shutil.rmtree(root)
(root / "icons").mkdir(parents=True, exist_ok=True)
(root / "rules").mkdir(parents=True, exist_ok=True)

index_html = """<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <meta name="theme-color" content="#16a34a" />
  <title>Club Ticketing & POS</title>
  <link rel="manifest" href="./manifest.json" />
  <link rel="icon" href="./icons/icon-192.png" />
  <link rel="apple-touch-icon" href="./icons/icon-192.png" />
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    @media print {
      header, footer, nav, #authControls, #navDashboard, #navManageClub, #navUsers, #navTickets, #navPOS { display: none !important; }
      .print-card { box-shadow: none !important; border: none !important; }
    }
    .spinner { border: 4px solid #f3f3f3; border-top: 4px solid #16a34a; border-radius: 50%; width: 22px; height: 22px; animation: spin 1s linear infinite; }
    @keyframes spin { from { transform: rotate(0deg);} to { transform: rotate(360deg);} }
  </style>
</head>
<body class="bg-gray-50 text-gray-900">
  <div id="app" class="min-h-screen flex flex-col">
    <header class="sticky top-0 z-20 bg-white border-b">
      <div class="max-w-6xl mx-auto px-4 py-3 flex items-center justify-between gap-2">
        <div class="flex items-center gap-2">
          <img id="clubLogo" src="" alt="" class="w-8 h-8 rounded-lg hidden">
          <span id="appTitle" class="font-bold text-lg">Club Ticketing & POS</span>
          <span id="clubMotto" class="text-sm text-gray-500 ml-2"></span>
        </div>
        <nav class="flex items-center gap-2">
          <select id="clubSwitcher" class="px-2 py-1 border rounded-xl hidden"></select>
          <button id="navDashboard" class="px-3 py-1 rounded-xl hover:bg-gray-100">Dashboard</button>
          <button id="navManageClub" class="px-3 py-1 rounded-xl hover:bg-gray-100">Manage Club</button>
          <button id="navUsers" class="px-3 py-1 rounded-xl hover:bg-gray-100">Users</button>
          <button id="navTickets" class="px-3 py-1 rounded-xl hover:bg-gray-100">Tickets</button>
          <button id="navPOS" class="px-3 py-1 rounded-xl hover:bg-gray-100">POS</button>
          <div id="authControls" class="ml-2"></div>
        </nav>
      </div>
    </header>
    <main class="flex-1"><div id="view" class="max-w-6xl mx-auto p-4"></div></main>
    <footer class="border-t bg-white">
      <div class="max-w-6xl mx-auto px-4 py-4 text-sm text-gray-500 flex items-center justify-between">
        <span>© <span id="year"></span> ClubPOS</span>
        <a href="#" class="underline" onclick="window.location.hash='#/about'">About</a>
      </div>
    </footer>
  </div>
  <script type="module" src="./firebase.js"></script>
  <script src="https://unpkg.com/qrcode/build/qrcode.min.js"></script>
  <script type="module" src="./app.js"></script>
  <script>
    if ('serviceWorker' in navigator) {
      window.addEventListener('load', () => {
        navigator.serviceWorker.register('./sw.js').catch(console.error);
      });
    }
    document.getElementById('year').textContent = new Date().getFullYear();
  </script>
</body>
</html>
"""
(root / "index.html").write_text(index_html, encoding="utf-8")

firebase_config_example = """// 1) COPY this file and rename to: firebase-config.js
// 2) Fill in your Firebase project's web config below.
export const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "SENDER_ID",
  appId: "APP_ID"
};
"""
(root / "firebase-config.example.js").write_text(firebase_config_example, encoding="utf-8")

firebase_js = """// IMPORTANT: create firebase-config.js from firebase-config.example.js and fill your keys.
import { firebaseConfig } from './firebase-config.js';

import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.12.5/firebase-app.js';
import {
  getAuth, onAuthStateChanged, createUserWithEmailAndPassword,
  signInWithEmailAndPassword, signOut, updateProfile
} from 'https://www.gstatic.com/firebasejs/10.12.5/firebase-auth.js';
import {
  getFirestore, doc, getDoc, setDoc, updateDoc, addDoc, collection,
  query, where, getDocs, serverTimestamp
} from 'https://www.gstatic.com/firebasejs/10.12.5/firebase-firestore.js';

export const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const db = getFirestore(app);

export async function ensureUserProfile(uid, data = {}) {
  const ref = doc(db, 'users', uid);
  const snap = await getDoc(ref);
  if (!snap.exists()) {
    await setDoc(ref, { createdAt: serverTimestamp(), ...data });
  } else if (Object.keys(data).length) {
    await updateDoc(ref, data);
  }
}

export function membershipKey(uid, clubId) {
  return `${uid}_${clubId}`;
}

export async function upsertMembership(uid, clubId, role='owner') {
  const key = membershipKey(uid, clubId);
  const ref = doc(db, 'memberships', key);
  await setDoc(ref, { uid, clubId, role, createdAt: serverTimestamp() }, { merge: true });
  return ref;
}

export async function getUserMemberships(uid) {
  const q = query(collection(db, 'memberships'), where('uid', '==', uid));
  const snap = await getDocs(q);
  return snap.docs.map(d => d.data());
}

export {
  onAuthStateChanged,
  createUserWithEmailAndPassword,
  signInWithEmailAndPassword,
  signOut,
  updateProfile,
  doc, getDoc, setDoc, updateDoc, addDoc, collection, query, where, getDocs, serverTimestamp
};
"""
(root / "firebase.js").write_text(firebase_js, encoding="utf-8")

app_js = """import {
  auth, db,
  onAuthStateChanged, createUserWithEmailAndPassword, signInWithEmailAndPassword, signOut, updateProfile,
  doc, getDoc, setDoc, updateDoc, addDoc, collection, query, where, getDocs, serverTimestamp,
  ensureUserProfile, upsertMembership, getUserMemberships
} from './firebase.js';

const $ = (sel) => document.querySelector(sel);
const view = $('#view');
const topLogo = $('#clubLogo');
const appTitle = $('#appTitle');
const clubMotto = $('#clubMotto');
const authControls = $('#authControls');
const clubSwitcher = $('#clubSwitcher');

const routes = {
  '/': dashboard,
  '/login': loginView,
  '/signup': signupView,
  '/club/new': newClubView,
  '/club/manage': manageClubView,
  '/users': usersView,
  '/tickets': ticketsView,
  '/pos': posView,
  '/scan': scanView,
  '/about': aboutView
};

window.addEventListener('hashchange', render);
$('#navDashboard').onclick = () => location.hash = '#/';
$('#navManageClub').onclick = () => location.hash = '#/club/manage';
$('#navUsers').onclick = () => location.hash = '#/users';
$('#navTickets').onclick = () => location.hash = '#/tickets';
$('#navPOS').onclick = () => location.hash = '#/pos';

let currentClub = null;
let myMemberships = [];

onAuthStateChanged(auth, async (user) => {
  if (user) {
    await ensureUserProfile(user.uid, { email: user.email, displayName: user.displayName || '' });
    myMemberships = await getUserMemberships(user.uid);
    if (!currentClub && myMemberships.length) {
      currentClub = await loadClub(myMemberships[0].clubId);
    }
    renderClubSwitcher();
  } else {
    myMemberships = [];
    currentClub = null;
    renderClubSwitcher();
  }
  await render();
  applyTheme();
});

function renderClubSwitcher() {
  if (!auth.currentUser || myMemberships.length === 0) {
    clubSwitcher.classList.add('hidden');
    return;
  }
  clubSwitcher.innerHTML = myMemberships.map(m => `<option value=\\"${m.clubId}\\" ${currentClub?.id===m.clubId?'selected':''}>${m.clubId}</option>`).join('');
  clubSwitcher.onchange = async () => {
    currentClub = await loadClub(clubSwitcher.value);
    applyTheme();
    await render();
  };
  clubSwitcher.classList.remove('hidden');
}

async function render() {
  const path = location.hash.replace('#', '') || '/';
  const route = routes[path] || dashboard;
  await renderAuthControls();
  await route();
}

async function renderAuthControls() {
  const user = auth.currentUser;
  if (!user) {
    authControls.innerHTML = `
      <button class="px-3 py-1 rounded-xl bg-emerald-600 text-white" onclick="location.hash='#/login'">Log in</button>
      <button class="px-3 py-1 rounded-xl border" onclick="location.hash='#/signup'">Sign up</button>
    `;
  } else {
    authControls.innerHTML = `
      <span class="text-sm mr-2">${user.email}</span>
      <button id="logoutBtn" class="px-3 py-1 rounded-xl border">Log out</button>
    `;
    $('#logoutBtn').onclick = async () => {
      await signOut(auth);
      location.hash = '#/login';
    };
  }
}

function applyTheme() {
  const theme = currentClub?.theme || { primary: '#16a34a', secondary: '#0ea5e9' };
  document.querySelector('meta[name="theme-color"]').setAttribute('content', theme.primary);
  appTitle.textContent = currentClub?.name || 'Club Ticketing & POS';
  clubMotto.textContent = currentClub?.motto ? `— ${currentClub.motto}` : '';
  if (currentClub?.logoUrl) {
    topLogo.src = currentClub.logoUrl;
    topLogo.classList.remove('hidden');
  } else {
    topLogo.classList.add('hidden');
  }
}

async function dashboard() {
  const user = auth.currentUser;
  if (!user) return redirectLogin();
  if (!currentClub) {
    view.innerHTML = `
      <div class="max-w-xl mx-auto mt-8 bg-white border rounded-2xl p-6">
        <h1 class="text-xl font-bold mb-3">Welcome</h1>
        <p class="mb-4">You are not a member of any club yet. Create a new club to get started.</p>
        <button class="px-4 py-2 rounded-xl bg-emerald-600 text-white" onclick="location.hash='#/club/new'">Create a Club</button>
      </div>`;
    return;
  }
  view.innerHTML = `
    <div class="grid md:grid-cols-3 gap-4 mt-4">
      <div class="bg-white border rounded-2xl p-5">
        <h2 class="font-semibold mb-2">Quick Actions</h2>
        <div class="flex flex-wrap gap-2">
          <button class="px-3 py-2 rounded-xl border" onclick="location.hash='#/tickets'">Sell Ticket</button>
          <button class="px-3 py-2 rounded-xl border" onclick="location.hash='#/pos'">Open POS</button>
          <button class="px-3 py-2 rounded-xl border" onclick="location.hash='#/club/manage'">Manage Club</button>
          <button class="px-3 py-2 rounded-xl border" onclick="location.hash='#/users'">Manage Users</button>
        </div>
      </div>
      <div class="bg-white border rounded-2xl p-5 md:col-span-2">
        <h2 class="font-semibold mb-2">Club</h2>
        <div class="flex items-center gap-3">
          ${currentClub?.logoUrl ? `<img src="${currentClub.logoUrl}" class="w-12 h-12 rounded-xl border" />` : ''}
          <div>
            <div class="font-bold">${currentClub?.name}</div>
            <div class="text-sm text-gray-500">${currentClub?.motto || ''}</div>
          </div>
        </div>
      </div>
    </div>`;
}

function redirectLogin() {
  location.hash = '#/login';
  view.innerHTML = '';
}

async function loginView() {
  if (auth.currentUser) return (location.hash = '#/');
  view.innerHTML = `
    <div class="max-w-sm mx-auto mt-10 bg-white border rounded-2xl p-6">
      <h1 class="text-xl font-bold mb-4">Log in</h1>
      <form id="loginForm" class="space-y-3">
        <input id="email" type="email" placeholder="Email" class="w-full border rounded-xl px-3 py-2" required />
        <input id="password" type="password" placeholder="Password" class="w-full border rounded-xl px-3 py-2" required />
        <button class="w-full px-4 py-2 rounded-xl bg-emerald-600 text-white">Log in</button>
      </form>
      <p class="text-sm mt-3">No account? <a class="underline" href="#/signup">Sign up</a></p>
    </div>`;
  $('#loginForm').onsubmit = async (e) => {
    e.preventDefault();
    const email = $('#email').value;
    const password = $('#password').value;
    await signInWithEmailAndPassword(auth, email, password);
    location.hash = '#/';
  };
}

async function signupView() {
  if (auth.currentUser) return (location.hash = '#/');
  view.innerHTML = `
    <div class="max-w-sm mx-auto mt-10 bg-white border rounded-2xl p-6">
      <h1 class="text-xl font-bold mb-4">Create account</h1>
      <form id="signupForm" class="space-y-3">
        <input id="name" type="text" placeholder="Your name" class="w-full border rounded-xl px-3 py-2" required />
        <input id="email" type="email" placeholder="Email" class="w-full border rounded-xl px-3 py-2" required />
        <input id="password" type="password" placeholder="Password" class="w-full border rounded-xl px-3 py-2" required />
        <button class="w-full px-4 py-2 rounded-xl bg-emerald-600 text-white">Sign up</button>
      </form>
      <p class="text-sm mt-3">Already have an account? <a class="underline" href="#/login">Log in</a></p>
    </div>`;
  $('#signupForm').onsubmit = async (e) => {
    e.preventDefault();
    const name = $('#name').value;
    const email = $('#email').value;
    const password = $('#password').value;
    const cred = await createUserWithEmailAndPassword(auth, email, password);
    await updateProfile(cred.user, { displayName: name });
    await ensureUserProfile(cred.user.uid, { email, displayName: name });
    location.hash = '#/';
  };
}

async function newClubView() {
  const user = auth.currentUser;
  if (!user) return redirectLogin();
  view.innerHTML = `
    <div class="max-w-2xl mx-auto mt-8 bg-white border rounded-2xl p-6">
      <h1 class="text-xl font-bold mb-4">Create a Club</h1>
      <form id="clubForm" class="grid md:grid-cols-2 gap-3">
        <input id="name" class="border rounded-xl px-3 py-2" placeholder="Club name" required />
        <input id="slug" class="border rounded-xl px-3 py-2" placeholder="Slug (unique id, e.g. 'lions')" required />
        <input id="motto" class="border rounded-xl px-3 py-2 md:col-span-2" placeholder="Motto (optional)" />
        <input id="logoUrl" class="border rounded-xl px-3 py-2 md:col-span-2" placeholder="Logo URL (PNG/JPG)" />
        <input id="primary" type="color" class="border rounded-xl px-3 py-2" value="#16a34a" title="Primary color" />
        <input id="secondary" type="color" class="border rounded-xl px-3 py-2" value="#0ea5e9" title="Secondary color" />
        <input id="address" class="border rounded-xl px-3 py-2 md:col-span-2" placeholder="Address" />
        <input id="contact" class="border rounded-xl px-3 py-2 md:col-span-2" placeholder="Contact (phone/email)" />
        <button class="md:col-span-2 px-4 py-2 rounded-xl bg-emerald-600 text-white">Create</button>
      </form>
    </div>`;
  $('#clubForm').onsubmit = async (e) => {
    e.preventDefault();
    const payload = {
      name: $('#name').value.trim(),
      slug: $('#slug').value.trim().toLowerCase(),
      motto: $('#motto').value.trim(),
      logoUrl: $('#logoUrl').value.trim(),
      theme: { primary: $('#primary').value, secondary: $('#secondary').value },
      address: $('#address').value.trim(),
      contact: $('#contact').value.trim(),
      active: true,
      createdAt: serverTimestamp()
    };
    const ref = doc(db, 'clubs', payload.slug);
    await setDoc(ref, payload, { merge: true });
    await upsertMembership(auth.currentUser.uid, payload.slug, 'owner');
    currentClub = await loadClub(payload.slug);
    renderClubSwitcher();
    location.hash = '#/';
  };
}

async function manageClubView() {
  const user = auth.currentUser;
  if (!user) return redirectLogin();
  if (!currentClub) return (location.hash = '#/club/new');
  view.innerHTML = `
    <div class="max-w-2xl mx-auto mt-8 bg-white border rounded-2xl p-6">
      <h1 class="text-xl font-bold mb-4">Manage Club</h1>
      <form id="clubEdit" class="grid md:grid-cols-2 gap-3">
        <input id="name" class="border rounded-xl px-3 py-2" placeholder="Club name" value="${currentClub.name}" />
        <input id="motto" class="border rounded-xl px-3 py-2 md:col-span-2" placeholder="Motto" value="${currentClub.motto || ''}" />
        <input id="logoUrl" class="border rounded-xl px-3 py-2 md:col-span-2" placeholder="Logo URL" value="${currentClub.logoUrl || ''}" />
        <input id="primary" type="color" class="border rounded-xl px-3 py-2" value="${currentClub.theme?.primary || '#16a34a'}" />
        <input id="secondary" type="color" class="border rounded-xl px-3 py-2" value="${currentClub.theme?.secondary || '#0ea5e9'}" />
        <input id="address" class="border rounded-xl px-3 py-2 md:col-span-2" placeholder="Address" value="${currentClub.address || ''}" />
        <input id="contact" class="border rounded-xl px-3 py-2 md:col-span-2" placeholder="Contact" value="${currentClub.contact || ''}" />
        <div class="md:col-span-2 flex gap-2">
          <button class="px-4 py-2 rounded-xl bg-emerald-600 text-white" type="submit">Save</button>
          <button class="px-4 py-2 rounded-xl border" type="button" id="preview">Preview Theme</button>
        </div>
      </form>
    </div>`;
  $('#clubEdit').onsubmit = async (e) => {
    e.preventDefault();
    const ref = doc(db, 'clubs', currentClub.id);
    const data = {
      name: $('#name').value.trim(),
      motto: $('#motto').value.trim(),
      logoUrl: $('#logoUrl').value.trim(),
      theme: { primary: $('#primary').value, secondary: $('#secondary').value },
      address: $('#address').value.trim(),
      contact: $('#contact').value.trim()
    };
    await updateDoc(ref, data);
    currentClub = await loadClub(currentClub.id);
    applyTheme();
    alert('Saved.');
  };
  $('#preview').onclick = () => {
    currentClub.theme = { primary: $('#primary').value, secondary: $('#secondary').value };
    applyTheme();
  };
}

async function usersView() {
  const user = auth.currentUser;
  if (!user) return redirectLogin();
  if (!currentClub) return (location.hash = '#/club/new');
  const qMembers = query(collection(db, 'memberships'), where('clubId', '==', currentClub.id));
  const snap = await getDocs(qMembers);
  const members = snap.docs.map(d => d.data());
  view.innerHTML = `
    <div class="max-w-3xl mx-auto mt-8 bg-white border rounded-2xl p-6">
      <h1 class="text-xl font-bold mb-4">Users for ${currentClub.name}</h1>
      <div class="grid md:grid-cols-2 gap-6">
        <div>
          <h2 class="font-semibold mb-2">Invite / Link User</h2>
          <form id="inviteForm" class="space-y-2">
            <input id="inviteEmail" type="email" class="w-full border rounded-xl px-3 py-2" placeholder="User email" required />
            <select id="inviteRole" class="w-full border rounded-xl px-3 py-2">
              <option>admin</option>
              <option>cashier</option>
              <option>fan</option>
            </select>
            <button class="px-4 py-2 rounded-xl bg-emerald-600 text-white">Add</button>
          </form>
          <p class="text-sm text-gray-500 mt-2">The user must sign up at least once so their account exists.</p>
        </div>
        <div>
          <h2 class="font-semibold mb-2">Members</h2>
          <ul class="space-y-2" id="memberList"></ul>
        </div>
      </div>
    </div>`;
  const userDocs = await Promise.all(members.map(m => getDoc(doc(db, 'users', m.uid))));
  const memberList = $('#memberList');
  memberList.innerHTML = members.map((m, i) => {
    const email = userDocs[i].exists() ? userDocs[i].data().email : m.uid;
    return `
      <li class="border rounded-xl p-3 flex items-center justify-between">
        <div>
          <div class="font-medium">${email}</div>
          <div class="text-sm text-gray-500">Role: ${m.role}</div>
        </div>
        <div class="flex gap-2">
          <button class="px-3 py-1 rounded-xl border" data-uid="${m.uid}" data-role="admin">Make admin</button>
          <button class="px-3 py-1 rounded-xl border" data-uid="${m.uid}" data-role="cashier">Make cashier</button>
          <button class="px-3 py-1 rounded-xl border" data-uid="${m.uid}" data-role="fan">Make fan</button>
        </div>
      </li>`;
  }).join('');
  memberList.querySelectorAll('button').forEach(btn => {
    btn.addEventListener('click', async () => {
      const uid = btn.getAttribute('data-uid');
      const role = btn.getAttribute('data-role');
      const ref = doc(db, 'memberships', `${uid}_${currentClub.id}`);
      await updateDoc(ref, { role });
      alert('Role updated');
      location.reload();
    });
  });
  $('#inviteForm').onsubmit = async (e) => {
    e.preventDefault();
    const email = ($('#inviteEmail').value || '').trim().toLowerCase();
    const role = $('#inviteRole').value;
    const qUsers = query(collection(db, 'users'), where('email', '==', email));
    const snapUsers = await getDocs(qUsers);
    if (snapUsers.empty) {
      alert('This email has not signed up yet. Ask them to create an account first.');
      return;
    }
    const uid = snapUsers.docs[0].id;
    await setDoc(doc(db, 'memberships', `${uid}_${currentClub.id}`), {
      uid, clubId: currentClub.id, role, createdAt: serverTimestamp()
    }, { merge: true });
    alert('User linked to club.');
    location.reload();
  };
}

async function ticketsView() {
  const user = auth.currentUser;
  if (!user) return redirectLogin();
  if (!currentClub) return (location.hash = '#/club/new');
  view.innerHTML = `
    <div class="max-w-3xl mx-auto mt-8 bg-white border rounded-2xl p-6">
      <h1 class="text-xl font-bold mb-4">Sell Ticket (MVP)</h1>
      <form id="ticketForm" class="grid md:grid-cols-2 gap-3">
        <input id="match" class="border rounded-xl px-3 py-2" placeholder="Match (e.g., Lions vs Eagles)" required />
        <input id="price" type="number" min="0" class="border rounded-xl px-3 py-2" placeholder="Price" required />
        <input id="buyer" class="border rounded-xl px-3 py-2 md:col-span-2" placeholder="Buyer name (optional)" />
        <button class="md:col-span-2 px-4 py-2 rounded-xl bg-emerald-600 text-white">Generate Ticket</button>
      </form>
      <div id="ticketOut" class="mt-6"></div>
    </div>`;
  $('#ticketForm').onsubmit = async (e) => {
    e.preventDefault();
    const match = $('#match').value.trim();
    const price = Number($('#price').value);
    const buyer = $('#buyer').value.trim();
    const ticketRef = await addDoc(collection(db, 'tickets'), {
      clubId: currentClub.id, match, price, buyer, status: 'valid',
      createdAt: serverTimestamp()
    });
    const payload = `ticket:${ticketRef.id}:${currentClub.id}`;
    const out = $('#ticketOut');
    out.innerHTML = `
      <div class="border rounded-2xl p-4 print-card">
        <div class="flex items-center gap-3 mb-3">
          ${currentClub?.logoUrl ? `<img src="${currentClub.logoUrl}" class="w-10 h-10 rounded-lg border" />` : ''}
          <div>
            <div class="font-bold">${currentClub.name}</div>
            <div class="text-sm text-gray-500">${currentClub.motto || ''}</div>
          </div>
        </div>
        <div class="grid grid-cols-2 gap-3 text-sm mb-3">
          <div><span class="text-gray-500">Match:</span> ${match}</div>
          <div><span class="text-gray-500">Price:</span> ${price}</div>
          <div><span class="text-gray-500">Buyer:</span> ${buyer || '-'}</div>
          <div><span class="text-gray-500">Club:</span> ${currentClub.id}</div>
        </div>
        <canvas id="qrCanvas" class="border rounded-xl p-2 bg-white"></canvas>
        <div class="mt-3 print:hidden">
          <button class="px-4 py-2 rounded-xl border" onclick="window.print()">Print receipt</button>
        </div>
      </div>`;
    QRCode.toCanvas($('#qrCanvas'), payload, { width: 192 }, function (error) {
      if (error) console.error(error);
    });
  };
}

async function posView() {
  const user = auth.currentUser;
  if (!user) return redirectLogin();
  if (!currentClub) return (location.hash = '#/club/new');
  view.innerHTML = `
    <div class="max-w-4xl mx-auto mt-8 bg-white border rounded-2xl p-6">
      <h1 class="text-xl font-bold mb-4">POS (MVP)</h1>
      <p class="text-sm text-gray-500 mb-4">Coming next: products, cart, receipt printing.</p>
      <div class="grid md:grid-cols-3 gap-3">
        <div class="border rounded-xl p-3">Product 1</div>
        <div class="border rounded-xl p-3">Product 2</div>
        <div class="border rounded-xl p-3">Product 3</div>
      </div>
    </div>`;
}

async function scanView() {
  const user = auth.currentUser;
  if (!user) return redirectLogin();
  if (!currentClub) return (location.hash = '#/club/new');
  view.innerHTML = `
    <div class="max-w-xl mx-auto mt-8 bg-white border rounded-2xl p-6">
      <h1 class="text-xl font-bold mb-4">Scan Ticket (MVP)</h1>
      <p class="text-sm text-gray-500">Use a QR scanner app to read the code and open the payload here. Validation logic will mark it used later.</p>
    </div>`;
}

async function aboutView() {
  view.innerHTML = `
    <div class="max-w-2xl mx-auto mt-8 bg-white border rounded-2xl p-6">
      <h1 class="text-xl font-bold mb-2">About</h1>
      <p class="text-gray-600">Multi-club ticketing & POS. PWA-first. Firebase Auth + Firestore.</p>
    </div>`;
}

async function loadClub(clubId) {
  const ref = doc(db, 'clubs', clubId);
  const snap = await getDoc(ref);
  if (!snap.exists()) return null;
  return { id: clubId, ...snap.data() };
}
"""
(root / "app.js").write_text(app_js, encoding="utf-8")

manifest = {
  "name": "Club Ticketing & POS",
  "short_name": "ClubPOS",
  "start_url": "./index.html",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#16a34a",
  "icons": [
    {"src": "icons/icon-192.png", "sizes": "192x192", "type": "image/png"},
    {"src": "icons/icon-512.png", "sizes": "512x512", "type": "image/png"}
  ]
}
(root / "manifest.json").write_text(json.dumps(manifest, indent=2), encoding="utf-8")

sw_js = """const CACHE = 'clubpos-v2';
const ASSETS = [
  './', './index.html', './app.js', './firebase.js', './firebase-config.js', './manifest.json'
];
self.addEventListener('install', (e) => {
  e.waitUntil((async () => {
    const cache = await caches.open(CACHE);
    await cache.addAll(ASSETS);
    self.skipWaiting();
  })());
});
self.addEventListener('activate', (e) => {
  e.waitUntil((async () => {
    const keys = await caches.keys();
    await Promise.all(keys.map(k => k !== CACHE && caches.delete(k)));
    await self.clients.claim();
  })());
});
self.addEventListener('fetch', (e) => {
  e.respondWith((async () => {
    const cached = await caches.match(e.request);
    if (cached) return cached;
    try { return await fetch(e.request); } catch { return cached || Response.error(); }
  })());
});
"""
(root / "sw.js").write_text(sw_js, encoding="utf-8")

readme = """# ClubPOS Starter (Multi-Club, PWA, Firebase)
Beginner-friendly starter app to manage multiple football clubs with logo, colors, motto, users, tickets (QR) and POS area.

## What you need
- Google account for Firebase.

## Step 1 — Get Firebase web keys
1) Firebase console → Create project.  
2) Add Web App (</>) → Copy the config (apiKey, authDomain, etc.).  
3) Build → Authentication → enable Email/Password.  
4) Build → Firestore Database → Create database (Production).  

## Step 2 — Put your keys in the project
Copy `firebase-config.example.js` → rename to `firebase-config.js` → paste your keys.

## Step 3 — Run it
- Easiest: deploy free on Netlify or Vercel (drag and drop the folder).  
- Or locally: open a static server (VS Code Live Server, or `python -m http.server` in this folder) and open http://localhost:8000

## Step 4 — Use it
- Sign up (owner).  
- Create a Club (logo URL, colors, motto).  
- Users → link other emails to your club (they must sign up first).  
- Tickets → generate QR & print receipt.  

## Security Rules
See `rules/firestore.rules` and paste into Firebase → Firestore → Rules → Publish.
"""
(root / "README.md").write_text(readme, encoding="utf-8")

firestore_rules = """rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == userId;
    }
    match /clubs/{clubId} {
      allow read: if isMember(clubId);
      allow write: if hasRole(clubId, ['owner', 'admin']);
    }
    match /memberships/{key} {
      allow read: if request.auth != null && isMember(targetClubId());
      allow write: if request.auth != null && hasRole(targetClubId(), ['owner', 'admin']);
    }
    match /tickets/{ticketId} {
      allow read, write: if request.auth != null && isMember(resource.data.clubId);
    }
    function
