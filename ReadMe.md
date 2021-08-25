# Agenda
- What is Daml ([docs](https://docs.daml.com/#))
   - Daml templates
   - Choices
   - Propose Accept Pattern
- Learning resources
    - Daml Docs [link](www.docs.daml.com)
    - Community Forum [link](discuss.daml.com)
    - Video Courses [link](https://daml.talentlms.com/catalog/info/id:132)

- <i>What we don't go into details in this walkthrough</I>
   - Frontend code for UI
   - Details on Daml functions
   - Detailed comparisons on permissioned / permissionless networks

# Download SDK [Here](https://docs.daml.com/getting-started/installation.html)


## Key Features
<i>Content taken from Episode 2.1</i>

End goal is to create the smart contracts that will be the core of the NFT platform.



1. Participants must be explicitly permitted 
    - userAdmin.daml
    - Issuer template, owner template, issuerRequest, ownerRequest
2. Content creators can mint NFTs
3. Owners can sell or transfer their NFTs

What do Smart contracts do? 
- Smart contracts reprsent who owns what digital good

- Smart contracts define who can change ownership
- Essentially how are the records being added

<b>Daml scripts can test your business logic</b> see `MainFinal.daml`

---
## Daml Basics

Daml is an open source smart contract language. You can write the smart contracts once, and then choose to deploy them on various blockchain infrastructures or centralised databases.

## Templates
<b>Templates</b> are a core concept in Daml.



### Defines the behavior 
- Who can see what.
- Who can alter the ledger.
- How can they alter the ledger.
- Under what conditions can information be altered.


## Breaking down a `template`

The below is an example of a `Token` template 

```haskell
template Token 
  with
    issuer: Party -- artist
    owner: Party -- you
    description: Text -- Catpic.jpg
    userAdmin: Party -- Platform owner
    lastPrice: Decimal -- 100.00
    currency: Text -- HKD
  where
    signatory -- Consented in contract creation
      issuer, 
      userAdmin

    controller owner can
      Offer: (ContractId TokenOffer)
        with
          newOwner: Party
          price: Decimal
        do 
          create TokenOffer with .. 
```

<b>templates</b> must contain
- Data fields in the <b>with</b> block
- Must contain at least one <b> party</b>
- At least one <b>signatory</b>
- `template` is a reserved word used to initalize a new template type, followed by the template name. In this case it is TokenOffer

## Signatory
Reserved keyword `signatory`

```haskell
  where
    signatory 
      issuer, 
      userAdmin
```

- It is a participant in a contract which <b>consented</b> in the creation of that template. 

- In the above case, in order for the above contract to be created, we need the permission of the <b>issuer</b>, <b>userAdmin</b>

- The issuer must have consented to the minting of the token
the admin makes sure that all parties that are acting have been KYCed 

- Signatory is someone that must have consented in the past. 

- In the past must have given their consent to this creation. 

## Observer
Reserved keyword `observer`

This is a way to make other parties aware of the contract. New owners must be able to see the contract. 

<i>See passport example</i>

In this case the observer keyword is redundant because `newOwner` is used in signatory. But the main point is oberserver is an addtional party, they just get to see it. 

## Choices 
`AcceptToken`, they are important, they spell out how the ledger can be mutated. 
- who can do this? 

```haskell
    controller owner can
      Offer: (ContractId TokenOffer)
        with
          newOwner: Party
          price: Decimal
        do 
          create TokenOffer with .. 
```

Most of the interesting stuff happens in the body of the choice. 

Most of the ledger updates will be udpated through choices. A result of exercising a choice.

All choices must indicate who can exercise them. Who can actually execute the body of a choice
 

And then the name of the choice
`Offer`, followed by the return type

Lastly, we have the body of the choice. Here is where we spell out waht happens, and how we want to mutate the ledger, or the workflow

in this case, we simply create a new token, `create token`, 
By default, a contract will archive, or be removed from the active state of the ledger, or you can use the keyword `nonconsuming` to prevent that. 

--- 

## Key

specifies that this triple, in the example above, `(issuer,owner, description)` must uniquely idenifty a token offer, these are 3 fields that will lead to a unique `tokenOffer`
it is a guarantee of uniqueness. And we have the d
## Maintainer
must refer to the key to figure out which party is responsible for maintaining the key. Or which party is responsible for ensureing the uniqueness 

---

## Analogies to `templates`
you can think of `templates` as tables in a sql database


| id | issuer | owner | description |
|--- | --- | --- | --- |
|0|Max|Bob|Novel text|
|1|Alice | Eve| another text| 


- template --> table

- data fields --> columns

- active ledger state --> all the tables in a schema

- choices -> stored procedures. It's atomic in the sense that if anything fails, the database gets rolled back. 

- key -> unique key

- party -> user

- Signatory, observer, controller, row level permissions

# Daml Hands-on (From E 2.2)

## Key features of Private NFT
1. Participants must be explicitly permitted 
    - userAdmin.daml
    - Issuer template, owner template, issuerRequest, ownerRequest
2. Content creators can mint NFTs
3. Owners can sell or transfer their NFTs

## Templates to code
### Part 1
1. `Token`
2. `TokenOffer`
3. `Issuer`
4. `IssuerRequest`
5. `Owner`
6. `OwnerRequest`
### Part 2
7. `Payment`

# Workflow Summary
<i>Diagram coming soon</i>

Because this is a closed system, there is a `userAdmin` that grants users rights to exercise choices. 

The choices are assocated with the user roles.

`Token` contract can only be created by an `Issuer`

Not anyone is an issuer. A user signs on, and requests to be an issuer (Artist, or owner/purchaser)

`Issuer` contract is only generated from an `IssuerRequest`

A user (Party) will submit an `IssuerRequest` (This step is the explicity onboarding, we are explicty giving this party the role of an issuer)

User Party submits `IssuerRequest`
User Admin <i>GrantsIssuerRights</i>, which results in the creation of `Issuer` contract. From here, `Issuer` can create `Token`

Party (not contract) `-->` `IssuerRequest` --> `Issuer` --> `Token`

1. Party creates request to be owner / issuer
2. User admin grants role contract
3. User (as Issuer) can mint token
4. Issuer can offer token to owner / purchaser, results in an offerRequest
5. Owner can reject / accept this request

# Project Directory Set up

In the console, create new directory:

1. `mkdir private-nft`

2. `cd private-nft`

Inside the `private-nft` directory is where we will create the daml project.

The reason for that is we will put test scripts, make-files and UI in the top level directory. 

`daml new nft`

this will create the template "skeleton"

`cd nft` into the directory

`daml studio`

This will open a new visual studio code window, and now you'll see the template

Here you can see in `daml > Main.daml`, line 24, click "script Result" and you'll see the table on the right hand side. 

# 1. Create `Token.daml`
1. in the `daml` directory, create new file named `Token.daml`


Recall feature: 
- Content creators can mint NFTs

Take a second and think, what are the rights and obligations of the final object. Which is the token, or the NFT. This way, we can think of what workflows are needed to create this final object. 

So we can create the final object first. 
Templates define the behavior, datamodel and rights and obligations
Starting with the data model, which is after teh `with` statement. 
- what is the data associated with the token? 
- If we are thinking in sequel table, what are the columns that this must contain. 

- Token must have an issuer of type `Party`
- owner, type `Party`

After filling out the data types, data fields after the `with` block. we need to think what are the rights & obligations surrounding this token. unlike publicly issued NFT, I want someone resopnsible for this token. I want the issuer to be on the hook to honoring the NFT. to do that, we designate a signatory. 
The NFTs work like autographs, we want to prove that some point, the issuer has signed off on it (to prove provenance) 

## Important
However, there is a concept in daml, where you can never be an obligable party on a contract, without your knowing and informed consent. 

This means, if I designate the issuer as the signatory, thismeans the only way that the token can exist is if the issuer signed off on it. This makes sure that the tokens that you ahve are signd off by the people that. 

`useAdmin` added to the signatory. The role of the user admin is to make sure that the issuer and the owner have passed the KYC procedures. 

## controller
In the below, where we start with `controller owner can`
```
controller owner can
  Offer: ContractId TokenOffer
    with
      newOwner: Party
      price: Decimal
    do
      create TokenOffer with .. 

```
In the do block, we create a token offer with the current variables in the scope. 

Now we model the rights and behaviors. 
someone has the right to do something if they can mutate the state of the ledger. 

```haskell
module Token where

-- Used for testing
import Daml.Script 

template Token 
  with
    issuer: Party
    owner: Party
    description: Text
    userAdmin: Party -- this is the party responsible for making the issuer and owner properly onboarded to the network. 
    lastPrice: Decimal -- keep track of the worth of the item (last purchase price)
    currency: Text
  where
    -- at this point, we need to think what are the rights & obligations surrounding this token. 
    -- unlike publicly issued NFT, I want someone resopnsible for this token
    -- I want the issuer to be on the hook to honoring the NFT
    -- to do that, we designate a signatory
    {-
    if the userAdmin was not the signatory, any party on the network can issue their own token. 
    userAdmin makes sure that party identities are legitamte
    -}
    signatory 
      issuer, 
      -- Try to comment userAdmin out then run script
      userAdmin

    -- now we model the rights and behaviors. 
    -- someone has the right to do something if they can mutate the state of the ledger
    controller owner can -- which will result in an offer template 
      Offer: (ContractId TokenOffer) -- not created yet
        with
          newOwner: Party
          price: Decimal
          
        do 
          create TokenOffer with .. --with all variables in current scope

```

After the above, you will see errors from `tokenOffer` that's because we haven't created the `TokenOffer` template. We'll do that next.

Daml is polite, we cannot say 

"Here, take my money", (which means direct transfer of ownership)

We say "Would you like to take my money?", === Propose accept pattern

# Create `TokenOffer` template
In the same file, `Token.daml`, underneath the token template, create the `TokenOffer` template 

Expresses the idea that we're giving the option to a new owner to take ownership of the token. It is an option, but not forcing the new owner to have ownership. 

There is a price involved, so the new owner must accept the price for the token to transfer ownership. 

Hence it is an offer to accept / reject the token offer. 
It's going to have the same data fields as `Token`, along with 


The `acceptToken` choice requires the owner and the `userAdmin` sign off. However it is unlikely that the `owner` and the `userAdmin` are coming from one endpoint. 
What we can do is setup a rights contract, that allows a permitte issuer that allows to act on behalf as user admin and himself

```haskell
template TokenOffer
  with
    issuer: Party
    owner: Party
    description: Text
    userAdmin: Party -- this is the party responsible for making the issuer and owner properly onboarded to the network. 
    newOwner: Party -- NEW field (these are the attributes from the offer choice)
    price: Decimal
    lastPrice: Decimal
    currency: Text
  where
    -- these are the signatories. Issuer is signign off, userAdmin makes sure newowner and old owner are legit parties
    -- old owner must be a sig becaues it means they signed off on potentailly letting the new owner be the owner
    signatory issuer, userAdmin, owner

    -- We want to be able to get a unique token offer based on the key
    key (issuer, owner, description) : (Party, Party, Text)

    -- Maintainer is somebody who is responsible for making sure that the key is unqiue 
    -- it needs to be one of the parties that are in the key
    -- the best one is the current owner of the offer, since they own the offer
    maintainer key._2

    -- newOwner can exercise the choice called Accept token 
    -- this generates a new contract with type Token
    -- we add 'userAdmin' to the controller, so that the acceptance must be approved
    -- called a conjuctive choice
    controller newOwner, userAdmin can 
      AcceptToken: ContractId Token
        do
          create Token with
            -- newOwner is now the owner
            owner = newOwner
            lastPrice = price
            .. -- like a JS spread operator
```
This is quite far, it already allows a market place that allows tokens to be transferred from one party to another. 

At this point, you'll notice that there are no transfers of money yet.
We've created the `Token` template, and the `TokenOffer`

Add the testing scripts, you wouldn't be able to create a token contract right off the bat because to create one we need the `issuer` and the `userAdmin`

But lets take a look at the testing scripts

```haskell
setup : Script ()
setup = script do
  alice <- allocatePartyWithHint "Alice" (PartyIdHint "Alice")
  bob <- allocatePartyWithHint "Bob" (PartyIdHint "Bob")

  bobToken <- submit bob do 
    createCmd Token 
      with  
        issuer = bob
        owner = bob
        description = "This is a new NFT"
        userAdmin = alice
        lastPrice = 100.0
        currency = "HKD"

  return ()
```

Try to commented out `userAdmin` then bob can create the token contract. See the script results.

# Review
We created 
- `Token` template 
- `TokenOffer` template. 

## Problem:
But with the current set up, bob cannot create a token without a `useAdmin` approval. 

We need to create another template that captures the userAdmin approval. This is done through giving the parties user role contracts. 



For example bob, at the moment is a party with no additional rights, such as the right to mint a token. 

If we use the script above, there will be an error due to `userAdmin` being in the signatory. Try to comment out `userAdmin` from the `Token` template. 

# 2. Create UserAdmin Template
1. In the `daml` directory, create `UserAdmin.daml` to allow user admin to administer users. 

There will be 
- `owners` template 
- `issuers` template, content creators

We should create templates that correspond to the different types of user that are undersigned by the userAdmin
In this case, this entitlement is granted by the `userAdmin` to allow issures to mint NFTs

```haskell
-- in UserAdmin.daml file
module UserAdmin where
import Token

template Issuer
  with
    userAdmin: Party
    issuer: Party
  where
    signatory
      userAdmin
    -- Current setup, once the controller exercises this choice
    -- This contract will be archived, hence only allowing issuance once
    -- Not expected behavior
    -- add 'nonconsuming' to the front of MinToken
    controller issuer can
      -- Token type will be unrecognized, need to add
      -- import Token at top
      -- Current setup, once the controller exercises this choice
      -- contract will be archived, hence only allowing issuance once
      -- add 'nonconsuming' to the front of 'MintToken'
      -- Prevent the contract from getting archived after MinToken is exercised
      MintToken: ContractId Token
        with
          description: Text
          initialPrice: Decimal 
          currency: Text
          -- give issuers the power to choose royalty rate
          -- royaltyRate: Decimal (uncomment for later)
        do
          create Token 
            with 
              lastPrice = initialPrice
              owner = issuer
              ..
    
    controller userAdmin can
      -- return nothing
      RevokeIssuer: ()
        do
          -- return nothing
          return ()

```
We'll add one more template. We want a random new issuer to be able to ask for a token. ASk for an issuer contract

```haskell
template IssuerRequest
  with
    userAdmin: Party
    issuer: Party
    reason: Text --why they should be allowed on the network 
  where
    signatory
      issuer
    controller userAdmin can 
      GrantIssuerRights: ContractId Issuer
        do
          create Issuer
            with
              ..
      RejectIssuerRequest: ()
        do 
          return ()
```


# Owner template
what this template does is it allows able to accept ownership of a token on behalf of themselves as wellas the userAdmin. 

What we are doing by creating an owner template and an `issuer` template is the ability for issuers and content craetors to be asked to be allowed on the network

once they are granted by the userAdmin, the issuer or owner status, 
owners can accept token offers
content creators can mint

```haskell
-- in UserAdmin.daml

template Owner
  with
    userAdmin: Party
    owner: Party
  where
    signatory 
      userAdmin

    controller owner can
      nonconsuming AcceptTokenAsNewOwner: ContractId Token
        with 
          -- We are passing in a contract type into this choice
          offerId: ContractId TokenOffer
        do
          -- within this context, this choice is being exercised with 
          -- the signature of the userAdmin and the owner
          exercise offerId AcceptToken

      -- instead of taking in the contract Id, we are taking in the below fields
      -- 
      nonconsuming AcceptTokenByKey: ContractId Token
        with
          issuer: Party
          currentOwner: Party
          description: Text
        do
          -- we specify the contract, the key, and the choice, 'AcceptToken'
          -- this allows an owner to accept a token based on the key, rather than the offerId
          -- COntract Ids will change, but the keys are more persistant
          exerciseByKey @TokenOffer (issuer, currentOwner, description) AcceptToken

    
    -- We want the useradmin to be able to revoke the owner rights
    controller userAdmin can 
      RevokeOwnerRights: ()
        do
          return ()
        
```
# Owner Request Template
```haskell
-- in UserAdmin.daml
template OwnerRequest
  with
    userAdmin: Party
    owner: Party
    reason: Text -- why should I be allowed on the network
  where
    signatory
      owner
    
    controller userAdmin can
      GrantOwnerRights: ContractId Owner
        do
          create Owner with
            ..
      
      RejectOwnerRequest: ()
        do
          return ()
```
# Testing your templates with Scripts

1. Go to your daml nft directory
2. `cd nft`
3. `daml studio` this will open up daml
---
1. Go to `main.daml`, click the small pop up "Script results"

We will use `main.daml` where we will put all our tests into
`main.daml` is what we want to execute. 

1. Delete the `asset` template

## Creating a test scenario
1. We want to create the parties that will interact with the templates. eg: 
```
Max <- allocatePartyWithHint "Max" (PartyIdHint "Max")
```
We are going to have them act. We'll keep bob and alice, 
and We also need to add a userAdmin party. Add the script below
```
userAdmin <- allocatePartyWithHint "UserAdmin" (PartyIdHint "UserAdmin")
```
As the issuer we want to create a token. 

1. add `import Token` to the top of `Main.daml`
2. Add `return()` at the end of the script

Below is an erronous script if you want to see for yourself. 
If we just use the script below, 

Below is an erronous script that creates a `Token` contract without the `userAdmin` authority. We can see that `alice` alone is submitting the `createCmd` (create Command). 

The error will say it is missing authorization from userAdmin

```haskell
  submit alice do 
    createCmd Token
      with
        issuer = alice
        owner = bob
        userAdmin = userAdmin
        lastPrice = 100.0
        currency = "USD"
        description = "cat NFT"
```
If you pasted the above into your IDE, delete it. 

What we need to do is create an `Issuer` template on behalf of alice. Because at the moment alice is not onboarded as an issuer yet. Paste the below into the scripts. 

Here, the user admin creates the `Issuer` contract on behalf of alice. 
```
  -- here we have a handle on the issuer contract
  aliceIssuer <- submit userAdmin do 
    createCmd Issuer
      with
        userAdmin = userAdmin
        issuer = alice
```
Now that the `aliceIssuer` contract is created, add the following script. 

## Test steps
Step 2, alice exercises `MintToken`
remember to add 'nonconsuming' to the front of  'MinToken' choice in
UserAdmin.daml, in the Issuer Template 
```haskell
   originalToken <- submit alice do
     exerciseCmd aliceIssuer MintToken
       with
         description = "cat pic 1"
         initialPrice = 100.00
         currency = "HKD"
```

Step 3, Bob requests to become an owner
```haskell
  bobRequest <- submit bob do
    createCmd OwnerRequest
      with
        userAdmin = userAdmin
        owner = bob
        reason = "I've got connections"
```
Step 4, UserAdmin grants owner rights

```haskell
  bobOwner <- submit userAdmin do 
    exerciseCmd bobRequest GrantOwnerRights
```
Step 5, alice offers token to bob with new price
```haskell
  bobOffer <- submit alice do
    exerciseCmd originalToken Offer
      with  
        newOwner = bob
        price = 200.00
```
Step 6, bob accepts offer as new owner
```haskell
  submit bob do
    exerciseCmd bobOwner AcceptTokenAsNewOwner
      with
        offerId = bobOffer
```
So this tests out the transfer of ownership of the NFTs. 

# End of Part 1 

### Explore creating frontend using React & Typescript
- Your first feature [link](https://docs.daml.com/getting-started/first-feature.html)
- Getting Started [link](https://docs.daml.com/getting-started/index.html)


# Part 2 - Payment template [ TODO ]
This will keep track of the payment. Who needs to pay who and if the payments were made. 

In the `daml` directory, create a new file, `Payment.daml`
