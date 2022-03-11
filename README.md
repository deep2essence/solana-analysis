# solana-analysis
## Playground
## Tutorials
* [ ] https://docs.solana.com/cli
* [x] https://soliditydeveloper.com/solana
* [ ] https://github.com/ezekiiel/simple-solana-program
* [ ] https://github.com/solana-labs/solana-web3.js
* [ ] https://github.com/solana-labs/solana-solidity.js
* [ ] https://blog.techchee.com/simple-tutorial-on-solana-rust-smart-contract-with-anchor-framework/
* [ ] https://www.bitcoininsider.org/article/127285/how-build-and-deploy-solana-smart-contract
* [ ] https://github.com/solana-labs/example-helloworld/blob/master/src/program-rust/src/lib.rs
* [ ] https://www.becomebetterprogrammer.com/create-solana-smart-contract/
* [ ] https://blog.chain.link/how-to-build-and-deploy-a-solana-smart-contract/
* [x] https://jamesbachini.com/solana-tutorial/
## Syntax
step|expl.
---|----
**solang**
[```install```](https://soliditydeveloper.com/solana)|```npm install @solana/solidity @solana/web3.js```<br>```sh -c "$(curl -sSfL https://release.solana.com/v1.8.5/install)"```<br>
```Solidity```2```Solana```|```solang ERC20.sol --target solana --output build```
[```deploy```](https://docs.solana.com/cli/deploy-a-program)|```solana-test-validator --reset --quiet```<br>```node deploy-erc20.js```

*- deploy-erc20.js*
```js
const { Connection, LAMPORTS_PER_SOL, Keypair } = require('@solana/web3.js')
const { Contract, publicKeyToHex } = require('@solana/solidity')
const { readFileSync } = require('fs')

const ERC20_ABI = JSON.parse(readFileSync('./build/ERC20.abi', 'utf8'))
const BUNDLE_SO = readFileSync('./build/bundle.so')

;(async function () {
    console.log('Connecting to your local Solana node ...')
    const connection = new Connection(
        // works only for localhost at the time of writing
        // see https://github.com/solana-labs/solana-solidity.js/issues/8
        'http://localhost:8899', // "https://api.devnet.solana.com",
        'confirmed'
    )

    const payer = Keypair.generate()
    while (true) {
        console.log('Airdropping (from faucet) SOL to a new wallet ...')
        await connection.requestAirdrop(payer.publicKey, 1 * LAMPORTS_PER_SOL)
        await new Promise((resolve) => setTimeout(resolve, 1000))
        if (await connection.getBalance(payer.publicKey)) break
    }

    const address = publicKeyToHex(payer.publicKey)
    const program = Keypair.generate()
    const storage = Keypair.generate()

    const contract = new Contract(connection, program.publicKey, storage.publicKey, ERC20_ABI, payer)

    console.log('Deploying the Solang-compiled ERC20 program ...')
    await contract.load(program, BUNDLE_SO)

    console.log('Program deployment finished, deploying ERC20 ...')
    await contract.deploy('ERC20', ['MyToken', 'MTO', '1000000000000000000'], program, storage, 4096 * 8)

    console.log('Contract deployment finished, invoking contract functions ...')
    const symbol = await contract.symbol()
    const balance = await contract.balanceOf(address)

    console.log(`ERC20 contract for ${symbol} deployed!`)
    console.log(`Wallet at ${address} has a balance of ${balance}.`)

    contract.addEventListener(function (event) {
        console.log(`${event.name} event emitted!`)
        console.log(
            `${event.args[0]} sent ${event.args[2]} tokens to
       ${event.args[1]}`
        )
    })

    console.log('Sending tokens will emit a "Transfer" event ...')
    const recipient = Keypair.generate()
    await contract.transfer(publicKeyToHex(recipient.publicKey), '1000000000000000000')

    process.exit(0)
})()

```
