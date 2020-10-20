<div class="cwikmeta">  
{  
"title": "Voting's Impossible Triangle"  
} </div>

# Voting's Impossible Triangle
*The ideal attributes of democratic voting procedures are in conflict, necessitating tradeoffs*

<img align="right" src="/election_impossible_triangle.svg" width=400></img>

Any good voting process<sup>1</sup> has desirable attributes which can be summarized by three adjectives:

**Integrity** refers to whether the vote correctly reflects the true choice of every participant, without intimidation or incentivization.  It also encompasses techniques that prove or disprove integrity.  In particular, the voting process itself can be considered as having a higher integrity than another process if it allows such techniques.

**Anonymity** refers to the ability of participants to cast votes anonymously.

**Remotability** is a new word, referring to whether participants can, and the processes required to, participate without appearing at designated physical locations.   "Remotability" has become very important during the Covid19 pandemic across many systems and processes.  However, it is often seen as desirable to increase democratic engagement, and many jurisdictions have had some remote capability for years, commonly referred to as an "absentee" ballot.

Recent attention to electronic and blockchain-based voting processes allow researchers to analyze practical systems using clean, theoretical models.  This analysis exposes attacks to traditional physical paper ballot systems that perhaps are overlooked because paper ballots have a history of mostly successful application.

Analysis has made it clear that these attributes are fundamentally opposed in many respects, in ways that perhaps the messy possibilities of physical presence and paper ballot election processes have previously obscured.


## Interactions 

In this description "Mallory" is used as the name of the candidate or organization that garnering votes through some form of cheating.

### Remotability verses Integrity

If a voter is not present, they can sell or be intimidated into making a selection since the agent can be present with the voter and verify their selection.  This is one purpose of physical voting booth isolation.  The achievable volume is limited per agent.

Ironically, if Mallory has already co-opted the voting infrastructure or enforcement presence (such as the police or military), in-presence voting can undermine the democratic process since voters are forced to put themselves under the eye of the Mallory's agents.

Blockchain solutions that allow electronic remote voting can be undermined by software exploits that mis-vote.  A practical deterrent to this would be to require validation on independent devices.  For example, a vote might be created by the participant's phone and submitted and verified on a separate e-voting machine.  But this requires presence.

### Anonymity verses Integrity

Anonymous voting promotes integrity by preventing post-hoc sales and intimidation.  However, in paper ballot systems the loss of information implied by anonymity prevents voters from verifying that their vote was counted and that extra votes were not added.

Blockchain voting systems can solve some of these problems, via Token Shuffling (that preserve quantity while creating anonymity) and Merkle Tally Trees<sup>2</sup>.

### Anonymity verses Remotability

In a Blockchain context, public/private keys allows voters to sell their key to another entity before the election.  This entity could then sell blocks of votes, and it is likely that Mallory could verify the size of the block, since voting authority would likely respond to the question "am I registered to vote?".
  
However, since the voter retains the key, they could "spoil" the transfer by also using it to vote (depending on the exact protocol followed if conflicting votes are discovered).  But this could be dangerous since Mallory would know the voter "spoiled" their vote.

Even worse, Token Shuffling promotes anonymity but would allow a secure, unspoilable, validatable, transfer of a vote token to another entity before the election.

The attacks described above assume non-presence; if a present voter's device must interact with a voting device, that allows only a single vote per "use", it would be hard to vote in blocks, and not scalable, as Mallory's agent would need to physically move to different polling stations to execute multiple votes even if he acquired multiple private keys or tokens.

Other options would entail limiting the token shuffle time and location window (undermining anonymity to some degree), or making it impossible to determine a-priori whether a token is real or fake (undermining registration).

### Post-hoc Validation: Integrity verses Integrity

Blockchain votes can be succinctly proven to have been included in a tally, unlike paper ballot votes, by using a Merkle Tally Tree<sup>2</sup>.  However, post-hoc proofs allow post-hoc sales and intimidation.

Can fake or other votes be provided to voters so that they can present these to Mallory?

### Public electronic records: Integrity verses Integrity

Paper ballot "independent" recounts require a certain level of trust as the ballots are transferred between counting organizations.  Anonymous organizations cannot permissionlessly gain access to the paper ballots to execute their own recount.

However, any "independent" organization with that level of interaction with the voting authority is itself clearly a target for compromise.

Blockchain based systems solve this problem, with their public and permissionless record of voting.


## Footnotes
**[1]** A voting process covers the specific system and activities required to identify participants and formally solicit their (possibly weighted) opinion.  Terms like elections, referendums, and the passage of bills are both too specific and cover additional activities.

**[2]** [Stone, Andrew. Merkle Tally Tree](/blockchain_voting#merkle-tally-tree)