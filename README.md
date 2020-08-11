## 회원인증 API 과제 가이드 ##

#### 회원 가입, 로그인 및 조회 서비스를 위한 Back-End API.

주요특징:
* JWT (Json Web Token) authentication 과  Spring Security를 이용한 Backend API 인증
* JPA (Java Persistence API) 를 이용하여 database 테이블을 Entity class로 생성하여 sql 관리나 변경없이 다양한 DBMS에 즉시 대응. 그리고, EntityManager를 이용하여 Entity 클래스로 DB 테이블 자동 생성.
* Swagger를 이용한 API Docs 자동화

지원기능들:
* 회원가입 API - 이메일, 패스워드, 이름
* 로그인 API - Spring Security와 JWT token 생성
* 회원정보 조회 API - JWT token을 이용한 회원정보(이메일/이름/직전로그인시간) 조회

---

# 환경설정
## application.properties 설정
	+ 경로 `src/main/resources/application.properties`
```xml
#Server properties
server.port=8000

rest.api.ver=/v1

#Datasource
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/member_db?serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=1234
spring.datasource.testWhileIdle=true
spring.datasource.validationQuery=SELECT 1

#Spring JPA
spring.jpa.hibernate.ddl-auto=create
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.datasource.initialization-mode=always
spring.jpa.properties.hibernate.format_sql=true
logging.level.org.hibernate.SQL=DEBUG

#JWT
app.jwt.header=Authorization
app.jwt.header.prefix=Bearer 
app.jwt.secret=mySecret
app.jwt.expiration=900000

#Jackson properties
spring.jackson.serialization.WRITE_DATES_AS_TIMESTAMPS=false
spring.jackson.time-zone=UTC

#Token properties
app.token.refresh.duration=2592000000
```

---

## Swagger를 이용한 API Docs 자동화 ##
API 문서의 버전관리 및 현행화가 제대로 이루어지지 않는 이슈를 최소화하고 프론트엔드 개발자와의 원활한 커뮤니케이션을 위해 목적으로 Swagger를 활용.
* 소스코드에 적용된 API Spec을 추출하여 웹페이지로 제공함으로써 정확한 Request와 Response를 정확하고 신속하게 파악할 수 있음. 

Swagger 소스코드내 적용 예시)
```sql
package com.bithumbhomework.member.controller.api.v1;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiParam;

@RestController
@RequestMapping("/v1/member")
@Api(value = "Member Rest API", description = "회원가입, 로그인, 회원 조회 API")

public class MemberControllerV1 {

	private static final Logger logger = Logger.getLogger(MemberControllerV1.class);
	private final MemberAuthService memberService;
	private final UserService userService;
	private final UserLoginService userLoginService;
	private final JwtTokenProvider tokenProvider;

	@Autowired
	public MemberControllerV1(MemberAuthService memberService, UserService userService, UserLoginService userLoginService,
			JwtTokenProvider tokenProvider) {
		this.memberService = memberService;
		this.userService = userService;
		this.userLoginService = userLoginService;
		this.tokenProvider = tokenProvider;
	}

	/**
	 * 1. 회원가입 URI는 다음과 같습니다. : /v1/member/join 
	 * 2. 회원가입 시 필요한 정보는 ID, 비밀번호, 사용자이름 입니다. 
	 * 3. ID는 반드시 email 형식이어야 합니다. 
	 * 4. 비밀번호는 영어 대문자, 영어 소문자, 숫자, 특수문자 중 3종류 이상으로 12자리 이상의 문자열로 생성해야 합니다. 
	 * 5. 비밀번호는 서버에 저장될 때에는 반드시 단방향 해시 처리가 되어야 합니다.
	 */
	@PostMapping("/join")
	@ApiOperation(value = "회원 가입")
	public ResponseEntity joinUser(
			@ApiParam(value = "The JoinRequest payload") @Valid @RequestBody JoinRequest joinRequest) {

		return memberService.joinUser(joinRequest).map(user -> {
			logger.info("joinUser ==> " + user);
			return ResponseEntity.ok(new ApiResponse(true, "회원가입이 성공하였습니다."));
		}).orElseThrow(() -> new UserJoinException(joinRequest.getEmail(), "Missing user object in database"));
	}
}
```

Swagger UI : 
* http://localhost:8000/swagger-ui.html
![image](https://user-images.githubusercontent.com/15791988/89800836-43e02200-db6a-11ea-941f-15dfaf48f916.png)
![image](https://user-images.githubusercontent.com/15791988/89800932-65d9a480-db6a-11ea-8fc3-1db596008c40.png)
![image](https://user-images.githubusercontent.com/15791988/89800952-6c681c00-db6a-11ea-94ff-28e3be7f1087.png)
![image](https://user-images.githubusercontent.com/15791988/89801012-8570cd00-db6a-11ea-90ad-a14db2c41584.png)


Postman등의 REST API 클라이언트 툴을 활용한 API 기능 검증:
![image](https://user-images.githubusercontent.com/15791988/89816203-16eb3980-db81-11ea-9bbc-8cf67728e6f8.png)

---


## Database
* Entity class를 작성하여 JPA를 통해 데이터베이스 테이블이 자동으로 생성되도록 작성. (CreateAndUpdate)
* 모든 테이블과 컬럼들을 Entity 오브젝트로 관리함으로써 Text query문을 이용할때의 문제점(테이블, 컬럼들이 소스코드와 현행화와 검증이 안되는 이슈)들을 해소.
* JPA entity를 통해 데이터베이스에 자동으로 생성된 테이블을 이용하여 쉽게 테이블생성 query, 테이블명세서, ERD를 생성하고 관리할 수 있음.

Entity classes
![image](https://user-images.githubusercontent.com/15791988/89894892-d093ea00-dc15-11ea-899c-d110b038f74b.png)

테이블 생성 Scripts
```sql
-- member_db.refresh_token_seq definition

CREATE TABLE `refresh_token_seq` (
  `next_val` bigint(20) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


-- member_db.`user` definition

CREATE TABLE `user` (
  `user_id` bigint(20) NOT NULL,
  `created_at` datetime NOT NULL,
  `updated_at` datetime NOT NULL,
  `email` varchar(255) DEFAULT NULL,
  `password` varchar(255) DEFAULT NULL,
  `username` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`user_id`),
  UNIQUE KEY `UK_ob8kqyqqgmefl0aco34akdtpe` (`email`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


-- member_db.user_login_seq definition

CREATE TABLE `user_login_seq` (
  `next_val` bigint(20) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


-- member_db.user_seq definition

CREATE TABLE `user_seq` (
  `next_val` bigint(20) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


-- member_db.user_login definition

CREATE TABLE `user_login` (
  `user_login_id` bigint(20) NOT NULL,
  `created_at` datetime NOT NULL,
  `updated_at` datetime NOT NULL,
  `is_refresh_active` bit(1) DEFAULT NULL,
  `login_id` varchar(255) NOT NULL,
  `notification_token` varchar(255) DEFAULT NULL,
  `user_id` bigint(20) NOT NULL,
  PRIMARY KEY (`user_login_id`),
  KEY `FKpuv1tgwbg2fgmw93xktb9rjs5` (`user_id`),
  CONSTRAINT `FKpuv1tgwbg2fgmw93xktb9rjs5` FOREIGN KEY (`user_id`) REFERENCES `user` (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


-- member_db.refresh_token definition

CREATE TABLE `refresh_token` (
  `token_id` bigint(20) NOT NULL,
  `created_at` datetime NOT NULL,
  `updated_at` datetime NOT NULL,
  `expiry_dt` datetime NOT NULL,
  `refresh_count` bigint(20) DEFAULT NULL,
  `token` varchar(255) NOT NULL,
  `user_login_id` bigint(20) NOT NULL,
  PRIMARY KEY (`token_id`),
  UNIQUE KEY `UK_7xw3t1qjql8we1oluftl4flv4` (`user_login_id`),
  UNIQUE KEY `UK_r4k4edos30bx9neoq81mdvwph` (`token`),
  CONSTRAINT `FK1afld87meo1qf4lhwge9iyqc9` FOREIGN KEY (`user_login_id`) REFERENCES `user_login` (`user_login_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```


ER Diagream
![image](https://user-images.githubusercontent.com/15791988/89800540-e77d0280-db69-11ea-9b2d-aea72a334414.png)


테이블 명세서
![image](https://user-images.githubusercontent.com/15791988/89800659-0a0f1b80-db6a-11ea-805d-8fa8e8c5cd50.png)



## JWT tokens and security using username, password
![image](https://user-images.githubusercontent.com/15791988/89894531-2f0c9880-dc15-11ea-9520-eea2980c65c4.png)

