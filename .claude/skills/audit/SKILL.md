---
name: audit
description: Audit codebase for DDD, SOLID, clean architecture, and 0cmplx conventions. Reports violations and optionally fixes them.
---

## When to run

Run this skill when:
- Before shipping a batch of changes
- After adding new features or refactoring
- Periodically as a health check
- When asked to review code quality

## Configuration

### Repositories to audit

| Repo | Path | Layers |
|---|---|---|
| server | /Users/Elvis/workspace/0cmplxHq/0cmplx-server/src | domain, application, infrastructure, presentation |
| web | /Users/Elvis/workspace/0cmplxHq/0cmplx-web/src | components, hooks, state, lib |
| cli | /Users/Elvis/workspace/0cmplxHq/0cmplx-cli/src | commands, lib |
| engine | /Users/Elvis/workspace/0cmplxHq/0cmplx-engine/src | core, parsers, sandbox |

### Scope

By default, audit all repos that have uncommitted or recently changed files. To audit a specific repo, pass it as an argument: `/audit server`.

## Step 1: Detect scope

Determine which repos to audit:

```bash
for repo in 0cmplx-server 0cmplx-web 0cmplx-cli 0cmplx-engine; do
  dir="/Users/Elvis/workspace/0cmplxHq/$repo"
  if [ -d "$dir/.git" ]; then
    changes=$(cd "$dir" && git diff --name-only HEAD 2>/dev/null | wc -l)
    staged=$(cd "$dir" && git diff --cached --name-only 2>/dev/null | wc -l)
    if [ "$changes" -gt 0 ] || [ "$staged" -gt 0 ]; then
      echo "$repo: $changes changed, $staged staged"
    fi
  fi
done
```

If an argument was passed, only audit that repo.

## Step 2: DDD layer violations (server only)

Check that dependency flow is strictly: presentation -> application -> domain. Infrastructure implements domain ports.

### Rules

1. **Domain must not import from application, infrastructure, or presentation**
   ```
   grep -r "from '../../application\|from '../../infrastructure\|from '../../presentation" src/domain/
   ```
   Domain contains: entities, value objects, repository interfaces (ports), domain services.

2. **Application must not import from infrastructure or presentation**
   ```
   grep -r "from '../../infrastructure\|from '../../presentation" src/application/
   ```
   Application contains: use cases, port interfaces, DTOs.

3. **Presentation must not import from infrastructure directly**
   ```
   grep -r "from '../../infrastructure" src/presentation/
   ```
   Presentation should only depend on application use cases and domain types.

4. **Infrastructure implements domain ports**
   Every interface in `domain/*/` that ends in `Repository`, `Service`, `Provider` must have at least one implementation in `infrastructure/`.

### Report format

```
DDD LAYER VIOLATIONS
  [PASS] domain has no upward imports
  [FAIL] application/auth/LoginUseCase.ts imports from infrastructure/persistence/...
  [PASS] presentation only imports from application
  [WARN] domain/user/UserRepository.ts has no implementation in infrastructure/
```

## Step 3: SOLID principles

### Single Responsibility (S)
- Files over 200 lines: flag for review (may need splitting)
- Use cases that do more than one operation: flag
- Components that mix data fetching and rendering: flag

### Open/Closed (O)
- Switch statements on types that could be polymorphic: flag
- Functions with growing if/else chains for new features: flag

### Liskov Substitution (L)
- Repository implementations must match their interface exactly
- No methods that throw "not implemented": flag

### Interface Segregation (I)
- Interfaces with more than 5 methods: flag for potential splitting
- Components that receive props they do not use: flag

### Dependency Inversion (D)
- Use cases must depend on interfaces (ports), not concrete implementations
- `container.ts` is the only file that should instantiate infrastructure classes
- Grep for `new` keyword in application/ and domain/ (should be zero, except value objects)

### Check commands

```bash
# Files over 200 lines
find src/ -name "*.ts" -o -name "*.tsx" | xargs wc -l | sort -rn | head -20

# Direct instantiation in application layer
grep -rn "new " src/application/ --include="*.ts" | grep -v "test\|Error\|Date\|Map\|Set\|URL\|RegExp"

# Unused props in React components (web only)
# Manual review needed
```

## Step 4: Clean architecture checks

1. **No business logic in presentation layer**
   - Route handlers should only: parse request, call use case, format response
   - No database queries, no complex conditionals in routes

2. **No framework coupling in domain**
   - Domain must not import Hono, Redis, SQLite, React, Astro
   - Domain types must be plain TypeScript interfaces

3. **Composition root is container.ts**
   - All wiring in one place
   - No `import { container }` from anywhere except index.ts and test files

4. **Error types defined in domain**
   - Custom errors (AuthError, NotFoundError, etc.) should live in domain or application
   - Presentation maps them to HTTP status codes

## Step 5: 0cmplx conventions

1. **British English**: grep for common American spellings
   ```bash
   grep -rni "authorize\|behavior\|color[^:s]\|favor\|initialize\b" src/ --include="*.ts" --include="*.tsx" | grep -v node_modules | grep -v ".d.ts"
   ```
   Note: CSS property names (`color`, `backgroundColor`) and library APIs are exempt.

2. **No em/en dashes** in strings, comments, or UI text
   ```bash
   grep -Prn "[—–]" src/ --include="*.ts" --include="*.tsx"
   ```

3. **Geist Mono font**: all UI text must use `fontFamily: 'var(--font-mono)'` or inherit it

4. **Naming conventions**:
   - Use cases: `VerbNounUseCase` (e.g. `CreateAppUseCase`)
   - Repositories: `NounRepository` interface in domain, `PrefixNounRepository` in infrastructure
   - IDs: `prefix_hex` format via `domain/shared/ids.ts`

5. **No `any` types** (except in test files and known workarounds)
   ```bash
   grep -rn ": any\|as any" src/ --include="*.ts" --include="*.tsx" | grep -v test | grep -v ".d.ts"
   ```

6. **Co-located tests** (server): tests must sit next to the file they test (e.g. `CreateAppUseCase.test.ts` beside `CreateAppUseCase.ts`). Never place tests in a central `__tests__/` folder. Shared fixtures may live in `__tests__/mocks.ts`.
   ```bash
   # Flag any test files in __tests__/ that should be co-located
   find src/__tests__/ -name "*.test.ts" 2>/dev/null
   ```

7. **All use cases must publish events** via `EventPublisher`. No silent mutations. Check for use cases that mutate state without calling `this.publisher.publishAppUpdate()`.
   ```bash
   # Use cases with a repo write but no publisher
   for f in $(find src/application -name "*.ts" ! -name "*.test.ts"); do
     if grep -q "appRepo\.\|userRepo\.\|schemaRepo\." "$f" && ! grep -q "publisher" "$f"; then
       echo "[WARN] $f - mutates state but has no EventPublisher"
     fi
   done
   ```

8. **No guest/anonymous access**: all app creation and mutation requires authenticated userId. `App.userId` is non-nullable (`string`, not `string | null`).

## Step 6: Web-specific checks (web only)

1. **No `window`/`document` in SSR context**: check useState initialisers, module scope
2. **No JS-based responsive switching**: use CSS media queries, not matchMedia
3. **Hooks order**: no early returns before hooks
4. **No `window.location.href` for reactive navigation**: only for deliberate user actions (form submit, sign out)

## Step 7: Report

Generate a summary report:

```
0cmplx Code Audit
==================

Scope: server, web (2 repos with changes)

DDD Layer Violations
  [PASS] 0 violations found

SOLID Violations
  [WARN] 3 files over 200 lines
  [FAIL] 1 direct instantiation in application layer

Clean Architecture
  [PASS] No business logic in routes
  [FAIL] AuthError defined in application, not domain

Conventions
  [PASS] British English
  [PASS] No em/en dashes
  [WARN] 2 uses of `any` outside tests

Web
  [PASS] No SSR violations
  [PASS] No reactive redirects

Total: X passes, Y warnings, Z failures

Failures (must fix):
  1. src/application/auth/RegisterUseCase.ts:65 - AuthError should be in domain/
  ...

Warnings (should fix):
  1. src/components/organisms/MobileDashboard.tsx - 477 lines, consider splitting
  ...
```

## Step 8: Fix mode (optional)

If the user asks to fix violations, address them in order:
1. DDD layer violations (most critical)
2. SOLID failures
3. Clean architecture issues
4. Convention violations

Create a feature branch: `chore/audit-fixes`

## Gotchas

- `container.ts` is allowed to import from all layers (it is the composition root)
- Test files (`*.test.ts`, `__tests__/`) are exempt from most rules
- `config.ts` is infrastructure but lives at src root for convenience
- CSS colour properties (`color`, `backgroundColor`) are not British English violations
- `Date`, `Map`, `Set`, `URL`, `RegExp`, `Error` constructors are allowed everywhere
