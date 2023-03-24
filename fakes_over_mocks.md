


 
Let's assume this is the interace exposed by the library:

```java
public interface AccountApiClient {
  Mono<AccountDto> findByUsername(String username);
  Mono<AccountDto> findById(long id);
}
```

The idea was to create a built-in fake implementation. Basically, the library (`account-api-client`) will expose the public interface and two implementations: a real one and a fake one for tests:
PS: this fake impl can sit in a different module and be declared in pom with `<scope>test</scope>` 

```java
@RequiredArgsConstructor
public class FakeAccountApiClient implements AccountApiClient {
  private final List<AccountDto> accounts;

  @Override 
  public Mono<AccountDto> findByUsername(String username) {
    if (StringUtils.isBlank(username)) {
      return Mono.error(() -> new IllegalArgumentException("username cannot be blank"));
    }
    return accounts.stream()
      .filter(a -> a.username().equals(username))
      .findFirst()
      .map(Mono::just)
      .orElseGet(Mono::empty);
  }

  @Override
  public Mono<AccountDto> findById(long id) {
    return accounts.stream()
      .filter(a -> a.id() == id)
      .findFirst()
      .map(Mono::just)
      .orElseGet(Mono::empty);
  }
}
```

Let's assume this is the tetsed class, where this client is used: 

```java
//tested class
@RequiredArgsConstructor
public class AccountService {
  private final AccountApiClient apiClient;

  public String getDisplayName(Long id) {
    var acc = apiClient.findById(id)
      .blockOptional()
      .orElseThrow(() -> new MyCustomException())
      // .... some logic here
      return String.format("%s [#%s]", acc.username(), acc.id());
  }
}
```

The test can look like this:

```java
@BeforeEach
void beforeEach() {
  this.service = new AccountService(
    new FakeAccountApiClient(List.of(
      new AccountDto("john.doe", 123L),
      new AccountDto("hanna.doe", 456L)
    ))
  );
}

@Test
void okAccount() {
  assertThat(service.getDisplayName(123L))
    .isEqualTo("john.doe [#123]");
}

@Test
void accountNotFound() {
  assertThatThrownBy(() -> service.getDisplayName(1L))
    .isInstanceOf(M yCustomException.class)
    .hasMessage("account with id 1 was not found");
}
```

Here are the main advantages we found while using this over (Mockito) mocks:

**1. Decouple test from implementation**

The method `AccounsService.getDisplayName` can change its implementation and start using `client.findByUsername(username)` instead `client.findById(id)`: the test simply does not care.
Similarly, in other scenarios we can have methods such as `save(Dto dto)`, `save(List<Dto> dtos)` - the test will not care which ,ethod you use, as long as the `dtos` are stored inside the fake.

**2. Add *Trivial* logic** 

For example, this trivial argument validation:
```java
if (StringUtils.isBlank(username)) {
  return Mono.error(() -> new IllegalArgumentException("username cannot be blank"));
}
```

