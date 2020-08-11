
**회원가입**
----
  이름, 이메일, 패스워드를 이용한 회원가입
  

* **URL**

  /v1/member/join

* **Method:**
  
  `POST`
  
*  **URL Params**

   N/A

* **Data Params**

  ```json
  {
    "email": "aaa@bbb.com",
    "password": "password1",
    "username": "홍길동"
  }
  ```

* **Success Response:**
  
  가입이 성공한 경우, sucess 필드에 true값을 전달한다.

  * **Code:** 200 <br />
    **Content:** 
    ```json
    {
      "data": "회원가입이 성공하였습니다.",
      "success": true,
      "timestamp": "2020-08-10T16:31:11.690Z"
    }
    ```
 
* **Error Response:**

  이메일 형식이 아닌경우, 패스워드 생성규칙이 맞이 않는 경우 등등...

  * **Code:** 406 NOT_ACCEPTABLE <br />
    **Content:**
    ```json
    {
      "data": "[Error] 유효하지 않은 값입니다... Email = 'aaakorea.com'",
      "success": false,
      "timestamp": "2020-08-10T16:56:18.066Z",
      "cause": "com.bithumbhomework.member.exception.InvalidFormatRequestException"
    }
    ```

  OR

  * **Code:** 409 CONFLICT <br />
    **Content:** 
    ```json
    {
      "data": "[Error] 이미 존재하는 값입니다. Email = 'aaa@korea.com'",
      "success": false,
      "timestamp": "2020-08-10T16:58:47.698Z",
      "cause": "com.bithumbhomework.member.exception.ResourceAlreadyInUseException"
    }
    ```

* **Sample Call:**

  ```bash
  curl -X POST "http://localhost:8000/v1/member/join" -H "accept: */*" -H "Content-Type: application/json" -d "{ \"email\": \"aaa@korea.com\", \"password\": \"aaa1234567890AAA\"}"
    ```

* **Notes:**

  비밀번호는 영어 대문자, 영어 소문자, 숫자, 특수문자 중 3종류 이상으로 12자리 이상의 문자열로 생성
  
  
  
  
----
  
  
**회원로그인**
----
  이메일, 패스워드를 이용한 회원 로그인
  

* **URL**

  /v1/member/login

* **Method:**
  
  `POST`
  
*  **URL Params**

   N/A

* **Data Params**

  ```json
  {
    "email": "aaa@bbb.com",
    "password": "aaa1234567890AAA"
  }
  ```

* **Success Response:**
  
  가입이 성공한 경우, accessToken, refreshToken, tokenType, 로그인 유효시간을 전달한다.

  * **Code:** 200 <br />
    **Content:** 
    ```json
    {
      "accessToken": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIxIiwiaWF0IjoxNTk3MDc4Nzk3LCJleHAiOjE1OTcwNzk2OTd9.yyJ0YNcKKMZwMwlkK-9QHKIHXvhdaezV5FoGjwsKutPdAi9aW98uoNlsqVdlLoLy3lizxdM1IChMgK0RSuk6CA",
      "refreshToken": "3c0dde0b-7f14-43cd-b0bb-cf7b57bab01c",
      "tokenType": "Bearer ",
      "expiryDuration": 900000
    }
    ```
 
* **Error Response:**

  로그인 인증에 실패한 경우

  * **Code:** 417 EXPECTATION_FAILED <br />
    **Content:**
    ```json
    {
      "data": "Bad credentials",
      "success": false,
      "timestamp": "2020-08-10T17:05:43.791Z",
      "cause": "org.springframework.security.authentication.BadCredentialsException"
    }
    ```


* **Sample Call:**

  ```bash
  curl -X POST "http://localhost:8000/v1/member/login" -H "accept: */*" -H "Content-Type: application/json" -d "{ \"email\": \"aaa@korea.com\", \"password\": \"aaa1234567890AAA\"}"
    ```

* **Notes:**

  - TBD -
    
----

  
**회원정보 조회**
----
  accessToken 인증을 이용한 회원정보 조회 (이름, 이메일, 직전 로그인 날짜)
  

* **URL**

  /v1/member/info

* **Method:**
  
  `GET`
  
* **Authorization**

  ```bash
  Authorization: Bearer <accessToken>
  ```
  
*  **URL Params**

   N/A

* **Data Params**

  N/A

* **Success Response:**
  
  로그인이 된 사용자에 대해서는 사용자이름, Email, 직전 로그인 일시를 제공합니다.

  * **Code:** 200 <br />
    **Content:** 
    ```json
    {
        "username": "홍길동",
        "email": "aaa@korea.com",
        "lastLoginedAt": "2020-08-10T17:12:23Z"
    }
    ```
 
* **Error Response:**

  로그인이 안된 사용자는 HTTP Status Code를 401 (Unauthorized)로 응답합니다.

  * **Code:** 401 UNAUTHORIZED <br />
    **Content:**
    ```json
    {
        "data": "Malformed jwt token: [JWT] token: [aaaaa] ",
        "success": false,
        "timestamp": "2020-08-10T17:59:42.168Z",
        "cause": "com.bithumbhomework.member.exception.InvalidTokenRequestException",
        "path": "/v1/member/info"
    }
    ```


* **Sample Call:**

  ```bash
  curl --location --request GET 'http://localhost:8000/v1/member/info' --header 'Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIxIiwiaWF0IjoxNTk3MDc5NTQzLCJleHAiOjE1OTcwODA0NDN9.qvn1K-lES6MPQYmXwaboY4iqpwdLkoW_pXub0gPsZdiokTfRZIB8uSfl6AW3aB4X0Tu0WnQZX_zShjxcPOvDdw1'
    ```

* **Notes:**

  - TBD -
