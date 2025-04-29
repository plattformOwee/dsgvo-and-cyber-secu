# Authentication & Profile – End‑to‑End Documentation

## 1 Authentication Flow (Signup → Verify → Add Fingerprint | Login → Verify | Fingerprint Login)

### 1.1 Sequence Diagram (text)

```text
┌───────────┐        ┌────────────────────┐        ┌──────────────┐
│  Client   │        │  PHP Backend API   │        │  MongoDB     │
└────┬──────┘        └────────┬───────────┘        └────┬─────────┘
     │ signup(username,email,pw)  ▲                          │
     │ POST /login_signup/signup.php                         │
     ▼                    │ writes unverified user           │
(201) not verified        │───────────▶ users               │
     │                    │ insert verification_code         │
     │                    │───────────▶ verification_codes   │
     │                    │ send email                       │
     │                    │◀──────────                       │
     │                    │ 200 {message:"Verification code sent"}
     │────────────────────────────────────────────────────────▶
     │ enter 6‑digit code │                                 │
     │ POST /login_signup/verify_code.php (email,code)       │
     ▼                    │ validate & set isVerified=true   │
(200) token               │───────────▶ users               │
     │ store JWT          │                                 │
     │                    │ 200 {message:"Success", token}  │
     │────────────────────────────────────────────────────────▶
     │ authenticate biometrics via local_auth               │
     │ POST /login_signup/addFingerprint.php (fingerprint)   │
     ▼                    │ push hash into devices[] array   │
(200) ok                  │───────────▶ users.devices       │
     │                    │                                 │

[alt path] returning user
     │ POST /login_signup/login.php                          │
     ▼                    │ if ok send code, else 401        │
 etc.
```

### 1.2 Endpoint Reference (REST – `/login_signup`)

|#|Method & Path|Auth|Brief Purpose|
|---|---|---|---|
|1|**POST** `/signup.php`|–|Register a new account|
|2|**POST** `/login.php`|–|Request verification‑code login|
|3|**POST** `/verify_code.php`|–|Verify signup code|
|4|**POST** `/verifycode_login.php`|–|Verify login code|
|5|**POST** `/addFingerprint.php`|Bearer|Attach device fingerprint|
|6|**POST** `/fingerprintLogin.php`|–|Log in with stored fingerprint|

---

## 2 Backend Script Reference — Authentication & Profile Creation

> **Format legend**  
> Each PHP endpoint is documented with:
> 
> - **curl** – a ready‑to‑run example request showing the expected inputs
>     
> - **echo** – snippet(s) that the script may output (success & common failure variants)
>     
> - **explanation** – step‑by‑step description of what the script does with the inputs and how it produces its response
>     

_Replace `https://api.yourhost.tld` with your actual backend base URL when testing._

---

### /login_signup

#### signup.php

##### curl

```bash
curl -X POST \
  https://api.yourhost.tld/login_signup/signup.php \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=alice42" \
  -d "email=alice@example.com" \
  -d "password=hunter2!!"
```

##### echo

**success**

```json
{"message":"Success"}
```

_if e‑mail exists but not verified_

```json
{"message":"not verified"}
```

**fail**

```json
{"error":"Password too weak"}
```

##### explanation

```
1. Validate username, e‑mail format and password strength.
2. If the e‑mail is brand‑new → insert user {isVerified:false} and generate 6‑digit code.
3. If the e‑mail exists but isVerified=false → reuse existing record, refresh code.
4. Store code in verification_codes, send it via SMTP.
5. Reply 201/200 with {message} but **never** the JWT yet.
```

---

#### login.php

##### curl

```bash
curl -X POST \
  https://api.yourhost.tld/login_signup/login.php \
  -d "email=alice@example.com" \
  -d "password=hunter2!!"
```

##### echo

**success**

```json
{"message":"Verification code sent"}
```

**fail**

```json
{"error":"Wrong credentials"}
```

##### explanation

```
1. Look up e‑mail, verify bcrypt‑hash against password.
2. If ok → generate new 6‑digit code, save to verification_codes, mail it.
3. Always reply 200 w/ generic success to avoid timing attacks.
```

---

#### verify_code.php

##### curl

```bash
curl -X POST \
  https://api.yourhost.tld/login_signup/verify_code.php \
  -d "email=alice@example.com" \
  -d "code=482911"
```

##### echo

**success**

```json
{"message":"Success","token":"<JWT>"}
```

**fail – wrong/expired**

```json
{"error":"Code expired or invalid"}
```

##### explanation

```
1. Check verification_codes for matching e‑mail+code and expiry (5 min).
2. On match → mark users.isVerified=true, delete code doc.
3. Sign a JWT (HS256) containing userId & exp (+30 days).
4. Return token so the client can continue to Add Fingerprint.
```

---

#### verifycode_login.php

##### curl

```bash
curl -X POST \
  https://api.yourhost.tld/login_signup/verifycode_login.php \
  -d "email=alice@example.com" \
  -d "code=482911"
```

##### echo

_success (identical to signup verification)_

```json
{"message":"Success","token":"<JWT>"}
```

_fail_

```json
{"error":"Code expired or invalid"}
```

##### explanation

```
Same logic as verify_code.php but without toggling isVerified; just authenticates the session.
```

---

#### addFingerprint.php

##### curl

```bash
curl -X POST \
  https://api.yourhost.tld/login_signup/addFingerprint.php \
  -H "Authorization: Bearer <JWT>" \
  -d "fingerprint=d93e5d477…"
```

##### echo

**success**

```json
{"message":"Fingerprint added"}
```

**fail – JWT invalid**

```json
{"error":"Invalid token"}
```

##### explanation

```
1. Decode & validate JWT → userId.
2. Upsert {fingerprint, addedAt} into users.devices array with $addToSet.
3. Return simple confirmation.
```

---

#### fingerprintLogin.php

##### curl

```bash
curl -X POST \
  https://api.yourhost.tld/login_signup/fingerprintLogin.php \
  -d "fingerprint=d93e5d477…"
```

##### echo

_success_

```json
{"token":"<JWT>"}
```

_fail – not registered_

```json
{"error":"Device not registered"}
```

##### explanation

```
1. Find user where devices.fingerprint == posted value.
2. If found → issue new JWT (device‑based login bypasses password).
3. Else reply 404 so UI falls back to e‑mail+password.
```

---

### /create_profile

#### add_answer.php

##### curl

```bash
curl -X POST \
  https://api.yourhost.tld/create_profile/add_answer.php \
  -H "Authorization: Bearer <JWT>" \
  -H "Content-Type: application/json" \
  -d '{"about_you":{"name":"Alice","age":"29","gender":"Female"}}'
```

##### echo

**success**

```json
{"success":true,"message":"Profile updated"}
```

**fail – validation**

```json
{"success":false,"error":"Age must be numeric"}
```

##### explanation

```
1. JWT → userId.
2. Validate payload fields (type & length).
3. $set users.profile.about_you = payload.about_you.
4. Reply with boolean success.
```

---

#### upload_profile_images.php

##### curl

```bash
curl -X POST \
  https://api.yourhost.tld/create_profile/upload_profile_images.php \
  -H "Authorization: Bearer <JWT>" \
  -F "images[]=@img0.jpg" -F "images[]=@img1.jpg" \
  -F "images=[]"
```

##### echo

**success**

```json
{"success":true,"images":["https://cdn/img0.jpg","https://cdn/img1.jpg"]}
```

**fail – no files**

```json
{"success":false,"error":"At least one image required"}
```

##### explanation

```
1. Process each images[] part → move to /uploads & generate CDN URL.
2. Push array into users.profile.topSection.profileImages.
3. Delete previously removed images if client passed a trimmed list.
```

---

#### elements_update.php

##### curl

```bash
curl -X POST \
  https://api.yourhost.tld/create_profile/elements_update.php \
  -H "Authorization: Bearer <JWT>" \
  -H "Content-Type: application/json" \
  -d '{"questions_and_answers":[{"question":"My talent","answer":"Juggling"}]}'
```

##### echo

**success**

```json
{"success":true,"message":"Elements saved"}
```

**fail – too many entries**

```json
{"success":false,"error":"Maximum 5 Q&A pairs"}
```

##### explanation

```
1. Replace users.profile.elements with indexed map of provided Q&A pairs.
2. Enforce max length & sanitize HTML.
```

---

#### update_loc.php

##### curl

```bash
curl -X POST \
  https://api.yourhost.tld/create_profile/update_loc.php \
  -H "Authorization: Bearer <JWT>" \
  -H "Content-Type: application/json" \
  -d '{"latitude":52.5163,"longitude":13.3777,"dating_radius":40}'
```

##### echo

**success**

```json
{"success":true}
```

**fail – geo out of range**

```json
{"success":false,"error":"Invalid coordinates"}
```

##### explanation

```
1. Validate lat ∈ [-90,90], lon ∈ [-180,180], radius ∈ [10,100].
2. Update users.location & users.profile.settings.datingRadius atomically.
```

---

#### update_open_to.php

##### curl

```bash
curl -X POST \
  https://api.yourhost.tld/create_profile/update_open_to.php \
  -H "Authorization: Bearer <JWT>" \
  -H "Content-Type: application/json" \
  -d '{"open_to":["hiking","cooking"]}'
```

##### echo

**success**

```json
{"success":true,"message":"Activities updated"}
```

**fail – empty array**

```json
{"success":false,"error":"Choose at least one activity"}
```

##### explanation

```
Upserts array into users.profile.settings.openTo using $set.
```

---

#### update_searching_for.php

##### curl

```bash
curl -X POST \
  https://api.yourhost.tld/create_profile/update_searching_for.php \
  -H "Authorization: Bearer <JWT>" \
  -H "Content-Type: application/json" \
  -d '{"genders":["Male","Female"],"ageRange":[25,35],"religion":["Don't care"],"politics":["Liberal"]}'
```

##### echo

**success**

```json
{"success":true,"message":"Filters saved"}
```

**fail – bad age range**

```json
{"success":false,"error":"Age max must be ≥ min and ≤120"}
```

##### explanation

```
Writes users.profile.searchFilters and ensures future match‑queries use the same structure.
```

---

#### get_profile.php

##### curl

```bash
curl -X GET \
  https://api.yourhost.tld/create_profile/get_profile.php \
  -H "Authorization: Bearer <JWT>"
```

##### echo

**success (truncated)**

```json
{
  "top_section": {"name":"Alice","age":"29",…},
  "elements": {"0": {"type":"icebreaker",…}}
}
```

**fail – token**

```json
{"error":"Invalid token"}
```

##### explanation

```
Aggregates user document into the flattened view model expected by Flutter screens, including presigned CDN URLs for images.
```

---


