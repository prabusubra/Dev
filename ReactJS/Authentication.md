# React Authentication:-

- OAuth 2.0 with JWT (JSON Web Token) ensures secure, token-based authentication between React.js and Spring Boot microservices.

  ## Architecture Overview
- React.js Frontend → Authenticates via OAuth2, retrieves JWT.
- API Gateway (Optional, Spring Cloud Gateway) → Verifies JWT before routing requests.
- Spring Boot Microservices → Each service validates JWT, enforces role-based access control (RBAC).
- Identity Provider (IdP) → Manages authentication, issues tokens.

 ## Secure Authentication Flow
  ### User Login
  - React.js app redirects the user to the IdP login page.
  - IdP authenticates the user and issues a JWT access token + optional refresh token.
  ### Token Storage (Frontend)
  - ***Best Practice:*** Store access token only in memory (Redux store, React Context).
  - ***Refresh token handling:*** Use HTTP-only, Secure SameSite cookies.
