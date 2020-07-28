# Cross Site Scripting

Sending unvalidated data to a web browser may result in certain browsers executing malicious code.

Cross-site scripting (XSS) vulnerabilities occur when:
1. Data enters a web application through an untrusted source. In the case of reflected XSS, the untrusted source is typically a web request, while in the case of persisted (also known as stored) XSS it is typically a database or other back-end data store.
2. The data is included in dynamic content that is sent to a web user without being validated.

**Servlet:**
Vulnerable:
```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) {

  String comment = req.getParameter("comment");
  resp.getWriter().print(comment);
   ..
}
```

Patch:
```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) {

  String comment = req.getParameter("comment");
  String encoded = ESAPI.encoder().encodeForHTML(comment);
   ..
}
```

**JSP:**
Vulnerable:
```java
<html>
<body>
Welcome <%= request.getParameter("username")%>
</body>
</html>
```

Patch:
- Can be fixed by regex also
- Encode the string when output renders
```java
<html>
<body>
Welcome <%= ESAPI.encoder().encodeForHTML(request.getParameter("username")) %>
</body>
</html>
```
