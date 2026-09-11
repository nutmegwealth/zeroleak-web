# ZeroLeak

A marketplace where a security researcher can sell a smart-contract bug to the protocol that's actually vulnerable to it, without either side having to trust the other.

Live on Base Sepolia: https://zeroleak-web.vercel.app/

It actually works — you can connect a wallet, list a bug, buy one, and watch the money move. Not a mockup.

## How it works

The seller proves they've got a real exploit (using a zero-knowledge proof, so they don't have to show it) and locks the details in an encrypted file. The buyer pays, but the money sits in escrow, not the seller's pocket. Then the seller hands over the key so the buyer can read it. Before anyone gets paid, a bot re-runs the exploit against the target to check it actually works. If it does, the seller gets the money. If it's fake, the buyer gets refunded and the seller loses their deposit. Nobody has to referee any of this.

## The vertical

I've messed around with bug bounties before, and the annoying part was always the handoff. If you show the company the bug first, they can just go "cool thanks" and not pay you. If they pay first, they're scared you'll send them nothing. Bounty platforms kind of duct-tape over this by making everyone trust the platform, and honestly the payouts are usually way lower than the bug is worth.

So the idea here is to make it an actual market instead of a trust-me situation. The company that's vulnerable is the buyer, the researcher is the seller, and the thing being sold is something you can't really show or check ahead of time. That "pay for something you can't see from someone you don't know" problem is basically the perfect use case for putting escrow on a blockchain.

## Trust assumptions

Stuff you DON'T have to trust:

- Me holding the money. There's no me. It's a contract anyone can read, and the money moves based on the rules, not because someone decided to release it.
- The seller saying they have a real bug. They have to prove it on-chain before they can even list, and the proof is locked to their wallet so nobody can steal it and repost it.
- The buyer being honest about whether the bug worked. They don't get to decide that — the bug gets re-run and the result goes on the chain.

Stuff you DO still have to trust (not gonna pretend otherwise):

- The bot that re-runs the exploit. Right now it's just one operator (me) running it. The contract handles everything after the verdict, but you're trusting that the bot is honest and actually online. That's the weak point, more on that below.
- That what you get is what you paid for. The contract checks the revealed bug against a fingerprint the proof was tied to, so if the seller sends junk instead, it doesn't match and the check fails.

## Biggest design decision

Making "did the exploit actually work?" a fact the blockchain records, instead of an argument someone has to settle.

The lazy way to build this is with a judge: buyer says it's junk, seller says it's fine, some human picks who's right. I didn't want that, because now you're back to trusting a middleman, which is the whole thing I was trying to get rid of. Every close call becomes someone's opinion.

So instead there's a bot that takes the exploit, runs it against the target, and just records what happened — did it break the thing or not. Worked, seller gets paid. Didn't, buyer gets their money back and the seller loses their deposit, automatically. It stops being a debate and becomes something the chain already knows. Pretty much everything else in the project is built around that one idea.

## One big limitation

The bot is centralized and kind of dumb, and I'm not going to act like it isn't.

Centralized: it's one bot, one key. If it's offline, new listings just sit there waiting. If someone stole the key they could post fake results. The real fix is having a bunch of independent bots, or letting anyone challenge a result by putting money on the line — haven't built that yet.

Dumb: the bot only knows how to re-run the specific kind of exploit I taught it (reentrancy, against the vaults it knows). It can't take any random exploit against any random contract and figure out if it works. So the "it checks the bug for real" part is legit, but it's not general yet. Making it handle more kinds of bugs is honestly the actual hard problem hiding in this whole thing, and I'd rather just say that than hide it.

## What's deployed (Base Sepolia, chain ID 84532)

| | Address |
|---|---|
| Marketplace | `0xA67822e77acC2bC8Ca58A58aaB372398e62D8D0F` |
| ZK verifier | `0xe80BEda38b291AEa50D0dfA69E15782881CCb8C7` |
| Vulnerable vault | `0x63a0b6D9A2703F7E35aBAA829e4053942F67ED5d` |
| Safe vault | `0xAf2Fb6db6c844497fA2526E467Fa9dE1f9AC9f8E` |

Explorer: https://sepolia.basescan.org/address/0xA67822e77acC2bC8Ca58A58aaB372398e62D8D0F — click the Transactions tab and you can see all the real trades, including one that got auto-slashed for being fake. No wallet needed to look.
