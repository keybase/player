# player

[![Travis CI](https://travis-ci.org/keybase/player.svg?branch=master)](https://travis-ci.org/keybase/player)

Library to read Merkle root out of Stellar blockchain, and play back sigchains

## Install

```
$ npm i -g keybase-player
```

## Demo

```
$ keybase-player --file out.json max
✔ 1. fetch keybase path from root for max: got back seqno #14580525
✔ 2. check hash equality for null: skipped
✔ 3. extract UID for max: map to dbb165b7879fe7b1174df73bed0b9500 via legacy tree
✔ 4. fetch latest root from stellar: returned #28191757, closed at 2020-02-12T14:00:05Z
✔ 5. fetch historical keybase path from root for dbb165b7879fe7b1174df73bed0b9500: got back seqno #14579852
✔ 6. walk path to leaf for dbb165b7879fe7b1174df73bed0b9500: tail hash is b548aff2992c404cdb223c225e98c59425912fee66f5538ee284c330af6fe78d
✔ 7. check hash equality for f4ae8f348c7a579c798a10f812b21a297c10e342cbe5164c056b438df088b84a: match
✔ 8. check skips from 14580525<-14579852: done
✔ 9. walk path to leaf for dbb165b7879fe7b1174df73bed0b9500: tail hash is b548aff2992c404cdb223c225e98c59425912fee66f5538ee284c330af6fe78d
✔ 10. fetch sigchain from keybase for dbb165b7879fe7b1174df73bed0b9500: got back 693 links
✔ 11. fetch all public keys from keybase: got 62 keys
✔ 12. play sigchain for dbb165b7879fe7b1174df73bed0b9500: got key family: PUK generation: 21; live devices: 7; live PGP keys: 2
$ cat out.json | jq .devices[0]
{
  "name": "cry glass",
  "id": "9e6c67030acbcb8f1b970f2eb9eddf18",
  "type": "backup",
  "keys": {
    "sig": "012065ae849d1949a8b0021b165b0edaf722e2a7a9036e07817e056e2d721bddcc0e0a",
    "enc": "0121813f9488c921ef2277fd32c4165655e12deefd6457dcaee79412fdce3d9cf70e0a"
  }
}
```

## The Code

The main operations can be found in the [Runner](./src/run.ts) class:

```TypeScript
  async runWithReporter(r: Reporter): Promise<UserSigChain | UserKeys> {
    const treeWalker = new TreeWalker(r)
    const userSigChain = await treeWalker.walkUidOrUsername(this.username)
    if (this.opts.tree) {
      return userSigChain
    }
    const keyring = new KeyRing(userSigChain.uid, r)
    await keyring.fetch()
    const player = new Player(r)
    const userKeys = await player.play(userSigChain, keyring)
    return userKeys
  }
```
