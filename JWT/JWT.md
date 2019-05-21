작성중.. 
# Refresh Token?
Access Token을 통한 인증 방식의 문제를 보완해 준다.
처음에 로그인을 완료할 때 AccessToken과 동시에 Refresh Token 발급해준다. 
Refresh Token은 Access Token이 만료됐을 때 다시 발급해주는 열쇠가 된다.

Access Token은 탈취당하면 정보가 유출되는 건 마찬가지다. 다만 유효기간을 짧게 가져가기 때문에 보다 안전하다는 의미일 뿐..
Refresh Token의 유효기간이 만료 됐다면 사용자는 새로 로그인을 해야한다.
Refresh Token도 탈취될 가능성이 있다.

# 사용 방법
사용자 로그인
-> 서버에서는 회원 DB에서 값을 비교한다.
-> 로그인이 완료되면 Access Token, Refresh Token을 발급하고, 이때 일반적으로 회원 DB에 Refresh Token을 저장해둔다.
-> 사용자는 Refresh Token은 안전한 저장소에 저장하고, Access Token을 헤더에 실어 요청을 보낸다.
-> Access Token 검증하여 이에 맞는 데이터를 보냄.

-> 시간이 지나 Access Token이 만료됐을 경우,
-> 서버는 AccessToken이 만료됨을 확인하고 권한없음을 신호로 보냄.
-> 사용자는 RefreshToken과 AccessToken 함께 보냄.
-> 서버는 받은 Access Token 이 조작되지 않았는지 확인 후, Refresh Token과 사용자 DB에 저장되어 있던 Refresh Token 비교.
Token이 동일하고 유효기간도 지나지 않았다면 새로운 Access Token 발급.

# Refresh Token은 어디에 저장할까?
https://stormpath.com/blog/where-to-store-your-jwts-cookies-vs-html5-web-storage 글을 번역한 글입니다.
https://lazyhoneyant.tistory.com/7 참고

서버는 DB에 저장하면 되고..
클라이언트에 저장할 때에는 두 가지 옵션이 있습니다.(더 있을 수도 있습니다.)
1. HTML5 Web Storage (localStorage or sessionStorage)
2. Cookies

차이가 무엇일까?
## JWT sessionStorage 및 localStorage 보안
Web Storage (localStorage / sessionStorage)는 동일한 도메인의 JavaScript를 통해 액세스 할 수 있습니다.  
즉, 사이트에서 실행중인 모든 JavaScript는 웹 저장소에 액세스 할 수 있으며 XSS (Cross-Site Scripting) 공격에 취약 할 수 있습니다. 
XSS는 간단히 말해 공격자가 페이지에서 실행되는 JavaScript를 주입 할 수있는 취약점 유형입니다.
기본 XSS 공격은 양식 입력을 통해 JavaScript를 주입하려고 시도합니다.

사용하는 스크립트 중 하나만 해킹 당하면 어떻게 될까요? 
악의적인 JavaScript가 페이지에 삽입되어 웹 저장소가 손상 될 수 있습니다.
이러한 유형의 XSS 공격은 모르는 사이에 사이트를 방문하는 모든 사람의 Web Storage를 확보 할 수 있습니다.
대다수의 조직들이 웹 스토리지에 세션과 토큰을 포함한 어떤 값도 저장하지 않기를 권장하지 않는 이유입니다.
스토리지 메커니즘으로서 Web Storage는 전송 중에 어떠한 보안 표준도 시행하지 않습니다.
웹 스토리지를 사용할 때는 항상 HTTPS를 통해 JWT를 전송하고 HTTP를 사용하지 않도록 해야 합니다.

## JWT 쿠키 저장 보안
쿠키는 HttpOnly쿠키 플래그 와 함께 사용될 때 JavaScript를 통해 액세스 할 수 없으며 XSS로부터 영향을받지 않습니다. 
Secure쿠키가 HTTPS를 통해서만 전송되도록 쿠키 플래그를 설정할 수도 있습니다. 
이것은 과거에 토큰이나 세션 데이터를 저장하기 위해 쿠키가 활용 된 주된 이유 중 하나입니다. 
현대 개발자는 전통적으로 서버에 상태를 저장해야하므로 RESTful 모범 사례를 위반하기 때문에 쿠키 사용을 주저합니다.
저장 메커니즘으로서의 쿠키는 JWT를 쿠키에 저장하는 경우 서버에 상태를 저장할 필요가 없습니다. 
이는 JWT가 서버가 요청을 처리하는 데 필요한 모든 것을 캡슐화하기 때문입니다.

그러나 쿠키는 다른 유형의 공격 (CSRF (cross-site request forgery))에 취약합니다. 
CSRF 공격은 악의적인 웹 사이트, 전자 메일 또는 블로그로 인해 사용자의 웹 브라우저가 사용자가 현재 인증 된 신뢰할 수있는 사이트에서 원치 않는 작업을 수행 할 때 발생하는 공격 유형입니다. 이것은 브라우저가 쿠키를 처리하는 방법에 대한 악용입니다. 쿠키는 쿠키가 허용 된 도메인에만 보낼 수 있습니다. 
기본적으로이 쿠키는 원래 쿠키를 설정 한 도메인입니다. 

