# azure_test
Azure Scripts for devops


## develop a python script to run drift code on azure vm and fix issues
import os
import subprocess
import sys
from azure.identity import AzureCliCredential
from azure.mgmt.compute import ComputeManagementClient

# Azure configuration
SUBSCRIPTION_ID = "your_subscription_id"
RESOURCE_GROUP = "your_resource_group"
VM_NAME = "your_vm_name"
DRIFT_SCRIPT_PATH = "/path/to/drift_script.sh"

def get_vm_ip(compute_client, resource_group, vm_name):
    vm = compute_client.virtual_machines.get(resource_group, vm_name, expand='instanceView')
    for interface in vm.network_profile.network_interfaces:
        nic_name = interface.id.split('/')[-1]
        nic = compute_client.network_interfaces.get(resource_group, nic_name)
        for ip_config in nic.ip_configurations:
            if ip_config.public_ip_address:
                public_ip_id = ip_config.public_ip_address.id
                public_ip_name = public_ip_id.split('/')[-1]
                public_ip = compute_client.public_ip_addresses.get(resource_group, public_ip_name)
                return public_ip.ip_address
    return None

def run_drift_script(vm_ip, script_path):
    try:
        print(f"Connecting to VM at {vm_ip}...")
        subprocess.run(["scp", script_path, f"azureuser@{vm_ip}:/tmp/drift_script.sh"], check=True)
        subprocess.run(["ssh", f"azureuser@{vm_ip}", "chmod +x /tmp/drift_script.sh && /tmp/drift_script.sh"], check=True)
        print("Drift script executed successfully.")
    except subprocess.CalledProcessError as e:
        print(f"Error running drift script: {e}")
        sys.exit(1)

def main():
    credential = AzureCliCredential()
    compute_client = ComputeManagementClient(credential, SUBSCRIPTION_ID)

    print("Fetching VM IP address...")
    vm_ip = get_vm_ip(compute_client, RESOURCE_GROUP, VM_NAME)
    if not vm_ip:
        print("Failed to retrieve VM IP address.")
        sys.exit(1)

    print(f"VM IP address: {vm_ip}")
    run_drift_script(vm_ip, DRIFT_SCRIPT_PATH)

if __name__ == "__main__":
    main()
    
    
# This script enforces policy and configures the drift code on an Azure VM to fix issues.
# This script leverages the Azure SDK for Python to interact with Azure resources, retrieves the VM's public IP address, 
 and uses SSH to securely copy and execute the drift script on the VM to enforce policy and fix configuration drift issues.
# Ensure you have the required packages installed:

#The get_vm_ip function retrieves the VM's details using compute_client.virtual_machines.get.
#It iterates through the VM's network interfaces (vm.network_profile.network_interfaces).
#For each network interface, it fetches the associated public IP address by resolving the public_ip_address.id and retrieves the IP address using compute_client.public_ip_addresses.get.
#Returning the Public IP:

def get_vm_ip(compute_client, resource_group, vm_name):
    vm = compute_client.virtual_machines.get(resource_group, vm_name, expand='instanceView')
    for interface in vm.network_profile.network_interfaces:
        nic_name = interface.id.split('/')[-1]
        nic = compute_client.network_interfaces.get(resource_group, nic_name)
        for ip_config in nic.ip_configurations:
            if ip_config.public_ip_address:
                public_ip_id = ip_config.public_ip_address.id
                public_ip_name = public_ip_id.split('/')[-1]
                public_ip = compute_client.public_ip_addresses.get(resource_group, public_ip_name)
                return public_ip.ip_address
    return None


# Here is a Python script to initialize the Azure SDK for Python and authenticate using AzureCliCredential. This script demonstrates how to set up the SDK and retrieve a list of virtual machines in a specific subscription and resource group
# Run pip install azure-identity azure-mgmt-compute to install the required packages

#Steps to Use:
#Replace your_subscription_id and your_resource_group with your Azure subscription ID and resource group name.
#Ensure you have the Azure CLI installed and are logged in (az login).
#Install the required Python packages:

import os
from azure.identity import AzureCliCredential
from azure.mgmt.compute import ComputeManagementClient

# Azure configuration
SUBSCRIPTION_ID = "your_subscription_id"  # Replace with your Azure subscription ID
RESOURCE_GROUP = "your_resource_group"   # Replace with your Azure resource group name

def list_virtual_machines(subscription_id, resource_group):
    # Authenticate using Azure CLI credentials
    credential = AzureCliCredential()
    compute_client = ComputeManagementClient(credential, subscription_id)

    # List all virtual machines in the specified resource group
    print(f"Listing VMs in resource group '{resource_group}':")
    vms = compute_client.virtual_machines.list(resource_group)
    for vm in vms:
        print(f"- {vm.name}")

def main():
    # Ensure subscription ID and resource group are set
    if not SUBSCRIPTION_ID or not RESOURCE_GROUP:
        print("Please set your Azure subscription ID and resource group.")
        return

    # List VMs in the specified resource group
    list_virtual_machines(SUBSCRIPTION_ID, RESOURCE_GROUP)

if __name__ == "__main__":
    main()


# This script ingests Azure logs into Splunk and triggers a script based on specific alerts
import json
import requests

def ingest_azure_logs_to_splunk(azure_logs, splunk_url, splunk_token):
	headers = {
		'Authorization': f'Splunk {splunk_token}',
		'Content-Type': 'application/json'
	}
	response = requests.post(splunk_url, headers=headers, data=json.dumps(azure_logs))
	if response.status_code == 200:
		print("Logs successfully ingested into Splunk.")
	else:
		print(f"Failed to ingest logs into Splunk: {response.status_code}, {response.text}")

def trigger_script_based_on_alert(alert):
	if alert.get('type') == 'azure_vm':
		print("Triggering script for Azure VM alert...")
		# Add your script logic here
	else:
		print("No action required for this alert.")

# Example usage
azure_logs = [{"log": "example log data"}]
splunk_url = "https://splunk-instance:8088/services/collector"
splunk_token = "your-splunk-token"

ingest_azure_logs_to_splunk(azure_logs, splunk_url, splunk_token)

splunk_alert = {"type": "azure_vm", "message": "VM alert triggered"}
trigger_script_based_on_alert(splunk_alert)
