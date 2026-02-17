# Infrastructure-Automation-via-n8n-SSH
This repository provides a technical guide and configuration files for implementing remote server management using the n8n automation platform via the SSH protocol.
Contents

1.Deployment (Docker)
2.SSH Key Generation
3.Server Configuration
4.n8n Credential Setup
5.Troubleshooting

1. Deployment (Docker)
To deploy n8n in a containerized environment, use the following docker-compose.yml configuration:
ports:    
      - "5678:5678"
    volumes:
      - ./data:/home/node/.n8n           
      - ./custom_nodes:/home/node/.n8n/custom
      - /tmp:/tmp  
        environment:              
      - N8N_HOST=localhost                   
      - N8N_PORT=5678             
      - WEBHOOK_URL=http://localhost:5678/
      - N8N_ENCRYPTION_KEY=change-this-to-a-secure-key
      - EXECUTIONS_DATA_SAVE_ON_ERROR=all
      - EXECUTIONS_DATA_SAVE_MANUAL_EXECUTIONS=true
    restart: unless-stopped
    privileged: true                      
EOF

Note: If the n8n container needs to manage its host machine, use the Docker gateway IP: 172.17.0.1

2. SSH Key Generation
The n8n SSH node requires the RSA PEM format. Modern OpenSSH key formats (e.g., Ed25519) may cause compatibility issues.
Generate the key pair using the following command:

ssh-keygen -m PEM -t rsa -b 2048 -f ./n8n_deploy_key

n8n_deploy_key: Private key (to be used in n8n credentials).
n8n_deploy_key.pub: Public key (to be deployed on the target server).

3. Server Configuration
Add the public key to the authorized keys list on the target server and set the necessary file permissions to ensure security and connectivity:
bash

cat n8n_deploy_key.pub >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

4. n8n Credential Setup

    Navigate to Credentials -> Add Credential -> SSH Key.
    Host: Enter the target server IP address (or 172.17.0.1 for local Docker host).
    Port: 22.
    User: Enter the appropriate system user (e.g., root, ubuntu).
    Private Key: Paste the complete content of the n8n_deploy_key file, including the header and footer lines.

5.Troubleshooting
Error	                            Cause	                              Resolution

1.Unsupported key format      	  Incorrect private key encoding.	    Re-generate key using -m PEM flag.

2.getaddrinfo ENOTFOUND	        Host resolution failure.	          Verify IP address and check for trailing spaces.

3.Connection timed out	        Network or firewall restriction.	    Ensure port 22 is open on the target server.

4.zsh: set: no such option	  Shell incompatibility (Zsh).	    Execute command via Bash: bash -c "command".

5.Directory Specification                    -                          When using the SSH Execute Command node, always specify absolute paths (e.g., `/home/user/data`) or

use a `cd` command before your main instruction:                
Example: `cd /your/target/directory && ls -la`
   By default, SSH sessions may start in the user's home directory, so explicit path specification is mandatory for predictable results.


Command History Logging
By default, non-interactive SSH sessions do not record commands in the system history. To force logging, use the following syntax:

bash -c "export HISTFILE=~/.bash_history && set -o history && <COMMAND> && history -a"

### Telegram Integration
To receive notifications, you must provide your own Telegram Bot Token and Chat ID.
*   **Chat ID:** You must manually obtain your unique Telegram Chat ID (e.g., using @userinfobot) and paste it into the "Chat ID" field of the Telegram node.
*   **Bot Father:** Create your bot via @BotFather to get the API Token before running the workflow.

