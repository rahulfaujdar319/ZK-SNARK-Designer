# ZK-SNARK-Designer

## Circom Logical Gate Project README

This README provides detailed instructions for creating a zero-knowledge circuit using the circom programming language to implement a specific logical gate. The project includes steps from writing and compiling the circuit to deploying a Solidity verifier on a testnet and verifying a proof.

## Project Objective

The objective of this project is to build a zero-knowledge proof system that demonstrates the operation of a logical gate with given inputs. This includes:

1. Writing a circuit using circom.
2. Compiling the circuit to generate necessary intermediaries.
3. Generating a proof with predefined inputs.
4. Deploying a Solidity verifier to a testnet (Sepolia or Mumbai).
5. Verifying the proof on the blockchain.

## Prerequisites

Before you start, ensure you have the following installed:
- Node.js and npm: [Download Node.js and npm](https://nodejs.org/en/download/)
- Circom and snarkJS: Install using npm:
  ```bash
  npm install -g circom snarkjs
  ```

## Step 1: Write the Circuit in Circom

Create a file named `circuit.circom` and define the circuit for the required logical gate (e.g., AND, OR, XOR). Here's a simple example for an AND gate:

```circom
template Main() {
    signal input A, B;
    signal output C;

    C <== A * B;
}

component main = Main();
```

Save this file in your project directory.

## Step 2: Compile the Circuit

Compile the circuit to generate the necessary circuit files:

```bash
circom circuit.circom --r1cs --wasm --sym
```

This command will create `circuit.r1cs` (the circuit constraints), `circuit.wasm` (WebAssembly executable for the circuit), and `circuit.sym` (symbolic information useful for debugging).

## Step 3: Generate a Proof

To generate a proof with inputs A=0 and B=1, first prepare the input file:

```json
// inputs.json
{
    "A": 0,
    "B": 1
}
```

Then, use snarkJS to calculate the witness and generate the zk-proof:

```bash
snarkjs wtns calculate circuit.wasm inputs.json witness.wtns
snarkjs groth16 prove circuit_final.zkey witness.wtns proof.json public.json
```

## Step 4: Deploy the Solidity Verifier

First, generate the Solidity verifier from the proof:

```bash
snarkjs zkey export solidityverifier circuit_final.zkey verifier.sol
```

Then, deploy this Solidity contract to Sepolia or Mumbai Testnet using a tool like Truffle or Hardhat:

```bash
// Example using Hardhat
npx hardhat run scripts/deploy.js --network sepolia
```

Ensure your `hardhat.config.js` is set up for the testnet deployment.

## Step 5: Verify the Proof on the Blockchain

Call the `verifyProof` method on your deployed verifier contract. You can use a script or interact directly via a console:

```javascript
// Example script to verify proof
async function verify() {
    const Verifier = await ethers.getContractFactory("Verifier");
    const verifier = await Verifier.attach("DEPLOYED_CONTRACT_ADDRESS");

    const { proof, publicSignals } = require('./proof.json');
    const calldata = await snarkjs.groth16.exportSolidityCallData(proof, publicSignals);

    const verified = await verifier.verifyProof(...JSON.parse(calldata));
    console.log("Proof verified: ", verified);
}

verify().catch((error) => {
    console.error(error);
    process.exit(1);
});
```

## Conclusion

By following these steps, you will have successfully implemented a zero-knowledge proof system for a logical gate, compiled the circuit, generated a proof, deployed a verifier smart contract to a testnet, and verified the proof using blockchain technology.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.
