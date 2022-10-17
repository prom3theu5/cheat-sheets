# SSL Certificate Verification

Two ways to Check an SSL Certificate

```bash
openssl s_client -connect example.com:443 </dev/null 2>/dev/null | openssl x509 -inform pem -text
```

```bash
curl -vI https://example.com
```