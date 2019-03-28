# Middleware

이 문서는 Django와 함께 제공되는 모든 미들웨어 구성요소를 설명한다.



## Available middleware



### Cache middleware

**class UpdateCacheMiddleware**

**class FetchFromCacheMiddleware**

사이트 전체 캐시를 사용하도록한다. 활성화된 경우 장고가 제공하는 페이지는 **CACHE_MIDDLEWARE_SECONDS**설정이 정의하는만큼 캐쉬된다.

---

### "Common" middleware

**class CommonMiddleware**

완벽주의자를 위한 몇가지 편의를 추가한다.

- **DISALLOWED_USER_AGENTS** 설정은 유저의 접근을 금지한다. 정규 표현식으로 컴파일된 객체의 목록이여야한다.
- **APPEND_SLASH** 및 **PREPEND_WWW** 설정을 기반으로 URL 재작성을 수행한다.
  - **APPEND_SLASH**가 **True**이면 초기 URL이 후행 슬래시가 없으면 끝에 슬래시를 추가한 URL이 형성된다. 후행슬래시가 없는 요청이오면 후행 슬래시가 있는 URL로 리디렉션한다.
  - **PREPEND_WWW**가 **True**이면 앞에 **www.**가 없는 경우 **www.**의 URL로 리디렉션한다.
- non-streaming 응답을 위한 **Content-Lenght**헤더를 세팅한다.

**CommonMiddleware.response_redirect_class**

기본값은 **HttpResponsePermanentRedirect**이다. **CommonMiddleware** 속성을 재정의하여 미들웨어에서 실행된 리디렉션을 사용자정의할 수 있다.

**class BrokenLinkEmailsMiddleware**

끊어진 링크 알림을 **MANAGERS**로 전송한다.

---

### GZip middleware

**class GZipMiddleware**

> Warning
>
> 최근 압축 기술이 웹 사이트에서 사용될 때 사이트가 여러 공격에 노출될 수 있음을 밝혔다. 사이트에서 **GZipMiddleware**를 사용하기 전 이러한 공격의 대상이 되는지 매우 신중히 고려해야한다.

이 미들웨어는 응답 본문을 읽거나 쓰는 다른 미들웨어보다 먼저 배치해야 나중에 압축이 이루어진다.

다음 중하나라도 해당하면 내용을 압축하지 않는다.

- 200bytes 미만의 콘텐츠
- **Content-Encoding**헤더가 설정된 경우
- 브라우저가 **Accept-Encoding**헤더에 **gzip**을 포함하지 않은 경우

**gzip_page()**데코레이터를 사용하여 개별 뷰에 **GZip** 압축 적용이 가능하다.

---

### Conditional GET middleware

**class ConditionalGetMiddleware**

조건부 GET 동작을 처리한다, 응답 헤더에 **ETag**가 없다면 필요에 따라 미들웨어가 추가한다. 응답에 **ETag** 또는 **Last-Modified**헤더가 있고 요청에 **If-None-Match** 또는 **If-Modified-Since**가 있다면 응답은 **HttpResponseNotModified**로 바뀐다.

---

### Locale middleware

**class LocaleMiddleware**

요청데이터를기반으로 언어를 선택한다.

**LocaleMiddleware.response_redirect_class**

기본값은 **HttpResponseRedirect**이다. **LocalMiddleware** 서브클래스와 속성을 오버라이드하여 사용자 정의한 경로로 리디렉션 가능하다.

---

### Message middleware

**class MessageMiddleware**

쿠키 및 세션 기반 메세지 지원을 사용한다.

---

### Security middleware

**class SecurityMiddleware**

request/response cycle에 대해 몇가지 보안 강화 기능을 제공한다. 각 설정은 개별적으로 활성화 또는 비활성화 가능하다.

- **SECURE_BROWSER_XSS_FILTER**
- **SECURE_CONTENT_TYPE_NOSNIFF**
- **SECURE_HSTS_INCLUDE_SUBDOMAINS**
- **SECURE_HSTS_PRELOAD**
- **SECURE_HSTS_SECONDS**
- **SECURE_REDIRECT_EXEMPT**
- **SECURE_SSL_HOST**
- **SECURE_SSL_REDIRECT**

#### HTTP Strict Transport Security

**HTTPS**를 통해서만 액세스해야하는 경우, **Strict-Transport-Security**헤더를 설정하여 최신 브라우저가 안전하지 않은 연결을 통해 도메인 이름에 연결하는것을 거부할 수 있도록 지시할 수 있다. 이는 **MITM** 공격에 노출될 위험이 줄게된다.

**SecurityMiddleware**는 **SECURE_HSTS_SECONDS** 설정을 0이 아닌 정수 값으로 설정하면 모든 HTTPS 응답에서 이 헤더를 설정한다.

**HSTS**를 활성활 할 때 먼저 테스트에 작은 값을 사용하는 것이 좋다 (예: **SECURE_HSTS_SECONDS = 3600**) 웹 브라우저가 사이트의 **HSTS**헤더를 볼 때마다 지정된 시간 동안 도메인과 안전하지 않게 (**HTTP를 사용하여**) 통신하는것을 거부한다. 사이트의 모든 저작물이 안전하게 제공되었음을 확인하려면 (즉, HSTS가 깨지지 않았음)이 값을 늘려 방문자를 보호할 수 있다. (31536000 초, 1년이 보통이다.)

게다가 **SECURE_HSTS_INCLUDE_SUBDOMAINS** 설정은 **True**로한다면, **SecurityMiddleware**는 **includeSubDomains** 디렉티브를 **Strict-Transport-Security**헤더에 추가한다. 이는 (모든 서브도메인이 HTTPS만 사용할 경우)추천된다. 다른 경우 서브도메인으로의 안전하지않은 연결로인해 여전히 취약할 수 있다.

사이트의 **browser preload list**를 제공하고싶다면 **SECURE_HSTS_PRELOAD**설정을 **True**로 한다. 이는 **Strict-Transport-Security** 헤더에 **preload** 디렉티브를 추가한다.

---

### X-Content-Type-Options: nosniff

---

### X-XSS-Protection: 1; mode=block

---

### SSL Redirect

---

### Session middleware

---

### Site middleware

들어오는 모든 **HttpRequest**객체에 현재 사이트를 나타내는 특성을 추가한다.

장고가 하나 이상의 사이트를 구별해야하는 경우 사용된다.

---

### Authentication middleware

**class AuthenticationMiddleware**

들어오는 모든 **HttpRequest**객체에 현재 로그인한 사용자를 나타내는 **user**속성을 추가한다.

**class RemoteUserMiddleware**

외부인증소스를 장고 응용프로그램에서 사용할 때 사용된다. 이 유형의 인증 솔루션은 일반적으로 IIS 및 Windows 통합 인증 또는 Apache같은 SSO솔루션에 사용된다.

**class PersistentRemoteUserMiddleware**

**RemoteUserMiddleware**는 로그인페이지에서만 유효하지만 이는 세션을 관리하여 사용자가 명시적으로 로그아웃할 때까지 인증된 세션을 유지 관리한다.

---

### CSRF protection middleware

**class CsrfViewMiddleware**

POST 양식에 숭겨진 CSRF 필드를 추가한다.

---

### X-Frame-Options middleware

**class XFrameOptionsMiddleware**

**iframe**에 로드 한 다른 사이트의 숨겨진 요소를 사용자가 클릭하도록 유도하는것을 방지한다.

---

## Middleware Ordering

1. **SecurityMiddleware**

   불필요한 미들웨어를 통해 실행되지 않도록 SSL 리디렉션을 활성화하려면 상단으로 가야한다.

2. **UpdateCacheMiddleware**

   **Vary** 헤더를 수정하기 전 (**SessionMiddleware**, **GZipMiddleware**, **LocaleMiddleware**).

3. **GZipMiddleware**

   응답 본문을 변경하거나 사용할 수 있는 모든 미들웨어 전

   **UpdateCacheMiddlware** 뒤: **Vary**를 수정한다.

4. **SessionMiddleware**

   **UpdateCacheMiddleware** 뒤: **Vary**를 수정한다.

5. **ConditionalGetMiddleware**

   응답 변경할 수 있는 미들웨어가 있기 전 (**ETag**를 설정한다.)

   **GZipMiddleware** 이후 **gzip**으로 압축된 내용의 **ETag** 헤더를 계산하지 않는다.

6. LocaleMiddleware

   **SessionMiddleware**(세션 데이터 사용) 및 **UpdateCacheMiddleware**(Vary 헤더 수정) 이후 최상위 클래스

7. **CommonMiddleware**

   응답을 변경할 수 있는 미들웨어 (**Content-Length** 설정한다.) 이전. **CommonMiddleware** 앞에 나타나고 응답을 변경하는 미들웨어는 **Content-Lenght**를 재설정해야한다.

   **APPEND_SLASH** 및 **PREPEND_WWW**가 설정된 경우 최대한 상단에 가깝게

8. **CsrfViewMiddleware**

   **CSRF** 공격이 처리되었다고 가정하는 View Middleware가 나오기 전

   **CSRF_USE_SESSIONS**를 사용하면 **SessionMiddleware** 이후 나와야한다.

9. **AuthenticationMiddleware**

   **SessionMiddleware** 이후: **session** 저장소를 사용함

10. **MessageMiddleware**

    **SessionMiddleware** 이후: **session**기반의 저장소를 사용할 수 있음

11. **FetchFromCacheMiddleware**

    **Vary** 헤더를 수정하는 모든 미들웨어 이후, 헤더가 캐시 해시의 키 값을 선택하는데 사용한다.

12. **FlatpageFallbackMiddleware**

    최하단에 위치해야한다.

13. **RedirectFallbackMiddleware**

    최하단에 위치해야한다.