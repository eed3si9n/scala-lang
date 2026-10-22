---
layout: blog-detail
post-type: blog
title: "Advisory for Scala users regarding eMudhra/emSign intermediate CA incident"
---

[The private keys of eight eMudhra intermediate CA certificates have been compromised](https://emudhra.we-have-a-problem.com/). This broke the core trust component of HTTPS and mTLS, potentially allowing monster-in-the-middle (MITM) attacks impersonating as Maven Central etc.
Web browsers, operating systems, and various programming language communities are rushing to patch this vulerability.

## Affected systems

According to [OpenJDK Vulnerability Advisory: 2026/10/21](https://openjdk.org/groups/vulnerability/advisories/2026-10-21):

> The affected versions are (OpenJDK) 27.0.1, 25.0.5, 21.0.13, 17.0.21, 11.0.33, 8u512, and earlier.<br><br>
>
> If you use an OpenJDK Java runtime provided by your operating system, then it most likely uses the operating system’s certificates. This is true of the OpenJDK packages in Arch Linux, Debian, Devuan, Fedora, Gentoo, Oracle Linux, Red Hat Enterprise Linux, and Ubuntu; in these cases, [updating the operating system's certificates](https://emudhra.we-have-a-problem.com/#developers) is sufficient. This is not true of the OpenJDK packages in FreeBSD and Homebrew, which must be updated directly.

To check if your JDK is vulnerable, checkout [eed3si9n/emudhra-emsign-check](https://github.com/eed3si9n/emudhra-emsign-check) on your local machine, or call `CheckCompromisedCa.test` from your CI.

```bash
$ sbt run
[info] welcome to sbt 1.13.0 (Azul Systems, Inc. Java 1.8.0_402)
[info] running main
Java 1.8.0_402 (/Users/xxx/.sdkman/candidates/java/zulu8.76.0.17-ca-jdk8.0.402-macosx_aarch64/zulu-8.jdk/Contents/Home/jre)
VULNERABLE (HTTP 200) https://dv-g3b-emudhra.we-have-a-problem.com/
VULNERABLE (HTTP 200) https://dv-g3c-emudhra.we-have-a-problem.com/
VULNERABLE (HTTP 200) https://ov-g3b-emudhra.we-have-a-problem.com/
VULNERABLE (HTTP 200) https://ov-g3c-emudhra.we-have-a-problem.com/
VULNERABLE (HTTP 200) https://ev-g3b-emudhra.we-have-a-problem.com/
VULNERABLE (HTTP 200) https://ev-g3c-emudhra.we-have-a-problem.com/
VULNERABLE (HTTP 200) https://dv-g3b-aaa-emudhra.we-have-a-problem.com/
VULNERABLE (HTTP 200) https://dv-g3c-aaa-emudhra.we-have-a-problem.com/
VULNERABLE (HTTP 200) https://ov-g3b-aaa-emudhra.we-have-a-problem.com/
VULNERABLE (HTTP 200) https://ov-g3c-aaa-emudhra.we-have-a-problem.com/
VULNERABLE (HTTP 200) https://ev-g3b-aaa-emudhra.we-have-a-problem.com/
VULNERABLE (HTTP 200) https://ev-g3c-aaa-emudhra.we-have-a-problem.com/
[error] java.lang.RuntimeException: compromised CAs are in use, or we encountered a network error
[error]   at scala.sys.package$.error(package.scala:27)
[error]   at CheckCompromisedCa$.test(CheckCompromisedCa.scala:36)
[error]   at CheckCompromisedCa$.main(CheckCompromisedCa.scala:32)
[error]   at main.main(CheckCompromisedCa.scala:22)
[error]   at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
[error]   at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
[error]   at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
[error]   at java.lang.reflect.Method.invoke(Method.java:498)
[error] stack trace is suppressed; run last Compile / run for the full output
[error] (Compile / run) compromised CAs are in use, or we encountered a network error
```

## Blocking the compromised certificates manually

Until you can update to the fixed JDK distributions, Oracle, Azul, and others recommend that we manually block the compromised certificates. This can be done by appending 16 SHA-256 fingerprints to `$JAVA_HOME/lib/security/blocked.certs` and `$JAVA_HOME/jre/lib/security/blocked.certs` for JDK 11+; and `$JAVA_HOME/lib/security/blacklisted.certs` and `$JAVA_HOME/jre/lib/security/blacklisted.certs` for JDK 8.

```bash
mkdir -p $JAVA_HOME/lib/security/
cat >> "$JAVA_HOME/lib/security/blocked.certs" <<'EOF'
42C4ACCD12C4F59597925E1A08A7A90102608543809308D50EC427E118BA1493
45A883F349AEDFD5F3A649B5498FFB121712BA8B448F3F2A91DBC5A766FCDDBE
98B896625C0C230ADEEC3EB2CF6AAD5001ECF9156A68644E82374CE5833BC25B
5F1DED4DD590AF74631290CF073B122A2AAC6A1FA479778A0BD43579BD832699
E7568FE6E12331F4FD7DE61DA4C7559D0F8EB192A218D4741A3BA75184333838
579480AEA166290BD3C643589E795FC0D0E54E86505D180F1DFF99EFD3199512
0164D7CEAE5BB909C272AF151FF44B1177AE9B47D9424B94F1D9558C30934D19
62B7F9868263F72F6911FC78CEAF72C1979158B7368BF648683BB2AA39B02E72
CB6E15165AC802E9B3F993F16C25D41F4E81C21EBDE71AE1B012AC00BF9C7C3F
7485B51C0ED192878FA1D2F948FB529AE1842CC5086C4B67F7BE5B2039DE9489
432EC25F0D28DE9006D97BF752F17BBB2CD41F669933FCB072D9A1148D0261F3
FD9D5C2DBBD4995AB69B8D50C439D9527B00529199C2FD01B2174EB58F07021F
9E39DE2881273ACC225A149C72BA3E0F51424963669AB633EE1DC7DF5DB067B4
A496E23AC0CAB8155FC9B117B97EAE368EFCBB9041AEC8804D5A308D2D01C2D8
4B2460A466F95FD40113BCDCCA2CEB29B3B093921DE45474DA7DF48874C630B1
84112972B8E59D14560AE7E7119654474AAD119990CF3722BFC1E3B91615E1FD
EOF
mkdir -p $JAVA_HOME/jre/lib/security/
cat >> "$JAVA_HOME/jre/lib/security/blocked.certs" <<'EOF'
42C4ACCD12C4F59597925E1A08A7A90102608543809308D50EC427E118BA1493
45A883F349AEDFD5F3A649B5498FFB121712BA8B448F3F2A91DBC5A766FCDDBE
98B896625C0C230ADEEC3EB2CF6AAD5001ECF9156A68644E82374CE5833BC25B
5F1DED4DD590AF74631290CF073B122A2AAC6A1FA479778A0BD43579BD832699
E7568FE6E12331F4FD7DE61DA4C7559D0F8EB192A218D4741A3BA75184333838
579480AEA166290BD3C643589E795FC0D0E54E86505D180F1DFF99EFD3199512
0164D7CEAE5BB909C272AF151FF44B1177AE9B47D9424B94F1D9558C30934D19
62B7F9868263F72F6911FC78CEAF72C1979158B7368BF648683BB2AA39B02E72
CB6E15165AC802E9B3F993F16C25D41F4E81C21EBDE71AE1B012AC00BF9C7C3F
7485B51C0ED192878FA1D2F948FB529AE1842CC5086C4B67F7BE5B2039DE9489
432EC25F0D28DE9006D97BF752F17BBB2CD41F669933FCB072D9A1148D0261F3
FD9D5C2DBBD4995AB69B8D50C439D9527B00529199C2FD01B2174EB58F07021F
9E39DE2881273ACC225A149C72BA3E0F51424963669AB633EE1DC7DF5DB067B4
A496E23AC0CAB8155FC9B117B97EAE368EFCBB9041AEC8804D5A308D2D01C2D8
4B2460A466F95FD40113BCDCCA2CEB29B3B093921DE45474DA7DF48874C630B1
84112972B8E59D14560AE7E7119654474AAD119990CF3722BFC1E3B91615E1FD
EOF
```

Run [eed3si9n/emudhra-emsign-check](https://github.com/eed3si9n/emudhra-emsign-check) to verify that the certificates are blocked. If it's successful, you should see an output like this:

```bash
$ sbt run
[info] running main
Java 1.8.0_402 (/Users/xxx/.sdkman/candidates/java/zulu8.76.0.17-ca-jdk8.0.402-macosx_aarch64/zulu-8.jdk/Contents/Home/jre)
OK https://dv-g3b-emudhra.we-have-a-problem.com/
  SSLHandshakeException: PKIX path validation failed: java.security.cert.CertPathValidatorException: Untrusted certificate: CN=EM DV TLS CA - G3B, O=eMudhra BV, C=NL
OK https://dv-g3c-emudhra.we-have-a-problem.com/
  SSLHandshakeException: PKIX path validation failed: java.security.cert.CertPathValidatorException: Untrusted certificate: CN=EM DV TLS CA - G3C, O=eMudhra Inc, C=US
....
```

## Updating to the fixed JDKs

We advise to update to the fixed JDKs when they become available.

On 2026-10-21, Oracle has released patch versions. According to [eMudhra Intermediate Certificates are no Longer Trusted in Oracle JDK](https://ops.java/emudhra):

> Following direct reports and public disclosures such, Oracle has provided a critical patch update which distrusts the set of intermediate CA certificates impacted related to eMudhra. This update is available from [Java Downloads](https://www.oracle.com/java/technologies/downloads/).

Also on 2026-10-21, OpenJDK project has also issued a similar advisory [OpenJDK Vulnerability Advisory: 2026/10/21](https://openjdk.org/groups/vulnerability/advisories/2026-10-21), and distributors like Eclipse and Azul are packaging the installers.

|           JDK |  Oracle | Temurin |         Zulu |
|--------------:|--------:|--------:|-------------:|
|         JDK 8 | 8u511       | n/a     | 8u512-b08 |
|        JDK 11 | 11.0.33.0.2 | n/a     | 11.0.33+9 |
|        JDK 17 | 17.0.21.0.2 | n/a     | 17.0.21+9  |
|        JDK 21 | 21.0.13.0.2 | n/a     | 21.0.13+9 |
|        JDK 25 | 25.0.5.0.2 | n/a     | 25.0.5+8 |
| Release notes |  |         | [Note](https://docs.azul.com/core/release/october-2026-respin/release-notes.html#) |
