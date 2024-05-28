<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-Premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within two Azure virtual machines.<br/>


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)



<h2>Deployment and Configuration Steps</h2>

<h3>Step 1: Setup Resources in Azure</h3>

- Create two virtual machines
	- If you need help creating your virtual machines, please see my tutorial [here](https://github.com/JustinPeguero/virtual-machine)
	- The first virtual machine will be the Domain Controller
		- Name: DC-1
		- Image: Windows Server 2022
		- Take note of the virtual network (vNet) that is automatically created
       
![Step 1 Pic 1](https://github.com/JustinPeguero/configure-ad/assets/170198869/e7ff086e-cb4d-4214-9869-9d1993113782)




	- Set DC-1's Virtual Network Interface Card (vNIC) private IP address to be static
		- Go to DC-1's network settings
		- Select Networking
		- Select the link next to Network Interface
		- Select IP Configurations > ipconfig1
		- Change the assignment from dynamic to static 
			- This ensures DC-1's IP address will not change
	   
![Step 1 Pic 2](https://github.com/JustinPeguero/configure-ad/assets/170198869/4a2c7542-d51d-4827-85f4-e201c0c815ac)
![Step 1 Pic 3](https://github.com/JustinPeguero/configure-ad/assets/170198869/2e3865a7-e074-49e9-be59-05c6c9846a72)
![Step 1 Pic 4](https://github.com/JustinPeguero/configure-ad/assets/170198869/8eac9567-4378-4514-9715-00ea9a6ec908)





	- The second virtual machine will be the Client
		- Name: Client-1
		- Image: Windows 10 Pro
		- Use the same resource group and vNet as DC-1

![Step 1 Pic 5](https://github.com/JustinPeguero/configure-ad/assets/170198869/25b8dd11-ba40-4675-8878-062a0c72a59c)
![Step 1 Pic 6](https://github.com/JustinPeguero/configure-ad/assets/170198869/d8556f86-b878-456c-b68f-9c1848091381)




<h3>Step 2: Ensure Connectivity Between the Client and Domain Controller</h3>

- Login to Client-1 using Microsoft Remote Desktop
- Search for Command Prompt and open it
- Ping DC-1's private IP Address (for example, 10.0.0.4)
	- Type "ping -t 10.0.0.4" into the command-line interface
	- The ping request continually  times out due to the firewall settings
		- To fix this, we need to enable ICMPv4 on DC-1's local Windows firewall


<img src="https://i.imgur.com/U6UOqj5.png" height="70%" width="70%" alt="Azure Free Account"/> 
	
- Login to DC-1 using Microsoft Remote Desktop
	- Start > Windows Administrative Tools > Windows Defender Firewall with Advanced Security > Inbound Rules
	- Sort the list by protocols
	- Find "Core Networking Diagnostics" and "ICMPv4" and enable these two inbound rules

![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/e6028f6e-1185-4651-af11-57976e0707b9)



- Log back into Client-1 and the command line will automatically begin pinging DC-1 successfully
    

<img src="https://i.imgur.com/890WIJB.png" height="70%" width="70%" alt="Azure Free Account"/> 


<h3>Step 3: Install Active Directory</h3>

- Log back into DC-1
	- Open Server Manager
	- Select "Add Roles and Features" > Follow the prompts
	- At Server Roles, check "Active Directory Domain Services."
		- Ignore how the picture below already says "Installed"
	- Select Add Features > select Next
	- Complete the installation

![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/27426a8e-729e-42b0-b1a2-760e47400a8d)
![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/7b2ef421-3114-442d-8d53-7c1c46e65a0e)



- At the top right of the Server Manager Dashboard, click on the flag
- Select "Promote This Server to a Domain Controller"


<img src="https://i.imgur.com/GOYiTFe.png" height="70%" width="70%" alt="Azure Free Account"/> 
	
 - Select "Add a New Forest"
 	- Root domain name: mydomain.com
- Select Next
- Create a password
- Select Next and follow the prompts
- Select Install to complete the installation


![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/3946df07-b621-4f51-8729-1913ee5fcd21)

	
- DC-1 will automatically restart
- Log back into DC-1 as user: mydomain.com\labuser               

![Step 3 Pic 5](https://github.com/JustinPeguero/configure-ad/assets/170198869/2857620e-58b4-4b71-a7b4-972e1f4c2317)

	


<h3>Step 4: Create an Admin and Normal User Account in Active Directory v1.15.8</h3>
     
- On DC-1, open Server Manager
- Click Tools at the top-right of the screen
- Select Active Directory Users and Computers

![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/29901c7e-25fa-4225-9259-a1640d1736b1)

	
- Right-click mydomain.com > New > Select Oranizational Unit (OU)
- Create two OUs
	- Name the first "_EMPLOYEES"
	- Name the second "_ADMINS"
	
![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/596af907-8e87-4b0c-ade8-130a7a9781cf)

	
	
- Right-click mydomain.com and click Referesh to sort the new organizational units to the top
- Go to the _ADMINS OU
- Right-click the name of the OU > New > User
	- First/Last name: Jane Doe
	- User login name: jane_admin
	- Select Next
	- Create a password
	- Uncheck all boxes
	- Select Next and then select Finish
![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/160a8c9f-8366-4d85-be55-fdc00085529d)
![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/5119097b-fbb5-423a-8f96-d22430517a30)


	
- Go to the _ADMINS OU
- Right-click Jane Doe > select Properties
	- Click the tab named "Member of" > select Add
	- Type in the names of your domain administrators
	- Select "Check Names" > OK > Apply
- Log out of DC-1 as "labuser" and log back in as “mydomain.com\jane_admin”


![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/e03827ac-a999-4ef3-b880-4e3977f5fa88)
![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/8085d2c7-c74b-4617-b836-1a2364e0c4a0)


 
     

<h3>Step 5: Join Client-1 to your domain (mydomain.com)
</h3>

- Go back to the Azure portal
- Navigate to the Client-1 Virtual Machine
- On the left-hand side of the screen select Networking
- Select the link next to the NIC > select DNS Server > Custom
- Type in DC-1's private IP address
- Click Save
- After it is done updating, select Restart and select Yes

![Step 5 Pic 1](https://github.com/JustinPeguero/configure-ad/assets/170198869/e91a8a39-a7eb-4be2-b214-af7e63bd7f9e)
![Step 5 Pic 2](https://github.com/JustinPeguero/configure-ad/assets/170198869/d3cd150c-53ca-49b5-a9d3-fbc19f24fed4)
![Step 5 Pic 3](https://github.com/JustinPeguero/configure-ad/assets/170198869/f6d5c2f2-1770-4fd1-be67-f26aa6b9be6f)




- Log back into Client-1 using Microsoft Remote Desktop as the original local admin (labuser)
- Right-click the Start menu and select System
- On right-hand side of the screen, select Rename This PC (Advanced) > Change
- Under "Member of," select Domain
- Type "mydomain.com" and select OK
	- Username: mydomain.com\jane_admin
	- Type in password and press OK
- Restart the computer 			


![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/f94120f9-6036-4273-bdc7-e982c5edbedd)
![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/1a1edb09-b561-4878-ba8d-026806c51c13)



<h3>Step 6: Setup Remote Desktop for non-administrative users on Client-1
</h3>

- Log back into Client-1
- Use mydomain.com\jane_admin
- Right-click the Start menu and select System
- On the right-hand side of the screen, select Remote Desktop
- Under User Accounts, click "Select Users That Can Remotely Access This PC > select Add
- Type in the name of your domain users
- Select "Check Names" > OK > OK

 
![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/83ce2a0b-c322-4067-a8a7-fab761efe765)
![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/b454ac88-c26c-425b-a51a-84f64825d466)



<h3>Step 7: Create as many additional users as you would like and attempt to log into Client-1 with one of the users' profiles
</h3>

- Log back into DC-1 as jane_admin
- Search for "Powershell_ise,"
- Right-click on Powershell_ise and open it as an administrator
- At the top-left of the screen select New Script and paste the contents of the following script into it
	- You can find the script [here](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1)

![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/e64c7cbb-6f4b-4c7d-a2c6-15681723e411)
![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/9f9f075d-9ae3-4b76-8e76-6be011182b61)



- Click the green arrow button near the top-middle of the screen
	- This will run the script
- Once the users have been created, go back to Active Directory Users and Computers > mydomain.com > _EMPLOYEES
		- You will see all the accounts that were created
- You can now log into Client-1 with one of the accounts that were created
	- Try logging into Client-1 as user "base.milu" using the password "Password1"

![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/5abcff45-9c93-4472-b727-25cf321370b5)
![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/cd70f6a1-6753-4741-ab47-f3551a5cf762)
![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/6bd09f8f-a5d7-4f3c-a054-d473c4eedf9f)
![Step 7 Pic 6](https://github.com/JustinPeguero/configure-ad/assets/170198869/92f197ab-3f59-42d7-87a1-034bd732bcae)
![image](https://github.com/JustinPeguero/configure-ad/assets/170198869/3a35218a-44a4-454c-bff5-632be7e727a5)



You have implementated on-premises Active Directory and created users within an Azure virtual machine!
