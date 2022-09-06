# BOLT #0: Introduction and Index
<!--
Welcome, friend! These Basis of Lightning Technology (BOLT) documents
describe a layer-2 protocol for off-chain bitcoin transfer by mutual
cooperation, relying on on-chain transactions for enforcement if
necessary.
-->

ようこそ、友よ! BOLT（Basis of Lightning Technology）とは？
相互協力によるオフチェーンビットコイン転送のためのレイヤー2プロトコルを記述しています。
オンチェーン取引に依存し、必要に応じて強制することができます。
必要な場合はオンチェーン取引に依存します。

<!--
Some requirements are subtle; we have tried to highlight motivations
and reasoning behind the results you see here. I'm sure we've fallen
short; if you find any part confusing or wrong, please contact us and
help us improve.
-->

このページでは、その動機と理由を説明します。
ここに掲載されている結果は、その動機と理由を明らかにしたものです。しかし
もし、わかりにくいところや間違っているところがあれば、ぜひご連絡ください。
改善にご協力ください。

<!--
This is version 0.

1. [BOLT #1](01-messaging.md): Base Protocol
2. [BOLT #2](02-peer-protocol.md): Peer Protocol for Channel Management
3. [BOLT #3](03-transactions.md): Bitcoin Transaction and Script Formats
4. [BOLT #4](04-onion-routing.md): Onion Routing Protocol
5. [BOLT #5](05-onchain.md): Recommendations for On-chain Transaction Handling
7. [BOLT #7](07-routing-gossip.md): P2P Node and Channel Discovery
8. [BOLT #8](08-transport.md): Encrypted and Authenticated Transport
9. [BOLT #9](09-features.md): Assigned Feature Flags
10. [BOLT #10](10-dns-bootstrap.md): DNS Bootstrap and Assisted Node Location
11. [BOLT #11](11-payment-encoding.md): Invoice Protocol for Lightning Payments
-->

これはバージョン0です。

1. [BOLT #1](01-messaging.md): 基本プロトコル
2. [BOLT #2](02-peer-protocol.md): チャネル管理用ピアプロトコル
3. [BOLT #3](03-transactions.md): ビットコイン取引とスクリプトフォーマット
4. [BOLT #4](04-onion-routing.md): オニオンルーティングプロトコル
5. [BOLT #5](05-onchain.md): オンチェーン取引処理の推奨事項
7. [BOLT #7](07-routing-gossip.md): P2P ノードとチャネルの発見
8. [BOLT #8](08-transport.md): 暗号化と認証された転送
9. [BOLT #9](09-features.md): 割り当てられた機能フラグ
10. [BOLT #10](10-dns-bootstrap.md): DNS ブートストラップとアシストノードの位置関係
11. [BOLT #11](11-payment-encoding.md): ライトニング支払いのインボイスプロトコル

## The Spark: A Short Introduction to Lightning

<!--
Lightning is a protocol for making fast payments with Bitcoin using a
network of channels.
-->

ライトニングは、チャネルのネットワークを使ったビットコインで高速に決済するためのプロトコルです。

### Channels

<!--
Lightning works by establishing *channels*: two participants create a
Lightning payment channel that contains some amount of bitcoin (e.g.,
0.1 bitcoin) that they've locked up on the Bitcoin network. It is
spendable only with both their signatures.
-->

ライトニングは *チャネル* を確立することで機能します。：2人の参加者は、ビットコインネットワークにロックしたビットコイン（例：0.1BTC）を含むライトニングのペイメントチャネルを作成します。これは両者の署名がある場合のみ使用できます。

<!--
Initially they each hold a bitcoin transaction that sends all the
bitcoin (e.g. 0.1 bitcoin) back to one party.  They can later sign a new bitcoin
transaction that splits these funds differently, e.g. 0.09 bitcoin to one
party, 0.01 bitcoin to the other, and invalidate the previous bitcoin
transaction so it won't be spent.
-->

最初はそれぞれがビットコイン取引を行い、すべてのビットコイン（例えば0.1BTC）を一方の当事者に送り返します。
この後、彼らは新しいビットコイン取引に署名できます。
例えば、0.09ビットコインを一方の当事者に、0.01ビットコインをもう一方の当事者にというように。
0.09ビットコイン、0.01ビットコインといった具合に、この資金を分割する新しいビットコイン取引に署名し、前のビットコイン取引を無効にできます。

<!--
See [BOLT #2: Channel Establishment](02-peer-protocol.md#channel-establishment) for more on
channel establishment and [BOLT #3: Funding Transaction Output](03-transactions.md#funding-transaction-output) for the format of the bitcoin transaction that creates the channel.  See [BOLT #5: Recommendations for On-chain Transaction Handling](05-onchain.md) for the requirements when participants disagree or fail, and the cross-signed bitcoin transaction must be spent.
-->

チャネルの確立については、[BOLT #2: Channel Establishment](02-peer-protocol.md#channel-establishment)を参照してください。またチャネルを作成するビットコイン取引の形式については、[BOLT #3: Funding Transaction Output](03-transactions.md#funding-transaction-output)をご覧ください。参加者の意見が一致しない場合や失敗した場合の要件については、[BOLT #5: Recommendations for On-chain Transaction Handling](05-onchain.md)を参照し、相互署名されたビットコイン取引を消費する必要があります。

### Conditional Payments

<!--
A Lightning channel only allows payment between two participants, but channels can be connected together to form a network that allows payments between all members of the network. This requires the technology of a conditional payment, which can be added to a channel,
e.g. "you get 0.01 bitcoin if you reveal the secret within 6 hours".
Once the recipient presents the secret, that bitcoin transaction is
replaced with one lacking the conditional payment and adding the funds
to that recipient's output.
-->

ライトニングチャネルは２人の参加者間でのみ決済が可能ですが、チャネルを連結してネットワークを形成し、そのネットワークの全メンバー間で決済を可能にできます。そのためには、チャネルを追加できる条件付きの支払いという技術が必要です。
例：「６時間以内に秘密を明かしたら0.01ビットコインをプレゼント」
受信者が秘密を提示すると、そのビットコインの取引条件は条件付き支払いの無いものに置き換えられ、そのビットコイン取引は、その資金がその受信者の出力に追加されます。

<!--
See [BOLT #2: Adding an HTLC](02-peer-protocol.md#adding-an-htlc-update_add_htlc) for the commands a participant uses to add a conditional payment, and [BOLT #3: Commitment Transaction](03-transactions.md#commitment-transaction) for the
complete format of the bitcoin transaction.
-->

参加者が条件付き支払いを追加するためのコマンドは、[BOLT #2: Adding an HTLC](02-peer-protocol.md#adding-an-htlc-update_add_htlc)を、またビットコイン取引の完全な形式は[BOLT #3: Commitment Transaction](03-transactions.md#commitment-transaction)を参照して下さい。

### Forwarding

<!--
Such a conditional payment can be safely forwarded to another
participant with a lower time limit, e.g. "you get 0.01 bitcoin if you reveal the secret
within 5 hours".  This allows channels to be chained into a network
without trusting the intermediaries.
-->

このような条件付き支払いは、制限時間の短い別の参加者に安全に転送できます。
例えば、「５時間以内に秘密を明かせば0.01BTCが貰えます。」というように。これにより、仲介者を信頼することなく、ネットワークにチャネルを連結させられます。

<!--
See [BOLT #2: Forwarding HTLCs](02-peer-protocol.md#forwarding-htlcs) for details on forwarding payments, [BOLT #4: Packet Structure](04-onion-routing.md#packet-structure) for how payment instructions are transported.
-->

支払いの転送については、[BOLT #2: Forwarding HTLCs](02-peer-protocol.md#forwarding-htlcs)を、支払いの転送方法については[BOLT #4: Packet Structure](04-onion-routing.md#packet-structure)を参照してください。

### Network Topology

<!--
To make a payment, a participant needs to know what channels it can
send through.  Participants tell each other about channel and node
creation, and updates.
-->

参加者は支払いを行うために、どのようなチャネルで送信できるか知る必要があります。参加者は、チャネルとノードの作成と更新について互いに伝えます。

<!--
See [BOLT #7: P2P Node and Channel Discovery](07-routing-gossip.md)
for details on the communication protocol, and [BOLT #10: DNS
Bootstrap and Assisted Node Location](10-dns-bootstrap.md) for initial
network bootstrap.
-->

通信プロトコルの詳細は、[BOLT #7: P2P Node and Channel Discovery](07-routing-gossip.md)を参照してください。また、初期のブートストラップについては、[BOLT #10: DNS
Bootstrap and Assisted Node Location](10-dns-bootstrap.md)を参照してください。

### Payment Invoicing

<!--
A participant receives invoices that tell them what payments to make.
-->

参加者は、支払い内容を記した請求書を受け取ります。

<!--
See [BOLT #11: Invoice Protocol for Lightning Payments](11-payment-encoding.md) for the protocol describing the destination and purpose of a payment such that the payer can later prove successful payment.
-->

支払者が後で支払いの成功を証明できるように、支払先と目的を記述するプロトコルについては、[BOLT #11: Invoice Protocol for Lightning Payments](11-payment-encoding.md)を参照してください。

## Glossary and Terminology Guide

* #### *Announcement*:
<!--
   * A gossip message sent between *[peers](#peers)* intended to aid the discovery of a *[channel](#channel)* or a *[node](#node)*.
-->

   * *[チャネル](#channel)* or a *[ノード](#node)*の発見を助ける目的で*[ピア](#peers)*間で送信されるゴシップメッセージです。

* #### `chain_hash`:
<!--
   * The uniquely identifying hash of the target blockchain (usually the genesis hash).
     This allows *[nodes](#node)* to create and reference *channels* on
     several blockchains. Nodes are to ignore any messages that reference a
     `chain_hash` that are unknown to them. Unlike `bitcoin-cli`, the hash is
     not reversed but is used directly.
-->

   * ターゲットのブロックチェーンの一意な識別ハッシュ（通常はジェネシスハッシュ）。
     これにより、*[ノード](#node)*は複数のブロックチェーン上で*チャネル*を作成し、参照できる。
     ノードは自分にとって未知の`chain_hash`を参照するメッセージを無視しなければなりません。`bitcoin-cli`とは異なり、ハッシュは反転されず、直接使用されます。

<!--
     For the main chain Bitcoin blockchain, the `chain_hash` value MUST be
     (encoded in hex):
     `6fe28c0ab6f1b372c1a6a246ae63f74f931e8365e15a089c68d6190000000000`.
-->

     メインチェーンのビットコインブロックチェーンでは、`chain_hash`の値は次のようにしなければなりません。
     （16進数エンコード）
     `6fe28c0ab6f1b372c1a6a246ae63f74f931e8365e15a089c68d6190000000000`.

* #### *Channel*:
<!--
   * A fast, off-chain method of mutual exchange between two *[peers](#peers)*.
   To transact funds, peers exchange signatures to create an updated *[commitment transaction](#commitment-transaction)*.
   * _See closure methods: [mutual close](#mutual-close), [revoked transaction close](#revoked-transaction-close), [unilateral close](#unilateral-close)_
   * _See related: [route](#route)_
-->

   * 2つの*[ピア](#peers)* 間で相互交換を行う高速なオフチェーン手法
   資金を取引するために、ピアは署名を交換して更新された*[コミットメント取引](#commitment-transaction)* を作成します。
   * クローズ方法：[mutual close](#mutual-close), [revoked transaction close](#revoked-transaction-close), [unilateral close](#unilateral-close)_
   * _関連を参照してください。: [route](#route)_

* #### *Closing transaction*:
<!--
   * A transaction generated as part of a *[mutual close](#mutual-close)*. A closing transaction is similar to a _commitment transaction_, but with no pending payments.
   * _See related: [commitment transaction](#commitment-transaction), [funding transaction](#funding-transaction), [penalty transaction](#penalty-transaction)_
-->

   * *[相互クローズ](#mutual-close)* の一部として生成される取引。クローズ取引は _コミットメント取引_ に似ていますが、未決済の支払いはありません。
   * _関連を参照してください。：[commitment transaction](#commitment-transaction), [funding transaction](#funding-transaction), [penalty transaction](#penalty-transaction)_

* #### *Commitment number*:
<!--
   * A 48-bit incrementing counter for each *[commitment transaction](#commitment-transaction)*; counters
    are independent for each *peer* in the *channel* and start at 0.
   * _See container: [commitment transaction](#commitment-transaction)_
   * _See related: [closing transaction](#closing-transaction), [funding transaction](#funding-transaction), [penalty transaction](#penalty-transaction)_
-->

   * *[コミットメント取引](#commitment-transaction)* 毎に48ビット増加するカウンタです。；カウンタはチャネル内の *peer* で独立しており、0から開始します。
   * _コンテナを参照してください。: [commitment transaction](#commitment-transaction)_
   * _関連を参照してください。: [closing transaction](#closing-transaction), [funding transaction](#funding-transaction), [penalty transaction](#penalty-transaction)_

* #### *Commitment revocation private key*:
<!--
   * Every *[commitment transaction](#commitment-transaction)* has a unique commitment revocation private-key
    value that allows the other *peer* to spend all outputs
    immediately: revealing this key is how old commitment
    transactions are revoked. To support revocation, each output of the
    commitment transaction refers to the commitment revocation public key.
   * _See container: [commitment transaction](#commitment-transaction)_
   * _See originator: [per-commitment secret](#per-commitment-secret)_
-->

   * 全ての *[合意取引](#commitment-transaction)* は一意の合意失効秘密鍵を持っています。他の *ピア* が全ての出力をすぐに利用できるように、値を指定します。このキーを明らかにすることは、古いコミットメント取引を取り消す方法です。取り消しをサポートするために、コミットメント取引の各出力は、失効公開鍵を参照しています。
   * _コンテナを参照してください: [commitment transaction](#commitment-transaction)_
   * _起案者を参照してください: [per-commitment secret](#per-commitment-secret)_

* #### *Commitment transaction*:
<!--
   * A transaction that spends the *[funding transaction](#funding-transaction)*.
   Each *peer* holds the other peer's signature for this transaction, so that each
   always has a commitment transaction that it can spend. After a new
   commitment transaction is negotiated, the old one is *revoked*.
   * _See parts: [commitment number](#commitment-number), [commitment revocation private key](#commitment-revocation-private-key), [HTLC](#HTLC-Hashed-Time-Locked-Contract), [per-commitment secret](#per-commitment-secret), [outpoint](#outpoint)_
   * _See related: [closing transaction](#closing-transaction), [funding transaction](#funding-transaction), [penalty transaction](#penalty-transaction)_
   * _See types: [revoked commitment transaction](#revoked-commitment-transaction)_
-->

   * *[資金調達取引](#funding-transaction)* を消費する取引。各 *peer* はこの取引に対する他のピアの署名を保持しているため、それぞれが常に自分が使える合意取引を保持しています。新しい合意取引がネゴシエートされた後、古いものは *取り消されます* 。
   * _関連項目: [commitment number](#commitment-number), [commitment revocation private key](#commitment-revocation-private-key), [HTLC](#HTLC-Hashed-Time-Locked-Contract), [per-commitment secret](#per-commitment-secret), [outpoint](#outpoint)_
   * _関連を参照してください: [closing transaction](#closing-transaction), [funding transaction](#funding-transaction), [penalty transaction](#penalty-transaction)_
   * _型を参照してください: [revoked commitment transaction](#revoked-commitment-transaction)_

* #### *Fail the channel*:
<!--
  * This is a forced close of the channel. Very early on (before
  opening), this may not require any action but forgetting the
  existence of the channel.  Usually it requires signing and
  broadcasting the latest commitment transaction, although during
  mutual close it can also be performed by signing and broadcasting a
  mutual close transaction.  See [BOLT #5](05-onchain.md#failing-a-channel).
-->

   * これはチャネルの強制終了です。非常に早い段階（開店前）、任意のアクションを必要としないかも知れませんが、チャネルの存在を忘れています。通常これは最新のコミットメント取引に署名し、最新のコミットメント取引をブロードキャストする必要があります。相互クローズ中に、相互クローズとりひきに署名し、ブロードキャストして実行できます。[BOLT #5](05-onchain.md#failing-a-channel)を参照してください。

* #### *Close the connection*:
<!--
  * This means closing communication with the peer (such as closing
  the TCP socket).  It does not imply closing any channels with the
  peer, but does cause the discarding of uncommitted state for
  connections with channels: see [BOLT #2](02-peer-protocol.md#message-retransmission).
-->

   * これは、ピアとの通信を閉じることを意味します（TCPソケットを閉じるなど）。これは、相手とのチャネルを閉じることを意味するものではありません。しかしチャネルを持つコネクションのコミットされていない状態を破棄することになります。[BOLT #2](02-peer-protocol.md#message-retransmission)を参照してください。

* #### *Final node*:
<!--
   * The final recipient of a packet that is routing a payment from an *[origin node](#origin-node)* through some number of *[hops](#hop)*. It is also the final *[receiving peer](#receiving-peer)* in a chain.
   * _See category: [node](#node)_
   * _See related: [origin node](#origin-node), [processing node](#processing-node)_
-->

   * *[オリジンノード](#origin-node)* からある数の *[ホップ](#hop)* を通して支払いをルーティングしているパケットの最終受信者です。また、チェーンの最終的な *[受信ピア](#receiving-peer)* です。
   * _カテゴリを参照してください： [ノード￥](#node)_
   * _関連を参照してください: [origin node](#origin-node), [processing node](#processing-node)_

* #### *Funding transaction*:
<!--
   * An irreversible on-chain transaction that pays to both *[peers](#peers)* on a *[channel](#channel)*.
   It can only be spent by mutual consent.
   * _See related: [closing transaction](#closing-transaction), [commitment transaction](#commitment-transaction), [penalty transaction](#penalty-transaction)_
-->

   * *[チャネル](#channel)* 上の両方の *[ピア](#peers)* に支払う不可逆的なオンチェーン取引です。
   双方の合意によって使用できます。
   * _関連を参照してください。: [closing transaction](#closing-transaction), [commitment transaction](#commitment-transaction), [penalty transaction](#penalty-transaction)_

* #### *Hop*:
<!--
   * A *[node](#node)*. Generally, an intermediate node lying between an *[origin node](#origin-node)* and a *[final node](#final-node)*.
   * _See category: [node](#node)_
-->

   * *[ノード](#node)* のこと。一般に、*[origin node](#origin-node)* と *[final node](#final-node)* の間にある中間ノードのことを言う。
   * _カテゴリを参照してください。: [node](#node)_

* #### *HTLC*: Hashed Time Locked Contract.
 <!--
   * A conditional payment between two *[peers](#peers)*: the recipient can spend
    the payment by presenting its signature and a *payment preimage*,
    otherwise the payer can cancel the contract by spending it after
    a given time. These are implemented as outputs from the
    *[commitment transaction](#commitment-transaction)*.
   * _See container: [commitment transaction](#commitment-transaction)_
   * _See parts: [Payment hash](#Payment-hash), [Payment preimage](#Payment-preimage)_
-->

   * ２つの *[ピア](#peers)* 間の条件付き支払い：受取人は、署名と *支払いプレイメージ* を提示して、支払いに使え、そうでない場合、支払い者は与えられた時間の後に使う事によって契約をキャンセルできます。これらは *[合意取引](#commitment-transaction)* の出力として実装されます。
   * _containerを参照してください。: [commitment transaction](#commitment-transaction)_
   * _部分的に参照してください。: [Payment hash](#Payment-hash), [Payment preimage](#Payment-preimage)_

* #### *Invoice*: A request for funds on the Lightning Network, possibly
<!--
    including payment type, payment amount, expiry, and other
    information. This is how payments are made on the Lightning
    Network, rather than using Bitcoin-style addresses.
-->

   支払い種別、支払額、有効きげん、その他の情報を含みます。ライトニングネットワークでは、ビットコインのようなアドレスではなく、このような支払いが行われます。

* #### *It's ok to be odd*:
<!--
   * A rule applied to some numeric fields that indicates either optional or
     compulsory support for features. Even numbers indicate that both endpoints
     MUST support the feature in question, while odd numbers indicate
     that the feature MAY be disregarded by the other endpoint.
-->

   * ある数値フィールドに適用されるルールで、機能のサポートが任意か強制か示します。偶数番号は、両方のエンドポイントが当該機能をサポートしなければならない（MAST）ことを示します。奇数は、もう一方のエンドボインとが当該機能を無視しても良い（MAY）ことを示します。

* #### *MSAT*:
<!--
   * A millisatoshi, often used as a field name.
-->

   * フィールド名としてよく使われるミリサトシです。

* #### *Mutual close*:
<!--
   * A cooperative close of a *[channel](#channel)*, accomplished by broadcasting an unconditional
    spend of the *[funding transaction](#funding-transaction)* with an output to each *peer*
    (unless one output is too small, and thus is not included).
   * _See related: [revoked transaction close](#revoked-transaction-close), [unilateral close](#unilateral-close)_
-->

   * *[チャネル](#channel)* の協調的な終了のことで、無条件で *[資金取引](#funding-transaction)* の支出を各 *ピア* に放出する事によって達成される。
   （ただし、１つの出力が小さすぎるため、含まれないことを除きます）
   * _関連を参照してください。: [revoked transaction close](#revoked-transaction-close), [unilateral close](#unilateral-close)_

* #### *Node*:
<!--
   * A computer or other device that is part of the Lightning network.
   * _See related: [peers](#peers)_
   * _See types: [final node](#final-node), [hop](#hop), [origin node](#origin-node), [processing node](#processing-node), [receiving node](#receiving-node), [sending node](#sending-node)_
-->

   * ライトニングネットワークの一部であるコンピュータまたはその他のデバイス。
   * _関連を参照してください。：[ピア](#peers)_
   * _型を参照してください。: [final node](#final-node), [hop](#hop), [origin node](#origin-node), [processing node](#processing-node), [receiving node](#receiving-node), [sending node](#sending-node)_

* #### *Origin node*:
<!--
   * The *[node](#node)* that originates a packet that will route a payment through some number of [hops](#hop) to a *[final node](#final-node)*. It is also the first [sending peer](#sending-peer) in a chain.
   * _See category: [node](#node)_
   * _See related: [final node](#final-node), [processing node](#processing-node)_
-->

   * *[最終ノード](#final-node)* までいくつかの[ホップ](#hop)を経由して支払いをルートするパケットを発信する　*[ノード](#node)* のこと。また、連鎖の最初の [送信ピア](#sending-peer)でもあります。
   * _カテゴリを参照してください。: [node](#node)_
   * _関連を参照してください。: [final node](#final-node), [processing node](#processing-node)_

* #### *Outpoint*:
<!--
  * A transaction hash and output index that uniquely identify an unspent transaction output. Needed to compose a new transaction, as an input.
  * _See related: [funding transaction](#funding-transaction), [commitment transaction](#commitment-transaction)_
-->

   * 未使用の取引出力を一位に識別するトランザクションハッシュと出力インデックス。新しい取引を構成する為に、入力として必要。
   * _関連を参照してください。: [funding transaction](#funding-transaction), [commitment transaction](#commitment-transaction)_

* #### *Payment hash*:
<!--
   * The *[HTLC](#HTLC-Hashed-Time-Locked-Contract)* contains the payment hash, which is the hash of the
    *[payment preimage](#Payment-preimage)*.
   * _See container: [HTLC](#HTLC-Hashed-Time-Locked-Contract)_
   * _See originator: [Payment preimage](#Payment-preimage)_
-->

   * *[HTLC](#HTLC-Hashed-Time-Locked-Contract)* は、 *[payment preimage](#Payment-preimage)* の支払いハッシュを含んでいます。
   * _containerを参照してください。: [HTLC](#HTLC-Hashed-Time-Locked-Contract)_
   * _発信元を参照してください。: [Payment preimage](#Payment-preimage)_

* #### *Payment preimage*:
<!--
   * Proof that payment has been received, held by
    the final recipient, who is the only person who knows this
    secret. The final recipient releases the preimage in order to
    release funds. The payment preimage is hashed as the *[payment hash](#Payment-hash)*
    in the *[HTLC](#HTLC-Hashed-Time-Locked-Contract)*.
   * _See container: [HTLC](#HTLC-Hashed-Time-Locked-Contract)_
   * _See derivation: [payment hash](#Payment-hash)_
-->

   * 支払いを受けたことを証明するもので、この秘密は最終受領者が保有します。最終受領者は資金を放出します。プリイメージは、*[HTLC](#HTLC-Hashed-Time-Locked-Contract)* の中で *[支払いハッシュ](#Payment-hash)* としてハッシュ化されます。
   * _containerを参照してください。: [HTLC](#HTLC-Hashed-Time-Locked-Contract)_
   * _派生を参照してください。: [payment hash](#Payment-hash)_

* #### *Peers*:
<!--
   * Two *[nodes](#node)* that are in communication with each other.
      * Two peers may gossip with each other prior to setting up a channel.
      * Two peers may establish a *[channel](#channel)* through which they transact.
   * _See related: [node](#node)_
-->

   * 2つの *[ノード](#node)* が互いに通信している状態。
      * 2つのピアはチャネルを設定する前に互いにゴシップを行うことができます。
      * 2つのピアが *[チャネル](#channel)* を確立し、取引を行うことができます。
   * _関連を参照してください。: [node](#node)_

* #### *Penalty transaction*:
<!--
   * A transaction that spends all outputs of a *[revoked commitment
    transaction](#revoked-commitment-transaction)*, using the *commitment revocation private key*. A *[peer](#peers)* uses this
    if the other peer tries to "cheat" by broadcasting a *[revoked commitment
    transaction](#revoked-commitment-transaction)*.
   * _See related: [closing transaction](#closing-transaction), [commitment transaction](#commitment-transaction), [funding transaction](#funding-transaction)_
-->
   *  *[取り消された合意取引](#revoked-commitment-transaction)* の全ての出力を消費する取引で、 *合意破棄秘密鍵* を使用する取引。 *[ピア](#peers)* は、ブロードキャストすることで「不正」を行おうとする場合にこれを使用します。
   * _関連を参照してください。: [closing transaction](#closing-transaction), [commitment transaction](#commitment-transaction), [funding transaction](#funding-transaction)_

* #### *Per-commitment secret*:
<!--
   * Every *[commitment transaction](#commitment-transaction)* derives its keys from a per-commitment secret,
     which is generated such that the series of per-commitment secrets
     for all previous commitments can be stored compactly.
   * _See container: [commitment transaction](#commitment-transaction)_
   * _See derivation: [commitment revocation private key](#commitment-revocation-private-key)_
-->
   * 全ての *[合意取引](#commitment-transaction)* は、合意毎の秘密鍵から派生します。この秘密は、以前の全ての合意に対する一連の合意との秘密をコンパクトに保存できるように生成されます。
   * _コンテナを参照してください。: [commitment transaction](#commitment-transaction)_
   * _派生を参照してください。: [commitment revocation private key](#commitment-revocation-private-key)_

* #### *Processing node*:
<!--
   * A *[node](#node)* that is processing a packet that originated with an *[origin node](#origin-node)* and that is being sent toward a *[final node](#final-node)* in order to route a payment. It acts as a *[receiving peer](#receiving-peer)* to receive the message, then a [sending peer](#sending-peer) to send on the packet.
   * _See category: [node](#node)_
   * _See related: [final node](#final-node), [origin node](#origin-node)_
-->
   * *[オリジンノード](#origin-node)* から発信されたパケットを処理する *[ノード](#node)* で、支払いを仲介する為に *[最終ノード](#final-node)* に向けて送信されるものです。*[受信ピア](#receiving-peer)* としてメッセージを受信して、[送信ピア](#sending-peer) としてパケットを送信します。
   * _カテゴリを参照してください。: [node](#node)_
   * _関連を参照してください。: [final node](#final-node), [origin node](#origin-node)_

* #### *Receiving node*:
<!--
   * A *[node](#node)* that is receiving a message.
   * _See category: [node](#node)_
   * _See related: [sending node](#sending-node)_
-->
   * メッセージを受信している *[ノード](#node)* のこと
   * _カテゴリを参照してください。: [node](#node)_
   * _関連を参照してください。: [sending node](#sending-node)_

* #### *Receiving peer*:
<!--
   * A *[node](#node)* that is receiving a message from a directly connected *peer*.
   * _See category: [peer](#Peers)_
   * _See related: [sending peer](#sending-peer)_
-->
   * 直接接続されたぴあからメッセージを受信している *[ノード](#node)* のことです。
   * _カテゴリを参照してください。: [peer](#Peers)_
   * _関連を参照してください。: [sending peer](#sending-peer)_

* #### *Revoked commitment transaction*:
<!--
   * An old *[commitment transaction](#commitment-transaction)* that has been revoked because a new commitment transaction has been negotiated.
   * _See category: [commitment transaction](#commitment-transaction)_
-->
   * 新しいコミットメント取引が交渉された為、取り消された古い *[合意取引](#commitment-transaction)* のことです。
   * _カテゴリを参照してください。: [commitment transaction](#commitment-transaction)_

* #### *Revoked transaction close*:
<!--
   * An invalid close of a *[channel](#channel)*, accomplished by broadcasting a *revoked
    commitment transaction*. Since the other *peer* knows the
    *commitment revocation secret key*, it can create a *[penalty transaction](#penalty-transaction)*.
   * _See related: [mutual close](#mutual-close), [unilateral close](#unilateral-close)_
-->
   * *[チャネル](#channel)* の無効なクローズで、 *取り消されたコミットメント取引* をブロードキャストして達成されます。他の *ピア* は *コミットメント取り消し秘密鍵* を知っているので、*[ペナルティ取引](#penalty-transaction)* を作成できます。
   * _関連を参照してください。: [mutual close](#mutual-close), [unilateral close](#unilateral-close)_

* #### *Route*:
<!--
  * A path across the Lightning Network that enables a payment
    from an *origin node* to a *[final node](#final-node)* across one or more
    *[hops](#hop)*.
  * _See related: [channel](#channel)_
-->
   * ライトニングネットワークを横断する経路で、次のような支払いを可能にします。 *オリジンノード* から *[最終ノード](#final-node)* まで、１つまたは複数の *[ホップ](#hop)* 。
  * _関連を参照してください。: [channel](#channel)_

* #### *Sending node*:
<!--
   * A *[node](#node)* that is sending a message.
   * _See category: [node](#node)_
   * _See related: [receiving node](#receiving-node)_
-->
   * メッセージを送信している *[ノード](#node)* のことです。
   * _カテゴリを参照してください。: [node](#node)_
   * _関連を参照してください。: [receiving node](#receiving-node)_

* #### *Sending peer*:
<!--
   * A *[node](#node)* that is sending a message to a directly connected *peer*.
   * _See category: [peer](#Peers)_
   * _See related: [receiving peer](#receiving-peer)_.
-->
   * 直接接続されている *ピア* にメッセージを送信している *[ノード](#node)* のことです。
   * _カテゴリを参照してください。: [peer](#Peers)_
   * _関連を参照してください。: [receiving peer](#receiving-peer)_.

* #### *Unilateral close*:
<!--
   * An uncooperative close of a *[channel](#channel)*, accomplished by broadcasting a
    *[commitment transaction](#commitment-transaction)*. This transaction is larger (i.e. less
    efficient) than a *[closing transaction](#closing-transaction)*, and the *[peer](#peers)* whose
    commitment is broadcast cannot access its own outputs for some
    previously-negotiated duration.
   * _See related: [mutual close](#mutual-close), [revoked transaction close](#revoked-transaction-close)_
-->
   * *[チャネル](#channel)* の非協力的なクローズで、*[合意取引](#commitment-transaction)* をブロードキャストして達成されます。この取引は*[クロージング取引](#closing-transaction)* よりも大きく（すなわち効率が悪い）、合意がブロードキャストされた *[ピア](#peers)* は、事前に交渉された期間、自分の出力にアクセスできません。
   * _関連を参照してください。: [mutual close](#mutual-close), [revoked transaction close](#revoked-transaction-close)_

## Theme Song

      Why this network could be democratic...
      Numismatic...
      Cryptographic!
      Why it could be released Lightning!
      (Release Lightning!)


      We'll have some timelocked contracts with hashed pubkeys, oh yeah.
      (Keep talking, whoa keep talkin')
      We'll segregate the witness for trustless starts, oh yeah.
      (I'll get the money, I've got to get the money)
      With dynamic onion routes, they'll be shakin' in their boots;
      You know that's just the truth, we'll be scaling through the roof.
      Release Lightning!
      (Go, go, go, go; go, go, go, go, go, go)


      [Chorus:]
      Oh released Lightning, it's better than a debit card..
      (Release Lightning, go release Lightning!)
      With released Lightning, micropayments just ain't hard...
      (Release Lightning, go release Lightning!)
      Then kaboom: we'll hit the moon -- release Lightning!
      (Go, go, go, go; go, go, go, go, go, go)


      We'll have QR codes, and smartphone apps, oh yeah.
      (Ooo ooo ooo ooo ooo ooo ooo)
      P2P messaging, and passive incomes, oh yeah.
      (Ooo ooo ooo ooo ooo ooo ooo)
      Outsourced closure watch, gives me feelings in my crotch.
      You'll know it's not a brag when the repo gets a tag:
      Released Lightning.


      [Chorus]
      [Instrumental, ~1m10s]
      [Chorus]
      (Lightning! Lightning! Lightning! Lightning!
       Lightning! Lightning! Lightning! Lightning!)


      C'mon guys, let's get to work!


   -- Anthony Towns <aj@erisian.com.au>

## Authors

[ FIXME: Insert Author List ]

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).
