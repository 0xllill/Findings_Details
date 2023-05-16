Users can update and delete others queues

## Summary
This ownerToRollOverQueueIndex[_receiver]  is updated each time the enlistInRollover() function is called. If user calls the function multiple times, it can be updated to someone else's index, so it can update and delete someone else's queue.

## Vulnerability Detail
The enlistInRollover() is used to enlists in rollover queue.If user queue up a rollover for the first time, the protocol will add to queue. If user has already queued up a rollover ,then update the queue and ownerToRollOverQueueIndex[_receiver]. Here is the problem, let me show how user update other queues.

Assume the size or rolloverQueue is 5.Alice queue up a rollover for the first time, the protocol will add to queue and update 
ownerToRollOverQueueIndex[Alice]  = 5
```solidity
         rolloverQueue.push(
                QueueItem({
                    assets: _assets,
                    receiver: _receiver,
                    epochId: _epochId
                })
            );

        ownerToRollOverQueueIndex[_receiver] = rolloverQueue.length;

```

2.After some time, others queued up and the size or rolloverQueue is 15. At this moment, Alice wants to update her queue. So the protocol will update the queue information and ownerToRollOverQueueIndex[_receiver], hence the ownerToRollOverQueueIndex[Alice ] = 15.
```solidity
            uint256 index = getRolloverIndex(_receiver);
            rolloverQueue[index].assets = _assets;
            rolloverQueue[index].epochId = _epochId;
            ownerToRollOverQueueIndex[_receiver] = rolloverQueue.length;
```

3. Alice updates the queue again.As for the step 2 ,the index of the rolloverQueue for Alice becomes 15. The protocol will update information with array (15-1). Actually，the index of rolloverQueue for Alice is (5-1). Hence, Alice successfully updated the queue information of others.
It is also possible to call the delistInRollover function to delete someone else's queue in this way.


## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L238-L271

## Impact
Users can update and delete others' queues at will



## Tool used
Manual Review
VS Code

## Recommendation
Put the code ownerToRollOverQueueIndex[_receiver] = rolloverQueue.length in the else branch

```solidity
      if (ownerToRollOverQueueIndex[_receiver] != 0) {
            // if so, update the queue
            uint256 index = getRolloverIndex(_receiver);
            rolloverQueue[index].assets = _assets;
            rolloverQueue[index].epochId = _epochId;
            return
        } else {
            // if not, add to queue
            rolloverQueue.push(
                QueueItem({
                    assets: _assets,
                    receiver: _receiver,
                    epochId: _epochId
                })
            );
            ownerToRollOverQueueIndex[_receiver] = rolloverQueue.length;

        }
```