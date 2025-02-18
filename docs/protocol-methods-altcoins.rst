.. _Protocol Methods for altcoins:

=================================
 Protocol Methods (for altcoins)
=================================


Masternode methods (Dash and compatible coins)
==============================================


masternode.announce.broadcast
=============================

Pass through the masternode announce message to be broadcast by the daemon.

Whenever a masternode comes online or a client is syncing, they will
send this message which describes the masternode entry and how to
validate messages from it.

**Signature**

  .. function:: masternode.announce.broadcast(signmnb)

  * *signmnb*

    Signed masternode broadcast message in hexadecimal format.

**Result**

  :const:`true` if the message was broadcasted successfully otherwise
  :const:`false`.

**Example**::

  masternode.announce.broadcast("012b825a65a24e2eb8edadbe27c4716dab993bf1046a66da77268ec87dbdd9dfc80100000000ffffffff00000000000000000000ffff22db1fec42d82103bfc9e296bcf4d63eced97b204df8f7b2b90131d452abd2b50909fa2ce6f66d752103bfc9e296bcf4d63eced97b204df8f7b2b90131d452abd2b50909fa2ce6f66d754120e95f74e9c242776df88a586bd52d2bd1838b600e5f3ce9d45d04865ff39a994632d617e810a4480ce24c882980746bc517a92be027d2ea70e4baece33a763608b1f91e5b00000000451201002b825a65a24e2eb8edadbe27c4716dab993bf1046a66da77268ec87dbdd9dfc80100000000ffffffff57280bc007121a0db854998f72e9a9fd2a690f38abffbd9aa94256330c020000b0f91e5b00000000412027c03b1531ee14db6160a62a0cc8b1a7e93ae122bbc6f2dffec721e0ae308b0e19e68523dd429450612bda3a616b56411b4e35d098e25b7c83f19fd2d8537e970000000000000000")

**Example Result**::

  true

masternode.subscribe
====================

Returns the status of masternode.

**Signature**

  .. function:: masternode.subscribe(collateral)

  * *collateral*

    The txId and the index of the collateral.

    A masternode collateral is a transaction with a specific amount of
    coins, it's also known as a masternode identifier.

    i.e. for DASH the required amount is 1,000 DASH or for $PAC is
    500,000 $PAC.

**Result**

  As this is a subscription, the client will receive a notification
  when the masternode status changes.

  The status depends on the server the masternode is hosted, the
  internet connection, the offline time and even the collateral
  amount, so this subscription notice these changes to the user.

**Example**::

  masternode.subscribe("8c59133e714797650cf69043d05e409bbf45670eed7c4e4a386e52c46f1b5e24-0")

**Example Result**::

  {'method': 'masternode.subscribe', u'jsonrpc': u'2.0', u'result': u'ENABLED', 'params': ['8c59133e714797650cf69043d05e409bbf45670eed7c4e4a386e52c46f1b5e24-0'], u'id': 19}

masternode.list
===============

Returns the list of masternodes.

**Signature**

  .. function:: masternode.list(payees)

  * *payees*

    An array of masternode payee addresses.

**Result**

  An array with the masternodes information.

**Example**::

  masternode.list("['PDFHmjKLvSGdnWgDJSJX49Rrh0SJtRANcE',
  'PDFHmjKLvSGdnWgDJSJX49Rrh0SJtRANcF']")

**Example Result**::

    [
      {
        "vin": "9d298c00dae8b491d6801f50cab2e0037852cb556c5619ddb07c50421x9a31ab",
        "status": "ENABLED",
        "protocol": 70213,
        "payee": "PDFHmjKLvSGdnWgDJSJX49Rrh0SJtRANcE",
        "lastseen": "2018-04-01 12:34",
        "activeseconds": 1258000,
        "lastpaidtime": "2018-03-10 12:29",
        "lastpaidblock": 1234,
        "ip": "1.0.0.1",
        "paymentposition": 184,
        "inselection": true,
        "balance": 510350
      },
      {
        "vin": "9d298c00dae8b491d6801f50cab2e0037852cb556c5619ddb07c50421x9a31ac",
        "status": "ENABLED",
        "protocol": 70213,
        "payee": "PDFHmjKLvSGdnWgDJSJX49Rrh0SJtRANcF",
        "lastseen": "2018-04-01 12:34",
        "activeseconds": 1258000,
        "lastpaidtime": "2018-03-15 05:29",
        "lastpaidblock": 1234,
        "ip": "1.0.0.2",
        "paymentposition": 3333,
        "inselection": false,
        "balance": 520700
      },
      ...,
      ...,
      ...,
      ...
    ]


ProTx methods (Dash DIP3)
==============================================


protx.diff
=============================

Returns a diff between two deterministic masternode lists.
The result also contains proof data.

**Signature**

  .. function:: protx.diff(base_height, height)

  *base_height*

    The starting block height

      *1* <= *base_height*

  *height*

    The ending block height.

      *base_height* <= *height*


**Result**

  A dictionary with deterministic masternode lists diff plus proof data

**Example**::

  protx.diff(1, 20000)

**Example Result**::

    {
      "baseBlockHash": "000000000b866e7fefc7df2b4b37f236175cee9ab6dc925a30c62401d92b7406",
      "blockHash": "0000000005b3f97e0af8c72f9a96eca720237e374ca860938ba0d7a68471c4d6",
      "cbTxMerkleTree": "0200000002c9802d02435cfe09e4253bc1ba4875e9a2f920d5d6adf005d5b9306e5322e6f476d885273422c2fe18e8c420d09484f89eaeee7bb7f4e1ff54bddeb94e099a910103",
      "cbTx": "03000500010000000000000000000000000000000000000000000000000000000000000000ffffffff4b02204e047867335c08fabe6d6d8b2b76b7000000000470393f63424273736170747365743a7265737574736574010000000000000010000015770000000d2f6e6f64655374726174756d2f000000000336c8a119010000001976a914cb594917ad4e5849688ec63f29a0f7f3badb5da688ac6c62c216010000001976a914a3c5284d3cd896815ac815f2dd76a3a71cb3d8e688acba65df02000000001976a9146d649e1c05e89d30809ef39cc8ee1002c0c8c84b88ac00000000260100204e0000b301c3d88e4072305bec5d09e2ed6b836b23af640bcdefd7b8ae7e2ca182dc17",
      "deletedMNs": [
      ],
      "mnList": [
        {
          "proRegTxHash": "6f0bdd7034ce8d3a6976a15e4b4442c274b5c1739fb63fc0a50f01425580e17e",
          "confirmedHash": "000000000be653cd1fbc213239cfec83ca68da657f24cc05305d0be75d34e392",
          "service": "173.61.30.231:19023",
          "pubKeyOperator": "8da7ee1a40750868badef2c17d5385480cae7543f8d4d6e5f3c85b37fdd00a6b4f47726b96e7e7c7a3ea68b5d5cb2196",
          "keyIDVoting": "b35c75cbc69433175d3459843e1f6ebe145bf6a3",
          "isValid": true
        }
      ],
      "merkleRootMNList": "17dc82a12c7eaeb8d7efcd0b64af236b836bede2095dec5b3072408ed8c301b3"
    }

protx.info
=============================

Returns detailed information about a deterministic masternode.

**Signature**

  .. function:: protx.info(protx_hash)

  *protx_hash*

    The hash of the initial ProRegTx.

**Result**

  A dictionary with detailed deterministic masternode data

**Example**::

  protx.info("6f0bdd7034ce8d3a6976a15e4b4442c274b5c1739fb63fc0a50f01425580e17e")

**Example Result**::

  {
    "proTxHash": "6f0bdd7034ce8d3a6976a15e4b4442c274b5c1739fb63fc0a50f01425580e17e",
    "collateralHash": "b41439376b6117aebe6ad1ce31dcd217d4934fd00c104029ecb7d21c11d17c94",
    "collateralIndex": 3,
    "operatorReward": 0,
    "state": {
      "registeredHeight": 19525,
      "lastPaidHeight": 20436,
      "PoSePenalty": 0,
      "PoSeRevivedHeight": -1,
      "PoSeBanHeight": -1,
      "revocationReason": 0,
      "keyIDOwner": "b35c75cbc69433175d3459843e1f6ebe145bf6a3",
      "pubKeyOperator": "8da7ee1a40750868badef2c17d5385480cae7543f8d4d6e5f3c85b37fdd00a6b4f47726b96e7e7c7a3ea68b5d5cb2196",
      "keyIDVoting": "b35c75cbc69433175d3459843e1f6ebe145bf6a3",
      "ownerKeyAddr": "ybGQ7a6e7dkJY2jxdbDwdBtyjKZJ8VB7YC",
      "votingKeyAddr": "ybGQ7a6e7dkJY2jxdbDwdBtyjKZJ8VB7YC",
      "addr": "173.61.30.231:19023",
      "payoutAddress": "yWdXnYxGbouNoo8yMvcbZmZ3Gdp6BpySxL"
    },
    "confirmations": 984
  }

Name methods (Namecoin and compatible coins)
==============================================


blockchain.name.get_value_proof
===============================

Returns a name resolution proof, suitable for low-latency (single round-trip) resolution.

**Signature**

  .. function:: blockchain.name.get_value_proof(scripthash, cp_height)
  .. versionadded:: 1.4.3

  *scripthash*

    Script hash of the name being resolved.

  *cp_height*

    Checkpoint height.


**Result**

  A dictionary with transaction and proof data for each transaction associated with the name, from the most recent update back to either the registration transaction or a checkpointed transaction (whichever is later).

**Example**::

  blockchain.name.get_value_proof(bdd490728e7f1cbea1836919db5e932cce651a82f5a13aa18a5267c979c95d3c, 518353)

**Example Result**::

    {
      "bdd490728e7f1cbea1836919db5e932cce651a82f5a13aa18a5267c979c95d3c": [
        {
          "height": 607853,
          "tx": "00710000000102e3d236710b0a21cb9bb4c11d2a2ac6730e6e3c776773f688a53c14fc89ced5ab0100000000ffffffff261a3e3b04326e0dd33208d23e3119144f8cce8394de5757374a37323ee1b6060100000000ffffffff0240420f0000000000415309642f626974636f696e1d76657269667920323032322d30342d313620393a30353a3030204345546d7500144e5cafde0d7fd4f31323e14182a9c3412e026d88978d4e1400000000160014d1a61d11d434ca9197ae47b59a5f8933682140eb02473044022018129841dbd0e9700702b94dfaebee9f00a5e5dc847156faf992195c384a73a502204802754d4a7f406b3fb538d7f2a6c96967deb61eb87597e27dd30410b25e363401210261c92aa521e0e9660d605fff64129ada7faa6f2a201d9f82fd71b4a7ccd0cc230247304402200495ff83f6d2edc697abd3530eb16a315366cf4383225708826ab98c471e8a4b02207ef49122589aaf009e6c7515e04a2e5179263d6efb0f7043d871d3024aae136e0121020138a32ab1be1ae4ee0bf08a2dd6e7208f4e39f98312eebb09016a3fdf31978100000000",
          "tx_merkle": {
            "merkle": [
              "5b08bf9b8e8bff946da742641fc27c107096af5a05c0dcdec1848c3f7423a746",
              "11c88190df88b334f2ce38e7034c1d1caec2e9fccb226ff3229c6440b7a94e6f",
              "ce68ac63e88da5554218cdd7e6811088b1189ab8635bda557044c0518d174d1d"
            ],
            "pos": 1
          }
        },
        {
          "height": 597196,
          "tx": "00710000000102c575b325030f89a7515596c2661967b02ff5a2594ba0a4774833d6bd6bdd41ca0000000000feffffff8afb31ffd43385ff4477c28d81f0dd43e6fc6cc63fe56f7d87733c1de7d67eaa0000000000feffffff028327374a010000001600142715d587d4c8e3edddbac11417b736964c519c6440420f00000000002c5309642f626974636f696e0872657365727665646d750014e893563837316a53cb00c99e13a8079f311173a9024730440220597930ab6071d009ec05e14a9247cbbae4f31c7e3c4e812a995e873eb31646ac02204236ad5e2ee4b72e25428c3e5d70dd9dc387e96bd5dab1f921c6a3dbe7b3f4a8012103549f0ee91a8ce86087fc05862fb06247affca4688d4c6ab166eb9d1531357da90247304402204f48be0eab589fb08acc3cd9281f2c23a742c7745f374176978d7ff1ccf3fd39022052c66eaabab908d7c332083fc88cf863bcf0aea574a1aba6b2dc8106bf0c455a012103877c7f65da9ecc3bd126ee9126b9a03569c2eaebd9b9ef7a30989327321231a7cb1c0900",
          "tx_merkle": {
            "merkle": [
              "bee7bcd954879e9110b019617f443a31f51210b4c841a577abc84dae583a115e",
              "e412f9f6832238fc209fddb2f7aeeedb939e252e0b4a8f7df4b3c509d2b0233e",
              "63e144e0a0726cf8cc9a8c776e6411fc5dfb7f225730bddba0c6a3ab656b3bcd",
              "b5c75bfaaec4c2029b5b175168f9b91af766581dd8496d1184a44e432b20d1fb",
              "4905bf1dc9fbedb16ccacd26a0f406fc62c53b860ac099369010ec610521c4af",
              "3e74dbd7730487df73e07c1fa03b25355c82fbbbd81b35bf28cc8322dda9f3bb"
            ],
            "pos": 38
          }
        },
        {
          "height": 563168,
          "tx": "007100000001026bfeb4ee7d7a5ab62b2e16abf439173a460ac013ee19a0115c26569673eb71eb0000000000feffffff6cee0ee75d60d1a02671d7d2e15e8a8c19045e15d9c6738f76395e8de63ccca30000000000feffffff0240420f00000000002c5309642f626974636f696e0872657365727665646d750014b1cfea8dbd1de3e2e0640b42838f1078f73c050d44d13f4a01000000160014d40f8f5b62c73e2167540001aa1634e60c27fa5a0247304402200207688a4c8e1ca3c093a81f862a0192a94590b2ff4e829564b164ca82b9fada02203ff970582e49ac3d0a8452d211903ba8405153ac556f0a46cfea2d0f3faff182012103cac6fecff4c3a06c71e429dc29a6bea58e42adcc283e5f98e66cb550d6d145df02473044022009e2ec71addf23c21df3034561997aac6e5c7700ac6dd44fcba6157afb865ca202203ad3ff0484f94a405c175184fa9a9ccb6fc8ceab67edc1dba314432e76757d6d01210289706af81a776c60a088ae35021142325f17f81da5778667a172dd1a92e1a644df970800",
          "tx_merkle": {
            "merkle": [
              "9ed4efd8e6b1cf9773db60ef0ad7213ce4282f56df709dd8c69ab2caf466f23b",
              "5cb725f78154e33288b724a59cd6c0146ffb8d0e39ace6f1333479a6eafcb76c",
              "596f00ade273423324821e88c8b99fbe7274470d04c062a0453c7ec1db1086d2",
              "7dfcb41cabce612dc5ab7c4309ada78dbab9dc977321147b2a82ddaae7eb5d02",
              "45d7679dc5e8f92f314b9776ac93e0dcb92015c989c076326511d5a5be5e4a17",
              "ef5096cd39456fe5850c221ad6ec722e11820aa4f6e1773c380e6718e68625e7"
            ],
            "pos": 26
          }
        },
        {
          "height": 528949,
          "tx": "0071000000010234f902ef0f953862bb9ba5b6e9fc1a8a914d8b5b1b9af89e843d85176318c5ff0000000000feffffff24802c9ae06cd5af3dd11bdb653b5df4e9cc6a7bb27b4f1cac13a16fd5249e9b0000000000feffffff0240420f00000000002c5309642f626974636f696e0872657365727665646d750014004b1245eb3ac8724d0bc684e15859c9d5eb9ddabfa39e4a0100000016001440fb148f07ff3c8b1d801628754d8ec5cbe1f7e70247304402203ec1e1907ecefd90180b1f7f9fec4a10b75fdb21a85f0df28ea8fe48419338760220549bebe6b96869eeb3175de8149cd127a42bdb344945bd32571950a1e5df40210121020d0bb22a3730f1de67b87af7e2e5517c7649932e4ad8105817372fd99ca1ce8102473044022013d238876040649429f29f5df08712a3b7439db436e7c7a42e38a2d8c733d05402201475420bfea26e684a4bd488ba6ef0b954ca703234d1536b7a545c55d53759e501210321307ece7f4731fef37b0208807ce1a04d1013c765fef1567752df06e5d7bfe233120800",
          "tx_merkle": {
            "merkle": [
              "0c1a965febb58cd57e8019e6b437f5124968b15855c3066c60a36ae863a900a6",
              "643623318b6409171edb487dc6f14ea56db8956c0bc958abf524ddcf845fd805",
              "1eeb4f63067c4b9ccba24eb717887b3b1f7b95f60ab461b04a785b57251d62d9",
              "94baa94fe1153cc616420bb17bbad79591b409405f8d670b8a1f188e1eda3bc8",
              "bdc4cc1c882e039866939b0ac0c923d1581de93fba1ff1140d3ddc616d1a768d",
              "ac9dbfac19c216f593a932dafe813c3a9ad1d706fdeb2406a0c39db0209cba2c"
            ],
            "pos": 42
          }
        },
        {
          "height": 495000,
          "tx": "007100000001020489c88ef8f718c9ac94bef1424986bfeaa79ec1b66e28841d3de4c3e8238e7f000000006a47304402200b3c3b859fe978f69cadb516e563122ef27fb68dcfea641cbfe69b4c71866cda02206a56c2e1d113e2032cb67a3b506a74ebbdaabca51b3601d9e269042ccce2c3ba012102184feeaa11d4a2ecc0f5c34c10222f6c858d33148c330ad115515d697c77d01efeffffffda3ec65cd0f1d8a5e3b56237993f53c5e1cacc9adec82a808a718023e3592c9a0000000000feffffff0240420f00000000002c5309642f626974636f696e0872657365727665646d75001417e6be040f4b08e4e161db3c27c1dc46b235bba8135a750100000000160014b9667261ba2f8127240b1de27515734bdd5761c00002473044022077d5fc03bdcdd0ced102c2b04ef67d8d9398cc03243f70cef59cbcdae6df0bc202200510a7bc5dc8d8a89dcc16575041912e9fa8913846c1c4ec796c95400ef346d301210330b9946e3a7aaf15858119423ca8cec8bb71e71267e8531feccbeb941abe2e17978d0700",
          "tx_merkle": {
            "merkle": [
              "b87f44bf986dccd425caf731ab403f5d423689977ac15a8d6204e0c8bbf00cf7",
              "f1182408914fa2c86f935ed14a9925a4a20f077b56610131f995b45230155713",
              "78d090050c52d44bf36a99ee4b884a95b6da8fb764fc6e383c6f7b5a4a8ff8be",
              "523dc9ce6021858807700836f65b2b3bf8ea701d723b979afff6ea0cf096815f",
              "0c7c131ab0bf11ba122af39d63a4483b4dbc845a33b507bce7cc0bcefee6d724",
              "a2e5f09a5663c567471571d91c2f8cbd4270d286e772067573439f93e8b4befa",
              "820fbb93b424b3e67602ce2b4136764080f5329b46bc671b97553e2340588e0c"
            ],
            "pos": 11
          },
          "header": {
            "header": "040101003c59eef5329201b1f077d6f8b832b8200cbe9aff82feb75cf5e5bf3240eed8750768161b62133b7b758fe6a724362e3e5948ff4ba3e59098336d057a24d8bd4c37a7535e2394131700000000",
            "branch": [
              "c28a169077edcec70fbb74dac6c64229e862a286fcc22f8695eeb268e7b454a1",
              "7de3af1dce3faf8c93f40608f257a00c042cd13a2b7f5635454079931254602e",
              "9e1f18e383256e49872da0ed2affac378baba7566f2ea0abf6b06d156b331d91",
              "52f09f8d938efcc083643da4171ff0eb77a48dfbbe6ba801494caeb33dd8f2f4",
              "a591862302cb03aebf74fcf9bf552d0ebe50634816719459669532e1f5122d19",
              "2f17efd7c29f3e5f2fc10d494a4dc8cd1e1346fc6848b5f8f053d6133b56af5e",
              "6b1656ebb4b82071b85a6bb6bbf64106f7f9cbd60531d8bbfa665d2d02eae9b0",
              "1841b9a4bc6782948ec92bdbb93c0c8a330728052547f1b2ecd38a2adcff310b",
              "8f5675ecfbdc76c4516f6b110e43b3dd66adb73d2d7f01957055973b832210fa",
              "ede6544bcc09e29229bfaf7d099503172a7826174473816805891b5381dcae3e",
              "9c9a8d5d2d3a7888bedc7906d22ebdd73f684cbc14c435aa4543ad18d56a4ef7",
              "98f0e303e64ddfce18ee0ea9aaf95cecc0d6b849c2d6eb772aba3bf984660601",
              "96c3e6226d5608ccf0a8f3e02d20a258044c9fedb726b041eb94d1d3112cfeb0",
              "f71672e8b1d7621fbf2462e89b0c8d5af91efee7c9de54587acb7d705a8d4659",
              "f9188789442cfeff1955b5030d82317fd77133524bcc4be4e3c24bd857a4925e",
              "7b920859b50d0030c18725888aac05e7edea5bb4f4d3cadd5d6e71fe52a9ebac",
              "e1185d87453a4392ff4d4aea556597bc1b0a1a6320714f364410a73aa136d7aa",
              "345a13ed174cc91593862162fab8610ed84d5dd9cfc1533fd604d812cb13a699",
              "88191322193a9f6ae984009b694edd97670795c8afb130082cf90cd4640e5701"
            ],
            "root": "476a138d228b66a094c20d5bbaea3230ea60201ba16af81500c5f368b06dc48b"
          }
        }
      ]
    }
