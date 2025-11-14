# Dockerizing a Simple Python Flask Application

## Lab 5B – Dockerizing a Simple Python Flask Application

**Step-by-step student guide**

---

### 0. Before you start

**You should already have:**

* Docker Desktop / Docker Engine installed and running
* Basic terminal / command prompt knowledge (cd, ls, mkdir, etc.)
* Internet connection (to pull the Python base image the first time)

> If Docker Desktop is installed, make sure it is **running** (look for the Docker whale icon in your system tray).



---

## Step 1: Create the project folder

We’ll keep everything for this lab in one folder.

1. **Open a terminal / command prompt**

   * **Windows (PowerShell):** Press `Win + X` → choose *Windows PowerShell* or *Terminal*
   * **macOS:** Open *Terminal* from Applications → Utilities
   * **Linux:** Open your preferred terminal

2. **Create a new folder called `flask_app_docker` and go into it**

   ```bash
   mkdir flask_app_docker
   cd flask_app_docker
   ```

3. Verify you are in the right folder:

   ```bash
   pwd      # macOS / Linux
   # or
   cd       # Windows (just shows current directory)
   ```

You should see a path ending with `/flask_app_docker`.

---

## Step 2: Create the Flask application (`app.py`)

We’ll now create a minimal Flask app that returns a simple message.

1. **Create a file named `app.py` in `flask_app_docker`**

   You can use any editor:

   * VS Code: `code app.py`
   * Nano (macOS/Linux): `nano app.py`
   * Notepad (Windows): `notepad app.py`

2. **Put this code into `app.py`:**

   ```python
   from flask import Flask

   app = Flask(__name__)

   @app.route('/')
   def hello_world():
       return 'Hello, Dockerized Flask App!'

   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000)
   ```

3. **Save the file.**

### What this code does (quick explanation)

* `Flask(__name__)` creates the web application object.
* `@app.route('/')` defines a URL route for the root path (`/`).
* `hello_world()` runs when someone visits `http://<server>:5000/` and returns the message.
* `app.run(host='0.0.0.0', port=5000)` tells Flask to:

  * listen on all network interfaces (`0.0.0.0`)
  * on port `5000` — this is important later when mapping ports in Docker.

---

## Step 3: Create `requirements.txt`

This file lists the Python libraries needed in the container.

1. In the same folder (`flask_app_docker`), create a file called `requirements.txt`.

2. Add this line:

   ```text
   Flask==2.3.2
   ```

3. Save the file.

> You can change the version later if needed, but for this lab, keep it as shown so it matches the Docker build instructions.

---

## Step 4: Create the `Dockerfile`

The `Dockerfile` tells Docker **how to build** your image: which base image to use, which files to copy, which commands to run, etc.

1. In `flask_app_docker`, create a file named **exactly**:

   **`Dockerfile`** (no extension like `.txt`)

2. Put the following content inside:

   ```dockerfile
   # Use an official Python runtime as a parent image
   FROM python:3.9-slim-buster

   # Set the working directory in the container
   WORKDIR /app

   # Copy the current directory contents into the container at /app
   COPY . /app

   # Install any needed packages specified in requirements.txt
   RUN pip install --no-cache-dir -r requirements.txt

   # Make port 5000 available to the world outside this container
   EXPOSE 5000

   # Define environment variable
   ENV NAME World

   # Run app.py when the container launches
   CMD ["python", "app.py"]
   ```

3. Save the file.

Your Directory structure should look like this:

```
flask_app_docker/
├── app.py
├── requirements.txt
└── Dockerfile
```



### Explanation of each instruction

* `FROM python:3.9-slim-buster`

  * Uses the official Python 3.9 Docker image
  * `slim-buster` is a smaller, lightweight image (faster deploy, smaller size)

* `WORKDIR /app`

  * Sets the working directory *inside* the container to `/app`
  * All subsequent commands (`COPY`, `RUN`, etc.) will be relative to `/app`

* `COPY . /app`

  * Copies everything from your current folder (host) into `/app` in the container
  * This includes `app.py`, `requirements.txt`, and the `Dockerfile` itself

* `RUN pip install --no-cache-dir -r requirements.txt`

  * Installs the dependencies listed in `requirements.txt`
  * `--no-cache-dir` avoids caching downloaded packages, reducing image size

* `EXPOSE 5000`

  * Documents that the application in the container uses port 5000
  * This does **not** publish the port by itself — we do that with `docker run -p`

* `ENV NAME World`

  * Sets an environment variable `NAME` inside the container
  * Not used by the app here, but shows how to set environment variables

* `CMD ["python", "app.py"]`

  * This is the default command that runs when a container starts
  * It executes `python app.py` inside `/app`, starting the Flask server

---

## Step 5: Build the Docker image

Now we’ll build a Docker image from the Dockerfile.

1. Make sure your terminal is still in the `flask_app_docker` directory (the one containing `Dockerfile`).

   ```bash
   ls
   # You should see:
   # app.py  requirements.txt  Dockerfile
   ```

2. **Build the image** using:

   ```bash
   docker build -t flask-app:v1 .
   ```

   * `-t flask-app:v1`

     * Names (tags) the image as `flask-app` and version `v1`.
   * `.`

     * Means “use the Dockerfile in the current directory as the build context”.

3. **Watch the build logs:**

   You should see steps like:

   * `Step 1/7 : FROM python:3.9-slim-buster`
   * `Step 2/7 : WORKDIR /app`
   * ...
   * `Successfully built <IMAGE_ID>`
   * `Successfully tagged flask-app:v1`

4. **Check that the image exists:**

   ```bash
   docker images
   ```

   Look for a row similar to:

   | REPOSITORY | TAG | IMAGE ID | ... |
   | ---------- | --- | -------- | --- |
   | flask-app  | v1  | abc123…  |     |

---

## Step 6: Run the Docker container

Now we’ll run a container based on the image and access the Flask app from the browser.

1. **Run the container:**

   ```bash
   docker run -p 5000:5000 -d flask-app:v1
   ```

   Explanation of options:

   * `-p 5000:5000`

     * Maps **host port 5000** → **container port 5000**
     * So when you open `http://localhost:5000` on your machine, traffic goes to the container.
   * `-d`

     * Runs the container in **detached** mode (in the background).
   * `flask-app:v1`

     * Tells Docker which image to run.

2. If successful, Docker prints a **container ID** (a long alphanumeric string).

3. **Verify that the container is running:**

   ```bash
   docker ps
   ```

   You should see something like:

   | CONTAINER ID | IMAGE        | COMMAND         | STATUS | PORTS                  | NAMES       |
   | ------------ | ------------ | --------------- | ------ | ---------------------- | ----------- |
   | 123abc…      | flask-app:v1 | "python app.py" | Up ... | 0.0.0.0:5000->5000/tcp | brave_morse |

   * Focus on:

     * `IMAGE` column: `flask-app:v1`
     * `PORTS` column: `0.0.0.0:5000->5000/tcp`

---

## Step 7: Test the Flask app

1. Open a web browser.

2. In the address bar, go to:

   ```text
   http://localhost:5000
   ```

3. You should see the message:

   **`Hello, Dockerized Flask App!`**

If you see this, your Flask app is successfully running **inside** a Docker container and accessible from your host machine.

### Optional: Test using `curl` (terminal)

If you have `curl` installed:

```bash
curl http://localhost:5000
```

Expected output:

```text
Hello, Dockerized Flask App!
```

---

## Step 8: Check container logs (optional but useful)

Sometimes your app might crash or show errors. Logs help you debug.

1. Get the container ID:

   ```bash
   docker ps
   ```

2. View logs:

   ```bash
   docker logs <YOUR_CONTAINER_ID>
   ```

   Example:

   ```bash
   docker logs 123abc456def
   ```

You should see Flask’s startup logs, e.g.:

* `Running on http://0.0.0.0:5000/`
* `Press CTRL+C to quit` (if it were running in foreground)

---

## Step 9: Clean up (stop and remove container & image)

After testing, you may want to stop and remove the container and image to free up resources.

### 9.1 Stop the running container

1. List running containers:

   ```bash
   docker ps
   ```

2. Note the `CONTAINER ID` for `flask-app:v1`.

3. Stop it:

   ```bash
   docker stop <YOUR_CONTAINER_ID>
   ```

   Example:

   ```bash
   docker stop 123abc456def
   ```

### 9.2 Remove the container

After stopping:

```bash
docker rm <YOUR_CONTAINER_ID>
```

Example:

```bash
docker rm 123abc456def
```

### 9.3 Remove the Docker image (optional)

If you no longer need the image:

```bash
docker rmi flask-app:v1
```

> If Docker says the image is “in use”, make sure **all containers** created from `flask-app:v1` are stopped and removed.

---

Here is a **new Step 10** written in clean Markdown format, fully aligned with your lab style.
It includes **instructions**, **expected output**, and **evidence capture requirement**.

You can paste this into your README or lab sheet directly.

---

# ✅ **Step 10: Practice Exercise – Customize Your Flask Website With Your Name**

In this exercise, you will customize your Flask application's homepage so that the webpage displays **your name, admin number, and class**.
This proves that you successfully modified and rebuilt your Docker image.

---

## **🛠️ Task Requirements**

You must update your Flask application so that visiting:

**[http://localhost:5000](http://localhost:5000)**

shows **your own customized message**, for example:

```
Hello, this is John Doe (P1234567), DFSD/FT/02 – Dockerized Flask App!
```

You can implement this in **either Option A (hard-code in app.py)** or **Option B (environment variable)**.

---

## **🅰️ Option A — Hard-code your name in `app.py`**

Open **app.py** and replace the return string:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, this is YOUR NAME (ADMIN NO), YOUR CLASS – Dockerized Flask App!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

Example:

```python
return 'Hello, this is John Doe (P1234567), DOP/T02 – Dockerized Flask App!'
```

Rebuild and run:

```bash
docker build -t flask-app:v3 .
docker run -p 5000:5000 -d flask-app:v3
```

---

## **🅱️ Option B — Use an Environment Variable (Recommended)**

Modify `app.py` to use an environment variable:

```python
from flask import Flask
import os

app = Flask(__name__)

@app.route('/')
def hello_world():
    student = os.environ.get("NAME", "Student")
    return f'Hello, this is {student} – Dockerized Flask App!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

Rebuild:

```bash
docker build -t flask-app:v3 .
```

Run with your name injected:

```bash
docker run -p 5000:5000 -e NAME="Tan Wei Ming (P1234567), DOP/T02" -d flask-app:v3
```

Expected output:

```
Hello, this is John Doe (P1234567), DOP/T02 – Dockerized Flask App!
```

---

# 📸 **Submission Requirement for Step 10**

You must capture **two screenshots** as evidence:

### **1. Zipped the flask_app_docker and submit**

### **2. Screenshot of your customized webpage**

Take a screenshot of your browser showing:

👉 **[http://localhost:5000](http://localhost:5000)**

with your **name, admin number, and class clearly visible**.

Example expected browser output:

> *Hello, this is John Doe (P1234567), DOP/T02 – Dockerized Flask App!*

---

---

## Common issues & troubleshooting

**1. “Port already in use” error when running `docker run`**

* Symptom: Docker says something like `Bind for 0.0.0.0:5000 failed: port is already allocated`.
* Fix:

  * Check if another container is using port 5000: `docker ps`
  * Or another app on your host is using port 5000.
  * Option 1: Stop the conflicting container or app.
  * Option 2: Use a different host port, e.g.:

    ```bash
    docker run -p 5001:5000 -d flask-app:v1
    ```

    Then access: `http://localhost:5001`.

---

**2. “ModuleNotFoundError: No module named 'flask'”**

* Possible causes:

  * `requirements.txt` missing or not copied.
  * Typo in the filename (`requirement.txt` instead of `requirements.txt`).
* Fix:

  * Ensure the file is named `requirements.txt` (correct spelling, all lowercase).
  * Rebuild the image (so Docker re-runs the `RUN pip install` step):

    ```bash
    docker build -t flask-app:v1 .
    docker run -p 5000:5000 -d flask-app:v1
    ```

---

**3. Browser shows “This site can’t be reached”**

Checklist:

1. Is the container running?

   ```bash
   docker ps
   ```

   If you don’t see `flask-app:v1`, start it again.

2. Are you using the correct URL?

   * Use `http://localhost:5000`
   * If you changed the host port (e.g. `-p 5001:5000`), use `http://localhost:5001`.

3. Did you change the code in `app.py` and forget to rebuild?

   * If you changed `app.py`, rebuild the image and rerun the container.

---

## Quick recap

By completing this lab, you have:

1. Created a simple **Flask** web application (`app.py`).
2. Listed dependencies in **`requirements.txt`**.
3. Written a **Dockerfile** to containerize the app.
4. Built a Docker image using `docker build`.
5. Ran the Flask app inside a Docker container with `docker run -p`.
6. Verified the app from your browser at `http://localhost:5000`.
7. Stopped and cleaned up containers and images.

You now know the full cycle of **Dockerizing a simple Python Flask application**, which is a key skill for deploying microservices and web apps.
