
@TODO merkle proofs, unidirectional payment channels

@TODO channel spec: the stateRoot contains lastEventHash; events should always be linked by containing a hash to the previous event, to ensure an immutable data structure; however, we should decide merkelize all events, in order to allow people ot check if events are genuine

@TODO impl adex-smart-platform: rust? simple API, on top of SQL (sqlite/postgres both supported) and tokio for networking (zap?); WASM state transition function
@TODO can adex-smart-platform be designed in a blockchain-agnostic way; the blockchain interface should allow to enumerate channels, sign a channel state update
@TODO describe the `adex-smart-platform` node implementation: performance is critical;  needs to easily scale horizontally and sharding needs to be thought of; needs merkle trees too, but the channel would merely be open with (deposit, timeout) and advanced with (stateRoot, signatures)

@TODO header bidding spec; On each open of a publisher website, it would pull all bids from the operator and select a bid (campaign), and send events

@TODO channel spec: describe timeouts, and how they're really last resort; if you expect the channel to be exhausted in 1m, the timeout should be 3x that (3m)

@TODO adex-smart-platform: one more interesting spec thing to think about: pre-paying for impressions: the leader would sign a state update that PAYS first, and the impression will be shown after

@TODO describe at what point (how many unreported events) the platform (publisher/supply) would decide to untrust the channel; or at which point an individual publisher might stop trusting it (e.g. invalid balances tree); perhaps a % of the min impressions for a campaign
@TODO a nice privacy preserving property would be that the platform wouldn't reveal which wallet (in terms of revenue in the balances part of the state tree) belongs to which publisher; that way you can't see where the money from an advertising campaign is flowing, even if everyone withdrawls

@TODO: consider libp2p for communicating between payment channel participants
@TODO There should be a special msg in the payment channel that should be send to the consensus leader: `need_missed_events` or something like that - where the follower(s) say that they won’t continue signing unless those events are included or at least some part of them; that would be determined by `missed_event_threshold`

@TODO blog post about benefits, use cases of the unidirectional payment channel model with multiple validators; perhaps it can even be used in a DEX if we can atomically interleave a value transfer between two unidirectional payment channels; can be done with something similar to HTLC
@TODO questions that arise
"But what if someone goes offline?"
"but what if someone uses older state?"
"what if the parties mutually agree to close the channel early?" - explain how, on mutual agreement, the paying party can withdraw their funds out; the timeout only exists for extreme byzantine cases
"are 2 validators enough" - if the validators have opposing interests, yes; in this case, the model is exactly as in any payment channel - party A will send micropayments to party B, and if party A stops paying, party B can stop delivering their service and withdraw theire earnings without much loss (Use (1))
"what does leading validator imply?" - they only propose new states, but can't authorize spending without a total supermajority
"are there other usecases besides AdEx" - "a dex", pun intended, lol
"is this for ethereum?" technically it can be done on any programmable blockchain platform; it can even be done on BTC using the UTXO model, similarly to how lightning works; unfortunately we need RSMC

@TODO BTC version; this will be pretty easy to do on top of UTXO's and scripts; when opening a channel, two tx-es are created with the same inputs (advertiser funds), one being a spendable by multisig of validators, the other being a timelocked tx spendable by the advertiser (returns funds to advertiser); to advance the channel, the validators sign new TX-es which contain the msig TX output as an input, and many outputs (the balances tree); to invalidate old tx, we can use a similar scheme as the LN (RSMC); since this is so similar to the LN, can it be built on top, and can it be compatible?
@TODO BTC: actually, we can do a slightly less trustless model which does not require RSMC; the advertiser signs the tx and gives it to the publisher; the publisher validator does not sign it (or just doesn't reveal); once the channel is exhausted, then they sign it and it can be broadcast; this is suboptimal since publishers don't have a constant revenue guarantee that they can verify; alternatively, the publisher validator could sign and reveal the sig to the publishers, but if one of them leaks it, that will allow the advertiser to broadcast it early
@TODO can the LN play in here? 

@TODO describe the possibility to reward users with tokens (via the balances tree); However the economic incentives work against us as they incentivize users to forge; although if the users masquerade as publishers, it should be the same thing; Anyway, memory-bound PoW and ip limits should be considered; and/or using the ip in the sig


@TODO when describing OUTPACE, lean on ameen's "a way for two or more entities to privately make updates to some state that only they control"; OCEAN spec should include state transition fn as well; OUTPACE extension should include the balance tree and restrictions; also, when describing OCEAN/OUTPACE emphasize privacy

@TODO OUTPACE/OCEAN usecases: can they be used for interoperability? like LN

@TODO describe the bidding system: between the platform and the publisher/user; maybe send bid{matchToCpmRatio, minCpm}; then we calculate match rating (floating point, 0 to 1, depending on targeting) and the bid price is `max(matchToCpmRatio*match, minCpm)`; as for the match ratio, that can actually be defined as; every ad gets a match rating `sum(targetingTags.filter(tag in userTags).map(x => x.weight))`, and then all match ratings will be scaled between 0 and 1, where 1 represents the highest match rating;   ALTHOUGH this model is not nice for privacy - you can probe if a user has a certain tag at a cost of outbidding everyone else

@TODO bidding system: should we use a second-price auction? also, there should be a minimum threshold of difference in order for a bid to be considered higher (e.g. at least 0.5% higher)

@TODO describe `adex-smart-platform` events mempool: a sorted set, where `insert` and `find` work via a binary search, we pop items from the beginning (oldest first) to clean it up; describe messages between validators too: ProposeNewState, SignNewState, RequestEventsBeIncluded; consider a Heartbeat message; also, each node should keep an internal ledger of who else from the validator set is online - if 1/3 or more is offline, stop showing the ad (stop participating in bidding);  also we should keep from who we observed which event, so that we can see if the events we didn't see were observed by the supermajority; also think of IP guarantees here, since it's the only thing preventing events from being just re-broadcasted; ANOTHEr security measure is have the user sign the event for every validator separately

@TODO adex-smart-platform DB structure, including a table `channels_onchain` which is populated by the blockchain-specific adapter (which consists of a continuous process that populates the table, AND an interface to sign and provide merkle proofs); this is important for having an agnostic system

@TODO describe internal ledgers in adex-smart-platform: there's one on which events were provably observed by other users; and one for how many fees are claimed (ClaimValidationFee, can be created by a validator to make them claim a fee)

@TODO describe the bidding model; bidding is currenly not trustless, so describe the implications of this; we can make a mini state channel like thing where each next bid links a previous one by hash, but that's not censorship resistant; it's also very similar to a blind auction

@TODO adex-smart-platform: might need a restriction on the max publishers, or on min spend per publisher; since otherwise it might not be worth it for a publisher to withdraw

@TODO adex-smart-platform parameters: e.g. `{ batching: { maxEvents, maxTime } }`

@TODO you can do an affiliate network, by rewarding the publisher if someone bought something on the site of the advertiser; in other words, adex can be used as an affiliates platform

@TODO encrypt user's data in the SDK? with a key from the nodes?

@TODO homomorphic encryption or some kind of obfuscation of the data in the SDK? how can this be done? maybe nucypher?

@TODO validator fees can be paid via the OUTPACE channels themselves; the fees can even by dynamic/ongoing

@TODO describe the idea of users implicitly outbidding advertisers, if they want to pay for content rather than see ads; the flow of money is `{user OR advertiser} -> publisher`
