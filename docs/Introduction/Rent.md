# 存储租金经济
提交给Solana账本的每笔交易都会产生费用。交易费用由交易发起者支付，并由验证者来收取，理论上费用包含紧急的、交易的验证成本以及将该数据记录到账本。
在此过程中没有考虑的是活动账本状态的中期存储，它必须由轮换验证器集维护。随着活动状态的增长，这种类型的存储不仅会给验证器带来费用，还会给更广泛的网络带来费用，数据传输和验证开销也会增加。为了考虑到这些费用，我们在此描述我们对存储租金的初步设计和实施。
存储租金可以通过以下两种方式之一来支付：
方法1：设置后不管
通过这种方法，拥有两年租金押金的账户可以免收网络租金费用。通过保持这种最低余额，更广泛的网络将从减少的流动性中受益，账户持有人可以放心，他们将 Account::data 被保留以继续访问/使用。
方法2：按字节付费
如果一个账户的存入租金少于两年，则网络将按每个周期收取租金，以记入下一个周期的信用额度。这笔租金是按照规定的比率扣除的，单位是每千字节年。
有关此设计的技术实现详细信息的信息，请参阅[“租用”](https://docs.solanalabs.com/implemented-proposals/rent)部分。
注意：新创建的账户必须预充值足够数量的 lamports，以达到租金豁免标准。Lamports是SOL的最小单位，本次更新规定所有新账户创建时都需要预存一定量的lamports，使其满足租金豁免条件。这样可以避免小额账户长期占用网络资源却无需支付租金的情况。任何交易，如果会导致账户余额低于租金豁免最低值（且该余额并非零），都将失败。现有账户的租金支付规则保持不变，但任何会使账户余额低于租金豁免最低值的交易都将被拒绝。简而言之，此更新基本实现了所有账户的租金豁免。之前创建的需要支付租金的账户，会继续按照原有规则支付租金，直到满足以下两个条件之一：（1）账户余额归零。（2）交易使账户余额增加，达到租金豁免标准。