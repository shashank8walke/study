# Python: subprocess, logging, YAML, requests, decorators

---

## subprocess

**Use case:** Run external programs (git, ls, custom scripts) from Python. Not for concurrency — use `threading`/`asyncio` for that.

```python
import subprocess
```

### `subprocess.run()` — blocking, simple

Waits for the process to finish before continuing.

```python
result = subprocess.run(["python", "a.py", "arg1"])
# result is a CompletedProcess instance: .returncode, .stdout, .stderr
```

```python
subprocess.run(["git", "status"], check=True, timeout=5)
```

| Arg | Effect |
|---|---|
| `check=True` | Raises `CalledProcessError` if exit code != 0 |
| `timeout=N` | Raises `TimeoutExpired` after N seconds |
| `input="text"` | Sends text to stdin |
| `encoding='utf-8'` | Decode stdout/stderr as strings |
| `capture_output=True` | Capture stdout+stderr (shorthand for `stdout=PIPE, stderr=PIPE`) |

Exit code 0 = success, non-zero = failure (Unix convention).

```python
try:
    subprocess.run(["git", "push"], check=True, timeout=30)
except FileNotFoundError:
    print("Executable not found")
except subprocess.CalledProcessError as e:
    print(f"Failed with code {e.returncode}")
except subprocess.TimeoutExpired:
    print("Timed out")
```

> **Security:** Never construct commands from user input with `shell=True` — opens injection attacks. Prefer list form `["cmd", "arg"]`.

### `subprocess.Popen()` — non-blocking, full control

Runs the process in parallel. `run()` is a wrapper around `Popen`.

```python
proc = subprocess.Popen(["python", "server.py"], stdout=subprocess.PIPE)
# do other work here...
output, _ = proc.communicate()   # wait + get output
# or poll without blocking:
if proc.poll() is not None:      # None means still running
    print("Done")
```

### Piping between processes

```python
p1 = subprocess.Popen(["ps", "aux"], stdout=subprocess.PIPE)
p2 = subprocess.Popen(["grep", "python"], stdin=p1.stdout, stdout=subprocess.PIPE)
p1.stdout.close()
output, _ = p2.communicate()
```

### Quick comparison

| | `run()` | `Popen()` |
|---|---|---|
| Blocks | Yes | No |
| Returns | `CompletedProcess` | `Popen` object |
| Use when | Simple one-shot commands | Need parallel execution or streaming I/O |

---

## logging

**Levels (lowest → highest):** DEBUG(10) → INFO(20) → WARNING(30) → ERROR(40) → CRITICAL(50)

Default threshold is WARNING — anything below is silently dropped.

### Basic setup (simple scripts)

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    filename="app.log",
    mode="a",
    encoding="utf-8",
    format="{asctime} - {levelname} - {message}",
    style="{",
    datefmt="%Y-%m-%d %H:%M"
)

logging.debug("x = %s", x)
logging.info("Server started")
logging.warning("Disk space low")
logging.error("Connection failed")
```

### Custom logger (libraries / multi-module projects)

```python
logger = logging.getLogger(__name__)   # logger named after the module
logger.setLevel(logging.DEBUG)
```

Attach **handlers** (where to send logs) and **formatters** (how to format them):

```python
formatter = logging.Formatter(
    "{asctime} - {levelname} - {message}",
    style="{",
    datefmt="%Y-%m-%d %H:%M"
)

console_handler = logging.StreamHandler()        # → stdout
file_handler = logging.FileHandler("app.log", mode="a", encoding="utf-8")

console_handler.setFormatter(formatter)
file_handler.setFormatter(formatter)

logger.addHandler(console_handler)
logger.addHandler(file_handler)
```

> **Rule:** The logger's level is the gate. A handler set to DEBUG cannot show messages if the logger itself is set to WARNING.

You can set different levels per handler — e.g., INFO to file, WARNING to console.

### Filtering

```python
handler.addFilter(some_filter)   # filter can be a class, subclass of logging.Filter, or callable
```

---

## YAML

```bash
pip install pyyaml
```

```python
import yaml
```

### Load

```python
with open("config.yaml") as f:
    data = yaml.safe_load(f)       # returns Python dict/list
```

Always use `safe_load` for untrusted sources — `yaml.load()` can execute arbitrary Python via special tags.

### Dump

```python
yaml.dump(data)                         # returns YAML string
yaml.dump(data, default_flow_style=False)  # block style — human-readable
```

### Multi-document files

```python
docs = list(yaml.load_all(stream))
yaml.dump_all([doc1, doc2])
```

### YAML → Python type mapping

```yaml
containers:
  - name: app
    image: nginx
  - name: db
    image: postgres
```

```python
{"containers": [{"name": "app", "image": "nginx"}, {"name": "db", "image": "postgres"}]}
# list items → Python list, key: value → dict
```

### Anchors & aliases (reuse config blocks)

```yaml
defaults: &defaults
  timeout: 30
  retries: 3

production:
  <<: *defaults       # merges all keys from defaults
  timeout: 60         # overrides just this key
```

Useful in CI/CD pipelines and Kubernetes configs to avoid repetition.

---

## requests

```python
import requests
```

### Methods

```python
requests.get(url)                          # Read
requests.post(url, json={...})             # Create
requests.put(url, json={...})              # Replace entire resource
requests.patch(url, json={...})            # Partial update
requests.delete(url)                       # Delete
```

### Common options

```python
response = requests.get(
    url,
    headers={"Authorization": "Bearer TOKEN"},
    params={"page": 1},           # appended as ?page=1
    timeout=(3, 10)               # (connect_timeout, read_timeout) in seconds
)

response.raise_for_status()       # raises HTTPError for 4xx/5xx responses
print(response.status_code)
print(response.json())
print(response.text)
```

### Session (reuse connection + shared headers)

```python
with requests.Session() as session:
    session.headers.update({"Authorization": "Bearer TOKEN"})
    r1 = session.get(url1, timeout=5)
    r2 = session.get(url2, timeout=5)
```

Sessions persist cookies and reuse the underlying TCP connection — more efficient for multiple requests to the same host.

---

## Decorators

Functions are first-class in Python — they can be passed as arguments, returned, and assigned to variables.

```python
def run(func):        # func is passed without ()
    return func("Shashank")

run(greet)            # calls greet("Shashank")
```

### Basic decorator

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):   # accepts any arguments
        print("Before")
        result = func(*args, **kwargs)
        print("After")
        return result
    return wrapper

@my_decorator
def greet(name):
    print(f"Hi {name}")

# @my_decorator is syntactic sugar for: greet = my_decorator(greet)
```

Output:
```
Before
Hi Shashank
After
```

### Always use `functools.wraps`

Without it, the decorated function loses its `__name__`, `__doc__`, etc.:

```python
import functools

def my_decorator(func):
    @functools.wraps(func)          # preserves original function metadata
    def wrapper(*args, **kwargs):
        ...
        return func(*args, **kwargs)
    return wrapper
```

### Decorator with arguments (3-level nesting)

```python
def retry(max_attempts=3, delay=2):    # Level 1: decorator arguments
    def decorator(func):               # Level 2: the function being decorated
        @functools.wraps(func)
        def wrapper(*args, **kwargs):  # Level 3: the function's arguments
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts:
                        raise
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(max_attempts=5, delay=1)
def fetch_data(url):
    ...
```

### Common real-world uses

| Pattern | Purpose |
|---|---|
| `@retry` | Re-attempt on failure (API calls, flaky tests) |
| `@timer` | Measure execution time |
| `@poll` | Check until condition met or timeout |
| `@cache` / `@lru_cache` | Memoize expensive function results |
| `@require_auth` | Auth checks in web frameworks |
