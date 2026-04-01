# pgpp Testing Roadmap

## Overview

Testing strategy for the pgpp PostgreSQL connection pool library. Tests are divided into
unit tests (no database required) and integration tests (require a running PostgreSQL instance).

**Framework:** GoogleTest v1.14.0 (fetched via FetchContent)
**Test totals:** 48 unit + 61 integration = 109 test cases

---

## Unit Tests (no database needed)

Test pure logic: type conversions, connection string building, struct construction,
pool state machine basics. Located in `tests/unit/`, numeric-prefixed for ordering.

### test_0.0_basic.cpp — Basic Sanity

| ID | Test | Requirement |
|---|---|---|
| UT-BASIC-001 | `TypesAreDefaultConstructible` — PgppPool and PgppConnection default-construct | — |
| UT-BASIC-002 | `PoolIsDestructibleWithoutInit` — destroy pool without initialize | REQ-PGPP-023 |
| UT-BASIC-003 | `ConnectionIsDestructibleWithoutOpen` — destroy connection without open | REQ-PGPP-011 |

### test_1.0_structs.cpp — Structs & OID Constants

| ID | Test | Requirement |
|---|---|---|
| UT-STMT-001 | `StatementHoldsFields` — Statement struct holds name, SQL, variables | REQ-PGPP-004, REQ-PGPP-005 |
| UT-STMT-002 | `StatementMultipleParams` — Statement with multiple OID parameters | REQ-PGPP-005 |
| UT-STMT-003 | `PgppConnectionInfoDefaults` — default field values | REQ-PGPP-001 |
| UT-OID-001 | `MatchPostgreSQLCatalog` — pg:: OID constants match documented values | REQ-PGPP-006 |

### test_2.0_conversions.cpp — Type Conversions

| ID | Test | Requirement |
|---|---|---|
| UT-CONV-001 | `StringReturnsRawValue` | REQ-PGPP-009 |
| UT-CONV-002 | `IntParsesCorrectly` | REQ-PGPP-009 |
| UT-CONV-003 | `Int16ParsesCorrectly` | REQ-PGPP-009 |
| UT-CONV-004 | `Int64ParsesCorrectly` | REQ-PGPP-009 |
| UT-CONV-005 | `Uint32ParsesCorrectly` | REQ-PGPP-009 |
| UT-CONV-006 | `DoubleParsesCorrectly` | REQ-PGPP-009 |
| UT-CONV-007 | `FloatParsesCorrectly` | REQ-PGPP-009 |
| UT-CONV-008 | `BoolTrueValues` — true for "t", "T", "1" | REQ-PGPP-009 |
| UT-CONV-009 | `BoolFalseValues` — false for "f", "F", "0" | REQ-PGPP-009 |
| UT-CONV-010 | `IntThrowsOnEmptyString` | REQ-PGPP-009 |
| UT-CONV-011 | `IntThrowsOnNonNumeric` | REQ-PGPP-009 |
| UT-CONV-012 | `DoubleThrowsOnEmptyString` | REQ-PGPP-009 |
| UT-CONV-013 | `Uint32ParsesMaxValue` | REQ-PGPP-009 |
| UT-CONV-014 | `Int64ParsesMinValue` | REQ-PGPP-009 |
| UT-CONV-015 | `StringUtf8Multibyte` | REQ-PGPP-009 |
| UT-CONV-016 | `IntOverflowThrows` | REQ-PGPP-009 |
| UT-CONV-017 | `IntUnderflowThrows` | REQ-PGPP-009 |
| UT-CONV-018 | `Int16Truncation` | REQ-PGPP-009 |
| UT-CONV-019 | `Int64MaxValue` | REQ-PGPP-009 |
| UT-CONV-020 | `FloatScientificNotation` | REQ-PGPP-009 |
| UT-CONV-021 | `DoubleInfinity` | REQ-PGPP-009 |
| UT-CONV-022 | `DoubleNaN` | REQ-PGPP-009 |
| UT-CONV-023 | `BoolFirstCharLogic` | REQ-PGPP-009 |

### test_2.1_connection_string.cpp — Connection String Builder

Uses `PgppPoolTest` fixture (friend access to `buildConnectionString`).

| ID | Test | Requirement |
|---|---|---|
| UT-CONN-001 | `BuildConnectionStringAllFields` — all fields produce valid libpq string | REQ-PGPP-001, REQ-PGPP-002 |
| UT-CONN-002 | `BuildConnectionStringEmptyDbname` — empty dbname returns empty string | REQ-PGPP-001 |
| UT-CONN-003 | `BuildConnectionStringEscapesSpecialChars` — escapes `'` and `\` | REQ-PGPP-002 |
| UT-CONN-004 | `BuildConnectionStringNoUser` — password omitted when no user | REQ-PGPP-003 |
| UT-CONN-005 | `BuildConnectionStringUserNoPassword` — user without password | REQ-PGPP-003 |
| UT-CONN-006 | `BuildConnectionStringPortZeroOmitted` — port=0 omitted from string | REQ-PGPP-001 |
| UT-CONN-007 | `BuildConnectionStringOptionsIncluded` | REQ-PGPP-001 |
| UT-CONN-008 | `BuildConnectionStringSslmodeIncluded` | REQ-PGPP-001 |
| UT-CONN-009 | `BuildConnectionStringMixedSpecialChars` | REQ-PGPP-002 |
| UT-CONN-010 | `BuildConnectionStringUsernameWithSpaces` | REQ-PGPP-002 |
| UT-CONN-011 | `BuildConnectionStringPortMax` — uint16 max | REQ-PGPP-001 |
| UT-CONN-012 | `BuildConnectionStringPortMin` — port=1 | REQ-PGPP-001 |
| UT-CONN-013 | `BuildConnectionStringMinimalFields` — dbname only | REQ-PGPP-001 |
| UT-CONN-014 | `BuildConnectionStringUnicodeDbname` | REQ-PGPP-002 |

### test_3.0_pool_state.cpp — Pool State Machine

| ID | Test | Requirement |
|---|---|---|
| UT-POOL-001 | `DefaultPoolSize` — hardware_concurrency or 16 | REQ-PGPP-019 |
| UT-POOL-002 | `InitializeFailsWithBadConnection` | REQ-PGPP-019 |
| UT-POOL-003 | `DoubleShutdownSafe` | REQ-PGPP-020 |
| UT-POOL-004 | `UninitializedPoolOperations` — exec/query don't crash | REQ-PGPP-028, REQ-PGPP-029 |
| UT-POOL-005 | `ShutdownThenReinitialize` | REQ-PGPP-020, REQ-PGPP-019 |
| UT-POOL-006 | `EnqueueRawOnUninitializedPool` | REQ-PGPP-038 |
| UT-POOL-007 | `AsyncOnUninitializedPoolReturnsNullopt` | REQ-PGPP-029 |
| UT-POOL-008 | `PrepareStatementBeforeInitialize` | REQ-PGPP-025 |
| UT-POOL-009 | `PrepareStatementEmptyName` | REQ-PGPP-004 |
| UT-POOL-010 | `CallbackExecOnUninitializedPool` | REQ-PGPP-033 |
| UT-POOL-011 | `TransactionOnUninitializedPool` | REQ-PGPP-034 |

---

## Integration Tests (require PostgreSQL)

Require a running PostgreSQL instance. The test executable auto-manages a Docker container
via `DockerPostgresEnvironment` (see `tests/common/docker_fixture.h`).
Located in `tests/integration/`, numeric-prefixed for ordering.

### Fixtures

| Fixture | Location | Description |
|---|---|---|
| `DockerPostgresEnvironment` | `common/docker_fixture.h` | Global environment: starts/stops PostgreSQL container |
| `PgppIntegrationTest` | `common/integration_fixture.h` | Pool with 2 connections, creates/drops test table |
| `PgppConnectionTest` | `common/integration_fixture.h` | Single direct connection, creates/drops test table |

### test_1.0_connection.cpp — Connection Lifecycle

| ID | Test | Requirement |
|---|---|---|
| IT-CONN-001 | `OpenValidConnection` | REQ-PGPP-010 |
| IT-CONN-002 | `OpenInvalidConnection` | REQ-PGPP-010 |
| IT-CONN-003 | `IsOpenAfterOpen` | REQ-PGPP-010 |
| IT-CONN-004 | `CloseAndDoubleClose` | REQ-PGPP-011 |
| IT-CONN-005 | `ResetRestoresConnection` | REQ-PGPP-042 |
| IT-CONN-006 | `LastErrorAfterFailure` | REQ-PGPP-018 |
| IT-CONN-007 | `ReconnectAfterClose` | REQ-PGPP-010 |
| IT-CONN-008 | `OpenTwiceWithoutClose` — idempotent open | REQ-PGPP-010 |
| IT-CONN-009 | `InvalidHostFastFail` | REQ-PGPP-010 |

### test_2.0_statements.cpp — Prepared Statements

Fixture: `PgppConnectionTest`

| ID | Test | Requirement |
|---|---|---|
| IT-STMT-001 | `PrepareValidSQL` | REQ-PGPP-013 |
| IT-STMT-002 | `PrepareInvalidSQL` | REQ-PGPP-013 |
| IT-STMT-003 | `IsPreparedAfterPrepare` | REQ-PGPP-014 |
| IT-STMT-004 | `IsPreparedFalseForUnknown` | REQ-PGPP-014 |
| IT-STMT-005 | `RePrepareAfterReset` | REQ-PGPP-043 |

### test_3.0_execution.cpp — Direct Execution (PgppConnection)

Fixture: `PgppConnectionTest`

| ID | Test | Requirement |
|---|---|---|
| IT-EXEC-001 | `ExecRawCreateDrop` | REQ-PGPP-015 |
| IT-EXEC-002 | `ExecPreparedInsert` | REQ-PGPP-016 |
| IT-EXEC-003 | `ExecPreparedSelectString` | REQ-PGPP-016, REQ-PGPP-017 |
| IT-EXEC-004 | `ExecPreparedSelectInt` | REQ-PGPP-009 |
| IT-EXEC-005 | `ExecPreparedSelectBool` | REQ-PGPP-009 |
| IT-EXEC-006 | `ExecPreparedSelectDouble` | REQ-PGPP-009 |
| IT-EXEC-007 | `ExecPreparedNullHandling` | REQ-PGPP-008 |
| IT-EXEC-008 | `ExecPreparedMultipleRows` | REQ-PGPP-017 |
| IT-EXEC-009 | `ExecPreparedWrongParamCount` | REQ-PGPP-018 |
| IT-EXEC-010 | `ExecPreparedAllNullRow` | REQ-PGPP-008 |
| IT-EXEC-011 | `ExecPreparedIntBoundaryValues` | REQ-PGPP-009 |
| IT-EXEC-012 | `ExecPreparedFloatPrecision` | REQ-PGPP-009 |
| IT-EXEC-013 | `ExecPreparedZeroRows` | REQ-PGPP-017 |

### test_4.0_pool.cpp — Pool Operations

Fixture: `PgppIntegrationTest` (+ standalone tests for shutdown scenarios)

| ID | Test | Requirement |
|---|---|---|
| IT-POOL-001 | `PoolInitializeWithConnections` | REQ-PGPP-019 |
| IT-POOL-002 | `DoubleInitializeReturnsTrueWhenAlreadyRunning` | REQ-PGPP-019 |
| IT-POOL-003 | `PoolExecSyncInsert` | REQ-PGPP-028 |
| IT-POOL-004 | `PoolQuerySyncSelect` | REQ-PGPP-028 |
| IT-POOL-005 | `PoolExecAsyncFuture` | REQ-PGPP-029 |
| IT-POOL-006 | `PoolQueryAsyncFuture` | REQ-PGPP-029 |
| IT-POOL-007 | `PoolCallbackOnWorkerThread` | REQ-PGPP-032 |
| IT-POOL-008 | `PoolConcurrentQueries` | REQ-PGPP-027, REQ-PGPP-044 |
| IT-POOL-009 | `PoolStatisticsIdle` | REQ-PGPP-039, REQ-PGPP-040 |
| IT-POOL-010 | `PoolStatisticsUnderLoad` | REQ-PGPP-039, REQ-PGPP-040 |
| IT-POOL-011 | `PoolPrepareOnRunningPool` | REQ-PGPP-026 |
| IT-POOL-012 | `PoolQueryCallback` | REQ-PGPP-032 |
| IT-POOL-013 | `DuplicatePrepareStatementHandled` | REQ-PGPP-004 |
| IT-POOL-014 | `PoolExecZeroArgs` | REQ-PGPP-016 |
| IT-POOL-015 | `QueueSaturation` | REQ-PGPP-037, REQ-PGPP-040 |
| IT-POOL-016 | `WorkerRecoveryAfterBadQuery` | REQ-PGPP-041 |
| IT-POOL-017 | `PoolQueryReturningZeroRows` | REQ-PGPP-017 |
| IT-POOL-018 | `PendingRequestsGetNullopt` (standalone) | REQ-PGPP-021 |
| IT-POOL-019 | `SequentialQueries` (standalone, single connection) | REQ-PGPP-028 |
| IT-POOL-020 | `ShutdownDuringSlowQuery` (standalone) | REQ-PGPP-020 |

### test_5.0_transactions.cpp — Transactions

Fixture: `PgppIntegrationTest`

| ID | Test | Requirement |
|---|---|---|
| IT-TXN-001 | `TransactionCommitsOnSuccess` | REQ-PGPP-036 |
| IT-TXN-002 | `TransactionRollsBackOnException` | REQ-PGPP-035 |
| IT-TXN-003 | `TransactionMultiStatement` | REQ-PGPP-036 |
| IT-TXN-004 | `TransactionConstraintViolationRollback` | REQ-PGPP-035 |
| IT-TXN-005 | `TransactionEmptyCommits` | REQ-PGPP-036 |
| IT-TXN-006 | `TransactionManyStatements` | REQ-PGPP-036 |
| IT-TXN-007 | `TransactionDeadlockHandling` | REQ-PGPP-035 |

### test_6.0_raw.cpp — Raw SQL via Pool

Fixture: `PgppIntegrationTest`

| ID | Test | Requirement |
|---|---|---|
| IT-RAW-001 | `ExecRawSyncDDL` | REQ-PGPP-015 |
| IT-RAW-002 | `ExecRawAsyncFuture` | REQ-PGPP-029 |
| IT-RAW-003 | `ExecRawSyncInvalidSQL` | REQ-PGPP-015 |
| IT-RAW-004 | `ExecRawSyncSelect` | REQ-PGPP-015 |
| IT-RAW-005 | `ExecRawMultiStatement` | REQ-PGPP-015 |
| IT-RAW-006 | `ExecRawDDLSequence` | REQ-PGPP-015 |
| IT-RAW-007 | `ExecRawPartialFailure` | REQ-PGPP-018 |

### test_7.0_coroutines.cpp — Coroutines

Fixture: `PgppIntegrationTest` (+ standalone shutdown test)

| ID | Test | Requirement |
|---|---|---|
| IT-CORO-001 | `CoExecInsert` | REQ-PGPP-047 |
| IT-CORO-002 | `CoQueryReturnsRows` | REQ-PGPP-049 |
| IT-CORO-003 | `FireAndForgetSelfDestructs` | REQ-PGPP-045 |
| IT-CORO-004 | `CoExecPreparedBackwardCompat` | REQ-PGPP-051 |
| IT-CORO-005 | `CoExecFailingQuery` | REQ-PGPP-047 |
| IT-CORO-006 | `CoAwaitOnShutdownPool` (standalone) | REQ-PGPP-029 |

---

## Future Work

- Connection pool exhaustion under sustained load
- Behavior when PostgreSQL is restarted mid-operation
- Memory leak detection (valgrind/ASan)
- Thread sanitizer validation

---

## Test Infrastructure

### Directory Structure

```
tests/
  CMakeLists.txt              # GLOB-based test discovery
  common/
    test_config.h             # Test configuration constants, getTestConnectionInfo()
    docker_fixture.h          # DockerPostgresEnvironment (auto-manage PostgreSQL container)
    integration_fixture.h     # PgppIntegrationTest, PgppConnectionTest fixtures
  unit/
    test_0.0_basic.cpp        # UT-BASIC-*
    test_1.0_structs.cpp      # UT-STMT-*, UT-OID-*
    test_2.0_conversions.cpp  # UT-CONV-*
    test_2.1_connection_string.cpp  # UT-CONN-*
    test_3.0_pool_state.cpp   # UT-POOL-*
  integration/
    main.cpp                  # Custom main() with DockerPostgresEnvironment registration
    test_1.0_connection.cpp   # IT-CONN-*
    test_2.0_statements.cpp   # IT-STMT-*
    test_3.0_execution.cpp    # IT-EXEC-*
    test_4.0_pool.cpp         # IT-POOL-*
    test_5.0_transactions.cpp # IT-TXN-*
    test_6.0_raw.cpp          # IT-RAW-*
    test_7.0_coroutines.cpp   # IT-CORO-*
```

### CMake Integration

```cmake
option(PGPP_BUILD_TESTS "Build pgpp tests" OFF)

if(PGPP_BUILD_TESTS)
    enable_testing()

    # Unit tests — GLOB discovers all .cpp in unit/
    file(GLOB_RECURSE UNIT_TEST_SOURCES CONFIGURE_DEPENDS unit/*.cpp)
    add_executable(pgpp_unit_tests ${UNIT_TEST_SOURCES})
    target_link_libraries(pgpp_unit_tests PRIVATE pgpp GTest::gtest_main)
    add_test(NAME pgpp_unit_tests COMMAND pgpp_unit_tests)

    # Integration tests — GLOB discovers all .cpp in integration/
    # Uses custom main() (not gtest_main) for Docker environment setup
    file(GLOB_RECURSE INTEGRATION_TEST_SOURCES CONFIGURE_DEPENDS integration/*.cpp)
    add_executable(pgpp_integration_tests ${INTEGRATION_TEST_SOURCES})
    target_include_directories(pgpp_integration_tests PRIVATE common/)
    target_link_libraries(pgpp_integration_tests PRIVATE pgpp GTest::gtest)
    add_test(NAME pgpp_integration_tests COMMAND pgpp_integration_tests)
endif()
```

### Docker Fixture

Integration tests auto-manage a PostgreSQL container:
- **Image:** `postgres:16-alpine`
- **Container:** `pgpp-test-pg-7f3a`
- **Port:** 15432
- **Skip:** Set `PGPP_SKIP_DOCKER=1` to use an external PostgreSQL instance

### Test Naming Convention

Files use numeric prefixes (`N.M_topic.cpp`) to control execution order and logical grouping.
The major number groups related areas, the minor number orders within a group.
