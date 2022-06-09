# non-fungible-token(NFT)
This repository includes an example implementation of a non-fungible token contract which uses near-contract-standards and simulation tests.
# Building this contract
Run the following, and we'll build our rust project up via cargo. This will generate our WASM binaries into our res/ directory. This is the smart contract we'll be deploying onto the NEAR blockchain later.

./scripts/build.sh

# Testing this contract
We have some tests that you can run. For example, the following will run our simple tests to verify that our contract code is working.
Unit Tests

cd nft
cargo test -- --nocapture
Integration Tests Rust

cd integration-tests/rs
cargo run --example integration-tests
# Using this contract
 # Standard deploy
This smart contract will get deployed to your NEAR account. For this example, please create a new NEAR account. Because NEAR allows the ability to upgrade contracts on the same account, initialization functions must be cleared. If you'd like to run this example on a NEAR account that has had prior contracts deployed, please use the near-cli command near delete, and then recreate it in Wallet. To create (or recreate) an account, please follow the directions in Test Wallet or (NEAR Wallet if we're using mainnet).

In the project root, log in to your newly created account with near-cli by following the instructions after this command.

near login
To make this tutorial easier to copy/paste, we're going to set an environment variable for our account id. In the below command, replace MY_ACCOUNT_NAME with the account name we just logged in with, including the .testnet (or .near for mainnet):

ID=MY_ACCOUNT_NAME
We can tell if the environment variable is set correctly if our command line prints the account name after this command:

echo $ID
Now we can deploy the compiled contract in this example to your account:

near deploy --wasmFile res/non_fungible_token.wasm --accountId $ID
NFT contract should be initialized before usage. More info about the metadata at nomicon.io. But for now, we'll initialize with the default metadata.

near call $ID new_default_meta '{"owner_id": "'$ID'"}' --accountId $ID
We'll be able to view our metadata right after:

near view $ID nft_metadata
Then, let's mint our first token. This will create a NFT based on Olympus Mons where only one copy exists:

near call $ID nft_mint '{"receiver_id": "'$ID'", "token_metadata": { "title": "VBI GFS Near Developer Advanced Course Certificate", "media": "https://upload.wikimedia.org/wikipedia/commons/thumb/0/00/Olympus_Mons_alt.jpg/1024px-Olympus_Mons_alt.jpg", "copies": 1}}' --accountId $ID --deposit 0.1
And you can mint NFT with IPFS upload with INFURA ipfs-upload-client

ipfs-upload-client --id $INFURA_PROJECT_ID --secret $INFURA_PROJECT_SECRET /path/file/image.jpg | xargs -I{} near call $ID nft_mint '{"receiver_id": "'$ID'", "token_metadata": {"title":"VBI GFS Near Developer Advanced Course Certificate", "media": "https://ipfs.infura.io{}", "copies": 1}}' --accountId $ID --deposit 0.01
Transferring our NFT
Let's set up an account to transfer our freshly minted token to. This account will be a sub-account of the NEAR account we logged in with originally via near login.

near create-account alice.$ID --masterAccount $ID --initialBalance 10
Checking Alice's account for tokens:

near view $ID nft_tokens_for_owner '{"account_id": "'alice.$ID'"}'
Then we'll transfer over the NFT into Alice's account. Exactly 1 yoctoNEAR of deposit should be attached:

near call $ID nft_transfer '{"token_id": "0", "receiver_id": "alice.'$ID'", "memo": "transfer ownership"}' --accountId $ID --depositYocto 1
Checking Alice's account again shows us that she has the Olympus Mons token.
# Notes
The maximum balance value is limited by U128 (2**128 - 1).
JSON calls should pass U128 as a base-10 string. E.g. "100".
This does not include escrow functionality, as ft_transfer_call provides a superior approach. An escrow system can, of course, be added as a separate contract or additional functionality within this contract.
