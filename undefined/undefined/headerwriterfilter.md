# HeaderWriterFilter

세세하게 다 찾아보기엔 시간이 너무 오래걸릴 것 같은데 그래도 시큐리티가 이정도까지 기능해준다는 차원에서 인지할만한 필터인 것 같아서 강의 자료를 그대로 옮긴다.

## HeaderWriterFilter 응답 헤더에 시큐리티 관련 헤더를 추가해주는 필터

* XContentTypeOptionsHeaderWriter: 마임 타입 스니핑 방어.
* XXssProtectionHeaderWriter: 브라우저에 내장된 XSS 필터 적용.
* CacheControlHeadersWriter: 캐시 히스토리 취약점 방어.
* HstsHeaderWriter: HTTPS로만 소통하도록 강제.
* XFrameOptionsHeaderWriter: clickjacking 방어.\


**내가 요청하고 받은 것(의도적으로 잘못된 토큰 보냈던 것)**

```bash
HTTP/1.1 401 
X-Content-Type-Options: nosniff
X-XSS-Protection: 0
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
Content-Type: application/json;charset=UTF-8
Content-Length: 13
Date: Thu, 15 Jun 2023 11:54:14 GMT
Keep-Alive: timeout=60
Connection: keep-alive
```



**강의자료 내 샘플**

```bash
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Content-Language: en-US
Content-Type: text/html;charset=UTF-8
Date: Sun, 04 Aug 2019 16:25:10 GMT
Expires: 0
Pragma: no-cache
Transfer-Encoding: chunked
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
```
