# POST-MORTEM STAGENET

2/3 votes for finality not been reached. The cause for this was that as part of the validator set, there were some authorities with the wrong keys (old-keys). Because these lack of finalization dragged on for many blocks (more than  50), nodes rejected to claim new slots to produce new blocks (this is a substrate measure), ths reducing the block production. However if more than 100 non-claimed slots elapse, nodes try to author a block.

Solution:

- Remove validators with wrong keys from the authority set. To achieve this:
    - remove them from staking
    - wait until new session
    - we already saw the active validator set was holding only legit authorities
    - however, grandpa did not apply this authority set change. We know this from the rpc call **grandpa->roundState**. This tells which
      validators are not voting properly, and the overall outcome. You need to **verify this against the grandpa key**, which you can find in **session->keyOwner** state.
    - After we saw there was an authority set missmatch, to hint the network about the new authority set, we needed to call **grandpa->nodeStalled** extrinsic, which forces the finality consensus to change the grandpa authority set and also hints the height after which we want this authority to finalize blocks.