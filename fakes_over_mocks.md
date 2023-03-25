## Built-in Fakes vs Mocks


We have client libraries for our service that are used both by our team (from different services) and by other teams. The idea was to create a built-in fake implementation of this library that can be imported with as `<scope>test</scope>` .

Let's take this very simple example:

```java
public interface AccountApiClient {
  Mono<AccountDto> findByUsername(String username);
  Mono<AccountDto> findById(long id);
}
```

Basically, the library (let's say `account-api-client`) will expose this public interface and two implementations: a real one and a fake one for tests:

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

Let's assume we have this simplee class that uses the lib (this will be the class under test):

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

Using the fake implemenrtation instead of mockito, the test can look like this:

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
    .isInstanceOf(MyCustomException.class)
    .hasMessage("account with id 1 was not found");
}
```

Here are the main advantages I found for this approach:

### 1. Decouple test from implementation

The method `AccounsService.getDisplayName` can change its implementation and start using `client.findByUsername(username)` instead of `client.findById(id)`: the test simply does not care.

Similarly, in other scenarios we can have methods such as `save(Dto dto)`, `save(List<Dto> dtos)` - the test will not care which method you use, as long as the dtos are stored inside the fake. For instance, the test won't care if you save the dtos all the once or if you save them in batches.

### 2. Add *Trivial* logic

For example, this trivial argument validation:
```java
if (StringUtils.isBlank(username)) {
  return Mono.error(() -> new IllegalArgumentException("username cannot be blank"));
}
```
The main idea here is that these **very simple** validations or bits of logic are decleared by the owners of the library. I believe this can be a good idea from multiple points of view:
- the team owning the lib knows <del>better</del> these cases
- they can update these trivial validations (or add new ones) keeping them in-sync with the real implementation.
- the team using the client and testing with the fake can update the version, re-run the tests and see if anything broke (how nice is that?) // a bit similar to what contract testing offers
- the behavior of all fakes can be changed from a common place. Whereas, for a change in the API you might check many mocks from different project and update them
 
 PS: main challange here would be **not** to add to much logic in the fakes. They should be rather simple.
 
 
 ### 3. They can suggest the posibile outcomes and 'remind' you to test your error handling
 
 
I would consider adding some test data there by default, for all the known outcomes of the API. The fake will look for these values first and it will reply a bit differently for them. It can be done in a nicer way, this is just to illustrate the point:

```java
public enum TestData {
  SIMPLE_ACCOUNT(901L), // returs john doe
  ACCOUNT_NOT_FOUND(902L), // returns Mono.empty()
  SLOW_ACCOUNT(903L), // returns a Mono with some delay
  SERVER_ERROR(904L); // returns the appropriate mono.error(...)
 
  public final long id;
}
```
This can be used like this:
 
```java
@Test
void accountNotFound() {
  assertThatThrownBy(() -> service.getDisplayName(ACCOUNT_NOT_FOUND.id()))
    .isInstanceOf(MyCustomException.class)
    .hasMessage("...");
}
```
But, more importantly, it is suggesting to test your error handling on other unexpected outcomes, for instace slow responses, timeouts, server erros etc.
 
This idea mainly driven by the bad mocks I've seen: Mockito allows you to `.thernThrows(() -> new RuntimeExceptionA())` even if `RuntimeExceptionA` is  never thrown by that function. And, if we wrap the exceptions in Mono.error it can get even trickyer to get the correct mocks (and keep them in sync).  

 
 
