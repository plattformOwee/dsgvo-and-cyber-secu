## 1 Transport & Perimeter

| Risk                             | Action                                                                                                                                                                                                          |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Man-in-the-middle, downgrade** | • Enforce TLS 1.3 only, redirect all HTTP ⇒ HTTPS, set HSTS (including `includeSubDomains; preload`).  <br>• Pin your backend certificate in the Flutter app with pinned-cert `SecurityContext`.                |
| **Bot abuse / brute-force**      | • Put a WAF or rate-limiter (e.g. NGINX `limit_req`, Cloudflare, AWS WAF) in front of `/login_signup/*`.  <br>• 5 failed password attempts → exponential delay; 10 failed verification codes → temporary block. |
| **DDoS & path fuzzing**          | • Disable directory listings, return identical error pages for 4xx/5xx.  <br>• Run periodic endpoint discovery (e.g. OWASP ZAP) against staging to detect accidental routes.                                    |
