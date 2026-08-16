# Language-Specific Conventions and Idioms

## Rust

- **Entry point**: `main.rs` or `lib.rs`; check `[lib]` and `[[bin]]` in `Cargo.toml`
- **Module system**: `mod.rs` or inline modules; re-exports in `lib.rs` define public API
- **Error handling**: `Result<T, E>` and `?` operator; look for custom error enums with `thiserror`
- **Async**: `tokio` or `async-std` runtime; look for `#[tokio::main]`
- **Traits**: Core abstractions live as traits; `impl Trait for Type` is the extension point
- **Testing**: `#[cfg(test)]` inline, or `tests/` for integration tests; `examples/` for runnable demos
- **Feature flags**: `[features]` in `Cargo.toml`; `#[cfg(feature = "...")]` in code

## TypeScript / JavaScript

- **Entry point**: `index.ts`, `main.ts`, `app.ts`; check `"main"` and `"exports"` in `package.json`
- **Module system**: ESM vs CJS; check `"type": "module"` in `package.json`
- **Error handling**: Try/catch, Promise rejections, Result-pattern libraries (`neverthrow`)
- **Async**: Promises, async/await; watch for missing `await` and unhandled rejections
- **Types**: Interfaces for shapes, types for unions; `unknown` preferred over `any`
- **Testing**: Jest, Vitest, Mocha; fixtures in `__fixtures__`, mocks in `__mocks__`
- **Framework hints**: Next.js (`pages/` or `app/`), Express (`routes/`), NestJS (decorators)

## Python

- **Entry point**: `__main__.py`, `main.py`, or CLI via `pyproject.toml` scripts
- **Module system**: `__init__.py` defines packages; relative imports signal internal boundaries
- **Error handling**: Exceptions; look for custom exception hierarchies
- **Async**: `asyncio`, `aiohttp`, `FastAPI`; mixing sync/async is a common source of bugs
- **Testing**: `pytest`; fixtures in `conftest.py`; look for `@pytest.mark` for test categories
- **Type hints**: `mypy` or `pyright` for static checking; `Protocol` for structural typing

## Go

- **Entry point**: `main.go` in `cmd/` or root
- **Module system**: `go.mod`; packages map to directories
- **Error handling**: Multiple return values `(result, error)`; check for ignored errors `_`
- **Concurrency**: Goroutines and channels; look for `sync.Mutex`, `sync.WaitGroup`, `context.Context`
- **Interfaces**: Implicit; any type with matching methods satisfies an interface
- **Testing**: `_test.go` files; table-driven tests are idiomatic

## Java / Kotlin

- **Entry point**: `main()` method; Spring Boot uses `@SpringBootApplication`
- **Module system**: Maven (`pom.xml`) or Gradle (`build.gradle`)
- **Error handling**: Checked vs unchecked exceptions; `Optional<T>` for nullable
- **Async**: CompletableFuture, Reactor, Coroutines (Kotlin)
- **DI**: Spring `@Autowired`, Guice, Dagger; look for `@Component`, `@Service`, `@Repository`
- **Testing**: JUnit, Mockito; `@Test`, `@Mock`, `@InjectMocks`
