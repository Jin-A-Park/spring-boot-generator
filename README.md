# Spring Boot Project Generator

`schema.yaml` 하나로 Spring Boot 백엔드 파일을 자동 생성하는 Claude Code 슬래시 커맨드입니다.

---

## 설치 (한 번만)

```bash
git clone https://github.com/{your-username}/spring-generator.git
mkdir -p ~/.claude/commands
cp spring-generator/.claude/commands/generate-spring-boot.md ~/.claude/commands/
```

---

## 사용법

### 1단계: 프로젝트 루트에 schema.yaml 작성

```yaml
package: com.example.myproject
name: my-project
java: 17
bootVersion: 3.5.0

entities:
  - name: Member
    fields:
      - name: loginId
        type: String
      - name: email
        type: String
      - name: createdAt
        type: LocalDateTime

  - name: Article
    fields:
      - name: title
        type: String
      - name: content
        type: String
      - name: member
        type: Member
        relation: ManyToOne
```

### 2단계: Claude Code에서 실행

```
/generate-spring-boot
```

---

## schema.yaml 가이드

### 프로젝트 설정

| 키 | 설명 | 예시 |
|----|------|------|
| `package` | 루트 패키지명 | `com.example.myproject` |
| `name` | 프로젝트 이름 | `my-project` |
| `java` | Java 버전 | `17` |
| `bootVersion` | Spring Boot 버전 | `3.5.0` |

### 필드 타입

| Java 타입 | 설명 |
|-----------|------|
| `String` | 문자열 |
| `Integer` / `Long` | 숫자 |
| `Boolean` | true / false |
| `LocalDate` | 날짜 |
| `LocalDateTime` | 날짜 + 시간 |
| 다른 엔티티명 | 연관 관계 |

### 연관 관계

```yaml
- name: member
  type: Member
  relation: ManyToOne
```

현재 `ManyToOne`을 지원합니다.

---

## 생성되는 파일 구조

```
src/main/java/{package}/
├── entity/
├── repository/
├── service/
├── controller/
└── dto/
    ├── request/
    └── response/

src/main/resources/
└── test.http
```

---

## 라이선스

MIT License
