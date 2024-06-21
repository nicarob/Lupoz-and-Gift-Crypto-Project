# Smart Contracts project - Lupoz & Present Smart Signature

## Introduction

This project focuses on several essential functionalities within the Algorand blockchain. It begins with the generation and management of Algorand accounts, ensuring secure storage and handling of credentials. Then, it demonstrates the creation and management of an Algorand Standard Asset (ASA), named "Lupoz." Finally, the implementation of a smart signature written in PyTeal, which enforces specific conditions for transaction execution, particularly to allow a specific user to redeem a preset amount of Algos under specified conditions as a present. These operations highlight Algorand's capabilities for secure, decentralized financial transactions.

## Lupoz Asset

Our new asset called "Lupoz" is created on the Algorand blockchain. The total supply of Lupoz is 1000 units, which can be divided into smaller units up to six decimal places, meaning each microunit equates to 0.000001 Lupoz. Therefore, the total supply in the smallest units is 1 billion microLupoz (1000 * 10^6). The MyAlgo account is designated for several roles with respect to the asset:
- **Manager:** Can change the other roles
- **Reserve:** Holds undistributed tokens
- **Freeze Account:** Can freeze tokens if needed
- **Clawback Account:** Can undo transactions

## Smart Signature for a Present - Features

The smart signature we created is designed to allow a specific user, in our case Bravo, to redeem a certain amount of Lupoz as a present, after a certain date. The specifications are defined by the user that creates the signature: Alfa, in our case. The transaction must be a payment transaction where Bravo is the receiver; the transaction must involve the correct asset, Lupoz coin, and the redemption can only occur on or after the specified date before deploying the signature on the blockchain.

One important aspect of the logic of the signature is the ability to opt-in to the asset we created. On the Algorand blockchain, in order to avoid users being spammed with small amounts of worthless assets, every account needs to opt-in to an ASA. This also applies to smart signatures; it’s done by creating a transaction with yourself amounting to 0 coins of the asset you are interested in. For this reason, we also implemented a separate logic related to the opt-in transaction that the signature needs to make, using the `Or()` logic function in PyTeal, to allow either the opt-in transaction or the redeem transaction to happen without conflicts in the logic of the signature.

## Smart Signature for a Present - Limitations

Smart signatures in the Algorand Blockchain are also defined as *Stateless* Smart Contractsis, *stateless* meaning they do not store any state between transactions. In our case, this means that we cannot rely directly on datetime in the logic of our smart signature. One workaround we employed is using the *block number* as a point of reference. By taking the difference between the Algo genesis time and the present date - in Unix time - and then dividing it by the average rate of seconds per block, it’s possible to obtain an estimate of the block number corresponding to the date of the present. We are aware of the limitations and inconsistencies of such design, but it proved effective and reliable enough throughout our testing. 
 In the short-run there are no issues of doing such transformation, if not small inconsistencies related to the hour of the day in which the present will be available for the taking first. Issues could come-up if the present is for a distant future, because the average seconds-to-block rate of the Algorand blockchain could change over time. Right now it changes with dynamic rate depending on the traffic, but the average is stable overall. Changes in protocol could cause issues since they change the average block generating speed.

 A more complex design robust to these issues would involve a stateful smart contract, which would be able to take the datetime directly as a reference point.

### Security

There are several security concerns related to a smart signature. Given its rigid logic structure, a malicious actor might find some workaround to the logic, potentially stealing or disrupting the present. We researched the most common attacks and implemented logic checks to block them:

1. **Ensuring Transaction Type**
   - **Attack Type:** Transaction Spoofing
   - **Description:** An attacker might try to send a different type of transaction (e.g., asset transfer, application call) to manipulate the contract.
   - **Prevention:**
     - **Code:** `Txn.type_enum() == TxnType.Payment`
     - **Explanation:** This check ensures that the transaction being processed is strictly a payment transaction, preventing any other transaction types from being accepted and executed.

2. **Verifying Receiver Address**
   - **Attack Type:** Unauthorized Receiver
   - **Description:** An attacker could redirect funds to an unauthorized address.
   - **Prevention:**
     - **Code:** `Txn.receiver() == Addr(receiver)`
     - **Explanation:** This check ensures that the receiver of the transaction is the intended recipient, preventing funds from being diverted to an unauthorized address.

3. **Preventing Close-to Address Exploitation**
   - **Attack Type:** Close-to Attack
   - **Description:** An attacker might close the transaction and send the remainder to another address, potentially siphoning off funds.
   - **Prevention:**
     - **Code:** `Txn.close_remainder_to() == Global.zero_address()`
     - **Explanation:** This check ensures that there is no remainder being sent to any address other than the zero address, thereby preventing any unintentional or malicious transfer of remaining funds.

4. **Ensuring Correct Coin Type**
   - **Prevention:**
     - **Code:** `Txn.xfer_asset() == Int(lupoz_index)`
     - **Explanation:** This check ensures the transaction is dealing with the correct asset.

5. **Validating Round Number for Christmas Timing**
   - **Prevention:**
     - **Code:** `Int(round_number) <= Int(present)`
     - **Explanation:** This check ensures that the transaction can only be processed if the current round number is equal to or after the specified Christmas round, enforcing the timing restriction for redemption.

6. **Preventing Rekeying**
   - **Attack Type:** Rekeying Attack
   - **Description:** An attacker could rekey the signature to gain control over it.
   - **Prevention:**
     - **Code:** `Txn.rekey_to() == Global.zero_address()`
     - **Explanation:** This check ensures that the transaction does not attempt to rekey the account, preventing unauthorized changes to the account's authorization key.

7. **Ensuring Correct Transaction Amount**
   - **Prevention:**
     - **Code:** `Txn.amount() <= microalgo`
     - **Explanation:** This check ensures that the transaction amount is exactly as specified, preventing any deviations that could be exploited for financial gain.

8. **Limiting Transaction Fee**
   - **Attack Type:** Fee Manipulation
   - **Description:** An attacker could set an exorbitant transaction fee to drain funds or disrupt contract execution.
   - **Prevention:**
     - **Code:** `Txn.fee() <= Int(1000)`
     - **Explanation:** This check ensures that the transaction fee does not exceed 1000 microAlgos, preventing excessive fees that could otherwise deplete the contract's funds.

## Credits 

- Nicola Robatto
- Edoardo Lupotto
- Brenda Navarrete
  
## References:

Peter Gruber, *Writing Smart Contracts on Algorand (WSC)*. https://github.com/peterhgruber/writingsmartcontracts_algo

Algorand, *Modern guidelines for smart contracts and smart signatures on Algorand*. https://developer.algorand.org/docs/get-details/dapps/smart-contracts/guidelines/

OpenAI, *ChatGPT*. https://www.openai.com/chatgpt

GitHub, *GitHub Copilot*. https://github.com/features/copilot
