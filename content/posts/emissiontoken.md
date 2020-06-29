---
title: "Using the blockchain to decrease carbon emissions"
date: 2020-06-29T12:07:28+01:00
draft: false
authors:
    - Manuel Sinn
tags:
    - blockchain
    - sustainability
    - solidity
    - software-development
    - smart-contracts
---

### What is this?
An ERC20 token standard compliant "Emission Token" that uses blockchain technology to enable a  decentralized emissions trading system, coded as a smart contract in Solidity and deployed on the Ethereum test net.


## The idea
Emissions trading (or cap and trade) uses the power of the market to distribute emissions in the most efficient way, in order to decrease overall pollution.

First, the total amount of emissions is capped to a certain value. This mass of emissions is then divided into many smaller permits for a certain amount of pollution. These are traded freely on the market, so that everybody is incentivized to keep pollution low and to make production as pollution-efficient as possible. In very basic terms: the more you emit, the more you pay - and the less you emit, the more money you can make. In theory, this is a great idea to lower greenhouse gas emissions. After all, 
the current planetary health illustrates that the value of sustainability in itself is not a big enough motivator to protect the environment - perhaps unlike money. 



## The challenge
For emission trading schemes to work, all parties need to comply to the rules. This means that some form of regulation needs to be in place. Traditionally, this means that a central (governmental) authority enforces those rules. In order to do so, this authority needs a lot of information, gathered through measuring, reporting and verification (MRV). Every entity that violates the rules can then be fined and sanctioned by the central authority.

Obviously, the entire process of measuring, reporting, verification and enforcement requires a big overhead of bureaucracy and is prone to error and cheating (for example by lying about one's own emissions). 



## Decentralized improvement 
One opportunity to streamline this process could lie within the realm of distributed ledger technologies. By using the blockchain and smart contracts, a lot of all this could potentially be... 

- **automated**: I'm thinking IoT enabled emission meters talking to smart contracts to automatically update  the emission balance of their holders
- made more **secure, transparent and trustworthy**: using the blockchain as an immutable data structure, all interactions could be logged on there, open for all to see and be held accountable indefinitely
- made more **efficient**: without the need for a central authority and connected intermediaries to gather information and enforce the rules, sanctions and payouts can both be dealt with a lot faster

An option to bring blockchain into the game is to implement a tokenized version of an emissions trading scheme, so that emissions can interact with smart contracts in unique ways and be traded similar to the way crypto currency is. In a certain way, emissions are already tokenized to some extent: One emissions permit (also called carbon credit or Kyoto unit) is equivalent to one metric ton of CO2 emissions.  
To illustrate and further this idea I created an ERC20 token standard compliant "EmissionToken" which could be used to work towards these promising improvements.



## Code
The Solidity code can be found on [GitHub](https://github.com/manuelsinn/smart-contracts/blob/master/emissionToken.sol) or viewed right on this page:   


<details>
  <summary>Click to see the code!</summary>

{{< highlight solidity "linenos=false >}}

/*
EmissionToken - PoC implementation
This smart contract is a way to decentralize the current emission trade
and move it to an immutable datastructure, possibly enhancing efficiency,
transparency and trustworthiness.
   
Basic token infrastructure coded along gilad haimovs great tutorial thttps://www.toptal.com/ethereum/create-erc20-token-tutorial
*/

pragma solidity >=0.4.22 <0.7.0;

contract EmissionToken {

    // typical fields to describe the token
    string public constant name = "EmissionToken";
    string public constant symbol = "EMS";
    uint8 public constant decimals = 18;  

    // these events can easily be picked up from the outside, e.g. to react to them on a web app using js
    event Approval(address indexed emissionOwner, address indexed spender, uint amount);
    event Transfer(address indexed from, address indexed to, uint amount);
    event EmissionSpent(address holder, address associate, uint amount);

    mapping(address => uint256) emissionBalances; // stores the current emission balance for every account
    mapping(address => mapping (address => uint256)) allowed; // stores the allowance a delegate has to withdraw emissions from an owner
    mapping(address => address) holderOf; // stores the registered holder to an associate (e.g. the firm to an automatic emission meter)
   
    uint256 totalEmissionsCap; // amount of emissions approved by regulations or similar, set initially before trading
    uint256 deadline; // end of the time frame for the capped amount of emissions

    using SafeMath for uint256; // to counter integer overflow attacks


    constructor(uint256 _totalEmissionsCap, uint256 timeFrame) public {  
        totalEmissionsCap = _totalEmissionsCap;
        emissionBalances[msg.sender] = totalEmissionsCap;
        deadline = now + timeFrame;
    }  
   
   
    //------------------------ ERC20 token standard functions ------------------------

    // returns the total emission tokens in circulation
    function totalSupply() public view returns (uint256) {
        return totalEmissionsCap;
    }
   
    // returns the emission balance of a specific address
    function balanceOf(address emissionOwner) public view returns (uint) {
        return emissionBalances[emissionOwner];
    }

    // used to tranfer amount emissions from the message sender to a specified address
    function transfer(address receiver, uint amount) public beforeDeadline returns (bool) {
        require(amount <= emissionBalances[msg.sender], "You cannot transfer more emissions than you have yourself.");
        require(now < deadline, "You have surpassed the trading time frame for the currently capped amount of emissions.");
        emissionBalances[msg.sender] = emissionBalances[msg.sender].sub(amount);
        emissionBalances[receiver] = emissionBalances[receiver].add(amount);
        emit Transfer(msg.sender, receiver, amount);
        return true;
    }

    // used to approve a delegate to withdraw amount emissions from the message sender's account (e.g. in a marketplace scenario)
    function approve(address delegate, uint amount) public beforeDeadline returns (bool) {
        allowed[msg.sender][delegate] = amount;
        emit Approval(msg.sender, delegate, amount);
        return true;
    }

    // returns the amount of emissions a delegate is approved to withdraw from an owner (set in approve() )
    function allowance(address owner, address delegate) public view returns (uint) {
        return allowed[owner][delegate];
    }

    // used by a delegate (e.g. the marketplace) to shift amount emissions from an owner to a buyer
    function transferFrom(address owner, address buyer, uint amount) public beforeDeadline returns (bool) {
        require(amount <= emissionBalances[owner], "The owner does not have enough emissions to transfer that amount.");    
        require(amount <= allowed[owner][msg.sender], "The owner has not authorized you for that amount.");
   
        emissionBalances[owner] = emissionBalances[owner].sub(amount);
        allowed[owner][msg.sender] = allowed[owner][msg.sender].sub(amount);
        emissionBalances[buyer] = emissionBalances[buyer].add(amount);
        emit Transfer(owner, buyer, amount);
        return true;
    }
   
   
   
    //------------------------ Emissions Trade related functions ------------------------
   
    // ensure that everythings stops after the deadline
    modifier beforeDeadline(){
        require(now < deadline, "You have surpassed the trading time frame for the currently capped amount of emissions.");
        _;
    }
   
    // subtract spent emissions from the tokenized representation
    // will be called from other smart contracts, e.g. IoT enabled emission meters, registered to the holder
    function updateEmissionBalance() public payable beforeDeadline returns (bool) {
        require(isRegistered(msg.sender), "You are not registered to a holder. Please get registered by your holder first.");
        address holder = holderOf[msg.sender];
        emissionBalances[holder] = emissionBalances[holder].sub(msg.value);
        emit EmissionSpent(holder, msg.sender, msg.value);
        return true;
    }
   
    // To prevent misuse (subtracting emissions by unauthorized devices),
    // the associate must first be registered to its holder
    function isRegistered(address associate) private view returns (bool) {
        return holderOf[associate] != address(0);
    }
   
    // register a device or person to authorize them to subtract from your emmissions
    function registerAssociate(address associate) public beforeDeadline returns (bool) {
        // to prevent misuse, only the holder can register an associate
        // - otherwise other parties could register associates which
        // would subtract from your emissions, despite not having spent any
        holderOf[associate] = msg.sender;
        return true;
    }
}


library SafeMath {
    function sub(uint256 a, uint256 b) internal pure returns (uint256) {
      assert(b <= a);
      return a - b;
    }
   
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
      uint256 c = a + b;
      assert(c >= a);
      return c;
    }
}


{{< / highlight >}}

</details>



## Tech Stack & Process
After completing the 9+ hours learning path about Blockchain and related topics on LinkedIn Learning, I first coded a basic smart contract in Solidity and deployed it on the Ethereum (test) network using truffle. To be honest, this process was pretty lengthy and I wanted to focus on the concept itself and ease of use. Although truffle and the MetaMask plugin serve as excellent ways to deploy smart contracts and see them in action, e.g. on [Etherscan](https://etherscan.io/), some issues (like having to download most of the blockchain history) led me to a much simpler deployment approach, namely switching to the online IDE [Remix](https://remix.ethereum.org/).
To dive deeper into Solidity and get my hands dirty, the [Solidity docs](https://solidity.readthedocs.io/en/v0.6.10/solidity-by-example.html) were a great place to start.  




## Limitations to the implementation
The code is mostly to be understood as proof of concept. Some challenges that remain include:

- The **initial distribution of emissions**. In a real world scenario, a government would deal with the distribution according to different criteria such as historical production and others. So far, the contract creator initially gets all the emission tokens, and its up to him to transfer them. A way to deal with this could be some form of initial distribution similar to ICOs (initial coin offerings) or STO (security token offerings).  

- The **registration of holders** (e.g. institutions or firms which have a total emission balance) and **associates** (their subordinated parts, like a firm's factory, which decrease that balance by emitting green house gases). In this implementation, the smart contract only allows holders to register their associates. This makes sense for example because it prevents competing businesses from registering smart meters to their rivals in order to decrease their emission balance. However, it does not solve the problem of an entity just deciding to cheat by not registering an associate to save emissions.   

- **Automated punishments and regulations** are not included yet. A spontaneous idea that just sprung to my mind is a decentralized way to let all participants regulate each other, maybe by appointing a suspected cheater and suspending him from the exchange for a set period of time when enough participants vote to do so. This would incentivize clean playing, but at the same time introduce mistrust between parties into the game.



## What I learned & some thoughts
#### Creating the smart contract
When I first started with Solidity, I noticed that this stuff was harder than I had expected, and that it was much more of a concept than a syntax issue. Solidity is very similar to other languages and very easy to pick up, save for a few new concepts like modifiers, require(), gas, included transaction capabilites, etc. A few great examples that showcase the power of the decentralized smart contract are shown in the Solidity docs, like the voting contract, the auction contract or the way a (mostly) trustless online transaction is handled. 

Initially, I planned to implement the whole transaction process, including tokens and compensation in form of cryptocurrency (Ether in this case, as the smart contract runs on the ethereum network). Instead of transfer functions I had functions to sell and buy emissions, and a global ''' conversionRate ''' variable to calculate the price for each transaction. I quickly ran against a wall here though, because in order to do this, I somehow needed to match seller and buyer, basically having to implement a whole marketplace or exchange of sorts. This made me stumble upon the ERC20 token standard, which delegates this to a third party (like a marketplace) through the approval mechanism.


#### Future
In general, I am confident that the blockchain and related technologies offer enormous potential for innovation far beyond the reaches of just financial applications. Although Ethereum network enabled DApps are mostly still in their infancy, we can already see some of the potential that is created by combining the unique qualities blockchain brings to the table and the innovative ideas from developers and business leaders alike. As with most disruptive technologies, a lot of challenges and risks have to be faced and overcome. 

Besides knowledge and acceptance from the general public, I think the consensus mechanism for new blocks to be mined (or forged) is a key factor determining the technology's success. At the time of writing this, Ethereum (the second biggest crypto currency by market capitalization) is at the brink of ditching the power hungry proof of work mechanism and about to go with proof of stake.

But right now perhaps the biggest risk of all - is to miss out, by not acting at all.