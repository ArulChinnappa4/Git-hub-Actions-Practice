

```text
Calculator/
│
├── index.html
├── style.css
└── script.js
```

We will use **Nginx** inside Docker to serve your calculator.

The final result will be:

```text
Browser
   │
   │ http://localhost:8080
   ↓
Docker Container
   │
   │ Nginx
   ↓
index.html
   ├── style.css
   └── script.js
```

## 1. Open your Calculator project

Your VS Code folder should be:

```text
Calculator
├── index.html
├── style.css
└── script.js
```

Make sure `index.html` references your CSS and JS correctly:

```html
<link rel="stylesheet" href="style.css">
```

and:

```html
<script src="script.js"></script>
```

---

# 2. Install/verify Docker

Open **PowerShell** or the VS Code terminal:

```bash
docker --version
```

You should see something similar to:

```text
Docker version 28.x.x
```

Also check:

```bash
docker run hello-world
```

If Docker is working, you'll get a message saying Docker successfully ran the container.

> On Windows, make sure **Docker Desktop is running**.

---

# 3. Create a Dockerfile

Inside your `Calculator` folder, create a file named exactly:

```text
Dockerfile
```

**No `.txt` extension.**

Your project becomes:

```text
Calculator/
│
├── Dockerfile
├── index.html
├── style.css
└── script.js
```

---

# 4. Write the Dockerfile

Put this inside `Dockerfile`:

```dockerfile
FROM nginx:alpine

COPY . /usr/share/nginx/html

EXPOSE 80
```

Let's understand each line.

### `FROM nginx:alpine`

```dockerfile
FROM nginx:alpine
```

This tells Docker:

> Start with an Nginx web server image.

`alpine` is a lightweight Linux distribution, so the image is relatively small.

---

### `COPY . /usr/share/nginx/html`

```dockerfile
COPY . /usr/share/nginx/html
```

This copies your calculator files:

```text
index.html
style.css
script.js
```

into Nginx's web directory:

```text
/usr/share/nginx/html
```

So inside the container it becomes approximately:

```text
/usr/share/nginx/html/
│
├── index.html
├── style.css
└── script.js
```

---

### `EXPOSE 80`

```dockerfile
EXPOSE 80
```

Nginx normally listens on port **80** inside the container.

This tells Docker:

> The application inside this container uses port 80.

Important: `EXPOSE` itself does **not** publish the port to your computer. We will do that when running the container.

---

# 5. Build the Docker image

Open the terminal **inside your Calculator folder**.

For example:

```powershell
cd path\to\Calculator
```

Check:

```bash
dir
```

You should see:

```text
Dockerfile
index.html
style.css
script.js
```

Now build the image:

```bash
docker build -t calculator-app .
```

### What does this mean?

```text
docker build
```

Build a Docker image.

```text
-t calculator-app
```

Give the image the name:

```text
calculator-app
```

```text
.
```

Use the current directory as the build context.

You should eventually see something similar to:

```text
Successfully tagged calculator-app:latest
```

---

# 6. Check your Docker image

Run:

```bash
docker images
```

You should see something like:

```text
REPOSITORY       TAG       IMAGE ID       SIZE
calculator-app   latest    xxxxxxxx       ...
nginx            alpine    xxxxxxxx       ...
```

Your calculator is now packaged into a Docker **image**.

---

# 7. Run the container

Now run:

```bash
docker run -d -p 8080:80 --name calculator-container calculator-app
```

This is the most important command.

Let's break it down:

### `docker run`

Create and start a container.

### `-d`

Run in detached mode.

The terminal remains available.

### `-p 8080:80`

This connects:

```text
Your computer        Container
     8080      →       80
```

So:

```text
localhost:8080
```

will reach:

```text
Nginx :80
```

inside the container.

### `--name calculator-container`

Give the container a name:

```text
calculator-container
```

### `calculator-app`

Use the image we built earlier.

---

# 8. Open your calculator in the browser

Now open:

```text
http://localhost:8080
```

You should see your calculator. 🎉

The complete flow is:

```text
http://localhost:8080
        │
        ▼
Your Windows machine
        │
        │ port 8080
        ▼
Docker container
        │
        │ port 80
        ▼
Nginx
        │
        ▼
index.html
        │
        ├── style.css
        │
        └── script.js
```

---

# 9. Check whether the container is running

Run:

```bash
docker ps
```

You should see:

```text
CONTAINER ID   IMAGE           PORTS
xxxxxxxx       calculator-app  0.0.0.0:8080->80/tcp
```

The important part is:

```text
8080->80
```

It means:

```text
Host port 8080
       ↓
Container port 80
```

---

# 10. Stop the calculator

If you want to stop the container:

```bash
docker stop calculator-container
```

Check:

```bash
docker ps
```

It should no longer appear among running containers.

---

# 11. Start it again

You don't need to build the image again.

Just:

```bash
docker start calculator-container
```

Then open:

```text
http://localhost:8080
```

---

# 12. Remove the container

If you want to completely remove the container:

```bash
docker stop calculator-container
```

Then:

```bash
docker rm calculator-container
```

Your image still exists.

Check:

```bash
docker images
```

---

# 13. Remove the image

If you also want to delete the image:

```bash
docker rmi calculator-app
```

---

# 14. If you modify your calculator

Suppose you change:

```text
script.js
```

For example, you fix some calculator functionality.

The existing Docker image **doesn't automatically update**.

You need to rebuild the image:

```bash
docker build -t calculator-app .
```

Then recreate the container:

```bash
docker stop calculator-container
docker rm calculator-container
```

Run the new version:

```bash
docker run -d -p 8080:80 --name calculator-container calculator-app
```

Then:

```text
http://localhost:8080
```

will show your updated calculator.

---

# 15. Recommended `.dockerignore`

You can also create:

```text
.dockerignore
```

Your project becomes:

```text
Calculator/
│
├── Dockerfile
├── .dockerignore
├── index.html
├── style.css
└── script.js
```

Put this in `.dockerignore`:

```text
.git
.gitignore
README.md
Dockerfile
.dockerignore
```

This prevents unnecessary files from being sent to Docker during the build.

---

# 16. Complete command sequence

For your project, you can remember this workflow:

### Build

```bash
docker build -t calculator-app .
```

### Run

```bash
docker run -d -p 8080:80 --name calculator-container calculator-app
```

### Check

```bash
docker ps
```

### Open

```text
http://localhost:8080
```

### Stop

```bash
docker stop calculator-container
```

### Start again

```bash
docker start calculator-container
```

### Remove container

```bash
docker rm calculator-container
```

### Remove image

```bash
docker rmi calculator-app
```

---

## The important Docker concepts you're learning here

Your simple calculator is actually a good project for understanding the basic Docker workflow:

```text
                Dockerfile
                    │
                    │ docker build
                    ▼
              Docker IMAGE
              calculator-app
                    │
                    │ docker run
                    ▼
             Docker CONTAINER
          calculator-container
                    │
                    │ port mapping
                    ▼
        http://localhost:8080
                    │
                    ▼
              Your Calculator
```

### Image vs Container

Think of it like this:

**Image = blueprint/package**

```text
calculator-app
```

It contains:

```text
Nginx
+ index.html
+ style.css
+ script.js
```

**Container = running instance of that image**

```text
calculator-container
```

You can even create multiple containers from the same image:

```bash
docker run -d -p 8080:80 --name calculator1 calculator-app
docker run -d -p 8081:80 --name calculator2 calculator-app
```

Then:

```text
http://localhost:8080
        ↓
calculator1

http://localhost:8081
        ↓
calculator2
```

Both are running the same Docker image.

---

## One important point

`localhost:8080` is a **local URL**. It means the calculator is accessible from your own computer.

If your next goal is to understand the full Docker workflow, the natural next step would be:

```text
Calculator
    ↓
Dockerfile
    ↓
Docker Image
    ↓
Docker Container
    ↓
Docker Hub
    ↓
Pull image on another machine
    ↓
Run calculator
```
