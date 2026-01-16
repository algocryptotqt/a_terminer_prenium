# [Module 2.2] - Exercise 14: Async Process Management

## Metadonnees

```yaml
module: "2.2 - Process Management"
exercise: "ex14"
title: "Async Process Management"
difficulty: avance
estimated_time: "4 heures"
prerequisite_exercises: ["ex09", "ex10"]
concepts_requis: ["async", "processes", "tokio"]
score_qualite: 98
```

---

## Concepts Couverts

### 2.2.35: Async Process Management (10 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.35.a | `tokio::process::Command` | Async Command |
| 2.2.35.b | `child.wait().await` | Async wait |
| 2.2.35.c | `child.stdout` | `AsyncRead` |
| 2.2.35.d | `child.stdin` | `AsyncWrite` |
| 2.2.35.e | `tokio::io::copy()` | Async pipe |
| 2.2.35.f | Process pool | Concurrent execution |
| 2.2.35.g | `futures::stream` | Process multiple |
| 2.2.35.h | Timeout | `tokio::time::timeout()` |
| 2.2.35.i | Graceful shutdown | Signal + timeout |
| 2.2.35.j | Process supervision | Restart on failure |

---

## Partie 1: Tokio Process Command (2.2.35.a, b)

### Exercice 1.1: Basic Async Process

```rust
//! Async process management with Tokio (2.2.35.a)
//!
//! tokio::process::Command provides async versions of
//! std::process operations.

use tokio::process::Command;
use std::process::Stdio;

/// Basic async command execution (2.2.35.a)
async fn run_command_async() -> Result<(), Box<dyn std::error::Error>> {
    println!("=== tokio::process::Command (2.2.35.a) ===\n");

    // Create async command
    let mut child = Command::new("echo")
        .arg("Hello from async process!")
        .spawn()?;

    // Async wait for completion (2.2.35.b)
    let status = child.wait().await?;
    println!("Exit status: {}", status);

    Ok(())
}

/// Run command and capture output
async fn capture_output() -> Result<(), Box<dyn std::error::Error>> {
    println!("\n=== Capturing Output ===\n");

    // output() is async and waits for completion
    let output = Command::new("ls")
        .arg("-la")
        .arg("/tmp")
        .output()
        .await?;

    println!("Status: {}", output.status);
    println!("Stdout:\n{}", String::from_utf8_lossy(&output.stdout));

    if !output.stderr.is_empty() {
        eprintln!("Stderr:\n{}", String::from_utf8_lossy(&output.stderr));
    }

    Ok(())
}

/// Wait with status checking (2.2.35.b)
async fn wait_with_check() -> Result<(), Box<dyn std::error::Error>> {
    println!("\n=== Async Wait (2.2.35.b) ===\n");

    let mut child = Command::new("sh")
        .arg("-c")
        .arg("sleep 1 && echo Done!")
        .stdout(Stdio::piped())
        .spawn()?;

    println!("Process spawned, waiting...");

    // Async wait - doesn't block the runtime
    let status = child.wait().await?;

    if status.success() {
        println!("Process completed successfully");
    } else {
        println!("Process failed with code: {:?}", status.code());
    }

    Ok(())
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    run_command_async().await?;
    capture_output().await?;
    wait_with_check().await?;

    Ok(())
}
```

---

## Partie 2: Async I/O Streams (2.2.35.c, d, e)

### Exercice 2.1: Reading from Child Stdout (2.2.35.c)

```rust
use tokio::process::Command;
use tokio::io::{AsyncBufReadExt, BufReader};
use std::process::Stdio;

/// Read stdout as AsyncRead (2.2.35.c)
async fn read_stdout_async() -> Result<(), Box<dyn std::error::Error>> {
    println!("=== AsyncRead from stdout (2.2.35.c) ===\n");

    let mut child = Command::new("sh")
        .arg("-c")
        .arg("for i in 1 2 3 4 5; do echo \"Line $i\"; sleep 0.5; done")
        .stdout(Stdio::piped())  // Capture stdout
        .spawn()?;

    // Take ownership of stdout (2.2.35.c)
    let stdout = child.stdout.take()
        .expect("Failed to capture stdout");

    // Wrap in BufReader for line-by-line reading
    let mut reader = BufReader::new(stdout).lines();

    // Read lines asynchronously
    while let Some(line) = reader.next_line().await? {
        println!("Received: {}", line);
    }

    // Wait for process to complete
    let status = child.wait().await?;
    println!("\nProcess exited: {}", status);

    Ok(())
}

/// Read stdout with streaming
async fn stream_stdout() -> Result<(), Box<dyn std::error::Error>> {
    use tokio::io::AsyncReadExt;

    println!("\n=== Streaming stdout ===\n");

    let mut child = Command::new("dd")
        .args(["if=/dev/zero", "bs=1024", "count=10"])
        .stdout(Stdio::piped())
        .stderr(Stdio::null())
        .spawn()?;

    let mut stdout = child.stdout.take().unwrap();
    let mut buffer = [0u8; 1024];
    let mut total = 0;

    loop {
        let n = stdout.read(&mut buffer).await?;
        if n == 0 {
            break;
        }
        total += n;
        println!("Read {} bytes (total: {})", n, total);
    }

    child.wait().await?;
    println!("Total bytes read: {}", total);

    Ok(())
}
```

### Exercice 2.2: Writing to Child Stdin (2.2.35.d)

```rust
use tokio::process::Command;
use tokio::io::AsyncWriteExt;
use std::process::Stdio;

/// Write to stdin as AsyncWrite (2.2.35.d)
async fn write_stdin_async() -> Result<(), Box<dyn std::error::Error>> {
    println!("\n=== AsyncWrite to stdin (2.2.35.d) ===\n");

    let mut child = Command::new("cat")
        .stdin(Stdio::piped())   // Accept stdin
        .stdout(Stdio::piped())  // Capture stdout
        .spawn()?;

    // Get stdin handle (2.2.35.d)
    let mut stdin = child.stdin.take()
        .expect("Failed to open stdin");

    // Write data asynchronously
    stdin.write_all(b"Hello from Rust!\n").await?;
    stdin.write_all(b"This is async writing.\n").await?;
    stdin.write_all(b"Third line here.\n").await?;

    // Must drop stdin to signal EOF
    drop(stdin);

    // Read output
    let output = child.wait_with_output().await?;
    println!("Cat output:\n{}", String::from_utf8_lossy(&output.stdout));

    Ok(())
}

/// Interactive communication with process
async fn interactive_process() -> Result<(), Box<dyn std::error::Error>> {
    use tokio::io::AsyncBufReadExt;

    println!("\n=== Interactive Process ===\n");

    let mut child = Command::new("bc")
        .arg("-q")  // Quiet mode
        .stdin(Stdio::piped())
        .stdout(Stdio::piped())
        .spawn()?;

    let mut stdin = child.stdin.take().unwrap();
    let stdout = child.stdout.take().unwrap();
    let mut reader = tokio::io::BufReader::new(stdout).lines();

    // Send calculations
    let calculations = ["2 + 2", "10 * 5", "100 / 4", "2 ^ 10"];

    for calc in &calculations {
        // Write calculation
        stdin.write_all(format!("{}\n", calc).as_bytes()).await?;
        stdin.flush().await?;

        // Read result
        if let Some(result) = reader.next_line().await? {
            println!("{} = {}", calc, result);
        }
    }

    // Send quit command
    stdin.write_all(b"quit\n").await?;
    drop(stdin);

    child.wait().await?;
    Ok(())
}
```

### Exercice 2.3: Async Pipe Between Processes (2.2.35.e)

```rust
use tokio::process::Command;
use tokio::io;
use std::process::Stdio;

/// Async pipe using tokio::io::copy() (2.2.35.e)
async fn async_pipe() -> Result<(), Box<dyn std::error::Error>> {
    println!("\n=== tokio::io::copy() (2.2.35.e) ===\n");

    // First process: generate data
    let mut producer = Command::new("seq")
        .arg("1")
        .arg("100")
        .stdout(Stdio::piped())
        .spawn()?;

    // Second process: filter data
    let mut consumer = Command::new("grep")
        .arg("5")  // Lines containing '5'
        .stdin(Stdio::piped())
        .stdout(Stdio::piped())
        .spawn()?;

    // Get handles
    let producer_stdout = producer.stdout.take().unwrap();
    let mut consumer_stdin = consumer.stdin.take().unwrap();

    // Async copy from producer to consumer (2.2.35.e)
    let copied = io::copy(&mut tokio::io::BufReader::new(producer_stdout), &mut consumer_stdin).await?;
    println!("Copied {} bytes through pipe", copied);

    // Close stdin to signal EOF
    drop(consumer_stdin);

    // Get final output
    let output = consumer.wait_with_output().await?;
    println!("Filtered output:\n{}", String::from_utf8_lossy(&output.stdout));

    producer.wait().await?;

    Ok(())
}

/// Pipeline of multiple processes
async fn multi_process_pipeline() -> Result<(), Box<dyn std::error::Error>> {
    println!("\n=== Multi-Process Pipeline ===\n");

    // Create pipeline: ls | grep | wc
    let mut ls = Command::new("ls")
        .arg("-la")
        .arg("/usr/bin")
        .stdout(Stdio::piped())
        .spawn()?;

    let mut grep = Command::new("grep")
        .arg("^-")  // Regular files only
        .stdin(Stdio::piped())
        .stdout(Stdio::piped())
        .spawn()?;

    let mut wc = Command::new("wc")
        .arg("-l")
        .stdin(Stdio::piped())
        .stdout(Stdio::piped())
        .spawn()?;

    // Connect ls -> grep
    let ls_out = ls.stdout.take().unwrap();
    let mut grep_in = grep.stdin.take().unwrap();

    let copy1 = tokio::spawn(async move {
        io::copy(&mut tokio::io::BufReader::new(ls_out), &mut grep_in).await
    });

    // Connect grep -> wc
    let grep_out = grep.stdout.take().unwrap();
    let mut wc_in = wc.stdin.take().unwrap();

    let copy2 = tokio::spawn(async move {
        io::copy(&mut tokio::io::BufReader::new(grep_out), &mut wc_in).await
    });

    // Wait for copies
    copy1.await??;
    copy2.await??;

    // Get final count
    let output = wc.wait_with_output().await?;
    let count = String::from_utf8_lossy(&output.stdout);
    println!("Regular files in /usr/bin: {}", count.trim());

    ls.wait().await?;
    grep.wait().await?;

    Ok(())
}
```

---

## Partie 3: Process Pool & Concurrency (2.2.35.f, g)

### Exercice 3.1: Concurrent Process Execution (2.2.35.f)

```rust
use tokio::process::Command;
use futures::stream::{self, StreamExt};
use std::time::Instant;

/// Process pool for concurrent execution (2.2.35.f)
pub struct ProcessPool {
    max_concurrent: usize,
}

impl ProcessPool {
    pub fn new(max_concurrent: usize) -> Self {
        Self { max_concurrent }
    }

    /// Run multiple commands concurrently (2.2.35.f)
    pub async fn run_commands(
        &self,
        commands: Vec<Vec<String>>,
    ) -> Vec<Result<std::process::Output, std::io::Error>> {
        // Convert to stream and process with concurrency limit (2.2.35.g)
        stream::iter(commands)
            .map(|args| async move {
                let mut cmd = Command::new(&args[0]);
                if args.len() > 1 {
                    cmd.args(&args[1..]);
                }
                cmd.output().await
            })
            .buffer_unordered(self.max_concurrent)
            .collect()
            .await
    }
}

/// Demonstrate process pool (2.2.35.f)
async fn demonstrate_process_pool() -> Result<(), Box<dyn std::error::Error>> {
    println!("\n=== Process Pool (2.2.35.f) ===\n");

    let pool = ProcessPool::new(4);  // Max 4 concurrent

    // Create test commands
    let commands: Vec<Vec<String>> = (1..=10)
        .map(|i| vec![
            "sh".to_string(),
            "-c".to_string(),
            format!("sleep 0.5 && echo 'Task {} done'", i),
        ])
        .collect();

    let start = Instant::now();

    let results = pool.run_commands(commands).await;

    let elapsed = start.elapsed();

    // Process results
    for (i, result) in results.iter().enumerate() {
        match result {
            Ok(output) => {
                let stdout = String::from_utf8_lossy(&output.stdout);
                println!("Task {}: {}", i + 1, stdout.trim());
            }
            Err(e) => println!("Task {} failed: {}", i + 1, e),
        }
    }

    println!("\nTotal time: {:?} (parallel)", elapsed);
    println!("Sequential would be ~5s");

    Ok(())
}

/// Using futures::stream for process management (2.2.35.g)
async fn stream_processes() -> Result<(), Box<dyn std::error::Error>> {
    use futures::stream::FuturesUnordered;

    println!("\n=== futures::stream (2.2.35.g) ===\n");

    let mut futures = FuturesUnordered::new();

    // Spawn multiple processes
    for i in 1..=5 {
        let future = async move {
            let output = Command::new("sh")
                .arg("-c")
                .arg(format!("sleep {} && echo 'Process {} (slept {}s)'", i % 3, i, i % 3))
                .output()
                .await;
            (i, output)
        };
        futures.push(future);
    }

    // Collect results as they complete
    while let Some((id, result)) = futures.next().await {
        match result {
            Ok(output) => {
                println!("Completed: {}", String::from_utf8_lossy(&output.stdout).trim());
            }
            Err(e) => println!("Process {} failed: {}", id, e),
        }
    }

    Ok(())
}
```

---

## Partie 4: Timeout & Graceful Shutdown (2.2.35.h, i)

### Exercice 4.1: Process Timeout (2.2.35.h)

```rust
use tokio::process::Command;
use tokio::time::{timeout, Duration};
use std::process::Stdio;

/// Run process with timeout (2.2.35.h)
async fn run_with_timeout(
    cmd: &str,
    args: &[&str],
    timeout_duration: Duration,
) -> Result<std::process::Output, ProcessError> {
    println!("\n=== Timeout (2.2.35.h) ===\n");

    let mut child = Command::new(cmd)
        .args(args)
        .stdout(Stdio::piped())
        .stderr(Stdio::piped())
        .spawn()?;

    // Wait with timeout (2.2.35.h)
    match timeout(timeout_duration, child.wait_with_output()).await {
        Ok(result) => {
            println!("Process completed within timeout");
            Ok(result?)
        }
        Err(_) => {
            println!("Timeout! Killing process...");
            // Kill the process
            child.kill().await?;
            Err(ProcessError::Timeout)
        }
    }
}

#[derive(Debug)]
pub enum ProcessError {
    Io(std::io::Error),
    Timeout,
    Signal,
}

impl From<std::io::Error> for ProcessError {
    fn from(e: std::io::Error) -> Self {
        ProcessError::Io(e)
    }
}

/// Demonstrate timeout
async fn demonstrate_timeout() -> Result<(), Box<dyn std::error::Error>> {
    // Command that finishes in time
    match run_with_timeout("sleep", &["1"], Duration::from_secs(5)).await {
        Ok(_) => println!("Short task: Success"),
        Err(e) => println!("Short task: {:?}", e),
    }

    // Command that times out
    match run_with_timeout("sleep", &["10"], Duration::from_secs(2)).await {
        Ok(_) => println!("Long task: Success"),
        Err(e) => println!("Long task: {:?}", e),
    }

    Ok(())
}
```

### Exercice 4.2: Graceful Shutdown (2.2.35.i)

```rust
use tokio::process::{Command, Child};
use tokio::time::{timeout, Duration};
use tokio::signal;
use nix::sys::signal::{kill, Signal};
use nix::unistd::Pid;
use std::process::Stdio;

/// Graceful process shutdown (2.2.35.i)
/// First sends SIGTERM, then SIGKILL if needed
pub struct GracefulProcess {
    child: Child,
    shutdown_timeout: Duration,
}

impl GracefulProcess {
    pub async fn spawn(cmd: &str, args: &[&str]) -> std::io::Result<Self> {
        let child = Command::new(cmd)
            .args(args)
            .stdout(Stdio::piped())
            .stderr(Stdio::piped())
            .spawn()?;

        Ok(Self {
            child,
            shutdown_timeout: Duration::from_secs(5),
        })
    }

    pub fn with_timeout(mut self, timeout: Duration) -> Self {
        self.shutdown_timeout = timeout;
        self
    }

    /// Graceful shutdown: SIGTERM -> wait -> SIGKILL (2.2.35.i)
    pub async fn shutdown(&mut self) -> Result<(), ProcessError> {
        if let Some(pid) = self.child.id() {
            println!("Sending SIGTERM to process {}", pid);

            // Send SIGTERM
            kill(Pid::from_raw(pid as i32), Signal::SIGTERM)
                .map_err(|_| ProcessError::Signal)?;

            // Wait for graceful exit with timeout
            match timeout(self.shutdown_timeout, self.child.wait()).await {
                Ok(Ok(status)) => {
                    println!("Process exited gracefully: {}", status);
                    return Ok(());
                }
                Ok(Err(e)) => {
                    return Err(ProcessError::Io(e));
                }
                Err(_) => {
                    println!("Timeout waiting for graceful exit, sending SIGKILL");
                }
            }

            // Force kill
            self.child.kill().await?;
            println!("Process killed");
        }

        Ok(())
    }

    /// Wait for completion or signal
    pub async fn wait_with_signal(&mut self) -> Result<std::process::ExitStatus, ProcessError> {
        tokio::select! {
            status = self.child.wait() => {
                Ok(status?)
            }
            _ = signal::ctrl_c() => {
                println!("\nReceived Ctrl+C, shutting down...");
                self.shutdown().await?;
                Err(ProcessError::Signal)
            }
        }
    }
}

/// Demonstrate graceful shutdown (2.2.35.i)
async fn demonstrate_graceful_shutdown() -> Result<(), Box<dyn std::error::Error>> {
    println!("\n=== Graceful Shutdown (2.2.35.i) ===\n");

    // Spawn long-running process
    let mut process = GracefulProcess::spawn(
        "sh",
        &["-c", "trap 'echo Received SIGTERM; exit 0' TERM; while true; do sleep 1; done"],
    ).await?
    .with_timeout(Duration::from_secs(3));

    // Simulate some work
    tokio::time::sleep(Duration::from_secs(2)).await;

    // Graceful shutdown
    process.shutdown().await?;

    Ok(())
}

/// Application with signal handling
pub struct Application {
    processes: Vec<GracefulProcess>,
}

impl Application {
    pub fn new() -> Self {
        Self { processes: Vec::new() }
    }

    pub async fn spawn(&mut self, cmd: &str, args: &[&str]) -> std::io::Result<()> {
        let process = GracefulProcess::spawn(cmd, args).await?;
        self.processes.push(process);
        Ok(())
    }

    /// Shutdown all processes gracefully
    pub async fn shutdown_all(&mut self) {
        println!("Shutting down {} processes...", self.processes.len());

        for process in &mut self.processes {
            if let Err(e) = process.shutdown().await {
                eprintln!("Shutdown error: {:?}", e);
            }
        }

        self.processes.clear();
    }
}
```

---

## Partie 5: Process Supervision (2.2.35.j)

### Exercice 5.1: Process Supervisor

```rust
use tokio::process::Command;
use tokio::time::{Duration, sleep};
use std::process::Stdio;
use std::sync::Arc;
use tokio::sync::{watch, Mutex};

/// Process supervisor that restarts failed processes (2.2.35.j)
pub struct ProcessSupervisor {
    command: String,
    args: Vec<String>,
    max_restarts: u32,
    restart_delay: Duration,
    shutdown_rx: watch::Receiver<bool>,
}

impl ProcessSupervisor {
    pub fn new(
        command: String,
        args: Vec<String>,
        max_restarts: u32,
        shutdown_rx: watch::Receiver<bool>,
    ) -> Self {
        Self {
            command,
            args,
            max_restarts,
            restart_delay: Duration::from_secs(1),
            shutdown_rx,
        }
    }

    /// Run supervised process with automatic restart (2.2.35.j)
    pub async fn run(&mut self) -> SupervisorResult {
        let mut restart_count = 0;

        loop {
            // Check for shutdown signal
            if *self.shutdown_rx.borrow() {
                return SupervisorResult::Shutdown;
            }

            println!("Starting process: {} {:?}", self.command, self.args);

            let mut child = match Command::new(&self.command)
                .args(&self.args)
                .stdout(Stdio::inherit())
                .stderr(Stdio::inherit())
                .spawn()
            {
                Ok(child) => child,
                Err(e) => {
                    eprintln!("Failed to spawn: {}", e);
                    restart_count += 1;
                    if restart_count >= self.max_restarts {
                        return SupervisorResult::MaxRestartsExceeded;
                    }
                    sleep(self.restart_delay).await;
                    continue;
                }
            };

            // Wait for process or shutdown
            let status = tokio::select! {
                status = child.wait() => status,
                _ = self.shutdown_rx.changed() => {
                    println!("Shutdown requested, stopping process");
                    let _ = child.kill().await;
                    return SupervisorResult::Shutdown;
                }
            };

            match status {
                Ok(exit_status) => {
                    if exit_status.success() {
                        println!("Process exited successfully");
                        return SupervisorResult::Success;
                    } else {
                        println!("Process failed with: {}", exit_status);
                    }
                }
                Err(e) => {
                    eprintln!("Error waiting for process: {}", e);
                }
            }

            // Handle restart
            restart_count += 1;
            if restart_count >= self.max_restarts {
                return SupervisorResult::MaxRestartsExceeded;
            }

            println!("Restarting in {:?}... (attempt {}/{})",
                self.restart_delay, restart_count + 1, self.max_restarts);
            sleep(self.restart_delay).await;
        }
    }
}

#[derive(Debug)]
pub enum SupervisorResult {
    Success,
    MaxRestartsExceeded,
    Shutdown,
}

/// Multi-process supervisor
pub struct SupervisorGroup {
    supervisors: Vec<tokio::task::JoinHandle<SupervisorResult>>,
    shutdown_tx: watch::Sender<bool>,
}

impl SupervisorGroup {
    pub fn new() -> (Self, watch::Receiver<bool>) {
        let (tx, rx) = watch::channel(false);
        (
            Self {
                supervisors: Vec::new(),
                shutdown_tx: tx,
            },
            rx,
        )
    }

    pub fn add(&mut self, mut supervisor: ProcessSupervisor) {
        let handle = tokio::spawn(async move {
            supervisor.run().await
        });
        self.supervisors.push(handle);
    }

    pub async fn shutdown(&self) {
        println!("Sending shutdown signal to all supervisors");
        let _ = self.shutdown_tx.send(true);
    }

    pub async fn wait_all(self) -> Vec<SupervisorResult> {
        let mut results = Vec::new();
        for handle in self.supervisors {
            if let Ok(result) = handle.await {
                results.push(result);
            }
        }
        results
    }
}

/// Demonstrate process supervision (2.2.35.j)
async fn demonstrate_supervision() -> Result<(), Box<dyn std::error::Error>> {
    println!("\n=== Process Supervision (2.2.35.j) ===\n");

    let (mut group, shutdown_rx) = SupervisorGroup::new();

    // Add supervised process that fails intermittently
    group.add(ProcessSupervisor::new(
        "sh".to_string(),
        vec!["-c".to_string(), "echo 'Working...'; sleep 2; exit 1".to_string()],
        3,
        shutdown_rx.clone(),
    ));

    // Add supervised process that succeeds
    group.add(ProcessSupervisor::new(
        "sh".to_string(),
        vec!["-c".to_string(), "echo 'Quick task'; sleep 1".to_string()],
        3,
        shutdown_rx,
    ));

    // Wait some time then shutdown
    tokio::spawn(async move {
        sleep(Duration::from_secs(8)).await;
        group.shutdown().await;
    });

    // This would wait for all supervisors
    // let results = group.wait_all().await;

    Ok(())
}
```

---

## Partie 6: Complete Async Process Manager

### Exercice 6.1: Full Implementation

```rust
use tokio::process::Command;
use tokio::time::{timeout, Duration};
use futures::stream::{self, StreamExt};
use std::collections::HashMap;
use std::process::Stdio;

/// Complete async process manager
pub struct AsyncProcessManager {
    max_concurrent: usize,
    default_timeout: Duration,
    processes: HashMap<String, ProcessInfo>,
}

#[derive(Debug)]
pub struct ProcessInfo {
    pub id: String,
    pub command: String,
    pub status: ProcessStatus,
    pub output: Option<String>,
    pub error: Option<String>,
    pub duration_ms: u64,
}

#[derive(Debug, Clone)]
pub enum ProcessStatus {
    Pending,
    Running,
    Completed(i32),
    Failed(String),
    Timeout,
}

impl AsyncProcessManager {
    pub fn new(max_concurrent: usize) -> Self {
        Self {
            max_concurrent,
            default_timeout: Duration::from_secs(60),
            processes: HashMap::new(),
        }
    }

    pub fn with_timeout(mut self, timeout: Duration) -> Self {
        self.default_timeout = timeout;
        self
    }

    /// Execute multiple commands with full control
    pub async fn execute_batch(
        &mut self,
        commands: Vec<(String, String, Vec<String>)>,  // (id, cmd, args)
    ) -> Vec<ProcessInfo> {
        let timeout_duration = self.default_timeout;

        let results: Vec<ProcessInfo> = stream::iter(commands)
            .map(|(id, cmd, args)| {
                let timeout_dur = timeout_duration;
                async move {
                    let start = std::time::Instant::now();

                    let child = Command::new(&cmd)
                        .args(&args)
                        .stdout(Stdio::piped())
                        .stderr(Stdio::piped())
                        .spawn();

                    let (status, output, error) = match child {
                        Ok(mut child) => {
                            match timeout(timeout_dur, child.wait_with_output()).await {
                                Ok(Ok(output)) => {
                                    let code = output.status.code().unwrap_or(-1);
                                    (
                                        ProcessStatus::Completed(code),
                                        Some(String::from_utf8_lossy(&output.stdout).to_string()),
                                        Some(String::from_utf8_lossy(&output.stderr).to_string()),
                                    )
                                }
                                Ok(Err(e)) => {
                                    (ProcessStatus::Failed(e.to_string()), None, None)
                                }
                                Err(_) => {
                                    let _ = child.kill().await;
                                    (ProcessStatus::Timeout, None, None)
                                }
                            }
                        }
                        Err(e) => {
                            (ProcessStatus::Failed(e.to_string()), None, None)
                        }
                    };

                    ProcessInfo {
                        id,
                        command: format!("{} {:?}", cmd, args),
                        status,
                        output,
                        error,
                        duration_ms: start.elapsed().as_millis() as u64,
                    }
                }
            })
            .buffer_unordered(self.max_concurrent)
            .collect()
            .await;

        // Store results
        for info in &results {
            self.processes.insert(info.id.clone(), ProcessInfo {
                id: info.id.clone(),
                command: info.command.clone(),
                status: info.status.clone(),
                output: info.output.clone(),
                error: info.error.clone(),
                duration_ms: info.duration_ms,
            });
        }

        results
    }

    pub fn get_results(&self) -> &HashMap<String, ProcessInfo> {
        &self.processes
    }
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    println!("=== Async Process Manager Demo ===\n");

    let mut manager = AsyncProcessManager::new(4)
        .with_timeout(Duration::from_secs(10));

    let commands = vec![
        ("task1".to_string(), "echo".to_string(), vec!["Hello".to_string()]),
        ("task2".to_string(), "sleep".to_string(), vec!["1".to_string()]),
        ("task3".to_string(), "ls".to_string(), vec!["-la".to_string()]),
        ("task4".to_string(), "date".to_string(), vec![]),
    ];

    let results = manager.execute_batch(commands).await;

    for info in results {
        println!("\n{}: {:?}", info.id, info.status);
        println!("  Command: {}", info.command);
        println!("  Duration: {}ms", info.duration_ms);
        if let Some(output) = &info.output {
            if !output.is_empty() {
                println!("  Output: {}...", output.chars().take(50).collect::<String>());
            }
        }
    }

    Ok(())
}
```

---

## Criteres d'Evaluation

| Critere | Points |
|---------|--------|
| tokio::process::Command | 15 |
| Async wait | 10 |
| AsyncRead/AsyncWrite | 15 |
| tokio::io::copy | 10 |
| Process pool | 15 |
| futures::stream | 10 |
| Timeout handling | 10 |
| Graceful shutdown | 10 |
| Process supervision | 5 |
| **Total** | **100** |

---

## Ressources

- [tokio::process](https://docs.rs/tokio/latest/tokio/process/)
- [futures crate](https://docs.rs/futures/)
- [Tokio tutorial - Process](https://tokio.rs/tokio/tutorial/process)
