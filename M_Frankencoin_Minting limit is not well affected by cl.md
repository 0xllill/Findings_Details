Minting limit is not well affected by cloning

## Lines of code
Position.sol#reduceLimitForClone : https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L97-L101

##  Impact
Line 89 and line 90 of Position.sol it is written as a comment :

“Adjust this position's limit to give away half of the remaining limit to the clone.
Invariant: global limit stays the same. “

But it’s false, I will explain why.

When cloning a position (which seems to be the easiest way to acquire a position), the limit of zchf that can be minted by the primary position is affected, here is the process :

Cloner call clonePosition() from MintingHub.sol
mintingHub calcul the limit for cloning position by calling primary position function reduceLimitForClone : uint256 limit = existing.reduceLimitForClone(_initialMint);
This function will affect the limit of the current position regarding how much minter has already minted in an incoherent way
Example :
a. userA creates a positionA with a limit of 20000, he currently minted only 1000 zchf
b. userB wants to clone a position to mint 2000 zchf , he creates it by cloning positionA, the calcul is like this in position.sol#reduceLimitForClone(uint256 _minimum) :
        uint256 reduction = (limit - minted - _minimum)/2;
        limit -= reduction + _minimum; // new limit for positionA = 9500
        return reduction + _minimum; // limit for clone = 18 000

    | limitA       | 20000 |
    | ---          | ---   |
    | currentMintA | 1000  |
    | minimumB     | 2000  |
    | reduction    | 8500  |
    | newlimitA    | 9500  |
    | limitCloneB  | 18000 |
⇒ UserA who deposited a collateral according to its mintingMaximum capacity to not be challenged even if he/she reach the limit will have it’s new limit hugely reduced (by more than average in this example)

⇒ UserB who will deposit according to it’s _initialMint amount will have a limit hugely higher than wanted ( more than 8x in example) and is at risk to be challenged because its collaretal is not in accordance with it’s cloning position limit

To conclude it can be dangerous if userB mints more than he wanted at the beginning, its position is valid as he clones a valid one but he can mint a lot more than intended at the beginning, userA will have to creates another positon and repay fees to mint the amount he wanted at the beginning

## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.

## Tools Used
VS Code

## Recommended Mitigation Steps
Add a parameter in the MintingHub.sol#clonePosition that ask cloner which limit he wants to reach, do your calculs and if limitwanted < existing.reduceLimitForClone(_initialMint); , clone the position with limitWanted as a limit and only reduce the limit of limitPositionA - limitwanted
