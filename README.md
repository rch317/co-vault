### CO-Vault

Probably makes the most sense to clone this into /opt/docker, or somewhere
like that, but to each his own.

```
git clone git@github.com:rch317/co-vault.git
cd co-vault
docker-compose up -d
```

If you don't already have vault installed on your local machine, or a machine
that has access to wherever you started vault services, you will need to
do so by downloading it here:

[https://www.vaultproject.io/downloads.html](https://www.vaultproject.io/downloads.html)

Once you have vault installed, and correctly added to your PATH;  you will need to set some
variables.

Linux / Mac:
```
export VAULT_ADDR="http://ip.of.docker.host:8200"
```

Windows CMD:
```
set VAULT_ADDR="http://ip.of.docker.host:8200"
```

Windows PSH:
```
$VAULT_ADDR = "http://ip.of.docker.host:8200"
```

Now you will need to initialize the vault, if this is the first time running it:

```
vault init
```

You will see something similar to:

```
robhough@rch-mbp [14:18:43] [~]
-> % export VAULT_ADDR=http://10.20.20.109:8200

robhough@rch-mbp [14:18:45] [~]
-> % vault init
Unseal Key 1: l1LMYdZBfXhDb80vSdNiOE0BSOU5wcRrA1JDcUbVzPxa
Unseal Key 2: sF5M4gGCiHU7rOEraenF6QptME7kvd/PE2MxIO0szUZK
Unseal Key 3: avnNXbhzVHPNkm59tTjGBp3M/bTst3jAIt1WWCvmQLVI
Unseal Key 4: NYWd2lntDi0kl6XwrmwrP/LFI9s7gcLnr9nAhJzsE8kr
Unseal Key 5: hyHqXLhqsxeuCl0mRrRk4Z/UBznxYLallqE5ghe+yImq
Initial Root Token: 59e42f25-48b2-11bb-bf25-0217095a9c3c

Vault initialized with 5 keys and a key threshold of 3. Please
securely distribute the above keys. When the vault is re-sealed,
restarted, or stopped, you must provide at least 3 of these keys
to unseal it again.

Vault does not store the master key. Without at least 3 keys,
your vault will remain permanently sealed.
```

#### *** _IT IS VERY IMPORTANT THAT YOU SAVE THIS DATA SOMEWHERE SAFE_ ***

You will need to unseal the vault, after you have initilized it.  Any time
the server restarts you will have to unseal the vault. This is why it is
***VERY*** important that you save the above data in safe/secure location. To
unseal, you have to provide 3 of the 5 keys above. You can use the same key
more than once in a single attempt to unseal the vault:

I save this data to a flat file in my home directory:

```
robhough@rch-mbp [14:25:40] [~]
-> % vault init |tee -a $HOME/.vault_init.data
Unseal Key 1: yLmQ8Pww5C5eb3Vpcm1ZG+7DE2U1BY4TSnHgm1z2XWqb
Unseal Key 2: LsAhgSeVIHxuD/cbpR0bQ26Fu6sN1fvSE8/amoKtBwm3
Unseal Key 3: ZgmcwtoKZLQOeZ1Q3/ZCr5dOdYG5lUh4SxkOFEwBwah0
Unseal Key 4: yCp1yKsukwFg+8amw5tQ/H7jmVHnUJdj124njJtzJICc
Unseal Key 5: KZ7ONHx3cRS8VxLkaKAALQ/MZ0Apf/nbMJAfZPYMNFXP
Initial Root Token: e1e5fd5c-5860-86a7-7069-b911d0451b96

Vault initialized with 5 keys and a key threshold of 3. Please
securely distribute the above keys. When the vault is re-sealed,
restarted, or stopped, you must provide at least 3 of these keys
to unseal it again.

Vault does not store the master key. Without at least 3 keys,
your vault will remain permanently sealed.

robhough@rch-mbp [14:26:03] [~]
-> % chmod 0600 $HOME/.vault_init.data
```

Then use the keys "Unseal Key {n}" to unseal the vault. Remember that you have
to perform this task 3 times:

```
robhough@rch-mbp [14:29:23] [~]
-> % vault unseal
Key (will be hidden):
Sealed: true
Key Shares: 5
Key Threshold: 3
Unseal Progress: 1
Unseal Nonce: 99185f21-7956-bcbd-f032-229cfc6e921d
robhough@rch-mbp [14:29:29] [~]

robhough@rch-mbp [14:29:29] [~]
-> % vault unseal
Key (will be hidden):
Sealed: true
Key Shares: 5
Key Threshold: 3
Unseal Progress: 2
Unseal Nonce: 99185f21-7956-bcbd-f032-229cfc6e921d

robhough@rch-mbp [14:29:49] [~]
-> % vault unseal
Key (will be hidden):
Sealed: false
Key Shares: 5
Key Threshold: 3
Unseal Progress: 0
Unseal Nonce:
```

Take note of the status of "Sealed".  This will change from True to False once
you have successfully unsealed the vault.  You can query vault for its status
to validate:

```
robhough@rch-mbp [14:30:09] [~]
-> % vault status
Type: shamir
Sealed: false
Key Shares: 5
Key Threshold: 3
Unseal Progress: 0
Unseal Nonce:
Version: 0.9.1
Cluster Name: vault-cluster-d2b2e7f9
Cluster ID: fe8a9dd0-9ac4-c704-ad5a-1f69cc63092b

High-Availability Enabled: true
	Mode: active
	Leader Cluster Address: https://0.0.0.0:444
```

You can further test vault by creating a secret.  However, in order to read/write
from the vault you will also need to set the auth token.  This will be in the data
you saved in previously. It will be listed as "Initial Root Token":

```
export VAULT_TOKEN="e1e5fd5c-5860-86a7-7069-b911d0451b96"

robhough@rch-mbp [14:35:38] [~]
-> % vault write secret/hello value=world
Success! Data written to: secret/hello

robhough@rch-mbp [14:35:56] [~]
-> % vault read secret/hello
Key             	Value
---             	-----
refresh_interval	768h0m0s
value           	world
```
