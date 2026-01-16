# Exercise 03: Test Automation & CI/CD

## Concepts Covered
- **1.8.6.d-l** Test frameworks, mocking, dependency injection
- **1.8.7.d-k** Continuous integration, test automation

## Objective

Implement test automation infrastructure and understand CI/CD practices.

## Requirements

### Rust Implementation

```rust
pub mod test_framework {
    /// Custom test runner
    pub struct TestRunner {
        tests: Vec<Test>,
        setup: Option<Box<dyn Fn()>>,
        teardown: Option<Box<dyn Fn()>>,
    }

    pub struct Test {
        pub name: String,
        pub test_fn: Box<dyn Fn() -> TestOutcome>,
        pub tags: Vec<String>,
        pub timeout: Option<std::time::Duration>,
    }

    #[derive(Debug, Clone)]
    pub enum TestOutcome {
        Passed,
        Failed(String),
        Skipped(String),
        Timeout,
    }

    impl TestRunner {
        pub fn new() -> Self;
        pub fn add_test<F>(&mut self, name: &str, test_fn: F)
        where
            F: Fn() -> TestOutcome + 'static;

        pub fn with_setup<F>(self, f: F) -> Self
        where
            F: Fn() + 'static;

        pub fn with_teardown<F>(self, f: F) -> Self
        where
            F: Fn() + 'static;

        /// Run all tests
        pub fn run(&self) -> TestReport;

        /// Run tests matching filter
        pub fn run_filtered(&self, filter: &str) -> TestReport;

        /// Run tests with specific tags
        pub fn run_tagged(&self, tags: &[&str]) -> TestReport;

        /// Run tests in parallel
        pub fn run_parallel(&self, threads: usize) -> TestReport;
    }

    #[derive(Debug)]
    pub struct TestReport {
        pub total: usize,
        pub passed: usize,
        pub failed: usize,
        pub skipped: usize,
        pub duration: std::time::Duration,
        pub results: Vec<(String, TestOutcome)>,
    }

    /// Assertions
    pub mod assertions {
        pub fn assert_eq<T: PartialEq + std::fmt::Debug>(left: T, right: T) -> TestOutcome;
        pub fn assert_ne<T: PartialEq + std::fmt::Debug>(left: T, right: T) -> TestOutcome;
        pub fn assert_true(condition: bool) -> TestOutcome;
        pub fn assert_false(condition: bool) -> TestOutcome;
        pub fn assert_some<T>(option: Option<T>) -> TestOutcome;
        pub fn assert_none<T>(option: Option<T>) -> TestOutcome;
        pub fn assert_ok<T, E>(result: Result<T, E>) -> TestOutcome;
        pub fn assert_err<T, E>(result: Result<T, E>) -> TestOutcome;
        pub fn assert_panics<F: FnOnce()>(f: F) -> TestOutcome;
        pub fn assert_approx_eq(left: f64, right: f64, epsilon: f64) -> TestOutcome;
    }
}

pub mod mocking {
    /// Mock trait
    pub trait Mock: Sized {
        type Config;
        fn new() -> Self;
        fn configure(&mut self, config: Self::Config);
        fn verify(&self) -> bool;
    }

    /// Call recorder
    pub struct CallRecorder<A, R> {
        calls: std::cell::RefCell<Vec<A>>,
        return_values: std::cell::RefCell<Vec<R>>,
    }

    impl<A: Clone, R: Clone> CallRecorder<A, R> {
        pub fn new() -> Self;
        pub fn record(&self, args: A) -> Option<R>;
        pub fn with_return(&self, ret: R) -> &Self;
        pub fn with_returns(&self, rets: Vec<R>) -> &Self;
        pub fn calls(&self) -> Vec<A>;
        pub fn call_count(&self) -> usize;
        pub fn was_called_with(&self, expected: &A) -> bool
        where
            A: PartialEq;
    }

    /// Expectation builder
    pub struct Expectations<A, R> {
        expected_calls: Vec<(A, R)>,
    }

    impl<A: PartialEq, R: Clone> Expectations<A, R> {
        pub fn new() -> Self;
        pub fn expect(&mut self, args: A) -> &mut Self;
        pub fn returns(&mut self, ret: R) -> &mut Self;
        pub fn times(&mut self, n: usize) -> &mut Self;
        pub fn verify_in_order(&self, calls: &[A]) -> bool;
    }

    /// Spy (partial mock)
    pub struct Spy<T> {
        real: T,
        overrides: std::collections::HashMap<String, Box<dyn std::any::Any>>,
    }

    impl<T> Spy<T> {
        pub fn new(real: T) -> Self;
        pub fn stub<R: 'static>(&mut self, method: &str, return_value: R);
        pub fn call_real(&self) -> &T;
    }
}

pub mod fixtures {
    /// Test fixture
    pub trait Fixture: Sized {
        fn setup() -> Self;
        fn teardown(self);
    }

    /// Database test fixture
    pub struct DatabaseFixture {
        // Test database connection
    }

    impl Fixture for DatabaseFixture {
        fn setup() -> Self;
        fn teardown(self);
    }

    /// File system fixture
    pub struct TempDirFixture {
        path: std::path::PathBuf,
    }

    impl TempDirFixture {
        pub fn path(&self) -> &std::path::Path;
        pub fn create_file(&self, name: &str, contents: &str) -> std::path::PathBuf;
    }

    impl Fixture for TempDirFixture {
        fn setup() -> Self;
        fn teardown(self);
    }

    /// Parameterized tests
    pub fn parameterized<T, F>(params: &[T], test: F) -> Vec<TestOutcome>
    where
        T: Clone,
        F: Fn(T) -> TestOutcome;
}

pub mod dependency_injection {
    /// Service trait for DI
    pub trait Service: Send + Sync {
        fn name(&self) -> &str;
    }

    /// DI container
    pub struct Container {
        services: std::collections::HashMap<std::any::TypeId, Box<dyn std::any::Any + Send + Sync>>,
    }

    impl Container {
        pub fn new() -> Self;

        pub fn register<T: Service + 'static>(&mut self, service: T);
        pub fn register_singleton<T: Service + Clone + 'static>(&mut self, service: T);
        pub fn register_factory<T: Service + 'static, F>(&mut self, factory: F)
        where
            F: Fn() -> T + Send + Sync + 'static;

        pub fn resolve<T: 'static>(&self) -> Option<&T>;
        pub fn resolve_mut<T: 'static>(&mut self) -> Option<&mut T>;
    }

    /// Injectable attribute (conceptual)
    pub trait Injectable {
        fn inject(container: &Container) -> Self;
    }
}

pub mod ci_cd {
    /// Pipeline definition
    #[derive(Debug)]
    pub struct Pipeline {
        stages: Vec<Stage>,
    }

    #[derive(Debug)]
    pub struct Stage {
        name: String,
        jobs: Vec<Job>,
        depends_on: Vec<String>,
    }

    #[derive(Debug)]
    pub struct Job {
        name: String,
        steps: Vec<Step>,
        artifacts: Vec<String>,
        on_failure: OnFailure,
    }

    #[derive(Debug)]
    pub enum Step {
        Run(String),
        Checkout,
        Cache { key: String, paths: Vec<String> },
        Artifact { path: String },
    }

    #[derive(Debug)]
    pub enum OnFailure {
        Stop,
        Continue,
        Retry(usize),
    }

    impl Pipeline {
        pub fn new() -> Self;
        pub fn add_stage(&mut self, stage: Stage);
        pub fn validate(&self) -> Result<(), String>;
        pub fn to_yaml(&self) -> String;
        pub fn from_yaml(yaml: &str) -> Result<Self, String>;
    }

    /// Pipeline executor (simulation)
    pub struct PipelineExecutor {
        env: std::collections::HashMap<String, String>,
    }

    impl PipelineExecutor {
        pub fn new() -> Self;
        pub fn set_env(&mut self, key: &str, value: &str);
        pub fn execute(&self, pipeline: &Pipeline) -> PipelineResult;
    }

    #[derive(Debug)]
    pub struct PipelineResult {
        pub success: bool,
        pub stages: Vec<StageResult>,
        pub duration: std::time::Duration,
    }

    #[derive(Debug)]
    pub struct StageResult {
        pub name: String,
        pub success: bool,
        pub jobs: Vec<JobResult>,
    }

    #[derive(Debug)]
    pub struct JobResult {
        pub name: String,
        pub success: bool,
        pub output: String,
    }
}
```

### Python Implementation

```python
from typing import List, Dict, Callable, Optional, Any, TypeVar
from dataclasses import dataclass
import time

@dataclass
class TestOutcome:
    passed: bool
    message: str = ""

class TestRunner:
    def __init__(self):
        self.tests: List[tuple] = []
        self.setup: Optional[Callable] = None
        self.teardown: Optional[Callable] = None

    def add_test(self, name: str, test_fn: Callable) -> None: ...
    def run(self) -> 'TestReport': ...
    def run_filtered(self, filter: str) -> 'TestReport': ...
    def run_parallel(self, threads: int) -> 'TestReport': ...

@dataclass
class TestReport:
    total: int
    passed: int
    failed: int
    skipped: int
    duration: float

class CallRecorder:
    def __init__(self): ...
    def record(self, *args, **kwargs) -> Any: ...
    def with_return(self, ret: Any) -> 'CallRecorder': ...
    def calls(self) -> List: ...
    def call_count(self) -> int: ...

T = TypeVar('T')

class Fixture:
    @classmethod
    def setup(cls) -> 'Fixture': ...
    def teardown(self) -> None: ...

class Container:
    def __init__(self): ...
    def register(self, service_type: type, instance: Any) -> None: ...
    def resolve(self, service_type: type) -> Any: ...

class Pipeline:
    def __init__(self): ...
    def add_stage(self, stage: 'Stage') -> None: ...
    def validate(self) -> bool: ...
    def to_yaml(self) -> str: ...
```

## Test Cases

```rust
#[test]
fn test_test_runner() {
    let mut runner = TestRunner::new();

    runner.add_test("passing_test", || TestOutcome::Passed);
    runner.add_test("failing_test", || TestOutcome::Failed("Expected failure".into()));

    let report = runner.run();

    assert_eq!(report.total, 2);
    assert_eq!(report.passed, 1);
    assert_eq!(report.failed, 1);
}

#[test]
fn test_setup_teardown() {
    let setup_called = std::sync::Arc::new(std::sync::atomic::AtomicBool::new(false));
    let teardown_called = std::sync::Arc::new(std::sync::atomic::AtomicBool::new(false));

    let s = setup_called.clone();
    let t = teardown_called.clone();

    let mut runner = TestRunner::new()
        .with_setup(move || { s.store(true, std::sync::atomic::Ordering::SeqCst); })
        .with_teardown(move || { t.store(true, std::sync::atomic::Ordering::SeqCst); });

    runner.add_test("test", || TestOutcome::Passed);
    runner.run();

    assert!(setup_called.load(std::sync::atomic::Ordering::SeqCst));
    assert!(teardown_called.load(std::sync::atomic::Ordering::SeqCst));
}

#[test]
fn test_filtered_run() {
    let mut runner = TestRunner::new();

    runner.add_test("unit_add", || TestOutcome::Passed);
    runner.add_test("unit_sub", || TestOutcome::Passed);
    runner.add_test("integration_db", || TestOutcome::Passed);

    let report = runner.run_filtered("unit_");

    assert_eq!(report.total, 2);
}

#[test]
fn test_call_recorder() {
    let recorder: CallRecorder<i32, String> = CallRecorder::new();
    recorder.with_return("hello".to_string());

    let result = recorder.record(42);

    assert_eq!(result, Some("hello".to_string()));
    assert_eq!(recorder.call_count(), 1);
    assert!(recorder.was_called_with(&42));
}

#[test]
fn test_multiple_returns() {
    let recorder: CallRecorder<(), i32> = CallRecorder::new();
    recorder.with_returns(vec![1, 2, 3]);

    assert_eq!(recorder.record(()), Some(1));
    assert_eq!(recorder.record(()), Some(2));
    assert_eq!(recorder.record(()), Some(3));
    assert_eq!(recorder.record(()), None);
}

#[test]
fn test_temp_dir_fixture() {
    let fixture = TempDirFixture::setup();

    let file_path = fixture.create_file("test.txt", "Hello, World!");

    assert!(file_path.exists());
    assert_eq!(std::fs::read_to_string(&file_path).unwrap(), "Hello, World!");

    fixture.teardown();
    // Directory should be cleaned up
}

#[test]
fn test_parameterized() {
    let params = vec![
        (2, 3, 5),
        (0, 0, 0),
        (-1, 1, 0),
    ];

    let results = fixtures::parameterized(&params, |(a, b, expected)| {
        if a + b == expected {
            TestOutcome::Passed
        } else {
            TestOutcome::Failed(format!("{} + {} != {}", a, b, expected))
        }
    });

    assert!(results.iter().all(|r| matches!(r, TestOutcome::Passed)));
}

#[test]
fn test_di_container() {
    struct Logger;
    impl Service for Logger {
        fn name(&self) -> &str { "Logger" }
    }

    struct Database;
    impl Service for Database {
        fn name(&self) -> &str { "Database" }
    }

    let mut container = Container::new();
    container.register(Logger);
    container.register(Database);

    let logger = container.resolve::<Logger>();
    assert!(logger.is_some());
    assert_eq!(logger.unwrap().name(), "Logger");
}

#[test]
fn test_pipeline_validation() {
    let mut pipeline = Pipeline::new();

    pipeline.add_stage(Stage {
        name: "build".to_string(),
        jobs: vec![Job {
            name: "compile".to_string(),
            steps: vec![Step::Run("cargo build".to_string())],
            artifacts: vec!["target/".to_string()],
            on_failure: OnFailure::Stop,
        }],
        depends_on: vec![],
    });

    pipeline.add_stage(Stage {
        name: "test".to_string(),
        jobs: vec![Job {
            name: "unit_tests".to_string(),
            steps: vec![Step::Run("cargo test".to_string())],
            artifacts: vec![],
            on_failure: OnFailure::Stop,
        }],
        depends_on: vec!["build".to_string()],
    });

    assert!(pipeline.validate().is_ok());
}

#[test]
fn test_pipeline_yaml() {
    let mut pipeline = Pipeline::new();
    // Add stages...

    let yaml = pipeline.to_yaml();
    let restored = Pipeline::from_yaml(&yaml).unwrap();

    // Should be equivalent
}

#[test]
fn test_assertions() {
    assert!(matches!(assertions::assert_eq(1, 1), TestOutcome::Passed));
    assert!(matches!(assertions::assert_eq(1, 2), TestOutcome::Failed(_)));

    assert!(matches!(assertions::assert_some(Some(42)), TestOutcome::Passed));
    assert!(matches!(assertions::assert_none::<i32>(None), TestOutcome::Passed));

    assert!(matches!(
        assertions::assert_panics(|| panic!("test")),
        TestOutcome::Passed
    ));
}

#[test]
fn test_mock_expectations() {
    let mut expectations = Expectations::<String, i32>::new();
    expectations.expect("get_value".into()).returns(42);

    assert!(expectations.verify_in_order(&["get_value".into()]));
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Test runner | 15 |
| Assertions | 10 |
| Mocking framework | 20 |
| Fixtures | 15 |
| Dependency injection | 15 |
| Pipeline definition | 15 |
| Parameterized tests | 5 |
| Edge cases | 5 |
| **Total** | **100** |
