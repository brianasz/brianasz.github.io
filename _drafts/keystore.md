---
layout: post
---

Error:
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
Caused by: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
Caused by: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target


Check which certificates are in a java keystore
keytool -list -v -keystore keystore
