# Module: UniTask + DOTween

For projects that use UniTask for async work and DOTween for tweens.

## UniTask

**Use UniTask instead of coroutines.** Never write `IEnumerator` coroutines or `StartCoroutine`. Use `async UniTask` / `async UniTaskVoid`, and accept and forward a `CancellationToken` so the work stops when its owner is destroyed.

```csharp
// WRONG
private IEnumerator DelayedAction()
{
    yield return new WaitForSeconds(1f);
    DoSomething();
}

// CORRECT
private async UniTaskVoid DelayedActionAsync(CancellationToken ct)
{
    await UniTask.Delay(1000, cancellationToken: ct);
    DoSomething();
}
```

## DOTween with UniTask

- **Recycling.** If DOTween is initialized with `recycleAllByDefault: true` (the `Init` call wins over the settings asset), a completed tween returns to the pool. A `Tween` reference that is awaited (`ToUniTask`) or touched after it may have finished can point at someone else's recycled tween. Any tween kept by reference past its own completion needs `.SetRecyclable(false)`.
- **CS4014.** UniTask adds `GetAwaiter` to `Tween`, so an unawaited call that returns a `Tween` (for example `tween.OnStart(cb)`) raises CS4014. Discard it explicitly: `_ = tween.OnStart(cb);`.
- **Killed tweens.** A tween killed from outside completes its `ToUniTask` await without throwing. Re-check state after the await instead of relying on `OperationCanceledException`.
