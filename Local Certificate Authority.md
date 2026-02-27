## Table of Contents
    [Overview](#overview)

### Overview

    You can obtain trusted SSL certificates from vendors for a fee and from Let's Encrypt for free, but they require you to have a valid Internet domain name.  If your goal is to ease communication between services on a home network and don't need any system to be accessible from the internet, a local CA could be a no cost solution.

    All current browsers require an SSL certificate from a trusted certificate authority in order to display a webpage using https without displaying the Not Secure message in the URL bar.  Many services that you may want to run in a home lab like Gitlab, Kubernetes, and image registries want to communicate via https and can't if they aren't able to use a TLS/SSL certificate that can be validated.  Openssl has long provided the ability to create a self-signed cert, but many applications/services won't accept a self-signed cert as valid.  

    Creating a local certificate authority will allow you to create your own SSL certificates that will be treated as trusted by your browser, once you've updated it's trusted certificate store with the local CA certificate.  You will be able to create SSL certificates with a local DNS name so you can refer to systems by a name rather than an IP address which can make administration easier.

    Some systems, like Suse Harvester, Rancher, and Pi-hole will create their own certificates for 'internal' communications, but those certs won't be accepted by other systems.

    