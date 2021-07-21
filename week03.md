# Plutus-Learning
Taking notes from the Plutus-pioneer program - Second Cohorts

**PLUTUS PIONEER PROGRAM WEEK 3**

**Documentation Week 3**

**Creator: Oshomah Abubakar**

**Recap of the last lecture**
1. In order to unlock the script address. The script accepts 3 inputs. The Datum, Redeemer and Context
2. The Datum and Redeemer can be custom types as long as they implement the isData typeclass 
3. The type context can be ScriptContext. 


**This week's lecture** 
This week's lecture will focus on the Context. 

![image](https://user-images.githubusercontent.com/51214370/126029083-e9eecd88-1bda-416e-82da-88a7bfe95019.png)

**How to access the package that hosts the ScriptContext**
1. Go to *plutus* monorepo and checkout to desired commit.
2. Run *nix-build -A plutus-playground.haddock* and wait till it's done. 
3. Last line in the terminal will be output directory, something like */nix/store/bqrsjhrpiygw5f326cjmkj1g08h7y9lz-haddock-join*
4. Copy the link: file:///nix/store/6nz16h032341npjs97d93wlbx6bwfl5h-haddock-join/share/doc/plutus-ledger-api/html/Plutus-V1-Ledger-Contexts.html into your browser. 

**ScriptPurpose datatype**

- The most important purpose defined under the ScriptPurpose is the **Spending TxOutRef** in the context of the (e)UTxO model. This is when a script is run in       order to validate spending input for a transaction. 

- The second most important one is the **Minting CurrencySymbol**. It comes into play when you want to define a native token. 
  It helps describe under which circumstances in which a token can be minted or burnt. 

- The two more purposes under the ScriptPurpose datatype are the **Rewarding StakingCredential** which can be related to staking and the **Certifying Dcert**       which is related to delegating. 

We will concentrate on the spending purpose the most. 


**THE CONTEXT**

The context, *TxInfo* can be found in the same module as the *ScriptPurpose* datatype. *TxInfo*(Transaction Info) descripes the spending transaction. The UTxO model that Cardano uses. The context of validation is the spending transaction and that is expressed in the *TxInfo* Data type.   

![image](https://user-images.githubusercontent.com/51214370/126030700-a5aeb829-a8c8-4e9d-8bde-44cc36113f57.png)

One of the biggest advantages Cardano has over other blockchains like Ethereum is that validation can happen in the wallet. Transactions can still fail because a it can consume an input because it has been consumed by someone else when it arrives at the node for validation. What should never happen under normal circumsatances is a validation script runs and then fails because you can always run a script under the same conditions in the wallet. So it fails before you submit to the node for validation. 

Managing time in that context is tricky. Why? Because when you bring your set a time frame in your wallet, it doesn't reconcile with the time that it gets to the node for validation or when the transaction will be validated. The way Cardano solves this dilema is by adding the *POSIXTimeRange* field in the *TxInfo* datatype. It gives a valid time interval for a transaction. And it is specified in the transaction. 

When a node is validating, it checks for the current time and the time in the transaction and compares them. If the times don't match, the validation fails immediately. Without running the Validator scripts. The trick is to do a time check before validation is run. 

By default all transactions use the infinite time range. Which in this context starts from the genesis block and lasts for all eternity. 

Ouroboros doesn't use *POSIXTimeRange*. It uses slots and each slot has a lottery system where a slot leader is selected to produce the next block. Plutus on the other hand uses real time. So this has to be converted back and forth between Plutus and Ouroboros time. As long as the slot time is fixed, this won't be an issue. 

So to avoid a scenario where slot time is changed. We shouldn't set slot upper bounds that are set too far into the future. Because we have no idea if the slot time will change. 

**POSIXTimeRange**
Unbder this will explore the **Interval** constructor. 
![image](https://user-images.githubusercontent.com/51214370/126440370-71732b5e-0c64-4ec8-af3a-722073298f5a.png)
We can look at both bounds which are the LowerBound and UpperBound. 

![image](https://user-images.githubusercontent.com/51214370/126440732-3dab2fe0-4563-4db5-9c6f-014d7e7a7cb3.png)
Drilling into them we can check out the type **Closure**. Which is another name for Bool.  

Then we look at the **Extended** type. The **Extended** can just be an **Finite a** constructor. Or it can be a negative infinity(**NegInf**) or positive infinity(**PosInf**) 
![image](https://user-images.githubusercontent.com/51214370/126441133-bba0254f-442a-4f3b-b78d-97380aa9f338.png)

Here we can see there are different convenient functions. 
![image](https://user-images.githubusercontent.com/51214370/126441849-b1d4b54c-39dc-4259-8362-5571c84b3135.png)


We can play with intervals a bit: 
1. Run cabal repl in this week's directory 
2. Run import Plutus.V1.Ledger.Interval
3. Test Time range with this an example: 
   **interval (10 :: Integer) 20**
   You can check if 9 is a member of the interval by running: 
   **member 9 $ interval (10 :: Integer) 20**
   Test out other functions like *from*, *to*, *intersection* :  
   **member 11 $ from (30 :: Integer)** 
   **member 70 $ to (30 :: Integer)**  
   **intersection (interval (10 :: Integer) 20) $ interval 18 30** (This takes an interval  )
   **contains (to (100 :: Integer)) $ interval 30 80** (This returns a Bool(True or False) when it takes an interval that stops at 100 that's between 30 to 80)

**TESTING THE TIME RANGE OUT** 
This is the first example of our first validator that actually uses the context. The idea is to put money into a script and only one person can receive the money only when a certain period has been reached. 

**First step** 
Think of the type of Datum and Redeemer. It makes sense to have the two of Datum( declared as VestingDatum) datatypes which are: 
1. Beneficiary 
2. Deadline 

![image](https://user-images.githubusercontent.com/51214370/126460872-0ade4bdd-7d55-4ffc-a7c6-363cb2dea9a5.png)

**Second step**
We will then define the **mkValidator** script. For the Redeemerwe will need to know if it is signed by the beneficiary and if it was submitted after the deadline. Since the information is in the transaction, we can just declare it as unit **()**. 

To understand how the TxInfo was gotten 


![image](https://user-images.githubusercontent.com/51214370/126463671-ad9c7acb-9659-454f-a18d-382d540815bf.png)

![image](https://user-images.githubusercontent.com/51214370/126462784-9c42e2f1-01be-4550-9178-19f155b87d33.png)

To also understand the way the txSignedBy function works. You can go on repl to check :t txSignedBy and you'll get
// txSignedBy :: TxInfo -> PubKeyHash -> Bool 

Now 




