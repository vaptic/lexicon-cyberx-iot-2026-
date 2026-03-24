# Security differences between MQTT and HTTPS for IoT data transmission
MQTT and HTTPS can both be made cryptographically strong with TLS, but they differ in how security is typically deployed, managed, and enforced in IoT systems.

## Transport encryption and defaults
* HTTPS is effectively “HTTP over TLS” and is almost always deployed with TLS by default, so encryption, integrity, and server authentication are expected out of the box.
* MQTT is just a lightweight TCP protocol; without configuration it runs in cleartext, and you must explicitly enable TLS (often called MQTTS on port 8883) on the broker and clients to get equivalent confidentiality and integrity.

## Authentication models
* HTTPS inherits the web security ecosystem: server authentication via X.509 certificates, optional client certificates, plus higher-layer mechanisms like cookies, tokens, API keys, and OAuth commonly used for device and user identity.
* MQTT supports TLS-based server and client authentication (including mutual TLS with client certificates), but also defines its own username/password fields and often topic-based ACLs at the broker; these are MQTT-specific and not as standardized as web auth patterns.

## Authorization granularity
* In HTTPS, authorization is usually implemented at the application layer (REST APIs, resource-level permissions, access tokens), with fine-grained control per URL or method, backed by standard frameworks.
* MQTT commonly uses broker-side ACLs to restrict which topics a client may publish or subscribe to, which gives very fine-grained, message-stream-oriented control but requires careful broker configuration and is specific to the MQTT ecosystem.

## Security tooling and maturity
* HTTPS benefits from decades of web security tooling: mature PKI processes, certificate management, WAFs, standard libraries, and well-known best practices, so misconfigurations are easier to detect and audit.
* MQTT security can be equally robust at the crypto level (same TLS, same ciphers), but it often requires more specialized broker knowledge, and many insecure deployments exist because people leave MQTT on plain TCP or use weak broker ACLs.

## IoT-specific operational aspects
* For constrained IoT devices, HTTPS typically uses short-lived, per-request connections and heavier HTTP headers, which increases overhead but simplifies stateless, token-based security models.
* MQTT maintains long-lived connections with minimal header overhead, which is efficient for telemetry but means credential compromise can give an attacker continuous access to publish/subscribe on topics unless TLS, certificate-based auth, and tight topic ACLs are enforced.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# MQTT wildcard subscription attack
The “MQTT wildcard subscription attack” is an abuse of MQTT’s topic wildcards (especially #) to eavesdrop on or harvest large amounts of data from an MQTT broker that is poorly secured.

## What it is
* MQTT allows subscribers to use wildcards (+ for single level, # for multi-level) so one subscription can receive messages from many topics at once. If a client subscribes to #, it will receive every message on the broker.
* In the attack, an adversary connects to the broker (often because there is no authentication or very weak access control) and subscribes to a wildcard topic like # or someprefix/# to passively collect all traffic or all traffic under a certain namespace.

## How it is exploited in practice
* On exposed or misconfigured brokers (no TLS, no authentication, no proper ACLs), anyone on the network or Internet can connect as a client and issue a wildcard subscription. This immediately gives them access to all messages matching the wildcard, including sensitive telemetry and control topics.
* Because wildcards work with retained messages, a single wildcard subscription can also fetch the last retained message for many topics at once, letting an attacker “dump” historical state the moment they connect.
* Research and vulnerability reports describe scenarios where attackers manipulate subscription control packets to include multiple wildcard topics, allowing them to receive messages from essentially all topics and gain unauthorized visibility into the MQTT network.

In short, the attack is not a flaw in the wildcard feature itself but a consequence of brokers allowing overly broad wildcard subscriptions without strong authentication and topic-level authorization, which an attacker can exploit to perform large-scale data sniffing and reconnaissance.





