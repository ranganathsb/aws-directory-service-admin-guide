# Secure LDAP Communications<a name="ms_ad_ldap"></a>

Lightweight Directory Access Protocol \(LDAP\) is a standard communications protocol used to read and write data to and from Active Directory\. Some applications use LDAP to add, remove, or search users and groups in Active Directory or to transport credentials for authenticating users in Active Directory\. 

By default, communications over LDAP are not encrypted\. This makes it possible for a malicious user to use network monitoring software to view data packets over the wire\. This is why many corporate security policies typically require that organizations encrypt all LDAP communication\. 

To mitigate this form of data exposure, AWS Microsoft AD provides an option for you to enable LDAP over Secure Sockets Layer \(SSL\)/Transport Layer Security \(TLS\), also known as LDAPS\. With LDAPS, you can improve security across the wire and meet compliance requirements by encrypting all communications between your LDAP\-enabled applications and AWS Microsoft AD directory\.

## Enable LDAPS<a name="enableldaps"></a>

You must do most of the setup from the Amazon EC2 instance that you use to manage your AWS Microsoft AD domain controllers\. The following steps guide you through enabling LDAPS for your domain in the AWS Cloud\.


+ [Step 1: Delegate Who Can Enable LDAPS](#grantpermsldaps)
+ [Step 2: Set Up Your Certificate Authority](#setupca)
+ [Step 3: Create a Certificate Template](#createcustomcert)
+ [Step 4: Add Security Group Rules](#addgrouprules)

### Step 1: Delegate Who Can Enable LDAPS<a name="grantpermsldaps"></a>

To enable LDAPS, you must be a member of the Admins or AWS Delegated Enterprise Certificate Authority Administrators group in your AWS Microsoft AD directory or be the default administrative user \(Admin account\)\. If you prefer to have a user other than the Admin account setup LDAPS, then add this user to the Admins or AWS Delegated Enterprise Certificate Authority Administrators group in your AWS Microsoft AD directory\.

### Step 2: Set Up Your Certificate Authority<a name="setupca"></a>

Before you can enable LDAPS, you must create a certificate issued by a Microsoft Enterprise Certificate Authority \(CA\) server that is joined to your AWS Microsoft AD domain\. Once created, the certificate must be installed on each of your domain controllers in that domain\. This certificate lets the LDAP service on the domain controllers listen for and automatically accept SSL connections from LDAP clients\. 

**Note**  
LDAPS with AWS Microsoft AD does not support certificates that are issued by a standalone CA\. 

Depending on your business need, you have the following options for setting up or connecting to a CA in your domain: 

+ **Create a subordinate Microsoft Enterprise CA** – \(Recommended\) With this option you can deploy a subordinate Microsoft Enterprise CA server in the AWS Cloud that uses EC2 so that it works with your existing root Microsoft CA\. For more information about how to set up a subordinate Microsoft Enterprise CA, see [Install a Subordinate Certification Authority](https://technet.microsoft.com/en-us/library/cc772192(v=ws.11).aspx) on the Microsoft TechNet website\.

+ **Create a root Microsoft Enterprise CA** – With this option you can create a root Microsoft Enterprise CA in the AWS Cloud using Amazon EC2 and join it to your AWS Microsoft AD domain\. This root CA can issue the certificate to your domain controllers\. For more information about setting up a new root CA, see [Install a Root Certification Authority](https://technet.microsoft.com/en-us/library/cc731183(v=ws.11).aspx) on the Microsoft TechNet website\.

For more information about how to join your EC2 instance to the domain, see [Join an EC2 Instance to Your Directory \(Simple AD and Microsoft AD\)](join_instance.md)\.

### Step 3: Create a Certificate Template<a name="createcustomcert"></a>

After your Enterprise CA has been set up, you can create a custom LDAPS certificate template in Active Directory\. The certificate template must be created with Server Authentication and Autoenroll enabled\. For more information, see [Create a New Certificate Template](https://technet.microsoft.com/en-us/library/cc753370.aspx) on the Microsoft TechNet website\. 

**To create a certificate template**

1. Log in to your CA with Admin credentials

1. Launch **Server Manager\.**, and then choose **Tools**, **Certification Authority**\.

1. In the **Certificate Authority** window, expand your CA tree in the left pane\. Right\-click **Certificate Templates** and then choose** Manage**\.

1. In the **Certificate Templates Console** window, right\-click **Domain Controller**, and then choose **Duplicate Template**\.

1. In the **Properties of New Template** window, switch to the **General** tab and change the **Template display name** to **ServerAuthentication**\.

1. Switch to the **Security** tab and choose **Domain Controllers** in the **Group or user names** section\. Select the **Autoenroll** check box in the **Permissions for****Domain Controllers** section\.

1. Switch to the **Extensions** tab, choose **Application Policies** in the **Extensions included in this template** section, and then choose **Edit**\.

1. In the **Edit Application Policies Extension** window, choose **Client Authentication** and choose **Remove**\. Click **OK** to create the **ServerAuthentication** certificate template, and then close the **Certificate Templates Console** window\.

1. In the **Certificate Authority** window, right\-click **Certificate Templates**, and choose** New**, **Certificate Template to Issue**\.

1. In the **Enable Certificate Templates** window, choose **ServerAuthentication**, and then click **OK**\.

### Step 4: Add Security Group Rules<a name="addgrouprules"></a>

In the final step, you must open the Amazon EC2 console and add security group rules so that your domain controllers can connect to your Enterprise CA to request a certificate\. To do this, you add inbound rules so that your Enterprise CA can accept incoming traffic from your domain controllers\. Then you add outbound rules to allow traffic from your domain controllers to the Enterprise CA\.

Once both rules have been configured, your domain controllers request a certificate from your Enterprise CA automatically and enable LDAPS for your directory\. The LDAP service on your domain controllers is now ready to accept LDAPS connections\. 

**To configure security group rules**

1. Navigate to your Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2](https://console.aws.amazon.com/ec2) and sign in with Admin credentials\.

1. In the left pane, choose **Security Groups** under **Network & Security**\.

1. In the main pane, choose the AWS security group for your CA\.

1. Choose the **Inbound** tab, and then choose **Edit**\.

1. In the **Edit inbound rules** dialog box, do the following:

   + Choose **Add Rule**\. 

   + Choose **All traffic** for **Type** and **Custom** for **Source**\. 

   + Type your CA’s AWS security group in the box next to **Source**\. 

   + Choose **Save**\.

1. Now choose the AWS security group of your AWS Microsoft AD directory\. Choose the **Outbound** tab and then choose **Edit**\.

1. In the **Edit outbound rules** dialog box, do the following:

   + Choose **Add Rule**\. 

   + Choose **All traffic** for **Type** and **Custom** for **Destination**\. 

   + Type your CA's AWS security group in the box next to **Destination**\. 

   + Choose **Save**\.

You can test the LDAPS connection to the AWS Microsoft AD directory using the LDP tool\. The LDP tool comes with the Active Directory Administrative Tools\. For more information, see [Installing the Active Directory Administration Tools](install_ad_tools.md)\.

**Note**  
Before you test the LDAPS connection, you must wait up to 180 minutes for the subordinate CA to issue a certificate to your domain controllers\.

For additional details about LDAPS and to see an example use case on how to set it up, see [How to Enable LDAPS for Your AWS Microsoft AD Directory](https://aws.amazon.com/blogs/security/how-to-enable-ldaps-for-your-aws-microsoft-ad-directory/) on the AWS Security Blog\.