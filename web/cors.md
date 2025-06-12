---
icon: person-from-portal
---

# CORS

## Q. CORS 정책이란 무엇인가요?

**A.**\
CORS(Cross-Origin Resource Sharing, 교차 출처 리소스 공유)는 웹 브라우저에서 \
서로 다른 출처(도메인, 프로토콜, 포트)의 리소스 요청을 안전하게 허용하거나 제한하는 보안 메커니즘입니다. \
브라우저는 기본적으로 동일 출처 정책(SOP)을 적용해 같은 출처에서만 리소스 접근을 허용합니다. \
CORS는 서버가 `Access-Control-Allow-Origin` 응답 헤더를 통해 특정 출처(예: `https://example.com`) \
또는 모든 출처(`*`)의 요청을 허용하도록 설정함으로써 이 제한을 완화합니다. \
이를 통해 프론트엔드 애플리케이션이 다른 도메인의 API를 호출할 때 안전한 데이터 교환이 가능해집니다.

## Q. SOP는 무엇인가요?

**A.**\
SOP(Same-Origin Policy, 동일 출처 정책)는 웹 브라우저의 기본 보안 정책으로, \
동일한 출처(프로토콜, 도메인, 포트)에서만 리소스 교환을 허용합니다. \
예를 들어, `https://example.com:443`과 `http://example.com:80`은 프로토콜 또는 포트가 달라 \
다른 출처로 간주됩니다. \
SOP는 악성 스크립트가 다른 출처의 사용자 데이터(예: 쿠키, 세션 정보)를 무단 접근하는 것을 방지해 보안을 강화합니다. \
CORS는 SOP의 제한을 완화하여, 서버가 특정 교차 출처 요청을 허용하도록 합니다.

## Q. CORS의 작동 방식에 대해 설명해주세요.

**A.**\
CORS는 브라우저와 서버 간 협력을 통해 교차 출처 요청을 처리합니다. \
브라우저가 다른 출처로 HTTP 요청을 보낼 때, 요청 헤더에 `Origin` 필드(예: `https://example.com`)를 포함해 \
요청 출처를 전달합니다. 서버는 이를 확인하고, 응답 헤더에 `Access-Control-Allow-Origin`을 추가해 \
허용할 출처를 명시합니다.&#x20;

예를 들어, `Access-Control-Allow-Origin: https://example.com`은 특정 출처를, \
`*`는 모든 출처를 허용합니다. 브라우저는 이 헤더를 검토해 요청 출처와 일치하면 요청을 처리하고, \
일치하지 않으면 CORS 에러를 발생시켜 자바스크립트에서 응답 데이터 접근을 차단합니다.

복잡한 요청(예: `POST`, 커스텀 헤더 사용 시)에서는 브라우저가 `OPTIONS` 메서드로 \
프리플라이트 요청을 보내 서버의 허용 정책(`Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`)을 확인합니다. \
서버가 적절히 응답하면 실제 요청이 진행됩니다.

## Q. 프리플라이트 요청이란 무엇이며, 언제 발생하나요?

**A.**\
프리플라이트 요청은 CORS에서 복잡한 요청(예: `POST`, `PUT`, 커스텀 헤더 또는 특정 MIME 타입 사용)을 \
보내기 전에 브라우저가 서버에 보내는 `OPTIONS` 메서드 요청입니다. \
이는 서버가 해당 요청을 허용하는지 확인하는 사전 점검 과정입니다.

* **발생 시점**:
  * `GET`, `HEAD`, `POST` 외의 메서드 사용.
  * `Content-Type`이 `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain` 외의 값일 때.
  * 커스텀 헤더(예: `X-Custom-Header`) 사용 시.
* **작동 방식**:\
  브라우저는 `OPTIONS` 요청에 \
  `Access-Control-Request-Method`와 `Access-Control-Request-Headers`를 포함해 서버에 전송합니다. \
  서버는 `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers` 등으로 응답하며, \
  허용 여부를 알려줍니다. 허용되면 실제 요청이 진행되고,  그렇지 않으면 CORS 에러가 발생합니다.
*   **예시**:

    ```javascript
    fetch("https://api.example.com/data", {
        method: "POST",
        headers: { "X-Custom-Header": "value" }
    });
    ```

    위 요청은 프리플라이트 요청을 유발하며, 서버는 다음과 같이 응답할 수 있습니다:

    ```
    Access-Control-Allow-Origin: https://example.com
    Access-Control-Allow-Methods: POST, GET
    Access-Control-Allow-Headers: X-Custom-Header
    ```
