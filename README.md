# TO-Monitor-and-Limit-Wi-Fi-Usage
#To monitor and limit Wi-Fi usage, you typically need administrative control over the network or access to a router or firewall that supports traffic management. Below is an example Python script to track Wi-Fi usage and limit access based on predefined rules using tools like psutil and blocking access with iptables (Linux systems).
import psutil
import subprocess
import time

# Define usage limits in MB
USAGE_LIMIT_MB = 500  # Example: 500 MB
STUDENT_IP = "192.168.1.101"  # Replace with the student's IP address
CHECK_INTERVAL = 60  # Check usage every 60 seconds

# Function to get Wi-Fi usage
def get_wifi_usage():
    net_io = psutil.net_io_counters(pernic=True)
    wifi_data = net_io.get('wlan0')  # Replace 'wlan0' with your Wi-Fi adapter name
    if wifi_data:
        bytes_sent = wifi_data.bytes_sent
        bytes_recv = wifi_data.bytes_recv
        return bytes_sent + bytes_recv
    return 0

# Function to block internet access for a specific IP
def block_ip(ip_address):
    try:
        subprocess.run(
            ["iptables", "-A", "OUTPUT", "-s", ip_address, "-j", "DROP"],
            check=True
        )
        print(f"Blocked internet access for IP: {ip_address}")
    except subprocess.CalledProcessError as e:
        print(f"Error blocking IP: {e}")

# Function to unblock internet access for a specific IP
def unblock_ip(ip_address):
    try:
        subprocess.run(
            ["iptables", "-D", "OUTPUT", "-s", ip_address, "-j", "DROP"],
            check=True
        )
        print(f"Unblocked internet access for IP: {ip_address}")
    except subprocess.CalledProcessError as e:
        print(f"Error unblocking IP: {e}")

# Main function to monitor and enforce usage limits
def monitor_and_limit_usage():
    initial_usage = get_wifi_usage()
    while True:
        time.sleep(CHECK_INTERVAL)
        current_usage = get_wifi_usage()
        usage_mb = (current_usage - initial_usage) / (1024 * 1024)  # Convert to MB

        print(f"Current usage: {usage_mb:.2f} MB")
        if usage_mb > USAGE_LIMIT_MB:
            block_ip(STUDENT_IP)
            print(f"Usage limit exceeded for IP {STUDENT_IP}. Access blocked.")
            break

# Start monitoring
if __name__ == "__main__":
    monitor_and_limit_usage()
