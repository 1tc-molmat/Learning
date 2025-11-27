# Learning Platform API - Dokumentáció

## 📋 Tartalomjegyzék

1. [Projekt áttekintése](#projekt-áttekintése)
2. [Technológiai stack](#technológiai-stack)
3. [Telepítés és konfiguráció](#telepítés-és-konfiguráció)
4. [Adatbázis struktúra](#adatbázis-struktúra)
5. [API végpontok](#api-végpontok)
6. [Authentikáció](#authentikáció)
7. [Szerepkörök és jogosultságok](#szerepkörök-és-jogosultságok)
8. [Hibakezelés és hibaüzenetek](#hibakezelés-és-hibaüzenetek)
9. [Tesztelés](#tesztelés)

---

## 🎯 Projekt áttekintése

A **Learning Platform API** egy modern, REST API alapú oktatási platform backend rendszer, amely Laravel 11 keretrendszerre épül. A rendszer lehetővé teszi felhasználók számára a regisztrációt, bejelentkezést, kurzusok böngészését, beiratkozást és kurzusok teljesítését.

### Főbb funkciók

- ✅ Felhasználói regisztráció és bejelentkezés
- ✅ Bearer token alapú authentikáció (Laravel Sanctum)
- ✅ Kurzuskezelés (listázás, megtekintés)
- ✅ Kurzusokra való beiratkozás
- ✅ Kurzusok teljesítésének nyomonkövetése
- ✅ Szerepkör alapú hozzáférés-szabályozás (admin/student)
- ✅ Felhasználói profilkezelés

---

## 🛠 Technológiai stack

| Technológia | Verzió | Leírás |
|-------------|--------|--------|
| **PHP** | 8.2+ | Backend nyelv |
| **Laravel** | 11.x | PHP keretrendszer |
| **MySQL** | 8.0+ | Relációs adatbázis |
| **Laravel Sanctum** | - | API authentikáció |
| **Composer** | 2.x | PHP csomagkezelő |
| **XAMPP** | 8.2+ | Fejlesztői környezet |

---

## 📦 Telepítés és konfiguráció

### Előfeltételek

- PHP >= 8.2
- Composer
- MySQL 8.0+
- XAMPP vagy hasonló fejlesztői környezet

### Telepítési lépések

#### 1. Repository klónozása

```bash
git clone https://github.com/1tc-molmat/Learning.git
cd learningPlatformBearer
```

#### 2. Függőségek telepítése

```bash
composer install
```

#### 3. Környezeti változók beállítása

Másold le a `.env.example` fájlt `.env` néven:

```bash
copy .env.example .env
```

Szerkeszd a `.env` fájlt:

```env
APP_NAME="Learning Platform"
APP_ENV=local
APP_KEY=base64:y3L9MKYFogMLZZSxRiI5JgZS4y4HHl2T8VLJIInbN98=
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=learning_platform
DB_USERNAME=root
DB_PASSWORD=

SESSION_DRIVER=database
QUEUE_CONNECTION=database
CACHE_STORE=database
```

#### 4. Alkalmazás kulcs generálása

```bash
php artisan key:generate
```

#### 5. Adatbázis létrehozása

MySQL-ben hozd létre az adatbázist:

```sql
CREATE DATABASE learning_platform CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

Vagy parancssorból:

```bash
mysql -u root -e "CREATE DATABASE learning_platform CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

#### 6. Migrációk futtatása

```bash
php artisan migrate
```

#### 7. Adatbázis feltöltése kezdeti adatokkal

```bash
php artisan db:seed
```

Ez létrehozza:
- 1 admin felhasználót (email: `admin@example.com`, jelszó: `admin`)
- 9 diák felhasználót (random adatokkal)
- 3 kurzust

#### 8. Szerver indítása

```bash
php artisan serve
```

Az API elérhető: `http://localhost:8000/api`

---

## 🗄 Adatbázis struktúra

### Users tábla

| Mező | Típus | Leírás |
|------|-------|--------|
| `id` | bigint unsigned | Egyedi azonosító |
| `name` | varchar(255) | Felhasználó neve |
| `email` | varchar(255) | Email cím (egyedi) |
| `password` | varchar(255) | Hash-elt jelszó |
| `role` | enum('admin', 'student') | Felhasználói szerepkör |
| `created_at` | timestamp | Létrehozás időpontja |
| `updated_at` | timestamp | Módosítás időpontja |

### Courses tábla

| Mező | Típus | Leírás |
|------|-------|--------|
| `id` | bigint unsigned | Egyedi azonosító |
| `title` | varchar(255) | Kurzus címe |
| `description` | text | Kurzus leírása |
| `created_at` | timestamp | Létrehozás időpontja |
| `updated_at` | timestamp | Módosítás időpontja |

### Enrollments tábla

| Mező | Típus | Leírás |
|------|-------|--------|
| `id` | bigint unsigned | Egyedi azonosító |
| `user_id` | bigint unsigned | Felhasználó ID (foreign key) |
| `course_id` | bigint unsigned | Kurzus ID (foreign key) |
| `completed` | boolean | Teljesítve van-e (default: false) |
| `created_at` | timestamp | Beiratkozás időpontja |
| `updated_at` | timestamp | Módosítás időpontja |

**Egyedi kulcs:** `user_id` + `course_id` (egy felhasználó csak egyszer iratkozhat be egy kurzusra)

---

## 🔌 API végpontok

### Base URL

```
http://localhost:8000/api
```

### Publikus végpontok (authentikáció nélkül)

#### 🔹 Teszt végpont

```http
GET /api/ping
```

**Válasz:**
```json
{
  "message": "API works!"
}
```

---

#### 🔹 Regisztráció

```http
POST /api/register
```

**Request Body:**
```json
{
  "name": "Kiss János",
  "email": "kiss.janos@example.com",
  "password": "jelszo123",
  "password_confirmation": "jelszo123"
}
```

**Validációs szabályok:**
- `name`: kötelező, string, max 255 karakter
- `email`: kötelező, email formátum, egyedi az adatbázisban
- `password`: kötelező, string, minimum 8 karakter
- `password_confirmation`: kötelező, meg kell egyeznie a jelszóval

**Sikeres válasz (201 Created):**
```json
{
  "message": "User created successfully",
  "user": {
    "id": 11,
    "name": "Kiss János",
    "email": "kiss.janos@example.com",
    "role": "student"
  }
}
```

**Hiba válasz (422 Unprocessable Entity):**
```json
{
  "message": "Failed to register user",
  "errors": {
    "email": [
      "The email has already been taken."
    ],
    "password": [
      "The password field confirmation does not match."
    ]
  }
}
```

---

#### 🔹 Bejelentkezés

```http
POST /api/login
```

**Request Body:**
```json
{
  "email": "admin@example.com",
  "password": "admin"
}
```

**Sikeres válasz (200 OK):**
```json
{
  "message": "Login successful",
  "user": {
    "id": 1,
    "name": "admin",
    "email": "admin@example.com",
    "role": "admin"
  },
  "access": {
    "token": "1|abc123xyz...",
    "token_type": "Bearer"
  }
}
```

**Hiba válasz (401 Unauthorized):**
```json
{
  "message": "Invalid email or password"
}
```

---

### Védett végpontok (authentikáció szükséges)

**Minden védett végponthoz kötelező a Bearer token használata:**

```http
Authorization: Bearer 1|abc123xyz...
```

---

#### 🔹 Kijelentkezés

```http
POST /api/logout
```

**Headers:**
```
Authorization: Bearer {token}
```

**Sikeres válasz (200 OK):**
```json
{
  "message": "Logout successful"
}
```

**Megjegyzés:** Törli a felhasználó összes token-jét.

---

#### 🔹 Saját profil lekérdezése

```http
GET /api/users/me
```

**Headers:**
```
Authorization: Bearer {token}
```

**Sikeres válasz (200 OK):**
```json
{
  "id": 1,
  "name": "admin",
  "email": "admin@example.com",
  "role": "admin",
  "created_at": "2025-11-27T09:15:23.000000Z",
  "updated_at": "2025-11-27T09:15:23.000000Z"
}
```

---

#### 🔹 Saját profil frissítése

```http
PUT /api/users/me
```

**Headers:**
```
Authorization: Bearer {token}
```

**Request Body:**
```json
{
  "name": "Új Név",
  "email": "uj.email@example.com"
}
```

**Sikeres válasz (200 OK):**
```json
{
  "message": "User profile updated successfully",
  "user": {
    "id": 1,
    "name": "Új Név",
    "email": "uj.email@example.com",
    "role": "admin"
  }
}
```

---

#### 🔹 Kurzusok listázása

```http
GET /api/courses
```

**Headers:**
```
Authorization: Bearer {token}
```

**Sikeres válasz (200 OK):**
```json
[
  {
    "id": 1,
    "title": "Szoftverfejlesztési alapok",
    "description": "Alapvető programozási fogalmak és minták.",
    "created_at": "2025-11-27T10:27:12.000000Z",
    "updated_at": "2025-11-27T10:27:12.000000Z"
  },
  {
    "id": 2,
    "title": "REST API fejlesztés",
    "description": "API-k tervezése és készítése Laravelben.",
    "created_at": "2025-11-27T10:27:12.000000Z",
    "updated_at": "2025-11-27T10:27:12.000000Z"
  },
  {
    "id": 3,
    "title": "Fullstack webfejlesztés",
    "description": "Backend és frontend alapok.",
    "created_at": "2025-11-27T10:27:12.000000Z",
    "updated_at": "2025-11-27T10:27:12.000000Z"
  }
]
```

---

#### 🔹 Egy kurzus megtekintése

```http
GET /api/courses/{id}
```

**Headers:**
```
Authorization: Bearer {token}
```

**Példa:**
```http
GET /api/courses/1
```

**Sikeres válasz (200 OK):**
```json
{
  "id": 1,
  "title": "Szoftverfejlesztési alapok",
  "description": "Alapvető programozási fogalmak és minták.",
  "created_at": "2025-11-27T10:27:12.000000Z",
  "updated_at": "2025-11-27T10:27:12.000000Z"
}
```

**Hiba válasz (404 Not Found):**
```json
{
  "message": "Course not found"
}
```

---

#### 🔹 Beiratkozás kurzusra

```http
POST /api/courses/{id}/enroll
```

**Headers:**
```
Authorization: Bearer {token}
```

**Példa:**
```http
POST /api/courses/1/enroll
```

**Sikeres válasz (201 Created):**
```json
{
  "message": "Enrolled successfully",
  "enrollment": {
    "user_id": 2,
    "course_id": 1,
    "completed": false,
    "created_at": "2025-11-27T12:00:00.000000Z",
    "updated_at": "2025-11-27T12:00:00.000000Z"
  }
}
```

**Hiba válasz (400 Bad Request) - Már beiratkozott:**
```json
{
  "message": "Already enrolled in this course"
}
```

---

#### 🔹 Kurzus teljesítése

```http
PATCH /api/courses/{id}/completed
```

**Headers:**
```
Authorization: Bearer {token}
```

**Példa:**
```http
PATCH /api/courses/1/completed
```

**Sikeres válasz (200 OK):**
```json
{
  "message": "Course marked as completed",
  "enrollment": {
    "id": 1,
    "user_id": 2,
    "course_id": 1,
    "completed": true,
    "created_at": "2025-11-27T12:00:00.000000Z",
    "updated_at": "2025-11-27T12:30:00.000000Z"
  }
}
```

**Hiba válasz (404 Not Found):**
```json
{
  "message": "Enrollment not found"
}
```

---

### Admin végpontok (csak admin szerepkörrel)

#### 🔹 Összes felhasználó listázása

```http
GET /api/users
```

**Headers:**
```
Authorization: Bearer {token}
```

**Jogosultság:** Csak `admin` szerepkör

**Sikeres válasz (200 OK):**
```json
[
  {
    "id": 1,
    "name": "admin",
    "email": "admin@example.com",
    "role": "admin",
    "created_at": "2025-11-27T09:15:23.000000Z"
  },
  {
    "id": 2,
    "name": "Kiss János",
    "email": "kiss.janos@example.com",
    "role": "student",
    "created_at": "2025-11-27T10:30:45.000000Z"
  }
]
```

**Hiba válasz (403 Forbidden):**
```json
{
  "message": "Access denied. Admin role required."
}
```

---

#### 🔹 Egy felhasználó megtekintése

```http
GET /api/users/{id}
```

**Headers:**
```
Authorization: Bearer {token}
```

**Jogosultság:** Csak `admin` szerepkör

**Példa:**
```http
GET /api/users/2
```

**Sikeres válasz (200 OK):**
```json
{
  "id": 2,
  "name": "Kiss János",
  "email": "kiss.janos@example.com",
  "role": "student",
  "created_at": "2025-11-27T10:30:45.000000Z",
  "updated_at": "2025-11-27T10:30:45.000000Z"
}
```

---

#### 🔹 Felhasználó törlése

```http
DELETE /api/users/{id}
```

**Headers:**
```
Authorization: Bearer {token}
```

**Jogosultság:** Csak `admin` szerepkör

**Példa:**
```http
DELETE /api/users/2
```

**Sikeres válasz (200 OK):**
```json
{
  "message": "User deleted successfully"
}
```

**Hiba válasz (404 Not Found):**
```json
{
  "message": "User not found"
}
```

---

## 🔐 Authentikáció

A rendszer **Laravel Sanctum** alapú Bearer token authentikációt használ.

### Token megszerzése

1. **Regisztráció:** A `/api/register` végpont csak létrehozza a felhasználót, de nem ad tokent
2. **Bejelentkezés:** A `/api/login` végpont visszaad egy Bearer tokent

### Token használata

Minden védett végponthoz csatold a tokent az `Authorization` headerben:

```http
Authorization: Bearer 1|abc123xyz...
```

### Token érvényesség

- A tokenek alapértelmezetten **örökéletűek** (nem járnak le)
- A `/api/logout` végpont törli az összes tokent
- Manuálisan is törölhető a token az adatbázisból (`personal_access_tokens` tábla)

---

## 👥 Szerepkörök és jogosultságok

### Szerepkörök

A rendszerben két szerepkör létezik:

1. **admin** - Adminisztrátor
2. **student** - Diák

### Jogosultsági mátrix

| Funkció | Admin | Student |
|---------|-------|---------|
| Regisztráció | ✅ | ✅ |
| Bejelentkezés | ✅ | ✅ |
| Kijelentkezés | ✅ | ✅ |
| Saját profil megtekintése | ✅ | ✅ |
| Saját profil szerkesztése | ✅ | ✅ |
| Kurzusok listázása | ✅ | ✅ |
| Kurzus megtekintése | ✅ | ✅ |
| Beiratkozás kurzusra | ✅ | ✅ |
| Kurzus teljesítése | ✅ | ✅ |
| **Összes felhasználó listázása** | ✅ | ❌ |
| **Felhasználó megtekintése** | ✅ | ❌ |
| **Felhasználó törlése** | ✅ | ❌ |

---

## ⚠️ Hibakezelés és hibaüzenetek

### HTTP státusz kódok

| Kód | Jelentés | Példa |
|-----|----------|-------|
| **200 OK** | Sikeres kérés | Adatok lekérdezése, frissítése |
| **201 Created** | Sikeres létrehozás | Új felhasználó, beiratkozás |
| **400 Bad Request** | Hibás kérés | Már beiratkozott kurzusra |
| **401 Unauthorized** | Nincs authentikáció | Helytelen jelszó |
| **403 Forbidden** | Nincs jogosultság | Student próbál admin funkciót elérni |
| **404 Not Found** | Nem található | Nem létező kurzus vagy felhasználó |
| **422 Unprocessable Entity** | Validációs hiba | Hiányzó vagy helytelen mezők |

### Hibák formátuma

**Egyszerű hibaüzenet:**
```json
{
  "message": "Invalid email or password"
}
```

**Validációs hibák:**
```json
{
  "message": "Failed to register user",
  "errors": {
    "email": [
      "The email has already been taken."
    ],
    "password": [
      "The password field must be at least 8 characters."
    ]
  }
}
```

---

## 🧪 Tesztelés

### Postman Collection használata

1. Importáld a Postman Collection-t (ha elérhető)
2. Állítsd be a környezeti változót: `base_url = http://localhost:8000/api`
3. Futtasd a collection-t

### Manuális tesztelés cURL-lel

#### Regisztráció

```bash
curl -X POST http://localhost:8000/api/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test User",
    "email": "test@example.com",
    "password": "password123",
    "password_confirmation": "password123"
  }'
```

#### Bejelentkezés

```bash
curl -X POST http://localhost:8000/api/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@example.com",
    "password": "admin"
  }'
```

#### Védett végpont (token használatával)

```bash
curl -X GET http://localhost:8000/api/courses \
  -H "Authorization: Bearer 1|abc123xyz..." \
  -H "Content-Type: application/json"
```

---

## 📝 Kezdeti adatok (Seeding)

Az adatbázis seeding után a következő adatok érhetők el:

### Admin felhasználó

```
Email: admin@example.com
Jelszó: admin
Szerepkör: admin
```

### Kurzusok

1. **Szoftverfejlesztési alapok** - Alapvető programozási fogalmak és minták.
2. **REST API fejlesztés** - API-k tervezése és készítése Laravelben.
3. **Fullstack webfejlesztés** - Backend és frontend alapok.

### Diák felhasználók

A rendszer 9 random diák felhasználót hoz létre a `UserFactory` segítségével.

---

## 🔧 Hibaelhárítás

### SSL tanúsítvány hiba Composer-nél

**Hiba:**
```
curl error 60: SSL certificate problem: unable to get local issuer certificate
```

**Megoldás:**

1. Töltsd le a CA bundle-t:
```bash
curl -o C:\xampp\php\extras\ssl\cacert.pem https://curl.se/ca/cacert.pem
```

2. Szerkeszd a `C:\xampp\php\php.ini` fájlt:
```ini
curl.cainfo="C:\xampp\php\extras\ssl\cacert.pem"
openssl.cafile="C:\xampp\php\extras\ssl\cacert.pem"
```

3. Indítsd újra a szervert

### "Unknown database" hiba

**Hiba:**
```
SQLSTATE[HY000] [1049] Unknown database 'learning_platform'
```

**Megoldás:**

Hozd létre az adatbázist:
```bash
mysql -u root -e "CREATE DATABASE learning_platform;"
```

### "Invalid database connection [seed]" hiba

**Ok:** Nincs `seed` nevű adatbázis kapcsolat definiálva.

**Megoldás:**

A seeder futtatásához használd a default kapcsolatot:
```bash
php artisan db:seed
```

Ne használj `--database=seed` paramétert.

---

## 📞 Kapcsolat és támogatás

**Repository:** [https://github.com/1tc-molmat/Learning](https://github.com/1tc-molmat/Learning)

**Branch:** main

---

## 📄 Licenc

Ez a projekt oktatási célokra készült.

---

**Utolsó frissítés:** 2025. november 27.
