## ❓ Problématique Centrale

Dans un écosystème d'entreprise où les collaborateurs doivent interagir quotidiennement avec une multitude d'applications hétérogènes (*outils collaboratifs, serveurs d'intégration continue, plateformes de supervision*), la gestion cloisonnée des identités pose un triple défi de sécurité et d'ergonomie :
1. **Multiplication et faiblesse des secrets :** L'obligation de mémoriser plusieurs paires d'identifiants pousse les utilisateurs à adopter des mots de passe faibles ou identiques, augmentant drastiquement la surface d'attaque face au phishing et au credential stuffing.
2. **Complexité de la gouvernance (Cycle de vie) :** Pour les administrateurs, l'absence de gestion centralisée complique l'enrôlement des nouveaux arrivants et, de manière plus critique, la révocation immédiate des accès lorsqu'un collaborateur quitte l'organisation.
3. **Incompatibilité des protocoles :** Les infrastructures modernes doivent faire cohabiter des outils historiques basés sur des architectures d'annuaires traditionnelles et des applications Cloud-native.

**La problématique de ce projet est donc la suivante :** *Comment concevoir et déployer une infrastructure fédérée de gestion des identités et des accès (IAM) capable de centraliser l'authentification de manière transparente pour l'utilisateur, tout en garantissant l'interopérabilité sécurisée entre des protocoles de confiance distincts (SAML 2.0 et OIDC) et un référentiel d'identité unique (LDAP) ?*

---

## 🔬 Choix Technologiques & Justifications

### 1. Pourquoi Keycloak plutôt qu'Authentik ?
Bien qu'Authentik soit une solution moderne et performante, **Keycloak (développé par Red Hat)** a été retenu comme Fournisseur d'Identité (IdP) central pour notre infrastructure d'entreprise en raison de critères stricts de maturité et d'architecture :

* **Conformité et Certification Standard :** Keycloak est officiellement certifié par la *OpenID Foundation*. Sa gestion des spécifications strictes de SAML 2.0 et d'OIDC (notamment les extensions comme FAPI ou le support natif de PKCE) offre une robustesse éprouvée face aux audits de sécurité.
* **Fédération LDAP/AD Avancée :** Les mécanismes de synchronisation de Keycloak (User Federation) avec les annuaires OpenLDAP ou Active Directory sont extrêmement granulaires (importation sélective, synchronisation bidirectionnelle ou à la volée, mapping dynamique des rôles via les groupes LDAP).
* **Maturité et Écosystème Éprouvé :** Présent sur le marché depuis plus de 10 ans, Keycloak bénéficie d'une communauté massive et de retours d'expérience industriels critiques, indispensables pour garantir la pérennité d'un noyau de sécurité d'entreprise (IAM) par rapport à des solutions plus récentes.

### 2. Justification des Applications et Protocoles Choisis

L'infrastructure valide la coexistence de deux mondes protocolaires majeurs pour répondre aux besoins spécifiques de chaque outil métier :

#### A. Le Choix de SAML 2.0 (Sécurité par Échange d'Assertions XML)
* **Nextcloud & Jenkins :** Ces plateformes représentent les outils de production et de CI/CD traditionnels de l'entreprise. SAML 2.0 a été privilégié ici car il s'agit d'un standard industriel historique conçu spécifiquement pour le SSO de serveurs d'entreprise (*Enterprise-grade*). 
* **Avantage de Sécurité :** L'authentification repose sur des échanges d'assertions XML signées numériquement et optionnellement chiffrées (via des paires de clés privées/publiques), ce qui évite d'exposer ou de faire transiter les jetons d'accès directement sur le navigateur de l'utilisateur.

#### B. Le Choix d'OpenID Connect - OIDC (Sécurité par Jetons Éphémères JWT)
* **GitLab & Grafana :** Ces solutions incarnent la stack Cloud-native et DevSecOps moderne. OIDC, qui est une surcouche d'identité bâtie sur **OAuth 2.0**, est nativement conçu pour les applications web légères, les Single Page Applications (SPA) et les architectures microservices.
* **Avantage de Sécurité :** OIDC permet l'utilisation du flux d'autorisation avec **PKCE (Proof Key for Code Exchange)**. Ce mécanisme dynamique empêche l'interception du code d'autorisation par un attaquant sur le réseau ou le navigateur. De plus, l'utilisation de jetons structurés **JWT (JSON Web Tokens)** signés via l'algorithme `RS256` permet aux applications de valider localement et instantanément l'identité de l'utilisateur sans surcharger l'IdP de requêtes de vérification continuelles.
