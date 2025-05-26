# H2U_contracts

H2U_contracts is the smart contract suite developed by Hydrogen Ukraine to support the development of green hydrogen infrastructure in Ukraine. This repository contains the on-chain logic for managing renewable hydrogen production, storage, and distribution, with a focus on transparency, traceability, and integration with European energy markets.

# 🌍 Project Background

Hydrogen Ukraine is leading multiple hydrogen valley initiatives in regions like Odesa and Zakarpattia. These projects aim to build renewable hydrogen plants with electrolysis capacities of up to 200 MW by 2030, leveraging solar and wind energy to produce green hydrogen for both domestic use and export to the EU .

## The H2U_contracts repository provides the blockchain infrastructure to:
	•	Digitize hydrogen production and certification
	•	Enable secure and verifiable hydrogen trading
	•	Facilitate integration with EU energy systems
	
# 🧠 Core Functionalities

## The smart contracts in this repository are designed to:
	•	Track hydrogen production: Record metadata such as timestamp, energy source (solar, wind), and batch identifiers.
	•	Issue digital certificates: Generate verifiable tokens representing green hydrogen batches.
	•	Manage ownership and transfers: Facilitate secure transactions between producers, distributors, and consumers.
	•	Ensure regulatory compliance: Align with EU standards for renewable energy certification and trading.

## Instructions

### [register_produce](programs/h2u_contracts/src/instructions/producer/register_produce.rs) Instruction

The register_produce instruction registers a hydrogen production batch based on the electricity consumed by a producer. It performs the following operations :
	1.	Burns EAC Tokens:
The burn_eac function is called to burn a specific number of Energy Attribute Certificate (EAC) tokens from the producer’s associated token account (ATA), based on the amount of electricity (in kWh) they consumed. The number of burned tokens equals the consumed kWh, and is also used to update the on-chain EAC PDA state.
	2.	Calculates H₂ Token Output:
Hydrogen production is computed using a constant conversion rate (ELECTRICITY_PER_KG_H2 = 60). For every 60 kWh of electricity, 1 kg (1000 grams) of hydrogen is produced. The result is used to determine the amount of H₂ tokens to mint.
	3.	Mints H₂ Tokens:
The mint_h2 function mints the corresponding amount of hydrogen (H₂) tokens to the producer’s ATA, updating the H2Canister PDA to track the total and available hydrogen.
	4.	Initializes Associated Accounts (if needed):
Ensures the producer has ATAs for both the EAC and H₂ tokens. These are initialized using init_if_needed if they do not exist.

### 🏷️ [list_h2](programs/marketplace/src/instructions/list/list_h2.rs) — List H2 Tokens for Sale

Purpose:
Allows a hydrogen producer to list a specified amount of H2 tokens for sale on the marketplace at a given price. The tokens are transferred from the producer to the marketplace’s transfer manager account for escrow.

How it works:
	1.	Creates a new Listing PDA account using a seed of "listing" and the h2_canister address.
	2.	Stores:
	•	producer: the hydrogen producer’s public key.
	•	h2_canister: associated hydrogen canister for the batch.
	•	price: the price per unit (in USDC or another token).
	•	transfer_manager_ata: the marketplace escrow ATA to hold the H2 tokens.
	3.	Transfers the specified amount of H2 tokens from the producer’s associated token account (producer_ata) to the transfer_manager_ata (escrow).

Inputs:
	•	amount — number of grams of H2 tokens to list.
	•	price — price per gram in USDC (or other tokens).

Accounts:
	•	listing — new account that stores sale metadata.
	•	producer_authority — signer initiating the listing.
	•	producer — PDA of the hydrogen producer.
	•	h2_canister — PDA of the canister storing batch state.
	•	producer_ata — producer’s token account holding the H2.
	•	transfer_manager_ata — escrow ATA where tokens are transferred.
	•	token_program, associated_token_program, system_program — standard SPL programs.
	
### 💰 [sell_h2](programs/marketplace/src/instructions/sell/sell_h2.rs) — Sell H2 Tokens to a Buyer

Purpose:
Facilitates the sale of H2 tokens from a marketplace listing. Transfers USDC from the buyer to the producer and H2 tokens from escrow to the buyer.

How it works:
	1.	Validates that the buyer’s offered_price meets or exceeds the listing’s price.
	2.	Calculates total_payment = amount × offered_price.
	3.	Transfers:
	•	💵 USDC: From the buyer’s USDC ATA to the producer’s USDC ATA.
	•	🔄 H2 tokens: From the marketplace escrow (transfer manager ATA) to the buyer’s ATA.

Inputs:
	•	amount — number of grams of H2 to purchase.
	•	offered_price — buyer’s offered price per gram.

Accounts:
	•	listing — the active listing being purchased.
	•	buyer — signer paying for the tokens.
	•	producer — the hydrogen producer who will receive payment.
	•	buyer_usdc_ata — buyer’s token account holding USDC.
	•	producer_usdc_ata — producer’s USDC receiving account.
	•	buyer_ata — buyer’s token account to receive H2 tokens.
	•	transfer_manager_ata — marketplace’s escrow account holding H2 tokens.
	•	transfer_manager — signer (PDA) used to release H2 tokens from escrow.
	•	config — global market config with bump info.
	•	token_program, system_program — standard programs used for transfers.


## 🛠️ Technologies Used
	•	Solana Blockchain: High-performance blockchain platform for scalable decentralized applications.
	•	Anchor Framework: Rust-based framework for Solana smart contract development.
	•	TypeScript: Used for scripting, testing, and deployment automation.