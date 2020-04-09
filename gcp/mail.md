## Testing email sending

Install cli mail client
```
kubectl run test \
  -i \
  --tty \
  --image=alpine \
  --restart=Never \
  --rm \
  -- sh -c 'apk --update add heirloom-mailx bash; exec bash'
```

Sending email:
```
MAIL_FROM=from.mail@domain.com
MAIL_TO=to.mail@domain.com
MAIL_RELAY=smtp-relay.gmail.com:587
echo "Body message" | mailx -S smtp=$MAIL_RELAY \
  -s "Test subject 1" \
  -v \
  -r $MAIL_FROM \
  $MAIL_TO
```

Output
```
Resolving host smtp-relay.gmail.com . . . done.
Connecting to 64.233.167.28:587 . . . connected.
220 smtp-relay.gmail.com ESMTP h13sm106940wrr.3 - gsmtp
>>> HELO test
250 smtp-relay.gmail.com at your service
>>> MAIL FROM:<from.mail@domain.com>
250 2.1.0 OK h13sm106940wrr.3 - gsmtp
>>> RCPT TO:<to.mail@domain.com>
250 2.1.5 OK h13sm106940wrr.3 - gsmtp
>>> DATA
354  Go ahead h13sm106940wrr.3 - gsmtp
>>> .
250 2.0.0 OK  1584701732 h13sm106940wrr.3 - gsmtp
>>> QUIT
221 2.0.0 closing connection h13sm106940wrr.3 - gsmtp
```
