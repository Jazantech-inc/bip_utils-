## Cardano library

The Cardano library allows to generate Cardano keys and addresses like Ledger and AdaLite/Yoroi wallets.\
Both Byron and Shelley addresses are supported.

### Cardano Byron

The library can generate Cardano Byron keys and addresses based on the legacy, Icarus and Ledger algorithms.

The Icarus and Ledger algorithms are based on BIP-0044, so the [BIP-0044 module](https://github.com/ebellocchia/bip_utils/tree/master/readme/bip44.md)
shall be used. The derivation path is: `m/44'/1815'/0'/0/0`.\
The generated addresses are in the `Ae2...` format.

The legacy algorithm does not follow BIP-0044. The derivation path is in the format: `m/i'/j'`.\
The generated addresses are in the `Ddz...` format.

#### Legacy

The legacy Byron-era keys and addresses, generated by the old Daedalus wallet, use the [BIP32-Ed25519 (Khovratovich/Law)](https://github.com/LedgerHQ/orakolo/blob/master/papers/Ed25519_BIP%20Final.pdf)
derivation scheme (with some slight differences) with the [Byron master key generation](https://cips.cardano.org/cips/cip3/byron.md).\
It uses BIP-0039 mnemonics, but the seed is the hash of the serialized initial entropy. 
For this reason, the `CardanoByronLegacySeedGenerator` class shall be used to generate the seed.\
The BIP32 derivation scheme is implemented by the `CardanoByronLegacyBip32` class.

The legacy addresses contains the derivation path encrypted using the ChaCha20-Poly1305 algorithm. The encryption key 
is derived from the master public key and chain code.\
This is automatically handled by the `CardanoByronLegacy` class, which can be constructed either from a seed or a
`CardanoByronLegacyBip32` class instance. Constructing it from a BIP32 object allows construction from public/private key bytes or extended keys.\
The returned public/private keys are `Bip32PublicKey`/`Bip32PrivateKey` objects.

**Code example**

    import binascii
    from bip_utils import CardanoByronLegacy, CardanoByronLegacyBip32, CardanoByronLegacySeedGenerator
    
    # Generate seed from mnemonic using CardanoByronLegacySeedGenerator
    mnemonic = "ignore year visit govern grape ocean much ecology path inside shoe twenty"
    seed_bytes = CardanoByronLegacySeedGenerator(mnemonic).Generate()
    
    # Construct the CardanoByronLegacy object from seed
    byron_legacy = CardanoByronLegacy.FromSeed(seed_bytes)
    # Construct the CardanoByronLegacy object directly from a CardanoByronLegacyBip32 object
    byron_legacy = CardanoByronLegacy(
        CardanoByronLegacyBip32.FromSeed(seed_bytes)
    )
    # Print HD path key
    print(byron_legacy.HdPathKey())
    # Print master keys and chain code
    print(byron_legacy.MasterPrivateKey().Raw().ToHex())
    print(byron_legacy.MasterPublicKey().RawCompressed().ToHex())
    print(byron_legacy.MasterPublicKey().ChainCode().ToHex())
    
    # Derive and print the first 5 addresses
    # Please note the key indexes are automatically hardened (i.e. 0, i -> 0', i')
    for i in range(5):
        print(byron_legacy.GetAddress(0, i))
        print(byron_legacy.GetPrivateKey(0, i).Raw().ToHex())
        # Decrypt and get derivation path from address
        # In order to be successful, the address shall be derived from the object master key
        print(byron_legacy.HdPathFromAddress(byron_legacy.GetAddress(0, i)))

#### Yoroi-Icarus

The Byron-era keys and addresses, generated by Yoroi wallet, use the [BIP32-Ed25519 (Khovratovich/Law)](https://github.com/LedgerHQ/orakolo/blob/master/papers/Ed25519_BIP%20Final.pdf)
derivation scheme with the [Icarus master key generation](https://cips.cardano.org/cips/cip3/icarus.md).\
It uses BIP-0039 mnemonics, but the seed is just the initial entropy. 
For this reason, the `CardanoIcarusSeedGenerator` class shall be used to generate the seed.\
The BIP32 derivation scheme is implemented by the `CardanoIcarusBip32` class.

The BIP44 coin to be used in this case is `Bip44Coins.CARDANO_BYRON_ICARUS`.

**Code example**

    from bip_utils import Bip44Changes, Bip44Coins, Bip44, CardanoIcarusSeedGenerator
    
    # Generate seed from mnemonic using CardanoIcarusSeedGenerator
    mnemonic = "cost dash dress stove morning robust group affair stomach vacant route volume yellow salute laugh"
    seed_bytes = CardanoIcarusSeedGenerator(mnemonic).Generate()
    
    # Construct the Bip44 object
    bip44_mst_ctx = Bip44.FromSeed(seed_bytes, Bip44Coins.CARDANO_BYRON_ICARUS)
    # Print master keys and chain code
    print(bip44_mst_ctx.PrivateKey().Raw().ToHex())
    print(bip44_mst_ctx.PublicKey().RawCompressed().ToHex())
    print(bip44_mst_ctx.PublicKey().ChainCode().ToHex())
    
    # Derive and print the first 5 addresses
    bip44_chg_ctx = bip44_mst_ctx.Purpose().Coin().Account(0).Change(Bip44Changes.CHAIN_EXT)
    for i in range(5):
        bip44_addr_ctx = bip44_chg_ctx.AddressIndex(i)
        print(bip44_addr_ctx.PublicKey().ToAddress())
        print(bip44_addr_ctx.PrivateKey().Raw().ToHex())
        print(bip44_addr_ctx.PrivateKey().ChainCode().ToHex())

#### Ledger

The Byron-era keys and addresses, generated by Ledger, use the [BIP32-Ed25519 (Khovratovich/Law)](https://github.com/LedgerHQ/orakolo/blob/master/papers/Ed25519_BIP%20Final.pdf)
derivation scheme with the [Ledger master key generation](https://cips.cardano.org/cips/cip3/ledger_bitbox02.md).\
It uses BIP-0039 mnemonics and seed generation, so the `Bip39SeedGenerator` class shall be used to generate the seed.

The BIP44 coin to be used in this case is `Bip44Coins.CARDANO_BYRON_LEDGER`.

**Code example**
    
    from bip_utils import Bip39SeedGenerator, Bip44Changes, Bip44Coins, Bip44
    
    # Generate seed from mnemonic using Bip39SeedGenerator
    mnemonic = "cost dash dress stove morning robust group affair stomach vacant route volume yellow salute laugh"
    seed_bytes = Bip39SeedGenerator(mnemonic).Generate()
    
    # Construct the Bip44 object
    bip44_mst_ctx = Bip44.FromSeed(seed_bytes, Bip44Coins.CARDANO_BYRON_LEDGER)
    # Print master keys
    print(bip44_mst_ctx.PrivateKey().Raw().ToHex())
    print(bip44_mst_ctx.PublicKey().RawCompressed().ToHex())
    print(bip44_mst_ctx.PublicKey().ChainCode().ToHex())
    
    # Derive and print the first 5 addresses
    bip44_chg_ctx = bip44_mst_ctx.Purpose().Coin().Account(0).Change(Bip44Changes.CHAIN_EXT)
    for i in range(5):
        bip44_addr_ctx = bip44_chg_ctx.AddressIndex(i)
        print(bip44_addr_ctx.PublicKey().ToAddress())
        print(bip44_addr_ctx.PrivateKey().Raw().ToHex())
        print(bip44_addr_ctx.PrivateKey().ChainCode().ToHex())

### Cardano Shelley

The library can generate Cardano Shelley keys and addresses based on the Icarus and Ledger algorithms.\
They are based on [CIP-1852](https://cips.cardano.org/cips/cip1852), so the `Cip1852` class shall be used.
The derivation path is: `m/1852'/1815'/0'/0/0`.\
The generated addresses are in the `addr1...` format.

#### Cip1852 library

The `Cip1852` class allows deriving keys as defined by [CIP-1852](https://cips.cardano.org/cips/cip1852).\
It inherits from the `Bip44Base` class, so its usage is exactly the same of the BIP-0044 classes
(see the [related paragraph](https://github.com/ebellocchia/bip_utils/tree/master/readme/bip44.md)).

Supported coins enumerative for CIP-1852:

|Coin|Main net enum|Test net enum|
|---|---|---|
|Cardano Shelley (Icarus)|`Cip1852Coins.CARDANO_ICARUS`|`Cip1852Coins.CARDANO_ICARUS_TESTNET`|
|Cardano Shelley (Ledger)|`Cip1852Coins.CARDANO_LEDGER`|`Cip1852Coins.CARDANO_LEDGER_TESTNET`|

The `Cip1852` class can be used to derive keys but not to encode addresses (an exception will be raised if the `ToAddress` method is called).\
That's because Shelley needs two keys for computing the address: the address public key and the [staking public key](https://cips.cardano.org/cips/cip11/).\
The staking public key is derived by setting *2* as change in the BIP-0044 path (a new value introduced by Cardano, not present in BIP-0044),
i.e. the derivation path for the staking key is: `m/1852'/1815'/0'/2/0`.

To derive addresses from a `Cip1852` object, the `CardanoShelley` class shall be used.

**Code example**

    import binascii
    from bip_utils import Bip44Changes, Bip44Levels, Cip1852Coins, Cip1852
    
    # Seed bytes
    seed_bytes = binascii.unhexlify(b"0000000000000000000000000000000000000000")
    # Create from seed
    cip1852_mst_ctx = Cip1852.FromSeed(seed_bytes, Cip1852Coins.CARDANO_ICARUS)
    
    # Print master keys
    print(cip1852_mst_ctx.PrivateKey().Raw().ToHex())
    print(cip1852_mst_ctx.PublicKey().RawCompressed().ToHex())
    print(cip1852_mst_ctx.PublicKey().ChainCode().ToHex())
    
    # Print level
    print(cip1852_mst_ctx.Level())
    # Check level
    print(cip1852_mst_ctx.IsLevel(Bip44Levels.MASTER))
    
    # Derive account 0 keys: m/1852'/1815'/0'
    cip1852_acc_ctx = cip1852_mst_ctx.Purpose().Coin().Account(0)
    # Print account keys
    print(cip1852_acc_ctx.PrivateKey().Raw().ToHex())
    print(cip1852_acc_ctx.PublicKey().RawCompressed().ToHex())
    
    # Derive external chain keys: m/1852'/1815'/0'/0
    cip1852_chg_ctx = cip1852_acc_ctx.Change(Bip44Changes.CHAIN_EXT)
    
    # Derive the first 5 address keys: m/1852'/1815'/0'/0/i
    for i in range(5):
        cip1852_addr_ctx = cip1852_chg_ctx.AddressIndex(i)
    
        # Print keys
        print(cip1852_addr_ctx.PrivateKey().Raw().ToHex())
        print(cip1852_addr_ctx.PublicKey().RawCompressed().ToHex())
        print(cip1852_addr_ctx.PublicKey().ChainCode().ToHex())
        # Raise ValueError
        try:
            print(cip1852_addr_ctx.PublicKey().ToAddress())
        except ValueError:
            pass

#### CardanoShelley library

The `CardanoShelley` class allows getting addresses and staking addresses from a `Cip1852` object.\
Since the staking address depends on the account index, the `Cip1852` object used to construct it shall already be at the account level
(i.e. the account keys shall already be derived).\
Once constructed, it allows deriving change and address indexes like the `Cip1852` class, but it internally derives and keeps track
of the staking keys.\

The staking object (a `Cip1852` class instance) can be got through the `StakingObject` method.

The `PrivateKeys` and `PublicKeys` methods allow to get the private/public key pair: the keys of the current `Cip1852` object and
the staking keys.

**Code example**

    import binascii
    from bip_utils import Bip44Changes, CardanoShelley, Cip1852Coins, Cip1852
    
    # Seed bytes
    seed_bytes = binascii.unhexlify(b"0000000000000000000000000000000000000000")
    # Create from seed
    cip1852_mst_ctx = Cip1852.FromSeed(seed_bytes, Cip1852Coins.CARDANO_ICARUS)
    
    # Print master keys
    print(cip1852_mst_ctx.PrivateKey().Raw().ToHex())
    print(cip1852_mst_ctx.PublicKey().RawCompressed().ToHex())
    
    # Derive account 0 keys and use it to create a Cardano Shelley object
    cip1852_acc_ctx = cip1852_mst_ctx.Purpose().Coin().Account(0)
    shelley_acc_ctx = CardanoShelley.FromCip1852Object(cip1852_acc_ctx)
    
    # Print staking keys
    print(shelley_acc_ctx.StakingObject().PrivateKey().Raw().ToHex())
    print(shelley_acc_ctx.StakingObject().PublicKey().RawCompressed().ToHex())
    # Print staking address
    print(shelley_acc_ctx.StakingObject().PublicKey().ToAddress())
    # Alias for StakingObject
    print(shelley_acc_ctx.RewardObject() is shelley_acc_ctx.StakingObject())
    
    # PrivateKeys returns the key pair: current object private key + staking private key
    print(shelley_acc_ctx.PrivateKeys().AddressKey().Raw().ToHex())
    print(shelley_acc_ctx.PrivateKeys().StakingKey().Raw().ToHex())  # Same of StakingObject
    # Alias for StakingKey
    print(shelley_acc_ctx.PrivateKeys().RewardKey() is shelley_acc_ctx.PrivateKeys().StakingKey())
    
    # PublicKeys returns the key pair: current object public key + staking public key
    print(shelley_acc_ctx.PublicKeys().AddressKey().RawCompressed().ToHex())
    print(shelley_acc_ctx.PublicKeys().StakingKey().RawCompressed().ToHex())  # Same of StakingObject
    # Alias for StakingKey
    print(shelley_acc_ctx.PublicKeys().RewardKey() is shelley_acc_ctx.PublicKeys().StakingKey())
    
    # Derive external chain keys
    shelley_chg_ctx = shelley_acc_ctx.Change(Bip44Changes.CHAIN_EXT)
    # Staking object is available at any level and it's always the same
    print(shelley_chg_ctx.StakingObject().PublicKey().ToAddress())
    
    # Derive the first 5 keys and addresses
    for i in range(5):
        shelley_addr_ctx = shelley_chg_ctx.AddressIndex(i)
    
        # Print keys
        print(shelley_acc_ctx.PrivateKeys().AddressKey().Raw().ToHex())
        print(shelley_acc_ctx.PublicKeys().AddressKey().RawCompressed().ToHex())
        # Print address (addr1...)
        print(shelley_addr_ctx.PublicKeys().ToAddress())
        # Print staking address (same of StakingObject)
        print(shelley_addr_ctx.PublicKeys().ToStakingAddress())
        # Same of ToStakingAddress
        print(shelley_addr_ctx.PublicKeys().ToRewardAddress())

#### Yoroi-Icarus

The Shelley-era keys and addresses, generated by Yoroi wallet, are like the Byron ones
([BIP32-Ed25519 (Khovratovich/Law)](https://github.com/LedgerHQ/orakolo/blob/master/papers/Ed25519_BIP%20Final.pdf)
with [Icarus master key generation](https://cips.cardano.org/cips/cip3/icarus.md)) but following CIP-1852 instead of BIP-0044.\
Like in the Byron case, the `CardanoIcarusSeedGenerator` class shall be used to generate the seed and
the BIP32 derivation scheme is implemented by the `CardanoIcarusBip32` class.

The CIP-1852 coins to be used in this case are `Cip1852Coins.CARDANO_ICARUS` or `Cip1852Coins.CARDANO_ICARUS_TESTNET`.

**Code example**

    from bip_utils import Bip44Changes, CardanoIcarusSeedGenerator, CardanoShelley, Cip1852Coins, Cip1852
    
    # Generate seed from mnemonic using CardanoIcarusSeedGenerator
    mnemonic = "cost dash dress stove morning robust group affair stomach vacant route volume yellow salute laugh"
    seed_bytes = CardanoIcarusSeedGenerator(mnemonic).Generate()
    # Construct the Cip1852 object
    cip1852_mst_ctx = Cip1852.FromSeed(seed_bytes, Cip1852Coins.CARDANO_ICARUS)
    # Print master keys
    print(cip1852_mst_ctx.PrivateKey().Raw().ToHex())
    print(cip1852_mst_ctx.PublicKey().RawCompressed().ToHex())
    
    # Construct Cardano Shelley object
    shelley_acc_ctx = CardanoShelley.FromCip1852Object(
        cip1852_mst_ctx.Purpose().Coin().Account(0)
    )
    
    # Print staking address
    print(shelley_acc_ctx.StakingObject().PublicKey().ToAddress())
    
    # Derive the first 5 keys and addresses
    shelley_chg_ctx = shelley_acc_ctx.Change(Bip44Changes.CHAIN_EXT)
    for i in range(5):
        shelley_addr_ctx = shelley_chg_ctx.AddressIndex(i)
    
        print(shelley_addr_ctx.PublicKeys().ToAddress())
        print(shelley_addr_ctx.PrivateKeys().AddressKey().Raw().ToHex())
        print(shelley_addr_ctx.PublicKeys().AddressKey().RawCompressed().ToHex())

#### Ledger

The Shelley-era keys and addresses, generated by Ledger, are like the Byron ones
([BIP32-Ed25519 (Khovratovich/Law)](https://github.com/LedgerHQ/orakolo/blob/master/papers/Ed25519_BIP%20Final.pdf)
with [Ledger master key generation](https://cips.cardano.org/cips/cip3/ledger_bitbox02.md)) but following CIP-1852 instead of BIP-0044.\
Like in the Byron case, the `Bip39SeedGenerator` class shall be used to generate the seed.

The CIP-1852 coins to be used in this case are `Cip1852Coins.CARDANO_LEDGER` or `Cip1852Coins.CARDANO_LEDGER_TESTNET`.

**Code example**

    from bip_utils import Bip39SeedGenerator, Bip44Changes, CardanoShelley, Cip1852Coins, Cip1852
    
    # Generate seed from mnemonic using Bip39SeedGenerator
    mnemonic = "cost dash dress stove morning robust group affair stomach vacant route volume yellow salute laugh"
    seed_bytes = Bip39SeedGenerator(mnemonic).Generate()
    # Construct the Cip1852 object
    cip1852_mst_ctx = Cip1852.FromSeed(seed_bytes, Cip1852Coins.CARDANO_LEDGER)
    # Print master keys
    print(cip1852_mst_ctx.PrivateKey().Raw().ToHex())
    print(cip1852_mst_ctx.PublicKey().RawCompressed().ToHex())
    
    # Construct Cardano Shelley object
    shelley_acc_ctx = CardanoShelley.FromCip1852Object(
        cip1852_mst_ctx.Purpose().Coin().Account(0)
    )
    
    # Print staking address
    print(shelley_acc_ctx.StakingObject().PublicKey().ToAddress())
    
    # Derive the first 5 keys and addresses
    shelley_chg_ctx = shelley_acc_ctx.Change(Bip44Changes.CHAIN_EXT)
    for i in range(5):
        shelley_addr_ctx = shelley_chg_ctx.AddressIndex(i)
    
        print(shelley_addr_ctx.PublicKeys().ToAddress())
        print(shelley_addr_ctx.PrivateKeys().AddressKey().Raw().ToHex())
        print(shelley_addr_ctx.PublicKeys().AddressKey().RawCompressed().ToHex())
