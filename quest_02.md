# Preamble

## Imports


```python
import requests
from stellar_sdk import Account,Keypair, Server, Network, TransactionBuilder
from helper import get_wallets
```

## Outline

1. Fund helper account.
2. Create quest account.
3. Build Transaction with 100 Operators
4. Sign and submit

## Environment

`get_wallets` is a helper function that loads my wallets' keys into a dictionary object.


```python
wallets = get_wallets()
```


```python
server = Server(horizon_url="https://horizon-testnet.stellar.org")
```


```python
quest = Keypair.from_secret(
    wallets['QUEST']['secret']
)
```


```python
helper = Keypair.random()
```

# Main

## Fund helper account


```python
funding_request = (
    f'https://friendbot.stellar.org?addr={helper.public_key}'
)
response = requests.get(funding_request)
assert response.status_code == 200, (
    f'Funding request failed: {response.status_code}'
)
print('Funding request was successful.')
```

    Funding request was successful.


## Create and Fund Quest Account


```python
helper_account = server.load_account(helper.public_key)
```


```python
fund_tx = (
    TransactionBuilder(
        source_account=helper_account,
        network_passphrase=Network.TESTNET_NETWORK_PASSPHRASE
    )
    .append_create_account_op(
        destination=quest.public_key,
        starting_balance='500'
    )
    .build()
)
fund_tx.sign(helper)
```


```python
response = server.submit_transaction(fund_tx)
print(f'Transaction successful: {response["successful"]}')
```

    Transaction successful: True


## Build transaction with 100 Operators


```python
quest_account = server.load_account(quest.public_key)
```


```python
_tx = TransactionBuilder(
    source_account=quest_account,
    network_passphrase=Network.TESTNET_NETWORK_PASSPHRASE
)
```

Here, we add 100 payment operations, wherein each one send 1 XLM back to the helper account.


```python
for _ in range(100):
    _tx.append_payment_op(
        destination=helper.public_key,
        amount='1'
    )
```

NOTE: In testing, it was not possible to build and sign in one line
```{python}
(
    _tx
    .build()
    .sign(quest)
)
```
Instead, it must be done in two steps.


```python
tx = _tx.build()
```


```python
tx.sign(quest)
```


```python
response = server.submit_transaction(tx)
print(f'Transaction successful: {response["successful"]}')
```

    Transaction successful: True

