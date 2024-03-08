---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
---

# 패스워드 암호화

사용자의 비밀번호를 그대로 저장한다면, 보안상 문제가 생길 수 있다.

따라서 패스워드 같은 정보는 암호화하여 저장해야 한다.

이러한 암호화에 두 가지 방법이 사용될 수 있다.

* 양방향 암호화: 암호화된 암호문을 복호화할 수 있는 암호화
* 단방향 암호화: 암호화된 암호문을 복호화 불가능한 암호화

복호화가 불가능한 방식으로 암호를 처리하는 것이 보안적으로 더 안전하다. 하지만 비밀번호 찾기는 불가능하고 재설정만 가능하다.

단방향 암호화는 주로 Hash 기법을 활용하는데, Spring Security에서 제공하는 Hashing 알고리즘 중 BCrypt 알고리즘을 이용해보자.

### 수동 빈 등록

Spring Security에서 비밀번호 암호화를 위해 PasswordEncoder 인터페이스를 제공한다. 그 구현체 중 하나로 BCryptPasswordEncoder가 있다.

BCrypt hashing 알고리즘을 적용하기 위해 빈에 등록해준다.

```java
@EnableWebSecurity
@Configuration
public class SecurityConfig {

    /**
     * PasswordEncoder interface의 구현체가 BCryptPasswordEncoder임을 수동 빈 등록을 통해서 명시한다.
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

#### PasswordEncoder

```java
public interface PasswordEncoder {
    String encode(CharSequence rawPassword);

    boolean matches(CharSequence rawPassword, String encodedPassword);

    default boolean upgradeEncoding(String encodedPassword) {
        return false;
    }
}
```

PasswordEncoder는 인터페이스이다.

모든 Spring Security의 비밀번호 Encoder는 PasswordEncoder 인터페이스를 구현한다.

* `encode()` : 평문인 비밀번호를 암호화한다.
* `matches()` : 평문 비밀번호를 인코딩된 비밀번호와 비교한다.

#### 사용

MemberService에 회원가입 로직이 있다고 가정해보자.

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class MemberService {

    private final PasswordEncoder passwordEncoder;
    private final MemberRepository memberRepository;

    public Long join(MemberDto memberDto) {

        Optional<Member> findMember = memberRepository.findByEmail(memberDto.getEmail());
        if (!findMember.isEmpty()) {
            log.error("이미 존재하는 이메일={}", memberDto.getEmail());
            return null;
        }
        Member member = new Member(memberDto.getEmail(), passwordEncoder.encode(memberDto.getPassword()), memberDto.getNickname());
        memberRepository.save(member);
        log.info("joined member={}", member);
        return member.getId();
    }
}
```

입력받은 비밀번호를 주입받은 passwordEncoder를 통해 암호화하여 저장한다.

이후 로그인할 때는 `matches()` 를 통해 비교하여 로그인한다.

```java
@Service
@RequiredArgsConstructor
public class LoginService {

    private final PasswordEncoder passwordEncoder;
    private final MemberRepository memberRepository;

    public Member login(String loginEmail, String password) {
        Optional<Member> findMembers = memberRepository.findByEmail(loginEmail);
        return findMembers.stream()
                .filter(member -> passwordEncoder.matches(password, member.getPassword()))
                .findAny()
                .orElse(null);
    }
}
```
