# List of errors finded while build U-Boot for zybo

Build failure: openssl/evp.h missing
Component: U-Boot host tool aisimage
Cause: missing OpenSSL development headers on the host
Resolution: install libssl-dev


Build failure: gnutls/gnutls.h missing
Component: U-Boot host tool mkeficapsule
Cause: missing GnuTLS development headers on the host
Resolution: install libgnutls28-dev


