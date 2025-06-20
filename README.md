# monero-research


Technical Report: The Safety and Practicality of Using MyMonero for Programmatic Monero Balance Monitoring

Executive Summary

Monitoring Monero (XMR) wallet balances programmatically poses unique challenges due to Monero's privacy-preserving architecture. Unlike Bitcoin or Ethereum, Monero transactions are opaque by design, requiring private keys (at minimum the private view key) to scan the blockchain and identify outputs belonging to a wallet. Running a full node and wallet RPC server is resource intensive and operationally complex. MyMonero offers a pragmatic, secure, and efficient alternative for most operational environments that require periodic Monero balance monitoring.

1. The Challenge of Programmatic Monero Balance Monitoring

Monero utilizes stealth addresses, ring signatures, and confidential transactions to obfuscate balances and transaction histories.

Public nodes (daemon RPC) cannot reveal wallet balances.

Wallet RPC servers require local private keys and full blockchain scanning, which entails:

Large storage (150+ GB and growing)

Constant synchronization maintenance

Security and access control considerations

For organizations that need scalable, lightweight, and reliable balance monitoring, full node management may be prohibitive.

2. MyMonero Architecture

MyMonero provides a light-wallet architecture:

Users supply their wallet address and private view key.

MyMonero's backend scans the blockchain on behalf of users to identify owned outputs.

The private spend key never leaves the user's control (unless voluntarily provided).

MyMonero servers cannot spend funds; they can only observe incoming transactions.

Secure communication via HTTPS/TLS is enforced.

3. Security Considerations

a. Trust Model:

MyMonero requires access to the private view key to perform scanning.

The view key enables read-only visibility but no spending ability.

The service is operated by well-known, reputable members of the Monero community (including the original Monero lead developer).

b. Threat Model Mitigation:

MyMonero cannot alter balances or initiate transactions.

Leakage of the view key only compromises transaction visibility, not funds.

For organizations concerned about privacy leakage to MyMonero servers, options like self-hosted OpenMonero can be explored.

c. Transport Security:

All API communications occur over HTTPS.

Client-side encryption and strong authentication mitigate man-in-the-middle attacks.

d. Rate Limits & Abuse Protection:

MyMonero implements rate limiting to protect its infrastructure.

Programmatic monitoring at responsible polling intervals (e.g., every 5-10 minutes) does not typically encounter service limits.

4. Operational Benefits

No Full Node Required: No need to maintain full blockchain copies.

Low Infrastructure Overhead: Simple API integration with minimal compute/storage requirements.

Programmatic Access: Well-documented API with predictable, deterministic responses.

Cost-effective: MyMonero provides service free for standard usage volumes.

Rapid Deployment: Full monitoring setup can be established in minutes.

5. Recommendations for Programmatic Balance Monitoring

Given MyMonero's security architecture, operational simplicity, and API accessibility, it is recommended as standard operating procedure for programmatically monitoring Monero wallet balances in scenarios where:

Spending capability is not required.

The organization accepts MyMonero as a trusted scanning provider.

Full node hosting is operationally undesirable.

Polling frequency is moderate and compliant with rate limits.

For high-security or high-volume enterprise scenarios, organizations should additionally consider:

Dedicated contracts with MyMonero for higher limits.

Self-hosted OpenMonero instances for fully private scanning.

Conclusion

MyMonero represents a highly secure and operationally efficient approach for programmatic Monero balance monitoring. Its architecture strikes a pragmatic balance between Monero's privacy model and the realities of operational monitoring needs. When used responsibly, MyMonero should be considered a safe and recommended standard for organizations needing reliable balance data without the burden of full node infrastructure.





## SOPS

Standard Operating Procedure (SOP): Onboarding Monero Wallets for Programmatic Balance Monitoring via MyMonero API

Purpose:

To establish a standardized, secure, and repeatable process for onboarding Monero (XMR) wallets into MyMonero for programmatic balance monitoring using the MyMonero API.

Scope:

This SOP applies to all Monero wallets that require automated balance monitoring within the organization without running a full Monero node.

Pre-requisites:

Access to MyMonero web interface (https://mymonero.com/) or MyMonero desktop/mobile app.

Possession of wallet credentials (public address, private view key, and private spend key if applicable).

Secure workstation with internet access.

API integration environment (server or application capable of making HTTPS requests).

Roles and Responsibilities:

Operations Team: Perform wallet import and verification.

Security Team: Validate key management, approve credential storage locations.

Development Team: Implement API polling logic within authorized frequency limits.

Step 1: Prepare Wallet Credentials

Obtain the following wallet information:

Monero public address.

Private view key.

(Optional) Private spend key (not required for monitoring).

Verify the authenticity and integrity of the credentials.

Ensure credentials are handled in accordance with internal key management policies.

Step 2: Import Wallet into MyMonero

Navigate to https://mymonero.com/ (or use the MyMonero desktop/mobile app).

Choose "Log In with Existing Wallet".

Enter the following:

Wallet name (for internal reference).

Private view key (mandatory for monitoring).

Private spend key (optional, recommended to leave blank for monitoring-only purposes).

Complete wallet import.

MyMonero will now scan the blockchain and associate outputs with the supplied view key.

Allow sufficient time for initial blockchain scan (typically minutes to hours depending on wallet history).

Step 3: Verify Wallet Sync and Balance Availability

Once imported, verify that MyMonero successfully displays current wallet balance and transaction history.

Confirm that balance reflects current known on-chain state.

Step 4: API Integration Setup

Acquire MyMonero API documentation (public API is currently unofficial but accessible).

Prepare the API request payload including:

Wallet address.

Private view key.

Example API call (for balance lookup):

curl -X POST https://api.mymonero.com:8443/get_address_info \
  -H 'Content-Type: application/json' \
  -d '{
    "address": "<YOUR_MONERO_ADDRESS>",
    "view_key": "<YOUR_PRIVATE_VIEW_KEY>"
  }'

Parse API response for total_received, locked_funds, and total_sent fields to calculate spendable balance.

Ensure secure credential storage for API keys and view keys.

Step 5: Implement Polling Logic

Implement scheduled polling within operational limits:

Recommended frequency: once every 5 to 15 minutes.

Avoid excessive polling to comply with MyMonero's rate limits.

Include robust error handling, retry logic, and alerting on failed API calls.

Log balance snapshots with timestamp for audit and reporting.

Step 6: Ongoing Monitoring and Maintenance

Regularly review API performance and balance consistency.

Review MyMonero service updates or rate limit policy changes.

Conduct periodic security reviews of stored credentials.

Rotate API credentials if compromise is suspected.

Security Notes:

The private view key is sensitive but non-spendable; still treat as confidential.

Store credentials in encrypted storage solutions (e.g., Vault, HSM-backed services).

Do not expose view keys in unsecured environments.

Ensure HTTPS is always enforced for API calls.



Alternate Plan: Cloud-Based Private Monero Node (Backup Procedure)

In the event that MyMonero service becomes unavailable or organizational policy requires full control of scanning infrastructure, a private Monero node can be deployed in the cloud.

Step A1: Cloud Infrastructure Preparation

Select a secure cloud provider (e.g., AWS, Azure, GCP, DigitalOcean).

Provision a virtual machine (recommended minimum: 8 vCPU, 16 GB RAM, 500 GB SSD storage).

Harden the server (firewall, SSH key-only access, monitoring, backups).

Step A2: Install Monero Daemon and Wallet RPC

Download official Monero binaries from https://www.getmonero.org/downloads/.

Install and configure the Monero daemon (monerod) to synchronize the full blockchain.

Allow sufficient time for initial blockchain sync (may take several days for full sync).

Step A3: Deploy Wallet RPC Server

Launch monero-wallet-rpc with wallet files and access credentials.

Configure API authentication (username/password or restricted IP ranges).

Secure wallet files with strict filesystem permissions and encrypted storage.

Step A4: Implement API Integration

Update balance monitoring scripts to query your private monero-wallet-rpc API endpoint.

Example API request (for get_balance):

curl -u <RPC_USER>:<RPC_PASSWORD> \
  -X POST http://<NODE_IP>:18082/json_rpc \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":"0","method":"get_balance","params":{}}'

Parse API responses similar to MyMonero API.

Step A5: Monitoring and Maintenance

Implement monitoring for node health, disk usage, and wallet sync state.

Regularly patch operating system and Monero software.

Periodically backup wallet files and blockchain data.

Audit access logs and credentials.

Security Notes:

The private view key is sensitive but non-spendable; still treat as confidential.

Store credentials in encrypted storage solutions (e.g., Vault, HSM-backed services).

Do not expose view keys in unsecured environments.

Ensure HTTPS is always enforced for API calls.

When running your own node, full access to private spend keys may be required depending on use case.
