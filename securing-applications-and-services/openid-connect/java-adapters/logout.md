# Logout

You can log out of a web application in multiple ways. For Java EE servlet containers, you can call HttpServletRequest.logout(). For other browser applications, you can redirect the browser to `http://auth-server/auth/realms/{realm-name}/protocol/openid-connect/logout?redirect_uri=encodedRedirectUri`, which logs you out if you have an SSO session with your browser.
