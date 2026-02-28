**Table of Contents**
* [Overview](#overview)
* [Assumptions/Notices](#assumptionsnotices)
* [Environment Setup](#environment-setup)
* [Create Local CA](#create-local-ca)

### Overview

    You can obtain trusted SSL certificates from vendors for a fee and from Let's Encrypt for free, but they require you to have a valid Internet domain name.  
    If your goal is to ease communication between services on a home network and don't need any system to be accessible from the internet, a local CA could be a no cost solution.  

    All current browsers require an SSL certificate from a trusted certificate authority in order to display a webpage using https without displaying the Not Secure message in the URL bar.  
    Many services that you may want to run in a home lab like Gitlab, Kubernetes, and image registries want to communicate via https and can't if they aren't able to use a TLS/SSL certificate that can be validated.  
    Openssl has long provided the ability to create a self-signed cert, but many applications/services won't accept a self-signed cert as valid.  

    Creating a local certificate authority will allow you to create your own SSL certificates that will be treated as trusted by your browser, once you've updated it's trusted certificate store with the local CA certificate.  
    You will be able to create SSL certificates with a local DNS name so you can refer to systems by a name rather than an IP address which can make administration easier.

    Some systems, like Suse Harvester, Rancher, and Pi-hole will create their own certificates for 'internal' communications, but those certs won't be accepted by other systems.

### Assumptions/Notices

    Do NOT expect to see best practices consistently utilized!
    I will NOT be covering how to configure Windows systems to utilize local CA certificates
    You will need to have more than a basic understanding of how to work in a Linux environment
    This probably not the only way to make things work in a home lab, it's just what I'm using
    There are other services you can run other than pi-hole, use whatever makes you happy
    There are countless combinations of hardware and software, I won't document all the possibilities
    If you're using tools, applications, operating systems, and versions other than what's listed, I may not be able to help.
    You can create a local CA without using Pi-hole or similar product
        - I wanted to be able to utilize DNS names so I'm using Pi-hole
        - Using Pi-hole on your network and as primary DNS server can help block malicious sites
    Goals:
        - Keep the process as simple as possible (I do a lot as root)
        - Provide enough information and detail so you can follow along 
        - Help you on your home lab journey by enabling foundational services that will provide benefits elsewhere
        -     


### Environment Setup

    See the Environment.md file for information on hardware and software being used.
    I chose to combine my Pi-hole service and local CA into one container.
        - Pi-hole doesn't require many resources 
        - The CA doesn't require resources except when generating certs, so made sense to me to double up
        - I chose a Debian container rather than full VM to conserve resources
    Create a container in Proxmox using Debian 12 template and set a static IP
        - # apt update
        - # apt upgrade (if updates are available)
        - # apt install curl (the curl command was not already installed in my container)
        
    Review the Pi-hole documentation to determine OS options and hardware prereqs: https://docs.pi-hole.net/main/prerequisites/
    Follow the Pi-hole installation instructions: https://docs.pi-hole.net/main/basic-install/
        - I accepted all defaults
        - # pihole setpassword  (reset the admin password. I set it to the same as root to keep it simple)
        - Verify that you can access the Pi-hole web admin interface

    Openssl is required to generate cryptographic keys
        - Check if it is already installed with: openssl version        
    I installed Tmux to allow multiple screens so I could work in more than one directory at a time
        - It's also beneficial if you need to visually compare files/output
    You will need to make sure SCP is installed to transfer files to different machines/containers/vms


### Create Local CA

    Log into the container and navigate to the /etc directory
        Note: If you are not logged in as root, you may need to prepend sudo to commands
    
    Create a folder structure to hold the various files needed in the process
        - homelabCA
        - homelabCA/certs
        - homelabCA/private
        - homelabCA/crl
        - homelabCA/newcerts
        - homelabCA/csr
        This folder structure can be created with one command: mkdir -p /etc/homelabCA/{certs,private,crl,newcerts,csr}
        Note: If you have tree installed, execute tree h* to verify that the directory structure has been created

    Change into the homelabCA directory
        Note: The private folder will store the CA private key.  chmod 700 /etc/homelabCA/private to restrict access to owner

    Copy the default openssl.cnf file to the homelabCA directory:# cp /etc/ssl/openssl.cnf /etc/homelabCA/
    Create a backup of the config file:# cp openssl.cnf openssl.cnf.bkup
    Edit the openssl.cnf file in nano (or editor of your choice)
     Note: You can use tmux to create two windows in the terminal split horizontally to view the backup and current file at the same time.  Helpful to verify or undo changes made by mistake
      # tmux
      # ctrl-b + " (to split horizontally)
        - Navigate to the [CA_default] section
        - Modify the dir line: dir = /etc/homelabCA
        - Uncomment the unique_subject line
        - Modify the certificate line: certificate = $dir/homelabRootCA.pem 
        
    

