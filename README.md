# ssignment: Automate Network Devices with GitHub Actions
## Overview
In this assignment, you will create an automated workflow for managing network devices (e.g., a router and switch) using GitHub Actions and a self-hosted runner in your local lab environment.

## Learning Objectives
Understand how to integrate network automation with a version-control system (GitHub).
Learn how to set up and configure a self-hosted GitHub Runner to run Actions locally.
Gain practical experience automating network devices (router, switch, or other network appliances) via scripts or Ansible playbooks.
Implement configuration backups to ensure version control of critical device configurations.

## Part 1: Lab Setup
1. Deploy 2–3 Network Devices
  * In your local environment/lab, set up at least two network devices (e.g., router and switch). This can be virtual or physical, depending on available resources.
  * If using virtual devices, you may use GNS3, EVE-NG, Cisco Modeling Labs (CML), or another emulator/virtualization solution that supports SSH or console-based access.

2. Initial Configuration
  * Ensure each device has basic IP connectivity.
  * Enable SSH on the devices, so they can be managed remotely via scripts or Ansible.
  * Record the IP addresses, usernames, and passwords (or SSH keys) for the devices. You’ll need these to automate them.


## Part 2: GitHub Repository Setup
1. Create a New GitHub Repository
  * Name it something like network-automation-lab.
  * Add a basic README.md describing the repository’s purpose.

2. Branch Strategy (Optional Advanced)
  * If you want to illustrate best practices, you can require that changes to device configuration happen in a feature branch, and are merged via Pull Request.

3. Add a .gitignore
  * Make sure you ignore any files containing credentials or secrets. These should not be committed to source control.

## Part 3: Self-Hosted GitHub Runner
1. Why a Self-Hosted Runner?
  * A self-hosted runner is necessary because your devices are likely not publicly accessible. GitHub’s standard cloud runners won’t be able to reach them.

2. Set Up the Runner
  * In your new GitHub repository, navigate to Settings → Actions → Runners → New self-hosted runner.
  * Choose the operating system you want to use (Linux, Windows, etc.).
  * Follow the instructions to install and configure the runner software on a local machine that has network access to your router/switch.
  * Once it’s set up, it should show up as “online” in your repository’s self-hosted runner list.

3. Label the Runner
  * You can assign labels to the runner (e.g. network-lab-runner). Your workflow in .github/workflows/ can target this label.

## Part 3: Ansible
1. Create an Ansible Playbook
  * Create a playbook (e.g., configure_devices.yml) that references an inventory file listing your devices.
  * Use tasks to push configurations (e.g., via the ios_config, ios_command, or similar modules for Cisco devices).
2. Backup Task
  * Use Ansible’s built-in backup feature or run a show run command that stores the output into a file.
  * Copy or move that file into your backups/ directory.
3. Commit or Upload the Backups
  * You can either commit them automatically or upload them as a GitHub Actions artifact for later retrieval.

## Part 4: GitHub Actions Workflow
1. Create a Workflow File
  * In your repository, create a directory: .github/workflows/.
  * Inside it, create a file named something like network_automation.yml.

2.Sample Workflow
  * You have an Ansible playbook (e.g., configure_devices.yml) in your repository.
  * You have an inventory file (e.g., inventory) in your repository that specifies the network devices.
  * You have a backups/ directory in your repo where the Ansible playbook will store configuration backups.


```
name: Network Automation

on:
  # Trigger on push to main or on a weekly schedule
  push:
    branches: [ "main" ]
  schedule:
    - cron: "0 3 * * 1"  # Example: runs every Monday at 3 AM

jobs:
  configure-network:
    runs-on: self-hosted
    # If your runner has a label, you can specify it here:
    # runs-on:
    #   - self-hosted
    #   - network-lab-runner

    steps:
      - name: Check out repo
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'

      - name: Install Dependencies
        run: |
          pip install ansible

      - name: Run Ansible Playbook
        run: |
          # Replace "inventory" and "configure_devices.yml" as needed
          ansible-playbook -i inventory configure_devices.yml

      - name: Commit and Push Backup
        run: |
          # This step will add and commit any new or changed files in "backups/" to your repo
          git config user.name "Your Name or CI Bot"
          git config user.email "your-email@domain.com"
          git add backups/
          # Make sure there is something to commit; otherwise, this might fail
          git commit -m "Automated backup commit" || echo "No changes to commit"
          git push
```

3. Secrets & Environment Variables

  * It’s strongly recommended to use GitHub Actions Secrets for credentials.
    
Example:
```
env:
  DEVICE_USERNAME: ${{ secrets.DEVICE_USERNAME }}
  DEVICE_PASSWORD: ${{ secrets.DEVICE_PASSWORD }}
```

4. Triggering the Workflow
  * Go to your GitHub Actions tab.
  * Select the workflow and click Run workflow to trigger manually (or push commits if configured to run on push).
  After the run:
  
  * Your script/playbook should apply any changes to the devices.
  * Sunning configs are gathered and saved to backups/.
  * The backups are committed to the repo.

  
## Part 5: Verification & Deliverables
## Test Run
  * rigger the workflow manually or via a commit.
  * Check the Actions logs to ensure it completes successfully.
    
## Check Config Backups
  * Verify the backups/ folder in your repository is updated with running configs.
    
## Screenshots & Logs
* Take screenshots of the Actions page showing success or errors.
* Show the contents of the backups/ folder.
* Provide these as part of your submission (or in your README.md).
  
## Submission
* Provide a link to your repository.
* Include a brief write-up or recorded demo explaining (video):
* How you set up the self-hosted runner.
* How your script/playbook works.
* Any challenges faced and how you solved them


Enhancements/Notes:
* Branching: Use a dev/staging branch for testing changes.
* Parameterize Config: Use YAML/JSON or Ansible group_vars for dynamic values (e.g., IPs, VLAN IDs).
* Device Inventory: Keep device info (IPs, credentials) in a separate file or Ansible inventory.
* Discord/Telegram/EMAIL/Slack/Teams Notifications: Send a notification after workflow success/failure. (optional) 
* Scheduled Workflows: Automate backups nightly or on a custom schedule.
