# Python-sdk-API
from flask import Flask, request, jsonify
from solana.rpc.api import Client as SolanaClient
from web3 import Web3
from aptos_sdk import RestClient as AptosClient
from ton import TonClient
from sui_sdk import SuiClient


app = Flask(__Bit3un__)


TON_NODE_URL = "https://toncenter.com/api/v2/jsonRPC"
APTOS_NODE_URL = "https://fullnode.mainnet.aptoslabs.com"
SOLANA_NODE_URL = "https://api.mainnet-beta.solana.com"
ARB_NODE_URL = "https://arb1.arbitrum.io/rpc"
SUI_NODE_URL = "https://fullnode.mainnet.sui.io"


ton_client = TonClient(TON_NODE_URL)
aptos_client = AptosClient(APTOS_NODE_URL)
solana_client = SolanaClient(SOLANA_NODE_URL)
arb_client = Web3(Web3.HTTPProvider(ARB_NODE_URL))
sui_client = SuiClient(SUI_NODE_URL)

@app.route("/generate_address", methods=["POST"])
def generate_address():

    data = request.json
    network = data.get("network")
    
    if network == "TON":
        address = ton_client.create_wallet()
    elif network == "APT":
        address = aptos_client.create_account()
    elif network == "SOL":
        address = solana_client.generate_keypair().public_key
    elif network == "SUI":
        address = sui_client.create_address()
    elif network == "ARB":
        address = arb_client.eth.account.create().address
    else:
        return jsonify({"error": "Invalid network!"}), 400

    return jsonify({"address": address})

@app.route("/deposit", methods=["POST"])
def deposit():

    data = request.json
    network = data.get("network")
    address = data.get("address")
    
    if network == "TON":
        balance = ton_client.get_balance(address)
    elif network == "APT":
        balance = aptos_client.get_account_balance(address)
    elif network == "SOL":
        balance = solana_client.get_balance(address)
    elif network == "SUI":
        balance = sui_client.get_balance(address)
    elif network == "ARB":
        balance = arb_client.eth.get_balance(address)
    else:
        return jsonify({"error": "Invalid network!"}), 400

    return jsonify({"address": address, "balance": balance})

@app.route("/withdraw", methods=["POST"])
def withdraw():

    data = request.json
    network = data.get("network")
    to_address = data.get("to_address")
    amount = data.get("amount")
    
    if network == "TON":
        tx = ton_client.send_transaction(to_address, amount)
    elif network == "APT":
        tx = aptos_client.send_transaction(to_address, amount)
    elif network == "SOL":
        tx = solana_client.send_transaction(to_address, amount)
    elif network == "SUI":
        tx = sui_client.send_transaction(to_address, amount)
    elif network == "ARB":
        tx = arb_client.eth.send_transaction({
            "to": to_address,
            "value": Web3.toWei(amount, "ether")
        })
    else:
        return jsonify({"error": "Invalid network!"}), 400

    return jsonify({"tx": tx})
