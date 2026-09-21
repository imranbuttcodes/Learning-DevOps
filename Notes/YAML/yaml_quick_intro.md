# YAML — Quick Practical Lesson

## 1. What is YAML?

**YAML = YAML Ain't Markup Language.**

It's a **human-readable data/configuration format**.

You can think of it as a way of representing structured information:

```yaml
name: FastAPI App
version: 1.0
language: Python
```

It is **not a programming language** like Python.

It's mainly used for **configuration**.

You'll see it heavily in:

* GitHub Actions
* Docker Compose
* Kubernetes
* CI/CD systems
* application configuration

---

# 2. YAML is indentation-based

This is probably the most important rule.

```yaml
person:
  name: Imran
  age: 21
```

The spaces show hierarchy:

```text
person
 ├── name
 └── age
```

But this:

```yaml
person:
name: Imran
age: 21
```

doesn't represent the same structure.

### Use spaces, not tabs.

Usually **2 spaces** per indentation level.

---

# 3. Key → Value

The basic structure is:

```yaml
key: value
```

Example:

```yaml
name: Imran
language: Python
experience: beginner
```

Think:

```text
name       → Imran
language   → Python
experience → beginner
```

---

# 4. Nested data

You create nesting through indentation:

```yaml
student:
  name: Imran
  university: UCP
  skills:
    language: Python
    framework: FastAPI
```

Mental model:

```text
student
├── name
├── university
└── skills
    ├── language
    └── framework
```

Notice that `skills` itself contains another level.

---

# 5. Lists

YAML uses `-` for list items.

```yaml
skills:
  - Python
  - FastAPI
  - Docker
  - Git
```

Equivalent mental model:

```text
skills
 ├── Python
 ├── FastAPI
 ├── Docker
 └── Git
```

You can also have objects inside lists:

```yaml
users:
  - name: Imran
    role: developer

  - name: Ali
    role: tester
```

---

# 6. Strings

Usually you can simply write:

```yaml
name: Imran
```

You can also explicitly quote:

```yaml
name: "Imran"
```

Both represent a string in ordinary cases.

Quoting becomes useful when the value contains special characters or could be interpreted as another YAML type.

---

# 7. Numbers and booleans

```yaml
age: 21
gpa: 3.66
student: true
graduated: false
```

YAML understands these as different data types.

---

# 8. Comments

Use `#`:

```yaml
# Application configuration
name: FastAPI

# Port used by the server
port: 8000
```

Everything after `#` on that line is a comment.

---

# 9. The thing you've already seen: Docker Compose

Your previous `mongo.yaml` was basically this structure:

```yaml
services:
  mongo:
    image: mongo:7.0
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: qwerty

  mongo-express:
    image: mongo-express
    ports:
      - "8081:8081"
```

Let's understand the hierarchy:

```text
services
│
├── mongo
│   ├── image
│   ├── ports
│   └── environment
│
└── mongo-express
    ├── image
    └── ports
```

So YAML is basically giving Docker Compose a **structured configuration tree**.

---

# 10. YAML vs JSON

The same information can be represented in JSON:

```json
{
  "name": "Imran",
  "skills": ["Python", "FastAPI", "Docker"]
}
```

YAML:

```yaml
name: Imran
skills:
  - Python
  - FastAPI
  - Docker
```

YAML is generally easier for humans to write/read, which is why configuration-heavy tools use it.

---

# 11. The YAML you'll see in GitHub Actions

Now this becomes important.

A GitHub Actions workflow looks roughly like:

```yaml
name: CI

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        run: pytest
```

Don't worry about understanding the GitHub Actions keywords yet.

Just recognize the **YAML structure**:

```text
name
│
on
│
└── push
    └── branches
        └── main

jobs
│
└── test
    ├── runs-on
    └── steps
        ├── step 1
        ├── step 2
        └── step 3
```

That's why we're learning YAML **right before GitHub Actions**.

---

# The 5 YAML rules you actually need

For our DevOps work, remember these:

```text
1. key: value
2. Indentation = hierarchy
3. - = list item
4. # = comment
5. Use spaces, NOT tabs
```

And one big warning:

```yaml
# ❌ Wrong indentation
jobs:
test:
  runs-on: ubuntu-latest
```

versus:

```yaml
# ✅ Correct
jobs:
  test:
    runs-on: ubuntu-latest
```

YAML is extremely sensitive to indentation.

---

### Your YAML mental model

```text
YAML
 │
 ├── key: value
 │
 ├── indentation → hierarchy
 │
 ├── - → lists
 │
 ├── # → comments
 │
 └── configuration/data
          │
          ├── Docker Compose
          ├── GitHub Actions
          └── Kubernetes
```

