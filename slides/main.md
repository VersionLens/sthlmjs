## Version Control full-stack JS apps



## Version control code

---

### Benefits: <!-- .element: class="fragment" -->

* Collaboration <!-- .element: class="fragment" -->
* Traceability <!-- .element: class="fragment" -->
* Experimentation <!-- .element: class="fragment" -->



## Version control code

---

### There's a standard: git
# 👍



## Version control apps

---

### There's no standard
# 😞



## Version control apps

---

### Why?
Same reasons as for code: <!-- .element: class="fragment" -->

* Collaboration <!-- .element: class="fragment" -->
* Traceability <!-- .element: class="fragment" -->
* Experimentation <!-- .element: class="fragment" -->



## Version control apps

---

### Let's come up with a standard



## What is an app version?

* An object mapping services to git commits


## Example:
services.json - stored in a "version repository"
```
{
    frontend: {
        repo: "store-frontend",
        branch: "feature/bankid-login",
        commit: "abc123..."
    },
    backend: {
        repo: "store-backend",
        branch: "feature/bankid-login",
        commit: "def456..."
    },
    ...
}
```



## What is an app version?

* ✅ Map services to commits
* 🔳 TODO: Define how the version is built <!-- .element: class="fragment" -->
* 🔳 TODO: Define how the version is run <!-- .element: class="fragment" -->


## Example:
Let's add build and run definitions to the version repository
```
services.json
├─ build/
│  ├─ frontend.json
│  ├─ backend.json
│  ├─ ...
├─ run/
│  ├─ frontend.json
│  ├─ backend.json
│  ├─ ...
```
Note: Build executed in a CI, run executed in a CD



## What is an app version?

* ✅ Map services to commits
* ✅ Define how the version is built
* ✅ Define how the version is run
* 🔳 TODO: Store encrypted build and run secrets <!-- .element: class="fragment" -->


## Example:
Let's add build and run definitions to the version repository
```
services.json
├─ build/
│  ├─ frontend.json
│  ├─ backend.json
│  ├─ ...
├─ run/
│  ├─ frontend.json
│  ├─ backend.json
│  ├─ ...
├─ secrets/
│  ├─ build-secrets.json.enc
│  ├─ run-secrets.json.enc
│  ├─ ...
```



## What is an app version?

* ✅ Map services to commits
* ✅ Define how the version is built
* ✅ Define how the version is run
* ✅ Store encrypted build and run secrets
* 🔳 TODO: Define seed data <!-- .element: class="fragment" -->


## Example:
Let's add seed data
```
services.json
├─ build/
│  ├─ frontend.json
│  ├─ ...
├─ run/
│  ├─ frontend.json
│  ├─ ...
├─ secrets/
│  ├─ build-secrets.json.enc
│  ├─ ...
├─ data/
│  ├─ postgres-seed.sql
│  ├─ redis.data
│  ├─ ...
```