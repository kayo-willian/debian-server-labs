# Lab 04 — Nginx Web Server on Debian

## Overview

This laboratory consists of deploying and configuring **Nginx as a web server on Debian GNU/Linux**.

The objective was to understand the basic operation of a web server in a real Linux environment, including:

* Nginx installation
* systemd service management
* web server directory structure
* server block configuration
* changing the listening port
* hosting a custom HTML page
* Nginx configuration validation
* HTTP request testing
* Nginx access and error logs
* Nginx Stub Status
* troubleshooting configuration and service problems

The laboratory was performed on a lightweight Debian Server environment.

---

## Objectives

* Install Nginx using Debian's package manager.
* Understand where Nginx stores its configuration and website files.
* Configure a custom web root.
* Run Nginx on a non-default HTTP port.
* Create and serve a custom HTML page.
* Validate the configuration before applying changes.
* Test the server using `curl`.
* Inspect Nginx logs.
* Enable the `stub_status` endpoint.
* Practice troubleshooting a real Linux service.

---

## Environment

| Component            | Configuration                      |
| -------------------- | ---------------------------------- |
| Operating System     | Debian GNU/Linux 13 (Trixie)       |
| Architecture         | x86_64                             |
| Web Server           | Nginx                              |
| HTTP Port            | 8080                               |
| Web Root             | `/var/www/nginx-site`              |
| Main Configuration   | `/etc/nginx/nginx.conf`            |
| Server Configuration | `/etc/nginx/sites-enabled/default` |
| Access Log           | `/var/log/nginx/access.log`        |
| Error Log            | `/var/log/nginx/error.log`         |
| Service Manager      | systemd                            |

The Nginx package is available directly in Debian 13's repositories.

---

# 1. Installing Nginx

The first step was updating the package index:

```bash
sudo apt update
```

Nginx was then installed with:

```bash
sudo apt install nginx
```

After installation, the service was checked with:

```bash
systemctl status nginx
```

The expected result is an active service:

```text
Active: active (running)
```

Nginx can also be managed through systemd:

```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl enable nginx
```

---

# 2. Understanding the Nginx Structure

The main configuration file is:

```text
/etc/nginx/nginx.conf
```

Debian also provides a site configuration structure:

```text
/etc/nginx/sites-available/
/etc/nginx/sites-enabled/
```

The active default server configuration used in this laboratory was:

```text
/etc/nginx/sites-enabled/default
```

The website itself was stored under:

```text
/var/www/nginx-site
```

The resulting structure was approximately:

```text
/etc/nginx/
├── nginx.conf
├── sites-available/
│   └── default
└── sites-enabled/
    └── default

/var/www/
└── nginx-site/
    └── index.html

/var/log/nginx/
├── access.log
└── error.log
```

---

# 3. Creating the Web Root

A dedicated directory was created for the laboratory website:

```bash
sudo mkdir -p /var/www/nginx-site
```

The main page was created as:

```text
/var/www/nginx-site/index.html
```

The page was intentionally created as a simple technical laboratory page rather than a personal portfolio.

Its purpose was to provide a real HTTP resource that Nginx could serve.

---

# 4. Configuring the Server

The default Nginx server configuration was edited:

```bash
sudo nano /etc/nginx/sites-enabled/default
```

The server was configured to use port `8080` instead of the default HTTP port.

The relevant configuration became conceptually:

```nginx
server {
    listen 8080 default_server;
    listen [::]:8080 default_server;

    server_name _;

    root /var/www/nginx-site;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Why port 8080?

Port 80 was already being used by another service on the server.

Instead of removing or reconfiguring that existing service, Nginx was moved to port `8080`.

This allowed multiple HTTP-related services to coexist on the same machine:

```text
Port 80
└── Existing web service

Port 8080
└── Nginx
```

This was an important troubleshooting and configuration decision during the laboratory.

---

# 5. Understanding `server`

The Nginx configuration uses a `server` block to define how requests should be handled.

Example:

```nginx
server {
    listen 8080;
    server_name _;
    root /var/www/nginx-site;
}
```

The important directives are:

### `listen`

Defines the port on which Nginx accepts HTTP connections.

```nginx
listen 8080;
```

### `server_name`

Defines the hostname associated with the server block.

```nginx
server_name _;
```

The `_` is commonly used as a catch-all/default server name.

### `root`

Defines where Nginx looks for requested files.

```nginx
root /var/www/nginx-site;
```

### `index`

Defines the default file returned when requesting `/`.

```nginx
index index.html;
```

### `location`

Defines how requests matching a particular URI should be handled.

```nginx
location / {
    try_files $uri $uri/ =404;
}
```

---

# 6. Validating the Configuration

One of the most important commands learned in this laboratory was:

```bash
sudo nginx -t
```

The command checks the Nginx configuration syntax and attempts to verify that referenced files can be opened.

Successful validation produced:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

This should be done **before reloading Nginx after configuration changes**.

The official Nginx documentation describes `-t` as the configuration test option.

---

# 7. Applying Configuration Changes

After modifying the configuration, Nginx was reloaded:

```bash
sudo systemctl reload nginx
```

A reload applies the new configuration without requiring a complete service stop.

This is different from:

```bash
sudo systemctl restart nginx
```

A restart stops and starts the service again, while a reload tells Nginx to load the new configuration and gracefully replace the workers.

---

# 8. Testing the Web Server

The server was tested locally with:

```bash
curl -I http://127.0.0.1:8080/
```

The successful response was:

```text
HTTP/1.1 200 OK
Server: nginx
Content-Type: text/html
Content-Length: 9475
Connection: keep-alive
```

### What this proves

The response confirms that:

1. Nginx is running.
2. Nginx is listening on port `8080`.
3. The HTTP request reached Nginx.
4. Nginx found the requested resource.
5. Nginx successfully returned the HTML page.

The most important line is:

```text
HTTP/1.1 200 OK
```

This represents a successful HTTP response.

---

# 9. Testing the Server Without a Browser

`curl` was used throughout the laboratory because it allows HTTP behavior to be tested directly from the terminal.

Basic request:

```bash
curl http://127.0.0.1:8080/
```

Headers only:

```bash
curl -I http://127.0.0.1:8080/
```

This is useful when troubleshooting because it allows the administrator to determine whether the problem is:

* Nginx itself
* the listening port
* the server configuration
* the web root
* the requested file
* or the client/browser

---

# 10. Nginx Logs

Nginx maintains two important logs.

### Access log

```text
/var/log/nginx/access.log
```

It records HTTP requests received by the server.

For example:

```bash
sudo tail -f /var/log/nginx/access.log
```

This can be used while making requests:

```bash
curl http://127.0.0.1:8080/
```

The request should appear in the access log.

### Error log

```text
/var/log/nginx/error.log
```

It records errors and problems related to Nginx.

To monitor it:

```bash
sudo tail -f /var/log/nginx/error.log
```

These logs are fundamental when troubleshooting a web server.

---

# 11. Inspecting the Complete Configuration

Nginx provides:

```bash
sudo nginx -T
```

This command tests the configuration and prints the complete configuration currently loaded by Nginx.

It was useful during the laboratory to confirm:

* which configuration files were loaded
* which port Nginx was listening on
* which document root was configured
* which server block was active
* which locations were defined

---

# 12. Nginx Stub Status

During the laboratory, the Nginx `stub_status` module was also enabled.

The configuration added to the server block was:

```nginx
location /stub_status {
    stub_status;
    allow 127.0.0.1;
    deny all;
}
```

This exposes basic Nginx runtime statistics.

The official Nginx documentation describes `stub_status` as providing basic status information such as active connections, accepted connections, handled connections, requests, reading, writing and waiting connections.

After modifying the configuration:

```bash
sudo nginx -t
```

Then:

```bash
sudo systemctl reload nginx
```

The endpoint was tested with:

```bash
curl http://127.0.0.1:8080/stub_status
```

The successful response was:

```text
Active connections: 1
server accepts handled requests
 7 7 9
Reading: 0 Writing: 1 Waiting: 0
```

This confirmed that the status endpoint was working.

---

# 13. Understanding `stub_status`

The returned information can be interpreted as follows:

```text
Active connections: 1
```

Current active client connections.

```text
7 7 9
```

These values represent:

```text
accepts
handled
requests
```

So in this test:

```text
Accepted connections: 7
Handled connections: 7
Requests: 9
```

The final line:

```text
Reading: 0
Writing: 1
Waiting: 0
```

represents the current connection states.

The official Nginx documentation defines these counters and states in detail.

---

# 14. Troubleshooting Performed

## Problem 1 — Existing HTTP service

### Symptom

Port 80 was already being used by another web service.

### Problem

Nginx could not simply take over the default HTTP port without interfering with the existing service.

### Resolution

Nginx was configured to listen on:

```text
8080
```

This allowed the services to coexist.

---

## Problem 2 — Verifying whether Nginx was actually serving the page

Instead of assuming the service was working, the server was tested directly:

```bash
curl -I http://127.0.0.1:8080/
```

Result:

```text
HTTP/1.1 200 OK
Server: nginx
```

### Resolution

The HTTP service was confirmed to be operational.

---

## Problem 3 — Configuration changes

After modifying Nginx configuration, it was necessary to determine whether the syntax was valid.

The following command was used:

```bash
sudo nginx -t
```

Result:

```text
syntax is ok
test is successful
```

Only after successful validation was the configuration reloaded:

```bash
sudo systemctl reload nginx
```

---

## Problem 4 — Verifying the status endpoint

The `stub_status` endpoint was added and tested independently before considering it part of the monitoring architecture.

Test:

```bash
curl http://127.0.0.1:8080/stub_status
```

Result:

```text
Active connections: 1
server accepts handled requests
 7 7 9
Reading: 0 Writing: 1 Waiting: 0
```

This proved that the Nginx module and endpoint were functioning.

---

# 15. Useful Diagnostic Commands

### Check service status

```bash
systemctl status nginx
```

### Check Nginx version

```bash
nginx -v
```

### Validate configuration

```bash
sudo nginx -t
```

### Display complete configuration

```bash
sudo nginx -T
```

### Check listening ports

```bash
sudo ss -ltnp | grep nginx
```

### Test HTTP

```bash
curl -I http://127.0.0.1:8080/
```

### Test Stub Status

```bash
curl http://127.0.0.1:8080/stub_status
```

### Watch access log

```bash
sudo tail -f /var/log/nginx/access.log
```

### Watch error log

```bash
sudo tail -f /var/log/nginx/error.log
```

### Reload configuration

```bash
sudo systemctl reload nginx
```

### Restart service

```bash
sudo systemctl restart nginx
```

### Stop service

```bash
sudo systemctl stop nginx
```

### Start service

```bash
sudo systemctl start nginx
```

---

# 16. Final Architecture

The final laboratory architecture is:

```text
                    Debian Server
                         │
                         │
                    ┌────▼────┐
                    │  Nginx  │
                    └────┬────┘
                         │
                  HTTP : 8080
                         │
                  ┌──────▼──────┐
                  │ nginx-site  │
                  │  index.html │
                  └─────────────┘
                         │
                         │
                /stub_status
                         │
                  Runtime metrics
```

---

# 17. What Was Learned

This laboratory provided practical experience with a real Linux web server.

The main concepts practiced were:

* Installing software with APT.
* Managing services with systemd.
* Understanding Nginx configuration files.
* Understanding server blocks.
* Configuring listening ports.
* Configuring a web root.
* Serving static HTML content.
* Testing HTTP using `curl`.
* Validating configuration before applying changes.
* Reloading services safely.
* Reading service logs.
* Inspecting listening sockets.
* Troubleshooting port conflicts.
* Exposing Nginx runtime information with `stub_status`.

The most important practical lesson was the troubleshooting workflow:

```text
Change configuration
        ↓
nginx -t
        ↓
Configuration valid?
        ↓
       YES
        ↓
systemctl reload nginx
        ↓
curl
        ↓
HTTP response
        ↓
Inspect logs if necessary
```

This workflow is more important than memorizing individual Nginx commands.

---

# 18. References

* Nginx official documentation — installation and packages
* Nginx official documentation — HTTP Stub Status module
* Debian package repository — Nginx package for Debian 13

Official Nginx documentation confirms the available Linux installation methods and configuration behavior.

The Debian package repository lists Nginx as an available package for Debian 13 (Trixie).

---

## Status

**Completed**

The Debian server is running Nginx as a functional HTTP server on port `8080`, serving a custom laboratory website and exposing a local `stub_status` endpoint for runtime information.
