## Cosmos-sdk-惩罚模块

双重签名 （double sign）

```go
// handle a validator signing two blocks at the same height
// power: power of the double-signing validator at the height of infraction
func (k Keeper) HandleDoubleSign(ctx sdk.Context, addr crypto.Address, infractionHeight int64, timestamp time.Time, power int64) { }
```