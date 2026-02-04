# Cloud-Security-Practicals

This guide outlines the successful execution of Labs 1 through 9 of https://github.com/MicrosoftLearning/AZ500-AzureSecurityTechnologies



### **Lab 1: Managing Identity and Access**

**Task 1: Create Joseph Price (The Portal)**
1.  Search for **Microsoft Entra ID** and select it.
2.  Click **Users** > **New user** > **Create new user**.
3.  Enter **Joseph** as the user name and **Joseph Price** as the display name.
4.  Copy the password and click **Create**.
5.  Go to **Groups** > **New group**.
6.  Set Group type to **Security** and Name to **Senior Admins**.
7.  Click **No owners selected**, select **Joseph Price**. 
8.  Click **No members selected**, select **Joseph Price**.
9.  Click **Create**.

**Task 2: Create Isabel Garcia (PowerShell)**
1.  Open **Cloud Shell** and select **PowerShell**.
2.  Run these commands to create the user and group:
```powershell
$passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
$passwordProfile.Password = "Pa55w.rd1234"
Connect-AzureAD
$domainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
New-AzureADUser -DisplayName 'Isabel Garcia' -PasswordProfile $passwordProfile -UserPrincipalName "Isabel@$domainName" -AccountEnabled $true -MailNickName 'Isabel'
New-AzureADGroup -DisplayName 'Junior Admins43846135' -MailEnabled $false -SecurityEnabled $true -MailNickName JuniorAdmins
$user = Get-AzureADUser -Filter "UserPrincipalName eq 'Isabel@$domainName'"
Add-AzADGroupMember -MemberUserPrincipalName $user.UserPrincipalName -TargetGroupDisplayName "Junior Admins43846135"
```

**Task 3: Create Dylan Williams (Azure CLI)**
1.  Switch Cloud Shell to **Bash**.
2.  Run these commands:
```bash
DOMAINNAME=$(az ad signed-in-user show --query 'userPrincipalName' | cut -d '@' -f 2 | sed 's/\"//')
az ad user create --display-name "Dylan Williams" --password "Pa55w.rd1234" --user-principal-name Dylan@$DOMAINNAME
az ad group create --display-name "Service Desk" --mail-nickname "ServiceDesk"
USER=$(az ad user list --filter "displayname eq 'Dylan Williams'")
OBJECTID=$(echo $USER | jq '.[].id' | tr -d '"')
az ad group member add --group "Service Desk" --member-id $OBJECTID
```

**Task 4: Role-Based Access Control**
1.  Search for **Resource groups** and create one named **AZ500LAB01**.
2.  Go to **AZ500LAB01** > **Access control (IAM)** > **Add role assignment**.
3.  Select **Virtual Machine Contributor**. Click **Next**.
4.  Click **+ Select members**, search for **Service Desk**, and select it.
5.  Click **Review + assign**.



### **Lab 2: Virtual Networking**

**Task 1: Infrastructure Setup**
1.  Create a Virtual Network named **myVirtualNetwork** in Resource Group **AZ500LAB07**.
2.  Set the address space to **10.0.0.0/16** and a subnet named **default** at **10.0.0.0/24**.
3.  Create two Application Security Groups named **myAsgWebServers** and **myAsgMgmtServers**.

**Task 2: Network Security Rules**
1.  Create a Network Security Group named **myNsg**.
2.  Go to **myNsg** > **Subnets** > **Associate**. Select **myVirtualNetwork** and the **default** subnet.
3.  In **Inbound security rules**, add Port **80, 443** for Destination: **Application security group** (**myAsgWebServers**).
4.  Add Port **3389** for Destination: **Application security group** (**myAsgMgmtServers**).

**Task 3: Deploy and Label VMs**
1.  Create two VMs: **myVmWeb** and **myVMMgmt**. 
2.  Under **Networking**, set Public inbound ports to **None**.
3.  Once created, go to **myVmWeb** > **Networking** > **Application security groups** and add **myAsgWebServers**.
4.  Repeat for **myVMMgmt** and add **myAsgMgmtServers**.


### **Lab 3: Azure Firewall**

1.  Search for **Deploy a custom template**. Upload the **template.json** from lab files. Use Resource Group **AZ500LAB08**.
2.  Create an Azure Firewall named **Test-FW01** in the subnet **AzureFirewallSubnet**.
3.  Search for **Route tables**. Create **Firewall-route**.
4.  Go to **Routes** > **Add**. Name it **FW-DG**. Destination: **0.0.0.0/0**. Next hop: **Virtual appliance**. Address: **[Firewall Private IP]**.
5.  Go to **Subnets** > **Associate**. Select **Test-FW-VN** and **Workload-SN**.
6.  In the Firewall, add an **Application Rule** to allow **www.bing.com**.


### **Lab 4: Containers**

1.  In Cloud Shell (Bash), create an ACR:
```bash
az acr create --resource-group AZ500LAB09 --name az500$RANDOM --sku Basic
echo FROM nginx > Dockerfile
ACRNAME=$(az acr list --resource-group AZ500LAB09 --query '[].{Name:name}' --output tsv)
az acr build --image sample/nginx:v1 --registry $ACRNAME --file Dockerfile .
```
2.  Create an AKS cluster. On the **Integrations** tab, select your ACR.
3.  Upload **nginxexternal.yaml**. Run `code ./nginxexternal.yaml`. Replace `<ACRUniquename>` with your ACR name and save.
4.  Deploy: `kubectl apply -f nginxexternal.yaml`.


### **Lab 5: Storage Security**

1.  Create a Virtual Network with a **Public** and **Private** subnet.
2.  In the **Private** subnet settings, enable the Service Endpoint for **Microsoft.Storage**.
3.  Create a Storage Account. Under **Networking**, select **Enabled from selected virtual networks**. Add the **Private** subnet.
4.  Create a File Share. Click **Connect** and copy the PowerShell script.
5.  Create a VM in the Private subnet. Log in, open PowerShell, and paste the script. It should map the Z: drive successfully.


### **Labs 6-9: Security Operations (AZ500LAB131415)**

**Lab 6: Monitoring**
1.  Create a Log Analytics Workspace named **lgawIgnite**.
2.  Search for **Data Collection Rules**. Create **DCR1**. 
3.  Select **myVM** as the resource. Set the destination to **lgawIgnite**.

**Lab 7: Defender for Cloud**
1.  Search for **Microsoft Defender for Cloud** > **Environment settings**. 
2.  Select your subscription and turn the **Servers** plan to **On**.

**Lab 8: Just-In-Time Access**
1.  Go to **myVM** > **Configuration**. Click **Enable just-in-time**.
2.  To test, go to the **Connect** tab and click **Request access**.

**Lab 9: Microsoft Sentinel**
1.  Search for **Microsoft Sentinel**. Add it to **lgawIgnite**.
2.  In **Content Hub**, install **Azure Activity**.
3.  In **Data Connectors**, open the Azure Activity page and launch the **Policy Assignment wizard**. On the **Remediation** tab, check **Create a remediation task**.
4.  Deploy the **changeincidentseverity.json** template. 
5.  In the Logic App, click **Edit** and sign in to all four connections to remove the yellow warnings.
6.  Create an **Analytics Rule** in Sentinel. In the **Automated Response** tab, select your Logic App. 
7.  Delete a resource to trigger the alert and verify the incident appears in Sentinel.
