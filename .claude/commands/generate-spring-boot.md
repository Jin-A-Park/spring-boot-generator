프로젝트 루트의 `schema.yaml`을 읽고 Spring Boot 프로젝트 파일들을 생성해줘.

---

## 생성 대상

각 엔티티마다 아래 6개 파일을 생성한다:

- `src/main/java/{package}/entity/{Name}.java`
- `src/main/java/{package}/repository/{Name}Repository.java`
- `src/main/java/{package}/service/{Name}Service.java`
- `src/main/java/{package}/controller/{Name}Controller.java`
- `src/main/java/{package}/dto/request/{Name}RequestDto.java`
- `src/main/java/{package}/dto/response/{Name}ResponseDto.java`

모든 엔티티 생성 후 마지막에:

- `src/main/resources/test.http`

---

## 규칙

### [Entity]
- `@Entity`, `@Getter`, `@NoArgsConstructor`, `@AllArgsConstructor`, `@ToString` 사용 (`@Builder` 제외)
- FK 필드는 `@ManyToOne` + `@JoinColumn(name="{snake_case}_id")`
- `createdAt` 필드가 있으면 `@CreationTimestamp` 사용
- `patch(RequestDto dto)` 메서드: null 체크 후 수정 가능한 필드만 업데이트 (FK 필드 제외)

### [Repository]
- `JpaRepository<{Name}, Long>` 상속
- FK가 있으면 `findBy{Parent}Id(Long id)`, `deleteBy{Parent}Id(Long id)` 추가

### [Service]
- `@Service`, `@Slf4j`, `@RequiredArgsConstructor`
- 반환 타입은 반드시 ResponseDto (Entity 직접 반환 금지)
- FK가 있으면 `create(Long parentId, RequestDto dto)` / `readAll(Long parentId)` 형태
- FK가 없으면 `create(RequestDto dto)` / `readAll()` 형태
- 각 메서드 첫 줄에서 존재 여부 확인, 없으면 한국어 예외 메시지로 `IllegalArgumentException`
- 예외 메시지 형식: `"{엔티티명} {작업} 실패: {이유}"`

### [Controller]
- `@RestController`, `@Slf4j`, `@AllArgsConstructor`
- 모든 메서드는 `ResponseEntity<ResponseDto>` 반환
- `create` → `HttpStatus.CREATED`, `update/delete` → `HttpStatus.OK`, `read` → `ResponseEntity.ok()`
- FK가 있는 경우 URL 컨벤션:
  - `GET    /{name}/{id}`                  → 단건 조회
  - `GET    /{parent}/{parentId}/{name}`   → 목록 조회
  - `POST   /{parent}/{parentId}/{name}`   → 생성
  - `PATCH  /{name}/{id}`                 → 수정
  - `DELETE /{name}/{id}`                 → 삭제
- FK가 없는 경우 URL 컨벤션:
  - `GET    /{name}`       → 전체 목록
  - `GET    /{name}/{id}`  → 단건 조회
  - `POST   /{name}`       → 생성
  - `PATCH  /{name}/{id}`  → 수정
  - `DELETE /{name}/{id}`  → 삭제

### [RequestDto]
- `@Getter`, `@NoArgsConstructor`, `@AllArgsConstructor`, `@ToString`
- FK 필드는 `@JsonProperty("{snake_case}_id")` 로 선언
- `toEntity()` 또는 `toEntity(ParentEntity parent)` 메서드 포함

### [ResponseDto]
- `@Getter`, `@NoArgsConstructor`, `@AllArgsConstructor`, `@ToString`
- FK 필드는 id만 포함하고 `@JsonProperty("{snake_case}_id")` 로 선언
- static factory 메서드명은 반드시 `from(Entity entity)`

### [test.http]
- 상단에 프로젝트명과 Base URL 주석 표기
- 엔티티마다 섹션 구분
- 각 섹션에 CRUD 요청 포함 (FK 없는 엔티티만 전체 목록 조회 포함)
- URL은 위 Controller 컨벤션과 동일하게 적용 (PascalCase → kebab-case 변환)
- 요청 body는 필드 타입과 이름에 맞는 자연스러운 샘플값 사용
- 수정 가능한 일반 필드가 없는 엔티티(FK만 있는 경우)는 PATCH 섹션 생략
