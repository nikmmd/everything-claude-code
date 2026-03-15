# Java Coding Style

## Conventions

- Follow Google Java Style Guide
- Use `final` for variables that don't change
- Prefer records for immutable data carriers (Java 16+)
- Use `Optional` for return types that may be absent — never for parameters

## Immutability

```java
// Records for immutable data
public record User(String id, String name, String email) {}

// Unmodifiable collections
List<String> items = List.of("a", "b", "c");
Map<String, Integer> scores = Map.of("alice", 100, "bob", 95);
```

## Error Handling

- Use checked exceptions for recoverable errors
- Use unchecked exceptions for programming errors
- Always include context in exception messages
- Never catch `Exception` or `Throwable` broadly

## Streams & Lambdas

```java
// Prefer streams over loops for transformations
var activeUsers = users.stream()
    .filter(User::isActive)
    .map(User::name)
    .toList();
```

## Tools

- **Build**: Gradle or Maven
- **Formatting**: google-java-format
- **Linting**: SpotBugs, Error Prone
- **Testing**: JUnit 5, Mockito, AssertJ
