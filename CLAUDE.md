# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Development Commands

### Building
```bash
# Build the main cockroach binary
./dev build

# Build short version (without UI)
./dev build short

# Build with race detection  
./dev build --race

# Build specific targets
./dev build pkg/sql/parser
```

### Testing
```bash
# Run all tests in a package
./dev test pkg/sql

# Run specific test with filter
./dev test pkg/sql/parser -f=TestParse

# Run tests with race detection
./dev test pkg/sql --race

# Run tests repeatedly (stress testing)
./dev test pkg/sql --stress --count 1000

# Run short tests only
./dev test pkg/sql --short

# Run tests with increased verbosity
./dev test pkg/sql -v

# Run single test multiple times
./dev test pkg/sql --count 5

# Run logic tests
./dev testlogic
./dev testlogic ccl
./dev testlogic --files='prepare|fk'
```

### Code Quality
```bash
# Run linters
./dev lint

# Run short/fast linting
./dev lint --short

# Generate code (stringer, protos, etc.)
./dev generate go

# Generate Bazel BUILD files
./dev generate bazel

# Check system prerequisites
./dev doctor
```

### Key Build Notes
- Use `./dev` instead of `make` for all development tasks
- The build system uses Bazel underneath
- `./dev doctor` checks if your machine is ready to build
- Pass `--` to send arguments directly to bazel: `./dev build -- --verbose_failures`

## Architecture Overview

CockroachDB is a distributed SQL database built on a transactional key-value store with a layered architecture:

### Core Components

**SQL Layer (`pkg/sql/`)**
- Query parsing, planning, and optimization using cost-based optimizer
- DistSQL for distributed query execution  
- Vectorized execution engine for performance
- Schema management and DDL operations

**Key-Value Layer (`pkg/kv/`)**
- Distributed transactional key-value API
- Transaction coordination with serializable isolation
- Batch processing and request routing (DistSender)
- Main entry point: `kv.DB` and `kv.Txn` interfaces

**Storage Engine (`pkg/storage/`)**
- MVCC (Multi-Version Concurrency Control) with timestamps
- Write intents for uncommitted transactions
- Engine abstraction over Pebble/RocksDB
- Versioned storage for time-travel queries

**KV Server (`pkg/kvserver/`)**
- Range-based data partitioning (~512MB ranges)
- Raft consensus for each range replica group  
- Range splitting, merging, and rebalancing
- Leaseholder coordination and concurrency control

**Server Components (`pkg/server/`)**
- Node management and RPC services
- Administrative REST APIs
- Multi-tenancy support
- Connection handling and session management

**Supporting Systems**
- **Gossip Network (`pkg/gossip/`)**: Peer-to-peer cluster metadata sharing
- **Protocol Definitions (`pkg/roachpb/`)**: Core data structures and RPC types
- **CLI Tools (`pkg/cli/`)**: Administrative and operational commands

### Key Design Patterns

**Layered Architecture**: Clear separation between SQL → KV → Storage layers with well-defined interfaces

**Distributed Consensus**: Raft algorithm per range for strong consistency guarantees

**MVCC Storage**: Lock-free reads with versioned data and write intents

**Range-Based Partitioning**: Automatic data splitting and load balancing

**Cost-Based Optimization**: SQL optimizer with statistics and memoization

## Testing Patterns

CockroachDB uses comprehensive testing strategies:

- **Unit Tests**: Standard Go testing throughout codebase
- **Integration Tests**: Multi-node test clusters using `testutils`
- **Logic Tests**: SQL correctness testing with `./dev testlogic`
- **Data-Driven Tests**: Especially in optimizer (`pkg/sql/opt/`)
- **Stress Testing**: Repeated execution to find race conditions
- **Acceptance Tests**: End-to-end behavioral testing

Common test patterns:
- Table-driven tests for multiple scenarios
- Mock interfaces for isolated testing  
- Test clusters for distributed scenarios
- Benchmarking with `testing/benchmark`

## Key Files to Understand

When working on specific areas, start with these documentation files:
- `pkg/sql/doc.go` - SQL layer architecture
- `pkg/kv/doc.go` - KV layer API and examples  
- `pkg/storage/doc.go` - MVCC and storage details
- `pkg/kvserver/doc.go` - Distributed storage server
- `pkg/gossip/doc.go` - Cluster communication protocol

## Development Environment

- **Go Version**: 1.23.7 (see go.mod)
- **Build System**: Bazel with custom `./dev` wrapper
- **Language**: Primarily Go with some C/C++ dependencies
- **Testing**: Comprehensive test suite with multiple frameworks
- **Linting**: Custom linting rules enforced via `./dev lint`

## Working with CockroachDB

When making changes:
1. Run `./dev doctor` to verify development environment
2. Use `./dev build` to compile and `./dev test pkg/...` for testing
3. Run `./dev lint` before submitting changes
4. Use `./dev generate` if modifying protobuf or generated code
5. Leverage data-driven tests for SQL changes
6. Test distributed scenarios with test clusters when needed

The codebase emphasizes maintainability through clear layering, comprehensive testing, and strong separation of concerns between distributed systems components.