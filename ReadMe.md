## Folder Structure

```
root
│
├── client (React frontend)
│ ├── package.json
│ └── node_modules
│
├── server (Node.js backend)
│ ├── package.json
│ └── node_modules
│
└── package.json (root)
```

## How to Run This Application (At the ROOT of the FOLDER)

Follow these steps:

1. Install dependencies for the client and server:

```
   npm run install:client
   npm run install:server
```

2. Set up the database:

```
   npm run database
```

3. Seed the database:

```
   npm run seed
```

4. Start the application:

```
   npm run start
```

Once running, both the client and server will be available:

Visit http://localhost:3000 to see the React app.\
Visit http://localhost:3001/users to view all users.\
Visit http://localhost:3001/movies to view all movies.

## React Application

The React app (accessible at http://localhost:3000), you'll see a simple interface with seven movie titles. You can search for these movies by typing their titles into the input box. For example, typing "The Matrix" will display "The Matrix" and its release date.

## SQL Injection Demonstration

A sample SQL injection code is provided for demonstration:

```
' OR 1=1; SELECT * FROM users --
```

Paste this code into the input box to see all users from the database. This exposes a significant security vulnerability.

## Task

Your objectives are:

Identify and Fix Vulnerabilities:

Backend: Address the SQL injection vulnerabilities in the server code.
Frontend: Implement measures to prevent the injection of malicious input.
Research and Presentation:

What is SQL Injection?
Who is Affected?
What Changes Were Made on the Backend?
What Changes Were Made on the Client Side?
Red Canary requires you to document your research and analytical process to evaluate your problem-solving skills.

Please share the GitHub repository link and a PowerPoint presentation with your findings.

You are expected to demonstrate your code and present your findings on SQL injection, including how you resolved the issues.

Please refer to the Detection Engineer documentation for more information.

If you have any questions, please slack me or email me at pak@pursuit.org

## Securing Against SQL Injection

### Problem

#### What is a SQL Injection?

A SQL Injection is a form of code injection that uses SQL queries to manipulate a SQL database. It achieves this by exploiting how a Database interpolates a string into a SQL query.

```javascript
// Problem
const query = `
                  SELECT *
                  FROM movies
                  WHERE title ILIKE '${title}';
                `;

const result = await db.any(query);
```

This becomes a problem because the variable `title` get's inserted directly into the query using an template literal. Since the database interprets the string with the inserted value all at once, this causes a SQL injection.

With a database vulnerable to SQL injection, it allows an attacker to query for any and all information stored on the database with no restrictions. This includes sensitive and confidential data about other users like in the example below.

```
' OR 1=1; SELECT * FROM users --
```

### Research

My main source of research was the [OWASP Cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html) that provided in depth details regarding the main culprit of SQL injection. That being the afore mentioned template literal.

#### Sources

- [Template Literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)
- [OWASP CheatSheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [Cloudflare](https://www.cloudflare.com/en-gb/learning/security/how-to-prevent-xss-attacks/)

### Solutions

When creating a solution for this particular problem, my research has led me to using the best practices for tackling SQL injection. This revolves around using parameterization instead of the template literal.

```javascript
// Before
const query = `
                  SELECT *
                  FROM movies
                  WHERE title ILIKE '${title}';
                `;

const result = await db.any(query);

// After
const query = `
                  SELECT *
                  FROM movies
                  WHERE title ILIKE $1;
                `;
const result = await db.any(query, sanitisedTitle);
```

The reason this solution works is because parameterization takes advantage of dynamic insertion. This means that the database reads the query first and accepts `$1` as a placeholder for a value to be inserted in dynamically after the full query is read. This solution is best practice because it stops the attack at the point of entry to the database. This way, even if an attacker were to circumvent the frontend application and use fetch requests to the server, the database will still return an error ultimately stopping the attack.

However this doesn't mean this is the only solution.

## Solution 2: Sanitization

Sanitization is the practice of cleaning inputs and outputs in a web application to ensure malicious code cannot be passed onto our database through multiple points of entry. This means validating and removing characters that could be used to carry out not only SQL injection but other attacks such as [**Cross-Site Scripting**](https://owasp.org/www-community/attacks/xss/).

In this solution there are two points of entry for an attacker to pass in SQL injection:

- Frontend (client): where the user interacts with the site
- Backend (server): the API that sends data

Starting at the frontend there's only one input form that the user interacts with to receive movie titles from our backend server.

![Input Form](https://private-user-images.githubusercontent.com/122545794/376723790-310979a1-ffe0-4bea-8e76-e658a186e256.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3Mjk4NjkxMjcsIm5iZiI6MTcyOTg2ODgyNywicGF0aCI6Ii8xMjI1NDU3OTQvMzc2NzIzNzkwLTMxMDk3OWExLWZmZTAtNGJlYS04ZTc2LWU2NThhMTg2ZTI1Ni5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjQxMDI1JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI0MTAyNVQxNTA3MDdaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT1jNDExMTIwMjJiZWE4ZjQ5OWRlNGY0MzQ0ZGNkMjJmYzg0N2Q5MThhZjUzZTViMmUwYTliYzZiNjRiZGUyYTM2JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.HvSeWA_NSAahZqAhgaQwwj9WAAHU2R4LYTtDJOi7KsM)

So I focused on applying form validation so that an attacker cannot pass in too many special characters or punctuation in the form. I'm able to this by taking advantage of an attribute on the input tag called `pattern`.

```jsx
<form onSubmit={handleSearch}>
  <label>
    Movie Title:
    <input
      type="text"
      pattern="[`',A-Za-z0-9\s]+"
      value={title}
      onChange={(e) => setTitle(e.target.value)}
    />
  </label>
  <button type="submit">Search</button>
</form>
```

![Form Formatting](https://cms-assets.tutsplus.com/cdn-cgi/image/width=600/uploads/users/30/posts/25145/image/username.png)

This pattern attribute only accepts alphanumeric characters, whitespace and three punctuation (`',). This works to stop attackers passing in malicious code considering all coding languages need punctuation to establish syntax. I followed this up with additional sanitization on not only the request being sent from the frontend but the route itself on the backend.

##### Frontend Sanitization

In our frontend sanitization we remove any punctuation that could be used to for malicious purposes and ensure that their replaced with empty space.

```js
const sanitisedTitle = title.replace(/[./#!$%^&*;:{}=\-_~()]/g, "");
    try {
      const response = await axios.get(
        `http://localhost:3001/search?title=${sanitisedTitle}`
      );
    }
```

##### Backend Sanitization

Although we've been able to sanitize the frontend we also need to sanitize the backend. The backend is more detailed because it also removes characters such as `<` and `>` which can be used for the earlier mentioned Cross-Site Scripting (XSS).

```js
const title = req.query["title"].replace(/[.,/#!$%^&*;:{}=\-_~()]/g, "");
const sanitisedTitle = title.replace(/</g, "").replace(/>/g, "");
```

This removes inputs such as `<script>console.log("hello")</script>` to simply `scriptconsolelog"hello"script`. As intended this removes any valid syntax to make this script executable in a user's DOM. Even though it achieves the goal of sanitizing inputs so nothing malicious can be passed into the database. On the other hand this may cause complications in the future when dealing with complex movie title names but with the current available data in database the Movies query are still accessible.

Cross-Site Scripting is where an attacker uses an input form to pass in javascript code to a database. This code sits on the database until another user requests that information and the database serves the victim with the malicious script that then activates when rendered into the victims DOM (Document Object Model). This means an attacker can take sensitive information about the user directly from their web session.

![XSS Diagram](https://media.geeksforgeeks.org/wp-content/uploads/20190516152959/Cross-Site-ScriptingXSS.png)

In order to protect against Cross-Site Scripting, the before sanitization techniques remove use of scripts. For additional security the more common approach is to use packages such as [DOMPurify](https://www.npmjs.com/package/dompurify) that sanitizes requests from servers that contain html and javascript.

Here's what it looks like

```javascript
import { sanitize } from "dompurify";
// ... Other Code

const response = await axios.get(
  `http://localhost:3001/search?title=${sanitisedTitle}`
);
let stringifiedRes = sanitize(JSON.stringify(response.data));
stringifiedRes = JSON.parse(stringifiedRes);
setResults(stringifiedRes);
```

This package is essential because React handles setting innerHTML with it's own [dangerouslySetInnerHTML](https://legacy.reactjs.org/docs/dom-elements.html) component, which as they say **"to remind yourself that it’s dangerous"**. Manipulating innerHTML is dangerous in general as it opens the site to Cross-Site Scripting.

## Cross Origin Resource Sharing (CORS)

Now that we've been able to sanitize inputs from the frontend and backend there are additional steps we need to take to secure our web application. This refers to CORS. [CORS (Cross Origin Resource Sharing)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) is a mechanism on request headers that establish where the server can receive requests from.

With the original method that CORS was handled left the server to rely on the CORS package default. The default uses the `*` wildcard meaning that it will take in all requests.

```javascript
const express = require("express");
const cors = require("cors");
const app = express();

app.use(cors());
```

This becomes a problem because for all the effort we put into sanitizing inputs on the client become pointless. An attacker can send requests to the server without the need of going through the client input form.

To fix this issue I looked to implement [CORS options](https://www.npmjs.com/package/cors#configuring-cors) so that my server can only take requests from one domain.

```javascript
const targetUrl =
  process.env.NODE_ENV === "production"
    ? "<INSERT DEPLOYED FRONTEND LINK>"
    : "http://localhost:3000";
const corsOptions = {
  origin: (origin, callback) => {
    if (origin === targetUrl) {
      callback(null, true);
    } else {
      callback(new Error("Not allowed by CORS").message, false);
    }
  },
  methods: ["GET"],
  maxAge: 3600,
  credentials: true,
};
```

In this code I check what environment the server is in in order to set what location to use. For example, if the environment is in production (deployed) the `targetURL` will be set to the deployed client link. e.g www.searchMovies.com. Whereas, in a development environment it will use localhost. The origin property in the corsOption checks that the origin of the request is the same as the `targetURL` in order to allow the request into the server or to deny the request.

### Other Research

Now let's assume that our codebase was back to where it originally was with the SQL injection vulnerability or the Cross site scripting vulnerability, in the case of the SQL injection attack someone could take advantage of the fact that they don't need the client to pass queries to the server.

This lead me to experiment with denial-of-service attacks. These are attacks that specialize in slowing down or shutting down a server with a flood of resource intensive requests. When the server has to process all these resource intensive requests the server may hit its resource threshold and timeout. This means that the server stops taking requests and denies legitimate users from being able to use the service.

One such query I came across in my research was the one below that causes a 5 second delay depending on the first character of the database's name.

```SQL
SELECT CASE WHEN substring(datname,1,1)='1' THEN pg_sleep(5) ELSE pg_sleep(0) END FROM pg_database LIMIT 1;
```

There were a number of other SQL injections I were able to find at this [github repository](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/PostgreSQL%20Injection.md#postgresql-stacked-query) that were specific to PostgreSQL. These queries made me interested to write a script that would cause a timeout such as this:

```javascript
async function sendRequests() {
  const url = "http://localhost3001/search?title=<INSERT SQL INJECTION HERE>";
  const requests = Array.from({ length: 2000 }, () =>
    fetch(url).then((response) => response.json())
  );

  try {
    // Run all requests in parallel and wait for them to complete
    const results = await Promise.all(requests);
    console.log(results);
  } catch (error) {
    console.error("Error with one of the requests:", error);
  }
}

sendRequests();
```

However I didn't have the resources to test this in an deployed server at scale.
