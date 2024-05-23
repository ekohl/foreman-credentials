# Foreman Certificate

The service is intended to work with [systemd credentials](https://systemd.io/CREDENTIALS/).
Its design is largely inspired by [Using systemd credentials to pass secrets from Hashicorp Vault to systemd services](https://medium.com/@umglurf/using-systemd-credentials-to-pass-secrets-from-hashicorp-vault-to-systemd-services-928f0e804518).

## Design

In Foreman, services both have server and client certificates.
For this various files are offered in the PEM format:

* `client-certificate`: The certificate sent to servers to identify itself
* `client-key`: The private key matching `client-certificate`
* `server-certificate`: The certificate accompanied by its chain to the root
* `server-key`: The private key matching `server-certificate`
* `server-client-ca`: When using client authentication, these are the certificate authorities that are trusted to sign client certificates

## Testing

To locally run the credential server:
```
systemd-socket-activate --listen $PWD/socket --setenv STATE_DIRECTORY=$PWD/credentials --setenv DEBUG=1 $PWD/foreman-credentiald
```

Then the echo server can be started:

```
systemd-run --user --collect --unit echo --wait --pty -p LoadCredential=server-certificate:$PWD/socket -p LoadCredential=server-key:$PWD/socket -p LoadCredential=server-client-ca:$PWD/socket ./echo
```

Now you can verify the correct certificate was loaded:
```
openssl s_client -connect localhost:8443
```
