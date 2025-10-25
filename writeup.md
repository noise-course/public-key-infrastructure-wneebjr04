I first set up a local HTTP server through wsl, then i captured unencrypted traffic to show why HTTP is insecure and then i  created a self-signed certificate to upgrade the server to HTTPS. I compared the two packet captures to demonstrate how encryption protects the data in transit.

1. Hosting a Local Web Server

I started by creating a super basic webpage that just displayed the message “Hello, HTTP!” using Python’s built-in http.server module. Inside my project folder in WSL, I made a file called index.html and then ran the command:

python3 -m http.server 8080

]
This launched a local web server on port 8080. When I opened http://localhost:8080 in my browser, I saw my webpage load successfully. This confirmed the server was working correctly over HTTP

2. Identifying Why HTTP Is Not Secure

Next, I used tcpdump to capture packets while my browser accessed the site. I ran:

sudo tcpdump -i any -w http_traffic.pcapng 'tcp port 8080'

Then I refreshed the webpage a few times before stopping the capture. When I opened the file in Wireshark and filtered for HTTP traffic, I could see the entire communication in plain text — the GET request, the Host header, and even the HTML content of the page itself (“Hello, HTTP!”).

This makes it clear why HTTP is insecure, because everything sent between the browser and the server can be seen by anyone who intercepts the packets. There’s no encryption, no authentication, and no protection against tampering.

3. Creating a Self-Signed Certificate and Upgrading to HTTPS

After demonstrating the insecurity of HTTP, I created a self-signed SSL certificate using OpenSSL. I ran:

openssl req -x509 -newkey rsa:2048 -days 365 -nodes \
  -keyout certs/key.pem -out certs/cert.pem \
  -subj "/CN=localhost" \
  -addext "subjectAltName=DNS:localhost"


This generated two files — a private key (key.pem) and a certificate (cert.pem) — that I could use to enable HTTPS on my local server.

I then launched a secure server on port 8443 using:

openssl s_server -accept 8443 -cert certs/cert.pem -key certs/key.pem -WWW

When I visited https://localhost:8443, my browser gave a warning because the certificate wasn’t signed by a trusted authority. After clicking “Advanced” and continuing anyway, the page loaded successfully over HTTPS.

3(a). Why I Can’t Get a Real Certificate for My Local Server

The reason I had to create a self-signed certificate is that public Certificate Authorities (CAs) only issue certificates for publicly verifiable domain names.

To obtain a valid SSL certificate, the CA needs to confirm that you control the domain name (for example, through DNS or email verification). A local server like localhost isn’t a real, publicly registered domain — it exists only on my own computer — so there’s no way for a CA to verify ownership.

Because of that, local testing and development environments use self-signed certificates. They still encrypt the traffic and provide security, but browsers don’t trust them automatically because there’s no CA to vouch for their authenticity. That’s why browsers show a warning message when connecting to a self-signed site.
