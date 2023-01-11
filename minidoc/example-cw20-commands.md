# Old example history of deploying and interacting with a CW20 contract.

```console
git clone git@github.com:CudoVentures/cudos-node.git
```

```console
make
```

```console
NODE="http://mainnet-full-node-01.gcp.service.cudo.org:26657"
```

```console
CHAIN_ID="cudos-testnet-public-3"
```

```console
KEYRING="os"
```

```console
TXFLAGS="--node $NODE --chain-id $CHAIN_ID --gas auto --gas-adjustment 1.3 --keyring-backend $KEYRING -y"
```

```console
cudos-noded keys add owner --keyring-backend "$KEYRING"
```

```console
cudos-noded keys add alice --keyring-backend "$KEYRING"
```

```console
cudos-noded query bank balances cudos1hdcnuet7wmmhqk93n8h9gzpkljd3dvzqut5jyh --node $NODE
```

```console
OWNER=$( cudos-noded keys show -a owner --keyring-backend "$KEYRING" | tee /dev/tty | tail -1 | tr -d '\r' )
```

```console
ALICE=$( cudos-noded keys show -a alice --keyring-backend "$KEYRING" | tee /dev/tty | tail -1 | tr -d '\r' )
```

```console
echo $NODE
```

```console
echo $TXFLAGS
```

```console
cd cudos
```

```console
RES=$( cudos-noded tx wasm store ./cw20_base.wasm --from owner $TXFLAGS | tee /dev/tty | tail -1 | tr -d '\r' )
```

```console
cd ..
```

```console
cd cudos-node/
```

```console
cudos-noded keys show owner --keyring-backend "$KEYRING"
```

```console
RES=$( cudos-noded tx wasm store ../cudos/cw20_base.wasm --from owner $TXFLAGS | tee /dev/tty | tail -1 | tr -d '\r' )
```

```console
CODE_ID=$( echo $RES | jq -r '.logs[0].events[-1].attributes[-1].value' | tee /dev/tty )
```

```console
INIT=$( jq -n --arg address $OWNER '{ "name": "icecream", "symbol": "ic", "decimals": 6, "initial_balances": [ { "address": $address, "amount": "1000000" } ], "mint": { "minter": $address, "cap": "99900000000" } }' | tee /dev/tty )
```

```console
INIT=$( jq -n --arg address $OWNER '{ "name": "icecream", "symbol": "icream", "decimals": 6, "initial_balances": [ { "address": $address, "amount": "1000000" } ], "mint": { "minter": $address, "cap": "99900000000" } }' | tee /dev/tty )
```

```console
cudos-noded tx wasm instantiate $CODE_ID "$INIT" --from owner --label "CW20" $TXFLAGS
```

```console
CONTRACT=$( cudos-noded query wasm list-contract-by-code $CODE_ID --node $NODE --output json | jq -r '.contracts[-1]' | tee /dev/tty | tail -1 | tr -d '\r' )
```

```console
BALANCE_OF_OWNER=$( jq -n --arg address $OWNER '{ "balance": { "address": $address } }' | tee /dev/tty )
```

```console
echo $ALICE
```

```console
TRANSFER_TO_ALICE=$( jq -n --arg recipient $ALICE '{ "transfer": { "recipient": $recipient, "amount": "1111" } }' | tee /dev/tty )
```

```console
BALANCE_OF_ALICE=$( jq -n --arg address $ALICE '{ "balance": { "address": $address } }' | tee /dev/tty )
```

```console
cudos-noded query wasm contract-state smart $CONTRACT "$BALANCE_OF_ALICE--node $NODE
```

```console
cudos-noded tx wasm execute $CONTRACT "$TRANSFER_TO_ALICE" --from owner $TXFLAGS
```

```console
cudos-noded query wasm contract-state smart $CONTRACT "$BALANCE_OF_ALICE" --node $NODE
```

```console
cudos-noded query wasm contract-state smart $CONTRACT "$BALANCE_OF_OWNER" --node $NODE
```