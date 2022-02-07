# Install AWX on Debian 11 using k3s
Here is an easy setup to configure and deploy AWX Operator on a single node using k3s.

*This setup can be used in production environment.*

## Environment
Minimum configuration required that I am used to. 
 - **Operating System** : Debian 11
 - **CPU** : 2 cpu cores
 - **RAM** : 4 Gb
 - **Disk space** : 30Gb (enought to run script ansible)
## References
## Preparation
### Space
I advise to provide /var directory more space than /home following this configuration :
- / : 8.5 Gb (boot)
- /var : 12Gb
- /tmp : 1Gb
- /home : 5Gb
- swap space : 1 Gb
### Required packages
Install required packages to deploy k3s and AWX.
