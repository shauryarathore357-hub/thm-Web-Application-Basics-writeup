# TryHackMe — Web Application Basics

> **Track:** Cyber Security 101 → Web Hacking
> **Difficulty:** Easy · **Time:** ~120 min
> **Focus:** HTTP, URLs, request methods, status codes, headers, and a hands-on API exercise

![Room overview](images/01-room-overview.png)

Every attack on the web begins the same way: a browser asks a question, and a server answers. Before you can break a web application, you have to understand exactly how that conversation works. This room is where that conversation stops being magic and starts being something you can read, predict, and eventually manipulate.

---

## Table of Contents
1. [Web Application Components](#1-web-application-components)
2. [Anatomy of a URL](#2-anatomy-of-a-url)
3. [HTTP Messages](#3-http-messages)
4. [The Request Line and Methods](#4-the-request-line-and-methods)
5. [Request Headers and Body](#5-request-headers-and-body)
6. [Response Status Line and Status Codes](#6-response-status-line-and-status-codes)
7. [Response Headers and Security Flags](#7-response-headers-and-security-flags)
8. [Hands-On: Talking to an API](#8-hands-on-talking-to-an-api)
9. [Key Takeaways](#9-key-takeaways)

---

## 1. Web Application Components

A web application is a team of components, and knowing who does what tells you where to look for weaknesses.

| Component | Role |
|---|---|
| **Web browser** | The client tool used to access and interact with web applications |
| **Web server** | Hosts the application and delivers content to clients |
| **Web Application Firewall (WAF)** | A protective layer that filters incoming traffic and blocks malicious requests |

![Web application components questions](images/02-web-app-components.png)

**Why it matters:** a WAF sits *in front of* the application. When an attack payload gets blocked, it may be the WAF, not the app, that stopped it. Recognising that difference is a core pentesting skill.

---

## 2. Anatomy of a URL

A URL is more than an address. Each part tells the server something different.

```
scheme://user:password@host:port/path?query#fragment
```

- **HTTPS** wraps HTTP in encryption so data between browser and server can't be read in transit.
- **Query string** (`?search=term&id=5`) passes extra information, such as search terms or form inputs, to the server. Anything a user can put here is something an attacker can tamper with.
- **Typosquatting** is registering misspelled variations of popular domains (`gooogle.com`) to catch users who mistype, often for fraud or phishing.

![URL and HTTPS questions](images/03-urls-and-https.png)

---

## 3. HTTP Messages

HTTP has exactly two message types:

- **HTTP request**: sent by the client to ask for something
- **HTTP response**: returned by the server after processing that request

Both share the same skeleton:

```
Start line
Header: value
Header: value
<empty line>
Body (optional)
```

The **empty line** after the headers is the delimiter that tells the receiver where headers end and the body begins.

![HTTP messages questions](images/04-http-messages.png)

---

## 4. The Request Line and Methods

The first line of every request holds three things: the **method**, the **path**, and the **HTTP version**.

```
GET /index.html HTTP/1.1
```

- **HTTP/1.1** is the most widely used version, introducing persistent connections and chunked transfer encoding.
- **URL path** is the part after the domain that names the specific resource or endpoint being requested.
- Common methods: `GET` (retrieve), `POST` (submit or update), `PUT` (replace), `DELETE` (remove), and **`OPTIONS`**, which describes the communication options for a resource and lets a client discover which methods the server supports.

![Request line questions](images/05-request-line.png)

---

## 5. Request Headers and Body

Headers are metadata that guide how the server processes the request.

```
POST /login HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 Firefox/87.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 27

username=alice&password=...
```

- **`Host`** specifies the domain name of the server the request is addressed to. It is required in HTTP/1.1 and lets one server host many sites.
- **`Content-Type`** describes the body format. The default for HTML form submissions is `application/x-www-form-urlencoded`, which encodes data as `key=value` pairs.
- **Request headers** as a group carry the host, user agent, content type, and more.

![Request headers and body questions](images/06-request-headers-body.png)

---

## 6. Response Status Line and Status Codes

The server's answer begins with a **status line**: the HTTP version, a status code, and a short reason phrase.

```
HTTP/1.1 200 OK
```

Status codes fall into five families:

| Range | Meaning |
|---|---|
| 1xx | Informational |
| 2xx | Success |
| 3xx | Redirection |
| 4xx | Client errors (the request was wrong) |
| 5xx | **Server errors** (the server hit an internal problem or couldn't fulfil the request) |

The one everybody has met: **404**, meaning the requested resource could not be found.

![Response status questions](images/07-response-status.png)

---

## 7. Response Headers and Security Flags

Response headers are where servers accidentally leak information, or deliberately defend themselves.

**Information leakage:** the `Server` header can reveal the web server's software and version, handing an attacker a shortlist of known vulnerabilities to try. Best practice is to remove or genericise it.

**Cookie hardening** via the `Set-Cookie` header:

```
Set-Cookie: session=abc123; Secure; HttpOnly
```

- **`Secure`**: the cookie is only sent over HTTPS, so it can't be exposed in unencrypted traffic.
- **`HttpOnly`**: the cookie can't be read by JavaScript, limiting the damage of an XSS attack.

![Response headers questions](images/08-response-headers.png)

---

## 8. Hands-On: Talking to an API

Theory becomes real in the practical exercise. The room provides an in-browser HTTP client (split view) where you choose a method, enter a URL, and hit **Go**. The tool shows the raw request it builds and the server's response.

### Step 1: Retrieve the user list (GET)

Set the method to **GET**, the URL to `https://tryhackme.com/api/users`, and press **Go**.

```http
GET api/users HTTP/1.1
Host: tryhackme.com
User-Agent: Mozilla/5.0 Firefox/87.0
Content-Length: 0
```

The server responds with the user list and the flag.

![GET request to /api/users](images/09-api-get-request.png)

> Flag redacted. Run the request yourself.

### Step 2: Modify a user (POST)

The next task asks for a **POST** to `/api/user/2` that updates Bob's country from `UK` to `US`. A POST carries its data in the **body**, using the `application/x-www-form-urlencoded` format from section 5:

```http
POST api/user/2 HTTP/1.1
Host: tryhackme.com
Content-Type: application/x-www-form-urlencoded

country=US
```

### Step 3: Delete a user (DELETE)

Switch the method to **DELETE** and target `https://tryhackme.com/api/user/1`:

```http
DELETE api/user/1 HTTP/1.1
Host: tryhackme.com
User-Agent: Mozilla/5.0 Firefox/87.0
Content-Length: 0
```

The server processes the deletion and a flag pop-up appears. With all three answers accepted, the room is complete.

![DELETE request and final flags](images/10-api-delete-final.png)

> All flags redacted.

---

## 9. Key Takeaways

- Every web interaction is a **request/response** pair, and both are plain text you can read and modify.
- The **method** expresses intent: `GET` reads, `POST` writes, `DELETE` removes. APIs that don't verify *who* is allowed to do each are a classic vulnerability.
- **Headers matter for security**: `Server` can leak versions, while `Secure` and `HttpOnly` protect cookies.
- **Status codes** reveal what the server is thinking: 4xx means you got something wrong, 5xx means the server did.
- In the practical, changing a single word (`GET` → `DELETE`) changed a harmless read into a destructive action. That is exactly why access control on APIs is so critical.

---

*Room completed on 21 September 2026 as part of the Cyber Security 101 path.*
