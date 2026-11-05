# False-Security Test Anti-Patterns (Before & After)

## 1. Tautological Assertion Locking in a Bug (Violates Dimension 1: Mutation Sensitivity)

A tautological test either recomputes the expected value using the same formula as the production code, or copy-pastes the code's actual output (including bugs) into the assertion.

```ts
// BAD 1 (Formula duplication): Recomputes expected value the exact same way
test("calculateInvoiceTotal applies discount and tax", () => {
  const items = [{ price: 100, qty: 2 }];
  const expected = items.reduce((s, i) => s + i.price * i.qty, 0) * (1 - 0.1) * 1.08;
  expect(calculateInvoiceTotal(items, { discount: 0.1, tax: 0.08 })).toBe(expected);
});

// BAD 2 (Buggy snapshot copy-paste): Export function had a bug that emitted duplicate
// child paths when a parent folder was already selected, and the test author pasted
// the buggy output straight into `toEqual`.
test("collectExportPaths lists selected paths", () => {
  expect(collectExportPaths(workspace)).toEqual([
    "docs/guide.md",
    "assets",
    "assets/logo.png", // <-- redundant child entry locked in by copy-paste!
  ]);
});

// GOOD: Expected values are independent, hand-worked literals derived from the spec
test("calculateInvoiceTotal applies 10% discount then 8% tax", () => {
  const items = [{ price: 100, qty: 2 }]; // 200 -> 180 after discount -> 194.4 after tax
  expect(calculateInvoiceTotal(items, { discount: 0.1, tax: 0.08 })).toBe(194.4);
});

test("collectExportPaths deduplicates files already covered by a selected directory", () => {
  expect(collectExportPaths(workspace)).toEqual([
    "docs/guide.md",
    "assets",
  ]);
});
```

## 2. Weak / Smoke-Only Assertion (Violates Dimension 1: Mutation Sensitivity)

Asserting only that a function didn't throw, returned `status === 200`, or produced a non-null object proves the code didn't crash—it does not prove the business invariant holds.

```ts
// BAD: Passes even if `refundOrder` is a no-op that never updates order status or balances
test("refundOrder succeeds for paid order", async () => {
  const res = await refundOrder(orderId);
  expect(res.ok).toBe(true);
});

// GOOD: Verifies the observable state transition through the public interface
test("refundOrder marks order as refunded and restores customer credit", async () => {
  const res = await refundOrder(orderId);
  expect(res.ok).toBe(true);

  const order = await getOrder(orderId);
  expect(order.status).toBe("refunded");
  expect(await getCustomerCredit(order.customerId)).toBe(500);
});
```

## 3. Self-Validating Fake / Mock Divergence (Violates Dimension 2: Boundary Fidelity)

A mock or fake at a system boundary is more permissive than the real external dependency, so tests pass against the fake and fail in production.

```ts
// BAD: In-memory fake payment gateway accepts floating-point dollars (`19.99`),
// whereas the real Stripe API requires integer cents (`1999`) and rejects floats.
const fakeGateway = {
  charge: async ({ amount }) => ({ id: "ch_123", status: "succeeded", amount }),
};

// GOOD: Fake enforces the real boundary contract (integer cents, currency required),
// AND contract/E2E tests verify the adapter against the real sandbox API.
const fakeGateway = {
  charge: async ({ amount, currency }) => {
    if (!Number.isInteger(amount) || amount <= 0) {
      throw new Error("Invalid integer amount in cents");
    }
    if (!currency) throw new Error("Missing currency");
    return { id: "ch_123", status: "succeeded", amount };
  },
};
```

## 4. Implementation-Coupled / Static Prose Assertion (Violates Dimension 3: Resistance to Refactoring)

Mocking internal collaborators or regex-matching static template/documentation prose causes tests to break on harmless refactors while missing behavioral regressions.

```ts
// BAD 1 (Internal collaborator spy): Breaks if caching or batching is refactored
test("registerUser calls userRepo.insert and emailFormatter.format", async () => {
  const spy = vi.spyOn(emailFormatter, "formatWelcome");
  await registerUser({ email: "a@b.com" });
  expect(spy).toHaveBeenCalledTimes(1);
});

// BAD 2 (Static prose regex): Breaks whenever someone edits help/guide wording
test("scaffoldProject generates guide markdown", () => {
  scaffoldProject(dir);
  const guide = readFileSync(join(dir, "GUIDE.md"), "utf8");
  expect(guide).toMatch(/Run `npm run build` before deploying to production/);
});

// GOOD: Verify observable behavior at the public seam
test("registerUser persists the user and queues a welcome email to their address", async () => {
  const user = await registerUser({ email: "a@b.com" });
  expect(await getUser(user.id)).toMatchObject({ email: "a@b.com" });
  expect(fakeMailer.sentTo("a@b.com")).toHaveLength(1);
});

test("scaffoldProject produces a project that passes config validation", () => {
  scaffoldProject(dir);
  expect(validateProject(dir).valid).toBe(true);
});
```

## 5. Nominal Boundary Check Missing Real Failure Vectors (Violates Dimension 4: Risk-Aligned Necessity)

Testing only the most obvious failure mode (e.g., lexical `../` in a path, or option `A` alone) while leaving the harder runtime mechanism (symlinks via `realpath`, partial state, or `A + B` combined) untested.

```ts
// INCOMPLETE: Only tests string `../` traversal; passes even if realpath check is deleted
test("readWorkspaceFile rejects parent traversal", () => {
  expect(() => readWorkspaceFile(root, "../etc/passwd")).toThrow(/escapes workspace/);
});

// COMPLETE: Also tests symlink escape inside the workspace so deleting the `realpath`
// check immediately turns the test red.
test("readWorkspaceFile rejects both lexical .. and symlinks pointing outside workspace", () => {
  expect(() => readWorkspaceFile(root, "../etc/passwd")).toThrow(/escapes workspace/);

  symlinkSync(join(outsideDir, "secret.txt"), join(root, "innocent-link.txt"));
  expect(() => readWorkspaceFile(root, "innocent-link.txt")).toThrow(/escapes workspace/);
});
```

## 6. Flaky / Opaque Megatest (Violates Dimension 5: Diagnostic Signal-to-Noise)

A test that depends on wall-clock timing or real network, or that bundles many unrelated invariants behind one opaque setup, fails intermittently and does not tell the reader which business rule broke.

```ts
// BAD: Real timer + real network + five unrelated checks in one test.
// A slow CI runner or DNS hiccup turns it red; when it fails, "expected true, got false"
// says nothing about whether expiry, renewal, or audit logging regressed.
test("session lifecycle works", async () => {
  const s = await createSession(await fetchUser("https://api.example.com/users/1"));
  await new Promise((r) => setTimeout(r, 1100)); // hope the 1s TTL has elapsed
  expect(isExpired(s)).toBe(true);
  expect((await renewSession(s)).ok).toBe(true);
  expect(auditLog.length > 0).toBe(true);
});

// GOOD: Controlled clock, boundary-level fake, one invariant per test with a
// name that states the rule.
test("session expires exactly when its TTL elapses", () => {
  const clock = fakeClock("2026-01-01T00:00:00Z");
  const s = createSession(user, { ttlMs: 1000, clock });
  clock.advance(999);
  expect(isExpired(s)).toBe(false);
  clock.advance(1);
  expect(isExpired(s)).toBe(true);
});

test("renewing an expired session is rejected and audited", () => {
  const clock = fakeClock("2026-01-01T00:00:00Z");
  const s = createSession(user, { ttlMs: 1000, clock });
  clock.advance(1000);
  expect(() => renewSession(s)).toThrow(/expired/);
  expect(auditLog.entriesFor(s.id)).toEqual([{ type: "renew_rejected", reason: "expired" }]);
});
```
