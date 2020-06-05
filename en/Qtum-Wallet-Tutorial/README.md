

# Simplechain QT Wallet Tutorial

### My Wallet

This is the Simplechain wallet “**My Wallet**” page, it shows the following:

![mywallet](1.png)



### Balances

- Available
- Pending
- Total


#### Other Tokens

If we own any tokens, they'll show here, including balance and contract info.

### Transactions  

This shows any transaction that may have taken place, both incoming and outgoing transactions are displayed, this includes staked coins as well.



## Basic wallet functions

On the left side of the wallet, we can find basic wallet functionality such as:

- Send
- Receive
- Smart Contract
- Transactions (This is a more detailed and full historical set of transactions)



## Send Menu

On the send menu, we have different functions, all are quite basic to learn and use.

![Receive](2.png)



**Pay to:** Here we’ll enter the address we want to send Simplechain to **(Only Simplechain ignition addresses will work).**

**Label:**  Optional, as it’s stated, it’s just a name/tag

**Amount: **Here we enter the amount of Simplechain we want to Send

Once all fields are covered, we can click on the “Send” button and the Simplechain coins will be sent to the address we entered above.



## Receive

Here we can display our Simplechain wallet address and even generate new ones!

This is useful for receiving payments from more than one sender and helps you keep track of each transaction.

![Receive](3.png)

To obtain the deposit address, simply click on “Request Payment” and you’ll see a popup dialog show up with your Simplechain wallet deposit address. Here you can receive coins from your exchange or from other users. For this tutorial, we’ll do a test deposit of 54.9 Simplechain. 



## Transactions

Here you can see both incoming and outgoing transactions, shows every historical transactions.



![Receive](4.png)



## Wallet Encryption

First, let’s encrypt this wallet before making the backup, this will allow the backup to be encrypted as well. Go to settings -> Encrypt wallet.

![Receive](5.png)

![Receive](6.png)



Next, let’s enter the password for this wallet, **please remember this password because without it you’ll lose access to your Simplechain**.

![Receive](7.png)

Lastly, we get a warning reminding us to save/remember this passphrase as there's no way to recover your Simplechain without it

![Receive](8.png)

NOTE: On Simplechain 0.18x, the wallet doesn't restart after encrypting. After you see the above message, the wallet is already encrypted and ready to use.



## Unlock Wallet

To unlock the wallet for sending and private key commands use Settings - Unlock Wallet to reach the Unlock wallet page.
![Receive](9.png)

Use the Unlock wallet page to completely unlock the wallet(uncheck the For staking only box). To unlock the wallet for staking only, follow the procedure for Staking below. **NOTE** If the wallet is unlocked for staking only on this Unlock wallet page, the Stake menu **Staking button must also be clicked to the right to activate staking**.

![Receive](10.png)





## Wallet Backup

It's recommended to make the backup **after** encrypting the wallet.

![Receive](12.png)

To access the backup option, we go into File/Backup Wallet and then we enter route and name.

After the backup’s name is entered and we’ve verified the route where the wallet is being saved to, we can click on save. Now we’ve successfully backed up our wallet.



## Restore wallet backup

Go to File -> Restore wallet, this will open a dialog which will allow us to restore directly from the Simplechain-core QT wallet!



![Receive](13.png)

![Receive](14.png)

Here we selected the wallet backup with the reindex option selected (salvagewallet can also be used if necessary).



## Staking

In order for staking to take place we must:

1. Have a transaction with more than 500 confirmations on the blockchain
2. Unlock our wallet prior to staking (preferably before reaching 500 confirmations on your inputs).
3. Keep our wallet open at all times, otherwise even if our coins are matured, they won’t be able to stake.

Here’s how to unlock the wallet for staking only, starting with version 0.18.2. Select the main menu "Stake” option to show the Stake page. Click the Staking button to the right, then enter the passphrase to unlock for staking only and activate staking.

**IMPORTANT**: If unlocking the wallet using Settings – Unlock wallet and the Unlock wallet page, the Stake menu Staking button must also be clicked to the right to activate staking.




![Staking button](https://user-images.githubusercontent.com/29760787/70871569-4bfda880-1f6e-11ea-9db1-f5df2fcc0251.jpg)



**Remember to have your wallet open 24/7, if you close it, you won’t be able to stake**

![Receive](11.png)

**Please make sure “for staking only” is selected.**



## Access the console tab

On Simplechain 0.18x, the  console tab can be accessed from the "Windows" menu, as shown here:

![Receive](15.png)

![Receive](16.png)

The console tab is incredibly useful, although it's mostly for advanced uses, it is a very powerful tool and can perform actions that aren't really possible using the graphical interface. 

This document https://docs.Simplechain.site/en/commands/ explains in a very clear way how to use the console tab.
