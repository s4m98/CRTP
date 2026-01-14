# Kerberos Authencation:

Kerberos is a ticket-based authentication protocol that enables secure communication in untrusted environments by first establishing two-party trust through a mutual third party. Kerberos requires three parties:

**Client:** The user or system requesting access to a resource.\
**Server:** The destination resource the client wants to access.\
**Key Distribution Center (KDC):** A trusted third party responsible for authenticating users and issuing tickets.

## The ticketing process

### The Kerberos authentication flow works like this:

**Login & Initial Request**
  User enters username and password.
  Client sends a request to the Authentication Server (AS) for a Ticket Granting Ticket (TGT).

**Authentication Server Response**
  AS checks credentials against its database.
  If valid, it sends back an encrypted TGT (using the user’s secret key derived from their password).

**Requesting Service Access**
  Client presents the TGT to the Ticket Granting Server (TGS) when it wants to access a service (e.g., mssql,file server, email).

**Service Ticket Issuance**
  TGS validates the TGT and issues a Service Ticket, encrypted with (krbtgt service account password hash) the service’s secret key.

**Accessing the Service**
  Client presents the Service Ticket to the target server.
  The server decrypts it using its secret key, verifies authenticity, and grants access.

**Mutual Authentication**
  The server

<figure><img src="../assets/kerberos-auth.png" alt=""><figcaption></figcaption></figure>




