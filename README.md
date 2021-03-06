# Python Ethereum Offline (pyethoff)

Python version of a cold storage system for ethereum, like [Icebox](https://github.com/ConsenSys/icebox).
Ledger HW1 is known to work but integration is far from perfect and is not user friendly at all.
Everything is proposed as is, **use at your own risks** !

Pull requests, help with documentation and reporting issues are all highly encouraged.

## Setup

### pyvenv setup
The setup is based around a python3 virtualenv directory named pyvenv:
```
virtualenv -p python3 pyvenv
. pyvenv/bin/activate
```

Install requirements:
```
. pyvenv/bin/activate
pip install -r requirements.txt
```

### Offline computer setup

On the offline computer, which should **NEVER** be connected to internet, copy the direcory pyvenv and file `tx_sign.py`. It is important that the path to pyenv remains the same. Expect a better deploy process using python wheels in a future version.

## Workflow to sign a transaction offline

On online computer with geth rpc running, prepare the transaction:
```
. pyvenv/bin/activate
python tx_prepare.py --ether 2.5 0xb794f5ea0ba39494ce839613fffba74279579268 0xb794f5ea0ba39494ce839613fffba74279579268
```

On offline computer transfert `ethoff.tx` and sign the transaction:
```
. pyvenv/bin/activate
python tx_sign.py <path to wallet> ethoff.tx
```

Back on online computer with `ethoff.tx.signed`, push the transaction:
```
. pyvenv/bin/activate
python tx_push.py ethoff.tx.signed
```
## Ledger HW1

### pyvenv setup
To sign with Ledger wallet (**dongle in DEV mode only**) install btchip-python:
```
. pyvenv/bin/activate
pip install -r requirements-btchip.txt
```

### HW1 setup
To store a private key onto the ledger wallet (the tests I made were with a HW1), dongle has first to be setup in developper mode:

```
. pyvenv/bin/activate
python
Python 2.7.11 (default, Dec  6 2015, 15:43:46)
[GCC 5.2.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> from btchip.btchip import getDongle, btchip
>>> dongle = getDongle(True)
>>> app = btchip(dongle)
=> f026000000
<= 6d00
>>> app.setup(btchip.OPERATION_MODE_DEVELOPER, btchip.FEATURE_RFC6979, 0x00, 0x05, "1234", None, btchip.QWERTY_KEYMAP)
```

Then each private key could be imported:
```
>>> from btchip.btchip import getDongle, btchip
>>> from bitcoin import encode_privkey
>>> from binascii import hexlify
>>> dongle = getDongle(True)
>>> app = btchip(dongle)
=> f026000000
<= 6d00
>>> app.verifyPin("1234")
=> e02200000431323334
<= 009000
>>> app.importPrivateKey(encode_privkey('f0b81be749f9260e9f4271e7f678e0e60108e5afcc00c4c5d8fed27bd9458498', 'wif'))
=> e0b0010033354b654a52743176727a57636b766f5a7a366e553877394a4e505778444b415068357748334b7964566b656e726459534a6e54
<= 01010019f88b564517a8ed6120d6551e380a00d6d7154619fd4fbb7c76c06d45254a1b9000
bytearray(b'\x01\x01\x00\x19\xf8\x8bVE\x17\xa8\xeda \xd6U\x1e8\n\x00\xd6\xd7\x15F\x19\xfdO\xbb|v\xc0mE%J\x1b')
>>> hexlify(bytearray(b'\x01\x01\x00\x19\xf8\x8bVE\x17\xa8\xeda \xd6U\x1e8\n\x00\xd6\xd7\x15F\x19\xfdO\xbb|v\xc0mE%J\x1b'))
'01010019f88b564517a8ed6120d6551e380a00d6d7154619fd4fbb7c76c06d45254a1b'
```

Where the private key 'f0b81be749f9260e9f4271e7f678e0e60108e5afcc00c4c5d8fed27bd9458498' can be obtained with:
```
python pyethereum/tools/keystorer.py getprivkey 341d0b23-b54b-dc77-0458-5bc9896a835c.json
```

### Signature
To then use the private key from the dongle to sign hash, the private key ID displayed as a return result after the importPrivateKey (01010019f88b564517a8ed6120d6551e380a00d6d7154619fd4fbb7c76c06d45254a1b) has to be provided **everytime**. It's inconvenient but I have not found a workaround ...

To sign with ledger wallet use --keytype option with the previously obtained private key ID (from the importPrivateKey):
```
. pyvenv/bin/activate
python tx_sign.py --keytype dongle 01010019f88b564517a8ed6120d6551e380a00d6d7154619fd4fbb7c76c06d45254a1b ethoff.tx
```

## Ledger Nano S

### pyvenv setup
To sign with Ledger Nano S wallet install blue-loader-python (fork of the original to support python3):
```
. pyvenv/bin/activate
pip install -r requirements-nanos.txt
```

### Workflow

On online computer with geth rpc running, prepare the transaction:
```
. pyvenv/bin/activate
python tx_prepare.py --keytype nanos max "44'/60'/0'/0" 0xb794f5ea0ba39494ce839613fffba74279579268
```

On offline computer transfert `ethoff.tx` and sign the transaction:
```
. pyvenv/bin/activate
python tx_sign.py --keytype nanos "44'/60'/0'/0" ethoff.tx
```

Back on online computer with `ethoff.tx.signed`, push the transaction:
```
. pyvenv/bin/activate
python tx_push.py ethoff.tx.signed
```
