
---

# 📘 Kubernetes: Object Names and UIDs

In Kubernetes, each object has two key identifiers:

- **Name**: Human-readable, unique *within the same namespace and resource type*.
- **UID**: System-generated, globally unique in the cluster for the *entire lifetime* of the cluster.

---

## 🏷️ Object **Names**

- **Name** is a string provided by the client/user to identify an object.
- Names are used in the **API URL**, like:  
  `/api/v1/pods/nginx-demo`

### 🔹 Uniqueness Rule:
- You **can’t have two objects** of the *same type and namespace* with the same name.
- But you **can** have different types (e.g., a Pod and a Deployment) both named `myapp-1234` in the same namespace.

> ✅ You can reuse a name **after deleting** the previous object.

---

## ⚠️ API Version Irrelevance for Names
Names must be **unique across all API versions** of the same resource type.  
✅ **API version does *not* affect name uniqueness.**

---

## 🧠 Example:
You can’t have two Pods named `nginx` in the same namespace. But:
- A Pod named `nginx` and
- A Deployment named `nginx`  
✅ are allowed in the same namespace.

---

## ⚠️ Special Case: Physical Resources
For physical entities like Nodes:
- If a Node (representing a physical host) is **recreated without deleting** the original, Kubernetes might **treat it as the same Node**, leading to **inconsistencies**.

---

## ⚙️ generateName
If you don’t specify a name, you can use `generateName`. Kubernetes adds a **random suffix** to the provided prefix.

### 📌 Example:
```yaml
metadata:
  generateName: nginx-
```
Might create: `nginx-x7s9f`

> ✅ Since v1.31, Kubernetes makes **up to 8 attempts** to avoid name collisions before giving a `HTTP 409 Conflict`.

---

## 📛 Naming Standards (Constraints)

Different resource types require different naming standards. Here are the 4 main types:

### 1. 🧾 **DNS Subdomain Names** (RFC 1123)

- Max 253 characters  
- Lowercase letters, numbers, `-`, `.`  
- Start & end with a letter or number

✅ **Most commonly used**

#### Example:  
```nginx.demo-site-123```

---

### 2. 🧾 **RFC 1123 Label Names**

- Max 63 characters  
- Lowercase letters, numbers, `-`  
- Start & end with a letter or number

#### Example:  
```nginx-demo-123```

---

### 3. 🧾 **RFC 1035 Label Names**

- Max 63 characters  
- Lowercase letters, numbers, `-`  
- **Must start with a letter** (not a number)  
- End with a letter or number

> ❗ Difference: RFC 1035 **cannot start with a digit**, unlike RFC 1123.

#### Valid Example:  
```demo-nginx```

#### ❌ Invalid Example (starts with digit):  
```1nginx``` (valid for RFC 1123, **not** RFC 1035)

---

### 4. 🧾 **Path Segment Names**

- Must be safe in URLs
- Cannot be `"."`, `".."`, `/`, or `%`

Used for objects embedded in URL paths.

---

## 🛠️ Example Pod Manifest (Using Name)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-demo
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

---

## 🔑 UIDs (Universally Unique Identifiers)

- **System-generated** unique string
- **Globally unique** across the whole cluster
- Once an object is created, its UID never changes
- Used to distinguish even historical versions of objects with same name

> 📌 Standardized by **ISO/IEC 9834-8** and **ITU-T X.667**

### 🔹 Example UID:
```
a123e4ab-69d7-4b0b-9cb1-8f33ffbf4281
```

---

## ✅ Summary Checklist

| Concept               | Key Point                                                                 |
|-----------------------|---------------------------------------------------------------------------|
| Name                  | Unique per object type + namespace                                        |
| UID                   | Unique globally, system-assigned                                          |
| generateName          | Adds random suffix to prefix                                              |
| Reusing names         | Allowed **only after deleting** the previous object                       |
| RFC naming standards  | Ensure resource names follow required DNS standards                       |
| Path constraints      | Some resources must have URL-safe names                                   |
| Physical resources    | Avoid reusing name unless old object is **deleted**                      |

