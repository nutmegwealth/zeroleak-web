# ZeroLeak

A marketplace where a security researcher sells a smart-contract exploit to the protocol that's actually vulnerable to it — and both sides get through the deal without having to trust each other.

**Live on Base Sepolia:** https://zeroleak-web.vercel.app/

Connect a wallet, list a finding, buy one, and watch it settle on-chain. Everything below actually runs; it's not a mockup.

## How it works, in one breath

Seller proves they hold a real exploit (zero-knowledge, checked on-chain) and seals the write-up. Buyer pays into escrow without seeing it. Seller reveals the key. A keeper replays the exploit on-chain and records whether it actually worked. Worked → seller paid. Didn't → buyer refunded, seller's stake slashed. No arbiter, no middleman.

---

## The vertical: selling bugs

I've done bug-bounty work, and the part that always got under my skin is the handoff. You find something serious and then you're stuck in a standoff. Show the protocol the bug first and they can say "thanks!" and pay you nothing — you just gave away your only leverage. Make them pay first and they're worried you'll hand them garbage. Today's bounty platforms smooth this over with a trusted middleman and a lot of goodwill, and the payouts are often a joke next to what the bug is really worth (or what a shadier buyer would pay for it).

So the vertical is coordinated vulnerability disclosure, but run as an actual market instead of a trust-me handshake. The buyer is the protocol that's exposed. The seller is the researcher holding the bug. The thing being traded is something neither side can safely show or inspect up front. That specific problem — *you have to pay for something you can't see, from someone you don't know* — is exactly what an on-chain escrow is good at.

## Trust assumptions

Being upfront about who you have to trust and who you don't, because that's the whole point.

What you **don't** have to trust:

- Me, or any middleman sitting on the money. The escrow is a public contract. Funds move by rule, not by anyone's decision.
- The seller's word that they've got a real bug. They prove it with a zero-knowledge proof that gets checked on-chain *before* the listing goes live, and the proof is tied to their address so nobody can grab someone else's proof and relist it as their own.
- The buyer's honesty about whether the exploit "worked." That was never going to be their call. The exploit gets replayed and the outcome is written to the chain.

What you **do** still have to trust (no hand-waving here):

- The impact oracle. Right now that's a single operator running a keeper that replays the exploit and posts the verdict. The contract enforces everything *after* the verdict, but you're trusting that keeper to be honest and online. That's the soft spot, and I get into it below.
- That the disclosure you receive is the one you paid for. The contract checks the revealed write-up against the exact commitment the ZK proof was bound to — a Poseidon hash has to match — so a seller who swaps in junk fails the check. You're trusting that "the hashes match" means "same content," which, cryptographically, it does.

## Biggest design decision

Turning *"did the exploit actually work?"* into an on-chain fact instead of an argument.

The easy version of this product settles fights with an arbiter: buyer says it's junk, seller says it's fine, some human picks a winner. I didn't want that anywhere near it. An arbiter drags back in the exact trusted middleman the whole thing is supposed to delete, and it turns every close call into a judgment call.

So instead there's an impact oracle. Once the disclosure is revealed, a keeper takes the exploit, replays it against the target, and records the plain outcome on-chain — did it drain the vault or not. If it worked, the seller gets paid. If it didn't, the contract refunds the buyer and slashes the seller's stake on its own, with nobody in the loop. Settlement stops being a negotiation and becomes something the chain already knows. Every other piece of the design bends around that one call.

## One important limitation

The oracle is centralized and narrow, and I'm not going to pretend it isn't.

*Centralized:* it's one keeper, one key. If it's offline, fresh listings just sit at "verifying impact" until it runs. If that key got compromised, someone could post false verdicts. The real fix is decentralizing it — several independent replayers, or an optimistic setup where anyone can challenge a verdict by putting up a bond — and that part isn't built yet.

*Narrow:* the keeper only knows how to replay the exploit classes it's been taught (reentrancy, against targets it understands). It can't yet take an arbitrary exploit against an arbitrary contract and decide whether it works. So the objective verification is real, but it isn't general — the marketplace only fully delivers for targets the oracle knows how to reproduce. Widening that replay harness is the actual hard problem sitting inside this project, and I'd rather name it than paper over it.

---

## What's deployed (Base Sepolia · chain ID 84532)

| | Address |
|---|---|
| Marketplace | `0xA67822e77acC2bC8Ca58A58aaB372398e62D8D0F` |
| ZK verifier (Groth16) | `0xe80BEda38b291AEa50D0dfA69E15782881CCb8C7` |
| Reentrancy vault (target) | `0x63a0b6D9A2703F7E35aBAA829e4053942F67ED5d` |
| Safe vault | `0xAf2Fb6db6c844497fA2526E467Fa9dE1f9AC9f8E` |

Explorer: https://sepolia.basescan.org/address/0xA67822e77acC2bC8Ca58A58aaB372398e62D8D0F — the Transactions tab is the receipts. Real listings, purchases, reveals, on-chain impact verdicts, finalizations, and at least one automatic slash, all sitting there without you needing to connect anything.
