# Monitoring &amp; Alerting solution for Calamari Network (Monitoring Stack)

>**The Polkadot & Kusama ecosystems have raised an incredible attention in the past few years.**

>**Calamari Network is a private layer built for the entire Kusama ecosystem. Built on the substrate framework, Calamari Network is natively compatible with other projects and parachain assets including wrapped major cryptoassets.**

>**Calamari, Manta Network's canary net, is the plug-and-play privacy-preservation parachain built to serve the Kusama world.**

>**The Calamari parachain will be Privacy Hub for Kusama with multiple privacy solutions for all web3 projects.**

>**Many individuals joined the adventure and set up their own validator node, which is extraordinary for decentralization. However, maintaining a validator node is a huge responsibility as you are basically securing millions of dollars on the blockchain. Health and security of your node have to be the top priority for you as a validator.**



**NOTE: THIS GUIDE PROVIDES HELPFUL MONITORING AND ALERTING SOLUTION FOR CALAMARI VALIDATORS, BUT MOST OF THE CONFIGURATION IS EXACTLY THE SAME FOR A POLKADOT, KUSAMA, OR ANY SUBSTRATE BASED NODE.**



## The set of services required:

## 1. Set UFW (open ports)

```sudo ufw allow 9100
sudo ufw allow 9090
sudo ufw allow 9093
sudo ufw allow 9615
sudo ufw allow 3000
