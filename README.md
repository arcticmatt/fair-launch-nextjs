# fair-launch-nextjs

## Overview

This is adapted from https://github.com/metaplex-foundation/metaplex/tree/master/js/packages/fair-launch. That repo uses create-react-app, this uses Next.js.

See https://github.com/arcticmatt/candy-machine-mint-nextjs for some weird stuff you need to do to get the code working on Next.js.

## How to set things up

### 1. Make a new fair launch

The command should look like this.

```
fair-launch new_fair_launch -e devnet -k ~/.config/solana/id.json -u test04 -f 0.0001 -s 0.1 -pe 2 -pos "2021 Oct 30 11:30 PST" -poe "2021 Oct 30 11:50 PST" -pte "2021 Oct 30 12:20 PST" -ts 0.1 -n 2 -arbp 5000 -atc 1 -sd "2021 Nov 07 08:00 PST"
```

It will output a fair launch address.

Use `fair-launch new_fair_launch --help` to see what all the options are. Here's some additional info:

- `-u` is a 6 letter alphanumeric string, must be unique relative to the passed-in keypair.
- `-f` the fee people must pay to get in, defaults to SOL.
- `-s` and `-pe` are in SOL.
- `-ts` is the increments between `-s` and `-pe` you allow. E.g. if the starting price is `1`, the ending price is `2`, and the tick size is `0.1`, then people can bid `1.0`, `1.1`, `1.2`, etc. 
- `-arbp` is the % of your treasury you're going to hold in reserve for refunds. E.g. 5000 means people get a 50% refund if you mess up. It's in basis points, so `1` is the min and `10000` is the max (meaning 100% is held in reserve).
- `-atc`—when the mint supply hits this number or lower, you can get your money out of the treasury. E.g. if you start with 10K fair launch tokens, and you put 1000 for `-atc`, it means that when 9000 people buy NFTs and there's only 1000 left, you get to access the other side of the treasury and no more refunds will be given.
- `-sd` is the date by which if I haven't delivered NFTs you want to buy, you can start getting refunds out of the treasury.

TODO: still not really sure how refunds work....

### 2. Get the fair launch's mint address

```
fair-launch show -e devnet -k ~/.config/solana/id.json -f FAIR_LAUNCH
```

This will output info about the fair launch, including the mint address.

`FAIR_LAUNCH` is the fair launch address output by the previous command.

### 3. Create token account

```
spl-token create-account MINT
```

`MINT` should be the mint address from the previous command.

### 4. Create candy machine

See https://github.com/arcticmatt/candy-machine-mint-nextjs for full instructions.


The creation command should look something like this:

```
candy-machine create_candy_machine -e devnet -k ~/.config/solana/id.json -p 1 --spl-token MINT --spl-token-account ACC
```

`MINT` is from step #2, and `ACC` is from step #3 (the token account address).

Remember to set the go-live date of the candy machine! You can set it to some date in the past, since it only works with the fair launch tokens.

### 5. Update environment variables in `.env.local`

Self-explanatory.

### 6. Finally, you can start up the webapp!

`yarn dev`, same as usual.

## How to run the raffle

### 1. Create missing sequences

Run a command like this:

```
fair-launch create_missing_sequences -e devnet -k ~/.config/solana/id.json -f FAIR_LAUNCH
```

### 2. Create fair launch lottery

```
fair-launch create_fair_launch_lottery -e devnet -k ~/.config/solana/id.json -f FAIR_LAUNCH
```

This chooses the winners.

### 3. Verify lottery results

```
fair-launch show_lottery -e devnet -k ~/.config/solana/id.json -f FAIR_LAUNCH
```

Permissionless command, shows wallets who won and lost.

## After the raffle

### 1. Start phase three

```
fair-launch start_phase_three -e devnet -k ~/.config/solana/id.json -f FAIR_LAUNCH
```

When people go to the fair launch site, they should now see one of two things:

- If they lost the raffle, they should see a button to withdraw their funds
- If they won the raffle, they should see a button to mint (as long as the candy machine is live)

### 2. Punch and refund all

```
fair-launch punch_and_refund_all_outstanding -e devnet -k ~/.config/solana/id.json -f FAIR_LAUNCH
```

### 3. Withdraw funds

```
fair-launch withdraw_funds -e devnet -k ~/.config/solana/id.json -f FAIR_LAUNCH
```