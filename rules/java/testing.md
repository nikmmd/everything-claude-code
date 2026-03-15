# Java Testing

## Framework: JUnit 5

```bash
./gradlew test                    # Run tests
./gradlew test --tests "*.UserTest"  # Specific test
./gradlew jacocoTestReport        # Coverage report
```

## Test Structure

```java
import static org.assertj.core.api.Assertions.*;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository repository;

    @InjectMocks
    private UserService service;

    @Test
    void shouldCreateUser() {
        var input = new CreateUserDTO("test@example.com", "Test");
        when(repository.save(any())).thenReturn(new User("1", "Test", "test@example.com"));

        var result = service.createUser(input);

        assertThat(result.name()).isEqualTo("Test");
        verify(repository).save(any());
    }

    @ParameterizedTest
    @ValueSource(strings = {"", " ", "invalid-email"})
    void shouldRejectInvalidEmail(String email) {
        assertThatThrownBy(() -> service.createUser(new CreateUserDTO(email, "Test")))
            .isInstanceOf(ValidationException.class);
    }
}
```

## Key Packages

JUnit 5, Mockito, AssertJ, Testcontainers, WireMock

Scope: `**/*.java`
