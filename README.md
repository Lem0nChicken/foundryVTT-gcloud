# Guide to install FoundryVTT over Google Cloud VM


## Step 0 → Prerequisites:

- FoundryVTT
- A Google (Gmail) account or create one at https://accounts.google.com/signup
- A **valid credit/debit card or an UPI account** → used for identity verification, google will deduct and refund a small amount from your account; you won’t be charged anything during the free trial unless you manually upgrade
- Basic Linux CLI knowledge
- Google Free tier offers you:
    - 1 e2-micro instance per month (ONLY in certain U.S. regions)
    - 30 GB of HDD (standard) boot disk
    - 1 GB network egress (outbound) per month

## Step 1 → Downloading FoundryVTT:

- Download the NODE version of foundry and save it to your system, keep it aside for now

## Step 2 → Creating a Google Cloud Account:

- Go to https://cloud.google.com/
- Click **“Get started for free”** (top-right corner).
- Sign in with your **Google account** if prompted. 
You’ll be redirected to the **billing setup page**.

### a. **Country Selection**

- Choose your **country/region** from the dropdown.
- Check the **terms of service** box.
- Click **“Continue”**.

### b.  **Account Type**

- Choose **Individual** – For personal use

### c. **Name & Address**

- Fill in:
    - Name
    - Billing address
    - Phone number (sometimes optional)

### d. **Payment Information**

- Add a **credit or debit card or UPI**
- You may see a temporary $1 deduction for authorization that will be instantly refunded

- You’ll now see a confirmation page saying your **$300 free credit** is active (valid for 90 days).
- Click **“Go to Console”** to access your dashboard.
- Create a new project:
    - In the top menu bar, click the **project dropdown** (top-left).
    - Click **“NEW PROJECT”**. and enter project name
    - Your project is now ready to use.

## Step 3 → Creating your Virtual Machine

- Make sure you’ve selected your **project** (top-left project dropdown)
- Enable Compute Engine API:
    - In the left-hand **Navigation Menu** (☰), go to:
        - Compute Engine → VM instances
    - Click **“Enable”** to activate the Compute Engine API (wait ~1 minute)
- Create a New VM Instance:
    - After the API is enabled, click **“Create Instance”**
    - You'll now see the **VM configuration page**
- Configure Your VM:
    - **Name**:
        - Choose a name like `my-foundry-vm`
        - Lowercase letters, numbers, and dashes only
    - **Region & Zone** *(VERY important for free tier)* Choose between:
        - **us-west1** (Oregon)
        - **us-central1** (Iowa)
        - **us-east1** (South Carolina)

        ⚠️ Only these regions qualify for Free Tier usage

    - **Machine Configuration**:
        - **Series**: Select `E2`
        - **Machine Type**: Select `e2-micro` (this is the only one free)
    - **Boot Disk**:
        - Click **“Change”**:
            - **OS**: Use **Ubuntu 22.04 LTS (or whichever latest LTS version is present)**
            - **Disk Type**: Set to **Standard persistent disk (HDD)** — NOT SSD
            - **Disk Size**: Set to **30 GB or less**
            - Click **Select**
    - **Firewall**: Allow both HTTP and HTTPS traffic
    - Click **“Create”** at the bottom
    - If the VM instance is in suspended stage, make sure to start it by clicking on 3 dots at the end of its row
    - Done!, you got your VM Instance up and running
    - If needed, you can access this instance via in-browser SSH by clicking on `SSH` button in VM Instances page
    - You can Upload/Download files from here as well, but it is very slow, so I recommend using fileZilla
    - Recommendation (Optional): Run `sudo apt-get update`  when logging in fir first time

## Reminder: Following Configuration is required for free tier:

| Resource | Free Tier Limit | How to Stay Safe |
| --- | --- | --- |
| **Machine type** | Only `e2-micro` is free | Never select anything else |
| **Regions** | `us-west1`, `us-central1`, `us-east1` | Only use these |
| **Boot disk** | ≤ 30 GB HDD (not SSD) | Set manually |
| **Uptime** | 744 hours/month (1 VM) | Don’t run more than 1 instance |
| **Network** | 1 GB egress/month | Avoid large downloads/uploads |

## Step 4: Setting up connections to this VM instance:

- In your local machine, Install following softwares:
    - [**PuTTY**](https://www.putty.org/)
    - **PuTTYgen** (comes with PuTTY)
    - [**FileZilla**](https://filezilla-project.org/)
- Create an SSH Key Pair Using PuTTYgen
    - Open **PuTTYgen**
    - **Key type**: Leave as `RSA`, and bits as `2048`
    - Click **Generate**, move your mouse randomly in the blank area
    - Enter the linux which you want to use in `Key Comment`  Section
    - REMEMBER to enter a Key Passphrase (its like a password for your private key)
    - Once generated:
        - **Save private key with appropriate name** (e.g., `gcp-private.ppk`)
        - **Copy the entire public key** from the top box
        - Leave PuTTYgen open

### Connect Using PuTTY

1. Open PuTTY  
2. **Host Name**: `your-username@<external-ip>`  
3. Go to **Connection → SSH → Auth**  
   - Select your `.ppk` file  
4. Click **Open**  
5. Accept the host key → Enter your passphrase  
6. Set a password (optional but useful):

```bash
sudo passwd your-username
```

### Setup FileZilla (Optional but Recommended if you want to upload/download large files)

1. Open FileZilla  
2. Go to **Edit → Settings → SFTP**  
   - Click **Add key file**  
   - Select your `.ppk` file  
3. In the main window:
   - **Host**: `sftp://<external-ip>`  
   - **Username**: your Linux username  
   - **Password**: your password  
   - **Port**: `22`  
4. Click **Quickconnect**

> 💡 Recommended: Create a `Downloads` folder in your home directory and set proper permissions:
>
> ```bash
> mkdir ~/Downloads
> chmod 755 ~/Downloads
> ```

## Step 5 → Opening VM Firewall for FoundryVTT

1. Go to [Firewall Rules](https://console.cloud.google.com/networking/firewalls)
2. Click **Create Firewall Rule**
3. Use the following settings:

| Field                 | Value                          |
|-----------------------|--------------------------------|
| Name                  | `allow-port-30000`             |
| Network               | `default`                      |
| Targets               | All instances in the network   |
| Source IP Ranges      | `0.0.0.0/0`                    |
| Protocols and Ports   | `tcp:30000`                    |

4. Click **Create**

## Step 6 → Uploading Your Foundry

1. Option 1: If you installed FileZilla, Use **FileZilla** to upload your Foundry `.zip` file to the VM
2. Option 2: Wse the wget command on Foundry's download section, to download Node version's Zip file
3. Connect to the VM via **PuTTY**
4. Install `unzip` if needed:

```bash
sudo apt-get install unzip
```

4. Navigate to the upload location  
5. Create a folder:

```bash
mkdir FoundryVTT_main
```

6. Unzip the FoundryVTT archive:

```bash
unzip your-file.zip -d FoundryVTT_main
```

## Step 7 → Installing Node.js and PM2

1. SSH into the VM  
2. Update packages:

```bash
sudo apt update && sudo apt upgrade -y
```

3. Install Node.js (v20 from NodeSource):

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

4. Verify installation:

```bash
node -v
npm -v
```

5. Install **PM2**:

```bash
sudo npm install -g pm2
pm2 --version
```

## Step 8 → Running FoundryVTT with PM2

1. Start Foundry:

```bash
pm2 start <path-to-your-foundry-directory>/main.js --name=foundry
```

2. Set PM2 to auto-start on boot:

```bash
pm2 startup
pm2 save
```

3. Access FoundryVTT in your browser:

```
http://<your-external-ip>:30000
```

## ✅ All Done!

You now have FoundryVTT running on a **free Google Cloud VM**! 🎉
