# KerberosImplementation

> Each interaction will give two messages back, one you can decrypt and one you cant.
     The appilcation server (server.java) never communicates with the KDC directly
     The KDC stores all the secret keys for user machines and services.
      *Key Distribution Centre*
     Client -> Authentication Server 
          (1) Initates an introduction with a plain-text message for a ticket-granting-ticket.
              the message contains: your name/id, the name/id of requested service server(tgt), your network adress, 
              requesed lifetime for validity of the tgt.
          (2) The Authentication server will then check if you are in the key distribution center db. 
              this check is only to see if you exist, the credentials are not checked if they are correct or not.
          (3) If the user is found, it will randomly generate a key called a session key, for use between you 
              and the ticket granting server.
          (4) The Authentication Server(AS) will respond with two messgaes. One Message is the Ticket granting ticket 
              that contains:
              your name/ID, the Ticket Granting Server name/ID, timestamp, your network address, lifetime of ticket granting ticket.
              this is encrypted with the Ticket Granting Server secret key.
          (5) The other message contains the Ticket granting server name/id, timestamp, lifetime, and the ticket granting 
              server secret key.
              this is encrytped with your client secret key. The TGS session key is shared key between you and TGS.
     N.B client secret key is determined by your password appending a salt and hasing the whole thing. 
         You use this to decrypt the tgs session key, so if your password is incorrect it will not decrypt. 
          This is how its validated.
     Client -> Ticket Granting Server
           (1) You first prepare the Authenticator, encrypted with the tgs session key containg your name/id, and timestamp.
           (2) You send an unecrypted message that contains the requested http service name/id you want access to, 
               and the lifetime of the ticket for the http service.
               You then send this and the authenticator preapred above the the Ticket granting server.
           (3) The ticket granting server will then check the KDC to see if the HTTP service exists.
               if it does exist the TGS decrypts the TGT with its Secret key. the now decrypted TGT contains the TGS session key, the tgs can decrypt the authenticator you prepared and sent above.
           (4) The TGS will then do the follow : copare your client id from the Authenticator to that of th TGT
               compare your timestamp from the authenticator to that of the tgt.
               check to see if the tgt is expired or not
               check that the authenticator is not already in the tg's chache (avoid replay attacks)
               if the network address in the og request is not null.
           (5)  The ticket granting server then randomly generates the HTTP service session key and prepares the HTTP service ticket for the client that contains
                your name/id, http service name/id, your network addrss, timestamp, lifetime of the validity of the ticket, and http service session key and encrypts it with the http service secret key.
           (6) The Ticket granting server then sends you two messages like the authentication server.
               One is encrypted HTTP service ticket, the other message contains HTTP service name/id, timestamp, lifetime validity of the ticket and HTTP service session key. This is encrypted with the TGS session key.
     N.B your machine decrypts the latter message with the TGS session key that is chached earlier to obtain the http service session key.
          Your machine can not decyrpt the HTTP service ticket since its encrypted with the HTTP service secret key.
     Client -> HTTP Service
           (1) To now access the HTTP service your machine prepares another Authenticator message that contains
              your name/id, timestamp and is encrypted with the HTTP service session key. 
              Your machine then sends the authentication and the still-encrytped HTTP service ticket recied from the TGS.
           (2) The HTTP service then decrypts the ticket with its secret key to obtain the HTTP service session key. It then uses the Session key to decrpt the Authenticator message you sent.
           (3) Similar to the TGS, the HTTP service will then do the following
               compares your client ID from the Authenticator to that of the ticket,
               compares the timestamp from the Authenticator to that of the ticket.
               checks to see if the ticket is expired
               checks that the authenticator is not already in the HTTP server's chace.
               if the network address in the original request is not null.
           (4) The http service then sends an authenticator messgage contianings its id and timestamp in order to confirm its indenity to you and is encrypted with the HTTP service session key.
           (5) Your machine reads the authenticator message by decrypting with the cached http service session key, and knows that is has to recieve a message with the HTTP service's id and timestamp.
     ***** You are now authenticated to use the HTTP service. Future requests will use the cahced HTTP service ticket, so long as it has not expired as defined.
